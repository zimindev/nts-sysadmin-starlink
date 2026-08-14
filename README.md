# Starlink & Home Network — Complete Quick Reference

> A practical English-language cheat sheet covering Starlink, networking, MikroTik, VLANs, Wi-Fi, DNS, DHCP, Firewall, NAT, VPN, QoS, troubleshooting, and security.

---

## 1. Starlink Basics

**Starlink** provides Internet access through a satellite antenna and a Starlink router.

### Basic topology

```text
Satellite
   ↓
Starlink Dish
   ↓
Starlink Router
   ↓
Wi-Fi / Ethernet
   ↓
Devices
```

### Main components

- **Dish / Terminal** — communicates with Starlink satellites.
- **Router** — provides Wi-Fi and routing.
- **Power supply** — powers the system.
- **Ethernet** — connects Starlink to another router or network device.
- **Starlink App** — configuration, diagnostics, statistics, and status.

---

# 2. Starlink Installation

## Dish placement

The dish needs a clear view of the sky.

### Avoid

- Trees
- Tall buildings
- Roof structures
- Walls
- Poles or objects blocking the sky

### Prefer

- Open roof
- Open ground
- High mounting position
- Clear sky view

Obstructions can cause:

- Packet loss
- Short disconnects
- Higher latency
- Video-call interruptions
- Gaming problems

Use the Starlink App obstruction checker before final installation.

---

# 3. Starlink Router Modes

## Normal Mode

```text
Starlink
   ↓
Starlink Router
   ↓
Wi-Fi / LAN
```

The Starlink router handles:

- DHCP
- NAT
- Wi-Fi
- Routing
- Basic network management

## Bypass Mode

```text
Starlink
   ↓
Starlink Router
   ↓
Ethernet
   ↓
Your Router
   ↓
LAN / Wi-Fi / VLAN / VPN
```

Bypass Mode is useful when using:

- MikroTik
- Ubiquiti
- TP-Link
- ASUS
- OpenWrt
- pfSense
- OPNsense

### Important

**Bypass Mode does NOT automatically provide a public IPv4 address.**

If the Starlink connection uses CGNAT, CGNAT can still exist in Bypass Mode.

---

# 4. CGNAT

**CGNAT = Carrier-Grade NAT.**

It allows multiple customers to share public IPv4 addresses.

Typical topology:

```text
Home Router
    ↓
Starlink
    ↓
CGNAT
    ↓
Public Internet
```

### Why CGNAT matters

Incoming IPv4 connections may not work normally.

Potential problems:

- Port forwarding
- Hosting a game server
- Hosting a web server
- Direct SSH access
- Direct VPN server access
- Remote access to NAS

### Possible solutions

Depending on the service and configuration:

- IPv6
- Tailscale
- ZeroTier
- VPS + WireGuard
- VPN services with inbound connectivity

---

# 5. Home Network Architecture

A simple network:

```text
                 Starlink
                    │
                    ▼
              Main Router
                    │
             ┌──────┴──────┐
             │             │
           Wi-Fi         Ethernet
             │             │
       Phones / TV        PC / NAS
```

Advanced network:

```text
                    Starlink
                       │
                  Bypass Mode
                       │
                       ▼
                    MikroTik
                       │
          ┌────────────┼────────────┐
          │            │            │
       VLAN 10      VLAN 20      VLAN 30
        Main          IoT         Guest
          │            │            │
         PC           TV          Guests
         NAS          IoT
```

---

# 6. Recommended IP Plan

| VLAN | Purpose | Network | Gateway |
|---:|---|---|---|
| 10 | Main | `192.168.10.0/24` | `192.168.10.1` |
| 20 | IoT | `192.168.20.0/24` | `192.168.20.1` |
| 30 | Guest | `192.168.30.0/24` | `192.168.30.1` |
| 40 | CCTV | `192.168.40.0/24` | `192.168.40.1` |
| 50 | Management | `192.168.50.0/24` | `192.168.50.1` |
| — | WireGuard VPN | `10.10.10.0/24` | `10.10.10.1` |

Example devices:

