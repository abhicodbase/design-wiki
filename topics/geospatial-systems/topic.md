---
id: geospatial-systems
title: "Geospatial Indexing & Systems"
category: "Application Architectures"
difficulty: Advanced
readTime: 11
tags:
  - geospatial
  - geohash
  - quadtree
  - s2
  - uber
  - yelp
---

Geospatial systems process location-based data (latitude and longitude) to answer queries like 'find nearby drivers' or 'list restaurants within 2 miles'. Standard database indexes (B-Trees) cannot handle multi-dimensional queries (filtering on both lat and lon simultaneously) efficiently. Geospatial indexes solve this by converting 2D coordinates into a 1D format, or partitioning space hierarchically.

## Key Takeaways
* Geohash divides the Earth's surface into grid cells, representing each cell as a Base32 string. Longer strings indicate smaller, more precise cells.
* Quadtree is a tree structure where each internal node has exactly four children, recursively dividing a 2D space into quadrants based on density.
* Google S2 maps the Earth onto a cube, then uses Hilbert Curves to project 2D coordinates into a 1D index (64-bit integer), preserving spatial locality.
* Write-heavy workloads (e.g., live vehicle tracking) require in-memory spatial indexes (like Redis Geo), while read-heavy static stores use databases (PostGIS).
* Spatial queries generally search a bounding box or radial distance, filtering out far-away points using approximate cell matches first.

## Core Concepts
### Geohash
Interleaves latitude and longitude bits to create a string (e.g., 'dr5reg'). Nearby points often share the same string prefix, enabling quick database prefix queries. Drawback: edge case boundary errors where close points have different prefixes.

### Quadtree
A dynamic in-memory data structure. If a quadrant exceeds a capacity limit (e.g., 100 points), it splits into four sub-quadrants. Highly effective for non-uniform data (dense cities vs. sparse rural areas).

### Google S2 Geometry
Uses Hilbert Curves (space-filling curves) to index coordinates. Hilbert curves guarantee that points close in 2D space remain close in 1D sorted order, resolving the boundary limitations of standard Geohash.

## Architectural Trade-offs
In-memory Quadtrees are extremely fast for live, high-frequency location updates but are difficult to distribute and replicate across servers. Geohashing is simple to store in standard databases, but requires query logic adjustments to check adjacent cells to avoid edge boundary misses.

## Technology & Tools
* **Redis Geo**: Uses sorted sets and Geohashes to store coordinate points in memory, offering rapid updates and radial searches.
* **PostGIS**: Spatial database extender for PostgreSQL, supporting complex GIS queries, polygon intersection, and R-Tree indexing.
* **Elasticsearch Geo**: Supports geospatial queries on large document indexes, allowing users to filter search results by location.

## Real-Life Examples & FAANG Case Studies
### Uber Ride Matching (DISCO)
Uber's Demand and Supply Engine (DISCO) tracks real-time driver locations and matches them with riders using Google S2 cells, managing millions of updates per minute.

#### Bottlenecks
* High Write Throughput: Drivers send GPS updates every 4 seconds. Storing this directly in a disk-based database would crash it. Uber buffers locations in an in-memory ring buffer mapped to S2 cells.

#### Corner Cases
* Unequal Density Splits: In dense urban spots (like Manhattan during rush hour), thousands of drivers congregate. S2 allows dynamic cell levels, keeping cell sizes small for dense areas and large for rural regions to prevent computational overload.
* Disconnected Drivers: If a driver enters a tunnel, their GPS connection drops. DISCO must predict their route or ignore stale records to avoid matching riders with 'ghost' drivers.

### Yelp / Google Maps 'Nearby Places'
Yelp uses spatial indexing to display restaurants, bars, and businesses within a user's search radius, combining geographic filters with rating and relevance criteria.

#### Bottlenecks
* Distance Matrix Calculations: When a user requests 'restaurants near me sorted by rating', Yelp must calculate the distance between the user and hundreds of candidate businesses, which is CPU-expensive.

#### Corner Cases
* Cross-Boundary Queries: A user standing at the exact intersection of four Geohash cells will miss restaurants just 10 meters away in another cell unless the system queries the target cell plus all 8 adjacent cells.
* High Density Scaling: In cities like Tokyo, a search might find 10,000 restaurants in a 500m radius. The system must apply pagination and early termination in spatial tree traversal to maintain sub-100ms response times.

## Interview Cheat Sheet & Tips
* Clearly articulate why standard SQL indexes (B-Trees) fail: they can only index one dimension (e.g., latitude) efficiently, forcing a slow scan on the other.
* Contrast Geohash (static grid, simple database queries) with Quadtree (dynamic memory structure, excellent for variable density).
* Explain how to handle the boundary problem: query the target grid cell and all 8 surrounding cells.

