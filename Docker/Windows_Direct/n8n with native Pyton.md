# 🪟 راهنمای جامع: اجرای n8n با Custom Runner در ویندوز (Docker Desktop)

این راهنما مخصوص کاربرانی است که می‌خواهند n8n را روی سیستم عامل **Windows** با استفاده از **Docker Desktop** اجرا کنند. تمام مسیرها و دستورات متناسب با محیط ویندوز (PowerShell) تنظیم شده‌اند.

---

## 📋 پیش‌نیازها

1. **Docker Desktop** نصب و در حال اجرا باشد (پیشنهاد می‌شود از حالت WSL2 استفاده کنید).
2. یک ویرایشگر متن مثل **VS Code** یا Notepad.
3. دسترسی به اینترنت (و تنظیم پروکسی در داکر دسکتاپ اگر تحریم هستید).

---

## 📂 ۱. ساختار پوشه‌ها

ابتدا باید پوشه‌های پروژه را بسازیم.
**PowerShell** را باز کنید و دستورات زیر را اجرا کنید:

```powershell
# ساخت پوشه اصلی و ورود به آن
mkdir n8n-deployment
cd n8n-deployment

# ساخت پوشه مخصوص رانر
mkdir python-runner
```

ساختار نهایی باید به این شکل باشد:
```text
n8n-deployment/
├── docker-compose.yml
└── python-runner/
    ├── Dockerfile
    └── n8n-task-runners.json
```

---

## ⚙️ ۲. ایجاد فایل کانفیگ (`n8n-task-runners.json`)

1. برنامه **Notepad** یا **VS Code** را باز کنید.
2. کد زیر را داخل آن کپی کنید.
3. فایل را با نام `n8n-task-runners.json` درون پوشه‌ی **`python-runner`** ذخیره کنید.

**محتوای فایل:**

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

## 🐳 ۳. ایجاد فایل `Dockerfile`

1. یک فایل متنی جدید باز کنید.
2. محتوای زیر را در آن کپی کنید.
3. فایل را دقیقاً با نام `Dockerfile` (بدون پسوند .txt) در پوشه‌ی **`python-runner`** ذخیره کنید.

**محتوای فایل:**

```dockerfile
# استفاده از ایمیج پایه رسمی
FROM docker.arvancloud.ir/n8nio/runners:latest

USER root

# نصب کتابخانه‌های پایتون (لیست دلخواه خود را اینجا ویرایش کنید)
# نکته: برای نصب چندین پکیج از فاصله استفاده کنید (مثلا: requests pandas numpy)
RUN cd /opt/runners/task-runner-python && \
    uv pip install requests pandas numpy

# کپی فایل کانفیگ به مسیر صحیح در لینوکس (داخل کانتینر)
COPY n8n-task-runners.json /etc/n8n-task-runners.json

# تنظیم دسترسی‌ها
RUN chmod 644 /etc/n8n-task-runners.json

USER runner
```

---

## 🏗️ ۴. ساخت ایمیج (Build) در پاورشل

حالا باید ایمیج سفارشی را بسازیم.
در همان پنجره **PowerShell** (مطمئن شوید در مسیر `n8n-deployment` هستید)، دستور زیر را اجرا کنید:

```powershell
# بیلد کردن ایمیج از روی پوشه python-runner
docker build --no-cache -t my-custom-runner:latest ./python-runner
```

*منتظر بمانید تا دانلود و نصب پکیج‌ها تمام شود.*

---

## 📦 ۵. ایجاد فایل `docker-compose.yml`

1. یک فایل متنی جدید باز کنید.
2. محتوای زیر را کپی کنید.
3. فایل را با نام `docker-compose.yml` در پوشه‌ی اصلی **`n8n-deployment`** (کنار پوشه python-runner) ذخیره کنید.

**محتوای فایل:**

```yaml
version: '3.8'

services:
  # --- n8n Main Service ---
  n8n:
    image: docker.arvancloud.ir/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    environment:
      - NODE_ENV=production
      - WEBHOOK_URL=http://localhost:5678
      
      # غیرفعال‌سازی تله‌متری برای پایداری بیشتر
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_PERSONALIZATION_ENABLED=false

      # --- تنظیمات اتصال به رانر ---
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_RUNNERS_BROKER_PORT=5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecretPassword123 # رمز عبور باید با رانر یکی باشد

      # تنظیمات دیتابیس
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_db_password

    depends_on:
      - postgres
    restart: always

  # --- Custom Task Runner ---
  python-runner:
    image: my-custom-runner:latest  # استفاده از ایمیجی که در مرحله ۴ ساختیم
    container_name: python-runner
    pull_policy: never # جلوگیری از دانلود، چون ایمیج روی سیستم شماست
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679
      - N8N_RUNNERS_AUTH_TOKEN=MySecretPassword123
      - N8N_RUNNERS_LAUNCHER_LOG_LEVEL=debug
      - N8N_RUNNERS_AUTO_SHUTDOWN_TIMEOUT=0
    depends_on:
      - n8n
    restart: always

  # --- Database ---
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n_db_password
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  n8n_data:
  postgres_data:
```

---

## 🚀 ۶. اجرا

در PowerShell دستور زیر را اجرا کنید:

```powershell
docker compose up -d
```

---

## ✅ ۷. تست نهایی

1. مرورگر را باز کنید و به آدرس `http://localhost:5678` بروید.
2. مراحل ثبت‌نام اولیه n8n را انجام دهید.
3. یک ورک‌فلو جدید بسازید (+).
4. نود **Code** را انتخاب کنید.
5. زبان را روی **Python** قرار دهید.
6. کد زیر را کپی و اجرا کنید:

```python
import requests
import urllib3

# در محیط‌های تست لوکال گاهی SSL ارور می‌دهد، این خط وارنینگ را حذف می‌کند
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# تست پکیج خارجی
response = requests.get("[https://jsonplaceholder.typicode.com/todos/1](https://jsonplaceholder.typicode.com/todos/1)", verify=False)

return [{
    "json": {
        "Message": "Hello from Windows Docker!",
        "Library_Status": "Requests Installed Successfully",
        "API_Data": response.json()
    }
}]
```

اگر خروجی سبز شد، یعنی شما با موفقیت n8n را با رانر اختصاصی روی ویندوز راه‌اندازی کردید! 🎉

---

## ⚠️ نکته مهم در مورد تغییرات

اگر فایل `n8n-task-runners.json` یا `Dockerfile` را تغییر دادید (مثلاً کتابخانه جدیدی اضافه کردید)، باید مراحل زیر را انجام دهید:

1. کانتینرها را متوقف و حذف کنید:
   ```powershell
   docker compose down
   ```
2. ایمیج را دوباره بیلد کنید (خیلی مهم):
   ```powershell
   docker build --no-cache -t my-custom-runner:latest ./python-runner
   ```
3. دوباره اجرا کنید:
   ```powershell
   docker compose up -d
   ```
