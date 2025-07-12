# Scalability in System Design

Scalability is the ability of a system to handle increased load by adding resources. This section explains scaling strategies, horizontal vs vertical scaling, and best practices for designing scalable systems.

## Scaling Strategies
- **Vertical Scaling:** Adding more power (CPU, RAM) to existing machines
- **Horizontal Scaling:** Adding more machines to distribute load
- **Auto-scaling:** Dynamically adjusting resources based on demand

## Example
E-commerce sites use horizontal scaling to handle traffic spikes during sales events.

## Best Practices
- Design stateless services for easier scaling
- Use load balancers to distribute traffic
