---
title: "Apartment Management System"
meta_title: "Apartment Management System - Property Management and Tenant Portal"
description: "Built a comprehensive apartment management system with tenant portal, maintenance tracking, rent collection, and property analytics for a real estate management company."
date: 2024-01-20T00:00:00Z
image: "/images/banner.png"
categories: ["Technology", "Performance"]
author: "Deep Kulshreshtha"
tags: ["contributions", "project", "tech", "apartment", "management", "real-estate", "portal"]
draft: false
toc: true
weight: 2
---

# Apartment Management System

A web-based application designed to manage apartments, users, transactions, and maintenance activities in a residential society. It includes user authentication, a RESTful API, a responsive UI, and soft delete functionality to ensure data integrity.

Git : https://github.com/dkulsh/ApartmentMgmt


## Features

1. **User Authentication and Authorization**
   - Secure login with username and password.
   - Role-based access (Admin vs. Regular users).
2. **RESTful API**
   - Comprehensive CRUD (Create, Read, Update, Delete) operations for all entities.
   - Soft delete support (mark `isDeleted = true` instead of permanent deletion).
3. **Responsive UI**
   - Built with Material Design principles for mobile-friendly design.
4. **Soft Delete Functionality**
   - All entities include an `isDeleted` field to maintain data integrity.

## Tech Stack

- **Database**: MySQL
- **Backend**: Python with FastAPI
- **Frontend**: HTML, CSS (with Material Design principles)

## Prerequisites

- Python 3.8 or higher
- MySQL Server
- pip (Python package manager)

## Installation

1. **Clone the repository**

```
git clone https://github.com/yourusername/apartment-management-system.git
cd apartment-management-system
```

2. **Create a virtual environment**

```
python -m venv venv
```

3. **Activate the virtual environment**

On Windows:
```
venv\Scripts\activate
```

On macOS/Linux:
```
source venv/bin/activate
```

4. **Install dependencies**

```
pip install -r requirements.txt
```

5. **Set up the database**

Create a MySQL database named `apartment_mgmt`:

```
CREATE DATABASE apartment_mgmt;
```

6. **Configure the environment variables**

Update the `.env` file with your database credentials:

```
DATABASE_URL=mysql+pymysql://username:password@localhost/apartment_mgmt
SECRET_KEY=09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

7. **Initialize the database with sample data**

```
python init_db.py
```

## Running the Application

1. **Start the server**

```
python main.py
```

2. **Access the application**

Open your browser and navigate to:
```
http://localhost:8000
```

## Default Login Credentials

- **Admin User**:
  - Username: admin
  - Password: admin123

- **Regular User**:
  - Username: user
  - Password: user123

## API Documentation

Once the application is running, you can access the API documentation at:
```
http://localhost:8000/docs
```

## Project Structure

```
apartment-management-system/
├── app/
│   ├── api/
│   │   ├── auth.py
│   │   ├── users.py
│   │   ├── apartments.py
│   │   ├── transactions.py
│   │   ├── maintenance.py
│   │   ├── summary.py
│   │   └── dashboard.py
│   ├── database/
│   │   └── database.py
│   ├── models/
│   │   └── models.py
│   ├── schemas/
│   │   └── schemas.py
│   ├── static/
│   │   └── css/
│   │       └── styles.css
│   └── templates/
│       ├── base.html
│       ├── login.html
│       ├── dashboard.html
│       ├── apartments.html
│       ├── users.html
│       ├── maintenance.html
│       ├── transactions.html
│       └── summary.html
├── .env
├── main.py
├── init_db.py
├── requirements.txt
└── README.md
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

