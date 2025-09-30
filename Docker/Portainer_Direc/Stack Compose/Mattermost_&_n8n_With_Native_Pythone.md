# راهنمای کامل راه‌اندازی پکیج مشتری (n8n + Mattermost) با Runner قدرتمند پایتون

این مستند به صورت گام به گام، فرآیند راه‌اندازی یک پکیج کامل و اختصاصی برای هر مشتری را شرح می‌دهد. این پکیج شامل سرویس‌های **n8n**، **Mattermost** و **PostgreSQL** است و n8n در آن به یک Runner سفارشی با قابلیت‌های گسترده پایتون مجهز شده است.

**استراتژی:**
1.  **ساخت ایمیج یک‌باره:** ابتدا یک ایمیج Runner قدرتمند (Golden Image) با تمام کتابخانه‌های لازم می‌سازیم.
2.  **استقرار استک کامل:** برای هر مشتری، یک استک کامل شامل هر سه سرویس را با استفاده از این ایمیج Runner راه‌اندازی می‌کنیم.

---

## بخش اول: ساخت ایمیج طلایی Runner (یک بار برای همیشه)

این بخش تنها یک بار (یا هر زمان که نیاز به آپدیت اساسی کتابخانه‌ها داشتید) انجام می‌شود.

### گام ۱: آماده‌سازی فایل‌ها

در سرور خود، پوشه‌ای برای ساخت ایمیج ایجاد کنید (`mkdir golden-runner && cd golden-runner`). سپس سه فایل زیر را در آن بسازید.

#### فایل ۱: `Dockerfile` (دستور ساخت پیشرفته)

```dockerfile
# همیشه نسخه آن را با نسخه n8n که در docker-compose استفاده می‌کنید، یکسان نگه دارید.
FROM n8nio/runners:1.114.0

# به کاربر root سوییچ می‌کنیم تا بتوانیم پکیج‌های سیستمی را نصب کنیم
USER root

# نصب وابستگی‌های سیستمی مورد نیاز برای کتابخانه‌های پایتون
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libffi-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# به کاربر پیش‌فرض n8n برمی‌گردیم
USER node

# نصب مرورگرهای مورد نیاز برای Playwright
RUN python -m playwright install --with-deps

# کپی و نصب کتابخانه‌های پایتون
COPY extras.txt /tmp/extras.txt
RUN pip install --no-cache-dir -r /tmp/extras.txt

# کپی فایل تنظیمات launcher
COPY n8n-task-runners.json /etc/n8n-task-runners.json
```

#### فایل ۲: `extras.txt` (لیست جامع کتابخانه‌ها)

```txt
# === Data Science & Analysis ===
pandas
numpy
scipy
scikit-learn
matplotlib
seaborn
openpyxl

# === Web Scraping & API Interaction ===
requests
httpx
beautifulsoup4
lxml
playwright
scrapy
python-dotenv

# === IT Automation & DevOps ===
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

# === Databases ===
psycopg2-binary
mysql-connector-python
redis
pymongo
sqlalchemy

# === Web & API Development ===
Flask
FastAPI
uvicorn
gunicorn

# === Social Media & Communication ===
tweepy
praw
slack_sdk
python-telegram-bot

# === Utilities & General Purpose ===
Pillow
jdatetime
arrow
pyjwt
cryptography
pydantic
```

#### فایل ۳: `n8n-task-runners.json` (فایل جامع مجوزها)

```json
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
```

### گام ۲: ساخت ایمیج Docker

حالا ایمیج جامع خود را بسازید.

```bash
# از یک نام و تگ معنادار استفاده کنید
docker build -t avacore/n8n-python-runner:1.114.0-full .
```

---

## بخش دوم: راه‌اندازی استک کامل مشتری در Portainer

برای هر مشتری جدید، از این بخش برای راه‌اندازی پکیج کامل و اختصاصی او استفاده کنید.

### گام ۱: ورود به Portainer و افزودن استک جدید

وارد حساب کاربری Portainer خود شوید، به بخش **Stacks** بروید و روی **Add stack** کلیک کنید.

### گام ۲: تنظیمات استک

1.  **Name:** یک نام برای استک مشتری انتخاب کنید، مثلا: `stack-akherati`.
2.  **Build method:** گزینه **Web editor** را انتخاب کنید.
3.  **Web editor:** کل محتوای فایل `docker-compose.yml` زیر را کپی کرده و در این بخش جای‌گذاری کنید.

### فایل `docker-compose.yml` (قالب کامل برای هر مشتری)

