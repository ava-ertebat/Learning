# Use This Stack Mattermost_&_n8n on Portainer Stack
-------------------------------
```
version: '3.8'

services:
  db_avaertebat:
    image: postgres:13-alpine
    container_name: db_avaertebat
    restart: unless-stopped
    volumes:
      - postgres_data_avaertebat:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=avaertebat_mm_user
      - POSTGRES_PASSWORD=Swordfish641@
      - POSTGRES_DB=avaertebat_mattermost_db
    configs:
      - source: init_db_script
        target: /docker-entrypoint-initdb.d/init-db.sql
        mode: 0644

  mattermost_avaertebat:
    image: mattermost/mattermost-team-edition:latest
    container_name: mattermost_avaertebat
    restart: unless-stopped
    depends_on:
      - db_avaertebat
    ports:
      - "4001:8065"
    volumes:
      - mattermost_config_avaertebat:/mattermost/config
      - mattermost_data_avaertebat:/mattermost/data
      - mattermost_logs_avaertebat:/mattermost/logs
      - mattermost_plugins_avaertebat:/mattermost/plugins
      - mattermost_client_plugins_avaertebat:/mattermost/client/plugins
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://avaertebat_mm_user:Swordfish641@@db_avaertebat:5432/avaertebat_mattermost_db?sslmode=disable

      - MM_SERVICESETTINGS_SITEURL=https://avaertebat.avacore.ir

  n8n_avaertebat:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n_avaertebat
    restart: unless-stopped
    depends_on:
      - db_avaertebat
    ports:
      - "5001:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_avaertebat
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=avaertebat_n8n_db
      - DB_POSTGRESDB_USER=avaertebat_mm_user
      - DB_POSTGRESDB_PASSWORD=Swordfish641@
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai-avaertebat.avacore.ir
      - WEBHOOK_URL=https://ai-avaertebat.avacore.ir/
    volumes:
      - n8n_data_avaertebat:/home/node/.n8n

volumes:
  postgres_data_avaertebat:
  mattermost_config_avaertebat:
  mattermost_data_avaertebat:
  mattermost_logs_avaertebat:
  mattermost_plugins_avaertebat:
  mattermost_client_plugins_avaertebat:
  n8n_data_avaertebat:

configs:
  init_db_script:
    content: |
      CREATE DATABASE avaertebat_n8n_db;
```
-------------------------------
In This Case `avertebat` Is Tested Name

For YourSelf Replase `avaertebat` With name You need
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
sudo docker cp mattermost_avaertebat:/mattermost/config/config.json ./config.json
```
In this Case My Container Name is `mattermost-app`

Step 3: Edit Config File With Editor Like `nano`
```
sudo nano ./config.json
```
Step 4:Move config File on Host To Container
```
sudo docker cp ./config.json mattermost_avaertebat:/mattermost/config/config.json
```
Step 5:
Find Container Volume Name
```
sudo docker run --rm -v mattermost_config_avaertebat:/mattermost/config alpine sh -c "chown -R 2000:2000 /mattermost/config && chmod -R 700 /mattermost/config && chmod -R 644 /mattermost/config/*"
```
In this Case My Container Volume Name is `mattermost_mattermost_config`
