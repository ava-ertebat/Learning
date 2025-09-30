# Production-Ready Deployment Guide: n8n & Mattermost Stack
## with a Custom Python Runner

This document provides a comprehensive, step-by-step guide for deploying a complete, isolated stack for each customer. The stack includes **n8n**, **Mattermost**, and a **PostgreSQL** database. The n8n service is supercharged with a custom "Golden Image" runner, pre-loaded with an extensive list of Python libraries for Data Science, Social Media, and IT Automation.

---

## A. Important Note on Permissions (`sudo`)

Docker commands require root privileges to interact with the Docker daemon. You have two primary methods to handle this:

1.  **Prefix with `sudo` (Default for this guide):** The simplest way is to prepend `sudo` to every `docker` and `docker-compose` command. All commands in this guide will use this format for maximum compatibility.

2.  **Add User to `docker` Group (Recommended):** To avoid typing `sudo` every time, you can add your user to the `docker` group. This is a one-time setup.

    ```bash
    # Add the current user to the docker group
    sudo usermod -aG docker ${USER}

    # You MUST log out and log back in for this change to take effect.
    # You can also run the following command to apply the new group membership immediately:
    newgrp docker
    ```

After doing this, you can run all `docker` commands without `sudo`.

---

## Part 1: Building the Golden Runner Image (One-Time Setup)

This process creates a single, powerful Docker image that all your customer stacks will share, saving significant disk space and centralizing library management.

### Step 1.1: Prepare the Project Directory

On your server, create a directory to hold the image definition files.

```bash
mkdir ~/golden-runner
cd ~/golden-runner
```

### Step 1.2: Create the Definition Files

We will create the `Dockerfile`, `extras.txt`, and `n8n-task-runners.json` files. The following commands will create and populate them instantly.

```bash
# Create the Dockerfile
cat <<EOF > Dockerfile
# Step 1: Use the official and ONLY available runner image (Alpine-based)
FROM n8nio/runners:1.114.0

# Step 2: Switch to root user for ALL installation and setup steps
USER root

# Step 3: Install system packages using apk for Alpine
RUN apk update && apk add --no-cache build-base libffi-dev openssl-dev

# Step 4: Copy the Python requirements file
COPY extras.txt /tmp/extras.txt

# Step 5: Install all Python libraries from the list (as root)
RUN pip install --no-cache-dir -r /tmp/extras.txt

# Step 6: Copy the launcher configuration file
COPY n8n-task-runners.json /etc/n8n-task-runners.json

# Step 7: NOW, at the very end, switch to the non-privileged user for runtime
USER node
EOF

# Create the extras.txt file with the list of Python libraries
cat <<EOF > extras.txt
# --- Data Science & Analysis ---
pandas
numpy
scipy
scikit-learn
matplotlib
seaborn
openpyxl

# --- Web Scraping & API Interaction ---
requests
httpx
beautifulsoup4
lxml
scrapy
python-dotenv

# --- IT Automation & DevOps ---
paramiko
netmiko
psutil
ansible
docker
pyvmomi
boto3
azure-identity
azure-mgmt-compute
google-api-python-client

# --- Databases ---
psycopg2-binary
mysql-connector-python
redis
pymongo
sqlalchemy

# --- Web & API Development ---
Flask
FastAPI
uvicorn
gunicorn

# --- Social Media & Communication ---
tweepy
praw
slack_sdk
python-telegram-bot

# --- Utilities & General Purpose ---
Pillow
jdatetime
arrow
pyjwt
cryptography
pydantic
EOF

# Create the n8n-task-runners.json configuration file
cat <<EOF > n8n-task-runners.json
{
  "task-runners": [
    {
      "runner-type": "javascript",
      "env-overrides": {}
    },
    {
      "runner-type": "python",
      "env-overrides": {
        "PYTHONPATH": "/opt/runners/task-runner-python",
        "N8N_RUNNERS_STDLIB_ALLOW": "json,math,random,datetime,os,sys,subprocess,re",
        "N8N_RUNNERS_EXTERNAL_ALLOW": "pandas,numpy,scipy,scikit-learn,matplotlib,seaborn,openpyxl,requests,httpx,beautifulsoup4,lxml,playwright,scrapy,dotenv,paramiko,netmiko,psutil,ansible,docker,pyvmomi,boto3,azure-identity,azure-mgmt-compute,google-api-python-client,psycopg2,mysql,redis,pymongo,sqlalchemy,flask,fastapi,uvicorn,gunicorn,tweepy,praw,slack_sdk,telegram,Pillow,jdatetime,arrow,jwt,cryptography,pydantic"
      }
    }
  ]
}
EOF
```

