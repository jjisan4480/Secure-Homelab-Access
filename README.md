# Secure Remote Access for Homelab (Zero Trust Architecture)

##  Project Overview
This project implements a **Zero Trust Network Access (ZTNA)** solution to provide secure, remote management of an on-premise Proxmox Virtualization Environment.

Instead of using traditional, insecure methods like Port Forwarding or exposing public IP addresses, this architecture utilizes an encrypted **Cloudflare Tunnel**. This ensures the home network remains completely closed to inbound traffic while allowing authorized access via an **Identity-Aware Proxy (IAP)**.

##  Architecture
**Traffic Flow:**
`User Device` -> `Cloudflare Edge (WAF + DDoS Protection)` -> `Identity Access Policy (MFA/Email)` -> `Encrypted Tunnel (cloudflared)` -> `Proxmox LXC Container` -> `Proxmox Web UI`

##  Technologies Used
* **Hypervisor:** Proxmox Virtual Environment (VE)
* **Networking:** Cloudflare Zero Trust (Tunnels), DNS Management
* **Security:** Cloudflare Access (Identity Provider), SSL/TLS Offloading
* **OS/Containerization:** Debian 12 (LXC Container)
* **Infrastructure:** Home Server (Bare Metal)

##  Key Features
* **Zero Open Ports:** No inbound ports (80/443) are opened on the ISP router, eliminating common attack vectors.
* **Identity-Aware Auth:** Integrated Cloudflare Access to enforce Single Sign-On (SSO) via Email OTP before the Proxmox login page is ever loaded.
* **CGNAT Bypass:** Successfully routed traffic through a carrier-grade NAT (ISP) without requiring a static Public IP.
* **Automatic HTTPS:** Full SSL/TLS encryption managed at the edge, removing the need for local certificate management.

##  Implementation Highlights
1.  **Infrastructure Prep:** Deployed a lightweight Debian LXC container within Proxmox to host the tunnel connector.
2.  **Tunnel Deployment:** Configured `cloudflared` daemon to establish a persistent outbound connection to Cloudflare's edge network.
3.  **Routing:** Mapped internal private IP (`192.168.x.x:8006`) to a public FQDN (`prox.mydomain.com`).
4.  **Security Policy:** Created a "Zero Trust" policy requiring specific email verification for access, effectively cloaking the server from public internet scans.
