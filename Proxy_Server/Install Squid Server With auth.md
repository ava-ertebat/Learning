# Install Squid Server With auth.md

Step 1:
```
sudo apt update -y
```
Step 2:
```
sudo apt install nano -y && sudo apt install squid -y
```
Step 3:
```
sudo apt install apache2-utils
```
Step 4:
```
htpasswd -c /etc/squid/passwords swordfish
```
Step 5:
```
sudo mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
sudo nano /etc/squid/squid.conf
```
Step 6:use This Config on /etc/squid/squid.conf
```
sudo nano /etc/squid/squid.conf
```
Then Copy This Config And Replace With Them
```
acl snmp_net src 127.0.0.1
acl snmp_r snmp_community squid_proxy
snmp_port 3401
snmp_access allow snmp_r snmp_net
snmp_access deny all
cache_effective_user proxy
acl localnet src 0.0.0.1-0.255.255.255	# RFC 1122 "this" network (LAN)
acl localnet src 10.0.0.0/8		# RFC 1918 local private network (LAN)
acl localnet src 100.64.0.0/10		# RFC 6598 shared address space (CGN)
acl localnet src 169.254.0.0/16 	# RFC 3927 link-local (directly plugged) machines
acl localnet src 172.16.0.0/12		# RFC 1918 local private network (LAN)
acl localnet src 192.168.0.0/16		# RFC 1918 local private network (LAN)
acl localnet src fc00::/7       	# RFC 4193 local private network range
acl localnet src fe80::/10      	# RFC 4291 link-local (directly plugged) machines
acl SSL_ports port 443
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
http_access allow localhost
include /etc/squid/conf.d/*.conf
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm Proxy Authentication
auth_param basic credentialsttl 2 hours
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
http_access allow localnet
http_access allow localhost
http_access deny all
http_port 0.0.0.0:48072
access_log /var/log/squid/access.log squid
cache_mem 1024 MB
##########################################################################
#Google Gemini
# وقتی این حجم پر شد، به طور خودکار محتوای قدیمی را پاک کن (رفتار پیش‌فرض)
cache_dir ufs /var/spool/squid 460800 16 256
# این خط به Squid می‌گوید که تاریخ انقضای تعیین شده توسط سرورها را نادیده بگیر
# و فایل‌های استاتیک را برای مدت بسیار طولانی در کش نگه دار
refresh_pattern -i \.(jpg|jpeg|gif|png|ico|mp4|mov|avi|wmv|mkv|flv|swf|css|js)$ 10080 95% 43200 override-expire override-lastmod reload-into-ims ignore-no-cache ignore-private
# این یک قانون کلی برای هر چیز دیگری است که با الگوی بالا مطابقت ندارد
refresh_pattern . 1440 90% 43200 override-expire
##########################################################################
maximum_object_size 1024 MB
coredump_dir /var/spool/squid
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern \/(Packages|Sources)(|\.bz2|\.gz|\.xz)$ 0 0% 0 refresh-ims
refresh_pattern \/Release(|\.gpg)$ 0 0% 0 refresh-ims
refresh_pattern \/InRelease$ 0 0% 0 refresh-ims
refresh_pattern \/(Translation-.*)(|\.bz2|\.gz|\.xz)$ 0 0% 0 refresh-ims
```

Step 7:
```
sudo systemctl restart squid
```
Step 8:test squid with curl
```
curl -v -x http://server1.osmm.ir:48072 -U 'swordfish:ecsxD5QpQGI1SeC' https://chatgpt.com/
```
## Enjoy
