# Read-Through Caching

## Description
Read-through caching allows the application to read from the cache. If the data is not present, it fetches from the database and updates the cache automatically.

## Pros
- Transparent to application
- Improved read performance

## Cons
- Cache miss penalty
- Complexity in cache management

## Use Cases
- Frequently read data with occasional updates
