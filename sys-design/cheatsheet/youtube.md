### YouTube

The popular implementations of an on-demand video streaming service are the following:

- YouTube
- Netflix
- Vimeo
- TikTok

---

#### Requirements

- The user (**client**) can upload video files
- The user can stream video content
- The user can search for videos based on the video title

---

#### Data storage

##### Database schema

<image src="imgs/youtube-database-schema.png" alt="YouTube; Database schema" caption="YouTube; Database schema" >

- The primary entities are the videos, the users, and the comments tables
- The relationship between the users and the videos is 1-to-many
- The relationship between the users and the comments table is 1-to-many
- The relationship between the videos and the comments table is 1-to-many

---

##### Type of data store

- The wide-column data store ([LSM](https://en.wikipedia.org/wiki/Log-structured_merge-tree) tree-based) such as [Apache HBase](https://hbase.apache.org/) is used to persist thumbnail images for clumping the files together, fault-tolerance, and replication
- A cache server such as Redis is used to store the metadata of popular video content
- Message queue such as Apache Kafka is used for the asynchronous processing (encoding) of videos
- A relational database such as MySQL stores the metadata of the users and the videos
- The video files are stored in a managed object storage such as AWS S3
- Lucene-based inverted-index data store such as Apache Solr is used to persist the video index data to provide search functionality

---

#### High-level design

- Popular video content is streamed from CDN
- Video encoding (**transcoding**) is the process of converting a video format to other formats (MPEG, HLS) to provide the best stream possible on multiple devices and bandwidth
- A message queue can be configured between services for parallelism and improved fault tolerance
  Codecs (H.264, VP9, HEVC) are compression and decompression algorithms used to reduce video file size while preserving video quality
- The popular video streaming protocols (data transfer standard) are **MPEG-DASH** (Moving Pictures Experts Group - Dynamic Adaptive Streaming over HTTP), **Apple HLS** (HTTP Live Streaming), **Microsoft Smooth Streaming**, and **Adobe HDS** (HTTP Dynamic Streaming)

---

#### Video upload workflow

<image src="imgs/youtube-upload-path.png" alt="YouTube; Video upload workflow" caption="YouTube; Video upload workflow" >

1. The user (**client**) executes a DNS query to identify the server
2. The client makes an HTTP connection to the load balancer
3. The video upload requests are rate limited to prevent malicious clients
4. The load balancer delegates the client's request to an API server (**web server**) with free capacity
5. The web server delegates the client's request to an app server that handles the API endpoint
6. The ID of the uploaded video is stored on the message queue for asynchronous processing of the video file
7. The title and description (**metadata**) of the video are stored in the metadata database
8. The app server queries the object store service to generate a pre-signed URL for storing the raw video file
9. The client uploads the raw video file directly to the object store using the pre-signed URL to save the system network bandwidth
10. The transcoding servers query the message queue using the publish-subscribe pattern to get notified on uploaded videos
11. The transcoding server fetches the raw video file by querying the raw object store
12. The transcoding server transcodes the raw video file into multiple codecs and stores the transcoded content on the transcoded object store
13. The thumbnail server generates on average five thumbnail images for each video file and stores the generated images on the thumbnail store
14. The transcoding server persists the ID of the transcoded video on the message queue for further processing
15. The upload handler service queries the message queue through the publish-subscribe pattern to get notified on transcoded video files
16. The upload handler service updates the metadata database with metadata of transcoded video files
17. The upload handler service queries the notification service to notify the client of the video processing status
18. The database can be partitioned through [consistent hashing](https://systemdesign.one/consistent-hashing-explained/) (key = user ID or video ID)
19. [Block matching](https://en.wikipedia.org/wiki/Block-matching_algorithm) or [Phase correlation](https://en.wikipedia.org/wiki/Phase_correlation) algorithms can be used to detect the duplicate video content
20. The web server (API server) must be kept stateless for scaling out through replication
21. The video file is stored in multiple resolutions and formats in order to support multiple devices and bandwidth
22. The video can be split into smaller chunks by the client before upload to support the resume of broken uploads
23. Watermarking and encryption can be used to protect video content
24. The data centers are added to improve latency and data recovery at the expense of increased maintenance workflows
25. Dead letter queue can be used to improve fault tolerance and error handling
26. Chaos engineering is used to identify the failures in networks, servers, and applications
27. Load testing and chaos engineering are used to improve fault tolerance
28. [RAID](https://en.wikipedia.org/wiki/RAID) configuration improves the hardware throughput
29. The data store is partitioned to spread the writes and reads at the expense of difficult joins, transactions, and fat client
30. Federation and sharding are used to scale out the database
31. The write requests are redirected to the leader and the read requests are redirected to the followers of the database
32. [Vitess](https://vitess.io/) is a storage middleware for scaling out MySQL
33. Vitess redirects the read requests that require fresh data to the leader (For example, update user profile operation)
34. Vitess uses a lock server (Apache Zookeeper) for automatic sharding and leader election on the database layer
35. Vitess supports RPC-based joins, indexing, and transactions on SQL database
36. Vitess allows to offload of partitioning logic from the application and improves database queries by caching

---

#### Video streaming workflow

<image src="imgs/youtube-stream-path.png" alt="YouTube; Video stream workflow" caption="YouTube; Video stream workflow" >

1. The client executes a DNS query to identify the server
2. The client makes an HTTP connection on the load balancer
3. The CDN is queried to verify if the requested video content is on the CDN cache
4. The CDN queries the transcoded object store on a cache miss
5. The load balancer delegates the client's request to a web server with free capacity using the weighted round-robin algorithm
6. The web server delegates the client's request to an app server using consistent hashing
7. The app server queries the metadata cache to fetch the metadata of the video
8. The app server queries the metadata database on a cache miss
9. The app server queries the thumbnail store to fetch the relevant thumbnail images of the video
10. The app server queries the transcoded object store to fetch the video content
11. The app server delegates the search queries of the client to the inverted index store
12. The read and write traffic are segregated for high throughput
13. Popular video content is streamed from CDN (in memory)
14. The push CDN model can be used for caching videos uploaded by users with a significant number of subscribers
15. The moderately streamed video content can be served from the video server directly (disk IO)
16. Consistent hashing can be used to load balance cache servers
17. Caching can be implemented on multiple levels to improve latency
18. LRU cache eviction policy can be used
19. Entropy or jitter is implemented on cache expiration to prevent the thundering herd failure
20. The video files are distributed to the data centers closer to the client when the client starts streaming
21. The traffic should be prioritized between the video streaming cluster (higher priority) and the general cluster to improve reliability
22. The videos can be recommended to the client based on geography, watch history ([KNN algorithm](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)), and [A/B testing](https://en.wikipedia.org/wiki/A/B_testing) results
23. The video file is split into chunks for streaming and improved fault tolerance
24. The chunks of a video file are joined together when the client starts streaming
25. The video chunks allow adaptive streaming by switching to lower-quality chunks if the high-quality chunks are slower to download
26. Different streaming protocols support different video encodings and playback players
27. The video content is streamed on TCP through buffering
28. The video offset is stored on the server to resume playback from any client device
29. The video resolution on playback depends on the client's device
30. The video file should be encoded into compatible bitrates (quality) and formats for smoother streaming and compatibility
31. The counter for likes on a video can be non-accurate to improve performance (transactions executed at sampled intervals)
32. The comments on video content are shown to the comment owner by data fetching from the leader (database) while other users can fetch from followers with a slight delay
33. The services can be prescaled for extremely popular channels as autoscaling might not meet the requirements of concurrent clients
34. The autoscaling requirements can be predicted by performing machine learning on traffic patterns
35. The autoscaling service should keep a time buffer due to the anticipated delays in the provisioning of resources
36. Fully baked container images (no additional script execution required after provisioning) improve the startup time of services
37. The infrastructure can be prewarmed before the peak hours using benchmark data through load testing
38. The emergency mode should shut down non-critical services to free up resources, and skip the execution of failed services for improved reliability
