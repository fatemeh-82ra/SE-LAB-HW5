
<div dir="rtl">


# گزارش استقرار پروژه Notes با Docker

## مقدمه

این پروژه یک نرم‌افزار ساده برای ثبت یادداشت‌ها است که به کمک **Django** و **PostgreSQL** توسعه یافته است.
هدف این پروژه، آشنایی با **Docker** و **استقرار نرم‌افزار با کانتینرها** است.

-----

## فایل‌های پیکربندی Docker

برای داکرایز کردن این پروژه، از دو فایل اصلی استفاده کردیم: `Dockerfile` برای ساخت ایمیج جنگو و `docker-compose.yml` برای هماهنگی و اجرای سرویس‌های وب و دیتابیس.

### Dockerfile

این فایل شامل دستورالعمل‌هایی برای ساخت ایمیج سرویس وب (جنگو) است.

```dockerfile
# 1. Start an pyhton image
FROM python:3.9-slim-bullseye

# 2. Set environment variables to prevent Python from buffering output
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 3. Set the working directory inside the container
WORKDIR /app

# 4. Copy the requirements file and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy the rest of your project's code into the container
COPY . .

# 6. Expose the port the Django app will run on
EXPOSE 8000

# 7. The default command to run when the container starts.
# This will start the Django development server.
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### docker-compose.yml

این فایل برای تعریف و مدیریت دو سرویس `web` و `db` و نحوه ارتباط آن‌ها با یکدیگر استفاده می‌شود.

```yaml
version: '3.8'

services:
  # سرویس دیتابیس (Postgres)
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=notes_db
      - POSTGRES_USER=notes_user
      - POSTGRES_PASSWORD=notes_password

  # سرویس وب (Django)
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - DB_NAME=notes_db
      - DB_USER=notes_user
      - DB_PASS=notes_password
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db

volumes:
  postgres_data:
```

-----

## راه‌اندازی و تست پروژه

### ۱. بالا آوردن کانتینرها

با این دستور، ایمیج‌ها ساخته شده و کانتینرها در پس‌زمینه اجرا می‌شوند.

```bash
docker-compose up -d --build
```

<img width="980" height="684" alt="image" src="https://github.com/user-attachments/assets/223f8721-06d0-46fd-91b6-cf5742c18919" />

### ۲. بررسی وضعیت

این دستور وضعیت کانتینرهای در حال اجرا را نمایش می‌دهد.

```bash
docker-compose ps
```

<img width="1018" height="94" alt="image" src="https://github.com/user-attachments/assets/9135fdbd-ca9f-4fac-ab9c-a5397babdd73" />

### ۳. مایگریشن دیتابیس

این دستورات جداول مورد نیاز برنامه را در دیتابیس PostgreSQL ایجاد می‌کنند.

```bash
docker-compose exec web python manage.py migrate
```

<img width="828" height="148" alt="image" src="https://github.com/user-attachments/assets/eea0d6ec-8d59-4a15-a613-a7ffbee128ce" />

### ۴. تست عملکرد API

برای تست کامل عملکرد برنامه، یک اسکریپت `bash` نوشتیم که تمام مراحل مورد نیاز (ساخت کاربر، لاگین، ایجاد یادداشت و دریافت یادداشت‌ها) را به صورت خودکار انجام می‌دهد.

**محتوای فایل `test_api.sh`:**

```bash
#!/bin/bash
COOKIE_FILE=cookies.txt

echo "===== Step 1: Create user ====="
curl -X POST http://127.0.0.1:8000/users/create/ -d "username=user1&password=1234"
echo -e "\n"

echo "===== Step 2: Login ====="
curl -c $COOKIE_FILE -X POST http://127.0.0.1:8000/users/login/ -d "username=user1&password=1234"
echo -e "\n"

echo "===== Step 3: Create Note 1 ====="
curl -b $COOKIE_FILE -X POST http://127.0.0.1:8000/notes/create/ -d "title=title1&body=body1"
echo -e "\n"

echo "===== Step 4: Create Note 2 ====="
curl -b $COOKIE_FILE -X POST http://127.0.0.1:8000/notes/create/ -d "title=title2&body=body2"
echo -e "\n"

echo "===== Step 5: Get all notes ====="
curl -b $COOKIE_FILE http://127.0.0.1:8000/notes/
echo -e "\n"
```

### ۵. اجرای اسکریپت تست

```bash
chmod +x test_api.sh
./test_api.sh
```

**خروجی اجرای موفق:**
<img width="1280" height="230" alt="image" src="https://github.com/user-attachments/assets/1c6dd456-eeb5-446c-a91e-fa56c855c3b2" />

-----

## کار با Docker

### لیست کردن ایمیج‌ها و کانتینرها

برای مشاهده ایمیج‌ها و کانتینرهای مربوط به این پروژه، از دستورات زیر استفاده کردیم.

```bash
# دستور مشاهده ایمیج‌ها
docker images

