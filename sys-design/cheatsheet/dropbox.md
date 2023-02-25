### Dropbox

The popular file hosting services are the following:

- Dropbox
- Google Drive
- OneDrive
- iCloud

---

#### Requirements

- The **client** (user) can upload and download files on any device
- The client can share files or directories with other clients
- The client can edit the files while offline
- The file changes are automatically synchronized across devices
- The operations on files are ACID compliant

---

#### Data storage

##### Database schema

<image src="imgs/dropbox-database-schema.png" alt="Dropbox; Database schema" caption="Dropbox; Database schema" >

- The **ACL** (access control list) table is used to define file permissions
- The relationship between the users and the devices tables is 1-to-many
- The relationship between the users and the ACL tables is 1-to-many
- The relationship between the files and the ACL tables is 1-to-many
- The relationship between the workspace and the files tables is 1-to-many
- The relationship between the files and the file_versions tables is 1-to-many
- The relationship between the file_versions and the blocks tables is 1-to-many

---

##### Type of data store

- An embedded SQL database such as [SQLite](https://www.sqlite.org/index.html) can be used as the metadata database on the client
- A SQL database such as MySQL stores the metadata on users, and files
- A cache server such as Redis stores the metadata and file blocks of frequently accessed files
- An object store can be used to store the file blocks
- The message queue such as Apache Kafka is used for synchronization across devices using the request-response pattern

---

#### High-level design

- The files are chunked into blocks of size 4 MB for storage
- The device-A of the client uploads file blocks and commits the metadata
- A long polling connection is maintained by device-B of the client to get notified of any file changes
- The device-B of the client fetches the updated metadata and the relevant file blocks
- Only the updated file blocks should be transferred to save bandwidth

---

#### Upload workflow

<image src="imgs/dropbox-upload-workflow.png" alt="Dropbox; Write Path" caption="Dropbox; Write Path" >

1. The client creates an HTTP connection on the load balancer to upload the file
2. The load balancer delegates the client's HTTP connection to a web server with free capacity using round robin algorithm
3. The web server delegates the file upload request to the block service
4. The block service persists the file blocks on the block data store
5. The web server delegates the file metadata change request to the metadata service
6. The metadata service persists the modified metadata on the metadata database
7. The metadata service puts a message on the message queue to notify other devices of the client of file changes
8. [Optimistic locking](https://en.wikipedia.org/wiki/Optimistic_concurrency_control) can be used for concurrent updates on the same file
9. The metadata database stores the metadata of the user and the file blocks (useful for offline editing)
10. The metadata database can be partitioned using consistent hashing (key = user-id or file-id)
11. The file deduplication can be performed inline optimally by the client through hashing of the file blocks
12. Alternatively, the server can perform the deduplication of file blocks at the expense of increased bandwidth and storage
13. The file conflicts on concurrent writes can be resolved through the last-write-wins policy or manual resolution by the client
14. The least recently accessed files can be archived and moved to a cold data store to save storage
15. The client's indexer service updates the internal metadata database
16. The client's chunker service splits the file into blocks and each block is identified by a unique hash value
17. The client's metadata database contains metadata on the files, file chunks, file versions, and the file path
18. The file versioning can be implemented using SHA-256 hash or checksum on files
19. The file blocks are compressed using gzip or bzip2 to save storage and bandwidth
20. The client encrypts the file blocks using the 128-bit AES algorithm
21. The client uploads file patches on small changes to files
22. Exponential backoff is used to improve fault tolerance
23. The block data store is replicated to improve availability and durability

---

#### Download workflow

<image src="imgs/dropbox-download-workflow.png" alt="Dropbox; Read Path" caption="Dropbox; Read Path" >

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the most frequently accessed files are on the cache
3. The client initiates an HTTP connection on the load balancer to fetch the file
4. The load balancer delegates the client's HTTP connection to a web server with free capacity
5. A long polling connection is maintained by the client to get notified of any file changes in real-time
6. The notification service queries the message queue for any file changes using the request-response pattern
7. The web server queries the metadata service to fetch the modified metadata on any file changes
8. The metadata service queries the metadata cache
9. The metadata database is queried on a cache miss
10. The web server queries the block service to fetch the modified file blocks
11. The block service queries the block cache
12. The block data store is queried on a cache miss
13. The block service queries the preview service to fetch the preview (screenshot) of the files
14. The preview service asynchronously queries the block store to generate a preview of the stored files
15. The preview service queries the preview cache to fetch the preview of recently accessed files
16. The preview data store is queried on a cache miss
17. LRU cache eviction policy can be used for cache servers
18. The cached previews are stored in an encrypted format for security
19. Thin clients such as mobile users can perform on-demand synchronization with remote servers to save bandwidth
20. The client can use long polling, HTTP/2, or SSE to receive updates from the server in real-time
21. The notification service can be implemented using the request-response pattern on the message queue
22. The request queue is a global queue that receives updates from all devices while the response queues are dedicated to each device and the messages are removed on delivery
23. A local search index is generated for searching the files
24. The ACL table is modified when the user shares a file with another user
25. The client's watcher service detects file changes on the workspace for automated synchronization
26. The client's indexer service communicates with the remote notification service
27. The client's chunker service joins the file blocks on the read operation to reconstruct the file
