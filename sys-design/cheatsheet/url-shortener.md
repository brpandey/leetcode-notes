# URL Shortening System Design


## How does a URL shortener work?

At a high level, the URL shortener executes the following operations:

1. the server generates a unique short URL for each long URL
2. the server encodes the short URL for readability
3. the server persists the short URL in the data store
4. the server redirects the client to the original long URL against the short URL

---

---

---

## Terminology

The following terminology might be useful for you:

- Microservices: designing software that is made up of small independent services, which have a specific purpose
- Service Discovery: the process of automatically detecting devices and services on a network
- CDN: a group of geographically distributed servers that speed up the delivery of web content by bringing the content closer to the users
- API: a software intermediary that allows two applications or services to talk to each other
- Encoding: the process of converting data from one form to another to preserve the usability of data
- Encryption: secure encoding of data using a key to protect the confidentiality of data
- Hashing: a one-way summary of data that cannot be reversed and is used to validate the integrity of data
- Bloom filter: a memory-efficient probabilistic data structure to check whether an element is present in a set

---

---

---

## What is a URL shortening service?

A URL shortening service is a website that substantially shortens a Uniform Resource Locator (URL). The short URL redirects the client to the URL of the original website. Some popular public-facing URL shortening services are tinyurl.com and bitly.com [1].

<image src="imgs/what-is-url-shortening-service.png" alt="What is a URL shortening service?" caption="What is a URL shortening service?" >

The reasons to shorten a URL are the following:

- track clicks for analytics
- beautify a URL
- disguise the underlying URL for affiliates
- some instant messaging services limit the count of characters on the URL

---

---

---

## Further system design learning resources

Books are a powerful medium to gather new knowledge. Check out the following books to set yourself up for success in the system design interview:

