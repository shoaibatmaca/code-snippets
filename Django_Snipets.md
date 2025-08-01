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

---

Now let Create our first project after installing the django and djangorestframework.Below is a full guide

## Step 1: Create Django Project

```bash
django-admin startproject core .
```

Here is core is a project name and the . dot used to dont create the inner folder

## Step 2: Create your app

```bash
python manage.py startapp app_name
```

## ⚙️Step 3: Update settings.py

    INSTALLED_APPS = [
    'rest_framework',
    'corsheaders',
    'django_celery_beat',
    'user',
    'app_name'

]

---

## ⚙️Step 4: JWT REST Framework Settings:

REST_FRAMEWORK = {
'DEFAULT_AUTHENTICATION_CLASSES': (
'rest_framework_simplejwt.authentication.JWTAuthentication',
),
}

## ⚙️Step 5: SimpleJWT Settings:

from datetime import timedelta

SIMPLE_JWT = {
'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
'AUTH_HEADER_TYPES': ('Bearer',),
}

---

🧑‍💻 Author
Maintained by Mr Kaglur
