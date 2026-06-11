---
id: web-crawler
title: "Distributed Web Crawler"
category: "Application Architectures"
difficulty: Advanced
readTime: 10
tags:
  - crawler
  - search-engine
  - scraping
  - storage
  - googlebot
---

A web crawler (like Googlebot) is an automated system that browses the World Wide Web to discover, fetch, and index pages for search engines. It starts with a list of seed URLs, fetches the pages, extracts outgoing links, and recursively adds them to a queue. Designing a web crawler at search-engine scale requires handling politeness, deduplication, content processing, and petabyte-scale storage.

## Key Takeaways
* The URL Frontier is the core scheduler queue that decides which URL to crawl next, balancing priority and politeness.
* Politeness dictates that the crawler should not spam a single domain with requests, adhering to robots.txt rules and delaying consecutive hits.
* HTML parsing extracts outgoing URLs and feeds them back into the frontier, while filtering out non-HTML assets (unless requested).
* Content Deduplication uses cryptographic hashes (SimHash or MinHash) to identify near-duplicate web pages, avoiding redundant index storage.
* DNS resolution is a major bottleneck; the crawler must maintain a local DNS cache to avoid exhausting DNS servers.

## Core Concepts
### URL Frontier
A prioritized queue system. It manages priority queues (ranking domains by value/popularity) and politeness queues (ensuring only one thread fetches from a single domain at any given time, utilizing delays).

### Politeness Protocol (Robots.txt)
A standard text file hosted by websites specifying which paths crawlers are allowed or forbidden to access. The crawler must download and cache robots.txt before fetching any page from a domain.

### SimHash (Near-Duplicate Detection)
A locality-sensitive hashing algorithm that generates similar hashes for similar texts. Useful to identify scraped or slightly modified content (like same article with different ads).

### DNS Cache Buffer
DNS queries are blocking network calls. Crawlers resolve hostnames to IP addresses using a highly responsive, distributed local DNS cache to prevent crawler threads from stalling.

## Architectural Trade-offs
Crawling faster increases the coverage and fresh state of the search index but risks triggering security firewalls, breaking target servers, or violating politeness protocols. Storing full HTML pages requires petabytes of disk storage, necessitating compression and early discarding of useless media assets.

## Technology & Tools
* **Apache Nutch**: Extensible, highly modular, and distributed open-source web crawler framework built on top of Hadoop.
* **Scrapy**: Fast, high-level screen scraping and web crawling framework for Python, ideal for smaller-scale directed crawls.
* **HBase / HDFS**: Distributed storage layers used to store raw crawled HTML page repositories and indexing metadata.

## Real-Life Examples & FAANG Case Studies
### Googlebot (Google Search Crawler)
Googlebot crawls billions of web pages daily, feeding Google's indexing pipeline and search results.

#### Bottlenecks
* DNS Resolution Saturation: Resolving hostnames for billions of unique domains chokes external DNS servers. Googlebot uses private distributed DNS servers and aggressive local caching.
* Crawling Dynamic JS Pages: Modern pages use React/Angular. Simply downloading HTML is not enough. Googlebot must run headless Chrome rendering engines, which consume 100x more CPU and RAM than plain HTML parsers.

#### Corner Cases
* Spider Traps (Infinite Loops): Buggy or malicious sites generate infinite dynamic subdirectories (e.g., site.com/page/1/2/3/...). Googlebot uses path-depth limiters, pattern matching, and loop detectors to discard these URLs.
* Rapid Content Changes (News Sites): Breaking news sites must be crawled every few minutes, while static blogs need updates monthly. Googlebot dynamically recalculates crawl frequency for each URL based on historical change frequency.

## Interview Cheat Sheet & Tips
* Emphasize the design of the URL Frontier: explain how it uses separate queues for Priority (importance) and Politeness (server safety).
* Explain DNS resolution bottlenecks and how a local custom DNS cache resolves this.
* Discuss crawler footprint: specify how to set a distinct User-Agent header (e.g., Googlebot) and provide a webmaster contact URL.

