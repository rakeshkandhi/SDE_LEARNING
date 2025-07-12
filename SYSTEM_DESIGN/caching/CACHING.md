# Caching in System Design

## What is Caching?
Caching is a technique used to store frequently accessed data in a temporary storage area (cache) to reduce access time, improve performance, and decrease load on the underlying data source.

## Why Use Caching?
- **Performance:** Reduces latency and speeds up data retrieval.
- **Scalability:** Decreases load on databases and backend services.
- **Cost Efficiency:** Minimizes expensive operations and resource usage.

## Types of Caching
1. **Client-Side Caching:** Data is cached on the user's device (browser, mobile app).
2. **Server-Side Caching:** Data is cached on the server, often in memory (e.g., Redis, Memcached).
3. **CDN Caching:** Content Delivery Networks cache static assets geographically closer to users.
4. **Database Caching:** Query results or computed data are cached to avoid repeated expensive queries.

## Caching Strategies
- [**Write-Through Cache:**](write_through.md) Data is written to cache and database simultaneously.
- [**Write-Back (Write-Behind) Cache:**](write_back.md) Data is written to cache first, then asynchronously to the database.
- [**Read-Through Cache:**](read_through.md) Application reads from cache; if not found, fetches from database and updates cache.
- [**Cache Aside (Lazy Loading):**](cache_aside.md) Application loads data into cache only when needed.
- [**Cache Invalidation Techniques**](invalidation.md) Approach to managing stale data in cache.

## Cache Invalidation
Ensuring cache consistency is critical. Common strategies:
- **Time-to-Live (TTL):** Cached data expires after a set time.
- **Explicit Invalidation:** Application removes/updates cache when data changes.
- **LRU (Least Recently Used):** Removes least recently accessed items when cache is full.

## Common Tools
- **Redis:** In-memory key-value store, supports advanced data structures.
- **Memcached:** Simple, high-performance distributed memory caching system.
- **CDNs:** Akamai, Cloudflare, AWS CloudFront for static asset caching.

## Example Use Cases
- Caching user session data for fast authentication.
- Caching product catalog in e-commerce to reduce database load.
- Caching API responses to improve throughput and reduce latency.

## Best Practices
- Set appropriate TTLs to balance freshness and performance.
- Monitor cache hit/miss rates and adjust strategy as needed.
- Avoid caching sensitive or highly dynamic data.
- Use distributed caching for scalability in large systems.

---
For more details, see the system design introduction in [introduction.md](../introduction.md).
