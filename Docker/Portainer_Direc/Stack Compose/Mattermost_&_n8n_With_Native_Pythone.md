# راهنمای کامل راه‌اندازی n8n با قابلیت Native Python

این مستند به صورت گام به گام، فرآیند راه‌اندازی یک نمونه (Instance) از n8n به همراه قابلیت اجرای کدهای پایتون به صورت Native را شرح می‌دهد. این راهنما برای یک ساختار چندمشتری (Multi-tenant) طراحی شده که در آن هر مشتری ساب‌دامین مخصوص به خود را دارد (مثلاً `client-name.avacore.ir`).

**پیش‌نیازها:**
* دسترسی به یک سرور با Docker و Docker Compose نصب شده.
* نصب و راه‌اندازی Portainer برای مدیریت آسان‌تر کانتینرها.
* دسترسی به ترمینال (CLI) سرور برای ساخت ایمیج سفارشی.

---

## بخش اول: ساخت ایمیج سفارشی Runner (از طریق CLI)

چرا این بخش ضروری است؟ ایمیج پیش‌فرض n8n runner تنها شامل کتابخانه‌های استاندارد پایتون است. برای استفاده از کتابخانه‌های قدرتمندی مانند `pandas`, `requests`, `numpy` و غیره، باید یک ایمیج Docker سفارشی بسازیم که این کتابخانه‌ها را در خود داشته باشد. این کار فقط یک بار (یا هر زمان که نیاز به افزودن کتابخانه جدیدی داشتید) انجام می‌شود.

### گام ۱: ایجاد پوشه پروژه

در سرور خود، یک پوشه برای نگهداری فایل‌های ساخت ایمیج ایجاد کنید.

```bash
mkdir custom-n8n-runner
cd custom-n8n-runner
```

### گام ۲: ایجاد فایل `Dockerfile`

با یک ویرایشگر متن (مانند `nano`)، فایلی به نام `Dockerfile` بسازید و محتوای زیر را در آن قرار دهید.

```bash
nano Dockerfile
```

**محتوای `Dockerfile`:**
```dockerfile
# از آخرین نسخه پایدار ایمیج رسمی runners به عنوان پایه استفاده می‌کنیم.
# همیشه نسخه آن را با نسخه n8n که در docker-compose استفاده می‌کنید، یکسان نگه دارید.
FROM n8nio/runners:1.114.0

# فایل نیازمندی‌های پایتون را به داخل ایمیج کپی می‌کنیم.
COPY extras.txt /tmp/extras.txt

# کتابخانه‌های مشخص شده در فایل extras.txt را با pip نصب می‌کنیم.
RUN pip install --no-cache-dir -r /tmp/extras.txt

# فایل تنظیمات launcher را برای فعال‌سازی کتابخانه‌های نصب‌شده، کپی می‌کنیم.
# این فایل به n8n اجازه می‌دهد تا از این کتابخانه‌ها در نود Code استفاده کند.
COPY n8n-task-runners.json /etc/n8n-task-runners.json
```

### گام ۳: ایجاد فایل `extras.txt` (لیست کتابخانه‌های پایتون)

این فایل، لیست تمام کتابخانه‌های پایتون است که می‌خواهید نصب شوند.

```bash
nano extras.txt
```

**محتوای نمونه `extras.txt`:**
```txt
# هر کتابخانه را در یک خط بنویسید و برای پایداری، حتما نسخه آن را مشخص کنید.
pandas==2.2.2
requests==2.32.3
numpy==1.26.4
# jdatetime==4.1.0  # برای مثال، یک کتابخانه کار با تاریخ شمسی
```

### گام ۴: ایجاد فایل `n8n-task-runners.json` (فایل مجوزها)

این فایل به n8n اعلام می‌کند که استفاده از کدام کتابخانه‌ها مجاز است. این یک اقدام امنیتی مهم است.

```bash
nano n8n-task-runners.json
```

