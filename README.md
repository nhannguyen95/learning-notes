## Terms
- [Reliability, Scalability and Maintainability](./Chapter%201%3A%20Reliable%2C%20Scalable%2C%20and%20Maintainable%20Applications.md)
- Head-of-line blocking.
- Tail latency.
- Object-relational impedance mismatch.
- Object-relational mapping (ORM): frameworks that help reduce the amount of boilerplate code required for application code objects and database tables transition layer.
- Relational Model (SQL).
- Document Model.
- Graph-like Data Model.
- MapReduce: a programming model for processing large amounts of data in bulk across many machines, popularized by Google.
- Rolling upgrade (staged rollout).
- Backward compatibility: newer code can read data that was written by older code.
- Forward compatibility: older code can read data that was written by newer code.
- Representational State Transfer (REST): REST is not a protocol, but rather a design philosophy that builds upon the principles of HTTP. It emphasizes simple data formats, using URLs for identifying resources and using HTTP features for cache control, authentication, and content type negotiation.
- OpenAPI (Swagger): a definition format which can be used to describe RESTful APIs and produce documentation.
- Simple Object Access Protocol (SOAP): SOAP is an XML-based protocol for making network API requests. Although it is most commonly used over HTTP, it aims to be independent from HTTP and avoids using most HTTP features.
- Web Services Description Language (WSDL): an XML-based language used to describe APIs of a SOAP web service. WSDL enables code generation so that a client can access a remote service using local classes and method calls (which are encoded to XML messages and decoded again by the framework).
- Remote procedure calls (RPC): REST seems to be the predominant style for public APIs. The main focus of RPC frameworks is on requests between services owned by the same organization, typically within the same datacenter.
- Encoding (or serialization): the translation from the in-memory representation to a byte sequence.
- Decoding (or deserialization): the reverse of encoding.
- OAuth.
- Message broker (message queue, message-oriented middleware).
- Actor model: a programming model for concurrency in a single process.
- Distributed actor frameworks: integrate a message broker and the actor programming model into a single framework.
- [Replication](./replication.md): keeping a copy of the same data on multiple machines that are connected via a network.
- Partitioning (sharding):
- Eventual consistency.
- Read-after-write consistency (read-your-writes consistency).
- Cloud Native Computing Foundation (CNCF): open source software foundation dedicated to making cloud native computing universal and sustainable.

## Technologies
- Application-level technologies: Redis, Memcached.
- Apache Kafka
- ElasticSearch
- Solr
- Hadoop
- Hibernate: Java ORM.
- Relational databases: PostgreSQL.
- Document-oriented databases: MongoDB, RethinkDB, CouchDB, Expresso.
- Property graph model: Neo4j, Titan, InfiniteGraph.
- Triple-store model: Datomic, AllegroGraph.
- Declarative query languages for graphs: Cypher, SPARQL, Datalog.
- Data textual encoding formats (language-neutral): JSON, XML, CSV.
- Data binary encoding formats (language-neutral): Protocol Buffers, Thrift, Avro.
- Encoding built-in support libraries: `java.io.Serialization` (Java), Marshal (Ruby), pickle (Python). These libraries encode in-memory objects into language-specific byte sequence formats.
- RPC frameworks:
  - Thrift RPC (built-in).
  - Avro RPC (built-in).
  - gRPC: an RPC implementation using Protocol Buffers.
  - Finagle.
  - Rest.li: uses JSON over HTTP.
- Open source message brokers: RabbitMQ, ActiveMQ, HornetQ, NATS, Apache Kafka.
- Distributed actor frameworks: Akka, Orleans, Erlang OTP.
- OpenAPI: API definition technology.
- Envoy.
- Prometheus.
- GraphQL.
- Build automation tools: Bazel, Maven, Gradle.
- Websocket.
- WebRTC.

## Tools
- https://github.com/Netflix/chaosmonkey

## Resources
- [Designing Data-Intensive Application](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321).

## Tech template

What?

Use cases

Advantages/disadvantages

Real-world usage example(s)