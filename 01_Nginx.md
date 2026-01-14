# Nginx

## What is Nginx?
Nginx (pronounced "engine-x") powers high-performance web delivery. It works as an HTTP server, reverse proxy server, and mail proxy server. Developers value its stability, rich feature set, simple configuration, and low resource consumption.

## Key Features
- **Reverse Proxying with Caching**: Distribute load and cache content intelligently.
- **Load Balancing**: Route traffic efficiently across multiple upstream servers.
- **Static File Serving**: Serve assets like images, CSS, and JS with extreme speed.
- **TLS/SSL Termination**: Handle encryption and decryption here, freeing up application servers.

## Setup Guide (Basic Installation)

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
The main configuration file lives at `/etc/nginx/nginx.conf`. It uses a hierarchical structure of "blocks" (enclosed in `{}`).

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
*   **`user`**: Set the user and group credentials for worker processes. Security Tip: Avoid running as root.
*   **`worker_processes`**: Define the number of worker processes. Use `auto` to match CPU cores.
*   **`pid`**: storing the process ID of the main process.

### Events Context
*   **`worker_connections`**: Limit the simultaneous connections per worker. Total max connections equals `worker_processes` multiplied by `worker_connections`.

### HTTP Context
*   **`sendfile`**: Enable the `sendfile()` kernel call for zero-copy file transfer. Boosts performance significantly.
*   **`tcp_nopush`**: Send headers and the beginning of the file in one packet (works with `sendfile`).
*   **`tcp_nodelay`**: Disable Nagle's algorithm. Send data immediately. Excellent for real-time apps.
*   **`keepalive_timeout`**: Keep client connections open for a specified time. Reduces TCP handshake overhead.
*   **`gzip`**: Compress data to reduce payload size.
*   **`include`**: Import other configuration files to keep the main config clean.
    *   `/etc/nginx/sites-available/`: Define individual site configs here.
    *   `/etc/nginx/sites-enabled/`: Symlink active sites from `sites-available`.

## Server Block (Virtual Host)
Find these in `/etc/nginx/sites-available/example.com`.

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

*   **`listen`**: Define the port and IP. `default_server` captures mismatched requests.
*   **`server_name`**: Specify the domains this block serves.
*   **`root`**: Set the directory for static files.
*   **`location`**: Match specific URL patterns.
    *   `try_files $uri $uri/ =404`: Serve the file or directory, or fall back to 404.
    *   `autoindex on`: List directory files (use carefully).

## Common Commands
- `nginx -t`: Test the configuration for syntax errors.
- `sudo systemctl reload nginx`: Reload the configuration without downtime.
