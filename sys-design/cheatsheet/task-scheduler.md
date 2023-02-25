
### Distributed Job Scheduler

The keywords _job_ and _task_ are used interchangeably. Some of the popular job schedulers are the following:

- Apache Airflow
- Cron

---

#### Requirements

- Schedule and execute one-time or recurring jobs
- The job status and execution logs must be searchable
- The job scheduler is distributed

---

#### Data storage

##### Database schema

<image src="imgs/task-scheduler-db-schema.png" alt="Task scheduler; Database schema" caption="Task scheduler; Database schema" >

- The primary entities of the database are the Users table and the Tasks table
- The relationship between the Users and the Tasks tables is 1-to-many
- A database index must be created on the execution time of the tasks in the Tasks table to improve the latency of task execution

##### Type of data store

- The binary executable files of the tasks are stored in a managed object storage such as AWS S3
- The message queue such as Apache Kafka is used as a dead-letter queue and for deduplication of task execution requests
- The cache server such as Redis keeps the next immediate tasks to be executed in a min-heap data structure based on the task execution time
- The SQL database such as Postgres stores the task metadata
- The NoSQL data store such as Cassandra stores the task execution logs and task failure logs
- Apache Zookeeper is used for service discovery

---

#### High-level design

1. The task scheduler triages the tasks based on the execution time of the task and keeps the next immediate tasks on the task cache
2. The task scheduler stores the metadata of all tasks on the SQL database
3. The task executor fetches the next immediate task from the task cache and the worker node executes the task
4. The task execution logs are stored on the NoSQL data store

---

#### Write path

<image src="imgs/task-scheduler-write-path.png" alt="Task scheduler; Task scheduling workflow" caption="Task scheduler; Task scheduling workflow" >

1. The client creates an HTTP connection to the web server to schedule a task
2. The binary executable file of the task is stored on the object store using the presigned URL
3. The message queue stores the metadata of the task for asynchronous processing and fault tolerance
4. The task scheduler triages the tasks based on the execution time and assigns a unique ID to the scheduled task
5. If the execution time of the new task is within the next hour, the task is stored on the task cache as well as on the SQL database
6. The metadata of the tasks are persisted on the SQL database for durability and fault tolerance
7. If the execution time of the new task is within the next hour, the task executor is notified using the message queue publish-subscribe pattern
8. Task cache implements a min-heap data structure based on the execution time of the task using the Redis sorted set
9. The min-heap data structure on the task cache allows logarithmic time complexity O(log n) for the scheduling and execution of tasks and constant time complexity O(1) for checking the execution time of the next immediate task
10. The task executor runs periodically (every half an hour) to update the task cache with the next immediate tasks on the SQL database
11. LRU cache eviction is used for the task cache

---

#### Read path

<image src="imgs/task-scheduler-read-path.png" alt="Task scheduler; Task execution workflow" caption="Task scheduler; Task execution workflow" >

1. The task executor queries the task cache to fetch the next immediate task at the top of the min-heap
2. On failover, the task executor queries the SQL database replicas to fetch the next immediate task to be executed
3. The task executor persists the next immediate tasks on the message queue
4. Worker nodes fetch the metadata of the next immediate tasks on the message queue through the publish-subscribe pattern
5. The binary executable file of the task on the object store is fetched
6. The task execution log is stored on the message queue for asynchronous processing
7. The log aggregator service processes the execution logs
8. The task execution logs are persisted on the NoSQL data store
9. The metadata of failed tasks are stored on the dead-letter message queue for further analysis
10. The alerting service is notified of the failure and the failure detection service further processes the failed task data
11. The logs of the failed tasks are stored on the NoSQL data store
12. The client creates an HTTP connection to the web server to view the status of a task
13. The web server delegates the request to the task status service
14. The task status queries the task cache to view the status of an ongoing task
15. The task status queries the SQL database to view the status of an executed task
16. The task status queries the NoSQL data store to view the execution logs of a task
17. The task status queries the NoSQL data store to view the logs of a failed task
18. The system components such as the worker nodes send regular heartbeats to Apache Zookeeper for fault tolerance
19. The task cache uses the write-behind (write-back) caching pattern to persist the task status on the SQL database
20. Multithreading is implemented to parallelize and increase the throughput across a single machine
21. The [condition variable](https://learn.microsoft.com/en-us/windows/win32/sync/condition-variables) must be implemented on the task executor service to wake up the thread when the execution time of the next immediate task has elapsed
22. Atomic clocks can be used to synchronize the system time across worker nodes
23. Database rows must be locked by the task executor service to prevent the distribution of the same task to multiple task executor instances
24. The task executor must acquire a lock or a semaphore on the task cache data structure to prevent the distribution of the same task to multiple task executor instances
25. The task cache can be partitioned using consistent hashing (key = minute of the execution time)