- [Designing Data-Intensive Applications](https://amzn.to/40tlRDt) by Martin Kleppmann
- [System Design Interview — An insider’s guide Volume 1](https://amzn.to/3G0MN5o) by Alex Xu
- [System Design Interview — An insider’s guide Volume 2](https://amzn.to/3BKMngO) by Alex Xu
- [Hacking the System Design Interview](https://amzn.to/3FGPjft) by Stanley Chiang

---

<a href="https://forms.gle/1bDAVExKKL1CRVSS7" target="_blank">
  <img src="https://i.imgur.com/3YLVY0k.png"/>
</a>

---

---

---

## Questions to ask the Interviewer

### Candidate

1. What are the use cases of the system?
2. What is the amount of Daily Active Users (DAU) for writes?
3. How many years should we persist the short URL by default?
4. What is the anticipated read: write ratio of the system?
5. What is the usage pattern of the shortened URL?
6. Who will use the URL shortener service?
7. What is the reasonable length of a short URL?

---

### Interviewer

1. URL shortening and redirection to the original long URL
2. 100 million DAU
3. 5 years
4. 100: 1
5. Most of the shortened URLs will be accessed only once after the creation
6. General public
7. At most 9 characters

---

---

---

## Requirements

### Functional Requirements

- URL shortening Service similar to [TinyURL](https://tinyurl.com/), or [Bitly](https://bitly.com)
- A **client** (user) enters a long URL into the system and the system returns a shortened URL
- The client visiting the short URL must be redirected to the original long URL
- Multiple users entering the same long URL must receive the same short URL (1-to-1 mapping)
- The short URL should be readable
- The short URL should be collision-free
- The short URL should be non-predictable
- The client should be able to choose a custom short URL
- The short URL should be web-crawler friendly ([SEO](https://en.wikipedia.org/wiki/Search_engine_optimization))
- he short URL should support analytics (not real-time) such as the number of redirections from the shortened URL
- The client optionally defines the expiry time of the short URL

---

### Non-Functional Requirements

- High Availability
- Low Latency
- High Scalability
- Durable
- Fault Tolerant

---

### Out of Scope

- The user registers an account
- The user sets the visibility of the short URL

---

---

---

## High-Level Design

### Encoding

Encoding is the process of converting data from one form to another. The following are the reasons to encode a shortened URL:

- improve the readability of the short URL
- make short URLs less error-prone

The encoding format to be used in the URL shortener must yield a deterministic (no randomness) output. The potential data encoding formats that satisfy the URL shortening use case are the following:

<image src="imgs/url-shortener-encoding-formats.png" alt="URL shortener; Encoding formats" caption="URL shortener; Encoding formats" >

The base58 encoding format is similar to base62 encoding except that base58 avoids non-distinguishable characters such as O (uppercase O), 0 (zero), and I (capital I), l (lowercase L). The characters in base62 encoding consume 6 bits (2⁶ = 64). A short URL of 7 characters in length in base62 encoding consumes 42 bits.

The following generic formula is used to count the total number of short URLs that are produced using a specific encoding format and the number of characters in the output:


**Total count of short URLs = branching factor ^ depth**

where the branching factor is the base of the encoding format and depth is the length of characters.

The combination of encoding formats and the output length generates the following total count of short URLs:

<image src="imgs/total-count-short-url.png" alt="Total count of short URLs" caption="Total count of short URLs" >

The total count of short URLs is directly proportional to the length of the encoded output. However, the length of the short URL must be kept as short as possible for better readability. The base62 encoded output of 7-character length generates 3.5 trillion short URLs. A total count of 3.5 trillion short URLs is exhausted in 100 years when 1000 short URLs are used per second. The guidelines on the encoded output format to improve the readability of a short URL are the following:

- the encoded output contains only alphanumeric characters
- the length of the short URL must not exceed 9 characters

The **time complexity** of base conversion is O(k), where k is the number of characters (k = 7). The time complexity of base conversion is reduced to constant time O(1) because the number of characters is fixed [7].

In summary, a 7-character base62 encoded output satisfies the system requirement.

---

---

### Write path

The server shortens the long URL entered by the client. The shortened URL is encoded for improved readability. The server persists the encoded short URL into the database. The simplified block diagram of a single-machine URL shortener is the following:

<image src="imgs/simplified-url-shortener.png" alt="Simplified URL shortener" caption="Simplified URL shortener" >

The single-machine solution does not meet the scalability requirements of the URL shortener system. The key generation function is moved out of the server to a dedicated Key Generation Service (**KGS**) to [scale out](https://en.wikipedia.org/wiki/Scalability) the system.

<image src="imgs/url-shortener-key-generation-service.png" alt="URL shortener; Key Generation Service" caption="URL shortener; Key Generation Service" >

The different solutions to shortening a URL are the following:

- Random ID Generator
- Hashing Function
- Token Range

---

#### Random ID Generator solution

The Key Generation Service (**KGS**) queries the random identifier (**ID**) generation service to shorten a URL. The service generates random IDs using a random function or Universally Unique Identifiers ([UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)). Multiple instances of the random ID generation service must be provisioned to meet the demand for scalability.

<image src="imgs/url-shortener-random-id-generator.png" alt="URL shortener; Random ID Generator" caption="URL shortener; Random ID Generator" >

The random ID generation service must be stateless to easily replicate the service for scaling. The ingress is distributed to a random ID generation service using a [load balancer](<https://en.wikipedia.org/wiki/Load_balancing_(computing)>). The potential load-balancing algorithms to route the traffic are the following:

- round-robin
- least connection
- least bandwidth
- [modulo-hash](https://www.cs.cornell.edu/courses/cs312/2008sp/lectures/lec21.html#:~:text=With%20modular%20hashing,%20the%20hash,lowest-order%20bits%20of%20k.) function
- [consistent hashing](https://systemdesign.one/consistent-hashing-explained/)

The consistent hashing or the modulo-hash function-based load balancing algorithms might result in unbalanced (**hot**) replicas when the same long URL is entered by a large number of clients at the same time. The KGS must verify if the generated short URL already exists in the database because of the randomness in the output.

The random ID generation solution has the following tradeoffs:

- the probability of collisions is high due to randomness
- breaks the 1-to-1 mapping between a short URL and a long URL
- coordination between servers is required to prevent a collision
- frequent verification of the existence of a short URL in the database is a bottleneck

An alternative to the random ID generation solution is using Twitter’s Snowflake [8]. The length of the snowflake output is 64 bits. The base62 encoding of snowflake output yields an 11-character output because each base62 encoded character consumes 6 bits. The snowflake ID is generated by a combination of the following entities (real-world implementation might vary):

- Timestamp
- [Datacenter](https://en.wikipedia.org/wiki/Data_center) ID
- Worker node ID
- Sequence number

<image src="imgs/twitter-snowflake-id.png" alt="Twitter Snowflake ID" caption="Twitter Snowflake ID" >

The downsides of using snowflake ID for URL shortening are the following:

- probability of collision is higher due to the overlapping bits
- generated short URL becomes predictable due to known bits
- increases the complexity of the system due to time synchronization between servers

In summary, do not use the random ID generator solution for shortening a URL.

---

#### Hashing Function solution

The KGS queries the [hashing function](https://en.wikipedia.org/wiki/Hash_function) service to shorten a URL. The hashing function service accepts a long URL as an input and executes a hash function such as the message-digest algorithm ([MD5](https://en.wikipedia.org/wiki/MD5)) to generate a short URL [9]. The length of the MD5 hash function output is 128 bits. The hashing function service is replicated to meet the scalability demand of the system.

<image src="imgs/url-shortener-hashing-long-url.png" alt="URL shortener; Hashing the long URL" caption="URL shortener; Hashing the long URL" >

The hashing function service must be stateless to easily replicate the service for scaling. The ingress is distributed to the hashing function service using a load balancer. The potential load-balancing algorithms to route the traffic are the following:

- weighted round-robin
- least response time
- IP address hash
- modulo-hash function
- [consistent hashing](https://systemdesign.one/consistent-hashing-explained/)

The hash-based load balancing algorithms result in hot replicas when the same long URL is entered by a large number of clients at the same time. The non-hash-based load-balancing algorithms result in redundant operations because the MD5 hashing function produces the same output (short URL) for the same input (long URL).

<image src="imgs/url-shortener-hashing-function-service.png" alt="URL shortener; Hashing function service" caption="URL shortener; Hashing function service" >

The base62 encoding of MD5 output yields 22 characters because each base62 encoded character consumes 6 bits and MD5 output is 128 bits. The encoded output must be truncated by considering only the first 7 characters (42 bits) to keep the short URL readable. However, the encoded output of multiple long URLs might yield the same prefix (first 7 characters), resulting in a collision. Random bits are appended to the suffix of the encoded output to make it nonpredictable at the expense of short URL readability.

<image src="imgs/url-shortener-truncate-encoded-md5.png" alt="URL shortener; Truncating the encoded MD5 output" caption="URL shortener; Truncating the encoded MD5 output" >

n alternative hashing function for URL shortening is [SHA256](https://en.wikipedia.org/wiki/SHA-2). However, the probability of a collision is higher due to an output length of 256 bits. The tradeoffs of the hashing function solution are the following:

- predictable output due to the hash function
- higher probability of a collision

In summary, do not use the hashing function solution for shortening a URL.

---

#### Token Range solution

The KGS queries the token service to shorten a URL. An internal counter function of the token service generates the short URL and the output is monotonically increasing.

<image src="imgs/url-shortener-token-service-consistent-hash-ring.png" alt="URL shortener; Token service in consistent hash ring" caption="URL shortener; Token service in consistent hash ring" >

The token service must be horizontally partitioned ([shard](<https://en.wikipedia.org/wiki/Shard_(database_architecture)>)) to meet the scalability requirements of the system. The potential sharding schemes for the token service are the following:

- [list partitioning](<https://en.wikipedia.org/wiki/Partition_(database)>)
- [modulus partitioning](https://www.ibm.com/docs/en/iis/8.5?topic=partitioning-modulus-partitioner)
- [consistent hashing](https://systemdesign.one/consistent-hashing-explained/)

The list and modulus partitioning schemes do not meet the scalability requirements of the system because both schemes limit the number of token service instances. The sharding based on consistent hashing fits the system requirements as the token service scales out by provisioning new instances.

The ingress is distributed to the token service using a load balancer. The [percent-encoded](https://en.wikipedia.org/wiki/Percent-encoding) long URLs are load-balanced using consistent hashing to preserve the 1-to-1 mapping between the long and short URLs. However, a load balancer based on consistent hashing might result in hot shards when the same long URL is entered by a large number of clients at the same time.

The output of the token service instances must be non-overlapping to prevent a collision. A highly reliable distributed service such as [Apache Zookeeper](https://zookeeper.apache.org/) or [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) is used to coordinate the output range of token service instances. The service that coordinates the output range between token service instances is named the **token range service**.

<image src="imgs/url-shortener-token-range-service.png" alt="URL shortener; Token range service" caption="URL shortener; Token range service" >

When the key-value store is chosen as the token range service, the [quorum](<https://en.wikipedia.org/wiki/Quorum_(distributed_computing)#:~:text=A%20quorum%20is%20the%20minimum,operation%20in%20a%20distributed%20system.>) must be set to a higher value to increase the consistency of the token range service. The stronger consistency prevents a range collision by preventing fetching the same output range by multiple token services.

When an instance of the token service is provisioned, the fresh instance executes a request for an output range from the token range service. When the fetched output range is fully exhausted, the token service requests a fresh output range from the token range service.

<image src="imgs/token-range-service.png" alt="URL shortener; Token range service" caption="URL shortener; Token range service" >

The token range service might become a [bottleneck](<https://en.wikipedia.org/wiki/Bottleneck_(software)>) if queried frequently. Either the output range or the number of token range service replicas must be incremented to improve the reliability of the system. The token range solution is collision-free and scalable. However, the short URL is predictable due to the monotonically increasing output range. The following actions degrade the predictability of the shortened URL:

- append random bits to the suffix of the output
- token range service gives a randomized output range

The time complexity of short URL generation using token service is constant O(1). In contrast, the KGS must perform one of the following operations before shortening a URL to preserve the 1-to-1 mapping:

- query the database to check the existence of the long URL
- use the putIfAbsent procedure to check the existence of the long URL

Querying the database is an expensive operation because of the disk input/output (**I/O**) and most of the NoSQL data stores do not support the putIfAbsent procedure due to eventual consistency.

<image src="imgs/url-shortener-bloom-filter.png" alt="URL shortener; Bloom filter" caption="URL shortener; Bloom filter" >

A bloom filter is used to prevent expensive data store lookups on URL shortening. The time complexity of a bloom filter query is constant O(1) [10]. The KGS populates the bloom filter with the long URL after shortening the long URL. When the client enters a customized short URL, the KGS queries the bloom filter to check if the long URL exists before persisting the custom short URL into the data store. However, the bloom filter query might yield false positives, resulting in a database lookup. In addition, the bloom filter increases the operational complexity of the system.

When the client enters an already existing long URL, the KGS must return the appropriate short URL but the database is partitioned with the short URL as the partition key. The short URL as the partition key resonates with the read and write paths of the URL shortener.

<image src="imgs/url-shortener-inverted-index-store.png" alt="URL shortener; Inverted index data store (Long URL: Short URL)" caption="URL shortener; Inverted index data store (Long URL: Short URL)" >

A naive solution to finding the short URL is to build an [index](https://en.wikipedia.org/wiki/Database_index) on the long URL column of the data store. However, the introduction of a database index degrades the write performance and querying remains complex due to sharding using the short URL key.

The optimal solution is to introduce an additional data store (inverted index) with mapping from the long URLs to the short URLs (key-value schema). The additional data store improves the time complexity of finding the short URL of an already existing long URL record. On the other hand, an additional data store increases storage costs. The additional data store is partitioned using consistent hashing. The partition key is the long URL to quickly find the URL record. A key-value store such as DynamoDB is used as the additional data store.

<image src="imgs/url-shortener-scaling-token-service.png" alt="URL shortener; Scaling the token service" caption="URL shortener; Scaling the token service" >

The token service stores some short URLs (keys) in memory so that the token service quickly provides the keys to an incoming request. The keys in the token service must be distributed by an atomic data structure to handle concurrent requests. The output range stored in token service memory is marked as used to prevent a collision. The downside of storing keys in memory is losing the specific output range of keys on a server failure. The output range must be moved out to an external cache server to scale out the token service and improve its reliability.

<image src="imgs/token-server.png" alt="URL shortener; Token server" caption="URL shortener; Token server" >

The output of the token service must be encoded within the token server using an encoding service to prevent external network communication. An additional function executes the encoding of token service output.

In summary, use the token range solution for shortening a URL.

---

---

### Read path

The server redirects the shortened URL to the original long URL. The simplified block diagram of a single-machine URL redirection is the following:

<image src="imgs/simplified-url-shortener.png" alt="Simplified URL redirection" caption="Simplified URL redirection" >

The single-machine solution does not meet the scalability requirements of URL redirection for a read-heavy system. The disk I/O due to frequent database access is a potential bottleneck.

The URL redirection traffic (**Egress**) is cached following the [80/20 rule](https://en.wikipedia.org/wiki/Pareto_principle) to improve latency. The cache stores the mapping between the short URLs and the long URLs. The cache handles uneven traffic and traffic spikes in URL redirection. The server must query the cache before hitting the data store. The [cache-aside pattern](https://github.com/donnemartin/system-design-primer#cache-aside) is used to update the cache. When a cache miss occurs, the server queries the data store and populates the cache. The tradeoff of using the cache-aside pattern is the delay in initial requests. As the data stored in the cache is memory bound, the Least Recently Used ([LRU](<https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)>)) policy is used to evict the cache when the cache server is full.

<image src="imgs/url-redirection-caching-different-layers.png" alt="URL redirection; Caching at different layers" caption="URL redirection; Caching at different layers" >

The cache is introduced at the following layers of the system for scalability:

- client
- content delivery network ([CDN](https://en.wikipedia.org/wiki/Content_delivery_network))
- reverse proxy
- dedicated cache servers

A shared cache such as CDN or a dedicated cache server reduces the load on the system. On the other hand, the private cache is only accessible by the client and does not significantly improve the system’s performance. On top of that, the definition of TTL for the private cache is crucial because private cache invalidation is difficult. Dedicated cache servers such as [Redis](https://redis.io/) or [Memcached](https://memcached.org/) are provisioned between the following system components to further improve latency:

- server and data store
- load balancer and server

The typical usage pattern of a URL shortener by the client is to shorten a URL and access the short URL only once. The cache update on a single access usage pattern results in cache [thrashing](<https://en.wikipedia.org/wiki/Thrashing_(computer_science)>). A bloom filter on short URLs is introduced on cache servers and CDN to prevent cache thrashing. The bloom filter is updated on the initial access to a short URL. The cache servers are updated only when the bloom filter is already set (multiple requests to the same short URL).

<image src="imgs/url-redirection-cache-update.png" alt="URL redirection; Cache update" caption="URL redirection; Cache update" >

The cache and the data store must not be queried if the short URL does not exist. A bloom filter on the short URL is introduced to prevent unnecessary queries. If the short URL is absent in the bloom filter, return an HTTP status code of 404. If the short URL is set in the bloom filter, delegate the redirection request to the cache server or the data store.

The cache servers are scaled out by performing the following operations:

- partition the cache servers (use the short URL as the partition key)
- replicate the cache servers to handle heavy loads using [leader-follower topology](https://redis.io/docs/management/replication/)
- redirect the write operations to the leader
- redirect all the read operations to the follower replicas

<image src="imgs/url-redirection-scaling-cache-servers.png" alt="URL redirection; Scaling cache servers" caption="URL redirection; Scaling cache servers" >

When multiple identical requests arrive at the cache server at the same time, the cache server will [collapse the requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#request_collapse) and will forward a single request to the origin server on behalf of the clients. The response is reused among all the clients to save bandwidth and system resources.

<image src="imgs/cache-server-collapse-forwarding.png" alt="URL redirection; Cache server collapsed forwarding" caption="URL redirection; Cache server collapsed forwarding" >

The following intermediate components are introduced to satisfy the scalability demand for URL redirection:

- Domain Name System ([DNS](https://en.wikipedia.org/wiki/Domain_Name_System))
- Load balancer
- Reverse proxy
- CDN
- Controller service to automatically scale the services

<image src="imgs/url-redirection-workflow.png" alt="URL redirection workflow" caption="URL redirection workflow" >

The following smart [DNS services](https://github.com/donnemartin/system-design-primer#domain-name-system) improve URL redirection:

- weighted round-robin
- latency based
- geolocation based

The reverse proxy is used as an [API Gateway](https://aws.amazon.com/api-gateway/). The reverse proxy executes [SSL termination](https://en.wikipedia.org/wiki/TLS_termination_proxy) and compression at the expense of increased system complexity. When an extremely popular short URL is accessed by thousands of clients at the same time, the reverse proxy [collapse forwards](https://wiki.squid-cache.org/Features/CollapsedForwarding) the requests to reduce the system load. The load balancer must be introduced between the following system components to route traffic between the replicas or shards:

- client and server
- server and database
- server and cache server

The CDN serves the content from locations closer to the client at the expense of increased financial costs. The [Pull CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network) approach fits the URL redirection requirements. A dedicated controller service is provisioned to automatically scale out or scale down the system components based on the system load.

The [microservices](https://en.wikipedia.org/wiki/Microservices) architecture improves the fault tolerance of the system. The services such as [Etcd](https://etcd.io/) or Zookeeper help services find each other (known as service discovery). In addition, the Zookeeper is configured to monitor the health of the services in the system by sending regular heartbeat signals. The downside of microservices architecture is the increased operational complexity.



## Summary

<image src="imgs/shortened-url.png" alt="Shortened URL" caption="Shortened URL" >

The URL shortening service is a popular system design interview question. Although the use cases of a URL shortener seem trivial, building an internet-scale URL shortener is a challenging task.
T
