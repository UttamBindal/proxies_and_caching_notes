# Load Balancer

## What is a Load Balancer?
A load balancer acts as the "traffic cop" sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked.

## Types of Load Balancers
1.  **Layer 4 (Transport Layer)**: Distributes traffic based on IP address and port (TCP/UDP). It's faster but has less context.
2.  **Layer 7 (Application Layer)**: Distributes traffic based on content (HTTP headers, cookies, URL path). More intelligent but slightly more resource-intensive.

## Load Balancing Algorithms
-   **Round Robin**: Requests are distributed across the group of servers sequentially.
-   **Least Connections**: A new request is sent to the server with the fewest current connections to clients.
-   **IP Hash**: The IP address of the client is used to determine which server receives the request.

## Setup Guide & Configuration (Nginx)

### The `upstream` Module
Load balancing in Nginx is defined in the `http` context using the `upstream` block. This block defines a pool of backend servers that Nginx can proxy traffic to.

### Configuration Structure
```nginx
http {
    upstream my_backend_app {
        # Load Balancing Algorithm (default is Round Robin)
        least_conn; 
        
        # Server List
        server 10.0.0.1:3000 weight=3;
        server 10.0.0.2:3000;
        server 10.0.0.3:3000 max_fails=3 fail_timeout=30s;
        server 10.0.0.4:3000 backup;
        server 10.0.0.5:3000 down;
    }

    server {
        listen 80;
        server_name lb.example.com;

        location / {
            proxy_pass http://my_backend_app;
            proxy_set_header Host $host;
        }
    }
}
```

## Detailed Options & Parameters

### Algorithms
By default, Nginx uses **Round Robin** (cycling through servers). You can enable others by adding the directive at the top of the `upstream` block:

1.  **`least_conn;`**: Directs traffic to the server with the fewest active connections. Good for sessions impacting server load unevenly.
2.  **`ip_hash;`**: Uses the client's IP address to hash the request. Ensures a client always connects to the same server (Session Persistence/Sticky Sessions).
3.  **`hash $request_uri consistent;`**: Hashes based on a key (e.g., URL).

### Server Parameters
These parameters control the behavior of individual upstream servers:

*   **`weight=N`**: Sets the priority. Start with weight 1. A server with `weight=3` receives 3x more requests than one with `weight=1`.
*   **`max_fails=N`**: The number of failed attempts allowed during the `fail_timeout` period before the server is marked "unavailable". Default is 1.
*   **`fail_timeout=T`**: 
    1.  The time window for counting `max_fails`.
    2.  How long the server stays "unavailable" after failing. Default is 10s.
*   **`backup`**: This server is ONLY used if all other non-backup servers are busy or down. Great for failover pages or disaster recovery.
*   **`down`**: Marks the server as permanently offline. Useful for maintenance without removing the line from config.

## Health Checks (Passive)
Nginx Open Source supports "Passive Health Checks". If a request to a server fails (e.g., 502 Bad Gateway), Nginx notes the failure and reroutes that request to another server. It won't try that server again until `fail_timeout` expires.

*Note: Active Health Checks (pinging /health route actively) are a feature of Nginx Plus (Paid).*

## Benefits
-   **Scalability**: Easily add or remove servers.
-   **Redundancy/High Availability**: If one server goes down, traffic is rerouted.
-   **Flexibility**: Perform maintenance without downtime.
