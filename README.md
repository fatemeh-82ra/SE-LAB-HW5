# SE-LAB-HW5
# Notes Project - Docker Deployment

## مقدمه
این پروژه یک نرم‌افزار ساده برای ثبت یادداشت‌ها است که به کمک **Django** و **PostgreSQL** توسعه یافته است.  
هدف این پروژه، آشنایی با **Docker** و **استقرار نرم‌افزار با کانتینرها** است.

---

## راه‌اندازی پروژه

### 1. بالا آوردن کانتینرها
```bash
docker-compose up -d --build
```
<img width="980" height="684" alt="image" src="https://github.com/user-attachments/assets/223f8721-06d0-46fd-91b6-cf5742c18919" />

### 2. بررسی وضعیت
```bash
docker-compose ps
```
<img width="1018" height="94" alt="image" src="https://github.com/user-attachments/assets/9135fdbd-ca9f-4fac-ab9c-a5397babdd73" />

### 3. مایگریشن دیتابیس
```bash
docker-compose exec web bash
python manage.py makemigrations
python manage.py migrate
```
<img width="828" height="148" alt="image" src="https://github.com/user-attachments/assets/eea0d6ec-8d59-4a15-a613-a7ffbee128ce" />

### 4. ساخت فایل bash برای تست API
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
curl -b $COOKIE_FILE -X POST http://127.0.0.1:8000/notes/create/ -d "title=title1&body=body1&user_id=1"
echo -e "\n"

echo "===== Step 4: Create Note 2 ====="
curl -b $COOKIE_FILE -X POST http://127.0.0.1:8000/notes/create/ -d "title=title2&body=body2&user_id=1"
echo -e "\n"

echo "===== Step 5: Get all notes ====="
curl -b $COOKIE_FILE http://127.0.0.1:8000/notes/
echo -e "\n"
```

### 5. اجرای فایل bash
```bash
chmod +x test_api.sh
./test_api.sh
```
<img width="1280" height="230" alt="image" src="https://github.com/user-attachments/assets/1c6dd456-eeb5-446c-a91e-fa56c855c3b2" />

