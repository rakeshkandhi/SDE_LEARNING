# Load Balancing in System Design

Load balancing distributes incoming traffic across multiple servers to ensure reliability and performance. Learn about load balancing algorithms, hardware/software solutions, and best practices.

## Algorithms
- **Round Robin:** Evenly distributes requests
- **Least Connections:** Sends requests to server with fewest connections
- **IP Hash:** Uses client IP to determine server

## Example
Web applications use load balancers to prevent any single server from being overwhelmed.

## Best Practices
- Use health checks to detect failed servers
- Combine hardware and software solutions for flexibility