```yaml
version: '3.8'

services:
  # ------ سرویس دیتابیس (مشترک برای سرویس‌های این مشتری) ------
  db_akherati:
    image: postgres:13-alpine
    container_name: db_akherati
    restart: unless-stopped
    volumes:
      - postgres_data_akherati:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=akherati_mm_user
      - POSTGRES_PASSWORD=YOUR_SECURE_DB_PASSWORD_HERE
      - POSTGRES_DB=akherati_mattermost_db
    configs:
      - source: init_db_script_akherati
        target: /docker-entrypoint-initdb.d/init-db.sql
        mode: 0644

  # ------ سرویس Mattermost ------
  mattermost_akherati:
    image: mattermost/mattermost-team-edition:latest
    container_name: mattermost_akherati
    restart: unless-stopped
    depends_on:
      - db_akherati
    ports:
      - "4001:8065"
    volumes:
      - mattermost_config_akherati:/mattermost/config
      - mattermost_data_akherati:/mattermost/data
      - mattermost_logs_akherati:/mattermost/logs
      - mattermost_plugins_akherati:/mattermost/plugins
      - mattermost_client_plugins_akherati:/mattermost/client/plugins
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://akherati_mm_user:YOUR_SECURE_DB_PASSWORD_HERE@db_akherati:5432/akherati_mattermost_db?sslmode=disable
      - MM_SERVICESETTINGS_SITEURL=[https://akherati.avacore.ir](https://akherati.avacore.ir)

  # ------ سرویس اصلی n8n (کارفرما یا Broker) ------
  n8n_main_akherati:
    image: n8nio/n8n:1.114.0
    container_name: n8n_main_akherati
    restart: unless-stopped
    depends_on:
      - db_akherati
    ports:
      - "5001:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_akherati
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=akherati_n8n_db
      - DB_POSTGRESDB_USER=akherati_mm_user
      - DB_POSTGRESDB_PASSWORD=YOUR_SECURE_DB_PASSWORD_HERE
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai-akherati.avacore.ir
      - WEBHOOK_URL=[https://ai-akherati.avacore.ir/](https://ai-akherati.avacore.ir/)
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_NATIVE_PYTHON_RUNNER=true
      - N8N_RUNNERS_AUTH_TOKEN=YOUR_SUPER_SECRET_TOKEN_HERE
    volumes:
      - n8n_data_akherati:/home/node/.n8n

  # ------ سرویس Task Runner (کارگر) ------
  n8n_task_runners_akherati:
    image: avacore/n8n-python-runner:1.114.0-full
    container_name: n8n_task_runners_akherati
    restart: unless-stopped
    depends_on:
      - n8n_main_akherati
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n_main_akherati:5679
      - N8N_RUNNERS_AUTH_TOKEN=YOUR_SUPER_SECRET_TOKEN_HERE
    # --- هشدار امنیتی: برای استفاده از کتابخانه docker ---
    # volumes:
    #   - /var/run/docker.sock:/var/run/docker.sock

# تعریف Volume‌ها برای نگهداری دائمی داده‌ها
volumes:
  postgres_data_akherati:
  mattermost_config_akherati:
  mattermost_data_akherati:
  mattermost_logs_akherati:
  mattermost_plugins_akherati:
  mattermost_client_plugins_akherati:
  n8n_data_akherati:

# تعریف Config برای ساخت دیتابیس n8n در اولین اجرا
configs:
  init_db_script_akherati:
    content: |
      CREATE DATABASE akherati_n8n_db;
```

### گام ۳: ویرایش مقادیر ضروری و استقرار

**قبل از کلیک روی دکمه Deploy**، مقادیر زیر را مستقیماً در ویرایشگر Portainer ویرایش کنید:
* `YOUR_SECURE_DB_PASSWORD_HERE`: رمز عبور دیتابیس را با یک رمز امن جایگزین کنید (در ۳ مکان تکرار شده).
* `MM_SERVICESETTINGS_SITEURL`: آدرس کامل Mattermost مشتری را وارد کنید.
* `N8N_HOST`: ساب‌دامین مخصوص n8n این مشتری را وارد کنید.
* `WEBHOOK_URL`: آدرس کامل وب‌هوک n8n را وارد کنید.
* `YOUR_SUPER_SECRET_TOKEN_HERE`: یک رشته امن و طولانی به عنوان توکن ارتباطی ایجاد کنید و در ۲ مکان جایگزین نمایید.

پس از اطمینان از صحت مقادیر، روی دکمه **Deploy the stack** کلیک کنید.

---

## نگهداری و بروزرسانی

### افزودن کتابخانه پایتون جدید به همه مشتریان
1.  فایل‌های `extras.txt` و `n8n-task-runners.json` را در پوشه `golden-runner` ویرایش کنید.
2.  با دستور `docker build` ایمیج را با یک تگ نسخه جدید (مثلاً `avacore/n8n-python-runner:1.114.0-full-v2`) دوباره بسازید.
3.  در Portainer، استک هر مشتری را ویرایش کرده، تگ ایمیج `n8n_task_runners_...` را به نسخه جدید تغییر دهید و استک را مجدداً Deploy کنید.

### راه‌اندازی برای یک مشتری جدید (مثلاً `clientB`)
1.  در **بخش دوم**، یک استک جدید در Portainer بسازید.
2.  کل فایل `docker-compose.yml` را کپی کرده و تمام پسوندهای `_akherati` را به `_clientB` تغییر دهید.
3.  مقادیر مربوط به دامنه‌ها، پورت‌ها و سایر متغیرهای محیطی را برای مشتری جدید تنظیم کنید.
4.  استک جدید را Deploy کنید.
