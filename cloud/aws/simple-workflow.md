## Table of contents
- [Table of contents](#table-of-contents)
- [Concepts](#concepts)
  - [Domain](#domain)
  - [Workflows and Activities](#workflows-and-activities)
  - [Workflow Starters](#workflow-starters)
  - [Activity Workers](#activity-workers)
  - [Decider](#decider)
  - [Tasks](#tasks)
  - [Object Identifiers](#object-identifiers)
  - [Task Lists](#task-lists)
- [Topics](#topics)
  - [Polling for Tasks in SWF](#polling-for-tasks-in-swf)
  - [Life Cycle of an SWF Workflow Execution](#life-cycle-of-an-swf-workflow-execution)
- [References](#references)

---

## Concepts

### Domain

A named entity that will hold your workflows and activities, it provides a way of scoping SWF resources (workflows, activities, task lists) within your account.

Domains must have unique names within your AWS account. All of the workflows and activities for your application must be in the same domain to interact with one another.

### Workflows and Activities

A workflow is a set of activities, it is responsible for scheduling, coordinating and managing the execution of its activities (such as activities execution order) that can be run asynchronously across multiple computing devices.

An example of a workflow is an e-commerce order-processing workflow (which can involve both people and automated processes): verify order, charge credit card, ship order, record completion.

The [(activity) tasks](#tasks) in the example workflow above are sequential, but SWF also supports workflows with parallel processing of tasks, and the workflows can make decision once one or more of the parallel tasks have been completed.

You register each workflow/activity with SWF as an [workflow/activity type](#object-identifiers).

### Workflow Starters

A workflow starter is any application that can initiate workflow executions. In the e-commerce example, the workflow starter could be
- the website at which the customer places an order.
- the mobile app or system used by a customer service representative to place the order on behalf of the customer.

### Activity Workers

An activity worker is a process or a thread that performs the activity tasks that are part of your workflow. 

Each activity worker polls SWF for new tasks that are appropriate for that activity worker to perform. SWF assigns each activity task to exactly one activity worker. Once the task is assigned, no other activity worker can claim or perform that task. After receiving a task, the activity worker processes the task to completion and then reports to SWF that the task was completed and provides the result. SWF updates the workflow execution history with an event that indicates the task completed and then schedules a decision task to transmit the updated history to the decider.

The activity workers associated with a workflow execution continue processing tasks until the workflow execution itself is complete.

### Decider

At the heart of every workflow execution there is a decider, it is responsible for managing the execution of the workflow itself.

The decider receives decision tasks and responds to them, either by:
- scheduling new activities, cancelling and restarting activities, or
- setting the state of the workflow execution as complete, cancelled, or failed.

Before any decision tasks will be generated for the workflow to poll for, we need to start the workflow execution.

### Tasks

SWF interacts with activity workers and deciders by providing them with work assignments known as tasks.

There are 3 types of tasks in SWF:
- **Activity Task**: an activity task repensents one invocation of an activity, it tells an activity to perform its function (e.g. checking inventory or charge a credit card). The activity task contains all the information the activity worker needs to perform its function.
- **Lambda Task**: similar to an Activity Task, but executes a Lambda function instead of a SWF activity.
- **Decision Task**: a decision task tells a decider that the state of the workflow execution has changed so that the decider can determine the next activity that needs to be performed. The decision task contains the current workflow history.
    - A decicion task is scheduled whenever the state of the workflow changes (such as workflow started/completed, activity task completed/failed/timeout, etc. events).
    - Each decision task contains a paginated view of the entire workflow execution history, the decider analyzes this history and responds back to SWF with a set of decisions that specify what should happen next in the workflow execution.
    - Essentially, each decision task gives the decider an opportunity to assess the workflow and provide direction back to SWF.
    - SWF assigns each decision task to exactly one decider and allows only one decision task at a time to be active in a workflow execution.

### Object Identifiers

The following list describes how SWF objects (workflow executions, activities, etc.) are uniquely identified:
- **Workflow Type**: a registered workflow type is identified by (domain name, name, version).
- **Activity Type**: a registered activity type is identified by (domain name, name, version).
- **Decision Task and Activity Task**: each of these 2 tasks is identified by a unique task token. This token is generated by SWF.
- **Workflow Execution**: a single execution of a workflow is identified by the (domain, workflow ID, and run ID).

### Task Lists

Task lists provide a way of organizing the various tasks associated with a workflow. You can think of task lists as dynamic queues.

When a task is scheduled in SWF, you can specify a task list (queue) to put it in. Similarly when you poll SWF for a task you say which queue to get the task from.

TTask lists are dynamic in that you don't need to register a task list or explicitly create it through an action: simply scheduling a task creates the task list if it doesn't already exist.

There are separate lists for activity tasks and decision tasks. A task is always scheduled on only one task list; tasks are not shared across lists.

Decision task lists:
- Each workflow execution is associated with a specific decision task list.
- When a decider polls for a new decision task, the decider specifies a decision task list to draw from.
- A single decider could serve multiple workflow executions by calling `PollForDecisionTask` multiple times, each using a different task list, where each task list is associated to a workflow execution.
- The decider could poll a single decision task list that provides decision tasks for multiple workflow executions.
- You could also have multiple deciders serving a single workflow execution by having all of them poll the task list for that workflow execution.

Activity task lists:
- A single activity task list can contain tasks of different activity types.
- Tasks are scheduled on the task list in order. SWF returns the tasks from the list in order on a best effort basis (under some circumstances, the tasks may not come off the list in order).
- When an activity worker polls for a new task, it can specify an activity task list to draw from, and the activity worker will accept tasks only from that list. This way you can ensure that certain tasks get assigned only to particular activity workers. For example
    - You might create a task list that holds tasks that require the use of a high-performance computer and only workers running appropriate hardwares would poll that task list.
    - You might also create a task list for a particular geographic region, and only workers deployed in that region would pick up those tasks.
    - You could create a task list for high-priority orders and always check that list first.
- Assigning particular tasks to particular workers this way is called *task routing*.

## Topics

### Polling for Tasks in SWF

Deciders and activity workers communicate with SWF using *long polling*. They periodically initiates communication with SWF, notifying SWF of its availability to accept a task, and then specifies a task list to get tasks from.

If a task is available on the specified task list, SWF returns it immediately in the response. If no task is available, SWF holds the TCP connection open for up to 60 seconds so that, if a task becomes available during that time, it can be returned in the same connection. Otherwise, it returns an empty response and closes the connection. If this happens, the decider or activity worker should poll again.

### Life Cycle of an SWF Workflow Execution

From the start of a workflow execution to its completion, SWF interacts with actors (workflow starters, deciders, activity workers) by assigning them appropriate tasks, either activity tasks or decision tasks).

Read more [here](https://docs.aws.amazon.com/amazonswf/latest/developerguide/swf-dev-workflow-exec-lifecycle.html).

## References
- https://docs.aws.amazon.com/swf/index.html
- https://github.com/nhannguyen95/aws-swf-demo
