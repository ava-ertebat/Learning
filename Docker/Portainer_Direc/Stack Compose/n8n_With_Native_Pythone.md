# 🐳 راهنمای استقرار n8n با Custom Task Runner در Portainer

این راهنما مخصوص کسانی است که از **Portainer** برای مدیریت داکر استفاده می‌کنند. از آنجایی که بخش Stacks در Portainer به صورت پیش‌فرض از بیلد کردن فایل‌های لوکال پشتیبانی نمی‌کند، ما از روش **"Build Locally, Deploy Globally"** استفاده می‌کنیم.

---

## 📋 پیش‌نیازها

1. دسترسی SSH به سرور (برای ساخت ایمیج).
2. دسترسی ادمین به پنل Portainer.
3. نام شبکه (Network) در نظر گرفته شده (در این مثال شبکه به صورت خودکار توسط استک ساخته می‌شود).

---

## 🛠️ مرحله ۱: آماده‌سازی فایل‌ها در سرور (از طریق SSH)

قبل از رفتن به محیط گرافیکی پورتینر، باید ایمیج سفارشی رانر را در سرور بسازیم.

### ۱. ساخت پوشه و فایل‌ها
به سرور SSH بزنید و دستورات زیر را اجرا کنید:

```bash
# ساخت پوشه پروژه
mkdir -p /opt/n8n-custom-runner
cd /opt/n8n-custom-runner

# ایجاد فایل کانفیگ رانر
sudo nano n8n-task-runners.json
```

**محتوای فایل `n8n-task-runners.json` (حیاتی):**
*دقت کنید که از کلید `workdir` استفاده شده باشد.*

```json
{
  "task-runners": [
    {
      "runner-type": "python",
      "workdir": "/opt/runners/task-runner-python",
      "command": "/opt/runners/task-runner-python/.venv/bin/python",
      "args": [
        "-m", "src.main"
      ],
      "health-check-server-port": "5682",
      "env-overrides": {
        "PYTHONPATH": "/opt/runners/task-runner-python",
        "N8N_RUNNERS_EXTERNAL_ALLOW": "*",
        "N8N_RUNNERS_STDLIB_ALLOW": "*"
      }
    },
    {
      "runner-type": "javascript",
      "workdir": "/opt/runners/task-runner-javascript",
      "command": "/usr/local/bin/node",
      "args": ["/opt/runners/task-runner-javascript/dist/start.js"],
      "health-check-server-port": "5681",
      "env-overrides": {
        "NODE_FUNCTION_ALLOW_EXTERNAL": "*"
      }
    }
  ]
}
```
*(ذخیره با CTRL+X سپس Y)*

### ۲. ایجاد Dockerfile
```bash
sudo nano Dockerfile
```

**محتوای فایل `Dockerfile`:**

```dockerfile
FROM docker.arvancloud.ir/n8nio/runners:latest

USER root

# نصب کتابخانه‌های پایتون (لیست خود را اینجا ویرایش کنید)
# نصب کتابخانه‌های کاربردی پایتون (بدون هوش مصنوعی سنگین)
# لیست:
# requests, httpx -> ارتباطات شبکه
# pandas, openpyxl -> مدیریت داده و اکسل
# beautifulsoup4, lxml -> پردازش HTML و XML
# mysql-connector-python, sqlalchemy -> دیتابیس
# paramiko -> مدیریت سرور (SSH/SFTP)
# pillow -> پردازش تصویر
# pyjwt, cryptography -> امنیت
# python-dateutil, pytz -> مدیریت زمان و تاریخ (برای اسکجول‌ها)

RUN cd /opt/runners/task-runner-python && \
    uv pip install requests pandas numpy

# کپی کانفیگ صحیح به مسیر استاندارد
COPY n8n-task-runners.json /etc/n8n-task-runners.json
RUN chmod 644 /etc/n8n-task-runners.json

USER runner
```

---

## 🏗️ مرحله ۲: ساخت ایمیج (Build)

حالا باید ایمیج را بسازیم تا در لیست ایمیج‌های لوکال سرور (و پورتینر) قرار بگیرد.

```bash
# بیلد کردن ایمیج با تگ مشخص
sudo docker build --no-cache -t my-custom-runner:latest .
```

