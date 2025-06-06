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
```bash  
curl -sSL https://install.pi-hole.net | bash  
```  
During setup:  
Choose your network interface  
For upstream DNS provider, select “Custom” (you’ll add Unbound later)  

✅ Install Unbound (Local DNS Resolver)  
```bash  
sudo apt update  
sudo apt install unbound -y  
```  

✅ Install Cloudflared (Cloudflare DNS over HTTPS Proxy)  
```bash  
sudo apt install curl  
curl -s https://pkg.cloudflare.com/install.sh | sudo bash  
sudo apt install cloudflared -y  
```  

⚙️ Step 2: Configure Unbound for Local DNS Resolution  
🔧 Create Unbound config for Pi-hole:  
```bash  
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf  
```  
Paste this content:  
```  
server:  
    interface: 127.0.0.1  
    interface: ::1  
    port: 5335  
    access-control: 127.0.0.1/32 allow  
    access-control: ::1/128 allow  
    access-control: 10.0.0.0/8 allow  
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
    auto-trust-anchor-file: "/var/lib/unbound/root.key"  
    do-not-query-localhost: no  

forward-zone:  
    name: "."  
    forward-addr: 127.0.0.1@5053  
```  

🌐 Download Root DNS Hints:  
```bash  
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache  
```  

Restart and Enable Unbound:  
```bash  
sudo systemctl restart unbound  
sudo systemctl enable unbound  
```  

🛡️ Step 3: Configure Pi-hole to Use Unbound  
Visit the Pi-hole Admin Interface  
`http://localhost/admin`  

Go to Settings → DNS  
Uncheck all options under Upstream DNS Servers  
Add a custom DNS server:  
`127.0.0.1#5335`  
Click Save  

🌩️ Step 4: (Optional) Add Cloudflare DNS over HTTPS (DoH) via Cloudflared  
🔧 Run Cloudflared as a DNS proxy:  
```bash  
sudo cloudflared proxy-dns --port 5053 --upstream https://1.1.1.1/dns-query &  
```  

✅ Test DNS Resolution  
Test Unbound:  
```bash  
dig google.com @127.0.0.1 -p 5335  
```  
Test Cloudflared (DoH):  
```bash  
dig google.com @127.0.0.1 -p 5053  
```  
