---

### Facebook Newsfeed

The keywords _newsfeed_ and _timeline_ are used interchangeably. Some of the similar services to Facebook Newsfeed are the following:

- Twitter Timeline
- Instagram feed
- Google podcast feed
- Google news timeline
- Etsy feed
- Feedly
- Reddit feed
- Medium feed
- Quora feed

---

#### Requirements

- The user newsfeed must be generated in near real-time based on the feed activity from the people that a user follows
- The feed items contain text and media files (images, videos)

---

#### Data storage

##### Database schema

<image src="imgs/facebook-timeline-db-erd.png" alt="Facebook Newsfeed; Database schema" caption="Facebook Newsfeed; Database schema" >

- The primary entities of the database are the Users table, the FeedItems table, and the Follows table
- The relationship between the Users and the FeedItems tables is 1-to-many
- The relationship between the Users and the Follows tables is many-to-many
- The Follows is a join table to represent the relationship (follower-followee) between the users

##### Type of data store

- The media files (images, videos) are stored in a managed object storage such as AWS S3
- A SQL database such as Postgres stores the metadata of the user (followers, personal data)
- A NoSQL data store such as Cassandra stores the user timeline
- A cache server such as Redis stores the pre-generated timeline of a user

---

#### High-level design

<image src="imgs/home-timeline.png" alt="Facebook Newsfeed; Home timeline" caption="Facebook Newsfeed; Home timeline" >

1. The server stores the feed items in cache servers and the NoSQL store
2. The newsfeed generated is stored on the cache server
3. There is no feed publishing for inactive users but uses a pull model (fanout-on-load)
4. The feed publishing for active non-celebrity users is based on a push model (fanout-on-write)
5. The feed publishing for celebrity users is based on a hybrid push-pull model
6. The client fetches the newsfeed from the cache servers

---

#### Write path

<image src="imgs/facebook-newsfeed-write-path.png" alt="Facebook Newsfeed; Write path" caption="Facebook Newsfeed; Write path" >

1. The client creates an HTTP connection to the load balancer to create a feed item
2. The write requests to create feed items are rate limited
3. The load balancer delegates the client connection to a web server with free capacity
4. The feed item is stored on the message queue for asynchronous processing and the client receives an immediate response
5. The fanout service distributes the feed item to multiple services to generate the newsfeed for followers of the client
6. The object store persists the video or image files embedded in the feed item
7. The NoSQL store persists the timeline of users (feed items in reverse chronological order)
8. The SQL database stores the metadata of the users (user relationships) and the feed items
9. A limited number of feed items for users with a certain threshold of followers are stored on the cache server
10. The IDs of feed items are stored on the user timeline cache server for deduplication
11. The feed generation service subscribes to the fanout service for any updates
12. The feed generation service queries the in-memory user info service to identify the followers of a user and the category of a user (active non-celebrity users, inactive, celebrity users)
13. The feed generation service creates the home timeline for active non-celebrity users using a push model (fanout-on-write) in linear time O(n), where n is the number of followers
14. The feed items are ranked, sorted, and merged to generate the home timeline for a user
15. The home timeline for active users is stored on the cache server for quick lookups
16. There is no feed publishing for inactive users but uses a pull model (fanout-on-load)
17. The feed publishing for celebrity users is based on a hybrid push-pull model (merge celebrity feed items to the home timeline of a user on demand)
18. As an alternative, the feed publishing for celebrity users can use a push model only for the online followers in batches (not optimal solution)

---

#### Read path

<image src="imgs/facebook-newsfeed-read-path.png" alt="Facebook Newsfeed; Read path" caption="Facebook Newsfeed; Read path" >

1. The client executes a DNS query to resolve the domain name
2. The client queries the CDN to check if the feed items for the home timeline are cached on the CDN
3. The client creates an HTTP connection to the load balancer
4. The read requests to fetch the newsfeed are rate limited
5. The load balancer delegates the client connection to a web server with free capacity
6. The web server queries the timeline service to fetch the newsfeed
7. The timeline service queries the user info service to get the list of followee and identify the category of the user (active, inactive, following celebrities)
8. The home timeline cache is queried to fetch the list of feed item IDs
9. The feed items are fetched from the feed items cache server by executing an MGET operation on Redis
10. When the client executes a request to fetch the timeline of another user, the timeline service queries the user timeline cache server
11. The SQL database follower (replica) is queried on a cache miss
12. The media files embedded on feed items are fetched from the object store
13. The NoSQL data store is queried to fetch the user timeline on a cache miss
14. The inactive users fetch the home timeline using a pull model (fanout-on-load)
15. The active users following celebrity users use a hybrid model to fetch the home timeline (the feed items from celebrities are merged on demand)
