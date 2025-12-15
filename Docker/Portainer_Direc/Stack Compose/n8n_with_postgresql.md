# Use This Stack n8n And postgresql on Portainer Stack
-------------------------------
```
version: '3.8'

services:
  # --- دیتابیس (امن و ایزوله) ---
  db_avacore:
    image: postgres:13-alpine
    container_name: db_avacore
    restart: unless-stopped
    volumes:
      - postgres_data_avacore:/var/lib/postgresql/data
    ports:
      - "5002:5432"
    environment:
      - POSTGRES_USER=avacore_mm_user
      - POSTGRES_PASSWORD=Swordfish641@
      - POSTGRES_DB=avacore_n8n_db # نام دیتابیس اصلاح شد
    # نکته امنیتی: پورت‌ها را حذف کردیم تا از اینترنت قابل دسترسی نباشد
    networks:
      - Local_Lan

  # --- سرویس n8n ---
  n8n_avacore:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n_avacore
    restart: unless-stopped
    depends_on:
      - db_avacore
    ports:
      - "5001:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_avacore
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=avacore_n8n_db
      - DB_POSTGRESDB_USER=avacore_mm_user
      - DB_POSTGRESDB_PASSWORD=Swordfish641@
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai.avacore.ir
      - WEBHOOK_URL=https://ai.avacore.ir/
    volumes:
      - n8n_data_avacore:/home/node/.n8n
    networks:
      - Local_Lan

volumes:
  postgres_data_avacore:
  n8n_data_avacore:

networks:
  Local_Lan:
    external: true


```
-------------------------------
