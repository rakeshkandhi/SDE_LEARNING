# Load Balancing in System Design ⚖️

## Overview
Load balancing is a critical technique for distributing incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It improves application responsiveness, increases availability, and provides redundancy.

## Table of Contents
- [Why Load Balancing?](#why-load-balancing)
- [Types of Load Balancers](#types-of-load-balancers)
- [Load Balancing Algorithms](#load-balancing-algorithms)
- [Health Checks](#health-checks)
- [Session Persistence](#session-persistence)
- [SSL Termination](#ssl-termination)
- [Implementation Examples](#implementation-examples)
- [Popular Load Balancers](#popular-load-balancers)
- [Best Practices](#best-practices)

## Why Load Balancing?

### Benefits
1. **High Availability**: Eliminates single points of failure
2. **Scalability**: Distributes load across multiple servers
3. **Performance**: Reduces response time and increases throughput
4. **Flexibility**: Easy to add/remove servers
5. **Maintenance**: Zero-downtime deployments

### Without Load Balancer
```
Client → Single Server (Bottleneck, Single Point of Failure)
```

### With Load Balancer
```
Client → Load Balancer → [Server1, Server2, Server3, Server4]
```

## Types of Load Balancers

### 1. Layer 4 Load Balancers (Transport Layer)
Operate at the transport layer, making routing decisions based on IP and port.

**Characteristics:**
- Fast and efficient
- Protocol agnostic
- Lower latency
- Limited routing capabilities

**Example:**
```
Client:80 → Load Balancer → Server1:8080
Client:80 → Load Balancer → Server2:8080
```

### 2. Layer 7 Load Balancers (Application Layer)
Operate at the application layer, making intelligent routing decisions based on content.

**Characteristics:**
- Content-aware routing
- SSL termination
- Compression and caching
- Higher latency but more features

**Example:**
```
/api/users → User Service Servers
/api/orders → Order Service Servers
/api/products → Product Service Servers
```

### 3. Hardware vs Software Load Balancers

#### Hardware Load Balancers
**Examples:** F5 BIG-IP, Citrix NetScaler, Barracuda

**Pros:**
- High performance
- Dedicated hardware
- Advanced features
- Vendor support

**Cons:**
- Expensive
- Vendor lock-in
- Limited flexibility
- Complex configuration

#### Software Load Balancers
**Examples:** HAProxy, NGINX, AWS ALB, Google Cloud Load Balancer

**Pros:**
- Cost-effective
- Flexible and programmable
- Easy to scale
- Cloud-native

**Cons:**
- Resource overhead
- Requires management
- Performance depends on hardware

## Load Balancing Algorithms

### 1. Round Robin
Distributes requests evenly across all servers in rotation.

```python
class RoundRobinBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.current = 0

    def get_server(self):
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server
```

**Pros:** Simple, fair distribution
**Cons:** Doesn't consider server capacity or current load

### 2. Weighted Round Robin
Assigns different weights to servers based on their capacity.

```python
class WeightedRoundRobinBalancer:
    def __init__(self, servers_weights):
        self.servers_weights = servers_weights  # [(server, weight), ...]
        self.current_weights = [0] * len(servers_weights)

    def get_server(self):
        total_weight = sum(weight for _, weight in self.servers_weights)

        # Increase current weights
        for i, (_, weight) in enumerate(self.servers_weights):
            self.current_weights[i] += weight

        # Find server with highest current weight
        max_weight_index = max(range(len(self.current_weights)),
                              key=lambda i: self.current_weights[i])

        # Decrease selected server's current weight
        self.current_weights[max_weight_index] -= total_weight

        return self.servers_weights[max_weight_index][0]
```

### 3. Least Connections
Routes requests to the server with the fewest active connections.

```python
class LeastConnectionsBalancer:
    def __init__(self, servers):
        self.servers = servers
        self.connections = {server: 0 for server in servers}

    def get_server(self):
        return min(self.connections, key=self.connections.get)

    def add_connection(self, server):
        self.connections[server] += 1

    def remove_connection(self, server):
        self.connections[server] = max(0, self.connections[server] - 1)
```

### 4. Weighted Least Connections
Combines least connections with server weights.

### 5. IP Hash
Uses client IP to determine server assignment (ensures session persistence).

```python
import hashlib

class IPHashBalancer:
    def __init__(self, servers):
        self.servers = servers

    def get_server(self, client_ip):
        hash_value = int(hashlib.md5(client_ip.encode()).hexdigest(), 16)
        return self.servers[hash_value % len(self.servers)]
```

### 6. Least Response Time
Routes to server with lowest response time and fewest connections.

### 7. Resource-Based
Considers server CPU, memory, and other resources.

## Health Checks

### Active Health Checks
Load balancer actively monitors server health.

```python
import requests
import time
from threading import Thread

class HealthChecker:
    def __init__(self, servers, check_interval=30):
        self.servers = servers
        self.check_interval = check_interval
        self.healthy_servers = set(servers)
        self.start_monitoring()

    def check_server_health(self, server):
        try:
            response = requests.get(f"http://{server}/health", timeout=5)
            return response.status_code == 200
        except:
            return False

    def monitor_health(self):
        while True:
            for server in self.servers:
                if self.check_server_health(server):
                    self.healthy_servers.add(server)
                else:
                    self.healthy_servers.discard(server)
            time.sleep(self.check_interval)

    def start_monitoring(self):
        Thread(target=self.monitor_health, daemon=True).start()

    def get_healthy_servers(self):
        return list(self.healthy_servers)
```

### Passive Health Checks
Monitor server health based on actual request responses.

```python
class PassiveHealthChecker:
    def __init__(self, servers, failure_threshold=3):
        self.servers = servers
        self.failure_threshold = failure_threshold
        self.failure_counts = {server: 0 for server in servers}
        self.healthy_servers = set(servers)

    def record_success(self, server):
        self.failure_counts[server] = 0
        self.healthy_servers.add(server)

    def record_failure(self, server):
        self.failure_counts[server] += 1
        if self.failure_counts[server] >= self.failure_threshold:
            self.healthy_servers.discard(server)
```

### Health Check Endpoints
```python
# Flask health check endpoint
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health')
def health_check():
    # Check database connectivity
    if not check_database():
        return jsonify({"status": "unhealthy", "reason": "database"}), 503

    # Check external dependencies
    if not check_external_services():
        return jsonify({"status": "unhealthy", "reason": "dependencies"}), 503

    return jsonify({"status": "healthy"}), 200

def check_database():
    try:
        # Perform database query
        return True
    except:
        return False

def check_external_services():
    # Check external API availability
    return True
```

## Session Persistence (Sticky Sessions)

### Cookie-Based Persistence
```nginx
# NGINX configuration
upstream backend {
    ip_hash;  # Ensures same client goes to same server
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}
```

### Application-Level Session Sharing
```python
# Using Redis for session storage
import redis
import json

class SessionManager:
    def __init__(self):
        self.redis_client = redis.Redis(host='redis-cluster')

    def set_session(self, session_id, data):
        self.redis_client.setex(
            f"session:{session_id}",
            3600,  # 1 hour TTL
            json.dumps(data)
        )

    def get_session(self, session_id):
        data = self.redis_client.get(f"session:{session_id}")
        return json.loads(data) if data else None
```

## SSL Termination

### SSL Termination at Load Balancer
```
Client (HTTPS) → Load Balancer (SSL Termination) → Servers (HTTP)
```

**Benefits:**
- Reduces server CPU load
- Centralized certificate management
- Easier certificate updates

### End-to-End SSL
```
Client (HTTPS) → Load Balancer (HTTPS) → Servers (HTTPS)
```

**Benefits:**
- Enhanced security
- Compliance requirements
- Protection against internal threats

### NGINX SSL Configuration
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Implementation Examples

### HAProxy Configuration
```haproxy
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:80
    bind *:443 ssl crt /path/to/certificate.pem
    redirect scheme https if !{ ssl_fc }
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /health
    server web1 192.168.1.10:8080 check
    server web2 192.168.1.11:8080 check
    server web3 192.168.1.12:8080 check
```

### AWS Application Load Balancer (Terraform)
```hcl
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets           = aws_subnet.public[*].id

  enable_deletion_protection = false
}

resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

### Docker Compose with NGINX Load Balancer
```yaml
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2
      - app3

  app1:
    image: myapp:latest
    expose:
      - "8080"

  app2:
    image: myapp:latest
    expose:
      - "8080"

  app3:
    image: myapp:latest
    expose:
      - "8080"
```

## Popular Load Balancers

### Open Source
1. **HAProxy**
   - High performance
   - Layer 4 and Layer 7
   - Advanced features
   - Excellent documentation

2. **NGINX**
   - Web server + load balancer
   - Reverse proxy capabilities
   - SSL termination
   - Caching features

3. **Traefik**
   - Cloud-native
   - Automatic service discovery
   - Docker/Kubernetes integration
   - Let's Encrypt integration

### Cloud-Based
1. **AWS Elastic Load Balancer**
   - Application Load Balancer (Layer 7)
   - Network Load Balancer (Layer 4)
   - Classic Load Balancer (Legacy)

2. **Google Cloud Load Balancer**
   - Global load balancing
   - Anycast IP addresses
   - Auto-scaling integration

3. **Azure Load Balancer**
   - Layer 4 load balancing
   - Internal and external
   - High availability

### Enterprise
1. **F5 BIG-IP**
   - Hardware and software
   - Advanced traffic management
   - Security features
   - High performance

2. **Citrix NetScaler**
   - Application delivery controller
   - SSL acceleration
   - Compression and caching

## Best Practices

### Design Principles
1. **Redundancy**: Deploy multiple load balancers
2. **Health Monitoring**: Implement comprehensive health checks
3. **Graceful Degradation**: Handle server failures gracefully
4. **Security**: Implement proper SSL/TLS and security headers
5. **Monitoring**: Track performance metrics and alerts

### Configuration Guidelines
1. **Appropriate Algorithm**: Choose based on application characteristics
2. **Health Check Tuning**: Balance frequency vs overhead
3. **Session Management**: Consider stateless design or shared sessions
4. **SSL Configuration**: Use strong ciphers and protocols
5. **Logging**: Enable detailed logging for troubleshooting

### Monitoring Metrics
```python
# Key metrics to monitor
metrics = {
    'request_rate': 'requests per second',
    'response_time': '95th percentile latency',
    'error_rate': 'percentage of failed requests',
    'server_health': 'number of healthy servers',
    'connection_count': 'active connections per server',
    'ssl_handshake_time': 'SSL negotiation latency'
}
```

### Common Pitfalls
1. **Single Point of Failure**: Not having redundant load balancers
2. **Inadequate Health Checks**: Poor health check configuration
3. **Session Stickiness**: Over-reliance on sticky sessions
4. **Ignoring SSL**: Not implementing proper SSL termination
5. **Poor Monitoring**: Insufficient visibility into load balancer performance

### Scaling Considerations
1. **Horizontal Scaling**: Add more servers behind load balancer
2. **Load Balancer Scaling**: Scale load balancers themselves
3. **Geographic Distribution**: Use multiple load balancers across regions
4. **Auto-scaling Integration**: Automatically add/remove servers
5. **Capacity Planning**: Monitor and plan for traffic growth

## Real-World Scenarios

### E-commerce During Black Friday
```
Challenge: Handle 10x normal traffic
Solution:
- Pre-scale server capacity
- Implement circuit breakers
- Use CDN for static content
- Queue non-critical operations
```

### Global SaaS Application
```
Challenge: Serve users worldwide with low latency
Solution:
- Geographic load balancing
- Regional server deployments
- Anycast IP addresses
- Edge caching
```

### Microservices Architecture
```
Challenge: Route requests to appropriate services
Solution:
- Service mesh (Istio, Linkerd)
- API Gateway with routing rules
- Service discovery integration
- Circuit breaker patterns
```

---

**Next Steps:**
- Learn about [Microservices Architecture](../microservices/microservices.md)
- Explore [Scalability Patterns](../scalability/scalability.md)
- Study [Reliability and Fault Tolerance](../reliability/reliability.md)
```
