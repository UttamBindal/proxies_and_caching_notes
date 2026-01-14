# Firewall

## What is a Firewall?
A firewall is a network security device that monitors and filters incoming and outgoing network traffic based on an organization's previously established security policies. It stands as a barrier between a trusted internal network and an untrusted external network, such as the Internet.

## Types of Firewalls
-   **Packet-filtering firewalls**: Examine packets in isolation and block/allow based on IP/Port.
-   **Stateful inspection firewalls**: Track existing connections and allow return traffic automatically.
-   **Proxy firewalls**: Act as a gateway, inspecting application-level traffic.
-   **Next-Generation Firewalls (NGFW)**: Combine traditional firewall functions with other network device filtering (DPI, IPS).

## Key Concepts
-   **Inbound Rules**: Control traffic coming IN to your server.
-   **Outbound Rules**: Control traffic going OUT from your server.
-   **Default Policy**: Usually, block all incoming, allow all outgoing.

## Setup Guide & Configuration (UFW)

### Basic Commands
-   Check Status: `sudo ufw status`
-   Allow/Deny: `sudo ufw allow 80/tcp`, `sudo ufw deny from 192.168.1.5`
-   Enable/Disable: `sudo ufw enable`, `sudo ufw disable`

### Configuration Files
While UFW is a command-line frontend for iptables, it has configuration files that control its default behavior.

#### 1. `/etc/default/ufw`
This file controls the high-level defaults.

```bash
# /etc/default/ufw

IPV6=yes
DEFAULT_INPUT_POLICY="DROP"
DEFAULT_OUTPUT_POLICY="ACCEPT"
DEFAULT_FORWARD_POLICY="DROP"
```

*   **`IPV6=yes`**: Ensures your rules apply to IPv6 as well. **Critical**: If disabled, your server might be exposed via IPv6 even if IPv4 is blocked.
*   **`DEFAULT_INPUT_POLICY="DROP"`**: The "deny all" strategy. Blocks everything unless you explicitly allow it (Safest).
*   **`DEFAULT_OUTPUT_POLICY="ACCEPT"`**: Allows your server to talk to the outside world (e.g., `apt update`, `curl`).

#### 2. `/etc/ufw/before.rules`
Allows you to add raw generic `iptables` rules that run *before* the UFW command-line rules. Useful for:
*   Allowing ICMP (Ping) requests (enabled by default).
*   NAT (Network Address Translation) configuration for routing.

### Advanced Command Options

#### Rate Limiting (Brute Force Protection)
Instead of just `allow`, uses `limit`. This is excellent for SSH.

```bash
sudo ufw limit ssh
# OR
sudo ufw limit 22/tcp
```
*   **Effect**: Denies connections from an IP address that has attempted to initiate 6 or more connections in the last 30 seconds.

#### Logging
```bash
sudo ufw logging on
sudo ufw logging low
```
*   **Levels**: `low`, `medium`, `high`, `full`.
*   Logs are written to `/var/log/ufw.log` (or kernel logs). Useful for auditing blocked attempts.

#### Port Ranges
You can allow ranges instead of single ports.
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
Since rules are added in order, it's often easier to delete by number.

1.  List numbered rules:
    ```bash
    sudo ufw status numbered
    ```
2.  Delete rule number 3:
    ```bash
    sudo ufw delete 3
    ```
