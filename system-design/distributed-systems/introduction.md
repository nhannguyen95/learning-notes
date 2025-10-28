---
mindmap-plugin: basic
tags: []
---

# Introduction

## Concepts
- **Reliability**
	- Continue to work correctly, even when things go wrong
		- *Faults*: when a system component deviating form the spec
		- *Fault-tolerant* or *resilient*: systems that anticipates faults and can cope with them.
			- It only makes sense to tolerant ==certain types== of faults.
		- *Failure*: whole system stops working
			- Best to design fault-tolerant mechanisms that prevent faults from causing failures
			- Generally, tolerating faults is preferred to preventing faults, but in some cases (security) prevention is better
- **Scalability**
	- *Load parameters*: numbers used to describe system load
		- Application dependent
			- requests/sec
			- reads/writes
			- etc.
- **Maintainability**