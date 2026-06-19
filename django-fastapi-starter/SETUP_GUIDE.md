# Backend Development Setup Guide

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- PostgreSQL 12+
- Git
- Virtual environment tool (venv or virtualenv)

---

## Step 1: Environment Setup

### Create Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Configure Environment Variables
```bash
cp .env.example .env
# Edit .env with your configuration
```

---

## Step 2: PostgreSQL Setup

### Create Database
```sql
CREATE DATABASE backend_db;
CREATE USER backend_user WITH PASSWORD 'secure_password';
ALTER ROLE backend_user SET client_encoding TO 'utf8';
ALTER ROLE backend_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE backend_user SET default_transaction_deferrable TO on;
ALTER ROLE backend_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE backend_db TO backend_user;
```

### Alternative: Using pgAdmin
1. Open pgAdmin
2. Create new database "backend_db"
3. Create new role "backend_user"
4. Grant privileges

---

## Step 3: Django Setup

### Create Django Project
```bash
django-admin startproject config .
django-admin startapp api
```

### Directory Structure
```
django_project/
├── config/
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── api/
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py
│   └── tests.py
├── manage.py
└── requirements.txt
```

### Update settings.py
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'rest_framework',
    'corsheaders',
    'api',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': os.getenv('DB_PORT'),
    }
}

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ... other middleware
]

CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://localhost:8000",
]
```

### Run Migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

### Create Superuser
```bash
python manage.py createsuperuser
```

### Run Django Server
```bash
python manage.py runserver
# Visit http://localhost:8000
```

---

## Step 4: FastAPI Setup

### Create FastAPI Project Structure
```
fastapi_project/
├── main.py
├── models.py
├── schemas.py
├── database.py
├── api/
│   ├── users.py
│   ├── products.py
│   └── orders.py
├── tests/
│   └── test_api.py
└── requirements.txt
```

### database.py
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base
import os

DATABASE_URL = f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('DB_NAME')}"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Run FastAPI Server
```bash
uvicorn main:app --reload
# Visit http://localhost:8000/docs for interactive documentation
```

---

## Step 5: Testing

### Run Django Tests
```bash
python manage.py test api
# or with coverage
coverage run --source='.' manage.py test api
coverage report
```

### Run FastAPI Tests
```bash
pytest tests/
# or with coverage
pytest --cov=. tests/
```

### Example Test
```python
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_create_product():
    response = client.post(
        "/products",
        json={"title": "Test", "price": 10.0},
        headers={"Authorization": "Bearer token"}
    )
    assert response.status_code == 201
    assert response.json()["title"] == "Test"
```

---

## Step 6: Database Queries

### Django ORM
```python
from api.models import Product, Order

# Create
Product.objects.create(title="Laptop", price=999.99)

# Read
products = Product.objects.all()
product = Product.objects.get(id=1)

# Update
product.stock = 50
product.save()

# Delete
product.delete()

# Filter
active_products = Product.objects.filter(is_active=True)

# Aggregate
from django.db.models import Sum, Count
total = Product.objects.aggregate(Sum('price'))
```

### SQLAlchemy (FastAPI)
```python
from database import SessionLocal
from models import Product

db = SessionLocal()

# Create
new_product = Product(title="Laptop", price=999.99)
db.add(new_product)
db.commit()

# Read
products = db.query(Product).all()

# Update
db.query(Product).filter(Product.id == 1).update({"stock": 50})
db.commit()

# Delete
db.query(Product).filter(Product.id == 1).delete()
db.commit()
```

---

## Step 7: API Authentication

### JWT with FastAPI
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthCredentials
import jwt

security = HTTPBearer()

async def verify_token(credentials: HTTPAuthCredentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.get("/protected")
async def protected_route(payload = Depends(verify_token)):
    return {"message": f"Hello {payload.get('sub')}"}
```

---

## Common Commands

```bash
# Django
python manage.py runserver
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py test

# FastAPI
uvicorn main:app --reload
pytest tests/

# PostgreSQL
psql -U postgres
\c backend_db
\dt  # List tables
\d table_name  # Show table structure

# General
pip freeze > requirements.txt
python -m black .  # Format code
flake8 .  # Lint code
```

---

## Deployment Checklist

- [ ] Set DEBUG=False in production
- [ ] Use environment-specific settings
- [ ] Set up logging
- [ ] Configure database backups
- [ ] Use SSL certificates
- [ ] Set up monitoring
- [ ] Configure CORS properly
- [ ] Use strong SECRET_KEY
- [ ] Test all endpoints
- [ ] Document API

---

## Useful Resources

- [Django Official Docs](https://docs.djangoproject.com)
- [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial)
- [PostgreSQL Docs](https://www.postgresql.org/docs)
- [SQLAlchemy ORM](https://docs.sqlalchemy.org)
- [REST API Best Practices](https://restfulapi.net)

---

**Happy coding! 🚀**
