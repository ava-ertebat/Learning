# 🚀 Ultimate Guide: Deploying n8n with Custom External Task Runners

این مستندات نحوه راه‌اندازی **n8n Self-Hosted** را در حالت **External Runner** توضیح می‌دهد. این معماری برای محیط‌های پروداکشن توصیه می‌شود تا پردازش کدهای سنگین (Python/JS) از هسته اصلی n8n جدا شود.

همچنین نحوه نصب کتابخانه‌های جانبی (مثل `requests`) و رفع محدودیت‌های امنیتی در این راهنما پوشش داده شده است.

---

## 📂 1. ساختار فایل‌ها (File Structure)

ابتدا دایرکتوری‌های زیر را در سرور خود ایجاد کنید:

```text
n8n-deployment/
├── docker-compose.yml
└── python-runner/
    ├── Dockerfile
    └── n8n-task-runners.json
```

---

## ⚙️ 2. تنظیم فایل کانفیگ رانر (`n8n-task-runners.json`)

این مهم‌ترین فایل است. این فایل به `Launcher` می‌گوید که چگونه پایتون و نود را اجرا کند.

**نکات حیاتی:**
1. کلید مسیر دایرکتوری حتماً باید **`workdir`** باشد (نه `dir` یا `cwd`).
2. برای اجازه دسترسی به کتابخانه‌های خارجی، حتماً باید متغیرها در بخش **`env-overrides`** تعریف شوند.

محتوای فایل `python-runner/n8n-task-runners.json`:

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

---

## 🐳 3. ساخت Dockerfile سفارشی

برای نصب کتابخانه‌های اضافی پایتون (مثل `requests`, `pandas`, `numpy`) باید ایمیج رسمی را اکستند کنیم.

محتوای فایل `python-runner/Dockerfile`:

```dockerfile
# استفاده از آخرین نسخه پایدار رانر
FROM docker.arvancloud.ir/n8nio/runners:latest

USER root

# 1. نصب پکیج‌های مورد نیاز پایتون
# از uv برای سرعت بالاتر استفاده می‌شود
RUN cd /opt/runners/task-runner-python && uv pip install requests

# 2. کپی کردن فایل کانفیگ صحیح به مسیر پیش‌فرض لانچر
# مسیر پیش‌فرض در ایمیج اصلی /etc/n8n-task-runners.json است
COPY n8n-task-runners.json /etc/n8n-task-runners.json

# 3. تنظیم دسترسی فایل (جهت اطمینان)
RUN chmod 644 /etc/n8n-task-runners.json

USER runner
```

---

## 🛠️ 4. ساخت ایمیج رانر (Build)

قبل از اجرای سرویس‌ها، باید ایمیج رانر خود را بسازید. به دایرکتوری `python-runner` بروید یا مسیر را مشخص کنید.

```bash
# بیلد کردن ایمیج با نام مشخص (بدون استفاده از کش برای اطمینان از تغییرات)
sudo docker build --no-cache -t my-custom-runner:latest ./python-runner
```

---

## 📦 5. فایل Docker Compose

در این فایل، ارتباط بین `n8n` و `python-runner` برقرار می‌شود.

محتوای فایل `docker-compose.yml`:

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
      # تنظیمات عمومی
      - NODE_ENV=production
      - WEBHOOK_URL=[https://n8n.example.com](https://n8n.example.com)
      
      # غیرفعال کردن تله‌متری (برای تمیز ماندن لاگ‌ها و جلوگیری از خطای SSL)
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_PERSONALIZATION_ENABLED=false

      # --- تنظیمات اتصال به رانر (Task Runner) ---
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_RUNNERS_BROKER_PORT=5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecureSecretKey123 # باید با رانر یکی باشد

      # تنظیمات دیتابیس (Postgres)
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_password

    depends_on:
      - postgres
    restart: always

  # --- Custom Python/JS Runner ---
  python-runner:
    image: my-custom-runner:latest  # نام ایمیجی که در مرحله 4 ساختیم
    container_name: python-runner
    pull_policy: never # چون ایمیج لوکال است
    environment:
      # آدرس کانتینر n8n و پورت بروکر
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecureSecretKey123 # دقیقا مشابه n8n
      - N8N_RUNNERS_LAUNCHER_LOG_LEVEL=debug
      
      # پروکسی (در صورت نیاز)
      # - HTTP_PROXY=http://proxy:port
      # - HTTPS_PROXY=http://proxy:port
      # - NO_PROXY=n8n,localhost,127.0.0.1
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

---

## 🚀 6. راه‌اندازی نهایی

```bash
# اجرای سرویس‌ها در حالت Detached
sudo docker compose up -d
```

### ✅ تست صحت عملکرد
برای اطمینان از اینکه همه چیز درست کار می‌کند، یک نود `Code` در n8n ایجاد کنید، زبان را روی `Python` بگذارید و کد زیر را اجرا کنید:

```python
import requests
import urllib3

# غیرفعال کردن وارنینگ‌های SSL (اختیاری)
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# تست اتصال اینترنت و نصب بودن کتابخانه
url = "[https://jsonplaceholder.typicode.com/todos/1](https://jsonplaceholder.typicode.com/todos/1)"
response = requests.get(url, verify=False)

return [{
    "json": {
        "status": "Success",
        "message": "Requests library loaded successfully!",
        "api_response": response.json()
    }
}]
```

---

## ⚠️ عیب‌یابی (Troubleshooting)

### 🔴 خطا: `failed to chdir into configured dir (): no such file`
* **علت:** کلید تعریف دایرکتوری در فایل JSON اشتباه است.
* **راه‌حل:** مطمئن شوید در فایل `n8n-task-runners.json` از کلید **`workdir`** استفاده کرده‌اید (نه `dir` یا `directory`).

### 🔴 خطا: `Security violations detected ... 'requests' is disallowed`
* **علت:** لانچر n8n متغیرهای محیطی را به پروسه پایتون پاس نداده است.
* **راه‌حل:** در فایل JSON، در بخش `env-overrides` حتماً خط زیر را اضافه کنید:
    `"N8N_RUNNERS_EXTERNAL_ALLOW": "*"`

### 🔴 خطا: `Task request timed out after 60 seconds`
* **علت:** رانر به n8n وصل نشده یا کرش کرده است.
* **راه‌حل:** لاگ رانر را چک کنید (`docker logs python-runner`). اگر در حال ریستارت شدن است، فایل کانفیگ مشکل دارد. اگر ثابت است اما وصل نمی‌شود، پورت `5679` یا `AUTH_TOKEN` را در هر دو سرویس چک کنید.
