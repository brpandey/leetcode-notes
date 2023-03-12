## Requirements, Scope, & Back of the Envelope Calculations

System design interview questions are intentionally left open-ended. To design a proper system, it is critical to ask clarifying questions and also to establish scope. Cover assumptions and calculations with your interviewer so that you are on the same page.

### 8: DESIGN A URL SHORTENER

Step 1 - Understand the problem and establish design scope

* Candidate: Can you give an example of how a URL shortener work?
* Interviewer: Assume URL
https://www.foobar.com/q=key9998&c=1234&v=v3&l=xyz is the original URL. Your service creates an alias with shorter length: https://tinyurl.com/y7keocwj. If you click the alias, it redirects you to the original URL.

* Candidate: What is the traffic volume?
* Interviewer: 100 million URLs are generated per day.
* Candidate: How long is the shortened URL?
* Interviewer: As short as possible.
* Candidate: What characters are allowed in the shortened URL?
* Interviewer: Shortened URL can be a combination of numbers (0-9) and characters (a-z, A-Z).
* Candidate: Can shortened URLs be deleted or updated?
* Interviewer: For simplicity, let us assume shortened URLs cannot be deleted or updated.

Here are the basic use cases:
1. URL shortening: given a long URL => return a much shorter URL
2. URL redirecting: given a shorter URL => redirect to the original URL
3. High availability, scalability, and fault tolerance considerations

#### Back of the envelope estimation
* Write operation: 100 million URLs are generated per day.
* Write operation per second: 100 million / 24 / 3600 = 1160
* Read operation: Assuming ratio of read operation to write operation is 10:1, read operation per second: 1160 * 10 = 11,600
* Assuming the URL shortener service will run for 10 years, this means we must support 100 million * 365 * 10 = 365 billion records.
* Assume average URL length is 100.
* Storage requirement over 10 years: 365 billion * 100 bytes * 10 years = 365 TB

### 9: DESIGN A WEB CRAWLER

Step 1 - Understand the problem requirements and establish design scope

The basic algorithm of a web crawler is simple:

1. Given a set of URLs, download all the web pages addressed by the URLs.
2. Extract URLs from these web pages
3. Add new URLs to the list of URLs to be downloaded. Repeat these 3 steps.

* Candidate: What is the main purpose of the crawler? Is it used for search engine indexing, data mining, or something else?
* Interviewer: Search engine indexing.
* Candidate: How many web pages does the web crawler collect per month?
* Interviewer: 1 billion pages.
* Candidate: What content types are included? HTML only or other content types such as PDFs and images as well?
* Interviewer: HTML only.
* Candidate: Shall we consider newly added or edited web pages?
* Interviewer: Yes, we should consider the newly added or edited web pages.
* Candidate: Do we need to store HTML pages crawled from the web?
* Interviewer: Yes, up to 5 years
* Candidate: How do we handle web pages with duplicate content?
* Interviewer: Pages with duplicate content should be ignored.

Characteristics of a good web crawler:

* Scalability: The web is very large. There are billions of web pages out there. Web
crawling should be extremely efficient using parallelization.
* Robustness: The web is full of traps. Bad HTML, unresponsive servers, crashes,
malicious links, etc. are all common. The crawler must handle all those edge cases.
* Politeness: The crawler should not make too many requests to a website within a short
time interval.
* Extensibility: The system is flexible so that minimal changes are needed to support new
content types. For example, if we want to crawl image files in the future, we should not
need to redesign the entire system.

#### Back of the envelope estimation & assumptions

* Assume 1 billion web pages are downloaded every month.
* QPS: 1,000,000,000 / 30 days / 24 hours / 3600 seconds = ~400 pages per second.
* Peak QPS = 2 * QPS = 800
* Assume the average web page size is 500k.
* 1-billion-page x 500k = 500 TB storage per month. If you are unclear about digital
storage units, go through “Power of 2” section in Chapter 2 again.
* Assuming data are stored for five years, 500 TB * 12 months * 5 years = 30 PB. A 30 PB
storage is needed to store five-year content.

### 13: DESIGN A SEARCH AUTOCOMPLETE SYSTEM

Step 1 - Understand the problem and establish design scope

The first step to tackle any system design interview question is to ask enough questions to
clarify requirements. 

* Candidate: Is the matching only supported at the beginning of a search query or in the middle as well?
* Interviewer: Only at the beginning of a search query.
* Candidate: How many autocomplete suggestions should the system return?
* Interviewer: 5
* Candidate: How does the system know which 5 suggestions to return?
* Interviewer: This is determined by popularity, decided by the historical query frequency.
* Candidate: Does the system support spell check?
* Interviewer: No, spell check or autocorrect is not supported.
* Candidate: Are search queries in English?
* Interviewer: Yes. If time allows at the end, we can discuss multi-language support.
* Candidate: Do we allow capitalization and special characters?
* Interviewer: No, we assume all search queries have lowercase alphabetic characters.
* Candidate: How many users use the product?
* Interviewer: 10 million DAU.

#### Requirements summary

* Fast response time: As a user types a search query, autocomplete suggestions must show up fast enough. An article about Facebook’s autocomplete system [1] reveals that the system needs to return results within 100 milliseconds. Otherwise it will cause stuttering.
* Relevant: Autocomplete suggestions should be relevant to the search term.
* Sorted: Results returned by the system must be sorted by popularity or other ranking
models.
* Scalable: The system can handle high traffic volume.
* Highly available: The system should remain available and accessible when part of the
system is offline, slows down, or experiences unexpected network errors.

