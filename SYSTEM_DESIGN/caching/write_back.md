# Write-Back (Write-Behind) Caching

## Description
Write-back caching writes data to the cache first and asynchronously updates the database. This improves write performance but risks data loss if the cache fails before syncing.

## Pros
- Fast writes
- Reduced backend load

## Cons
- Risk of data loss
- Complexity in failure handling

## Use Cases
- High-throughput systems where eventual consistency is acceptable