### Step 1.3: Build the Docker Image

Run the build command from within the `~/golden-runner` directory. This may take a considerable amount of time.

```bash
# Use a professional naming convention for your image
sudo docker build -t avacore/n8n-python-runner:1.114.0-full .
```

Congratulations! Your powerful, shared runner image is now ready.

---

## Part 2: Deploying the Customer Stack

For each new customer, you will deploy the complete stack using one of the following methods.

### Method A: Deploying via Portainer UI

This method is ideal for quick deployments through a graphical interface.

1.  **Navigate to Stacks:** Log in to Portainer, and go to **Stacks** from the left menu.
2.  **Add a New Stack:** Click **Add stack**.
3.  **Name the Stack:** Give it a unique name, e.g., `stack-akherati`.
4.  **Use Web Editor:** Paste the entire content of the `docker-compose.yml` (from the template below) into the web editor.
5.  **Manually Edit Variables:** **Crucially**, before deploying, find and replace all `YOUR_..._HERE` placeholders in the editor with the correct values for the customer.
6.  **Deploy:** Click **Deploy the stack**.

### Method B: Deploying via Command Line (CLI)

This method is more robust, secure (by separating secrets), and easily automated.

1.  **Create a Directory:** For each customer, create a dedicated directory to hold their configuration.

    ```bash
    # Replace 'akherati' with the actual customer's name
    mkdir -p ~/stacks/akherati
    cd ~/stacks/akherati
    ```

2.  **Create the `docker-compose.yml` file:** Create the file and paste the template content into it. **Note:** This version uses `${VARIABLES}` which will be read from the `.env` file.

    ```bash
    # This command creates the file and opens it for editing
    nano docker-compose.yml
    ```

3.  **Create the `.env` file for Secrets:** This file will store all your secrets and variables, keeping your `docker-compose.yml` clean. **This file should never be committed to a public git repository.**

    ```bash
    # This command creates the .env file and opens it for editing
    nano .env
    ```
    Paste the following content into the `.env` file and fill in the values for your customer:

    ```env
    # --- General Customer Identifier ---
    CUSTOMER_TAG=akherati

    # --- PostgreSQL Credentials ---
    POSTGRES_USER=akherati_mm_user
    POSTGRES_PASSWORD=YOUR_VERY_SECURE_DATABASE_PASSWORD

    # --- Mattermost Configuration ---
    MATTERMOST_SITE_URL=[https://akherati.avacore.ir](https://akherati.avacore.ir)
    MATTERMOST_PORT=4001

    # --- n8n Configuration ---
    N8N_HOST=ai-akherati.avacore.ir
    N8N_PORT=5001

    # --- Shared Secret for n8n Runner ---
    RUNNERS_AUTH_TOKEN=YOUR_SUPER_LONG_AND_SECRET_RUNNER_TOKEN
    ```

4.  **Launch the Stack:** From inside the `~/stacks/akherati` directory, run the following command:

    ```bash
    # This will start all services in the background
    sudo docker-compose up -d
    ```

5.  **Manage the Stack (CLI):**
    * To stop the services: `sudo docker-compose down`
    * To view logs: `sudo docker-compose logs -f`
    * To update images and restart: `sudo docker-compose pull && sudo docker-compose up -d`

---

### Master `docker-compose.yml` Template

Use this content for both **Method A (Portainer)** and **Method B (CLI)**.

