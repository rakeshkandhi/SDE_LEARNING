# 🧠 System Design Topics — Structured & Complete

---

## Fundamentals
- What is System Design?
- Monolithic vs Microservices Architecture
- Horizontal vs Vertical Scaling
- Load Balancing
- Caching Strategies
- Data Partitioning (Sharding)
- Consistency, Availability, Partition Tolerance (CAP Theorem)
- Latency vs Throughput
- Functional vs Non-functional Requirements
- SLAs, SLOs, SLIs
- Traffic Estimation (DAU, QPS, MAU)
- System Design Interview Phases
- Cost vs Performance Trade-offs

---

## Communication & Networking
- HTTP vs HTTPS
- WebSockets
- REST vs gRPC
- API Gateway
- Message Queues (Kafka, RabbitMQ, SQS)
- Publish/Subscribe Pattern
- CDN (Content Delivery Networks)
- DNS Resolution
- TLS/SSL
- HTTP/2 and HTTP/3
- TCP vs UDP
- Reverse Proxy vs Forward Proxy
- IP, Port, NAT Basics
- Rate Limiting & Throttling Patterns (Token Bucket, Leaky Bucket)
- Retry & Timeout Policies
- Circuit Breakers (Hystrix)

---

## Databases
- SQL vs NoSQL
- RDBMS (PostgreSQL, MySQL)
- NoSQL (MongoDB, Cassandra, DynamoDB, Redis)
- ACID vs BASE
- Eventual Consistency
- Database Indexing
- Data Replication (Leader-Follower, Multi-Leader)
- Backup & Restore Strategies
- Schema Design
- Sharding Techniques
- Time-Series, Graph, Columnar Databases
- OLTP vs OLAP Workloads

---

## Caching
- In-Memory Caches (Redis, Memcached)
- Cache Invalidation Strategies (Write-through, Write-around, Write-behind)
- Eviction Policies (LRU, LFU, FIFO)
- CDN Caching
- Lazy Loading vs Preloading
- Read-through, Write-through caching
- Cache Stampede & Cache Snowballing
- TTL Management
- Local (client-side) vs Distributed Caching

---

## Scalability & Reliability
- Load Balancers (HAProxy, Nginx, ELB)
- Auto-scaling Groups
- Health Checks
- Failover Strategies
- Service Discovery (Consul, Eureka)
- Active-Passive vs Active-Active Replication
- Circuit Breakers & Bulkheads
- Graceful Degradation
- Backpressure Management (in Queues, Streams)
- Horizontal vs Vertical Scaling

---

## Storage & File Systems
- Object Storage (Amazon S3, GCS)
- Blob Storage
- Distributed File Systems (HDFS, GFS)
- Local File Systems vs Network File Systems
- Storage Tiering (Hot vs Cold)
- Media Storage & Streaming (Chunking, Adaptive Bitrate)
- CDN for Static Assets
- Metadata Management in File Systems
- File Upload & Virus Scanning Architecture

---

## Design Patterns in Systems
- Event-Driven Architecture
- CQRS (Command Query Responsibility Segregation)
- Saga Pattern
- Strangler Fig Pattern
- Bulkhead Pattern
- BFF (Backend for Frontend)
- Circuit Breaker Pattern
- Retry Pattern
- Fan-out/Fan-in Design
- Distributed Locks
- Idempotency Pattern

---

## Observability
- Monitoring (Prometheus, Grafana)
- Logging (ELK Stack: Elasticsearch, Logstash, Kibana)
- Distributed Tracing (Jaeger, Zipkin, OpenTelemetry)
- Alerting Systems (PagerDuty, VictorOps)
- Structured Logging vs Unstructured Logs
- Heartbeats & Health Endpoints
- Metrics Collection (Time-Series DBs)
- Dashboarding & SLO Reporting

---

## Security
- Authentication vs Authorization
- OAuth 2.0, JWT, SAML
- API Key & Secret Management (Vault, AWS Secrets Manager)
- Encryption: at Rest and in Transit (TLS, AES, RSA)
- Hashing Techniques (bcrypt, SHA256, Argon2)
- XSS, CSRF, SQL Injection Mitigation
- DDoS Mitigation Strategies
- Rate Limiting for Abuse Prevention
- Audit Logging & Compliance (GDPR, HIPAA)
- Multi-Tenant Data Isolation
