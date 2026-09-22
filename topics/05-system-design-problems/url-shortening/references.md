# References & Resources — URL Shortening Service

## 📹 Free Videos

* [Design TinyURL — Gaurav Sen](https://www.youtube.com/watch?v=eCLqmPBIEYs) — The go-to URL shortener system design walkthrough covering hashing, base62, and DB schema.
* [URL Shortener System Design — ByteByteGo](https://www.youtube.com/watch?v=ENkh0dMUcOQ) — Animated explainer covering KGS, capacity estimation, and caching layer.
* [Designing a URL Shortening Service — NeetCode](https://www.youtube.com/watch?v=AVztRY77xxA) — Interview-style mock question with architectural decisions and trade-off discussion.
* [System Design Interview: TinyURL — Exponent](https://www.youtube.com/watch?v=KCeE1SrAXlI) — Interviewer + candidate mock walkthrough, showing how to structure the answer.
* [Base64 Encoding Explained — Fireship](https://www.youtube.com/watch?v=8qkxeZmKmOY) — Quick deep-dive into the encoding mechanism behind short key generation.

## 📖 Free Articles & Engineering Blogs

* [Designing a URL Shortener — High Scalability](http://highscalability.com/blog/2009/6/24/designing-url-shortener.html) — Classic blog post on the original TinyURL-style architecture with scaling notes.
* [How TinyURL Works — HowStuffWorks](https://computer.howstuffworks.com/internet/basics/tinyurl.htm) — Plain-English overview of the redirect mechanism and use cases.
* [Key Generation Strategy for URL Shortening — Medium](https://medium.com/@sandeep4.verma/system-design-scalable-url-shortener-service-like-tinyurl-106f30f23a82) — Comparison of MD5 vs base62 vs UUID for short key generation.
* [Distributed Counter for Short ID Generation — DZone](https://dzone.com/articles/url-shortener-design) — How Twitter Snowflake-style distributed counters can power base62 IDs.
* [Consistent Hashing for URL Partitioning — AWS](https://aws.amazon.com/blogs/database/choosing-the-right-dynamodb-partition-key/) — DynamoDB partition key selection strategy (applicable to URL shortener sharding).

## 📄 Key Papers & Deep Dives

* [Source: Grokking the System Design Interview — educative.io](https://www.educative.io/courses/grokking-the-system-design-interview/m2ygV4E81AR) — Full chapter (paid, but the content of this topic.md is derived from the PDF version).
* [Consistent Hashing Original Paper — Karger et al.](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf) — Essential background for the partitioning section of this design.
