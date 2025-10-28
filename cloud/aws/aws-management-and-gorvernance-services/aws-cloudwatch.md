---
mindmap-plugin: basic
tags:
  - aws
  - cloudwatch
---

# AWS CloudWatch

## Introduction
- CloudWatch is a public service, can be used inside or outside AWS.
- CloudWatch offers metrics, events, logs, alarms for AWS resources.
	- CloudWatch Metrics
		- Automatically collect data from AWS services.
		- These metrics can be organized as dashboards.
	- CloudWatch Events
		- Delivers near real-time stream of system events that describes changes in AWS resources.
	- CloudWatch Logs
		- Log aggregation and monitoring service.
		- Support querying logs, monitoring logs from EC2 instances, defining log retention policy, etc.
	- CloudWatch Alarms
		- Alarms can be set up based on the metrics collected by CloudWatch Metrics.
		- The alarm can send a notification to Amazon SNS topic, trigger Auto Scaling action, etc.