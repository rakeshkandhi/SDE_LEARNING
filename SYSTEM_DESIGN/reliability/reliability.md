# Reliability in System Design 🛡️

## Overview
Reliability is the probability that a system performs correctly during a specific time duration. In system design, reliability ensures that services remain available and functional even when individual components fail.

## Table of Contents
- [Reliability Fundamentals](#reliability-fundamentals)
- [Types of Failures](#types-of-failures)
- [Fault Tolerance Strategies](#fault-tolerance-strategies)
- [Redundancy Patterns](#redundancy-patterns)
- [Failover Mechanisms](#failover-mechanisms)
- [Circuit Breaker Pattern](#circuit-breaker-pattern)
- [Disaster Recovery](#disaster-recovery)
- [Monitoring and Alerting](#monitoring-and-alerting)
- [Testing for Reliability](#testing-for-reliability)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)

## Reliability Fundamentals

### Key Metrics

#### Availability
Percentage of time a system is operational and accessible.

```
Availability = (Total Time - Downtime) / Total Time × 100%
```

**Common Availability Targets:**
- **99.9% (8.77 hours downtime/year)**: Basic web applications
- **99.95% (4.38 hours downtime/year)**: Business-critical applications
- **99.99% (52.6 minutes downtime/year)**: High-availability systems
- **99.999% (5.26 minutes downtime/year)**: Mission-critical systems

#### Mean Time Between Failures (MTBF)
Average time between system failures.

#### Mean Time To Recovery (MTTR)
Average time to restore service after a failure.

#### Recovery Point Objective (RPO)
Maximum acceptable data loss during a disaster.

#### Recovery Time Objective (RTO)
Maximum acceptable downtime during a disaster.

### Reliability vs Other Qualities

```
Reliability ≠ Availability
- Reliability: Probability of correct operation
- Availability: Percentage of operational time

Reliability ≠ Fault Tolerance
- Reliability: Overall system dependability
- Fault Tolerance: Ability to continue despite failures
```

## Types of Failures

### 1. Hardware Failures
Physical component failures that are inevitable.

**Examples:**
- Server crashes
- Disk failures
- Network equipment failures
- Power outages

**Mitigation:**
- Redundant hardware
- RAID configurations
- UPS systems
- Multiple data centers

### 2. Software Failures
Bugs, configuration errors, or software crashes.

**Examples:**
- Application bugs
- Memory leaks
- Configuration errors
- Dependency failures

**Mitigation:**
- Thorough testing
- Code reviews
- Gradual rollouts
- Rollback mechanisms

### 3. Human Errors
Mistakes made by operators, developers, or users.

**Examples:**
- Incorrect configurations
- Accidental deletions
- Deployment errors
- Security breaches

**Mitigation:**
- Automation
- Access controls
- Change management
- Training and documentation

### 4. Network Failures
Communication failures between system components.

**Examples:**
- Network partitions
- DNS failures
- Load balancer failures
- Internet connectivity issues

**Mitigation:**
- Multiple network paths
- DNS redundancy
- Circuit breakers
- Graceful degradation

## Fault Tolerance Strategies

### 1. Replication
Maintain multiple copies of data or services.

```python
class ReplicatedService:
    def __init__(self, replicas):
        self.replicas = replicas

    def write(self, data):
        """Write to majority of replicas"""
        success_count = 0
        required_success = len(self.replicas) // 2 + 1

        for replica in self.replicas:
            try:
                replica.write(data)
                success_count += 1
            except Exception as e:
                print(f"Write failed to {replica}: {e}")

        return success_count >= required_success

    def read(self, key):
        """Read from any available replica"""
        for replica in self.replicas:
            try:
                return replica.read(key)
            except Exception as e:
                print(f"Read failed from {replica}: {e}")
                continue

        raise Exception("All replicas failed")
```

### 2. Graceful Degradation
Reduce functionality rather than complete failure.

```python
class WeatherService:
    def __init__(self, primary_api, backup_api, cache):
        self.primary_api = primary_api
        self.backup_api = backup_api
        self.cache = cache

    def get_weather(self, location):
        try:
            # Try primary API
            weather = self.primary_api.get_weather(location)
            self.cache.set(location, weather, ttl=3600)
            return weather
        except Exception:
            try:
                # Try backup API
                weather = self.backup_api.get_weather(location)
                self.cache.set(location, weather, ttl=1800)
                return weather
            except Exception:
                # Return cached data if available
                cached_weather = self.cache.get(location)
                if cached_weather:
                    return cached_weather

                # Return basic weather info
                return {"status": "unavailable", "message": "Weather service temporarily unavailable"}
```

### 3. Bulkhead Pattern
Isolate resources to prevent cascading failures.

```python
import threading
from concurrent.futures import ThreadPoolExecutor

class BulkheadService:
    def __init__(self):
        # Separate thread pools for different operations
        self.user_pool = ThreadPoolExecutor(max_workers=10, thread_name_prefix="user")
        self.order_pool = ThreadPoolExecutor(max_workers=5, thread_name_prefix="order")
        self.payment_pool = ThreadPoolExecutor(max_workers=3, thread_name_prefix="payment")

    def process_user_request(self, request):
        return self.user_pool.submit(self._handle_user_request, request)

    def process_order_request(self, request):
        return self.order_pool.submit(self._handle_order_request, request)

    def process_payment_request(self, request):
        return self.payment_pool.submit(self._handle_payment_request, request)
```

### 4. Timeout and Retry Patterns
Handle slow or failing dependencies.

```python
import time
import random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1, max_delay=60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries:
                        raise e

                    # Exponential backoff with jitter
                    delay = min(base_delay * (2 ** attempt), max_delay)
                    jitter = random.uniform(0, delay * 0.1)
                    time.sleep(delay + jitter)

        return wrapper
    return decorator

@retry_with_backoff(max_retries=3, base_delay=1)
def call_external_api(url, timeout=5):
    import requests
    response = requests.get(url, timeout=timeout)
    response.raise_for_status()
    return response.json()
```

## Redundancy Patterns

### 1. Active-Passive (Hot Standby)
One active instance with passive backup ready to take over.

```
Primary Server (Active) ←→ Health Check
Secondary Server (Passive) ←→ Health Check

If Primary fails → Secondary becomes Active
```

### 2. Active-Active (Load Sharing)
Multiple active instances sharing the load.

```
Load Balancer → [Server1 (Active), Server2 (Active), Server3 (Active)]
```

### 3. N+1 Redundancy
N active components plus 1 spare.

```python
class NPlus1Cluster:
    def __init__(self, active_nodes, spare_nodes):
        self.active_nodes = active_nodes
        self.spare_nodes = spare_nodes
        self.failed_nodes = []

    def handle_node_failure(self, failed_node):
        if failed_node in self.active_nodes:
            self.active_nodes.remove(failed_node)
            self.failed_nodes.append(failed_node)

            # Activate spare node if available
            if self.spare_nodes:
                spare = self.spare_nodes.pop(0)
                self.active_nodes.append(spare)
                print(f"Activated spare node: {spare}")
```

### 4. Geographic Redundancy
Distribute components across multiple locations.

```
Region 1 (US-East): [App Servers, Database Primary]
Region 2 (US-West): [App Servers, Database Replica]
Region 3 (EU): [App Servers, Database Replica]
```

## Failover Mechanisms

### 1. Automatic Failover
System automatically switches to backup without human intervention.

```python
import time
import threading

class AutomaticFailover:
    def __init__(self, primary, secondary, health_check_interval=30):
        self.primary = primary
        self.secondary = secondary
        self.current_active = primary
        self.health_check_interval = health_check_interval
        self.monitoring = True

        # Start health monitoring
        threading.Thread(target=self._monitor_health, daemon=True).start()

    def _monitor_health(self):
        while self.monitoring:
            if not self._is_healthy(self.current_active):
                self._perform_failover()
            time.sleep(self.health_check_interval)

    def _is_healthy(self, server):
        try:
            return server.health_check()
        except:
            return False

    def _perform_failover(self):
        if self.current_active == self.primary:
            if self._is_healthy(self.secondary):
                self.current_active = self.secondary
                print("Failover: Switched to secondary")
        else:
            if self._is_healthy(self.primary):
                self.current_active = self.primary
                print("Failback: Switched to primary")
```

### 2. Manual Failover
Human operator initiates the failover process.

### 3. Database Failover
Special considerations for database systems.

```sql
-- PostgreSQL streaming replication setup
-- On primary server
SELECT pg_start_backup('backup_label');

-- On standby server
pg_basebackup -h primary_host -D /var/lib/postgresql/data -U replication -v -P

-- Promote standby to primary
SELECT pg_promote();
```

## Circuit Breaker Pattern
Prevent cascading failures by stopping calls to failing services.

```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60, expected_exception=Exception):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception

        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise e

    def _should_attempt_reset(self):
        return (time.time() - self.last_failure_time) >= self.recovery_timeout

    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN

# Usage example
circuit_breaker = CircuitBreaker(failure_threshold=3, recovery_timeout=30)

def unreliable_service():
    # Simulate unreliable external service
    import random
    if random.random() < 0.7:  # 70% failure rate
        raise Exception("Service unavailable")
    return "Success"

# Protected call
try:
    result = circuit_breaker.call(unreliable_service)
    print(f"Result: {result}")
except Exception as e:
    print(f"Error: {e}")
```

## Disaster Recovery

### 1. Backup Strategies

#### Full Backups
Complete copy of all data.

```python
import shutil
import datetime

class BackupManager:
    def __init__(self, source_dir, backup_dir):
        self.source_dir = source_dir
        self.backup_dir = backup_dir

    def full_backup(self):
        timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_path = f"{self.backup_dir}/full_backup_{timestamp}"
        shutil.copytree(self.source_dir, backup_path)
        return backup_path

    def incremental_backup(self, last_backup_time):
        # Only backup files modified since last backup
        timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_path = f"{self.backup_dir}/incremental_backup_{timestamp}"

        # Implementation would check file modification times
        # and only copy changed files
        pass
```

#### Database Backup Strategies
```sql
-- PostgreSQL backup
pg_dump -h localhost -U username -d database_name > backup.sql

-- MySQL backup
mysqldump -u username -p database_name > backup.sql

-- MongoDB backup
mongodump --host localhost --port 27017 --out /backup/directory
```

### 2. Replication Strategies

#### Master-Slave Replication
```
Master Database (Read/Write) → Slave Database (Read Only)
```

#### Master-Master Replication
```
Master Database 1 ←→ Master Database 2
(Both can handle reads and writes)
```

#### Multi-Region Replication
```python
class MultiRegionReplication:
    def __init__(self, regions):
        self.regions = regions
        self.primary_region = regions[0]

    def write_data(self, data):
        # Write to primary region first
        self.primary_region.write(data)

        # Asynchronously replicate to other regions
        for region in self.regions[1:]:
            try:
                region.replicate(data)
            except Exception as e:
                print(f"Replication failed to {region}: {e}")
                # Queue for retry
                self._queue_for_retry(region, data)
```

### 3. Recovery Procedures

#### Point-in-Time Recovery
```sql
-- PostgreSQL point-in-time recovery
pg_basebackup -h primary_host -D /var/lib/postgresql/data
# Edit recovery.conf
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2023-01-15 14:30:00'
```

#### Automated Recovery Scripts
```bash
#!/bin/bash
# Disaster recovery script

echo "Starting disaster recovery..."

# 1. Stop application services
systemctl stop myapp

# 2. Restore database from backup
pg_restore -d myapp_db /backups/latest_backup.sql

# 3. Sync data files
rsync -av /backup/data/ /var/lib/myapp/data/

# 4. Update configuration for new environment
sed -i 's/old_server/new_server/g' /etc/myapp/config.yml

# 5. Start services
systemctl start postgresql
systemctl start myapp

echo "Disaster recovery completed"
```

## Monitoring and Alerting

### 1. Health Checks
```python
from flask import Flask, jsonify
import psutil
import requests

app = Flask(__name__)

@app.route('/health')
def health_check():
    health_status = {
        "status": "healthy",
        "timestamp": datetime.datetime.now().isoformat(),
        "checks": {}
    }

    # Check database connectivity
    try:
        # Perform database query
        health_status["checks"]["database"] = "healthy"
    except Exception as e:
        health_status["checks"]["database"] = f"unhealthy: {str(e)}"
        health_status["status"] = "unhealthy"

    # Check external dependencies
    try:
        response = requests.get("https://api.external-service.com/health", timeout=5)
        if response.status_code == 200:
            health_status["checks"]["external_service"] = "healthy"
        else:
            health_status["checks"]["external_service"] = f"unhealthy: HTTP {response.status_code}"
            health_status["status"] = "degraded"
    except Exception as e:
        health_status["checks"]["external_service"] = f"unhealthy: {str(e)}"
        health_status["status"] = "degraded"

    # Check system resources
    cpu_percent = psutil.cpu_percent()
    memory_percent = psutil.virtual_memory().percent

    if cpu_percent > 90 or memory_percent > 90:
        health_status["status"] = "degraded"

    health_status["checks"]["cpu"] = f"{cpu_percent}%"
    health_status["checks"]["memory"] = f"{memory_percent}%"

    status_code = 200 if health_status["status"] == "healthy" else 503
    return jsonify(health_status), status_code
```

### 2. Alerting Rules
```yaml
# Prometheus alerting rules
groups:
- name: reliability_alerts
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value }} errors per second"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Service is down"
      description: "{{ $labels.instance }} has been down for more than 1 minute"

  - alert: HighLatency
    expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "High latency detected"
      description: "95th percentile latency is {{ $value }} seconds"
```

### 3. Observability Stack
```python
# Application metrics with Prometheus
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('http_request_duration_seconds', 'HTTP request latency')
ACTIVE_CONNECTIONS = Gauge('active_connections', 'Number of active connections')

def track_request(method, endpoint, status_code, duration):
    REQUEST_COUNT.labels(method=method, endpoint=endpoint, status=status_code).inc()
    REQUEST_LATENCY.observe(duration)

# Start metrics server
start_http_server(8000)
```

## Testing for Reliability

### 1. Chaos Engineering
Deliberately introduce failures to test system resilience.

```python
import random
import time
from threading import Thread

class ChaosMonkey:
    def __init__(self, services, failure_probability=0.1):
        self.services = services
        self.failure_probability = failure_probability
        self.running = False

    def start(self):
        self.running = True
        Thread(target=self._chaos_loop, daemon=True).start()

    def stop(self):
        self.running = False

    def _chaos_loop(self):
        while self.running:
            if random.random() < self.failure_probability:
                service = random.choice(self.services)
                self._introduce_failure(service)
            time.sleep(60)  # Check every minute

    def _introduce_failure(self, service):
        failure_type = random.choice(['kill', 'network_delay', 'cpu_stress'])

        if failure_type == 'kill':
            service.kill()
        elif failure_type == 'network_delay':
            service.add_network_delay(random.randint(100, 1000))
        elif failure_type == 'cpu_stress':
            service.stress_cpu(random.randint(50, 90))

        print(f"Chaos Monkey: Introduced {failure_type} to {service}")
```

### 2. Load Testing
```python
import asyncio
import aiohttp
import time

async def load_test(url, concurrent_requests=100, duration=60):
    start_time = time.time()
    request_count = 0
    error_count = 0

    async with aiohttp.ClientSession() as session:
        while time.time() - start_time < duration:
            tasks = []
            for _ in range(concurrent_requests):
                tasks.append(make_request(session, url))

            results = await asyncio.gather(*tasks, return_exceptions=True)

            for result in results:
                request_count += 1
                if isinstance(result, Exception):
                    error_count += 1

    error_rate = error_count / request_count * 100
    print(f"Load test completed: {request_count} requests, {error_rate:.2f}% error rate")

async def make_request(session, url):
    try:
        async with session.get(url, timeout=10) as response:
            return await response.text()
    except Exception as e:
        raise e
```

### 3. Disaster Recovery Testing
```python
class DisasterRecoveryTest:
    def __init__(self, primary_system, backup_system):
        self.primary_system = primary_system
        self.backup_system = backup_system

    def test_failover(self):
        print("Starting disaster recovery test...")

        # 1. Verify primary system is healthy
        assert self.primary_system.is_healthy()

        # 2. Simulate primary system failure
        self.primary_system.simulate_failure()

        # 3. Trigger failover
        failover_time = time.time()
        self.backup_system.activate()

        # 4. Verify backup system is operational
        assert self.backup_system.is_healthy()

        # 5. Measure recovery time
        recovery_time = time.time() - failover_time
        print(f"Failover completed in {recovery_time:.2f} seconds")

        # 6. Test data consistency
        self._verify_data_consistency()

        print("Disaster recovery test passed")

    def _verify_data_consistency(self):
        # Verify that data is consistent between systems
        pass
```

## Real-World Examples

### Netflix
**Challenge**: Serve 200M+ users with 99.99% availability

**Solutions:**
- Microservices architecture for fault isolation
- Chaos Monkey for proactive failure testing
- Multi-region deployment
- Circuit breakers for dependency failures
- Extensive monitoring and alerting

### Amazon
**Challenge**: Handle massive e-commerce traffic with high reliability

**Solutions:**
- Service-oriented architecture
- Multiple availability zones
- Auto-scaling and load balancing
- Eventual consistency for better availability
- Comprehensive backup and recovery

### Google
**Challenge**: Provide search and cloud services with minimal downtime

**Solutions:**
- Distributed systems design
- Redundancy at every level
- Global load balancing
- Site Reliability Engineering (SRE) practices
- Automated incident response

## Best Practices

### Design Principles
1. **Assume Failures Will Happen**: Design for failure, not just success
2. **Fail Fast**: Detect and handle failures quickly
3. **Graceful Degradation**: Reduce functionality rather than complete failure
4. **Isolation**: Prevent failures from cascading
5. **Monitoring**: Comprehensive observability and alerting

### Implementation Guidelines
1. **Redundancy**: Eliminate single points of failure
2. **Health Checks**: Implement comprehensive health monitoring
3. **Timeouts**: Set appropriate timeouts for all operations
4. **Retries**: Implement retry logic with exponential backoff
5. **Circuit Breakers**: Protect against cascading failures

### Operational Practices
1. **Regular Testing**: Test failover and recovery procedures
2. **Chaos Engineering**: Proactively introduce failures
3. **Incident Response**: Have clear procedures for handling incidents
4. **Post-Mortems**: Learn from failures and improve systems
5. **Documentation**: Maintain up-to-date runbooks and procedures

### Common Pitfalls
- Over-engineering for reliability
- Not testing failure scenarios
- Ignoring human factors in reliability
- Inadequate monitoring and alerting
- Not planning for disaster recovery

---

**Next Steps:**
- Learn about [Scalability Patterns](../scalability/scalability.md)
- Explore [Load Balancing](../load_balancing/load_balancing.md)
- Study [Security in System Design](../security/security.md)
```
