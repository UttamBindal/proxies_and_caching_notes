# Forward Proxy

## What is a Forward Proxy?
A forward proxy, or simply "proxy," serves as a gateway for client machines. It intercepts requests from clients to the internet and communicates with web servers on their behalf.

## Flow
`Client -> Forward Proxy -> Internet (or Destination Server)`

## Key Use Cases
1.  **Privacy/Anonymity**: Mask the client's internal IP address. The destination sees only the Proxy's IP.
2.  **Content Filtering**: Block access to specific websites (e.g., social media) in schools or workplaces.
3.  **Bypassing Geo-Restrictions**: Route through a proxy in a specific region to access locally restricted content.
4.  **Caching**: Cache frequently accessed resources to save bandwidth on the local network.

## Forward vs Reverse Proxy
-   **Forward Proxy**: Actively protect and serve the **Client**.
-   **Reverse Proxy**: Actively protect and serve the **Server**.

## Setup Guide & Configuration (Squid Proxy)

### Installation
```bash
sudo apt install squid
```

### Overview of `squid.conf`
The Squid configuration resides at `/etc/squid/squid.conf`. Users know it for its size and flexibility. It operates using ACLs (Access Control Lists) and Access Directives.

### Rule Logic
Squid checks rules **top-down**. The first matching rule determines the action (Allow or Deny).

### Basic Structure

```squid
# 1. Define ACLs (Who/What)
acl localnet src 10.0.0.0/8
acl localnet src 192.168.0.0/16
acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 443
acl CONNECT method CONNECT

# 2. Define Access Rules (Action)
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager

# YOUR RULES HERE
http_access allow localnet
http_access allow localhost

# Fallback Rule (Default Deny)
http_access deny all

# 3. Port Configuration
http_port 3128
```

## Detailed Directives

### 1. `acl` (Access Control List)
Define "groups" to match against.
*   **Syntax**: `acl <name> <type> <value>`
*   **Types**:
    *   `src`: Source IP address (Client IP).
    *   `dstdomain`: Destination domain (e.g., `.facebook.com`).
    *   `time`: Time of day/week (e.g., `M T W H F 09:00-17:00`).
    *   `port`: Destination port.

**Example: Block Social Media**
```squid
acl social_media dstdomain .facebook.com .twitter.com .instagram.com
```

### 2. `http_access` (The Gatekeeper)
Apply actions to ACLs.
*   **Syntax**: `http_access <allow|deny> <acl_name>`

**Example: Deny the Social Media ACL created above**
```squid
http_access deny social_media
```

### 3. `http_port`
*   `http_port 3128`: Use the standard proxy port.
*   `http_port 3128 intercept`: Enable "Transparent Proxying" (requires iptables redirection) so clients connect without manual browser configuration.

### 4. Caching Directives
Control disk and RAM storage usage.

```squid
# cache_dir <storage_type> <directory> <size_MB> <L1_dirs> <L2_dirs>
cache_dir ufs /var/spool/squid 100 16 256
```
*   `ufs`: Standard unix file system storage.
*   `100`: Allocate 100MB of disk.
*   `16 256`: Set directory structure depth for performance (keep defaults usually).

### 5. `visible_hostname`
Squid requires a detectable hostname. If detection fails, set it manually.
```squid
visible_hostname proxy.mycompany.com
```
