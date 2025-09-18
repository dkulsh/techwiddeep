---
title: "Inventory Management System"
meta_title: "Inventory Management System - Real-time Stock Tracking and Analytics"
description: "Built a comprehensive inventory management system with real-time stock tracking, automated reorder points, and advanced analytics for a retail chain with 50+ stores."
date: 2024-01-15T00:00:00Z
image: "/images/banner.png"
categories: ["Technology", "Performance"]
author: "Deep Kulshreshtha"
tags: ["contributions", "project", "tech", "inventory", "analytics", "real-time", "retail"]
draft: false
toc: true
weight: 1
---

# Warehouse Inventory Management System

A multi-tenant inventory and order management platform for wholesalers and dealers, built with FastAPI (Python), MySQL, and Bootstrap frontend.

Site : https://inventory-247.com/static/login.html

Git : https://github.com/dkulsh/inventory_mgmt

Hosting : GCP


## Features
- User authentication (JWT)
- Role-based access (SuperAdmin, Wholesaler, Dealer, etc.)
- Product management (CRUD, image upload to GCS)
- Order management (Booked/Requested, transactional)
- Responsive Bootstrap UI
- Single deployable unit (FastAPI serves static frontend)

## Tech Stack
- Backend: FastAPI, SQLAlchemy, MySQL, Google Cloud Storage
- Frontend: HTML, CSS (Bootstrap), JavaScript

## Setup

### 1. Clone the repo

### 2. Install dependencies

```
pip install -r requirements.txt
```


### 3. Configure environment variables

(DB, GCS, JWT secret, etc.)

### 4. Run the backend

```
uvicorn backend.main:app --reload
```


### 5. Access the app

- Visit `http://localhost:8000` for the UI
- API docs at `http://localhost:8000/docs`

## Docker
Build and run with Docker:

```
docker build -t inventory-mgmt .
docker run -p 8000:8000 inventory-mgmt
```

## Notes
- Place frontend build files in `backend/static/` for serving via FastAPI.
- See `Technical Specifications.md` for full requirements.






