# Caching Server

## What is a Caching Server?
A caching server is a dedicated network server or service consisting of software that saves Web pages or other Internet content locally. By placing previously requested info in temporary storage, a caching server speeds up access to data and reduces demand on an enterprise’s bandwidth.

## Types of Caching
1.  **Browser Cache**: Stored on the client's machine.
2.  **CDN (Content Delivery Network)**: Distributed network of proxy servers closest to users.
3.  **Application/Database Caching**: Storing database query results in memory (e.g., Redis, Memcached).
4.  **Web Server Caching**: Varnish, Nginx caching.

## Popular Tools
-   **Redis**: In-memory data structure store, used as a database, cache, and message broker.
-   **Memcached**: High-performance, distributed memory object caching system.
-   **Varnish Cache**: Web application accelerator also known as a caching HTTP reverse proxy.

## Setup Guide & Configuration (Redis)

### Installation
```bash
sudo apt update
sudo apt install redis-server
```

### Overview of `redis.conf`
The main configuration file is usually found at `/etc/redis/redis.conf`. It is a self-documented file with hundreds of directives.

### Key Configuration Sections

#### 1. Network & Security
```conf
bind 127.0.0.1 ::1
protected-mode yes
port 6379
requirepass "SecureP@ssw0rd!"
```
*   **`bind`**: By default, Redis listens only on localhost. To allow remote access, add the private IP of the interface (e.g., `bind 127.0.0.1 10.0.0.5`). **WARNING**: Never bind to `0.0.0.0` (public internet) without strict firewalls.
*   **`protected-mode`**: Security feature. If yes, and no password/bind is set, Redis only replies to loopback.
*   **`requirepass`**: Enforces authentication. Clients must use `AUTH <password>` to run commands.

#### 2. Memory Management (The most important/popular part)
```conf
maxmemory 2gb
maxmemory-policy allkeys-lru
```
*   **`maxmemory`**: Limits the memory usage. If 0, it uses all system memory (dangerous, can cause OOM crashes).
*   **`maxmemory-policy`**: What to do when memory is full.
    *   **`noeviction`**: Returns error on write operations (default).
    *   **`allkeys-lru`**: Removes least recently used keys (best for general caching).
    *   **`volatile-lru`**: Removes LRU keys that have an "expire" set.
    *   **`allkeys-random`**: Evicts random keys.

#### 3. Persistence (Data Safety vs Performance)
Redis is in-memory, but can save to disk.

**Option A: RDB (Snapshots)**
```conf
# save <seconds> <changes>
save 900 1   # Save after 900s if 1 key changed
save 300 10  # Save after 300s if 10 keys changed
save 60 10000 
dbfilename dump.rdb
```
*   **Pros**: Compact, fast restarts.
*   **Cons**: You lose data since the last snapshot if it crashes.

**Option B: AOF (Append Only File)**
```conf
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
```
*   **`appendfsync everysec`**: Fsyncs to disk every second. Good balance.
*   **`appendfsync always`**: Slow but safest.
*   **Pros**: Higher durability (1 sec data loss max).
*   **Cons**: Larger file size, slower restart.

#### 4. Clients
```conf
maxclients 10000
```
*   Limits the number of simultaneous connections.

### Applying Changes
After editing `redis.conf`:
```bash
sudo systemctl restart redis-server
```

## Benefits
-   Critically reduces latency.
-   Reduces load on backend databases.
-   Improves application throughput.
