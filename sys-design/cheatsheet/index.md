# System Design Interview Cheat Sheet


## System Design Template

|                       |                               |                       |
| --------------------- | ----------------------------- | --------------------- |
| **Requirements**      | Functional Requirements       | Use Cases             |
|                       | Non-Functional Requirements   |                       |
|                       | Daily Active Users            |                       |
|                       | Read-to-Write Ratio           |                       |
|                       | Usage Patterns                |                       |
|                       | Peak and Seasonal Events      |                       |
| **Database**          | Data Model                    |                       |
|                       | Entity Relationship Diagram   |                       |
|                       | SQL                           |                       |
|                       | Type of Database              |                       |
| **API Design**        | HTTP Verb                     |                       |
|                       | Request-Response Headers      |                       |
|                       | Request-Response Contract     |                       |
| **Capacity Planning** | Query Per Second (Read-Write) |                       |
|                       | Bandwidth (Read-Write)        |                       |
|                       | Storage                       |                       |
|                       | Memory                        | Cache (80-20 Rule)    |
| **High Level Design** | Basic Algorithm               |                       |
|                       | Data Flow                     | Read-Write Scenario   |
|                       | Tradeoffs                     |                       |
|                       | Alternatives                  |                       |
|                       | Network Protocols             | TCP                   |
|                       |                               | UDP                   |
|                       |                               | REST                  |
|                       |                               | RPC                   |
|                       |                               | WebSocket             |
|                       |                               | SSE                   |
|                       |                               | Long Polling          |
|                       | Cloud Patterns                | CQRS                  |
|                       |                               | Publish-Subscribe     |
|                       | Serverless Functions          |                       |
|                       | Data Structures               | CRDT                  |
|                       |                               | Trie                  |
| **Design Deep Dive**  | Single Point of Failures      |                       |
|                       | Bottlenecks (Hotspots)        |                       |
|                       | Concurrency                   |                       |
|                       | Distributed Transactions      | Two-Phase Commit      |
|                       |                               | Sagas                 |
|                       | Probabilistic Data Structures | Bloom Filter          |
|                       |                               | HyperLogLog           |
|                       | Coordination Service          | Zookeeper             |
|                       | Logging                       |                       |
|                       | Monitoring                    |                       |
|                       | Alerting                      |                       |
|                       | Tracing                       |                       |
|                       | Deployment                    |                       |
|                       | Security                      | Authorization         |
|                       |                               | Authentication        |
|                       | Consensus Algorithms          | Raft                  |
|                       |                               | Paxos                 |
| **Components**        | DNS                           |                       |
|                       | CDN                           | Push-Pull             |
|                       | Load Balancer                 | Layer 4-7             |
|                       |                               | Active-Active         |
|                       |                               | Active-Passive        |
|                       | Reverse Proxy                 |                       |
|                       | Application Layer             | Microservice-Monolith |
|                       |                               | Service Discovery     |
|                       | SQL                           | Leader-Follower       |
|                       |                               | Leader-Leader         |
|                       |                               | Indexing              |
|                       |                               | Federation            |
|                       |                               | Sharding              |
|                       |                               | Denormalization       |
|                       |                               | SQL Tuning            |
|                       | NoSQL                         | Graph                 |
|                       |                               | Document              |
|                       |                               | Key-Value             |
|                       |                               | Wide-Column           |
|                       | Message Queue                 |                       |
|                       | Task Queue                    |                       |
|                       | Cache                         | Query-Object Level    |
|                       |                               | Client                |
|                       |                               | CDN                   |
|                       |                               | Webserver             |
|                       |                               | Database              |
|                       |                               | Application           |
|                       | Cache Update Pattern          | Cache Aside           |
|                       |                               | Write Through         |
|                       |                               | Write Behind          |
|                       |                               | Refresh Ahead         |
|                       | Cache Eviction Policy         | LRU                   |
|                       |                               | LFU                   |
|                       |                               | FIFO                  |

