
### Tagging Service

Some of the popular tagging services are the following:

- JIRA tags
- Confluence tags
- Stackoverflow tags
- Twitter hashtags

---

#### Requirements

- Tag an item
- View the items with a specific tag in near real-time
- Scalable

---

#### Data storage

##### Database schema

<image src="imgs/tagging-service-db-schema.png" alt="Tagging service; Database schema" caption="Tagging service; Database schema" >

- The primary entities of the database are the Tags table, the Items table, and the Tags_Items table
- The Tags_Items is a join table to represent the relationship between the Items and the Tags
- The relationship between the Tags and the Items tables is many-to-many

##### Type of data store

- The media files (images, videos) and text files are stored in a managed object storage such as AWS S3
- A SQL database such as Postgres stores the metadata on the relationship between tags and items
- A NoSQL data store such as MongoDB stores the metadata of the item
- A cache server such as Redis stores the popular tags and items

---

#### High-level design

1. When a new item is tagged, the metadata is stored on the SQL database
2. The popular tags and items are cached on dedicated cache servers to improve latency
3. The non-popular tags and items are fetched by querying the read replicas of SQL and NoSQL data stores

---

#### Write path

<image src="imgs/tagging-service-write-path.png" alt="Tagging service; Write path" caption="Tagging service; Write path" >

1. The client makes an HTTP connection to the load balancer
2. The write requests to create an item or tag an item are rate limited
3. The load balancer delegates the client request to a web server with free capacity
4. The write requests are stored on the message queue for asynchronous processing and improved fault tolerance
5. The fanout service distributes the write request to multiple services to tag an item
6. The object store persists the text files or media files embedded in an item
7. The NoSQL data store persists the metadata of an item (comments, upvotes, published date)
8. The SQL database persists metadata on the relationship between tags and items
9. The tags info service is queried to identify the popular tags
10. If the item was tagged with a popular tag, the item is stored on the items cache server
11. The tags cache server stores the IDs of items that were tagged with popular tags
12. LRU cache is used to evict the cache servers
13. The data objects (items and tags) are replicated across data centers at the web server level to save bandwidth

---

#### Read path

<image src="imgs/tagging-service-read-path.png" alt="Tagging service; Read path" caption="Tagging service; Read path" >

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the tag data is cached on the CDN
3. The client creates an HTTP connection to the load balancer
4. The read requests to fetch the tags or items are rate limited
5. The load balancer delegates the client request to a web server with free capacity
6. The web server queries the tags service to fetch the tags
7. The tags service queries the tags info service to identify if the requested tag is popular
8. The lists of tagged items for a popular tag are fetched from the tags cache server
9. The tags service executes an MGET Redis request to fetch the relevant tagged items from the items cache server
10. The list of items tagged with non-popular tags is fetched from the read replicas of the SQL database
11. The items tagged with non-popular tags are fetched from the read replicas of the NoSQL data store
12. The media files embedded in an item are fetched from the object store
13. The trie data structure is used for typeahead autosuggestion for search queries on tags
