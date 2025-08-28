# 🚀 Django + Django REST Framework — Complete Beginner-to-Advanced Guide

## Table of Contents

1. About
2. Prerequisites
3. Project layout (recommended)
4. Quick setup (commands)
5. Virtual environment & packages
6. Create project + app
7. Settings best-practices
8. Models, Serializers, Views, URLs (example)
9. Authentication (Simple JWT) — full example
10. Pagination, Filtering, Permissions
11. File uploads and static/media
12. Background tasks (Celery + Redis) - quick setup
13. Docker + docker-compose example
14. Tests (unit + API)
15. Deployment tips (Gunicorn, Nginx, WhiteNoise) + env files
16. Useful commands & debugging tips
17. Contributing & Author

---

## 1. About

This guide helps you build a production-ready Django REST API from scratch. It includes code snippets you can copy-paste into your project and a sensible default structure for long-term maintenance.

---

## 2. Prerequisites

- Python 3.10+ (adjust as needed)
- pip
- Git
- (Optional) Docker

---

## 3. Recommended project layout

```
project-root/
├─ .env
├─ .gitignore
├─ docker-compose.yml
├─ Dockerfile
├─ core/                # Django project
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
├─ apps/
│  ├─ users/
│  │  ├─ models.py
│  │  ├─ serializers.py
│  │  ├─ views.py
│  │  └─ urls.py
│  └─ posts/
├─ requirements.txt
└─ manage.py
```

Keep apps inside an `apps/` folder for clarity.

---

## 4. Quick setup (commands)

```bash
# create project dir
mkdir myproject && cd myproject
python -m venv .venv
source .venv/bin/activate   # or .venv\Scripts\activate on Windows
pip install --upgrade pip
pip install django djangorestframework djangorestframework-simplejwt python-decouple django-cors-headers psycopg2-binary pillow
django-admin startproject core .
python manage.py startapp users
```

Add installed apps in `core/settings.py` (see section 7).

---

## 5. Virtual environment & packages

Recommended `requirements.txt` (generate with `pip freeze > requirements.txt`):

```
Django==5.2.x
djangorestframework==3.XX
djangorestframework-simplejwt==4.X
python-decouple
django-cors-headers
psycopg2-binary
Pillow
```

Use `python-decouple` or `python-dotenv` to keep secrets out of source control.

---

## 6. Create project + app (example files)

### `core/settings.py` (important parts)

```python
from pathlib import Path
from datetime import timedelta
from decouple import config

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default='False') == 'True'
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # third party
    'rest_framework',
    'corsheaders',

    # local apps
    'apps.users',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

# Static & media
STATIC_URL = '/static/'
MEDIA_URL = '/media/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
MEDIA_ROOT = BASE_DIR / 'media'

# REST framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}

# Simple JWT
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'AUTH_HEADER_TYPES': ('Bearer',),
}

# CORS
CORS_ALLOWED_ORIGINS = config('CORS_ALLOWED_ORIGINS', default='').split(',')
```

---

## 7. Models, Serializers, Views, URLs — Minimal example

### `apps/users/models.py`

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    bio = models.TextField(blank=True)
```

### `apps/users/serializers.py`

```python
from rest_framework import serializers
from .models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('id', 'username', 'email', 'first_name', 'last_name', 'bio')
```

### `apps/users/views.py`

```python
from rest_framework import viewsets, permissions
from .models import User
from .serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
```

### `apps/users/urls.py`

```python
from rest_framework.routers import DefaultRouter
from .views import UserViewSet

router = DefaultRouter()
router.register('users', UserViewSet)

urlpatterns = router.urls
```

And include it in `core/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('apps.users.urls')),
]
```

Run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 8. Authentication — Simple JWT (endpoints)

Install: `pip install djangorestframework-simplejwt`

Add URLs to `core/urls.py`:

```python
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns += [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

Usage:

- `POST /api/token/` with `{"username": "...", "password": "..."}` returns access + refresh.
- `POST /api/token/refresh/` with `{"refresh": "..."}` returns new access token.

Protect endpoints with `IsAuthenticated` or custom permissions.

---

## 9. Pagination, Filtering, and Permissions

- Pagination: default page number pagination already set in settings.
- Filtering: use `django-filter` (`pip install django-filter`) and add `rest_framework.filters.DjangoFilterBackend` to `DEFAULT_FILTER_BACKENDS`.
- Permissions: built-in (`IsAuthenticated`, `IsAdminUser`, `IsAuthenticatedOrReadOnly`) or custom classes.

Example of adding django-filter:

```python
INSTALLED_APPS += ['django_filters']
REST_FRAMEWORK['DEFAULT_FILTER_BACKENDS'] = ['django_filters.rest_framework.DjangoFilterBackend']
```

---

## 10. File uploads & static/media

- Use `MEDIA_ROOT`/`MEDIA_URL` in settings.
- During development, serve media with `urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)`.
- For production, configure object storage (S3 / Cloudflare R2) via `django-storages`.

---

## 11. Background tasks (Celery + Redis) — quick setup

Install: `pip install celery redis`

`core/celery.py`

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'core.settings')
app = Celery('core')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

`core/__init__.py`

```python
from .celery import app as celery_app

__all__ = ('celery_app',)
```

Settings example:

```python
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
```

Create tasks in an app `tasks.py` with `@shared_task`.

---

## 12. Docker + docker-compose (simple example)

**Dockerfile** (Python + Gunicorn)

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "core.wsgi:application", "--bind", "0.0.0.0:8000"]
```

**docker-compose.yml** (web + db + redis)

```yaml
version: "3.8"
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
  redis:
    image: redis:7
  web:
    build: .
    command: gunicorn core.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
volumes:
  postgres_data:
```

---

## 13. Tests (unit + API)

Install: `pip install pytest pytest-django djangorestframework` (or use Django TestCase)

Simple API test example with `APITestCase`:

```python
from rest_framework.test import APITestCase
from django.urls import reverse

class ProfileAPITest(APITestCase):
    def test_list_profiles(self):
        url = reverse('user-list')  # depends on your router name
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
```

Run `python manage.py test` or `pytest`.

---

## 14. Deployment tips

- Use `gunicorn` as WSGI server.
- Use `nginx` as reverse proxy and to serve static files.
- Use `WhiteNoise` to serve static files from Gunicorn in simpler setups.
- Store secrets in env vars and never commit `.env`.
- Use managed DB (RDS, Cloud SQL) and object storage for media.
- Configure HTTPS and HSTS on production.

Example `Procfile` for Heroku-style deployment:

```
web: gunicorn core.wsgi --log-file -
```

---

## 15. Useful commands & debugging tips

- `python manage.py shell` — open Django shell
- `python manage.py showmigrations`
- `python manage.py makemigrations && python manage.py migrate`
- `python manage.py runserver --settings=core.settings.local` — if you split settings
- `python manage.py test`
- Use `DEBUG = True` locally only; log errors in production

---

## 16. Contributing

If you want to contribute:

- Fork the repo
- Create a feature branch
- Open a PR with clear description and tests

---

## 17. Author

Maintained by **Muhammad Shoaib**. If you want a tailored version (e.g., with user registration, email verification, RSA JWT, or a complete Docker + CI pipeline), tell me what extra features you want and I will expand the guide.

---

_End of guide — copy this `README.md` to your project root and edit the env and project-specific values._
