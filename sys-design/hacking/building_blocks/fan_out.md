## 17. Fan-out Service

A fan-out service refers to:

* The delivery and spreading of data to multiple destinations. Typically, this is a one-
  to-many messaging pattern where the data is delivered in parallel asynchronously
  without blocking for responses.
* Creating copies of (replicating) the updated data in caches, databases, and servers for
  read access.
* The notification, search indexing, and updating that are associated with the new data.

Think of a fan-out service as an *umbrella* term for services that are *triggered after a write of
new data* that needs to be sent (copied) to *many recipients*.

The goals of these services are:
1. to efficiently propagate this new data to other users in different locations
2. to allow this data to be readily accessible and searchable assuming it will be read in the
   immediate future

A fan-out service is commonly used in social media applications, where a post or tweet will
trigger this process. For example, when a user posts a new tweet, the system uses a
fan-out service to notify followers about a user's new activity -- post/tweet.

The diagram below shows the services that fall under the term "fan-out services":
* search index service
* timeline update service
* notification service
* newsfeed blend service
* copy service

![](../imgs/0050.jpg)

Some of the services and components:

* Newsfeed: contains a list of display on the user's homepage:
  * news items
  * posts
  * ads customized per user
  
  This newsfeed is computed before a user's login (online ranking may take too long), and
  copies of the newsfeed are kept in caches and read replicas so that they can be served
  on a user's login.

* Newsfeed Blend Service: on creation of a new post or news item, the newsfeed blend service
  updates the copies of the newsfeed in the caches and read replicas. It combines
  ("blends") the new item into the existing newsfeed items based on the item's ranking
  or it discards items that are not relevant to the user. The caches and replicas are located
  in different zones and regions.

* Timeline: a list of items specific to a user:
  * their posts
  * activities
  * and other updates
  
  A timeline is presented in reverse chronological order (newest on top) and is shown on a 
  user's profile. Like newsfeeds, timelines are computed before users navigate to a profile and 
  are kept in caches and read replicas.

* Timeline Update Service: on a user's new activity, the timeline update service updates
  the copies of the timeline in the caches and read replicas. Because timelines are sorted
  in reverse chronological order (newest or greatest time first), this update simply
  appends the new item onto a list.

* Search Index Service: adds the new post to a reverse index used by the search engines
  for lookups. Users search for the new post based on keywords.

* Copy Service: copies the images, videos, and static content associated with the new
  post to object stores and CDNs, which are in different zones and regions. These copies
  are located physically closer to users who are **expected** to engage with the post.

* Notification Service: broadcasts a message to the user's followers, alerting them about
  the new post. The message contains a contextual link that will route the follower to
  the new post.

In the diagram, a follower ("read client") is notified that the user made a new post through the
notification. The read client sends a request to the Read API, which retrieves the follower's
newsfeed from the closest cache in zone 2. Similarly, the read client access images and videos
from zone 2.

One of the challenges of the fan-out service is to effectively scale a fan-out service,
considering new items and notifications need to be propagated to users quickly and
without race conditions.

*New comment reply to a new post, received before original post is received*
Suppose the following sequence of events happens:

1. User A has tens of millions of followers makes a post. The fan-out services start to
   update millions of newsfeeds and send out notifications.
2. User B, one of User A's followers, receives this notification. He creates a post
   commenting on User A's original post before millions of User A's followers have
   received the original post.
3. User B has a few thousand followers, and the fan-out for his post is completed before
   the fan-out for User A's post!
4. Other users are confused by the second post, having not seen the original post yet.

The system must have enough capacity so that notifications and updates can be processed
relatively quickly. In addition, the fan-out services must assure some sequential
consistency between posts in different regions such that e.g., *older writes must
appear before newer ones*.

Another challenge is that it is computationally costly to update newsfeeds at each new post or
news item.

This becomes more complex if multiple copies of the newsfeed are held in memory
for each user, which happens when it is not known which cache the read client will be closest
to.

#### Push
The fan-out service designed above used a push model, where new items are propagated
out to users as they are created. An alternative solution is to use a pull model for users that
don't login in frequently.

#### Pull
In the pull model, the fan-out would happen at a regular interval or when the read client makes
a request. Though this would be fewer updates for the system to process, users could receive
newsfeeds that are stale and don't contain the latest posts.