```text
PC       → 192.168.10.10
NAS      → 192.168.10.20
Printer  → 192.168.10.30

TV       → 192.168.20.10

Camera   → 192.168.40.10

Router   → 192.168.50.1
```

---

# 7. VLAN

**VLAN = Virtual Local Area Network.**

VLANs divide one physical network into multiple logical networks.

Example:

```text
VLAN 10 → Trusted devices
VLAN 20 → IoT
VLAN 30 → Guests
VLAN 40 → Cameras
VLAN 50 → Management
```

### Why use VLANs?

Benefits:

- Better security
- Network isolation
- Easier management
- Reduced broadcast traffic
- Separate access policies

---

# 8. IoT

**IoT = Internet of Things.**

Examples:

- Smart TV
- Smart plugs
- Smart lights
- IP cameras
- Robot vacuum
- Smart thermostat
- Wi-Fi air conditioner
- Smart home sensors

Recommended:

```text
IoT → Internet       ALLOW
IoT → Main LAN       BLOCK
IoT → Management     BLOCK
IoT → NAS             BLOCK
```

Specific exceptions can be created when required.

---

# 9. Guest Network

Guest devices should normally have Internet access only.

```text
Guest → Internet       ALLOW
Guest → Main LAN       BLOCK
Guest → IoT            BLOCK
Guest → CCTV           BLOCK
Guest → Management     BLOCK
Guest → Router         BLOCK
```

Example:

```text
SSID: HOME-GUEST
VLAN: 30
Network: 192.168.30.0/24
```

---

# 10. CCTV Network

IP cameras can be isolated in their own VLAN.

```text
VLAN 40
192.168.40.0/24
```

Recommended policy:

```text
Camera → Main LAN       BLOCK
Camera → Management     BLOCK
Camera → NAS            ALLOW if required
Camera → Internet       ALLOW only if required
```

If cloud functionality is not needed, Internet access can be restricted.

---

# 11. Management VLAN

Network equipment can be placed in a dedicated management network.

```text
VLAN 50
192.168.50.0/24
```

Examples:

- MikroTik
- Managed switches
- Wi-Fi access points
- Controllers
- Network monitoring systems

Recommended:

```text
Main → Management       ALLOW for admins
IoT → Management        BLOCK
Guest → Management      BLOCK
Internet → Management   BLOCK
```

---

# 12. DHCP

**DHCP = Dynamic Host Configuration Protocol.**

It automatically provides devices with:

- IP address
- Subnet mask
- Gateway
- DNS server

Example:

```text
DHCP Pool:
192.168.10.100 - 192.168.10.254
```

A device can receive:

```text
IP:      192.168.10.105
Mask:    255.255.255.0
Gateway: 192.168.10.1
DNS:     192.168.10.1
```

Use DHCP reservations for important devices such as:

- NAS
- Printers
- Servers
- Cameras
- Access points

---

# 13. DNS

**DNS = Domain Name System.**

DNS converts domain names into IP addresses.

```text
google.com
     ↓
IP address
```

Common public DNS servers:

### Cloudflare

```text
1.1.1.1
1.0.0.1
```

### Google

```text
8.8.8.8
8.8.4.4
```

Recommended home architecture:

```text
Client
  ↓
MikroTik DNS
  ↓
Upstream DNS
  ↓
Internet
```

---

# 14. Firewall

The firewall controls traffic between networks.

Basic principle:

> Allow what is required. Block everything else.

### Basic policy

```text
Internet → LAN        BLOCK
Internet → Router     BLOCK
Guest → Main          BLOCK
Guest → IoT           BLOCK
IoT → Main            BLOCK
IoT → Management      BLOCK

Main → Internet       ALLOW
IoT → Internet        ALLOW
Guest → Internet      ALLOW
```

### Important firewall concepts

- `established`
- `related`
- `invalid`
- `input`
- `forward`
- `output`
- `accept`
- `drop`
- `reject`

A typical firewall should:

1. Accept established/related traffic.
2. Drop invalid traffic.
3. Protect the router itself.
4. Block unwanted WAN-to-LAN traffic.
5. Allow required LAN-to-WAN traffic.
6. Restrict inter-VLAN traffic.

