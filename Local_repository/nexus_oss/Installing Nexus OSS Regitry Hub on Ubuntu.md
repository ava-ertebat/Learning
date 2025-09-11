# Installing Nexus OSS Regitry Hub on Ubuntu
## Requirements:
1:Docker With Docker Composev2
2:Portainer
Installing Nexus OSS Regitry Hub on Ubuntu



Stage1:

Step 1:بروزرسانی سیستم:
```
sudo apt update && sudo apt upgrade -y
```
Step 2:Install Nexus_OSS certificates Requirements:
```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```
3:افزودن کلید GPG Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
4:افزودن ریپوزیتوری Docker
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5:نصب Docker
```
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

6:افزودن کاربر به گروه Docker (برای اجرای بدون sudo)
```
sudo usermod -aG docker $USER
newgrp docker
```
7:بررسی نصب
```
docker --version
```

Stage 2:Config Container

1:ایجاد volume برای داده‌های Portainer
```
docker volume create portainer_data
```
Then Set Docker Repository Hub
1:
```
sudo nano /etc/docker/daemon.json
```
2:ذخیره به کمک نانو
```
# اگر certificate error داشتی
{
  "registry-mirrors": ["https://registry.nichosting.ir"],
  "insecure-registries": ["registry.nichosting.ir"]
}
```
3:
```
sudo nano /etc/hosts
```
تنظیم آدرس لوکال برای ریپوزیتوری های رسمی
```
127.0.0.1    registry-1.docker.io
127.0.0.1    auth.docker.io
127.0.0.1    production.cloudflare.docker.com
```
4:
```
sudo systemctl restart docker
```
## enjoy
