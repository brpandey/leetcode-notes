### Proximity Service

The popular implementations of the proximity service are the following:

- Tripadvisor
- Yelp
- Foursquare
- Google Maps

---

#### Requirements

- The client (**creator**) can add attractions (restaurants, markets)
- The client (**user**) can find all the nearby attractions within a given radius
- The users can view the relevant data and reviews of an attraction

---

#### Data storage

##### Database schema

<image src="imgs/proximity-service-database-schema.png" alt="Proximity service; Database schema" caption="Proximity service; Database schema" >

- The primary entities are the users, the media, the reviews, the locations, and the attractions tables
- The relationship between the users and the media table is 1-to-many
- The relationship between the users (creator) and attractions table is 1-to-many
- The relationship between the users and the reviews table is 1-to-many
- The relationship between the attractions and the reviews table is 1-to-many
- The relationship between the locations and the attractions table is 1-to-1

##### Type of data store

- SQL database such as Postgres persists the location data (latitude, longitude)
- The [PostGIS](https://postgis.net/) extension of Postgres is used to generate the [spatial index](https://en.wikipedia.org/wiki/Spatial_database) using [Geohash](https://en.wikipedia.org/wiki/Geohash) or [R-tree](https://en.wikipedia.org/wiki/R-tree)
- NoSQL data store such as MongoDB is used to store the metadata of an attraction
- An object store such as AWS S3 is used to store media files (images) of an attraction
- A cache server such as Redis is used to store the popular Geohashes and the locations
- A message queue such as Apache Kafka is used to orchestrate the asynchronous processing of the media files
- The Lucene-based inverted index data store such as Elasticsearch is used to store the search index to provide search functionality
- The [Trie](https://en.wikipedia.org/wiki/Trie) server is used to provide autocompletion functionality

---

#### High-level design

<image src="imgs/proximity-service-multi-datacenter.png" alt="Proximity service; Multi data center replication" caption="Proximity service; Multi data center replication" >

- The client (**creator**) updates the data stores (attraction and location) with the relevant metadata of an attraction by writing to the primary data center
- The DNS redirects the read requests from the client (**user**) to the secondary data centers to handle the read-heavy traffic
- The spatial index (Geohash) is used to identify the IDs of nearby attractions within a given radius

---

#### Write path

<image src="imgs/proximity-service-write-path.png" alt="Proximity service; Write path" caption="Proximity service; Write path" >

1. The client (**creator**) creates an HTTP connection to the load balancer
2. The write requests are rate limited for high availability
3. The load balancer delegates the client HTTP request to a server in the nearby primary data center
4. The latitude and longitude data of an attraction are added to the location database
5. The metadata (description) of an attraction is added to the attraction data store
6. The raw media files (images) are persisted on the object store
7. The message queue is used for the asynchronous processing of the raw media files
8. The index service fetches the Geohash and the ID of an attraction by querying the location database
9. The index service fetches the metadata of an attraction by querying the attraction data store
10. The index service stores the transformed metadata of an attraction on the inverted index store to provide the search functionality
11. The index service updates the trie server to provide autocompletion functionality
12. The spatial index is generated on the location database using Geohash
13. The Geohash (string value) is calculated by encoding the latitude and longitude values
14. Alternatively, the [Quadtree](https://en.wikipedia.org/wiki/Quadtree) data structure can be used to store the list of nearby attractions at the expense of an increased operational complexity
15. The standard relational database as the location database will perform poorly due to float values in database queries
16. The Geohash is performant because single index queries are faster and Geohash provides quicker proximity search
17. The stateful services are partitioned by geographical data for high availability
18. The compressed system logs are stored on the object store to save costs
19. The financially sensitive data must have a higher replication factor for durability

---

#### Read path

<image src="imgs/proximity-service-read-path.png" alt="Proximity service; Read path" caption="Proximity service; Read path" >

1. The client (**user**) queries the DNS for domain name resolution
2. The CDN is queried to check if the requested attraction is in the CDN cache
3. The client creates an HTTP connection to the load balancer
4. The read requests are rate limited to prevent malicious users
5. The load balancer delegates the client request to the server in the nearby secondary data center
6. The server queries the autocompletion service to provide relevant keyword suggestions
7. The server forwards the search query of the client to the search service
8. The search service queries the inverted index data store to find the relevant data records
9. The server delegates the client's request to find nearby attractions to the nearby service
10. The nearby service calculates the Geohash of the request and queries the Geohash cache to fetch the list of nearby attraction IDs
11. The location database (replica) is queried on a cache miss
12. The nearby service queries the attraction cache to fetch the metadata of an attraction
13. The attraction data store (replica) is queried on a cache miss
14. The relevant media files are fetched by querying the object store
15. The server queries the review service to fetch the relevant comments and ratings of an attraction
16. The review service queries the comment data store (replica) to fetch the comments
17. The client can cache the response with a TTL to improve the latency on further requests
18. LRU policy is used to evict the cache
19. The cache aside pattern is used to update the cache
20. Pagination should be used to fetch the response to improve the latency
21. The locations table is denormalized (create multi-character Geohash) for improved read performance
22. Service Discovery is implemented for the dynamic deployment of services
23. The [KNN](https://postgis.net/workshops/postgis-intro/knn.html) (K Nearest Neighbours) on PostGIS is used to fetch the list of nearby attractions in near [logarithmic time complexity](https://gis.stackexchange.com/questions/78974/what-is-the-complexity-of-neigborhood-search-in-postgis)
24. The euclidean formula can be used to distance between any attractions
25. The Geohash values with the same prefix are close to each other but not vice-versa
26. The attractions on neighboring Geohash grids are considered to resolve the boundary problem
