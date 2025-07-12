# Write-Through Caching

## Description
Write-through caching is a strategy where data is written to both the cache and the underlying database simultaneously. This ensures data consistency but may introduce write latency.

## Pros
- Data consistency
- Simple to implement

## Cons
- Higher write latency
- Increased backend load

## Use Cases
- Systems requiring strong consistency
