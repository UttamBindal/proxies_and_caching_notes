# Forward Proxy

## What is a Forward Proxy?
A forward proxy, commonly known simply as a "proxy," sits in front of a group of client machines. When those clients make requests to sites on the Internet, the proxy server intercepts those requests and then communicates with the web servers on behalf of those clients.

## Flow
`Client -> Forward Proxy -> Internet (or Destination Server)`

## Key Use Cases
1.  **Privacy/Anonymity**: Hides the client's internal IP address from the internet. The destination sees the Proxy's IP.
2.  **Content Filtering**: Schools or workplaces use proxies to block access to certain websites (e.g., social media).
3.  **Bypassing Geo-Restrictions**: Accessing content restricted to a certain region by routing through a proxy in that region.
4.  **Caching**: Similar to reverse proxies, forward proxies can cache frequently accessed resources to save bandwidth for a local network.

## Forward vs Reverse Proxy
-   **Forward Proxy**: Protects/Serve the **Client**.
-   **Reverse Proxy**: Protects/Serve the **Server**.

## Setup Guide & Configuration (Squid Proxy)

### Installation
```bash
sudo apt install squid
```

### Overview of `squid.conf`
Squid configuration is located at `/etc/squid/squid.conf`. It is famous for being very large. It works using ACLs (Access Control Lists) and Access Directives.

### Rule Logic
Squid checks rules **top-down**. The first rule that matches determines the action (Allow or Deny).

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
Defines "groups" of things to match against.
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
Applies actions to ACLs.
*   **Syntax**: `http_access <allow|deny> <acl_name>`

**Example: Deny the Social Media ACL created above**
```squid
http_access deny social_media
```

### 3. `http_port`
*   `http_port 3128`: The standard proxy port.
*   `http_port 3128 intercept`: Used for "Transparent Proxying" (requires iptables redirection), where clients don't need to configure their browser manually.

### 4. Caching Directives
Control how usage is stored on disk/ram.

```squid
# cache_dir <storage_type> <directory> <size_MB> <L1_dirs> <L2_dirs>
cache_dir ufs /var/spool/squid 100 16 256
```
*   `ufs`: Standard unix file system storage.
*   `100`: Use 100MB of disk.
*   `16 256`: Directory structure depth for performance (don't change usually).

### 5. `visible_hostname`
If squid cannot detect the system hostname, it fails.
```squid
visible_hostname proxy.mycompany.com
```
