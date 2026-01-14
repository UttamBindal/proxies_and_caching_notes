# Reverse Proxy

## What is a Reverse Proxy?
A reverse proxy guards your web servers. It intercepts client requests and forwards them to the origin server. Clients communicate directly with the proxy, remaining unaware of the backend architecture.

## Flow
`Client -> Internet -> Reverse Proxy -> Origin Server`

## Key Use Cases
1.  **Load Balancing**: Distribute client requests across multiple backend servers.
2.  **Security**: Hide the topology and characteristics of backend servers to prevent direct attacks.
3.  **SSL Termination**: Handle encryption and decryption at the proxy level, reducing load on backend servers.
4.  **Caching**: Serve static content directly to avoid hitting the application server.
5.  **Compression**: Compress outgoing data to accelerate loading times.

## Reverse vs Forward Proxy
-   **Reverse Proxy**: Sit on the server side to represent the server to the client.
-   **Forward Proxy**: Sit on the client side to represent the client to the server.

## Setup Guide & Configuration (Nginx)

### Core Configuration
Use the `proxy_pass` directive inside a `location` block to handle reverse proxying.

```nginx
location /app/ {
    proxy_pass http://127.0.0.1:3000/;
    
    # Header Forwarding
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Timeouts
    proxy_connect_timeout 60s;
    proxy_read_timeout 60s;
    
    # Buffering
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
}
```

## Detailed Explanations

### 1. `proxy_pass` & Trailing Slashes
The trailing slash defines the path behavior. **Handle with care**.

*   **Case A**: `proxy_pass http://localhost:3000/;`
    *   Request: `example.com/app/user`
    *   Sent to Backend: `http://localhost:3000/user` (The `/app/` part gets stripped).
*   **Case B**: `proxy_pass http://localhost:3000;` (No trailing slash)
    *   Request: `example.com/app/user`
    *   Sent to Backend: `http://localhost:3000/app/user` (The path passes acts-is).

### 2. Header Manipulation (`proxy_set_header`)
When Nginx proxies a request, the backend sees the request coming from Nginx (127.0.0.1), not the actual user. You must forward the original details.

*   **`Host $host`**: Preserve the original `Host` header (e.g., example.com). This is essential for apps using virtual hosts.
*   **`X-Real-IP $remote_addr`**: Pass the client's actual IP address.
*   **`X-Forwarded-For ...`**: Append the client IP to a list. Use this standard for tracing IP chains through multiple proxies.
*   **`X-Forwarded-Proto $scheme`**: Inform the app if the original protocol used `http` or `https` (vital for generating redirect URLs).

### 3. Timeouts
*   **`proxy_connect_timeout`**: Define the wait time for the backend to accept the connection (TCP handshake).
*   **`proxy_read_timeout`**: Define the wait time for the backend to send data back. Increase this (e.g., `300s`) for slow API routes to avoid `504 Gateway Timeout` errors.

### 4. Buffering
*   **`proxy_buffering on`**: Enable buffering (default). Nginx reads the entire response from the backend before sending it to the client.
    *   *Pros*: Offload the backend quickly.
    *   *Cons*: First byte latency might be higher for large streams.
*   **`proxy_buffering off`**: Stream data to the client immediately as it arrives from the backend.
    *   *Use Case*: Real-time event streaming or Comet applications.

## WebSocket Support
WebSockets require special headers (`Upgrade` and `Connection`) because they switch protocols from HTTP to WebSocket. Explicitly add these, as Nginx drops them by default.

```nginx
location /ws/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```