---

# 15. NAT

**NAT = Network Address Translation.**

It allows private addresses to communicate with the Internet.

Example:

```text
192.168.10.10
      ↓
MikroTik NAT
      ↓
Starlink
      ↓
Internet
```

Typical MikroTik configuration uses:

```text
srcnat
action=masquerade
out-interface=WAN
```

---

# 16. Port Forwarding

Port forwarding sends incoming traffic to a specific internal device.

Example:

```text
Internet
   ↓
TCP 443
   ↓
Router
   ↓
192.168.10.20
   ↓
Web Server
```

However, Starlink CGNAT may prevent direct inbound IPv4 connections.

Do not assume that port forwarding on your router will work through CGNAT.

---

# 17. Wi-Fi

Recommended SSIDs:

```text
HOME
HOME-IOT
HOME-GUEST
```

Example:

```text
HOME       → VLAN 10
HOME-IOT   → VLAN 20
HOME-GUEST → VLAN 30
```

### Security

Prefer:

- WPA3 when supported
- WPA2 for compatibility
- Strong unique passwords

Avoid:

- Open Wi-Fi
- Weak passwords
- Shared administrative passwords

---

# 18. Ethernet vs Wi-Fi

For critical devices:

```text
PC
NAS
Gaming Console
Server
Workstation
```

Ethernet is usually preferable.

Advantages:

- Lower interference
- More stable latency
- Better consistency
- No wireless congestion

Example:

```text
PC
 ↓
Ethernet
 ↓
MikroTik
 ↓
Starlink
```

---

# 19. VPN

A VPN creates an encrypted connection between devices or networks.

Common technologies:

- WireGuard
- OpenVPN
- IPsec
- Tailscale
- ZeroTier

## WireGuard

Example VPN network:

```text
10.10.10.0/24
```

```text
MikroTik → 10.10.10.1
Phone    → 10.10.10.2
Laptop   → 10.10.10.3
```

Possible use cases:

- Remote access to NAS
- Remote access to home LAN
- Secure access on public Wi-Fi
- Remote administration
- Site-to-site networking

---

# 20. Starlink + VPN + CGNAT

If Starlink uses CGNAT, a directly hosted VPN server may not accept inbound IPv4 connections.

Alternatives:

```text
Phone
  ↓
Tailscale / ZeroTier
  ↓
Home Network
```

or:

```text
Home
  ↓
WireGuard
  ↓
VPS with Public IP
  ↓
Remote Device
```

---

# 21. QoS

**QoS = Quality of Service.**

QoS controls bandwidth and traffic priority.

Example:

```text
VoIP       → High
Gaming     → High
DNS        → High
Web        → Normal
Streaming  → Normal
Downloads  → Low
```

QoS can help when the connection is saturated.

For Starlink, configure limits below the stable measured bandwidth rather than using a temporary maximum Speedtest result.

---

# 22. Gaming

Gaming depends more on:

- Latency
- Jitter
- Packet loss
- Stability
- Bufferbloat

than on raw download speed.

Example:

```text
Download: 150 Mbps
Upload:    20 Mbps
Ping:      45 ms
Loss:       0%
```

can be better for gaming than a faster but unstable connection.

Use Ethernet when possible.

---

# 23. Network Testing

## Ping

Linux / macOS:

```bash
ping 1.1.1.1
```

Windows:

```cmd
ping 1.1.1.1
```

100 packets on Linux:

```bash
ping -c 100 1.1.1.1
```

## Traceroute

Linux:

```bash
traceroute 1.1.1.1
```

Windows:

```cmd
tracert 1.1.1.1
```

## DNS test

Linux:

```bash
nslookup google.com
```

or:

```bash
dig google.com
```

Windows:

```cmd
nslookup google.com
```

---

# 24. Packet Loss

Packet loss means packets do not successfully reach their destination.

Example:

```text
0% packet loss  → Excellent
1% packet loss  → Noticeable in some applications
5% packet loss  → Problematic
10%+            → Serious problem
```

