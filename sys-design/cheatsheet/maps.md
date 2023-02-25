### Google Maps

The popular implementations of the navigation service are the following:

- Google Maps
- OpenStreetMap
- Bing Maps

---

#### Requirements

- The user (**client**) can find the nearby points of interest within a given radius
- The client can find the real-time estimated time of arrival (**ETA**) between two locations
- The client receive suggestions on the possible routes between the two locations

---

#### Data storage

##### Database schema

<image src="imgs/google-maps-database-schema.png" alt="Google Maps; Database schema" caption="Google Maps; Database schema" >

- The primary entities are the locations, the places, and the media tables
- The relationship between the locations and the places table is 1-to-many
- The relationship between the places and the media table is 1-to-many
- The [spatial index](https://en.wikipedia.org/wiki/Spatial_database) using [Geohash](https://en.wikipedia.org/wiki/Geohash) or [R-tree](https://en.wikipedia.org/wiki/R-tree) is created on the locations table

---

##### Type of data store

- SQL database such as Postgres persists the location data (latitude, longitude) and the [vector tiles](https://en.wikipedia.org/wiki/Vector_tiles)
- The [PostGIS](https://postgis.net/) extension of Postgres is used to generate the spatial index
- The wide-column data store ([LSM](https://en.wikipedia.org/wiki/Log-structured_merge-tree) tree-based) such as Apache Cassandra is used to persist the trip data for high write throughput
- A cache server such as Redis is used to store the popular routes and the road segments
- A message queue such as Apache Kafka is used to handle heavy traffic
- The message queue is also configured as the pub-sub server
- NoSQL document store such as MongoDB is used to store the metadata of the road segments
- The media files and raster tiles are stored in a managed object storage such as AWS S3
- The CDN caches the popular tiles and the media files
- Lucene-based inverted-index data store such as Apache Solr is used to store the index data of locations

---

#### High-level design

<image src="imgs/navigation-service.png" alt="Navigation service" caption="Navigation service" >

- Calculation of ETA is the shortest path in a directed weighted graph problem
- The graph is partitioned and the routes are precomputed for scalability

<image src="imgs/google-maps-map-matching.png" alt="Google Maps; Map matching" caption="Google Maps; Map matching" >

- The map matching is performed to improve the accuracy of ETA
- The [Kalman filter](https://en.wikipedia.org/wiki/Kalman_filter) is used for map matching
- The cache and data stores are replicated and sharded geographically through [consistent hashing](https://systemdesign.one/consistent-hashing-explained/) to handle hotspots
- Monitoring and fallback can be configured for high availability

---

#### Points of Interest

<image src="imgs/google-maps-points-of-interest.png" alt="Google Maps; Points of interest" caption="Google Maps; Points of interest" >

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the nearby points of interest are cached on the CDN
3. The client creates a [WebSocket](https://en.wikipedia.org/wiki/WebSocket) connection on the load balancer for real-time updates
4. The load balancer delegates the client's connection to a server with free capacity
5. The server queries the trie server to provide autocompletion functionality
6. The server delegates the search query of the client to the search service
7. The search service queries the inverted index data store to find the relevant locations
8. The server delegates the points of interest query from the client to the location service
9. The location service calculates the Geohash from the request and queries the location cache to fetch the list of nearby points of interest IDs
10. The location database (replica) is queried on a cache miss
11. The location service queries the places cache to fetch the metadata of a place
12. The places data store (replica) is queried on a cache miss
13. The location service queries the object store to fetch the relevant media files and the raster tiles
14. The tile server on the location service provides SQL functionality on vector tiles
15. LRU policy is used to evict the cache
16. GPS signals are not accurate for the identification of the static location due to the multipath effect (reflection from a building)
17. The WiFi access point is used in addition to GPS signals to find the static location of a client accurately
18. The map is displayed on a web browser by placing multiple smaller vector tiles and raster tiles next to each other to make up a single bigger picture
19. The raster tiles (png, jpg) are useful for off-road trajectories to speed up the loading of detected map errors
20. Vector tiles allow the reduction of bandwidth by restricting data transfer to the current viewport and zoom level
21. Vector tiles allow to apply styling on the fly on the client and thereby reduce the server load
22. Vector tiles transfer the vector information (path and polygon) instead of transferring the image tiles
23. Vector information is grouped into tiles for caching and partitioning
24. The tiles can be pre-generated and cached to improve latency
25. Each zoom level shows a unique set of tiles with a different granularity of information
26. The number of tiles doubles when the user zooms out
27. Geohash is used to identify the tile on the map
28. The viewport of the client encloses the container that assigns absolute positions to the tiles
29. The container is moved when the user pans the map
30. Street view and satellite imagery are used to identify places in the world

---

#### Estimated Time of Arrival

<image src="imgs/google-maps-estimated-time-of-arrival.png" alt="Google Maps; Estimated Time of Arrival" caption="Google Maps; Estimated Time of Arrival" >

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the requested route is cached on the CDN
3. The client creates a WebSocket connection on the load balancer for real-time bi-directional communication on ETA, trip, and traffic data
4. The load balancer delegates the client's request to a server with free capacity
5. The server queries the route cache to fetch the list of possible routes to the destination
6. The route database (replica) is queried on a cache miss
7. The server queries the road segment cache to fetch the metadata of the road segments on the suggested route
8. The road segment data store (replica) is queried on a cache miss
9. The server queries the trip data store to recalculate the real-time ETA
10. The server queries the map-matching service to calculate the ETA accurately
11. The map-matching service queries the trip data store to fetch the list of recent GPS signals
12. The map matching service queries the road segment data store to identify the nearest road segments
13. The weather service publishes the latest weather data on the pub-sub server
14. The traffic service ([HyperLogLog](https://en.wikipedia.org/wiki/HyperLogLog) based) publishes the latest traffic data on the pub-sub server
15. The server fetches the latest weather and traffic data using the publish-subscribe pattern
16. The server queries the route ranking service to filter and sort the result
17. The server publishes the GPS signals on the message queue for asynchronous processing
18. Apache Spark executes jobs on the trip data to detect changes in roads, hot spots, and traffic patterns
19. Apache Spark job asynchronously updates the route database with any detected changes
20. The physical map is represented as a graph with road segments as the directed edges and road intersections as the nodes
21. The route between the origin and destination on the road network should be calculated in the least cost (function of time and distance)
22. Map matching is the problem of matching the raw GPS signals to the relevant road segments
23. The embedded sensors on the smartphones are used to geo-localize the users
24. The graph can be partitioned to pre-compute the possible routes within the partitions with time complexity of sqrt(n), where n = number of road intersections
25. The partitions can be created on different granularities (country, city) to improve the latency of long-distance queries
26. The interactions are applied on the boundaries of the partitions to improve the accuracy and scalability
27. [Dijkstra's](https://en.wikipedia.org/wiki/Dijkstra's_algorithm) algorithm (sub-optimal due to scalability) provides a time complexity of O(nlogn) for route calculation, where n = number of road intersections
28. The shortest path algorithm on the weighted graph such as the [Bellman-Ford](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm) provides a time complexity of O(ve) for route calculation, where v = number of vertices and e = number of edges
29. The edge weight of the graph is populated with weather and traffic data
30. The popular partitions and routes can be cached to improve the latency
31. Alternate routes can be suggested if applying the weather and traffic data on the shortest distanced route returns a delayed ETA
32. The time series data (seasonality), map data, routing, and machine learning can be used to predict the ETA
33. [Viterbi algorithm](https://en.wikipedia.org/wiki/Viterbi_algorithm) (dynamic programming) can be used to identify the most probable sequence of road segments using a sequence of GPS signals
34. Alternatively, the [KNN algorithm](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) with a minimum radius (k = 2) can be executed on the Geohash to identify the closest road segments
35. The marginalized particle filter running on an unscented Kalman filter is used for real-time map matching
36. Kalman filter is an optimal algorithm that uses multiple GPS signals to estimate the accurate road segment of the client
37. Kalman filter is also applied to the seasonality of the time series data
38. Additional data types such as traffic and weather can be added to marginalized particle filter
39. Machine learning on the time series historical data helps to predict traffic conditions