# دستور مشاهده کانتینرهای در حال اجرا
docker ps
```

<img width="1920" height="1028" alt="Screenshot (2618)" src="https://github.com/user-attachments/assets/9b1f6f97-9795-4ed5-8c5d-cfc75c3a6d2a" />
<img width="1920" height="1024" alt="Screenshot (2617)" src="https://github.com/user-attachments/assets/1fe1124f-53c3-42d1-9f02-87d6e535d2bd" />




### اجرای دستور داخل کانتینر

برای اجرای یک دستور دلخواه (مانند `ls -l`) در داخل کانتینر وب، از دستور زیر استفاده کردیم.

```bash
docker-compose exec web ls -l
```



برای اجرای یک دستور دلخواه دیگر در کانتینر در حال اجرای وب سرور، از دستور `docker compose exec` استفاده می‌کنیم. پرچم `-it` به ما یک شل تعاملی (interactive shell) می‌دهد. در دستور زیر، ما یک شل (`sh`) را در سرویس `web` خود باز می‌کنیم:

```bash
docker compose exec -it web sh
```

پس از اجرای این دستور، به محیط داخلی کانتینر دسترسی پیدا می‌کنیم و می‌توانیم دستورات لینوکسی را اجرا کنیم، فایل‌ها را ببینیم یا وضعیت پردازه‌ها را بررسی نماییم.

<img width="1920" height="1025" alt="Screenshot (2620)" src="https://github.com/user-attachments/assets/7691dc1d-1669-4948-83c0-e4b30d242366" />




-----

## پاسخ به سوالات

**۱. نقش‌های Dockerfile، ایمیج و کانتینر را توضیح دهید.**

  * **Dockerfile:** یک فایل متنی حاوی مجموعه‌ای از دستورالعمل‌ها است. این فایل به داکر می‌گوید که یک ایمیج مشخص را چگونه بسازد. در این فایل، سیستم‌عامل پایه، وابستگی‌ها، کدهای برنامه و دستورات لازم برای اجرای نرم‌افزار مشخص می‌شوند.
  * **Image:** نتیجه ساخت یک `Dockerfile` است. ایمیج یک پکیج سبک، مستقل و قابل اجراست که شامل همه چیز برای اجرای یک نرم‌افزار (کد، کتابخانه‌ها، تنظیمات) می‌باشد. ایمیج **قالب** نرم‌افزار است.
  * **Container:** کانتینر یک نمونه زنده و در حال اجرای یک ایمیج است. وقتی شما یک ایمیج را «اجرا» می‌کنید، در واقع یک کانتینر می‌سازید. کانتینر همان فرآیند در حال اجراست که از سیستم‌عامل میزبان و دیگر کانتینرها ایزوله شده است. شما می‌توانید از روی یک ایمیج واحد، کانتینرهای متعددی را ایجاد، اجرا، متوقف و حذف کنید. به عبارتی، کانتینر نسخه «در حال اجرای» یک ایمیج ثابت است.



## کوبرنتیز و ارتباط آن با داکر (Kubernetes & Docker)

### کوبرنتیز برای چه کاری استفاده می‌شود؟

**کوبرنتیز** (Kubernetes که اغلب K8s نامیده می‌شود) یک پلتفرم قدرتمند و متن‌باز برای **ارکستراسیون کانتینرها (container orchestration)** است. در حالی که داکر می‌تواند یک کانتینر را اجرا کند، کوبرنتیز برای مدیریت و خودکارسازی خوشه‌هایی (clusters) از نرم‌افزارهای کانتینری در مقیاس بسیار بزرگ طراحی شده است.

کاربردهای اصلی آن عبارتند از:

* **استقرار و مقیاس‌پذیری خودکار (Automated Deployment and Scaling)**: این ابزار فرآیند استقرار نرم‌افزارها را خودکار کرده و می‌تواند تعداد کانتینرهای در حال اجرا را بر اساس ترافیک یا میزان استفاده از پردازنده (CPU) به صورت خودکار کم یا زیاد کند.
* **خود-ترمیمی (Self-Healing)**: اگر یک کانتینر یا یک سرور از کار بیفتد، کوبرنتیز به طور خودکار آن را مجدداً راه‌اندازی یا جایگزین می‌کند تا از دسترس خارج نشدن نرم‌افزار را تضمین کند.
* **توازن بار و کشف سرویس (Load Balancing and Service Discovery)**: به طور خودکار ترافیک شبکه را بین چندین کانتینر توزیع می‌کند تا عملکرد پایدار بماند و به کانتینرها کمک می‌کند تا یکدیگر را پیدا کرده و با هم ارتباط برقرار کنند.
* **مدیریت پیکربندی (Configuration Management)**: به شما اجازه می‌دهد تا تنظیمات و اطلاعات حساس (مانند رمزهای عبور یا کلیدهای API) را بدون نیاز به ساخت مجدد ایمیج‌هایتان مدیریت کنید.

### ارتباط با داکر

کوبرنتیز و داکر فناوری‌های مکمل یکدیگرند و با هم کار می‌کنند.

* **داکر** ابزاری برای **ساخت و اجرای کانتینرهای مجزا** است. داکر محیط اجرای کانتینر (container runtime) را فراهم می‌کند؛ یعنی همان موتوری که کانتینرها را عملاً اجرا می‌کند.
* **کوبرنتیز** ابزاری برای **مدیریت تعداد زیادی کانتینر** (که اغلب همان کانتینرهای داکر هستند) بر روی چندین ماشین مختلف است.

کوبرنتیز به محیط اجرای کانتینر مانند داکر دستور می‌دهد که چه کاری انجام دهد (مثلاً «سه کانتینر از این ایمیج را اجرا کن») و آن محیط اجرایی دستور را پیاده‌سازی می‌کند.

</div>
