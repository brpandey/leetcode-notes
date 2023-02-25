### Chat Service

The popular implementations of the chat service are the following:

- Whatsapp
- Facebook Messenger
- Slack
- Discord
- Telegram

---

#### Requirements

- The user (**sender**) can start a one-to-one chat conversation with another user (**receiver**)
- The sender can see an acknowledgment signal for receive and read
- The user can see the online status (**presence**) of other users
- The user can start a group chat with other users
- The user can share media (images) files with other users

---

#### Data storage

##### Database schema

<image src="imgs/chat-service-database-schema.png" alt="Chat service; Database schema" caption="Chat service; Database schema" >

- The primary entities are the groups, the presence, the users, the friends, and the messages tables
- The relationship between the groups and users tables is many-to-many
- The relationship between the users and friends table is many-to-many
- The relationship between the users and the messages table is 1-to-many
- The relationship between the users and the presence table is 1-to-1
- The friends table is the join table to show the relationship between users

##### Type of data store

- SQL database such as Postgres persists the metadata (groups, avatar, friends, WebSocket URL) for ACID compliance
- NoSQL data store ([LSM](https://en.wikipedia.org/wiki/Log-structured_merge-tree) tree-based) such as Cassandra is used to store the chat messages
- An object store such as AWS S3 is used to store media files (images) included in the chat conversation
- A cache server such as Redis is used to store the transient data (presence status and group chat messages) to improve latency
- The publish-subscribe server is implemented using a message queue such as Apache Kafka
- The message queue stores the messages for asynchronous processing (generation of chat conversation search index)
- Lucene-based inverted index data store such as Apache Solr is used to store chat conversation search index to provide search functionality

---

#### High-level design

- The user creates an HTTP connection for authentication and fetching of the relevant metadata
- The WebSocket connection is used for real-time bidirectional chat conversations between the users
- The set data type by Redis provides constant time complexity for tracking the online presence of the users
- The object store persists the media files shared by the user on the chat conversation

---

#### Workflow

<image src="imgs/chat-service-architecture.png" alt="Chat service; Architecture" caption="Chat service; Architecture" >

1. The client creates an HTTP connection to the load balancer
2. The load balancer delegates the HTTP request of the client to a server with free capacity
3. The server queries the authentication service for the authentication of the client
4. The server queries the SQL database to fetch the metadata such as the user groups, avatar, friends, WebSocket URL
5. The server queries the group cache to fetch the group chat messages
6. The server updates the message queue with asynchronous tasks such as the generation of the chat search index
7. The client creates a WebSocket connection to the gateway server for real-time bidirectional chat conversations
8. Consistent hashing (key = user_id) is used to delegate the WebSocket connection to the relevant chat server
9. The chat server delegates the WebSocket connection to the pub-sub server to identify the chat server of the receiver
10. The pub-sub server persists the chat messages on the chat data store for durability
11. The pub-sub server queries the presence service to update the presence status of the user
12. The presence service stores the presence status of the user on the presence cache
13. The pub-sub server delegates the WebSocket connection to the chat server of the receiver
14. The chat server relays the WebSocket connection through the CDN for group chat conversations
15. The CDN delegates the (group) chat message to the receiver
16. The pub-sub server invokes the push notification service to notify the offline clients
17. The server stores the chat media files on the object store and the URL of the media file is shared with the receiver
18. The admin service (not shown in the figure to reduce clutter) updates the pub-sub server with the latest routing information by querying the server
19. The mapping between the user_id and WebSocket connection ID (used to identify the user WebSocket for delegation) is stored in the memory of the chat server
20. The write-behind pattern is used to persist the presence cache on a key-value store
21. The cache-aside pattern is used to update the group cache
22. The presence cache is updated when the user connects to the chat server
23. Consistent hashing (key = user_id) is used to partition the presence cache
24. The offline client stores the chat message on the local storage until a WebSocket connection is established
25. The background process on the receiver can retrieve messages for an improved user experience
26. The clients must fetch the chat messages using pagination for improved performance
27. The pub-sub server stores the chat message on the data store and forwards the chat message when the receiver is back online
28. If the requirements are to keep the messages ephemeral on the server, a dedicated clean-up service is run against the replica chat data store to remove the delivered chat messages
29. If the requirements are to keep the messages permanently on the server, the chat data store acts as the source of truth
30. The one-to-one chat messages can be cached on the client for improved performance on future reads
31. The replication factor of the chat data store must be set to at least a value of three to improve the durability
32. The records in the presence cache are set with an expiry TTL of a reasonable time frame (5 minutes)
33. The sliding window algorithm can be used to remove the expired records on the presence cache
34. The last seen timestamp of the user is updated on the expiry of the records on the presence cache
35. The stateful services are partitioned by the partition keys user_id or group_id and are replicated across data centers for high availability
36. The hot shards can be handled by alerting, automatic re-partitioning of the shards, and the usage of high-end hardware
37. The Apache Solr can be used to provide search functionality
38. The publish-subscribe pattern is used to invoke job workers that run the search index asynchronously
39. The SQL database can be configured in single-master with semi-sync replication for high availability
40. The receive/read acknowledgment is updated through the WebSocket communication
41. The chat messages are end-to-end encrypted using a symmetric encryption key
42. The client creates a new HTTP connection on the crash of the server or the client
43. The gateway server performs SSL termination of the client connection
