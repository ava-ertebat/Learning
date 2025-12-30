# All-in-One AI & Automation Stack

This repository contains a Docker Compose stack for deploying a comprehensive suite of AI, Automation, and DevOps tools. It is optimized for environments running behind a corporate Proxy and Nginx Reverse Proxy (Portainer/Docker).

## 🚀 Services Included

* **n8n:** Workflow automation tool (The core of this stack).
* **Flowise:** Drag & drop UI to build your own LLM flows.
* **Ollama:** Local LLM runner.
* **Browserless:** Headless Chrome for scraping and automation.
* **Nexus Repository:** Artifact repository manager.
* **RocketChat:** Open-source team communication.
* **PostgreSQL:** Database backend for n8n.
* **MongoDB:** Database backend for RocketChat and others.

---

## 🛠 Prerequisites

1.  **Docker & Docker Compose** (or Portainer).
2.  **Nginx / Nginx Proxy Manager** handling SSL for domains (e.g., `n8n.avacore.ir`, `chat.avacore.ir`).
3.  **External Volumes:** Since volumes are defined as `external`, you must create them before deploying.

### 1. Create Required Volumes
Run the following commands in your terminal to create the persistent storage volumes:

```bash
docker volume create flowise_data
docker volume create n8n_data
docker volume create nexus-repository_data
docker volume create ollama
docker volume create postgres_data
docker volume create all_need_mongo_data
docker volume create all_need_rocketchat_data
```

---

## 📦 Deployment (Docker Compose)

Copy the configuration below into your `docker-compose.yml` or Portainer Stack Editor.

> **⚠️ Proxy Configuration Note:** This stack includes `HTTP_PROXY` and `HTTPS_PROXY` settings pointing to a specific internal proxy (`172.16.181.168`). **If you do not use this specific proxy, remove those lines from the YAML before deploying.**

```yaml
version: '3.8'

services:
  # --- Browserless ---
  browserless:
    image: docker.arvancloud.ir/browserless/chrome:latest
    container_name: browserless
    ports:
      - "4000:3000"
    volumes:
      - /usr/src/app/downloads:/home/you/downloads
    environment:
      - TOKEN=ava_search
      - HOST=0.0.0.0
      - PORT=3000
      - HTTP_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - HTTPS_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - NO_PROXY=localhost,127.0.0.1,flowise,n8n,ollama,postgresql,mongodb
    restart: always

  # --- Flowise ---
  flowise:
    image: docker.arvancloud.ir/flowiseai/flowise:latest
    container_name: flowise
    ports:
      - "3000:3000"
    volumes:
      - flowise_data:/root/.flowise
    environment:
      - PORT=3000
      - FLOWISE_USERNAME=ava-ertebat
      - FLOWISE_PASSWORD=Swordfish641@
      - HTTP_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - HTTPS_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - NO_PROXY=localhost,127.0.0.1,ollama,n8n,postgresql,mongodb
    restart: always

  # --- n8n ---
  n8n:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    environment:
      # Database Settings
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgresql
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=maindb
      - DB_POSTGRESDB_USER=ava-ertebat
      - DB_POSTGRESDB_PASSWORD=Swordfish641@
      
      # General Settings
      - N8N_PROTOCOL=https
      - N8N_HOST=n8n.avacore.ir
      - WEBHOOK_URL=[https://n8n.avacore.ir/](https://n8n.avacore.ir/)
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - NODE_ENV=production
      
      # Proxy Settings
      - HTTP_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - HTTPS_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      # Important: Exclude local services and internal domains from Proxy
      - NO_PROXY=localhost,127.0.0.1,.ir,postgresql,mongodb,ollama,browserless,flowise,rocketchat,.avacore.ir,chat.avacore.ir
    depends_on:
      - postgresql
    restart: always
    # Map external domains to local Host Gateway (Fixes NAT Loopback)
    extra_hosts:
      - "chat.avacore.ir:host-gateway"
      - "n8n.avacore.ir:host-gateway"

  # --- Nexus ---
  nexus-repository:
    image: registry.docker.ir/sonatype/nexus3:latest
    container_name: nexus-repository
    ports:
      - "82:82"
      - "83:83"
      - "5000:5000"
      - "8081:8081"
    volumes:
      - nexus-repository_data:/nexus-data
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m -Djava.util.prefs.userRoot=/nexus-data/javaprefs
      - JAVA_OPTS=-Xms4g -Xmx4g
    user: "nexus"
    restart: always

  # --- Ollama ---
  ollama:
    image: docker.arvancloud.ir/ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama:/root/.ollama
    environment:
      - OLLAMA_HOST=0.0.0.0:11434
      - HTTP_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - HTTPS_PROXY=[http://swordfish:Swordfish641@172.16.181.168:12195](http://swordfish:Swordfish641@172.16.181.168:12195)
      - NO_PROXY=localhost,127.0.0.1
    restart: always

  # --- PostgreSQL ---
  postgresql:
    image: docker.arvancloud.ir/postgres:17
    container_name: postgresql
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=ava-ertebat
      - POSTGRES_PASSWORD=Swordfish641@
      - POSTGRES_DB=maindb
    restart: always

  # --- MongoDB ---
  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=ava-ertebat
      - MONGO_INITDB_ROOT_PASSWORD=Swordfish641@
    command: mongod --oplogSize 128 --replSet rs0 --bind_ip_all
    restart: always

  # --- RocketChat ---
  rocketchat:
    image: rocket.chat:latest
    container_name: rocketchat
    depends_on:
      - mongodb
    ports:
      - "3001:3000"
    volumes:
      - rocketchat_data:/app/uploads
    environment:
      - MONGO_URL=mongodb://ava-ertebat:Swordfish641%40@mongodb:27017/rocketchat?authSource=admin
      - MONGO_OPLOG_URL=mongodb://ava-ertebat:Swordfish641%40@mongodb:27017/local?authSource=admin
      - ROOT_URL=[https://chat.avacore.ir](https://chat.avacore.ir)
      - PORT=3000
      - OVERWRITE_SETTING_Show_Setup_Wizard=completed
    restart: always

volumes:
  flowise_data:
    external: true
    name: flowise_data
  n8n_data:
    external: true
    name: n8n_data
  nexus-repository_data:
    external: true
    name: nexus-repository_data
  ollama:
    external: true
    name: ollama
  postgres_data:
    external: true
    name: postgres_data
  mongo_data:
    external: true
    name: all_need_mongo_data
  rocketchat_data:
    external: true
    name: all_need_rocketchat_data
```