For gaming, VoIP, and remote work, even small amounts of packet loss can be noticeable.

---

# 25. Latency

Latency is the time required for packets to travel between endpoints.

Typical interpretation:

```text
< 20 ms     Excellent
20–50 ms    Very good
50–100 ms   Acceptable
100–150 ms  Noticeable
150+ ms     High
```

Actual acceptable latency depends on the application and destination server.

Satellite Internet can have higher and more variable latency than fiber.

---

# 26. Jitter

**Jitter** is variation in latency.

Example:

```text
Ping:
40 ms
42 ms
41 ms
43 ms
```

Low jitter.

But:

```text
35 ms
90 ms
45 ms
150 ms
```

High jitter.

High jitter can affect:

- Gaming
- VoIP
- Video calls
- Remote desktop

---

# 27. Starlink Troubleshooting

## Internet is offline

Check:

1. Power
2. Starlink cable
3. Router status
4. Dish status
5. App status
6. Obstructions
7. Starlink service status

---

## Frequent disconnects

Check:

- Obstructions
- Dish position
- Cable connections
- Power stability
- Router temperature
- Starlink status
- Packet loss

---

## Slow Wi-Fi

Test using Ethernet.

If Ethernet is fast but Wi-Fi is slow, investigate:

- Distance
- Interference
- Wi-Fi channel
- 2.4 GHz congestion
- 5 GHz coverage
- Access point placement
- Client device limitations

---

# 28. Starlink Power Backup

Possible setup:

```text
Grid
 ↓
UPS
 ↓
Starlink
 ↓
Router
```

Or:

```text
Battery
   ↓
Inverter
   ↓
Starlink + Router
```

For backup power, consider:

- Starlink power consumption
- Router consumption
- UPS capacity
- Battery capacity
- Inverter efficiency
- Runtime requirements

Approximate battery energy:

```text
Energy (Wh) = Voltage (V) × Capacity (Ah)
```

Actual usable energy is lower because of losses and battery limitations.

---

# 29. MikroTik Basic Structure

A typical MikroTik setup:

```text
ether1 → WAN / Starlink

bridge-lan
 ├── ether2
 ├── ether3
 ├── ether4
 └── ether5

VLAN 10 → Main
VLAN 20 → IoT
VLAN 30 → Guest
VLAN 40 → CCTV
VLAN 50 → Management
```

RouterOS components commonly used:

```text
/interface
/interface bridge
/interface vlan
/ip address
/ip dhcp-client
/ip dhcp-server
/ip dns
/ip firewall filter
/ip firewall nat
/interface wireguard
/queue
```

---

# 30. Recommended Security Model

Use the following trust model:

```text
             TRUST LEVEL

Main LAN        ██████████
Management      ██████████
CCTV            ████
IoT             ███
Guest           ██
Internet        0
```

Main principles:

- Do not expose management interfaces to the Internet.
- Use strong passwords.
- Keep router firmware updated.
- Keep Wi-Fi encrypted.
- Separate IoT devices.
- Separate guests.
- Minimize open ports.
- Disable unused services.
- Use VPN for remote administration.
- Keep backups of router configuration.

---

# 31. Useful Network Commands

## Windows

```cmd
ipconfig
ipconfig /all
ping 1.1.1.1
tracert 1.1.1.1
nslookup google.com
arp -a
```

## Linux

```bash
ip addr
ip route
ping 1.1.1.1
traceroute 1.1.1.1
dig google.com
ss -tulpn
arp -n
```

---

# 32. Final Recommended Architecture

```text
                              🛰️
                           STARLINK
                              │
                              │
                       [ BYPASS MODE ]
                              │
                              ▼
                     ┌─────────────────┐
                     │     MikroTik    │
                     │                 │
                     │ WAN             │
                     │ NAT             │
                     │ Firewall        │
                     │ DHCP            │
                     │ DNS             │
                     │ VLAN            │
                     │ VPN             │
                     │ QoS             │
                     └────────┬────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
          VLAN 10          VLAN 20          VLAN 30
           MAIN              IoT             GUEST
             │                │                │
        ┌────┼────┐       ┌───┼───┐        Guests
        │    │    │       │   │   │
       PC   NAS  Phone    TV  IoT  Smart
             │
             ▼
          Wi-Fi AP

                         VLAN 40
                           CCTV
                             │
                          Cameras

                         VLAN 50
                       MANAGEMENT
                             │
                    Network Equipment
```