**محتوای `n8n-task-runners.json`:**
```json
{
  "task-runners": [
    {
      "runner-type": "javascript",
      "env-overrides": {
        "NODE_FUNCTION_ALLOW_BUILTIN": "",
        "NODE_FUNCTION_ALLOW_EXTERNAL": ""
      }
    },
    {
      "runner-type": "python",
      "env-overrides": {
        "PYTHONPATH": "/opt/runners/task-runner-python",
        "N8N_RUNNERS_STDLIB_ALLOW": "json,math,random,datetime",
        "N8N_RUNNERS_EXTERNAL_ALLOW": "pandas,requests,numpy,jdatetime"
      }
    }
  ]
}
```
**نکته:** هر کتابخانه‌ای که در `extras.txt` اضافه می‌کنید، باید نام آن را در بخش `N8N_RUNNERS_EXTERNAL_ALLOW` نیز وارد کنید.

### گام ۵: ساخت ایمیج Docker

حالا که تمام فایل‌ها آماده هستند، دستور زیر را در ترمینال (داخل پوشه `custom-n8n-runner`) اجرا کنید تا ایمیج شما ساخته شود.

```bash
# شما می‌توانید نام ایمیج (my-custom-runners:1.46.0) را به دلخواه تغییر دهید.
# فقط به یاد داشته باشید که از همین نام در فایل docker-compose استفاده کنید.
docker build -t my-custom-runners:1.46.0 .
```
تبریک! ایمیج سفارشی شما با موفقیت ساخته شد و آماده استفاده است.

---

## بخش دوم: راه‌اندازی استک در Portainer

حالا که ایمیج Runner آماده است، می‌توانیم کل استک را از طریق رابط کاربری Portainer راه‌اندازی کنیم.

### گام ۱: ورود به Portainer و بخش Stacks

وارد حساب کاربری Portainer خود شوید و از منوی سمت چپ، به بخش **Stacks** بروید.

### گام ۲: افزودن استک جدید

روی دکمه **Add stack** کلیک کنید.

### گام ۳: تنظیمات استک

1.  **Name:** یک نام برای استک خود انتخاب کنید. بهتر است نام مشتری را در آن لحاظ کنید، مثلا: `n8n-akherati`.
2.  **Build method:** گزینه **Web editor** را انتخاب کنید.
3.  **Web editor:** کل محتوای فایل `docker-compose.yml` زیر را کپی کرده و در این بخش جای‌گذاری کنید.

### فایل نهایی `docker-compose.yml`

