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
  postgres:
    # Mattermost v11.x requires PostgreSQL 16+
    image: postgres:16-alpine
    container_name: mattermost-db
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=mmuser
      - POSTGRES_PASSWORD=Secure_DB_Password
      - POSTGRES_DB=mattermost
    # Healthcheck ensures DB is ready before Mattermost starts
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U mmuser -d mattermost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - mattermost-net

  mattermost:
    image: mattermost/mattermost-team-edition:release-11.3
    container_name: mattermost-app
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    ports:
      - "8065:8065"
    volumes:
      # Named volumes allow Portainer/Docker to manage permissions automatically
      - mattermost_config:/mattermost/config:rw
      - mattermost_data:/mattermost/data:rw
      - mattermost_logs:/mattermost/logs:rw
      - mattermost_plugins:/mattermost/plugins:rw
      - mattermost_client_plugins:/mattermost/client/plugins:rw
    environment:
      # --- Database Connection ---
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      # Use %40 instead of @ if your password contains an @ symbol
      - MM_SQLSETTINGS_DATASOURCE=postgres://mmuser:Secure_DB_Password@postgres:5432/mattermost?sslmode=disable&connect_timeout=10

      # --- Site URL Configuration ---
      - MM_SERVICESETTINGS_SITEURL=[https://chat.example.com](https://chat.example.com)
      
      # --- CRITICAL FIX: WebSocket URL ---
      # Explicitly setting WSS prevents "Refresh Loops" behind Nginx
      - MM_SERVICESETTINGS_WEBSOCKETURL=wss://chat.example.com
      
      # --- Proxy & Security Settings ---
      - MM_SERVICESETTINGS_USELETSENCRYPT=false
      - MM_SERVICESETTINGS_FORWARD80TO443=false
      # Trust headers from the Nginx container
      - MM_SERVICESETTINGS_TRUSTEDPROXYIP=0.0.0.0/0
      - MM_SERVICESETTINGS_ALLOWCORSFROM=[https://chat.example.com](https://chat.example.com)
      # Force Secure Cookies (Required for modern browsers on HTTPS)
      - MM_SERVICESETTINGS_SESSIONCOOKIESECURE=true
      
      # --- Initial Admin Account (Optional) ---
      # If the setup wizard fails, this creates the admin user automatically
      - MM_ADMIN_USERNAME=admin-user
      - MM_ADMIN_PASSWORD=Secure_Admin_Password
      
      # --- Optimization ---
      # Prevents startup crashes by disabling automatic pre-packaged plugin loading
      - MM_PLUGINSETTINGS_AUTOMATICPREPACKAGEDPLUGINS=false
      - MM_PLUGINSETTINGS_ENABLE=true
      
    networks:
      - mattermost-net

networks:
  mattermost-net:
    driver: bridge

volumes:
  postgres_data:
  mattermost_config:
  mattermost_data:
  mattermost_logs:
  mattermost_plugins:
  mattermost_client_plugins:
```

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
