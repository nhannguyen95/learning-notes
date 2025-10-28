---

  

mindmap-plugin: basic

  

---

# Stream Processing

## **Event Stream**
- **Event**
	- A small, self-contained, immutable object
	- Contains a timestamp indicating what happened at some point in time
	- An event may be encoded as a text string, JSON, binary form, etc. It allows you to
		- Store an event in a file, DB, etc.
		- Send the event to another node over the network
	- Related events are usually grouped together into a *topic* or *stream*.
- **Stream**
	- Data that is incrementally made available over time.
	- Examples
		- Unix's stdin, stdout
		- Filesystem APIs
		- TCP connections
		- Audio, video delivering over internet

## Messaging Systems
- In principal, a file or DB is sufficient to connect producers and consumers
	- Producers write to file/DB
	- Consumers periodically polls DB to check new events since last run.
		- Problem: polling becomes expensive for continual processing with low delays
		- It better for consumers to be notified
			- Problem: traditional DB don't support this very well
- A common approach for notifying consumers about new events is to use a *massaging system*
	- Direct messaging
	- **Message broker**
		- A database optimized for handling message streams
		- It runs as server, with producers and consumers connecting to it as clients
			- Producers write msgs to it
				- MBs generally allow unbounded queueing (as opposed to drop msgs/backpressure) when faced with slow consumers
				- Some MBs keep msgs in memory, some write them to disk so they are not lost in case of broker crash.
			- Consumers receive msgs by reading from it
				- As a consequence of queuing, consumers are generally async
				- Producers only waits for MB to confirm it has buffered/queued the msg
				- Multiple consumers patterns
					- *Load balancing*
						- Each msg is delivered to 1 consumer
					- *Fan-out*
						- Each msg is delivered to all consumers
					- 2 patterns can be combined
						- 2 groups of consumers each subscribe to a topic
						- 1 group uses load balancing
						- 1 group uses fan-out
		- Ack and Redelivery
			- To make sure msgs not lost in case consumers crash, consumers must send acks to MB when finished processing msg
			- If consumers don't send acks or time out, MB assumes msg wasn't processed; MB might then redeliver the msg (to another consumer)
			- A msg might still be redelivered even if the msg was already processed in some cases e.g. ack was lost in network
			- Msg redelivering effect: the order of msgs sent by the producer is not the same as the order they are delivered by the MB. This can be an issue if there are casual dependencies between msgs
		- Compared to DB
			- DB: keep data until it's explicitly deleted. MB: msg automatically deleted when successfully delivered to consumers
			- DB: support various ways for data searching (i.e. indexes). MB: consumers to subscribe to a subset of topics matching some pattern
			- DB: clients won't be notified if new data is written (unless they reexecute the query). MB: notify clients when data changes
		- Examples
			- RabbitMQ, ActiveMQ, Azure Service Bus, Google Cloud Pub/Sub
		- **Log-based Message broker**