```yaml
version: '3.8'

# This template is designed for a single customer and includes all necessary services.
# When using the CLI method, it reads variables from a .env file.
# When using Portainer, you must manually replace the ${VARIABLES} or YOUR_..._HERE placeholders.

services:
  # --- PostgreSQL Database Service ---
  db:
    image: postgres:13-alpine
    container_name: db_${CUSTOMER_TAG:-akherati}
    restart: unless-stopped
    volumes:
      - postgres_data_${CUSTOMER_TAG:-akherati}:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=${POSTGRES_USER:-akherati_mm_user}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-YOUR_SECURE_DB_PASSWORD_HERE}
      - POSTGRES_DB=${CUSTOMER_TAG:-akherati}_mattermost_db # Initial DB for Mattermost
    configs:
      - source: init_db_script_${CUSTOMER_TAG:-akherati}
        target: /docker-entrypoint-initdb.d/init-db.sql
        mode: 0644

  # --- Mattermost Service ---
  mattermost:
    image: mattermost/mattermost-team-edition:latest
    container_name: mattermost_${CUSTOMER_TAG:-akherati}
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "${MATTERMOST_PORT:-4001}:8065"
    volumes:
      - mattermost_config_${CUSTOMER_TAG:-akherati}:/mattermost/config
      - mattermost_data_${CUSTOMER_TAG:-akherati}:/mattermost/data
      - mattermost_logs_${CUSTOMER_TAG:-akherati}:/mattermost/logs
      - mattermost_plugins_${CUSTOMER_TAG:-akherati}:/mattermost/plugins
      - mattermost_client_plugins_${CUSTOMER_TAG:-akherati}:/mattermost/client/plugins
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://${POSTGRES_USER:-akherati_mm_user}:${POSTGRES_PASSWORD:-YOUR_SECURE_DB_PASSWORD_HERE}@db_${CUSTOMER_TAG:-akherati}:5432/${CUSTOMER_TAG:-akherati}_mattermost_db?sslmode=disable
      - MM_SERVICESETTINGS_SITEURL=${MATTERMOST_SITE_URL:-[https://akherati.avacore.ir](https://akherati.avacore.ir)}

  # --- n8n Main Service (Broker) ---
  n8n-main:
    image: n8nio/n8n:1.114.0
    container_name: n8n_main_${CUSTOMER_TAG:-akherati}
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "${N8N_PORT:-5001}:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_${CUSTOMER_TAG:-akherati}
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${CUSTOMER_TAG:-akherati}_n8n_db
      - DB_POSTGRESDB_USER=${POSTGRES_USER:-akherati_mm_user}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD:-YOUR_SECURE_DB_PASSWORD_HERE}
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=${N8N_HOST:-ai-akherati.avacore.ir}
      - WEBHOOK_URL=https://${N8N_HOST:-ai-akherati.avacore.ir}/
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_NATIVE_PYTHON_RUNNER=true
      - N8N_RUNNERS_AUTH_TOKEN=${RUNNERS_AUTH_TOKEN:-YOUR_SUPER_SECRET_TOKEN_HERE}
    volumes:
      - n8n_data_${CUSTOMER_TAG:-akherati}:/home/node/.n8n

  # --- n8n Task Runner Service (Worker) ---
  n8n-runners:
    image: avacore/n8n-python-runner:1.114.0-full
    container_name: n8n_runners_${CUSTOMER_TAG:-akherati}
    restart: unless-stopped
    depends_on:
      - n8n-main
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n-main:5679
      - N8N_RUNNERS_AUTH_TOKEN=${RUNNERS_AUTH_TOKEN:-YOUR_SUPER_SECRET_TOKEN_HERE}
    # Security Warning: To use the 'docker' Python library inside this container,
    # you must mount the Docker socket. This has significant security implications.
    # Uncomment the following lines only if you fully trust the code being executed.
    # volumes:
    #   - /var/run/docker.sock:/var/run/docker.sock

# --- Named Volumes Definition ---
volumes:
  postgres_data_${CUSTOMER_TAG:-akherati}: {}
  mattermost_config_${CUSTOMER_TAG:-akherati}: {}
  mattermost_data_${CUSTOMER_TAG:-akherati}: {}
  mattermost_logs_${CUSTOMER_TAG:-akherati}: {}
  mattermost_plugins_${CUSTOMER_TAG:-akherati}: {}
  mattermost_client_plugins_${CUSTOMER_TAG:-akherati}: {}
  n8n_data_${CUSTOMER_TAG:-akherati}: {}

# --- Configs Definition for DB Initialization ---
configs:
  init_db_script_${CUSTOMER_TAG:-akherati}:
    content: |
      CREATE DATABASE ${CUSTOMER_TAG:-akherati}_n8n_db;
```
