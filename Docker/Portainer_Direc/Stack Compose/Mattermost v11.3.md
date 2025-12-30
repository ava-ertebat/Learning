# Mattermost v11.3 Deployment Guide (Portainer Edition)

This guide outlines how to deploy **Mattermost Team Edition v11.3** using **Portainer**.
It includes critical configuration fixes for **PostgreSQL 16 compatibility**, **WebSocket connections**, and **Secure Cookie handling** to prevent login loops when running behind a Reverse Proxy (like Nginx).

---

## Prerequisites

1.  **Portainer** installed and running.
2.  **Nginx** (or Nginx Proxy Manager) configured with a valid SSL certificate for your domain.
3.  A domain name (e.g., `chat.example.com`).

---

## Step 1: Deploying the Stack

1.  Log in to your **Portainer** dashboard.
2.  Go to **Stacks** and click **+ Add stack**.
3.  Name the stack (e.g., `mattermost`).
4.  Copy and paste the following configuration into the **Web editor**.

> **⚠️ Important:** Before deploying, replace `https://chat.example.com` with your actual domain and change `Secure_DB_Password` and `Secure_Admin_Password` to strong passwords.

```yaml
version: '3.8'

services:
  db_avaertebat:
    # طبق جدول نسخه ۱۱.۳، پستگرس 16 توصیه شده است
    image: docker.arvancloud.ir/postgres:16-alpine
    container_name: db_avaertebat
    restart: unless-stopped
    volumes:
      - postgres_data_avaertebat:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=avaertebat_mm_user
      - POSTGRES_PASSWORD=Swordfish641@
      - POSTGRES_DB=avaertebat_mattermost_db
    # اضافه کردن هلث‌چک برای اطمینان از سلامت دیتابیس
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U avaertebat_mm_user -d avaertebat_mattermost_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  mattermost_avaertebat:
    # استفاده از نسخه ۱۱.۳ درخواستی شما
    image: docker.arvancloud.ir/mattermost/mattermost-team-edition:release-11.3
    container_name: mattermost_avaertebat
    restart: unless-stopped
    depends_on:
      db_avaertebat:
        condition: service_healthy
    ports:
      # پورت استاندارد را مپ کردیم
      - "8065:8065"
    volumes:
      - mattermost_config_avaertebat:/mattermost/config:rw
      - mattermost_data_avaertebat:/mattermost/data:rw
      - mattermost_logs_avaertebat:/mattermost/logs:rw
      - mattermost_plugins_avaertebat:/mattermost/plugins:rw
      - mattermost_client_plugins_avaertebat:/mattermost/client/plugins:rw
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      # نکته مهم: کاراکتر @ در پسورد به %40 تبدیل شد تا لینک خراب نشود
      - MM_SQLSETTINGS_DATASOURCE=postgres://avaertebat_mm_user:Swordfish641%40@db_avaertebat:5432/avaertebat_mattermost_db?sslmode=disable&connect_timeout=10

      # --- تنظیمات آدرس ---
      - MM_SERVICESETTINGS_SITEURL=https://chat.avacore.ir
      
      # --- تنظیمات حیاتی برای Nginx (جلوگیری از لوپ رفرش) ---
      # این دو خط در داکیومنت قدیمی نبود اما برای نسخه جدید و انجین‌اکس الزامی است
      - MM_SERVICESETTINGS_WEBSOCKETURL=wss://chat.avacore.ir
      - MM_SERVICESETTINGS_TRUSTEDPROXYIP=0.0.0.0/0
      
      # --- تنظیمات امنیتی کوکی (چون HTTPS دارید) ---
      - MM_SERVICESETTINGS_SESSIONCOOKIESECURE=true
      
      # یوزر ادمین اولیه
      - MM_ADMIN_USERNAME=ava-ertebat
      - MM_ADMIN_PASSWORD=Swordfish641@
      
      # --- تنظیمات پلاگین‌ها (اصلاح شده برای رفع خطای آپلود) ---
      - MM_PLUGINSETTINGS_ENABLE=true
      # این خط اجازه می‌دهد فایل را از کامپیوتر خودت آپلود کنی:
      - MM_PLUGINSETTINGS_ENABLEUPLOADS=true
      # این خط بررسی امضای دیجیتال را برای پلاگین‌های دستی خاموش می‌کند تا خطا ندهد:
      - MM_PLUGINSETTINGS_REQUIREPLUGINSIGNATURE=false
      
      # تنظیمات قبلی برای جلوگیری از هنگ کردن
      - MM_PLUGINSETTINGS_AUTOMATICPREPACKAGEDPLUGINS=true
      - MM_PLUGINSETTINGS_ENABLEMARKETPLACE=false

volumes:
  postgres_data_avaertebat:
  mattermost_config_avaertebat:
  mattermost_data_avaertebat:
  mattermost_logs_avaertebat:
  mattermost_plugins_avaertebat:
  mattermost_client_plugins_avaertebat:
```
## This Details just for testing in here and you most to chage them
### chat.avacore.ir
### Username ava-ertebat 
### Password:Swordfish641@ and Swordfish64140% 


5.  Click **Deploy the stack**.

---

## Step 2: Nginx Configuration (Reverse Proxy)

Mattermost relies heavily on **WebSockets** for real-time messaging. If you do not configure this in Nginx, the client will constantly disconnect and refresh the page.

Add the following block to your Nginx configuration (inside the `server` block):

```nginx
location / {
    proxy_pass http://<YOUR_SERVER_IP>:8065;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Frame-Options SAMEORIGIN;
    client_max_body_size 50M;
}
```

---

## Troubleshooting & Tips

* **Refresh Loop / Invalid Session:**
    * This usually happens because of old cookies. Open your browser in **Incognito/Private Mode** to test the initial login.
    * Ensure `MM_SERVICESETTINGS_WEBSOCKETURL` is set to `wss://...` in the stack.

* **Permissions Error:**
    * Using the named volumes (as shown in the YAML above) lets Docker handle permissions automatically. Do not change volume paths to local folders (e.g., `./data`) unless you manually set `chown 2000:2000` on the host.

* **Plugins:**
    * Automatic pre-packaged plugin installation is disabled in this stack to ensure a fast boot. You can install plugins manually from the **System Console > Plugin Marketplace** after logging in.
