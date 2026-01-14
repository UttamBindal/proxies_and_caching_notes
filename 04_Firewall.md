# Firewall

## What is a Firewall?
A firewall stands as the primary sentry of network security. It monitors and filters incoming and outgoing network traffic based on strict security policies. It creates a robust barrier between your trusted internal network and untrusted external networks, such as the Internet.

## Types of Firewalls
-   **Packet-filtering firewalls**: Examine packets in isolation and block or allow them based on IP/Port.
-   **Stateful inspection firewalls**: Track active connections and automatically allow return traffic.
-   **Proxy firewalls**: Act as a gateway by inspecting application-level traffic.
-   **Next-Generation Firewalls (NGFW)**: Combine traditional functions with deep packet inspection (DPI) and intrusion prevention systems (IPS).

## Key Concepts
-   **Inbound Rules**: Control traffic arriving AT your server.
-   **Outbound Rules**: Control traffic leaving FROM your server.
-   **Default Policy**: Establish a baseline, typically blocking all incoming traffic and allowing all outgoing traffic.

## Setup Guide & Configuration (UFW)

### Basic Commands
-   **Check Status**: `sudo ufw status`
-   **Allow/Deny**: `sudo ufw allow 80/tcp`, `sudo ufw deny from 192.168.1.5`
-   **Enable/Disable**: `sudo ufw enable`, `sudo ufw disable`

### Configuration Files
UFW simplifies iptables management as a command-line frontend, but it also relies on precise configuration files for default behaviors.

#### 1. `/etc/default/ufw`
Define high-level defaults in this file.

```bash
# /etc/default/ufw

IPV6=yes
DEFAULT_INPUT_POLICY="DROP"
DEFAULT_OUTPUT_POLICY="ACCEPT"
DEFAULT_FORWARD_POLICY="DROP"
```

*   **`IPV6=yes`**: Apply rules to IPv6. **Critical**: Enabling this prevents exposure via IPv6 even if you block IPv4.
*   **`DEFAULT_INPUT_POLICY="DROP"`**: Adopt the "deny all" strategy. Block everything unless you explicitly allow it (Safest approach).
*   **`DEFAULT_OUTPUT_POLICY="ACCEPT"`**: Allow your server to communicate with the outside world (e.g., `apt update`, `curl`).

#### 2. `/etc/ufw/before.rules`
Add raw generic `iptables` rules here to run *before* the UFW command-line rules. Use this for:
*   Allowing ICMP (Ping) requests (enabled by default).
*   Configuring NAT (Network Address Translation) for routing.

### Advanced Command Options

#### Rate Limiting (Brute Force Protection)
Use `limit` instead of `allow` to protect services like SSH.

```bash
sudo ufw limit ssh
# OR
sudo ufw limit 22/tcp
```
*   **Effect**: Deny connections from an IP address that attempts to initiate 6 or more connections within 30 seconds.

#### Logging
```bash
sudo ufw logging on
sudo ufw logging low
```
*   **Levels**: Select `low`, `medium`, `high`, or `full`.
*   **Location**: Review logs in `/var/log/ufw.log` (or kernel logs) to audit blocked attempts.

#### Port Ranges
Allow entire ranges instead of single ports.
```bash
sudo ufw allow 6000:6007/tcp
sudo ufw allow 6000:6007/udp
```

#### Specific Interface Binding
Restrict rules to a specific network interface (e.g., allow MySQL only on the private LAN interface `eth1`).

```bash
sudo ufw allow in on eth1 to any port 3306
```

#### Deleting Rules
Since rules follow a specific order, delete them by number for accuracy.

1.  List numbered rules:
    ```bash
    sudo ufw status numbered
    ```
2.  Delete rule number 3:
    ```bash
    sudo ufw delete 3
    ```
