### What is Load Balancing?
Load balancing is a technique used to distribute network or application traffic across multiple servers to ensure no single server becomes overwhelmed. By spreading the load, it improves availability, reliability, and responsiveness. Load balancers can distribute traffic to web servers, databases, or other applications.

### Types of Load Balancing:
1. **Hardware Load Balancing**:
   - Uses physical appliances to distribute traffic.
   - Typically used in large enterprises with high traffic demands.
   - Examples: F5, A10 Networks.

2. **Software Load Balancing**:
   - Uses software-based solutions to distribute traffic.
   - Popular in cloud-based and smaller-scale deployments.
   - Examples: HAProxy, Nginx, Traefik.

3. **DNS-based Load Balancing**:
   - Distributes traffic by assigning different IP addresses to domain name requests.
   - Often used for global load balancing to direct users to geographically close servers.
   - Example: AWS Route 53, Google Cloud DNS.

### Load Balancing Techniques:
1. **Round Robin**:
   - Distributes requests sequentially across all servers in the pool.
   - Simple but doesn’t account for server load or health.

2. **Least Connections**:
   - Directs traffic to the server with the fewest active connections.
   - Ensures that no single server gets too overloaded.

3. **IP Hash**:
   - Uses the client’s IP address to determine which server receives the traffic.
   - Ensures that a specific client is directed to the same server for session persistence.

4. **Weighted Round Robin**:
   - Servers are assigned a weight, and the load balancer distributes traffic based on these weights.
   - Useful when servers have different capabilities.

5. **Health-based Load Balancing**:
   - Checks the health of each server and directs traffic only to servers that are online and responsive.
   - Increases reliability by avoiding downed or underperforming servers.

6. **Geographic Load Balancing**:
   - Directs users to the closest or fastest server based on their location.
   - Improves performance by reducing latency.