#### Back of the envelope estimation
* Assume 10 million daily active users (DAU).
* An average person performs 10 searches per day.
* 20 bytes of data per query string:
* Assume we use ASCII character encoding. 1 character = 1 byte
* Assume a query contains 4 words, and each word contains 5 characters on average.
* That is 4 x 5 = 20 bytes per query.
* For every character entered into the search box, a client sends a request to the backend
for autocomplete suggestions. On average, 20 requests are sent for each search query. For example, the following 6 requests are sent to the backend by the time you finish typing “dinner”.

* search?q=d
* search?q=di
* search?q=din
* search?q=dinn
* search?q=dinne
* search?q=dinner

* ~24,000 query per second (QPS) = 10,000,000 users * 10 queries / day * 20 characters / 24 hours / 3600 seconds.
* Peak QPS = QPS * 2 = ~48,000
* Assume 20% of the daily queries are new. 10 million * 10 queries / day * 20 byte per query * 20% = 0.4 GB. This means 0.4GB of new data is added to storage daily.


### 14: DESIGN YOUTUBE

Step 1 - Understand the problem and establish design scope
Besides watching a video, you can do a lot more on YouTube. For example, comment, share, or like a video, save a video to playlists, subscribe to a channel, etc. It is impossible to design everything within a 45- or 60-minute interview, so ask questions to narrow down the scope.

* Candidate: What features are important?
* Interviewer: Ability to upload a video and watch a video.
* Candidate: What clients do we need to support?
* Interviewer: Mobile apps, web browsers, and smart TV.
* Candidate: How many daily active users do we have?
* Interviewer: 5 million
* Candidate: What is the average daily time spent on the product?
* Interviewer: 30 minutes.
* Candidate: Do we need to support international users?
* Interviewer: Yes, a large percentage of users are international users.
* Candidate: What are the supported video resolutions?
* Interviewer: The system accepts most of the video resolutions and formats.
* Candidate: Is encryption required?
* Interviewer: Yes
* Candidate: Any file size requirement for videos?
* Interviewer: Our platform focuses on small and medium-sized videos. The maximum allowed video size is 1GB.
* Candidate: Can we leverage some of the existing cloud infrastructures provided by Amazon, Google, or Microsoft?
* Interviewer: That is a great question. Building everything from scratch is unrealistic for most companies, it is recommended to leverage some of the existing cloud services.

In the chapter, we focus on designing a video streaming service with the following features:
* Ability to upload videos fast
* Smooth video streaming
* Ability to change video quality
* Low infrastructure cost
* High availability, scalability, and reliability requirements
* Clients supported: mobile apps, web browser, and smart TV

#### Back of the envelope estimation

The following estimations are based on many assumptions, so it is important to communicate
with the interviewer to make sure she is on the same page.
* Assume the product has 5 million daily active users (DAU).
* Users watch 5 videos per day.
* 10% of users upload 1 video per day.
* Assume the average video size is 300 MB.
* Total daily storage space needed: 5 million * 10% * 300 MB = 150TB
* CDN cost, when a CDN serves a video, you are charged for data transferred out of the CDN. Using Amazon's CDN CloudFront for cost estimation, assume 100% of traffic is served from the United States. The average cost per GB is $0.02. For simplicity, we only calculate the cost of video streaming.
* 5 million * 5 videos * 0.3GB * $0.02 = $150,000 per day.

### 15: DESIGN GOOGLE DRIVE

Step 1 - Understand the problem and establish design scope

Designing a Google drive is a big project, so it is important to ask questions to narrow down
the scope.

* Candidate: What are the most important features?
* Interviewer: Upload and download files, file sync, and notifications.
* Candidate: Is this a mobile app, a web app, or both?
* Interviewer: Both.
* Candidate: What are the supported file formats?
* Interviewer: Any file type.
* Candidate: Do files need to be encrypted?
* Interview: Yes, files in the storage must be encrypted.
* Candidate: Is there a file size limit?
* Interview: Yes, files must be 10 GB or smaller.
* Candidate: How many users does the product have?
* Interviewer: 10M DAU.

Focus is on the following features:
* Add files. The easiest way to add a file is to drag and drop a file into Google drive.
* Download files.
* Sync files across multiple devices. When a file is added to one device, it is automatically
synced to other devices.
* See file revisions.
* Share files with your friends, family, and coworkers
* Send a notification when a file is edited, deleted, or shared with you.

Interesting features not wholly covered include:
* Google doc editing and collaboration. Google doc allows multiple people to edit the same document simultaneously. This is out of our design scope. Other than clarifying requirements, it is important to understand non-functional requirements:
* Reliability. Reliability is extremely important for a storage system. Data loss is
unacceptable.
* Fast sync speed. If file sync takes too much time, users will become impatient and
abandon the product.
* Bandwidth usage. If a product takes a lot of unnecessary network bandwidth, users will be unhappy, especially when they are on a mobile data plan.
* Scalability. The system should be able to handle high volumes of traffic.
* High availability. Users should still be able to use the system when some servers are offline, slowed down, or have unexpected network errors.

#### Back of the envelope estimation
* Assume the application has 50 million signed up users and 10 million DAU.
* Users get 10 GB free space.
* Assume users upload 2 files per day. The average file size is 500 KB.
* 1:1 read to write ratio.
* Total space allocated: 50 million * 10 GB = 500 Petabyte
* QPS for upload API: 10 million * 2 uploads / 24 hours / 3600 seconds = ~ 240
* Peak QPS = QPS * 2 = 480
