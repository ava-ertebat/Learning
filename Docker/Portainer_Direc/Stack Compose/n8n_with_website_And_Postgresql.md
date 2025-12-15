# 🌐 استک یکپارچه: وب‌سایت، دیتابیس و اتوماسیون (n8n)

این پروژه یک محیط کامل شامل **وب‌سایت شخصی (Nginx)**، **پایگاه داده (PostgreSQL)** و **ابزار اتوماسیون (n8n)** را با استفاده از داکر کامپوز راه‌اندازی می‌کند. تمام سرویس‌ها از طریق یک شبکه داخلی امن به هم متصل هستند.

---

## 📋 راهنمای شخصی‌سازی (قبل از شروع)
در فایل زیر مقادیر پیش‌فرضی وجود دارد که باید آن‌ها را بر اساس نیاز خود تغییر دهید. لطفاً جدول زیر را بررسی کنید:

| مقدار در فایل (پیش‌فرض) | توضیحات | مقدار پیشنهادی شما |
| :--- | :--- | :--- |
| `avacore` | نام پروژه یا نام کاربری شما در اسم کانتینرها | مثال: `myproject`, `company_name` |
| `Swordfish641@` | رمز عبور دیتابیس (بسیار مهم) | **یک رمز عبور قوی و پیچیده** |
| `ai.avacore.ir` | آدرس دامنه برای دسترسی به پنل n8n | دامنه یا ساب‌دامین اختصاصی شما |
| `/root/website` | مسیر فایل‌های HTML وب‌سایت روی سرور | مسیری که فایل‌های سایت شما آنجاست |

---

## 🛠 پیش‌نیازها

### ۱. ایجاد شبکه (Network)
این استک از یک شبکه خارجی به نام `Local_Lan` استفاده می‌کند تا سرویس‌ها بتوانند همدیگر را ببینند و مدیریت شوند. قبل از اجرای فایل، دستور زیر را در ترمینال بزنید:

```bash
docker network create Local_Lan
```

### ۲. آماده‌سازی فایل‌های وب‌سایت
مطمئن شوید که در مسیر `/root/website` (یا مسیری که در فایل تعیین می‌کنید)، فایل‌های وب‌سایت شما (مانند `index.html`) وجود داشته باشند.

---

## 🚀 فایل Docker Compose

کد زیر را در فایلی با نام `docker-compose.yml` ذخیره کنید.

> **⚠️ توجه امنیتی:** حتماً `POSTGRES_PASSWORD` را در هر دو بخش دیتابیس و n8n تغییر دهید.

```yaml
version: '3.8'

services:
  # ----------------------------------------
  # 1. وب‌سایت (Landing Page)
  # ----------------------------------------
  website_avacore: # [تغییر نام: به جای avacore اسم خود را بگذارید]
    image: nginx:alpine
    container_name: website_avacore
    restart: unless-stopped
    volumes:
      # نکته: مسیر سمت چپ (هاست) باید جایی باشد که فایل‌های سایت شما قرار دارد
      - /root/website:/usr/share/nginx/html
    networks:
      - Local_Lan

  # ----------------------------------------
  # 2. دیتابیس (PostgreSQL)
  # ----------------------------------------
  db_avacore: # [تغییر نام دلخواه]
    image: postgres:13-alpine
    container_name: db_avacore
    restart: unless-stopped
    volumes:
      - postgres_data_avacore:/var/lib/postgresql/data
    ports:
      - "5002:5432" # پورت بیرونی 5002 است
    environment:
      - POSTGRES_USER=avacore_mm_user       # [تغییر نام کاربری دلخواه]
      - POSTGRES_PASSWORD=Swordfish641@     # <--- [حتما رمز عبور را تغییر دهید]
      - POSTGRES_DB=avacore_n8n_db          # [تغییر نام دیتابیس دلخواه]
    networks:
      - Local_Lan

  # ----------------------------------------
  # 3. سرویس اتوماسیون (n8n)
  # ----------------------------------------
  n8n_avacore: # [تغییر نام دلخواه]
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n_avacore
    restart: unless-stopped
    depends_on:
      - db_avacore
    ports:
      - "5001:5678" # دسترسی به پنل n8n روی پورت 5001
    environment:
      # تنظیمات اتصال به دیتابیس
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_avacore       # باید دقیقا با نام کانتینر دیتابیس یکی باشد
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=avacore_n8n_db
      - DB_POSTGRESDB_USER=avacore_mm_user
      - DB_POSTGRESDB_PASSWORD=Swordfish641@ # <--- [باید با رمز دیتابیس بالا یکی باشد]
      
      # تنظیمات عمومی n8n
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai.avacore.ir              # <--- [دامنه خود را وارد کنید]
      - WEBHOOK_URL=https://ai.avacore.ir/  # <--- [دامنه خود را وارد کنید]
    volumes:
      - n8n_data_avacore:/home/node/.n8n
    networks:
      - Local_Lan

# ----------------------------------------
# Volumes & Networks
# ----------------------------------------
volumes:
  postgres_data_avacore:
  n8n_data_avacore:

networks:
  Local_Lan:
    external: true
```

## 💡 نکات تکمیلی
* **دسترسی به n8n:** پس از اجرا، پنل مدیریت در `https://YOUR-DOMAIN:5001` یا `http://IP:5001` در دسترس خواهد بود.
* **مسیر وب‌سایت:** اگر وب‌سایت لود نشد، پرمیشن‌های پوشه `/root/website` را بررسی کنید تا Nginx دسترسی خواندن داشته باشد.
