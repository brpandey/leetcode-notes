

### Pastebin

Similar services: GitHub gist

---

#### Requirements

- The client must be able to upload text data and receive a unique URL
- The received URL is used to access the text data

---

#### Data storage

##### Database schema

<image src="imgs/pastebin-db-schema.png" alt="Pastebin; Database schema" caption="Pastebin; Database schema" >

- The primary entities of the database are the Pastes table, the Users table
- The relationship between the Users and the Pastes tables is 1-to-many

##### Type of data store

- The content of a paste is stored in a managed object storage such as AWS S3
- A SQL database such as Postgres or MySQL is used to store the metadata (paste URL) of the paste

---

#### High-level design

1. The server generates a unique paste identifier (**ID**) for each new paste
2. The server encodes the paste ID for readability
3. The server persists the paste ID in the metadata store and the paste in the object storage
4. When the client enters the paste ID, the server returns the paste

---

#### Write path

<image src="imgs/pastebin-write-path.png" alt="Pastebin; Write path" caption="Pastebin; Write path" >

1. The client makes an HTTP connection to the server
2. Writes to Pastebin are rate limited
3. Key Generation Service (**KGS**) creates a unique encoded paste ID
4. The object storage returns a presigned URL
5. The paste URL (http://presigned-url/paste-id) is created by appending the generated paste ID to the suffix of the presigned URL
6. The paste content is transferred directly from the client to the object storage using the paste URL to optimize bandwidth expenses and performance
7. The object storage persists the paste using the paste URL
8. The metadata of the paste including the paste URL is persisted in the SQL database
9. The server returns the paste ID to the client for future access

---

#### Read path

<image src="imgs/pastebin-read-path.png" alt="Pastebin; Read path" caption="Pastebin; Read path" >

1. The client executes a DNS query to identify the server
2. The CDN is queried to verify if the requested paste is in the CDN cache
3. The client makes an HTTP connection to the load balancer or the reverse proxy server
4. The read requests are rate limited
5. The load balancer delegates the client connection to the server with free capacity
6. The server verifies if the paste exists by querying the bloom filter
7. If the paste exists, check if the paste is stored in the cache server
8. Fetch the metadata for the paste from the SQL database
9. Fetch the paste content from the object storage using the metadata
