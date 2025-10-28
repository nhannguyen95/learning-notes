AWS’s load balancer service, all types are fault-tolerant and scalable.
Load balancers can be used with more than web-servers - you can use load balancers in front of any systems that deal with request/response-style communication as long as the protocol is based on TCP.

## Application Load Balancer (ALB)

Load balancer that operates on Layer 7 (HTTP and HTTPs)

ALB is commonly used with Auto-scaling group from EC2 service. If the auto-scaling group adds or removes EC2 instances, it will also register new EC2 instances with the ALB and deregister EC2 instances that have been removed.

ALB consists of 3 required and 1 optional parts:
- Load balancer: define some core configs, like the subnets the load balancer runs in, whether the load balancer gets public IP addresses, whether it uses IPv4 or IPv4 and IPv6, and more.
- Listener: the listener defines the port and protocol that you can use to make requests to the load balancer. The listener can also terminate TLS. A listener links to a target group that is used as default if no other listener rules match the request.
- Target group: defines your group of backends. The target group is responsible for checking the backends by sending periodic health checks. Usually backends are EC2 instances, but could also be a Docker container running on EC2 Container Service or a machine in your data center paired with your VPC.
- Listener rule (optional): the rule can choose a different target group based on the HTTP path or host. Otherwise requests are forwarded to the default target group defined in the listener.

## Network Load Balancer (NLB)

Load balancer that operates on TCP.

## Classic Load Balancer (CLB)

Load balancer that operates on HTTP, HTTPS, TCP, TCP+TLS, the oldest type.

## Auto-scaling groups