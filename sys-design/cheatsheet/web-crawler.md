### Distributed Web Crawler

Some of the popular distributed web crawlers are the following:

- Google bot
- Bing bot
- Apache Nutch

---

#### Requirements

- An automated web crawler to crawl through the HTML files on the internet
- The web crawler is distributed
- The web crawler must start crawling from a set of seed webpages
- The web crawler must be polite to websites (obey the robots.txt file)

---

#### Data storage

##### Database schema

- The content store persists the document IDs and document content
- The seed URL store persists a list of seed URLs
- The URL store persists the list of URLs to crawl and the source URLs

##### Type of data store

- The URL storage persists extracted list of URLs on a NoSQL data store such as Hbase or HDFS
- The crawled content is stored in a managed object storage such as AWS S3 or on a NoSQL data store such as Apache HBase or Cassandra
- DNS persists domain names and the IP addresses
- The seed URL storage persists a list of seed URLs on a NoSQL data store such as Cassandra or HDFS
- The message queue such as Apache Kafka is used as the dead-letter queue
- The cache server such as Redis keeps the fresh crawled documents in memory for quicker processing
- The NoSQL data store such as Cassandra or an object store such as AWS S3 stores the content of crawled web pages
- Apache Zookeeper is used for service discovery

---

#### High-level design

At a high level, the web crawler executes steps 2 and 3 repeatedly.

1. The fetcher service crawls the URLs on the seed store
2. The extracted outlinks (URLs) on the crawled website are stored on the URL store
3. The fetcher service crawls the URLs on the URL store
4. The crawler uses the BFS algorithm

---

#### Workflow

<image src="imgs/distributed-web-crawler.png" alt="Distributed web crawler" caption="Distributed web crawler" >

1. The URL frontier queries the seed URL storage to fetch a list of URLs to be crawled
2. The URL frontier prioritizes the URLs to be crawled
3. The fetcher service queries the scheduler service to check if the URL has a predefined crawl schedule
4. The local DNS service is queried to identify the IP address of the origin server
5. The fetcher service server-side renders the web pages
6. The duplicate check service is queried to check for duplicate content on the web page
7. The fetcher service compresses the crawled web page and stores it on the content store for further processing such as building an inverted index
8. The fetcher service stores the crawled web page on the content cache for immediate processing
9. The fetcher service publishes the document ID to the message queue for asynchronous processing of the crawled web pages
10. The URL processor is informed about the crawled web page using the publish-subscribe pattern
11. The URLs are extracted, filtered, and normalized from the crawled web page by querying the content cache
12. The extracted URLs are stored on the URL storage for future crawling
13. The URL frontier queries the URL storage to fetch the URLs to crawl
14. The services sent heartbeat signals to the Apache zookeeper for improved fault tolerance
15. Only limited HTTP connections are created to an origin server to improve the politeness of the crawler
16. The local DNS service is used to improve latency
17. The URL frontier uses message queues to prioritize the URLs to be crawled and improve the politeness of the crawler
18. The fetcher service is multi-threaded to concurrently crawl multiple webpages
19. The fetcher service performs server-side rendering of the web pages to handle the dynamic web pages
20. The URL extractor, URL filter, and URL normalizer service runs on Apache Spark (MapReduce) jobs to improve throughput
21. The publish-subscribe pattern among services is implemented using the message queue
22. The message queue implements the backpressure pattern to improve the fault tolerance
23. The duplicate check service uses the simhash algorithm to detect the similarity in the content on the web pages
24. Consistent hashing is used to partition the content cache (key = document ID)
25. The read replicas of the content cache serve the latest documents for further processing
26. RPC is used for internal communication to improve latency
27. The stateful services are periodically checkpointed to improve the fault tolerance
28. The sitemap.xml file is used by the webmaster to inform the web crawler about the URLs on a website that is available for crawling
29. The robots.txt file is used by the webmaster to inform the web crawler about which portions of the website the crawler is allowed to visit
30. The web pages are fetched and parsed in stream jobs using Apache Flink to improve throughput
31. The web pages are ranked and further processed in batch jobs on Apache Spark
32. The bloom filter is used by the URL processor (extract, filter, normalizer) to verify if the URL was crawled earlier
33. The fetcher service skips crawling the non-canonical URLs but instead crawls the related canonical links
34. The Apache Gora is used as an SQL wrapper for querying the NoSQL data store
35. The Apache Tika is used to detect and parse multiple document formats
36. The web pages that return a 4XX or 5XX status code are excluded from crawling retries
37. The redirect pages are crawled on the HTTP 3XX response status code
38. The scheduler service returns a fixed or adaptive schedule based on the sitemap.xml file definition
39. The user-agent HTTP header in the request is set to the name of the crawler
40. The web crawler is distributed geographically to keep the crawler closer to the origin servers of the website to improve latency
41. The storage services are distributed and replicated for durability

---

#### URL Frontier

<image src="imgs/web-crawler-url-frontier.png" alt="Distributed web crawler; URL frontier" caption="Distributed web crawler; URL frontier" >

The prioritizer service puts the URLs based on the priority in distinct message queues. The online page importance calculation (OPIC) or link rank is used to assign priority to the URLs. Consistent hashing can be used to distribute the URLs across message queues.

The priority selector service fetches the URLs on the high-priority queue and puts the URLs of a specific website on a single message queue for sequential crawling. Sequential crawling improves the politeness of the crawler.
