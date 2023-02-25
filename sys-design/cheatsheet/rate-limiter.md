### API Rate Limiter

The internet scale distributed systems implement an API rate limiter for high availability and security.

---

#### Requirements

- Throttle requests exceeding the rate limit
- Distributed rate limiter

---

#### Data storage

##### Database schema

The NoSQL document store such as MongoDB is used to store the rate limiting rules and the throttled data

##### Type of data store

- The NoSQL document store such as MongoDB stores the rate-limiting rules and the throttled data
- The cache server such as Redis stores the rate-limiting rules and throttling data in-memory for faster lookups
- The message queue stores the dropped requests for analytics and auditing purposes

---

#### High-level design

- The cookie, user ID, or IP address is used to identify the client
- The rate limiter drops the request and returns the status code "429 too many requests" to the client if the throttling threshold is exceeded
- The rate limiter delegates the request to the API server if the throttling threshold is not exceeded

---

#### Throttling type and algorithms

| Throttling type  | Algorithms                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------- |
| Soft throttle    | <ul><li>fixed window</li><li>token bucket</li></ul>                                         |
| Hard throttle    | <ul><li>sliding window</li><li>sliding window with counter</li><li>leaking bucket</li></ul> |
| Dynamic throttle | all of the above with an additional system query to check for free resources                |

---

#### Workflow

<image src="imgs/API-rate-limiter.png" alt="API Rate Limiter" caption="API Rate Limiter" >

1. The client creates an HTTP connection to the web server
2. The web server forwards the request to the rate limiter service
3. The rate limiter service queries the rules cache to check the rate limit rule for the requested API endpoint
4. The read replicas of rules NoSQL data store are queried on a cache miss to identify the rate limit rule
5. The distributed lock such as Redis lock is used to handle concurrency when the same user makes multiple requests at the same time in a distributed system
6. The rate limiter service queries the throttle cache to verify if the throttle threshold is exceeded
7. The throttle cache uses a write-behind (write-back) cache pattern by storing the throttle data on the message queue to improve the latency
8. The throttle sync service executes a batch operation to store the throttle data on the message queue to the NoSQL document store
9. The NoSQL document stores persist the throttle data for fault tolerance, long-living rate limits, analytics, and auditing
10. The dropped requests (throttle threshold exceeded) are stored on the message queue for analytics and auditing
11. If the client request is not exceeding the throttle limit, the rate limiter delegates the client request to the API server
12. LRU cache eviction is used for cache servers
13. HTTP response headers indicate the relevant throttle limit data
14. [Consistent hashing](/consistent-hashing-explained/) is used to redirect the request from a user to the same subset of servers
