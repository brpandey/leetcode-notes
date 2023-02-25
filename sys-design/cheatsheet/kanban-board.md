
### Kanban Board

Some of the popular implementations of a Kanban board are the following:

- Trello
- JIRA board
- Microsoft Planner
- Asana board

---

#### Requirements

- The user can create lists and assign tasks on the Kanban board
- The user can see changes on the Kanban board in near real-time
- The user can make offline changes on the Kanban board
- The Kanban board is distributed

---

#### Data storage

##### Database schema

<image src="imgs/kanban-board-database-schema.png" alt="Kanban board; Database schema" caption="Kanban board; Database schema" >

- The primary entities are the boards, the lists, and the tasks tables
- The relationship between the boards and lists tables is 1-to-many
- The relationship between the lists and tasks tables is 1-to-many
- An additional table is created to track the versions of the board

##### Type of data store

- Document store such as MongoDB persists the metadata of the tasks for improved denormalization support (ability to query and index subfields of the document)
- The transient data (board activity level) is stored on the cache server such as Redis
- The download queue for board activity is implemented using a message queue such as Apache Kafka

---

#### High-level design

1. The changes on the Kanban board are synchronized in real-time through a WebSocket connection
2. The offline client synchronizes the delta of changes when connectivity is reacquired

---

#### Workflow

<image src="imgs/kanban-board-architecture.png" alt="Kanban board; Architecture" caption="Kanban board; Architecture" >

1. The client-A creates a [WebSocket](https://en.wikipedia.org/wiki/WebSocket) connection on the load balancer to push changes in real-time
2. The load balancer ([HAProxy](http://www.haproxy.org/)) uses the round-robin algorithm to delegate the client connection to a web socket server with free capacity
3. The changes on the Kanban board are persisted on the document store
4. Consistent hashing (key = board ID) is used to delegate the WebSocket connection to the relevant publish-subscribe (pub-sub) server
5. The pub-sub server (Apache Kafka) delegates the WebSocket connection to the subscribed web socket servers
6. The web socket server fetches the changes on the document store
7. The WebSocket connection can make a duplex communication to the other listening clients through the load balancer
8. The client-B receives the changes on the Kanban board
9. The cache server stores the transient metadata such as the activity level of a session or the temporary authentication key
10. The download queue to replay the changes in the sequential order is implemented using a message queue
11. The changes are asynchronously propagated to the followers (replicas) of the document store to achieve eventual consistency
12. The CDN serves the single-page dynamic application to the client to improve latency
13. The single-page application is cached on the browser to improve the latency of the subsequent requests
14. An event-driven architecture might be a good choice for the instant propagation of updates
15. The client invokes the server logic through a thin wrapper over a WebSocket connection
16. LRU policy is used to evict the cache servers
17. The document store makes it relatively trivial to run different versions of the Kanban board against the same database without major DB schema migrations
18. The document store is replicated using the leader-follower topology
19. The online clients fetch the changes from the leader document store
20. The offline clients fetch the changes from the followers of the document store
21. The offline clients store the time-stamped changeset locally and send the delta of the changeset when the connectivity is reacquired
22. The changeset is replayed in sequential order to prevent data hierarchy failures
23. The client synchronizes only the recently viewed and the starred boards to improve the performance
24. A new TCP WebSocket must be established on the server failure
25. The last-write-win policy is used to resolve conflicts on the board
26. Exponential backoff must be implemented on the synchronization service to improve the fault tolerance
