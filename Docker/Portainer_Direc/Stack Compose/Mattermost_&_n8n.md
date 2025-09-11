# Use This Stack Mattermost_&_n8n on Portainer Stack

```
version: '3.8'

services:
  db_sazehertebat:
    image: postgres:13-alpine
    container_name: db_sazehertebat
    restart: unless-stopped
    volumes:
      - postgres_data_sazehertebat:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=sazehertebat_mm_user
      - POSTGRES_PASSWORD=Swordfish641@
      - POSTGRES_DB=sazehertebat_mattermost_db
    configs:
      - source: init_db_script
        target: /docker-entrypoint-initdb.d/init-db.sql
        mode: 0644

  mattermost_sazehertebat:
    image: mattermost/mattermost-team-edition:latest
    container_name: mattermost_sazehertebat
    restart: unless-stopped
    depends_on:
      - db_sazehertebat
    ports:
      - "4001:8065"
    volumes:
      - mattermost_config_sazehertebat:/mattermost/config
      - mattermost_data_sazehertebat:/mattermost/data
      - mattermost_logs_sazehertebat:/mattermost/logs
      - mattermost_plugins_sazehertebat:/mattermost/plugins
      - mattermost_client_plugins_sazehertebat:/mattermost/client/plugins
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://sazehertebat_mm_user:Swordfish641@@db_sazehertebat:5432/sazehertebat_mattermost_db?sslmode=disable

      - MM_SERVICESETTINGS_SITEURL=https://sazehertebat.avacore.ir

  n8n_sazehertebat:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n_sazehertebat
    restart: unless-stopped
    depends_on:
      - db_sazehertebat
    ports:
      - "5001:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_sazehertebat
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=sazehertebat_n8n_db
      - DB_POSTGRESDB_USER=sazehertebat_mm_user
      - DB_POSTGRESDB_PASSWORD=Swordfish641@
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai-sazehertebat.avacore.ir
      - WEBHOOK_URL=https://ai-sazehertebat.avacore.ir/
    volumes:
      - n8n_data_sazehertebat:/home/node/.n8n

volumes:
  postgres_data_sazehertebat:
  mattermost_config_sazehertebat:
  mattermost_data_sazehertebat:
  mattermost_logs_sazehertebat:
  mattermost_plugins_sazehertebat:
  mattermost_client_plugins_sazehertebat:
  n8n_data_sazehertebat:

configs:
  init_db_script:
    content: |
      CREATE DATABASE sazehertebat_n8n_db;
```
# Edit Config File:
 - Find Container Name With ` sudo docker ps `
 - Copy config File And Edit in ```Host```
 - Move config File on Host To Container
 - Find Container Volume Name With  `sudo docker volume ls`
   
Step 1:
Find Container Name

```
sudo docker ps
```
Step 2:Copy config File And Edit in Host
```
sudo docker cp mattermost-app:/mattermost/config/config.json ./config.json
```
In this Case My Container Name is `mattermost-app`

Step 3:Edit With Editor Like `nano`
```
sudo nano ./config.json
```
Step 4:Move config File on Host To Container
```
sudo docker cp ./config.json mattermost-app:/mattermost/config/config.json
```
Step 5:
Find Container Volume Name
```
sudo docker run --rm -v mattermost_mattermost_config:/mattermost/config alpine sh -c "chown -R 2000:2000 /mattermost/config && chmod -R 700 /mattermost/config && chmod -R 644 /mattermost/config/*"
```
In this Case My Container Volume Name is `mattermost_mattermost_config`
