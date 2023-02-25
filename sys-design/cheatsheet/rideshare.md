### Ride-Hailing Service

The popular implementations of the ride-hailing service are the following:

- Uber
- Lyft
- Curb
- Grab

---

#### Requirements

- The rider can see all the available nearby drivers
- The driver can accept a trip requested by the rider
- The current location of the rider and driver should be continuously published on the trip confirmation

---

#### Data storage

##### Database schema

<image src="imgs/ride-hailing-service-database-schema.png" alt="Ride-hailing service; Database schema" caption="Ride-hailing service; Database schema" >

- The primary entities are the riders, the drivers, the vehicles, and the trips tables
- The relationship between the drivers and the vehicles table is 1-to-many
- The relationship between the drivers and trips table is 1-to-many
- The relationship between the riders and trips table is 1-to-many
- The trips table is a join table to represent the relationship between the riders and the drivers

---

##### Type of data store

- The wide-column data store ([LSM](https://en.wikipedia.org/wiki/Log-structured_merge-tree) tree-based) such as Apache Cassandra is used to persist the time-series location data of the client (driver and rider)
- The cache server such as Redis is used to store the current location of the driver and the rider for quick lookups
- Message queue such as Apache Kafka is used to handle the heavy traffic
- A relational database such as Postgres stores the metadata of the users

---

#### High-level design

- The DNS redirects the requests from the client (rider and driver) to nearby data centers
- The client (rider and driver) updates the data stores with Geohash of their real-time location
- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) is used for real-time bidirectional communication between the rider and the driver
- [Consistent hashing](https://systemdesign.one/consistent-hashing-explained/) is used to partition the data stores geographically

---

#### Write path

<image src="imgs/ride-hailing-service-write-path.png" alt="Ride-hailing service; Write path" caption="Ride-hailing service; Write path" >

1. The client (driver) creates a WebSocket connection on the load balancer to publish the current location (latitude, longitude) of the driver in real-time
2. The load balancer uses the round-robin algorithm to delegate the client's connection to a server with free capacity in the nearby data center
3. The Geohash of the driver location is persisted on the message queue to handle the heavy traffic
4. The Geohash of the driver location is stored on the wide-column data store for durability
5. The Geohash is stored on the point location cache to provide real-time location updates
6. The client (rider) creates a WebSocket connection on the load balancer to publish the current location (latitude, longitude) of the rider in real-time
7. The load balancer uses the round-robin algorithm to delegate the client's connection to a server with free capacity in the nearby data center
8. The Geohash of the rider location is persisted on the message queue to handle the heavy traffic
9. The analytics service (MapReduce based) queries the wide-column data store to generate offline analytics on the trip data
10. The controller service prevents hotspots by auto-repartitioning the stateful services
11. The point location cache is denormalized by the generation of multi-character Geohash to improve the read performance (provides zoom functionality)
12. The server holding the rider's WebSocket connection queries the point location cache to identify the available nearby drivers
13. As a naive approach, the [euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) can be used to find the nearest vehicles within a Geohash
14. Sharding of the services can be implemented on multiple levels such as the city level, geo sharding for further granularity, and the product level (capacity of the vehicle)
15. The hotspots are handled through replication and further partitioning of the stateful services by the driver ID
16. The wide-column data store is optimized for writes while the cache server is optimized for reads
17. The wide-column data store is replicated across multiple data centers for durability
18. The LRU policy is used to evict the cache server

---

#### Read path

<image src="imgs/ride-hailing-service-read-path.png" alt="Ride-hailing service; Driver accepting a trip request" caption="Ride-hailing service; Driver accepting a trip request" >

1. The client (driver) creates a WebSocket connection on the load balancer to receive updates on trip requests in real-time
2. The load balancer uses the round-robin algorithm to delegate the client's connection to a server with free capacity in the nearby data center
3. The server holding the driver's WebSocket connection must acquire a distributed lock to handle concurrency issues when accepting trip requests from concurrent unique riders
4. The server holding the driver's WebSocket connection invokes the trip service to confirm the trip
5. The trip service queries the pub-sub server to create a one-to-one communication channel between the driver and the rider
6. The server publishes the location updates by the driver on the pub-sub server
7. The pub-sub server persists the driver's location data on the trip data store for durability
8. The pub-sub server delegates the location updates by the driver to the server that is holding the rider's WebSocket connection using the publish-subscribe pattern
9. The server delegates the driver's location update to the load balancer that holds the rider's WebSocket connection
10. The driver's location updates are published to the rider
11. The state of the trip is cached on the client (rider and driver) for a fallback to another data center
12. Chaos Engineering can be used for resiliency testing
13. Services gossip protocol the state for high availability
