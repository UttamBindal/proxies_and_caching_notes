# Load Balancer

## What is a Load Balancer?
A load balancer directs traffic like a skilled coordinator. It sits in front of your servers and routes client requests to capable servers. This maximizes speed and capacity utilization while ensuring no single server carries too much load.

## Types of Load Balancers
1.  **Layer 4 (Transport Layer)**: Distribute traffic based on IP address and port (TCP/UDP). This type operates fast but lacks application context.
2.  **Layer 7 (Application Layer)**: Distribute traffic based on content (HTTP headers, cookies, URL path). This intelligence uses more resources but enables smarter routing decisions.

## Load Balancing Algorithms
-   **Round Robin**: Distribute requests sequentially across the group of servers.
-   **Least Connections**: Send new requests to the server with the fewest active connections.
-   **IP Hash**: Use the client's IP address to determine the target server.

## Setup Guide & Configuration (Nginx)

### The `upstream` Module
Nginx defines load balancing in the `http` context using the `upstream` block. This block creates a pool of backend servers for Nginx to use as a proxy target.

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
Nginx uses **Round Robin** (cycles through servers) by default. Enable other algorithms by adding the directive to the top of the `upstream` block:

1.  **`least_conn;`**: Direct traffic to the server with the fewest active connections. This helps when sessions load servers unevenly.
2.  **`ip_hash;`**: Hash the request based on the client's IP address. This ensures a client always connects to the same server (Session Persistence/Sticky Sessions).
3.  **`hash $request_uri consistent;`**: Hash based on a key (e.g., URL).

### Server Parameters
Control the behavior of individual upstream servers with these parameters:

*   **`weight=N`**: Set the priority. Start with weight 1. A server with `weight=3` receives three times as many requests as one with `weight=1`.
*   **`max_fails=N`**: Define the allowed failed attempts during the `fail_timeout` period before marking the server "unavailable". Default is 1.
*   **`fail_timeout=T`**: 
    1.  Define the time window for counting `max_fails`.
    2.  Set how long the server stays "unavailable" after failing. Default is 10s.
*   **`backup`**: Use this server ONLY when all other non-backup servers are busy or down. Ideal for failover pages or disaster recovery.
*   **`down`**: Mark the server as permanently offline. Use this for maintenance to keep the line in the config.

## Health Checks (Passive)
Nginx Open Source supports "Passive Health Checks". If a request to a server fails (e.g., 502 Bad Gateway), Nginx notes the failure and reroutes that request to another server. It avoids that server until `fail_timeout` expires.

*Note: Active Health Checks (pinging /health route actively) require Nginx Plus (Paid).*

## Benefits
-   **Scalability**: Add or remove servers with ease.
-   **Redundancy/High Availability**: Reroute traffic instantly if a server goes down.
-   **Flexibility**: Perform maintenance without causing downtime.
