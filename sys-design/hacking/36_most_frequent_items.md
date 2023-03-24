## 36. Design a System that Tracks the Most Frequently Accessed Items (13)

Designing a system that tracks the frequently accessed items is a popular system design
question (aka "Top N," "heavy hitters," "most popular items," or "real-time view
count.")

It is commonly used to create "Top N" lists for a large number of items in real-time using
limited space.

* YouTube has over 5 billion videos. What are the top 100 most viewed videost
* Amazon has over 100 million products. What are the top 100 bestselling product?
* Facebook has over 1 trillion posts/news items. What are the most liked or trending
  items?
* Twitter has over 200 billion tweets per year. What are the most popular tweets this
  year?
* Spotify has over 80 million songs. What are the top 100 most listened to songs

Each question can also be answered over different time ranges. What are the top N items in
the past 5 minutes, 1 hour, day, week, month, and all-time?

A core aspect of this problem is that these Top N items need to be computed in real-time. The
system needs to compute the top N items quickly and efficiently but is allowed to give up some
accuracy in return for these constraints.

Possible approaches:

* Why don't we increment a counter for each item and then sort them?
  Using a hash map from item id to count seems to be the most straightforward
  approach; however, it runs into a scaling problem because hash maps have O(n) space
  complexity. Suppose we use this approach to find the top tweets of the year. Suppose
  each counter uses ~100 bytes, which includes tweet metadata and the counter. Hence
  the memory requirements would be 200 billion tweets per year * 100 bytes = 200 TB,
  which might exceed capacity for a single server to track and sort in real-time.

* This tracking and sorting also must be performed for multiple time ranges and
  categories ( the top tweets about cats in the past week). The number of
  permutations of Top N lists increases the space requirements exponentially, the
  approach doesn't meet the real-time requirement, nor does it scale well. The sheer size
  of the sampling space makes it infeasible to keep a counter for each sample.

* Why don't we perform a MapReduce on the logs?
  In "Autocomplete" and "Core Components" MapReduce was used to count the frequency of
  search phrases. While MapReduce can generate counts of each item by processing the
  system's logs, this is performed offline,by batching the logs. This doesn't meet
  the real-time requirement; a real-time count needs to process streams instead of logs.

* Why don't we subsample the data?
  Since the requirements for accuracy have been relaxed in this question, one approach
  would be to subsample the data and extrapolate. For example, we could stagger the
  counting so that each item's count is only incremented for 1 out of every 10 minutes.
  We could then multiply the count by 10 to estimate the actual count. Though this
  would reduce the computational cost of the counting, it doesn't change the cardinality
  of the problem. That is, the number of items still stays the same, and the system still
  has the same memory and scalability issues. Additionally, there may be statistical bias
  and errors in the extrapolation if the usage is non-uniform or occurs in bursts.

* Why don't we maintain distributed counters and then aggregate them?
  Distributed counters would need to be aggregated and sorted, effectively turning this
  approach into the MapReduce approach. Additionally, the real-time requirement
  suggests that the counts be precomputed instead of requiring aggregation and sorting.

This question centers around trading accuracy for greater efficiency in limited space, 
a certain probabilistic data structure: the *count-min sketch (CMS)* solves this. A *sketch* is a
data structure that can keep a large amount of data in constant space but at the cost of accuracy.

### 1. Clarify the problem and scope the use cases
#### Use Cases
* A service tracks and returns the top N items by count that've been viewed, liked, or
  accessed.
* The top N items should be computed for multiple time ranges, including the past 5
  minutes, 1 hour, day, week, month, and all-time.
* The items can be partitioned into categories and sub-categories. The top N of each
  category and sub-category should be tracked as well.

#### Requirements
* The top N list should be updated and computed in real-time.
* The counts do not need to be accurate, but +/-10% is a tolerable margin of error.
* The system should be able to scale to trillions of items.
* Services of the system should be performant and have high availability.


