## Table of contents

- [Introduction](#introduction)
- [Concepts](#concepts)
  - [Task Definition](#task-definition)
  - [Task](#task)
  - [Service](#service)
    - [Deployment Type](#deployment-type)
      - [Rolling update](#rolling-update)
  - [Container instance](#container-instance)
    - [Container agent](#container-agent)
  - [Cluster](#cluster)
  - [Launch Type](#launch-type)
  - [Capacity Provider](#capacity-provider)
- [Running ECS on Fargate](#running-ecs-on-fargate)
  - [Workloads](#workloads)
  - [Tasks](#tasks)
  - [Task networking](#task-networking)
  - [Task CPU and memory](#task-cpu-and-memory)
  - [Task storage](#task-storage)
  - [Logging](#logging)
  - [Service load balancing](#service-load-balancing)
  - [Task placement](#task-placement)
- [Running ECS on EC2](#running-ecs-on-ec2)
  - [Task Networking](#task-networking)
  - [Task Placement](#task-placement)
- [Development Topics](#development-topics)
  - [Using data volumes in tasks](#using-data-volumes-in-tasks)
  - [Scheduling ECS tasks](#scheduling-ecs-tasks)
  - [Service load balancing](#service-load-balancing)
  - [Service auto scaling](#service-auto-scaling)
  - [Task scale-in protection](#task-scale-in-protection)
  - [Service throttle logic](#service-throttle-logic)
  - [Tagging ECS resources](#tagging-ecs-resources)
  - [Events and EventBridge](#events-and-eventbridge)
- [References](#references)

---

## Introduction

ECS is a managed container orchestration service that helps us run and scale containerized applications. It is a regional service that simplifies the management involved with running containers in a highly available manner across multiple AZs.

Container images are stored in and pulled from container registries such as the Elastic Container Registry (ECR).

ECS can be used along with the following AWS services:
- IAM.
- EC2 auto scaling: can be used along side with a Fargate task within a service to scale in response to a number of metrics, or with an EC2 task to scale the container instances within your cluster.
- ELB.
- ECR.
- CloudFormation: you can define clusters, task definitions, and services as entities in an AWS CloudFormation script.

## Concepts

### Task Definition

A task definition is a text file in JSON format that describes 1 or up to 10 containers from your application.

The task definition is like a blueprint for your application and specifies various parameters for your application:
- Docker image to use with each container in the task.
- CPU, memory usage for each task (or each container within a task).
- The [launch type](#launch-type) to use, detemines the infrastructure that your tasks are hosted on.
- Docker networking mode for the containers in the task.
- Logging configuration to use for your tasks.
- Whether the task continues to run if the container finishes for fails.
- The command that the container runs when it's started.
- Data volumns used with containers in the task.
- IAM role that your tasks use.
- etc.

The parameters that you use depend on the launch type you choose for the task. To ensure task definitions validate for use with a specific launch type, you can specify the `requiresCompatibilities` flag when registering task definitions.

For a definitive list of task definition parameters, see [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html).

For a reference to a task definition template, see [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-definition-template.html).

It is recommended by AWS to deploy multiple containers into the same task definition if the following conditions are required:
- Containers share a common lifecycle (launched and terminated together).
- Containers must run on the same underlying host (containers reference each other on a localhost port).
- Containers are required to share resources.
- Containers share data volumes.

If these conditions aren't required, it is recommended to span your application across multiple task definitions by combining related containers into their own task definitions, each representing a single component, so you can scale, provision and deprovision them separately. As an example, you can group your frontend service container and a log streaming container into 1 task definition, and backend service container into another one.

See [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/example_task_definitions.html) for a list of task definition examples.

After you create a task definition, you can run it as a task or a service.

### Task

A task is the instantiation of a task definition within a cluster. You can specify the number of tasks to run on your cluster after creating a task definition for your application.

You can run a standalone task or as part of a service.

When a stask is started, either manually or as part of a service, it can pass through several states before it finishes on its own or stopped manually.
- Some are meant to run as batch jobs that progress from `PENDING` to `RUNNING` to `STOPPED`.
- Some can be part of a service and are meant to continue running indefinitely, or to be scaled up and down as needed.

When a task status changes are requested (e.g. stopping a task or updating the desired count of a service to scale it up or down), ECS container agent tracks these changed as last known status `lastStatus` and the desired status `desiredStatus` of the task. Both can be seen with the API or CLI.

Task lifecycle flow:
- PROVISIONING: ECS performs additional steps before the task is launched, e.g. provisoning ENI when use `awsvpc` network mode.
- PENDING: ECS waits on the container agent to take further action, a task stays in this state until there are available resources for it.
- ACTIVATING: ECS perform additional steps before it transitions to RUNNING, e.g. creating service discovery resources for the task (if the task has it configured).
- RUNNING: the task is successfully running.
- DEACTIVATING: revert ACTIVATING, ECS performs additional steps before the task is stopped..
- STOPPING: revert PENDING, ECS is waiting on the container agent to take further action, e.g. container sends SIGTERM signal to notify that application needs to finish and shut down.
- DEPROVISIONING: revert PROVISIONING, ECS performs additional steps before the task transitions to `STOPPED` state.
- STOPPED: the task has been successfully stopped.
- DELETED: a transition state when a task stops, this is not displayed in the console.

### Service

ECS service is used to run and maintain your desired number of tasks simultaneously in an Amazon ECS cluster.

If any of your tasks fail or stop for any reason, ECS service scheduler launches another instance based on your task definition in order to maintain the desired number of tasks in the service (e.g. when the underlying infrastructure fails).

Service scheduler is recommended for long running stateless services/applications. You can use task placement strategies and constraints to customize how the scheduler places/terminates tasks, see [Task Placement](#task-placement) and [Task Placement](#task-placement-1).

2 service scheduler strategies:
- REPLICA:
    - places and mantains the desired number of tasks across the cluster. By default, tasks are spread across AZs.
    - For a service that runs tasks on Fargate, you don't need to specify task placement strategies or restraints.
- DAEMON:
 - deploys exactly 1 task on each active container instance that meets all of the task placement constraints you specify in the cluster. There's no need to specify a desired number of tasks, a task palcement strategy, or Service Auto Scaling policies in this trategy.
 - Fargate tasks do not support this strategy.

For a definitive list of service definition parameters, see [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_definition_parameters.html).

Also see [Service load balancing](#service-load-balancing-1).

#### Deployment Type

An ECS deployment type determines the deployment strategy that your service uses.

3 deployment types:
- Rolling update.
- Blue/green.
- External

##### Rolling update

ECS service scheduler replaces currently running tasks with new tasks.

`minimumHealthyPercent`: lower limit on the number of (current and new) tasks that should be running for a service during a deployment, as a percentage of the desired number of tasks for the service.

`maximumPercent`: upper limit on the number of (current and new) tasks that should be running for a service during a deployment, as a percent of the desired number of tasks for a service.

When a new service deployment is started/completed, ECS sends a service deployment state change event to EventBridge.

The rolling update deployment has 2 methods which provide a way for you to quickly identify when a deployment has failed, and then to optionally roll back the failure to the last working deployment:
- Circuit breaker: use when you want to stop a deployment when the tasks can't start/reach a steady state. There's an option that will automatically roll back a failed deployment to the last working state.
- CloudWatch alarms: use when you want to stop a deployment based on application metrics (when a specified CloudWatch alarm has gone into the ALARM state).

You can use either method or both methods together. When use both, the deployment is set to failed as soon as the failure criteria for either failure method is met.

### Container instance

When you run tasks with ECS using the EC2 launch type or Auto Scaling group capacity provider, the tasks are deployed on your active ECS container instances.

An ECS container instance is an EC2 instance that is running the ECS container agent and has been registered into an ECS cluster.

Tasks using the Fargate launch type are instead deployed onto infrastructure managed by AWS. So you might not have to interact directly with the concepts of container instance and container, but they're still there (?).

#### Container agent

The container agent runs on each container instance within ECS cluster.
- It is able to register the container instance into one of your clusters.
- It sends information about the current running tasks and resource utilization of your containers to ECS.
- It starts/stops tasks whenever it receives a request from ECS.

### Cluster

ECS cluster is a logical group of tasks, services, and that allows for shared capacity and common configurations. Your tasks and services are run on infrastructure that is registered to a cluster. The infrastructure can be provided by Fargate, EC2 instances or on-premise server. 

Clusters are Region specific and can be created within a new or existing VPC.

Possible states of a cluster:
- ACTIVE: cluster is ready to accept tasks, you can register container instances with the cluster if applicable.
- PROVISIONING: cluster has capacity providers associated with it and the resources needed for the capacity provider are being created.
- DEPROVISIONING: cluster has capacity providers associated with it and the resources needed for the capacity provider are being deleted.
- FAILED: cluster has providers associated with it and the resources needed for the capacity provider have failed to create.
- INACTIVE: cluster has been deleted. INACTIVE clusters may remain discoverable in your account for a period of time.

### Launch Type

Specifying a launch type when running a standalone task or create a service determines the infrastructure that the task or service is hosted on.

Available launch type:
- Fargate: run your containerized apps without the need of provisioning and managing the underlying infrastructure (serverless pay-as-you go option), you don't have to worry about scaling or patching the underlying EC2 instances, as each task is placed on its own compute and managed by AWS.
- EC2: run your containerized apps on EC2 instances that you register to your ECS cluster and manage yourself.
- External: run your containerized apps on on-premise servers or VMs that you register to your ECS cluster and manage remotely.

Launch type is one of the key differentiators for how you architecture your app on ECS. AWS guidance is in [Running ECS on Fargate](#running-ecs-on-fargate) and [Running ECS on EC2](#running-ecs-on-ec2).

### Capacity Provider

Launch type is a generic way to determine where your tasks get deployed (Fargate or EC2), and the choice is binary.

With capacity providers (CPs), you can launch your tasks across multiple strategies, enabling greater flexibility in how you run your containers. CPs are used to manage the infrastructure the tasks in your clusters use.

We often reference capacity provider strategies (CPSs) when we talk about capacity providers. The CPS determines how the tasks are spread across the cluster's CPs. This means you aren't tied to one way of launching your tasks. And example of CPSs is that we could choose to schedule a group of tasks to be spread across Fargate and Fargate Spot, or across EC2 and EC2 spot.

CPs also help to shift the focus from managing the underlying capacity to the application, especially for those who choose to run their tasks on self-managed EC2 instances (EC2 launch type) as the compute layer. This is because for those taking advantage of Fargate compute, the burden of managing and understading how to run and scale the compute for their clusters is lifted as the underlying compute layer is fully managed AWS. But for self-managed EC2 instances as the compute layer, the complexity of determining how to scale the compute layer in teandem with the tasks can be a burdern for teams to manage and maintain.

Read more [here](https://aws.amazon.com/blogs/containers/managing-compute-for-amazon-ecs-clusters-with-capacity-providers/).

## Running ECS on Fargate

### Workloads

Fargate launch type is suitable for the following workloads:
- Large workloads that require low operational overhead.
- Small workloads that have occasional burst.
- Tiny workloads.
- Batch workloads.

### Tasks

Each task that uses the Fargate launch type has its own isolation boundary that does not share underlying resources with any other tasks, these resources include: kernel, CPU, memory, ENI.

### Task networking

ECS task definitions for Fargate require `awsvpc` as network mode. This mode provides each task with its own elastic network interface and a primary private IPv4 address. This gives the task the same networking properties as EC2 instances.

When running a task or create a service with this mode, you must specify one or more subnets to attach the network interface and one or more security groups to apply to the network interface.

For a task in a public subnet to pull container images, a public IP address needs to be assigned to the task's elastic network interface, with a route to the internet or a NAT gateway that you can route requests to the internet.

For a task in a private subnet to pull container images, you need a NAT gateway in the subnet to route requests to the internet.

A network configuration is also required when creating a service or manually running tasks.

Read more about [task networking here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-networking.html).

### Task CPU and memory

ECS task definitions for Fargate require that CPU and memory are specified at task level. You can also specify them at container level but it's optional, most use cases are satisfied by only specifying at task level.

### Task storage

For ECS tasks hosted on Farget, EFS (for persistent storage) and bind mounts (for ephemeral storage) are supported.

### Logging

ECS task definitions for Fargate support `awslogs`, `splunk` and `awsfirelens` log drivers for log configuration.

The `awslogs` log driver configures your Fargate tasks to send log info to CloudWatch Logs.

### Service load balancing

ECS services on Fargate can optionally be configured to use ELB to distribute traffic evenly across the tasks in your service.

ECS services on Fargate support ALB and NLB load balancer types.

When you create a target group for these services, you must choose `ip` as the target type, not `instance`. This is because tasks that use the `awsvpc` network mode are associated with an elastic network interface, not an EC2 instance.

### Task placement

Task placement strategies and constraints aren't supported for tasks using Fargate launch type. Fargate tasks are spread across AZs.

## Running ECS on EC2

The EC2 launch type is suitable for large workloads that must be price optimized.

### Task Networking

The networking behavior of ECS tasks hosted on EC2 instances depends on the network mode defined in the task definition. It is recommended using `awsvpc` unless there's a specific need to use a different mode.

### Task Placement

When a task uses the EC2 launch type is launched, you can apply task placement constraints and strategies to customize how ECS places/terminates and runs your tasks.
- A task placement strategy is an algorithm for selecting instances for task placement or tasks for termination.
- A task placement constraint is a rule that's considered during task placement, e.g. placing tasks based on AZ or instance type.

## Development Topics

### Using data volumes in tasks

ECS supports the following data volume options for containers:
- EFS: simple, scalable, persistent file storage for use with your ECS tasks. Supported for tasks hosted on Fargate and EC2 instances launch types.
- FSx for Windows File Server volumes.
- Docker volumes: a Docker-managed volumn that's created under `var/lib/docker/volumes` on the host EC2 instance. Supported tasks hosted on for EC2 and external instances launch types.
- Bind mounts: a file or directory on a host, such as an EC2 instance or Fargate, is mounted into a container. Supported for tasks hosted on Fargate or EC2 instances.

### Scheduling ECS tasks

ECS provides the following task scheduling strategies:
- Service scheduler, suitable for long running stateless tasks and applications.
- Manually running, suitable for processes such as batch jobs and standalone tasks that perform work and then stop.
- Scheduled tasks (Cronjobs), suitable for tasks ran at set intervals in your cluster (`cron` expression can be used for more complicated scheduling, see [EventBridge - Cron expressions and rate expressions](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html)) or tasks that are started by and event (see [Event patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)).
- Custom schedulers: you create your own schedulers or use 3rd party ones.

### Service load balancing

You can optionally run your service behind a load balancer. The load balancer distributes traffic across the tasks associated with the service.

Features:
- Support tasks hosted on both Fargate and EC2 instances.
- ALB allow containers to use dynamic host port mapping (so multiple tasks from the same service are allowed per container instance).
- ALB support path-based routing and priority rules (so multiple services can use the same listener port on a single ALB).

It is recommended using ALB for ECS services so that you can take advantage of latest features.

### Service auto scaling

ECS leverages the Application Auto Scaling service to provide the ability to increase/decrease the desired count of tasks in your ECS service automatically.

ECS publishes CloudWatch metrics with your service's average CPU and memory usage. You can use these (and other CloudWatch metrics) to scale out/in your service.

Types of automatic scaling supported by ECS Service Auto Scaling:
- Target tracking scaling policies:
    - increase/decrease the number of tasks in the service based on a target value for a specific metric.
    - A target tracking scaling policy assumes that it should perform scale out when the specified metric is above the target value. Thus you cannot tell is to scale out when the specified metric is below the target value.
    - To ensure application availability, the services scales out proportionally to the metric as fast as it can, but scales in more gradually.
    - Example use case: scale your service based on CPU utilization metric provided by ECS.
- [Step scaling policies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-stepscaling.html): recommended using target tracking scaling policies instead.
- Scheduled scaling: increase/decrease the number of tasks in the service based on data and time.

Consider the following when using scaling policies:
- After a scaling policy is applied, we need to wait for it to take effect, this is called cooldown period.
    - For scale-in events, the intention is to scale in conservatively to project the application's availability, so scale in activities are blocked until the cooldown period (of the previous policy) has expired.
    - For scale-out events, the intention is to continuously but not excessively scale out:
        - If during the scale-in cooldown period, a scale-out activity occurs, Service Auto Scaling scales out the target immediately.
        - If during the scale-out cooldown period, a scale-out activity occurs, the scaling policy won't increase the desired capacity again unless either it is a larger scale out or the cooldown period ends.
- Application Auto Scaling turns off scale-in (run fewer tasks) processes while deployments are in progress, scale-out processes continue to occur unless suspended during a deployment.
- If a service's desired count is set below its minimum capacity, and an alamr triggers a scale in activity, Service Auto Scaling does not adjust the desired count (because it is already below its minimum capacity).
- If a service's desired count is set above its maximum capacity, and an alarm triggers a scale out (run more tasks) activity, Service Auto Scaling does not adjust the desired count (because it is already above the maximum capacity).

### Task scale-in protection

Sometimes you might want to safeguard mission-critical tasks from termination by scale-in events even during times of low utilization or during service deployments. For example:
- You have a queue-processing async app such as video transcoding job where some tasks need to run for hours even when cumulative service utilization is low.
- You have a gaming app that runs game servers as ECS tasks that need to continue running even if all users have logged-out to reduce startup latency of a server reboot.
- When you deploy a new code version, you need tasks to continue running because it would be expensive to reprocess.

You can set task scale-in protection in the following ways:
- ECS container agent endpoint.
    - Recommended for tasks that can self-determine the need to be protected (e.g. queue-based or job-processing workloads).
    - When a container starts processing work, e.g. by consuming an SQS message, you can set the `ProtectionEnabled` attribute (through the task scale-in protection endpoint) from within the container. ECS will not terminate this task during scale-in events. After your task finishes its work, you can unset the attribute using the same endpoint, making the task eligible for termination during subsequent scale-in events.
- ECS API.
    - You can use this method if your app has a component that tracks the status of active tasks.
    - You can use the `UpdateTaskProtection` API to mark one or more tasks as protected.
    - Example: your app is hosting game server sessions as ECS tasks. When a user logs in to a session on the server (task), you can mark the task as protected. After the user logs out, you can either unset protection specifically for this task or periodically unset protection for similar tasks that no longer have active sessions, depending on your requirements to keep idle servers.

You can combine both approaches, e.g. use ECS container agent endpoint to set task protection from within a container and use ECS API to unset task protection from your external controller service.

Similarly, to get the protection status of tasks in an ECS service, you can do 1 of the following:
- ECS container agent endpoint: configure the container definition to use the Amazon task scale-in protection endpoint path.
- ECS API: use the `GetTaskProtectionAPI`.

Something to consider before using task scale-in protection:
- Use ECS agent endpoint instead of the API if possible, as ECS agent has inbuilt retry mechanisms and a simpler interface.
- When setting `ProtectionEnabled` to true, by default task are protected for 2 hours, you can customize this protection period by using the `expiresInMinutes` attribute, min and max value is 1 min and 2880 min (48 hours). You should determine how long a task would need to complete its requisite work and set this attribute accordingly. If the protection expiration is set longer than necessary, then you will incur costs and face delays in the deployment of new tasks.
- Deployment considerations:
    - If Service is using a rolling update and there are still (old) tasks whose `ProtectionEnabled` is not yet unset or expired, you can adjust the `maximumPercentage` parameter in deployment configuration to a value that allows new tasks to be created when old tasks are protected.
    - CloudFormation has 3 hour timeout for stack update operation. Therefore if you set your task protection for longer than 3 hours, then you CloudFormation deployment may result in failure (`UPDATE_FAILED`) and rollback.
    - ECS will send out service events if protected tasks are keeping a (rolling or blue/green) deployment from reaching steady state, so that you can take remedial actions.

### Service throttle logic

ECS service scheduler includes logic that throttles how often service tasks are launched if they repeatedly fail to start.

If tasks for a service repeatedly fail to enter the RUNNING state (from PENDING to STOPPED directly), the time between subsequent restart attempts is incrementally increased up to a maximum of 15 minutes. This behavior reduces the effect that failing tasks have on your infrastructure cost.

ECS doesn't stop a failing service from retrying. The service throttle logic also does not provide any user-tunable parameters.

Some common causes that initiate this throttle logic:
- A lack of resources (ports, memory, CPU units in your cluster) to host your task with.
- ECS container agent can't pull your task Docker iamge, this can be due to bad image name, tag, etc.
- Insufficient disk space on your container instance to create the container.

### Tagging ECS resources

You can assign ARN and a unique resource identifier to task definitions, clusters, tasks, services and container instances. You can also tag these resources with values that you define to help you organize, identify and categorize them (e.g. by purpose, owner, environment, etc.).

### Events and EventBridge

ECS sends the following types of events to EventBridge:
- container instance state change events.
- task state change events.
- service action.
- service deployment state change events.

These events and their possible causes are described in greater detail [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwe_events.html).

In some cases, multiple events are generated for the same activity. E.g. if a container instance is terminated, events are generated for the container instance, container agent connection status, and every task that was running on the container instance.

## References
- https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/intro.html