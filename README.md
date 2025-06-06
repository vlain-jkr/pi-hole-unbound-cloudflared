# pi-hole-unbound-cloudflared
Home DNS Server

# 🛡️ Pi-hole + Unbound + Cloudflared DNS Firewall

Secure your entire network with recursive DNS resolution, DNSSEC, and DNS-over-HTTPS fallback.

---

## 🔧 What This Project Does

- Blocks ads, trackers, and telemetry at the network level with **Pi-hole**
- Resolves DNS queries **locally and privately** with **Unbound**
- Falls back to **Cloudflare DNS over HTTPS** using **Cloudflared** only when recursion fails
- DNSSEC-enabled, cache-optimized, and leak-resistant setup

---

## 🧰 Prerequisites

- Ubuntu-based system
- Terminal access with sudo privileges

---

## 📦 Install and Configure

✅ Install Pi-hole
<pre>
curl -sSL https://install.pi-hole.net | bash
</pre>

During setup:
Choose your network interface
For upstream DNS provider, select “Custom” (you’ll add Unbound later)
✅ Install Unbound (Local DNS Resolver)
<pre>
sudo apt update
sudo apt install unbound -y
</pre>


✅ Install Cloudflared (Cloudflare DNS over HTTPS Proxy)
<pre>
sudo apt install curl
curl -s https://pkg.cloudflare.com/install.sh | sudo bash
sudo apt install cloudflared -y
</pre>

⚙️ Step 2: Configure Unbound for Local DNS Resolution
🔧 Create Unbound config for Pi-hole:
<pre>
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
</pre>
Paste this content:
server:
    <pre>
    # Listen only on localhost for security
    interface: 127.0.0.1
    interface: ::1
    # Pi-hole listens on port 5335
    port: 5335
    # Access control (Allow only local network)
    access-control: 127.0.0.1/32 allow
    access-control: ::1/128 allow
    access-control: 10.0.0.0/8 allow
    # Security settings
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no
    cache-min-ttl: 3600
    cache-max-ttl: 86400
    prefetch: yes
    # DNSSEC
    auto-trust-anchor-file: "/var/lib/unbound/root.key"
    # Fallback only if recursive fails
    do-not-query-localhost: no

# ✅ Backup: Use local Cloudflared instance for DoH if recursion fails
forward-zone:
    name: "."
    forward-addr: 127.0.0.1@5053  # Cloudflared DoH
</pre>

🌐 Download Root DNS Hints:
<pre>
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
</pre>

Restart and Enable Unbound:
<pre>
sudo systemctl restart unbound
sudo systemctl enable unbound
</pre>

🛡️ Step 3: Configure Pi-hole to Use Unbound
Visit the Pi-hole Admin Interface
# Enter IP Address of the Device
<pre>
http://localhost/admin
</pre>

Go to Settings → DNS
Under Upstream DNS Servers, uncheck all options
Add a custom DNS server:
<pre>
127.0.0.1#5335
</pre>
Click Save

🌩️ Step 4: (Optional) Add Cloudflare DNS over HTTPS (DoH) via Cloudflared
🔧 Run Cloudflared as a DNS proxy:
<pre>
sudo cloudflared proxy-dns --port 5053 --upstream https://1.1.1.1/dns-query &
</pre>

🛠️ To run as a system service:


✅ Test DNS Resolution
Test Unbound:
<pre>
dig google.com @127.0.0.1 -p 5335
</pre>

Test Cloudflared (DoH):
<pre>
dig google.com @127.0.0.1 -p 5053
</pre>
