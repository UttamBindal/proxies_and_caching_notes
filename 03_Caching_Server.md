# Caching Server

## What is a Caching Server?
A caching server accelerates data delivery. This dedicated service saves web pages and internet content locally. By storing previously requested information, it provides faster access to data and reduces demand on enterprise bandwidth.

## Types of Caching
1.  **Browser Cache**: Store data directly on the client's machine.
2.  **CDN (Content Delivery Network)**: Use a distributed network of proxy servers close to users.
3.  **Application/Database Caching**: Store database query results in memory (e.g., Redis, Memcached).
4.  **Web Server Caching**: Implement caching at the server level using Varnish or Nginx.

## Popular Tools
-   **Redis**: Use this in-memory data structure store as a database, cache, and message broker.
-   **Memcached**: Deploy this high-performance, distributed memory object caching system.
-   **Varnish Cache**: Accelerate web applications with this caching HTTP reverse proxy.

## Setup Guide & Configuration (Redis)

### Installation
```bash
sudo apt update
sudo apt install redis-server
```

### Overview of `redis.conf`
The main configuration file resides at `/etc/redis/redis.conf`. This self-documented file contains hundreds of directives.

### Key Configuration Sections

#### 1. Network & Security
```conf
bind 127.0.0.1 ::1
protected-mode yes
port 6379
requirepass "SecureP@ssw0rd!"
```
*   **`bind`**: By default, Redis listens only on localhost. Allow remote access by adding the private IP of the interface (e.g., `bind 127.0.0.1 10.0.0.5`). **WARNING**: Always use strict firewalls before binding to `0.0.0.0` (public internet).
*   **`protected-mode`**: Enable this security feature. It ensures Redis only replies to loopback if no password or bind is set.
*   **`requirepass`**: Enforce authentication. Clients must use `AUTH <password>` to run commands.

#### 2. Memory Management (Critical Configuration)
```conf
maxmemory 2gb
maxmemory-policy allkeys-lru
```
*   **`maxmemory`**: Limit memory usage. Setting this to 0 uses all system memory, which risks OOM crashes.
*   **`maxmemory-policy`**: Define behavior when memory is full.
    *   **`noeviction`**: Return an error on write operations (default).
    *   **`allkeys-lru`**: Remove the least recently used keys (ideal for general caching).
    *   **`volatile-lru`**: Remove LRU keys that have an "expire" set.
    *   **`allkeys-random`**: Evict random keys.

#### 3. Persistence (Data Safety vs Performance)
Redis runs in-memory but supports saving to disk.

**Option A: RDB (Snapshots)**
```conf
# save <seconds> <changes>
save 900 1   # Save after 900s if 1 key changed
save 300 10  # Save after 300s if 10 keys changed
save 60 10000 
dbfilename dump.rdb
```
*   **Pros**: Creates compact files and allows fast restarts.
*   **Cons**: Risks data loss since the last snapshot if a crash occurs.

**Option B: AOF (Append Only File)**
```conf
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
```
*   **`appendfsync everysec`**: Fsync to disk every second. This offers a good balance.
*   **`appendfsync always`**: Fsync every time. Slowest but safest.
*   **Pros**: Provides higher durability (maximum 1 second data loss).
*   **Cons**: Results in larger file sizes and slower restarts.

#### 4. Clients
```conf
maxclients 10000
```
*   Limit the number of simultaneous connections.

### Applying Changes
After editing `redis.conf`:
```bash
sudo systemctl restart redis-server
```

## Benefits
-   Critically reduce latency.
-   Reduce load on backend databases.
-   Improve application throughput.
