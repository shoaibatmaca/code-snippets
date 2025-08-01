# 🚀 Django + Django REST Framework Setup Guide

Hey! Hope you're doing great.  
Below is a **complete hands-on guide** on how to install Django and its essential extensions like **Django REST Framework**, along with other commonly used libraries that every Django developer should be aware of.

---

## 📦 Step 1: Create a Virtual Environment

Creating a virtual environment helps isolate dependencies and ensures that the global Python environment on your machine remains unaffected.

### ✅ Option 1 – Custom Environment Name

```bash
python -m venv environment_name

```

### ✅ Option 2 – Common convention:

```bash
python -m venv .venv
```

## 📦 Step 2: Activate the Virtual enviornment

```bash
.venv\scritpts\activate
```

## 📦 Step 3: Install django and the Djangorestframework

```bash
pip install django djangorestframework
```

## 📦 Step 4: install JWT Authentication (SimpleJWT)

```bash
pip install djangorestframework_simplejwt
```

# 📦 Install Commonly Used Libraries

```bash
pip install python-decouple django-cors-headers  requests gunicorn whitenoise redis celery django-celery-beat django-countries channels psycopg2-binary python-dotenv dj-database-url python-decouple Pillow django-storages boto3
```

### These packages help with:

🔑 Env management & secrets (decouple, dotenv)

🌐 API security (cors-headers, simplejwt)

📦 Deployment (gunicorn, whitenoise, dj-database-url)

📷 File/image handling (Pillow, storages, boto3)

🔄 Background tasks (celery, redis, beat)

🌍 Country data, async channels, PostgreSQL, etc.

🧑‍💻 Author
Maintained by Mr Kaglur
