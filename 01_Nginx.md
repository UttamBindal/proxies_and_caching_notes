# Nginx

## What is Nginx?
Nginx (pronounced "engine-x") is a high-performance HTTP server, reverse proxy server, and mail proxy server. It is known for its stability, rich feature set, simple configuration, and low resource consumption.

## Key Features
- **Reverse Proxying with Caching**: Distributes load and caches content.
- **Load Balancing**: Distributes traffic efficiently across multiple upstream servers.
- **Static File Serving**: serve static assets (images, CSS, JS) extremely fast.
- **TLS/SSL Termination**: Handles encryption/decryption, offloading the application server.

## How to Setup (Basic Installation)

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install nginx
```

### CentOS/RHEL
```bash
sudo yum install epel-release
sudo yum install nginx
```

### Start and Enable Service
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Configuration Guide & Structure

### Overview of `nginx.conf`
The main configuration file is typically located at `/etc/nginx/nginx.conf`. It is structured hierarchically using "blocks" (enclosed in `{}`).

### Configuration Structure
```nginx
# 1. Main (Global) Context
user www-data;
worker_processes auto;
pid /run/nginx.pid;

# 2. Events Context
events {
    worker_connections 768;
    # multi_accept on;
}

# 3. HTTP Context
http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip Settings
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # Virtual Host Configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## Detailed Directive Explanation

### Global Context
*   **`user`**: Defines the user and group credentials used by worker processes. Security best practice: never run as root.
*   **`worker_processes`**:  Number of worker processes. Set to `auto` to match the number of CPU cores.
*   **`pid`**:  Defines the file that will store the process ID of the main process.

### Events Context
*   **`worker_connections`**: The maximum number of simultaneous connections that can be opened by a worker process. Total max connections = `worker_processes` * `worker_connections`.

### HTTP Context
*   **`sendfile`**: Enables the use of `sendfile()` kernel system call for efficient file transfer (zero-copy). Essential for performance.
*   **`tcp_nopush`**: Used with `sendfile`. Sends the response header and beginning of the file in one packet.
*   **`tcp_nodelay`**: Disables Nagle's algorithm. Sends data immediately without waiting for the packet to fill up. Good for real-time apps.
*   **`keepalive_timeout`**: How long (in seconds) to keep a connection open for the client. Reduces the overhead of establishing new TCP handshakes.
*   **`gzip`**: Enables compression to reduce payload size.
*   **`include`**: Imports other configuration files. This keeps the main config clean.
    *   `/etc/nginx/sites-available/`: Where you define individual site configs.
    *   `/etc/nginx/sites-enabled/`: Symlinks to `sites-available` for active sites.

## Server Block (Virtual Host)
Located usually in `/etc/nginx/sites-available/example.com`.

```nginx
server {
    listen 80 default_server;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location /images/ {
        autoindex on;
    }
}
```

*   **`listen`**: Port and IP to listen on. `default_server` handles requests that don't match other `server_name`s.
*   **`server_name`**: Domain names this block responds to.
*   **`root`**: Directory from which static files are served.
*   **`location`**: Matches specific URL patterns.
    *   `try_files $uri $uri/ =404`: Tries to serve the file ($uri), then the directory ($uri/), acts as a fallback if not found.
    *   `autoindex on`: Lists files in the directory (useful for file dumps, usually off for security).

## Common Commands
- `nginx -t`: Test configuration for syntax errors.
- `sudo systemctl reload nginx`: Reload configuration without downtime.
