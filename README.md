# Secure Remote Access for Homelab (Zero Trust Architecture)

## 📌 Project Overview
This project implements a **Zero Trust Network Access (ZTNA)** solution to provide secure, remote management of an on-premise Proxmox Virtualization Environment.

By utilizing an outbound **Cloudflare Tunnel**, this architecture completely eliminates the need for traditional port forwarding. The home network remains completely closed to inbound traffic from the internet, bypassing Carrier-Grade NAT (CGNAT) issues from ISPs. Access to the infrastructure is protected by an **Identity-Aware Proxy (IAP)** enforcing an Email One-Time PIN (OTP) policy.

## 🏗 Architecture Flow
`User Device` -> `Cloudflare Edge (WAF + DDoS Protection)` -> `Identity Access Policy (Email OTP)` -> `Encrypted Tunnel (cloudflared)` -> `Debian LXC Connector` -> `Proxmox Web UI`

## 🛠 Prerequisites
* Proxmox Virtual Environment (VE) running locally.
* A registered domain name.
* A free Cloudflare account with your domain configured and active.

---

## 🚀 Step-by-Step Implementation Guide

### Step 1: Establish the Cloudflare Tunnel
Instead of opening ports on a router, we create an outbound connection from the server to Cloudflare.

1. Log in to the **Cloudflare Zero Trust Dashboard**.
2. Navigate to **Zero Trust > Network > Tunnels** on the left sidebar.
3. Click **Create a tunnel**, name it (e.g., `Homelab-Tunnel`), and select `cloudflared` as the connector type.
4. Cloudflare will provide an installation command.
5. In your Proxmox environment, spin up a lightweight Debian LXC container.
6. Open the console for that container and paste the Cloudflare installation command. 

> 📸 **[Screenshot Placeholder:** Show the Proxmox LXC console displaying the successful installation of the `cloudflared` service. Mark the "Success" message with a green box. **]**

Once the connector is running, the tunnel status in Cloudflare will show as **Healthy**.

### Step 2: Route Traffic to the First Proxmox Node
Now we tell Cloudflare where to send the traffic when someone visits your domain.

1. In the Cloudflare Zero Trust Dashboard, go to **Networks > Tunnels**.
2. Click the three dots next to your healthy tunnel and select **Configure**.
3. Select the **Public Hostname** tab at the top.
4. Click the blue **Add a public hostname** button.
5. Fill in the required details:
   * **Subdomain:** Enter your desired prefix (e.g., `prox`).
   * **Domain:** Select your domain from the dropdown.
   * **Path:** Leave this entirely blank.
   * **Service Type:** Select **HTTPS** (Crucial for Proxmox).
   * **URL:** Enter the local IP and port of your Proxmox server (e.g., `10.0.0.213:8006`). Do not type `https://` in this box.

> 📸 **[Screenshot Placeholder:** Show the "Public Hostname" form filled out with the subdomain, domain, HTTPS type, and local IP. **]**

6. **Crucial Proxmox Setting:** Because Proxmox uses a self-signed SSL certificate locally, you must tell Cloudflare to accept it.
   * Scroll down and click to expand **Additional application settings**.
   * Click **TLS**.
   * Toggle **No TLS Verify** to the **ON** position.
7. Click **Save hostname**.

> 📸 **[Screenshot Placeholder:** Show the expanded TLS settings menu with the "No TLS Verify" toggle switched on. Mark the toggle with a red arrow. **]**

### Step 3: Enforce Identity & Access Policies (MFA)
At this point, the URL is live. We must secure it so that only authorized users can view the login page.

1. In the Zero Trust Dashboard, navigate to **Access > Applications**.
2. Click **Add an Application** and select **Self-hosted**.
3. **Application Configuration:**
   * **Application Name:** e.g., "Proxmox Access".
   * **Session Duration:** Set to **24 hours** (This requires you to verify your identity once per day).
   * **Application Domain:** Enter the exact subdomain and domain you created in Step 2.
   * Click **Next**.

> 📸 **[Screenshot Placeholder:** Show the "Basic Information" screen of the application setup, highlighting the Session Duration and Application Domain. **]**

4. **Add a Policy:**
   * **Policy Name:** e.g., "Allow Admin Email".
   * **Action:** Set to **Allow**.
   * Under the **Configure rules** section, set the Selector to **Emails**.
   * In the **Value** box, type your exact email address.
5. Click **Next**, leave the remaining settings as default, and click **Add application**.

> 📸 **[Screenshot Placeholder:** Show the Policy configuration screen with the "Emails" selector and the specific email address entered. Mark the rules section with a red box. **]**

### Step 4: If You Want To Add Additional Nodes to the Same Tunnel
A single `cloudflared` connector can route traffic to multiple devices on the same local network. You do not need multiple tunnels.

To add a second Proxmox node (or any other service):
1. Navigate back to **Zero Trust > Network > Tunnels > Configure**.
2. Go to the **Public Hostname** tab and click **Add a public hostname**.
3. Enter a new subdomain (e.g., `lab2`).
4. Select **HTTPS** and enter the local IP and port of the *second* system (e.g., `10.0.0.216:8006`).
5. Expand **Additional application settings > TLS** and turn **ON** the **No TLS Verify** toggle.
6. Click **Save hostname**.
7. Create a new Application Policy under **Access controls > Applications** for this new subdomain following the exact steps in Step 3.

> 📸 **[Screenshot Placeholder:** Show the main Tunnel overview screen displaying multiple routes/subdomains correctly pointing to different internal IP addresses. **]**

## ✅ Verification and Testing
To verify the setup is working correctly:
1. Open a new **Incognito/Private** browser window.
2. Navigate to your secure URL (`https://prox.yourdomain.com`).
3. You will be intercepted by a Cloudflare login screen requesting an email address.
4. Input your authorized email, retrieve the 6-digit OTP from your inbox, and enter it.
5. You will instantly be securely routed to your local Proxmox login screen.
