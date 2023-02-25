
### Online Marketplace

The popular implementations of an online marketplace are the following:

- Amazon
- eBay
- Shopee
- TicketMaster
- Airbnb

---

#### Requirements

- The user can create an inventory of products in the store
- The users can search for products on the storefront
- The user can make the payment to purchase a product on the storefront

---

#### Data storage

##### Database schema

<image src="imgs/online-marketplace-database-schema.png" alt="Online Marketplace; Database schema" caption="Online Marketplace; Database schema" >

- The primary entities are the inventory, the tickets (products), the users, the payments, and the purchase orders tables
- The relationship between the inventory and the tickets table is 1-to-many
- The relationship between the purchase orders and the tickets table is 1-to-many
- The relationship between the users and the purchase orders table is 1-to-many
- The relationship between the purchase orders and the payments table is 1-to-1

##### Type of data store

- Document store such as MongoDB persists unstructured data like the inventory and the tickets data
- Document store such as Splunk persists the logs for observability
- SQL database such as Postgres persists the purchase orders, payments information, and users metadata for ACID compliance
- An object store such as AWS S3 is used to store media files (images, videos) related to the tickets
- A cache server such as Redis is used to store the transient data (session data and trending tickets)
- Message queue such as Apache Kafka temporarily stores the logs for asynchronous processing
- Lucene-based inverted index data store such as Apache Solr is used to store index data for search functionality

---

#### High-level design

- The purchase of a ticket is executed as a two-phase commit
- The materialized conflict or the distributed lock is used to handle concurrency when multiple users try to buy the same unique ticket
- The inverted index store provides the search functionality
- The high-profile events might put users into a virtual waiting room before they can view the ticket listings to improve fairness

---

#### Write path

<image src="imgs/online-marketplace-write-path.png" alt="Online Marketplace; Write path" caption="Online Marketplace; Write path" >

1. The bulk service stores the JSON metadata files of the inventory and the related media files on the object store
2. The bulk service puts the metadata on the message queue for an asynchronous processing
3. The aggregation service is notified through the publish-subscribe pattern
4. The aggregation service queries the object store to fetch the JSON files
5. The aggregation service validates, normalizes the JSON data, and stores the transformed data on the inventory data store
6. The indexing service executes a batch fetch operation on the inventory data store
7. The indexing service transforms and stores the fetched data on the inverted index store
8. The ads service queries the inventory data store to display advertisements to the users

---

#### Read path

<image src="imgs/online-marketplace-read-path.png" alt="Online Marketplace; Read path" caption="Online Marketplace; Read path" >

1. The client queries the DNS for domain name resolution
2. The CDN is queried to check if the requested ticket is in the CDN cache
3. The client creates an HTTP connection to the server
4. The server delegates the client's request to view the ticket inventory
5. The inventory service queries the trending cache to check if the requested ticket is on the cache
6. The inventory service queries the read replicas of the inventory datastore on a cache miss
7. The inventory service queries the object store to fetch the relevant media files of the ticket
8. The server delegates the client's request to search for a ticket
9. The search service queries the inverted index store to fetch the relevant tickets
10. The search service queries the object store to fetch the relevant media files
11. The server delegates the client's request to purchase a ticket
12. The distributed lock defined with a short TTL is acquired to prevent concurrent users from purchasing the same unique ticket
13. The purchase service stores the session data in a linked hashmap with a reasonable TTL on the session cache
14. The purchase service stores the purchase information on the cart database and updates the inventory data store
15. The purchase service invokes the payment service to process the financial transaction
16. The server persists the logs on the message queue for asynchronous processing
17. The log service fetches the logs on the message queue for further processing
18. The log service aggregates the logs and stores the logs on the document store
19. The operations (13–15) to purchase a ticket are executed as a two-phase commit to support rollback on failure
20. Materialized conflict (alternative to distributed lock) is implemented by creating database rows for each ticket at the expense of storage costs and by acquiring a lock on database rows
21. The operation of putting a ticket into the shopping cart or navigating to the payment page should temporarily reserve the tickets for a reasonable time frame
22. Autoscaling of services and serverless components can be used to save costs
23. The [pgbouncer](https://www.pgbouncer.org/) can be used for connection pooling to Postgres
