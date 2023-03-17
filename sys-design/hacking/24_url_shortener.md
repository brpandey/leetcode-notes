### 24. Design a URL Shortener

Url Shortener simplifies and transforms a long URL to a shorter more compact url with just a domain name 
(e.g. tinyurl and bitly) and a compact URL path id token

Long Url
> https://www.walmart.com/ip/Acer-Aspire-5-15-6-Full-HD-IPS-Display-11th-Gen-Intel-Core-i7-1165G7-12GB-DDR4-512GB-NVMe-SSD-Pure-Silver-Windows-11-Home-A515-56-79N0/2809390326

Short Url
> http://tinyurl.com/xd32ab98

> http://goo.gl/2zabckd9 

In addition to a more compact form, since the generated url goes through an intermediary shortener service (tinyurl, bitly), url tracking and
statistics are possible e.g.
* track analytics
* audience clicks
* referrals
* ad campaigns

1. Clarify problem and scope use cases
   1. Use Cases
      1. Create shortened url - Users enter long url and receive short url
      2. When accessed, short url redirects to orignal url ([redirect codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) 301 Permanent, 307 Temporary)
      3. Short url has configurable lifespan, with a deafult setting
      4. Url tracking possible, can be done in either real-time mode or offline/delayed processing
   2. Requirements
      1. High availability, if one server is down, that shouldn't stop the service from functioning properly
      2. Highly performant, with minimum latency
      3. Generated short URL must be unique and not have been generated before
   3. Clarifying Questions
      1. How many anticipated users of service?
      2. What should the default lifespan for the generated url be? A few hours? A few days? A few months? Years?
      3. Is anonymous access important?
      4. Should the system maintain analytics for users and links?
      5. Should the service allow custom links
      6. Who is the intended end user? Should ad campaign/referral linking features be added?
      7. What should the max length of the shortened url generated token be?


2. Data model 
      1. ![Short Url, User, Click Metric](https://github.com/brpandey/markdown-notes/blob/main/sys-design/hacking/imgs/0055.jpg)

Approaches - Generating short url generated id 
* Snowflake
* Cryptographic Hash
* GUID generator

3. Back-of-the-envelope Calculations

4. High-level design
    1. This design doesn't handle scalability
        1. ![](https://github.com/brpandey/markdown-notes/blob/main/sys-design/hacking/imgs/0056.jpg)
    2. This design separates reads from writes as they both scale independently, 200:1 r:w
        1. ![](https://github.com/brpandey/markdown-notes/blob/main/sys-design/hacking/imgs/0057.jpg)
6. Detailed design - Focusing on analytics service
    1. ![](https://github.com/brpandey/markdown-notes/blob/main/sys-design/hacking/imgs/0058.jpg)
    2. ![](https://github.com/brpandey/markdown-notes/blob/main/sys-design/hacking/imgs/0059.jpg)
