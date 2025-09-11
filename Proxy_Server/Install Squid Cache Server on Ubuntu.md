# Install Squid Cache Server on Ubuntu.md
# Squid Transparent Proxy Setup Behind MikroTik Router
# Purpose: Enable HTTP caching for all internet traffic transparently in a company network
# Author: OpenAI ChatGPT

# 1. System Requirements
# - Ubuntu Server 22.04 LTS
# - MikroTik router (any model with NAT support)
# - Static IP for Squid server (e.g. 192.168.1.10)
# - Squid server and clients must be in the same LAN

# 2. Install Squid on Ubuntu
```
sudo apt update
sudo apt install squid -y
```
# 3. Configure Squid for Transparent Proxy
```
sudo nano /etc/squid/squid.conf
```
# Replace or append the following config:
```
http_port 3128 intercept

cache_mem 1024 MB
maximum_object_size 1024 MB
minimum_object_size 0 KB
cache_dir ufs /var/spool/squid 500000 16 256

acl localnet src 192.168.1.0/24
http_access allow localnet
http_access deny all

access_log /var/log/squid/access.log squid
```
# Save and exit: CTRL+O, ENTER, CTRL+X

# Initialize cache and restart Squid
```
sudo squid -z
sudo systemctl restart squid
```

# 4. Redirect Incoming Traffic to Squid (on Ubuntu)
# Find your interface name (example: ens160, enp0s3, etc.)
```
ip a
```
# Replace "eth0" below with your actual interface name
```
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3128
```
# (Optional) Install iptables-persistent to make rules permanent:
# sudo apt install iptables-persistent

# 5. MikroTik NAT Rule (Redirect HTTP to Squid)
# In Winbox or MikroTik CLI:

# IP > Firewall > NAT > Add new rule
# General Tab:
# - Chain: dstnat
# - Protocol: tcp
# - Dst Port: 80
# - In Interface: ether2 (or your LAN interface)
# Action Tab:
# - Action: dst-nat
# - To Address: 192.168.1.10 (Squid IP)
# - To Ports: 3128
# Save and Apply

# 6. Testing
# On a client computer, DO NOT set proxy manually.
# Browse any HTTP website (not HTTPS).
# Then run this on Squid server:
sudo tail -f /var/log/squid/access.log

# You should see access logs with HIT or MISS

# 7. (Optional) Install Web UI - lightsquid
sudo apt install lightsquid -y

# Then access via browser:
# http://<your-server-ip>/lightsquid/

# 8. Security: UFW Firewall Configuration
# Allow Squid only from MikroTik IP (e.g. 192.168.1.1)
sudo ufw allow from 192.168.1.1 to any port 3128

# Enable UFW if not already enabled
sudo ufw enable

# 9. Notes & Future Steps
# - This setup handles only HTTP (port 80). HTTPS interception requires ssl_bump (advanced).
# - Do not apply to port 443 unless you configure SSL cert injection.
# - Consider log rotation, fail2ban, and monitoring if running long-term.

# Done! Your Squid transparent proxy is now ready.
# Save this script as setup_squid_transparent_proxy.sh or a .txt for reference.
