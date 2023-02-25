
### Twitter Search

Some of the similar services to Twitter Search are the following:

- Facebook Search
- YouTube search
- Reddit search
- Google news search

---

#### Requirements

- Persist user status
- Query user status in near real-time

---

#### Data storage

##### Database schema

A positional inverted index is used to store the index.

##### Type of data store

- The archived data objects are stored on HDFS
- Lucene-based inverted index data store such as Apache Solr or Elasticsearch is used to store index data

---

#### High-level design

1. The fanout service queries the real-time index and the archive index to identify the relevant data objects
2. The fanout service queries the social graph service and user search service to filter the result data objects further

---

#### Read path

<image src="imgs/twitter-search-architecture.png" alt="Twitter search; Read path" caption="Twitter search; Read path" >

1. The client establishes an HTTP connection to the search server, which parses and tokenizes the search query
2. The search server delegates the search request to the fanout service
3. The fanout service queries the real-time index to identify the relevant most recent data objects
4. The fanout service queries the cache server to verify if the relevant archive index data is on the cache
5. The fanout service queries the archive index to identify the relevant archived data objects
6. The fanout service queries the social graph service to filter the result data objects by identifying the most relevant data objects to the user
7. The fanout service queries the user search service to further filter the result data object by applying personalization based on the user search history
8. The result data objects are merged by the fanout service and returned to the client
9. Wait-free concurrency model with optimistic locking (the search index is updated while being read at the same time) is implemented to achieve high throughput for the search service
10. RPC is used for internal communication between the system components to improve latency

---

#### Write path

<image src="imgs/twitter-search-write-path.png" alt="Twitter search; Write path" caption="Twitter search; Write path" >

1. The real-time stream is analyzed using Apache Flink to generate an in-memory Lucene index
2. The real-time index keeps data that is at most a few weeks old
3. MapReduce (Hadoop) jobs are executed on archive data objects stored on HDFS
4. The Lucene-based archive index is stored on the SSD
5. The Lucene-based user search service is updated depending on the search history of the user
6. The multithreading in search service is implemented through a single writer thread and multiple reader threads

<image src="imgs/twitter-search-update.png" alt="Twitter search; Updating the index" caption="Twitter search; Updating the index" >

When the client updates or deletes any published data object, the server updates the real-time index and archive index.