> **نکته مهم:** هر بار که بخواهید کتابخانه جدیدی به پایتون اضافه کنید یا فایل کانفیگ `json` را تغییر دهید، باید این دستور بیلد را مجدداً اجرا کنید و سپس در پورتینر استک را آپدیت کنید.

---

## 🌐 مرحله ۳: راه‌اندازی در پنل Portainer

حالا که ایمیج آماده است، مرورگر را باز کنید و وارد Portainer شوید.

1. وارد محیط **Local** شوید.
2. از منوی سمت چپ گزینه **Stacks** را انتخاب کنید.
3. دکمه **+ Add stack** را بزنید.
4. **Name:** یک نام انتخاب کنید (مثلاً `n8n-stack`).
5. **Build method:** گزینه `Web editor` را انتخاب کنید.

### محتوای Web Editor (Docker Compose)

کد زیر را در ادیتور پیست کنید.
*توجه:* بخش `build: .` حذف شده و به جای آن از ایمیج ساخته شده استفاده می‌کنیم.

```yaml
version: '3.8'

services:
  # --- n8n Main Instance ---
  n8n:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    environment:
      - NODE_ENV=production
      - WEBHOOK_URL=[https://n8n.example.com](https://n8n.example.com)
      
      # غیرفعال کردن تله‌متری (رفع خطاهای SSL و کاهش مصرف منابع)
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_PERSONALIZATION_ENABLED=false

      # --- تنظیمات اتصال به رانر ---
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_RUNNERS_BROKER_PORT=5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecureSecretKey123 

      # تنظیمات دیتابیس
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_password

    depends_on:
      - postgres
    restart: always

  # --- Custom Task Runner ---
  python-runner:
    # نام ایمیجی که در مرحله ۲ دستی ساختیم
    image: my-custom-runner:latest  
    container_name: python-runner
    # بسیار مهم: هرگز سعی نکن این ایمیج را از اینترنت دانلود کن (چون لوکال است)
    pull_policy: never 
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecureSecretKey123
      - N8N_RUNNERS_LAUNCHER_LOG_LEVEL=debug
      - N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT=0
      
      # تنظیمات پروکسی (در صورت نیاز آنکامنت کنید)
      # - HTTP_PROXY=http://ip:port
      # - HTTPS_PROXY=http://ip:port
      # - NO_PROXY=n8n,postgres,localhost
    depends_on:
      - n8n
    restart: always

  # --- Database ---
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n_password
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  n8n_data:
  postgres_data:
```

6. دکمه **Deploy the stack** را در پایین صفحه بزنید.

---

## ✅ مرحله ۴: تست نهایی

بعد از اینکه وضعیت کانتینرها در Portainer به **Healthy** یا **Running** تغییر کرد:

1. پنل n8n را باز کنید (`https://your-domain:5678`).
2. یک ورک‌فلو جدید بسازید.
3. نود **Code** را اضافه کرده و زبان را روی **Python** بگذارید.
4. کد زیر را برای تست اجرا کنید:

```python
import requests
import urllib3

# غیرفعال کردن خطاهای SSL
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# تست پکیج خارجی
response = requests.get("[https://jsonplaceholder.typicode.com/todos/1](https://jsonplaceholder.typicode.com/todos/1)", verify=False)

return [{
    "json": {
        "status": "Success",
        "library_installed": True,
        "data": response.json()
    }
}]
```

اگر خروجی سبز رنگ دریافت کردید، عملیات با موفقیت انجام شده است! 🚀

---

## 🔄 نحوه آپدیت کردن رانر (اضافه کردن پکیج جدید)

اگر خواستید مثلاً کتابخانه `scikit-learn` را اضافه کنید:

1. در سرور (SSH) فایل `Dockerfile` را ویرایش کنید و نام پکیج را اضافه کنید.
2. دستور بیلد را دوباره اجرا کنید:
   `sudo docker build --no-cache -t my-custom-runner:latest .`
3. در **Portainer**:
   * به بخش **Stacks** بروید.
   * روی استک n8n کلیک کنید.
   * دکمه **Editor** را بزنید.
   * بدون تغییر هیچ کدی، دکمه **Update the stack** را بزنید.
   * (مهم) تیک گزینه **Re-pull image and redeploy** را **بردارید** (چون ایمیج لوکال است و پول نمی‌شود). فقط آپدیت کنید تا کانتینر با ایمیج جدیدی که بیلد کردید جایگزین شود.
