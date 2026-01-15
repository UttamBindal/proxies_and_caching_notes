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
*   **`port`**: The default port. You can change this to obscure the service or run multiple instances.
*   **`timeout`**: Close the connection after a client is idle for N seconds. Setting it to 0 keeps connections open indefinitely.
*   **`tcp-keepalive`**: Sends "pings" to clients to detect dead connections and clear them out.
*   **`rename-command CONFIG ""`**: A common "hardening" tactic that disables dangerous commands like CONFIG or FLUSHALL so they can't be used by mistake or by an attacker. 

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
dbfilename dump.rdb # Name of the RDB file
dir /var/lib/redis # Directory for RDB files
```
*   **Pros**: Creates compact files and allows fast restarts.
*   **Cons**: Risks data loss since the last snapshot if a crash occurs.

**Option B: AOF (Append Only File)**
```conf
appendonly yes # Enable AOF
appendfilename "appendonly.aof" # Name of the AOF file
appendfsync everysec # Fsync to disk every second. This offers a good balance.
```
*   **`appendonly yes`**: Enable AOF persistence. Redis logs every single write operation (like SET, INCR, SADD) into a log file as they happen. When Redis restarts, it "replays" this log to reconstruct the entire dataset.
*   **`appendfsync everysec`**: Fsync to disk every second. This offers a good balance.
*   **`appendfsync always`**: Fsync every time. Slowest but safest.
*   **`appendfsync no`**: Never fsync. Lets the Operating System decide when to flush (risky).
*   **`auto-aof-rewrite-percentage`**: Since the log grows with every command, it can become huge. Redis handles this by "rewriting" the log in the background to its shortest possible version.

*   **`auto-aof-rewrite-percentage 100`**: Rewrite the log when it's 100% larger than the last rewrite.
*   **`auto-aof-rewrite-min-size 64mb`**: Don't rewrite if the log is smaller than 64MB.    

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

#### 5. General Settings Block
These options handle the basic operational behavior of the Redis process.

*   **`daemonize yes`**: If set to yes, Redis runs in the background as a "daemon." If no, it runs in the foreground (useful for debugging).
*   **`loglevel notice`**: Controls how much information is written to the logs. Options include debug, verbose, notice, and warning.
*   **`databases 16`**: Sets the number of logical databases. (Note: Most developers stick to DB 0).

## Benefits
-   Critically reduce latency.
-   Reduce load on backend databases.
-   Improve application throughput.
