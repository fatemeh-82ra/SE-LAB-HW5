
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

*(در اینجا خروجی دستورات بالا را قرار دهید)*

### اجرای دستور داخل کانتینر

برای اجرای یک دستور دلخواه (مانند `ls -l`) در داخل کانتینر وب، از دستور زیر استفاده کردیم.

```bash
docker-compose exec web ls -l
```

*(در اینجا خروجی دستور بالا را قرار دهید)*

-----

## پاسخ به سوالات

**۱. نقش‌های Dockerfile، ایمیج و کانتینر را توضیح دهید.**

  * **Dockerfile:** یک فایل متنی است که شامل دستورالعمل‌هایی برای ساخت یک ایمیج داکر می‌باشد. این فایل مانند یک **دستور پخت** عمل می‌کند.
  * **Image:** نتیجه ساخت یک `Dockerfile` است. ایمیج یک پکیج سبک، مستقل و قابل اجراست که شامل همه چیز برای اجرای یک نرم‌افزار (کد، کتابخانه‌ها، تنظیمات) می‌باشد. ایمیج **قالب** نرم‌افزار است.
  * **Container:** یک نمونه در حال اجرای یک ایمیج است. می‌توان چندین کانتینر از یک ایمیج اجرا کرد که هر کدام از دیگری ایزوله هستند. کانتینر **نمونه زنده** نرم‌افزار است.

**۲. Kubernetes برای چه کاری می‌تواند استفاده شود؟ چه ارتباطی با Docker دارد؟**

  * **Kubernetes** یک پلتفرم ارکستراسیون کانتینر است که برای مدیریت، خودکارسازی، مقیاس‌دهی و استقرار تعداد زیادی کانتینر در چندین ماشین استفاده می‌شود. در حالی که Docker Compose برای مدیریت چند کانتینر روی یک ماشین مناسب است، کوبرنتیز برای محیط‌های بزرگ و پروداکشن طراحی شده است.
  * **ارتباط آن با Docker** این است که کوبرنتیز کانتینرها را مدیریت می‌کند و داکر محبوب‌ترین ابزار برای ساخت آن کانتینرها است. کوبرنتیز می‌تواند کانتینرهای داکر را بر اساس نیازهای برنامه، اجرا، متوقف و مدیریت کند.

</div>
