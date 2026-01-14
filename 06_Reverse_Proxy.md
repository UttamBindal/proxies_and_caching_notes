# Reverse Proxy

## What is a Reverse Proxy?
A reverse proxy sits in front of one or more web servers and intercepts requests from clients. Ideally, clients are unaware they are communicating with a proxy; they believe they are talking directly to the origin server.

## Flow
`Client -> Internet -> Reverse Proxy -> Origin Server`

## Key Use Cases
1.  **Load Balancing**: Distributing client requests across multiple backend servers.
2.  **Security**: Hides the topology and characteristics of the backend servers. Can prevent DDoS attacks.
3.  **SSL Termination**: Handshake and encryption/decryption happen at the proxy, reducing load on backend servers.
4.  **Caching**: Serve static content directly without hitting the application server.
5.  **Compression**: Compress outgoing data to speed up loading times.

## Reverse vs Forward Proxy
-   **Reverse Proxy**: Sits on the server side. Represents the server to the client.
-   **Forward Proxy**: Sits on the client side. Represents the client to the server.

## Setup Guide & Configuration (Nginx)

### Core Configuration
Reverse proxying is primarily handled by the `proxy_pass` directive inside a `location` block.

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
The trailing slash is **CRITICAL**.

*   **Case A**: `proxy_pass http://localhost:3000/;`
    *   Request: `example.com/app/user`
    *   Sent to Backend: `http://localhost:3000/user` (The `/app/` part is stripped).
*   **Case B**: `proxy_pass http://localhost:3000;` (No trailing slash)
    *   Request: `example.com/app/user`
    *   Sent to Backend: `http://localhost:3000/app/user` (Path is passed as-is).

### 2. Header Manipulation (`proxy_set_header`)
When Nginx proxies a request, the backend sees the request coming from Nginx (127.0.0.1), not the actual user. We must forward the original details.

*   **`Host $host`**: Preserves the original `Host` header (e.g., example.com) instead of changing it to `localhost`. Essential for apps causing virtual hosts.
*   **`X-Real-IP $remote_addr`**: Passes the client's actual IP.
*   **`X-Forwarded-For ...`**: Appends the client IP to a list (in case of multiple proxies). Standard for tracing IP chains.
*   **`X-Forwarded-Proto $scheme`**: Tells the app if the original protocol was `http` or `https` (useful for generating redirect URLs).

### 3. Timeouts
*   **`proxy_connect_timeout`**: How long to wait for the backend to accept the connection (TCP handshake).
*   **`proxy_read_timeout`**: How long to wait for the backend to send data back. If you have a slow API route (e.g., generating a PDF), you **must** increase this (e.g., `300s`), otherwise Nginx returns `504 Gateway Timeout`.

### 4. Buffering
*   **`proxy_buffering on`**: Default. Nginx reads the entire response from the backend, then sends it to the client.
    *   *Pros*: Offloads the backend quickly.
    *   *Cons*: First byte latency might be higher for large streams.
*   **`proxy_buffering off`**: Stream data to client as it comes from backend.
    *   *Use Case*: Real-time event streaming or Comet applications.

## WebSocket Support
WebSockets require special headers (`Upgrade` and `Connection`) because they switch protocols from HTTP to WebSocket. Nginx drops these by default.

```nginx
location /ws/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```