```yaml
version: '3.8'

# ----------------------------------------------------------------------------------
# این فایل یک قالب آماده برای راه‌اندازی n8n برای یک مشتری خاص است.
# قبل از اجرا، مقادیر مشخص‌شده با YOUR_..._HERE را با اطلاعات صحیح جایگزین کنید.
# ----------------------------------------------------------------------------------

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
      - POSTGRES_PASSWORD=YOUR_SECURE_DB_PASSWORD_HERE # ⚠️ رمز عبور دیتابیس را تغییر دهید
      - POSTGRES_DB=akherati_mattermost_db # دیتابیس اولیه برای mattermost
    configs:
      - source: init_db_script_akherati
        target: /docker-entrypoint-initdb.d/init-db.sql
        mode: 0644

  # ------ سرویس Mattermost (اختیاری - بر اساس استک اصلی شما) ------
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
      - MM_SERVICESETTINGS_SITEURL=[https://akherati.avacore.ir](https://akherati.avacore.ir) # ⚠️ آدرس Mattermost

  # ------ سرویس اصلی n8n (کارفرما یا Broker) ------
  n8n_main_akherati:
    # ❗️نسخه این ایمیج باید با نسخه پایه ایمیج Runner شما یکی باشد
    image: n8nio/n8n:1.114.0
    container_name: n8n_main_akherati
    restart: unless-stopped
    depends_on:
      - db_akherati
    ports:
      - "5001:5678"
    environment:
      # --- تنظیمات دیتابیس ---
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=db_akherati
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=akherati_n8n_db # دیتابیس n8n
      - DB_POSTGRESDB_USER=akherati_mm_user
      - DB_POSTGRESDB_PASSWORD=YOUR_SECURE_DB_PASSWORD_HERE # ⚠️ همان رمز عبور دیتابیس
      # --- تنظیمات دامنه و پروتکل n8n ---
      - NODE_ENV=production
      - N8N_PROTOCOL=https
      - N8N_HOST=ai-akherati.avacore.ir # ⚠️ ساب‌دامین مخصوص n8n این مشتری
      - WEBHOOK_URL=[https://ai-akherati.avacore.ir/](https://ai-akherati.avacore.ir/)
      
      # ------ تنظیمات حیاتی برای فعال‌سازی Runner ------
      - N8N_RUNNERS_ENABLED=true
      - N8N_RUNNERS_MODE=external
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
      - N8N_NATIVE_PYTHON_RUNNER=true
      - N8N_RUNNERS_AUTH_TOKEN=YOUR_SUPER_SECRET_TOKEN_HERE # ⚠️ یک توکن امن و طولانی ایجاد کنید

    volumes:
      - n8n_data_akherati:/home/node/.n8n

  # ------ سرویس Task Runner (کارگر) ------
  n8n_task_runners_akherati:
    # ❗️از ایمیج سفارشی که در بخش اول ساختید استفاده کنید
    image: my-custom-runners:1.114.0
    container_name: n8n_task_runners_akherati
    restart: unless-stopped
    depends_on:
      - n8n_main_akherati
    environment:
      # آدرس کانتینر اصلی n8n برای اتصال
      - N8N_RUNNERS_TASK_BROKER_URI=http://n8n_main_akherati:5679
      # توکن امنیتی مشترک (باید با مقدار بالا یکسان باشد)
      - N8N_RUNNERS_AUTH_TOKEN=YOUR_SUPER_SECRET_TOKEN_HERE # ⚠️ همان توکن بالا را وارد کنید

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

### گام ۴: ویرایش مقادیر ضروری

**قبل از کلیک روی دکمه Deploy**، مقادیر زیر را مستقیماً در ویرایشگر Portainer ویرایش کنید:
* `YOUR_SECURE_DB_PASSWORD_HERE`: رمز عبور دیتابیس را با یک رمز امن جایگزین کنید (در ۳ مکان تکرار شده).
* `N8N_HOST`: ساب‌دامین مخصوص n8n این مشتری را وارد کنید (مثلاً `ai-akherati.avacore.ir`).
* `WEBHOOK_URL`: آدرس کامل وب‌هوک را وارد کنید (معمولاً همان آدرس `N8N_HOST` است).
* `YOUR_SUPER_SECRET_TOKEN_HERE`: یک رشته امن و طولانی به عنوان توکن ارتباطی ایجاد کنید و در ۲ مکان جایگزین نمایید.

### گام ۵: استقرار استک

پس از اطمینان از صحت مقادیر، روی دکمه **Deploy the stack** کلیک کنید. Portainer تمام سرویس‌ها را بر اساس تنظیمات شما ایجاد و اجرا خواهد کرد.

---

## نگهداری و بروزرسانی

### افزودن کتابخانه پایتون جدید
1.  فایل `extras.txt` را ویرایش کرده و کتابخانه جدید را اضافه کنید.
2.  فایل `n8n-task-runners.json` را ویرایش کرده و نام کتابخانه جدید را به لیست مجازها اضافه کنید.
3.  با دستور `docker build` (گام ۵ بخش اول) ایمیج را با یک تگ نسخه جدید (مثلاً `my-custom-runners:1.46.1`) دوباره بسازید.
4.  در Portainer، استک را ویرایش کرده و تگ ایمیج `n8n_task_runners_akherati` را به نسخه جدید تغییر دهید و استک را مجدداً Deploy کنید.

### راه‌اندازی برای یک مشتری جدید (مثلاً `clientB`)
1.  **بخش اول (ساخت ایمیج)** نیازی به تکرار ندارد، مگر اینکه مشتری جدید کتابخانه‌های متفاوتی بخواهد.
2.  در **بخش دوم**، یک استک جدید در Portainer بسازید.
3.  کل فایل `docker-compose.yml` را کپی کرده و تمام پسوندهای `_akherati` را به `_clientB` تغییر دهید (برای `container_name`, `volumes`, `configs`, و نام سرویس‌ها).
4.  مقادیر مربوط به دامنه (`N8N_HOST`) و سایر متغیرهای محیطی را برای مشتری جدید تنظیم کنید.
5.  استک جدید را Deploy کنید.
