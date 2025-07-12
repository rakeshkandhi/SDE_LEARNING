# System Design Case Studies 📚

## Overview
Real-world case studies provide invaluable insights into how large-scale systems are designed, implemented, and evolved. This section explores detailed examples of system design decisions, trade-offs, and lessons learned from successful companies.

## Table of Contents
- [Case Study Framework](#case-study-framework)
- [WhatsApp: Messaging at Scale](#whatsapp-messaging-at-scale)
- [Netflix: Video Streaming Platform](#netflix-video-streaming-platform)
- [Uber: Real-time Ride Matching](#uber-real-time-ride-matching)
- [Instagram: Photo Sharing Platform](#instagram-photo-sharing-platform)
- [Twitter: Social Media Timeline](#twitter-social-media-timeline)
- [Zoom: Video Conferencing](#zoom-video-conferencing)
- [Design Patterns Summary](#design-patterns-summary)
- [Lessons Learned](#lessons-learned)

## Case Study Framework

### Analysis Structure
1. **Problem Statement**: What challenge needs to be solved?
2. **Requirements**: Functional and non-functional requirements
3. **Scale Estimation**: Users, data, and traffic calculations
4. **High-Level Architecture**: System components and interactions
5. **Detailed Design**: Deep dive into critical components
6. **Data Storage**: Database design and data flow
7. **Scalability**: How the system handles growth
8. **Reliability**: Fault tolerance and disaster recovery
9. **Security**: Authentication, authorization, and data protection
10. **Monitoring**: Observability and performance tracking
11. **Trade-offs**: Design decisions and their implications
12. **Evolution**: How the system evolved over time

### Key Metrics to Consider
- **Users**: Daily/Monthly Active Users (DAU/MAU)
- **Data**: Storage requirements and growth rate
- **Traffic**: Requests per second (RPS), peak vs average
- **Latency**: Response time requirements
- **Availability**: Uptime requirements (99.9%, 99.99%, etc.)
- **Consistency**: Data consistency requirements

## WhatsApp: Messaging at Scale

### Problem Statement
Design a messaging system that can handle billions of users sending messages in real-time with high reliability and low latency.

### Requirements

#### Functional Requirements
- Send and receive text messages
- Group messaging
- Message delivery status (sent, delivered, read)
- Online/offline status
- Message history
- Media sharing (images, videos, documents)

#### Non-Functional Requirements
- **Scale**: 2 billion users, 100 billion messages/day
- **Latency**: < 100ms for message delivery
- **Availability**: 99.99% uptime
- **Consistency**: Eventual consistency acceptable
- **Security**: End-to-end encryption

### Scale Estimation
```
Users: 2 billion total, 500 million DAU
Messages: 100 billion/day = 1.16 million/second
Peak traffic: 3x average = 3.5 million messages/second
Storage: 100 billion messages × 100 bytes = 10 TB/day
Bandwidth: 3.5M messages/sec × 100 bytes = 350 MB/s
```

### High-Level Architecture
```
Client Apps → Load Balancer → API Gateway → Message Service
                                              ↓
WebSocket Servers ← → Message Queue ← → Database Cluster
       ↓                    ↓                ↓
Push Notification    Message Router    Message Storage
   Service              Service         (Sharded)
```

### Detailed Design

#### Message Flow
```python
class MessageService:
    def __init__(self):
        self.message_queue = MessageQueue()
        self.user_connections = UserConnectionManager()
        self.message_storage = MessageStorage()
        self.notification_service = NotificationService()

    async def send_message(self, sender_id, recipient_id, message_content):
        # 1. Validate message
        if not self.validate_message(message_content):
            raise ValidationError("Invalid message content")

        # 2. Create message object
        message = Message(
            id=generate_message_id(),
            sender_id=sender_id,
            recipient_id=recipient_id,
            content=message_content,
            timestamp=time.time(),
            status='sent'
        )

        # 3. Store message
        await self.message_storage.store(message)

        # 4. Check if recipient is online
        if self.user_connections.is_online(recipient_id):
            # Send via WebSocket
            await self.user_connections.send_message(recipient_id, message)
            message.status = 'delivered'
            await self.message_storage.update_status(message.id, 'delivered')
        else:
            # Queue for later delivery
            await self.message_queue.enqueue(message)
            # Send push notification
            await self.notification_service.send_push(recipient_id, message)

        return message
```

#### Database Design
```sql
-- Users table
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    phone_number VARCHAR(20) UNIQUE,
    username VARCHAR(50),
    last_seen TIMESTAMP,
    status VARCHAR(20),
    created_at TIMESTAMP
);

-- Messages table (sharded by user_id)
CREATE TABLE messages (
    message_id BIGINT PRIMARY KEY,
    sender_id BIGINT,
    recipient_id BIGINT,
    content TEXT,
    message_type VARCHAR(20), -- text, image, video, etc.
    status VARCHAR(20), -- sent, delivered, read
    created_at TIMESTAMP,
    INDEX idx_recipient_time (recipient_id, created_at),
    INDEX idx_sender_time (sender_id, created_at)
);

-- Group messages
CREATE TABLE group_messages (
    message_id BIGINT PRIMARY KEY,
    group_id BIGINT,
    sender_id BIGINT,
    content TEXT,
    created_at TIMESTAMP
);

-- Group members
CREATE TABLE group_members (
    group_id BIGINT,
    user_id BIGINT,
    role VARCHAR(20), -- admin, member
    joined_at TIMESTAMP,
    PRIMARY KEY (group_id, user_id)
);
```

#### Sharding Strategy
```python
class MessageSharding:
    def __init__(self, num_shards=1000):
        self.num_shards = num_shards

    def get_shard(self, user_id):
        """Shard messages by user_id for better locality"""
        return user_id % self.num_shards

    def get_message_db(self, user_id):
        shard_id = self.get_shard(user_id)
        return f"messages_db_shard_{shard_id}"

    def store_message(self, message):
        # Store in both sender and recipient shards for fast retrieval
        sender_db = self.get_message_db(message.sender_id)
        recipient_db = self.get_message_db(message.recipient_id)

        # Store in sender's shard
        self.store_in_shard(sender_db, message)

        # Store in recipient's shard if different
        if sender_db != recipient_db:
            self.store_in_shard(recipient_db, message)
```

### Key Design Decisions

#### 1. WebSocket vs HTTP Polling
**Decision**: WebSocket for real-time messaging
**Rationale**: Lower latency, reduced server load, better user experience
**Trade-off**: More complex connection management

#### 2. Message Storage Strategy
**Decision**: Dual storage (sender and recipient shards)
**Rationale**: Fast message retrieval for both parties
**Trade-off**: 2x storage cost, eventual consistency challenges

#### 3. Delivery Guarantees
**Decision**: At-least-once delivery with deduplication
**Rationale**: Ensures messages are not lost
**Trade-off**: Potential duplicate messages, requires deduplication logic

### Scalability Solutions

#### Connection Management
```python
class ConnectionManager:
    def __init__(self):
        self.connections = {}  # user_id -> connection
        self.user_servers = {}  # user_id -> server_id
        self.server_registry = ServiceRegistry()

    def add_connection(self, user_id, connection, server_id):
        self.connections[user_id] = connection
        self.user_servers[user_id] = server_id

        # Register with service discovery
        self.server_registry.register_user(user_id, server_id)

    def route_message(self, recipient_id, message):
        if recipient_id in self.connections:
            # User connected to this server
            return self.connections[recipient_id].send(message)
        else:
            # Find which server has the user
            server_id = self.server_registry.find_user_server(recipient_id)
            if server_id:
                # Forward to appropriate server
                return self.forward_to_server(server_id, recipient_id, message)
            else:
                # User offline, queue message
                return self.queue_message(recipient_id, message)
```

#### Message Queue for Offline Users
```python
class OfflineMessageQueue:
    def __init__(self, redis_client):
        self.redis = redis_client

    def queue_message(self, user_id, message):
        queue_key = f"offline_messages:{user_id}"
        self.redis.lpush(queue_key, json.dumps(message.to_dict()))
        self.redis.expire(queue_key, 86400 * 7)  # 7 days TTL

    def get_offline_messages(self, user_id):
        queue_key = f"offline_messages:{user_id}"
        messages = self.redis.lrange(queue_key, 0, -1)
        self.redis.delete(queue_key)  # Clear after retrieval

        return [json.loads(msg) for msg in messages]
```

### Challenges and Solutions

#### Challenge 1: Message Ordering
**Problem**: Ensuring messages are delivered in the correct order
**Solution**:
- Use message sequence numbers
- Client-side ordering based on timestamps
- Vector clocks for group messages

#### Challenge 2: Group Message Scalability
**Problem**: Delivering messages to large groups efficiently
**Solution**:
- Fan-out on write for small groups (< 100 members)
- Fan-out on read for large groups (> 100 members)
- Hybrid approach based on group size

#### Challenge 3: End-to-End Encryption
**Problem**: Encrypting messages while maintaining performance
**Solution**:
- Signal Protocol for key exchange
- Client-side encryption/decryption
- Server stores encrypted messages only

### Lessons Learned
1. **Start Simple**: WhatsApp initially used a simple architecture and scaled incrementally
2. **Erlang Choice**: Erlang's actor model was perfect for handling millions of concurrent connections
3. **Minimal Features**: Focus on core messaging functionality first
4. **Operational Excellence**: Invest heavily in monitoring and automation
5. **Team Size**: Small team (50 engineers) can build and maintain massive scale

## Netflix: Video Streaming Platform

### Problem Statement
Design a video streaming platform that can serve millions of users worldwide with high-quality video content and personalized recommendations.

### Requirements

#### Functional Requirements
- Stream videos in multiple qualities (480p, 720p, 1080p, 4K)
- User authentication and profiles
- Content catalog and search
- Personalized recommendations
- Content upload and encoding
- Offline viewing (mobile)
- Multiple device support

#### Non-Functional Requirements
- **Scale**: 200+ million subscribers, 1 billion hours watched/day
- **Availability**: 99.99% uptime
- **Latency**: < 2 seconds to start streaming
- **Bandwidth**: Adaptive bitrate streaming
- **Global**: Serve content worldwide with low latency

### Scale Estimation
```
Users: 200 million subscribers, 100 million concurrent viewers
Content: 15,000 titles, 4 hours average length
Storage: 15,000 × 4 hours × 5 GB/hour × 4 qualities = 1.2 PB
Bandwidth: 100M concurrent × 5 Mbps average = 500 Tbps
CDN: 1000+ edge locations worldwide
```

### High-Level Architecture
```
Client Apps → CDN (Edge Locations) → Origin Servers
                ↓                        ↓
        Video Content Cache        Video Processing Pipeline
                                          ↓
API Gateway → Microservices → Databases → Content Storage
    ↓              ↓              ↓           ↓
User Service   Recommendation  Metadata    Video Files
Auth Service   Service         Database    (S3/GCS)
```

### Detailed Design

#### Video Processing Pipeline
```python
class VideoProcessingPipeline:
    def __init__(self):
        self.encoding_service = VideoEncodingService()
        self.storage_service = CloudStorageService()
        self.cdn_service = CDNService()
        self.metadata_service = MetadataService()

    async def process_video(self, video_file, metadata):
        # 1. Validate and analyze video
        video_info = await self.analyze_video(video_file)

        # 2. Generate multiple quality versions
        encoding_jobs = []
        for quality in ['480p', '720p', '1080p', '4K']:
            if video_info.supports_quality(quality):
                job = self.encoding_service.create_job(
                    input_file=video_file,
                    output_quality=quality,
                    codec='H.264',
                    container='MP4'
                )
                encoding_jobs.append(job)

        # 3. Process encodings in parallel
        encoded_videos = await asyncio.gather(*[
            self.encoding_service.process(job) for job in encoding_jobs
        ])

        # 4. Upload to storage
        storage_urls = {}
        for quality, encoded_video in zip(['480p', '720p', '1080p', '4K'], encoded_videos):
            url = await self.storage_service.upload(
                file=encoded_video,
                path=f"videos/{metadata.content_id}/{quality}.mp4"
            )
            storage_urls[quality] = url

        # 5. Distribute to CDN
        await self.cdn_service.distribute(storage_urls)

        # 6. Update metadata
        await self.metadata_service.update_video_info(
            content_id=metadata.content_id,
            qualities=list(storage_urls.keys()),
            duration=video_info.duration,
            storage_urls=storage_urls
        )

        return storage_urls
```

#### Adaptive Bitrate Streaming
```python
class AdaptiveBitrateStreaming:
    def __init__(self):
        self.quality_levels = {
            '480p': {'bitrate': 1000, 'resolution': '854x480'},
            '720p': {'bitrate': 2500, 'resolution': '1280x720'},
            '1080p': {'bitrate': 5000, 'resolution': '1920x1080'},
            '4K': {'bitrate': 15000, 'resolution': '3840x2160'}
        }

    def generate_manifest(self, content_id, available_qualities):
        """Generate HLS manifest for adaptive streaming"""
        manifest = "#EXTM3U\n#EXT-X-VERSION:3\n"

        for quality in available_qualities:
            if quality in self.quality_levels:
                bitrate = self.quality_levels[quality]['bitrate']
                resolution = self.quality_levels[quality]['resolution']

                manifest += f"#EXT-X-STREAM-INF:BANDWIDTH={bitrate}000,RESOLUTION={resolution}\n"
                manifest += f"{content_id}/{quality}/playlist.m3u8\n"

        return manifest

    def select_quality(self, bandwidth, device_capabilities):
        """Select appropriate quality based on network and device"""
        # Start with highest quality device supports
        max_quality = device_capabilities.get('max_resolution', '4K')

        # Adjust based on available bandwidth
        if bandwidth < 1500:
            return '480p'
        elif bandwidth < 3000:
            return '720p'
        elif bandwidth < 7000:
            return '1080p'
        else:
            return min(max_quality, '4K')
```

#### Recommendation System
```python
class RecommendationEngine:
    def __init__(self):
        self.collaborative_filter = CollaborativeFiltering()
        self.content_filter = ContentBasedFiltering()
        self.popularity_ranker = PopularityRanker()
        self.ml_model = MachineLearningModel()

    async def get_recommendations(self, user_id, context):
        # Get user viewing history
        viewing_history = await self.get_user_history(user_id)

        # Generate recommendations from different algorithms
        collab_recs = await self.collaborative_filter.recommend(user_id, viewing_history)
        content_recs = await self.content_filter.recommend(viewing_history)
        popular_recs = await self.popularity_ranker.get_trending(context.region)

        # Combine using ML model
        combined_recs = await self.ml_model.rank_recommendations(
            user_id=user_id,
            candidates=collab_recs + content_recs + popular_recs,
            context=context
        )

        return combined_recs[:20]  # Return top 20

    async def record_interaction(self, user_id, content_id, interaction_type, duration):
        """Record user interaction for future recommendations"""
        interaction = {
            'user_id': user_id,
            'content_id': content_id,
            'type': interaction_type,  # view, like, share, complete
            'duration': duration,
            'timestamp': time.time()
        }

        # Store for real-time updates
        await self.interaction_store.record(interaction)

        # Update user profile
        await self.update_user_profile(user_id, interaction)
```

### Key Design Decisions

#### 1. Microservices Architecture
**Decision**: Break down into 700+ microservices
**Rationale**: Independent scaling, team autonomy, fault isolation
**Trade-off**: Increased complexity, network overhead

#### 2. CDN Strategy
**Decision**: Multi-tier CDN with 1000+ edge locations
**Rationale**: Reduce latency, handle massive bandwidth
**Trade-off**: High infrastructure cost, cache management complexity

#### 3. Chaos Engineering
**Decision**: Deliberately introduce failures (Chaos Monkey)
**Rationale**: Improve system resilience, find weaknesses
**Trade-off**: Potential service disruptions, engineering overhead

### Challenges and Solutions

#### Challenge 1: Global Content Distribution
**Problem**: Serving content globally with low latency
**Solution**:
- Multi-tier CDN architecture
- Regional content caching
- Predictive content pre-positioning

#### Challenge 2: Personalization at Scale
**Problem**: Generating personalized recommendations for 200M users
**Solution**:
- Hybrid recommendation algorithms
- Real-time and batch processing
- A/B testing for algorithm optimization

#### Challenge 3: Video Quality Optimization
**Problem**: Delivering best quality within bandwidth constraints
**Solution**:
- Adaptive bitrate streaming
- Per-title encoding optimization
- Network-aware quality selection

### Lessons Learned
1. **Embrace Failure**: Design for failure from the beginning
2. **Data-Driven Decisions**: Use A/B testing for all major changes
3. **Operational Excellence**: Invest heavily in monitoring and automation
4. **Culture**: Freedom and responsibility culture enables innovation
5. **Technology Evolution**: Continuously evolve architecture and technology stack