---

# 33. Quick Checklist

## Starlink

- [ ] Dish has clear sky visibility
- [ ] Cables are properly connected
- [ ] Power is stable
- [ ] Starlink App shows Online
- [ ] Obstructions have been checked
- [ ] Firmware is up to date

## Wi-Fi

- [ ] Strong Wi-Fi password
- [ ] WPA2/WPA3 enabled
- [ ] Guest network separated
- [ ] IoT network separated
- [ ] Wi-Fi AP positioned correctly

## MikroTik

- [ ] WAN configured
- [ ] LAN configured
- [ ] DHCP configured
- [ ] DNS configured
- [ ] NAT configured
- [ ] Firewall configured
- [ ] VLANs configured
- [ ] Management isolated
- [ ] Router backup created

## Security

- [ ] Strong administrator password
- [ ] Unused services disabled
- [ ] Remote administration protected
- [ ] IoT isolated
- [ ] Guest network isolated
- [ ] Unnecessary ports closed
- [ ] Firmware updated

## Performance

- [ ] Ethernet tested
- [ ] Wi-Fi tested
- [ ] Download speed tested
- [ ] Upload speed tested
- [ ] Ping tested
- [ ] Packet loss tested
- [ ] Jitter tested
- [ ] QoS considered if the connection saturates

---

# 34. Key Terms

| Term | Meaning |
|---|---|
| Starlink | Satellite Internet service |
| Dish / Terminal | Starlink satellite antenna |
| Router | Network device that routes traffic |
| WAN | Wide Area Network / Internet side |
| LAN | Local Area Network |
| WLAN | Wireless Local Area Network |
| VLAN | Virtual Local Area Network |
| IoT | Internet of Things |
| DHCP | Automatically assigns network configuration |
| DNS | Converts domain names to IP addresses |
| NAT | Translates private/public addresses |
| CGNAT | Carrier-Grade NAT used by providers |
| Firewall | Controls network traffic |
| VPN | Encrypted network connection |
| WireGuard | Modern VPN protocol |
| QoS | Quality of Service / traffic prioritization |
| SSID | Wi-Fi network name |
| IP | Internet Protocol address |
| Gateway | Device used to reach other networks |
| DNS Server | Server that resolves domain names |
| Packet Loss | Packets that fail to reach their destination |
| Latency | Network response delay |
| Jitter | Variation in latency |
| Bypass Mode | Starlink router routing disabled |

---

# 35. Golden Rules

1. **Keep the Starlink dish unobstructed.**
2. **Use Ethernet for critical devices whenever possible.**
3. **Use Bypass Mode when a dedicated router should control the network.**
4. **Do not confuse Bypass Mode with a public IP.**
5. **Assume CGNAT can prevent inbound IPv4 connections.**
6. **Separate IoT devices from trusted computers.**
7. **Keep Guest devices isolated from the home LAN.**
8. **Protect management interfaces.**
9. **Use a default-deny firewall strategy for unwanted traffic.**
10. **Use VPN instead of exposing unnecessary services to the Internet.**
11. **Monitor packet loss and latency, not only download speed.**
12. **Back up router configuration before major changes.**
13. **Update firmware and software regularly.**
14. **Document IP addresses, VLANs, and network rules.**
15. **Change one major network setting at a time and test after each change.**

---

## Conclusion

A well-designed Starlink network can be much more than a simple Wi-Fi connection.

A robust setup can use:

```text
Starlink
   +
Bypass Mode
   +
MikroTik
   +
VLAN
   +
Firewall
   +
DHCP
   +
DNS
   +
NAT
   +
VPN
   +
QoS
   +
Monitoring
```

This architecture provides better control, isolation, security, and troubleshooting capabilities while keeping Starlink as the Internet uplink.