---

## 🔗 How to Connect n8n to Mattermost (Important)

This stack is designed to handle scenarios where **Mattermost** runs in a separate Docker stack (isolated network) but on the same physical server.

Connecting n8n to Mattermost via the public domain (e.g., `https://chat.avacore.ir`) often fails due to **NAT Loopback** or **Proxy** issues. To solve this, we used a specific configuration in the `n8n` service.

### The Problem
When n8n tries to connect to `chat.avacore.ir`, the request might:
1.  Try to go through the defined `HTTP_PROXY` (and fail).
2.  Try to go out to the internet and loop back to the server, which many firewalls block (Connection Timeout).

### The Solution (Already applied in YAML)

1.  **Bypassing the Proxy:**
    We added the domain to the `NO_PROXY` environment variable:
    ```yaml
    - NO_PROXY=...,.avacore.ir,chat.avacore.ir
    ```

2.  **Using Host Gateway (Direct Connection):**
    We added an `extra_hosts` entry. This maps the domain `chat.avacore.ir` directly to the Docker Host's internal gateway IP (`172.17.0.1`), effectively bypassing the internet entirely while still using the domain name.
    ```yaml
    extra_hosts:
      - "chat.avacore.ir:host-gateway"
    ```

### Configuring Credentials in n8n Panel

When setting up the **Mattermost Credential** inside the n8n UI, use the following settings:

* **Base URL:** `https://chat.avacore.ir`
    * *Do NOT add `/api/v4` at the end.*
    * *Do NOT use the internal container name if they are in different stacks.*
* **Access Token:** Your Mattermost Personal Access Token (Long string).
* **Ignore SSL Issues:** **Toggle ON**.
    * *Since we are routing internally via the host gateway, the container might see self-signed traffic or IP mismatches. Enabling this ensures the connection succeeds.*

By following this setup, n8n will connect to Mattermost instantly without timeouts or proxy errors.
