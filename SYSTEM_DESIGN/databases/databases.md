# Database Design in System Design 🗄️

## Overview
Databases are the backbone of most applications, responsible for storing, retrieving, and managing data efficiently. Choosing the right database and designing it properly is crucial for system performance, scalability, and reliability.

## Table of Contents
- [Database Types](#database-types)
- [SQL vs NoSQL](#sql-vs-nosql)
- [Database Scaling Strategies](#database-scaling-strategies)
- [Data Modeling](#data-modeling)
- [Consistency Models](#consistency-models)
- [Database Selection Criteria](#database-selection-criteria)
- [Performance Optimization](#performance-optimization)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)

## Database Types

### 1. Relational Databases (SQL)
Structured data with predefined schema and ACID properties.

**Popular Options:**
- **PostgreSQL**: Advanced features, JSON support, extensible
- **MySQL**: Fast, reliable, widely adopted
- **Oracle**: Enterprise features, high performance
- **SQL Server**: Microsoft ecosystem, business intelligence

**Characteristics:**
- ACID compliance (Atomicity, Consistency, Isolation, Durability)
- Strong consistency
- Complex queries with JOINs
- Mature ecosystem and tooling

**Use Cases:**
- Financial systems
- E-commerce transactions
- CRM systems
- Any application requiring strong consistency

### 2. NoSQL Databases
Flexible schema designed for specific data models and use cases.

#### Document Databases
Store data as documents (JSON, BSON, XML).

**Examples:** MongoDB, CouchDB, Amazon DocumentDB

```json
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "preferences": {
    "theme": "dark",
    "notifications": true
  },
  "tags": ["premium", "early-adopter"]
}
```

**Use Cases:**
- Content management
- User profiles
- Product catalogs
- Real-time analytics

#### Key-Value Stores
Simple key-value pairs for fast lookups.

**Examples:** Redis, DynamoDB, Riak

```python
# Redis example
redis.set("user:123:session", "abc123xyz")
redis.set("user:123:cart", json.dumps(cart_items))
redis.expire("user:123:session", 3600)  # 1 hour TTL
```

**Use Cases:**
- Caching
- Session storage
- Shopping carts
- Real-time recommendations

#### Column-Family
Data stored in column families, optimized for write-heavy workloads.

**Examples:** Cassandra, HBase, Amazon SimpleDB

```cql
-- Cassandra example
CREATE TABLE user_activity (
    user_id UUID,
    timestamp TIMESTAMP,
    activity_type TEXT,
    details MAP<TEXT, TEXT>,
    PRIMARY KEY (user_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

**Use Cases:**
- Time-series data
- IoT sensor data
- Activity logs
- Analytics platforms

#### Graph Databases
Optimized for storing and querying relationships.

**Examples:** Neo4j, Amazon Neptune, ArangoDB

```cypher
// Neo4j example - Find friends of friends
MATCH (user:Person {name: "Alice"})-[:FRIEND]->(friend)-[:FRIEND]->(fof)
WHERE NOT (user)-[:FRIEND]->(fof) AND user <> fof
RETURN fof.name, COUNT(*) as mutual_friends
ORDER BY mutual_friends DESC
```

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

### 3. NewSQL Databases
Combine SQL benefits with NoSQL scalability.

**Examples:** CockroachDB, TiDB, VoltDB, Google Spanner

**Features:**
- ACID compliance
- Horizontal scalability
- SQL interface
- Distributed architecture

## SQL vs NoSQL

### When to Choose SQL

**Advantages:**
- ACID compliance
- Complex queries and joins
- Mature ecosystem
- Standardized query language
- Strong consistency

**Choose SQL When:**
- Complex relationships between data
- Need for transactions
- Regulatory compliance requirements
- Team expertise in SQL
- Structured, predictable data

### When to Choose NoSQL

**Advantages:**
- Horizontal scalability
- Flexible schema
- High performance for specific use cases
- Better handling of unstructured data
- Cloud-native design

**Choose NoSQL When:**
- Massive scale requirements
- Rapid development and iteration
- Unstructured or semi-structured data
- Geographic distribution
- Specific performance requirements

## Database Scaling Strategies

### 1. Read Scaling

#### Read Replicas
```
Write Requests → Master Database
Read Requests → [Replica1, Replica2, Replica3]
```

**Implementation:**
```python
class DatabaseRouter:
    def __init__(self, master_db, read_replicas):
        self.master_db = master_db
        self.read_replicas = read_replicas
        self.replica_index = 0

    def write(self, query, params):
        return self.master_db.execute(query, params)

    def read(self, query, params):
        replica = self.read_replicas[self.replica_index]
        self.replica_index = (self.replica_index + 1) % len(self.read_replicas)
        return replica.execute(query, params)
```

### 2. Write Scaling

#### Database Sharding
Partition data across multiple databases.

```python
def get_shard_key(user_id):
    """Determine which shard to use for a user"""
    return hash(user_id) % NUM_SHARDS

def get_user_shard(user_id):
    shard_id = get_shard_key(user_id)
    return f"user_db_shard_{shard_id}"

# Usage
user_id = "user123"
shard = get_user_shard(user_id)
db = get_database_connection(shard)
user = db.query("SELECT * FROM users WHERE id = ?", user_id)
```

**Sharding Strategies:**
- **Range-based**: Partition by value ranges
- **Hash-based**: Use hash function for distribution
- **Directory-based**: Lookup service for shard location
- **Geographic**: Partition by location

#### Federation (Functional Partitioning)
Split databases by feature/service.

```
User Service → User Database
Order Service → Order Database
Product Service → Product Database
Payment Service → Payment Database
```

### 3. Hybrid Approaches

#### CQRS (Command Query Responsibility Segregation)
Separate read and write models.

```
Write Commands → Write Database (Optimized for writes)
Read Queries → Read Database (Optimized for reads, denormalized)
```

## Data Modeling

### Relational Data Modeling

#### Normalization
Organize data to reduce redundancy.

**First Normal Form (1NF):**
- Atomic values in each cell
- No repeating groups

**Second Normal Form (2NF):**
- 1NF + No partial dependencies
- Non-key attributes depend on entire primary key

**Third Normal Form (3NF):**
- 2NF + No transitive dependencies
- Non-key attributes don't depend on other non-key attributes

#### Example: E-commerce Schema
```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Products table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    category_id INTEGER REFERENCES categories(id)
);

-- Orders table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total_amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Order items table
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL,
    price DECIMAL(10,2) NOT NULL
);
```

### NoSQL Data Modeling

#### Document Database Modeling
Embed related data in documents.

```json
{
  "_id": "order_123",
  "user": {
    "id": "user_456",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "items": [
    {
      "product_id": "prod_789",
      "name": "Laptop",
      "price": 999.99,
      "quantity": 1
    }
  ],
  "total": 999.99,
  "status": "shipped",
  "created_at": "2023-01-15T10:30:00Z"
}
```

#### Key-Value Modeling
Design keys for efficient access patterns.

```python
# User session data
"session:abc123" → {"user_id": "123", "expires": "2023-01-15T12:00:00Z"}

# Shopping cart
"cart:user:123" → {"items": [{"id": "prod1", "qty": 2}], "total": 49.98}

# User preferences
"prefs:user:123" → {"theme": "dark", "lang": "en", "notifications": true}
```

## Consistency Models

### Strong Consistency
All nodes see the same data simultaneously.

**Characteristics:**
- Immediate consistency
- Higher latency
- Lower availability during partitions

**Use Cases:**
- Financial transactions
- Inventory management
- User authentication

### Eventual Consistency
System will become consistent over time.

**Characteristics:**
- Lower latency
- Higher availability
- Temporary inconsistencies possible

**Use Cases:**
- Social media feeds
- Product catalogs
- Analytics data

### Consistency Patterns

#### Read-after-Write Consistency
Users see their own writes immediately.

```python
def update_user_profile(user_id, data):
    # Write to master
    master_db.update_user(user_id, data)

    # Invalidate cache
    cache.delete(f"user:{user_id}")

    # For immediate read, use master
    return master_db.get_user(user_id)
```

#### Session Consistency
Consistency within a user session.

```python
class SessionConsistentDB:
    def __init__(self, master_db, replica_dbs):
        self.master_db = master_db
        self.replica_dbs = replica_dbs
        self.session_writes = set()

    def write(self, key, value):
        self.master_db.write(key, value)
        self.session_writes.add(key)

    def read(self, key):
        if key in self.session_writes:
            return self.master_db.read(key)  # Read from master
        else:
            return random.choice(self.replica_dbs).read(key)  # Read from replica
```

## Database Selection Criteria

### Performance Requirements
- **Read vs Write Heavy**: Different databases optimize for different patterns
- **Latency Requirements**: In-memory vs disk-based storage
- **Throughput Needs**: Concurrent connections and operations per second

### Scalability Needs
- **Data Volume**: Current and projected data size
- **User Growth**: Expected number of concurrent users
- **Geographic Distribution**: Global vs regional deployment

### Consistency Requirements
- **ACID Compliance**: Financial vs social media applications
- **Real-time vs Eventual**: Immediate vs delayed consistency acceptable

### Operational Considerations
- **Team Expertise**: Available skills and learning curve
- **Operational Complexity**: Maintenance, backup, monitoring
- **Cost**: Licensing, infrastructure, operational overhead

## Performance Optimization

### Indexing Strategies

#### SQL Database Indexing
```sql
-- Primary key index (automatic)
CREATE TABLE users (id SERIAL PRIMARY KEY, email VARCHAR(255));

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Full-text search index
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name || ' ' || description));
```

#### NoSQL Indexing
```javascript
// MongoDB indexing
db.users.createIndex({ "email": 1 }, { unique: true })
db.orders.createIndex({ "user_id": 1, "created_at": -1 })
db.products.createIndex({ "name": "text", "description": "text" })

// Compound index for queries
db.events.createIndex({ "user_id": 1, "timestamp": -1, "event_type": 1 })
```

### Query Optimization

#### SQL Query Optimization
```sql
-- Use EXPLAIN to analyze query plans
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2023-01-01'
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5;

-- Optimize with proper indexing
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

#### NoSQL Query Optimization
```javascript
// MongoDB aggregation pipeline optimization
db.orders.aggregate([
  { $match: { created_at: { $gte: new Date('2023-01-01') } } },
  { $group: { _id: "$user_id", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])

// Use appropriate indexes
db.orders.createIndex({ "created_at": 1 })
db.orders.createIndex({ "user_id": 1, "amount": 1 })
```

### Connection Pooling
```python
# Database connection pooling
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600
)
```

## Real-World Examples

### Instagram
**Challenge**: Store billions of photos and user interactions

**Solution:**
- **PostgreSQL**: User data, relationships (sharded)
- **Cassandra**: Photo metadata, activity feeds
- **Redis**: Caching, session storage
- **S3**: Photo storage

### Netflix
**Challenge**: Personalized recommendations for 200M+ users

**Solution:**
- **Cassandra**: Viewing history, user preferences
- **MySQL**: Billing, account information
- **Elasticsearch**: Search and discovery
- **S3**: Content storage

### Uber
**Challenge**: Real-time location tracking and matching

**Solution:**
- **MySQL**: User accounts, trip data
- **Redis**: Real-time location data
- **Cassandra**: Trip history, analytics
- **Elasticsearch**: Location search

## Best Practices

### Design Principles
1. **Understand Your Data**: Structure, relationships, access patterns
2. **Plan for Scale**: Design for 10x current requirements
3. **Choose the Right Tool**: Different databases for different use cases
4. **Optimize for Common Queries**: Index frequently accessed data
5. **Monitor Performance**: Track key metrics and optimize bottlenecks

### Implementation Guidelines
1. **Start Simple**: Begin with a single database, scale as needed
2. **Measure Before Optimizing**: Profile queries and identify bottlenecks
3. **Use Appropriate Data Types**: Choose efficient storage formats
4. **Implement Proper Backup**: Regular backups and disaster recovery
5. **Security First**: Encryption, access controls, audit logs

### Common Pitfalls
- Over-normalization in SQL databases
- Under-indexing or over-indexing
- Ignoring query performance
- Not planning for data growth
- Choosing technology based on hype rather than requirements

---

**Next Steps:**
- Learn about [Caching Strategies](../caching/CACHING.md)
- Explore [Load Balancing](../load_balancing/load_balancing.md)
- Study [Scalability Patterns](../scalability/scalability.md)
