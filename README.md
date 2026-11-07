# FastAPI Authentication API

A production-style authentication and authorization backend built with **FastAPI**, **PostgreSQL**, and **JWT**.

It covers the full path behind a login button: user registration, password hashing, access and refresh tokens, protected routes, and role-based access control.

> A backend engineering project demonstrating practical authentication, API design, security fundamentals, and database skills.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Authentication Flow](#authentication-flow)
- [API Endpoints](#api-endpoints)
- [Database Model](#database-model)
- [JWT Details](#jwt-details)
- [Authorization](#authorization)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Security Considerations](#security-considerations)
- [What I Learned](#what-i-learned)
- [Roadmap](#roadmap)
- [License](#license)

---

## Features

**Authentication**
- User registration and login
- Password hashing and verification (Argon2 via Passlib)
- JWT access tokens with short expiry
- Refresh token flow
- Protected routes via FastAPI dependencies
- Logout endpoint

**Authorization**
- Role-based access control (`user`, `admin`)
- Current-user dependency
- Admin-only routes with permission checks

**User management**
- Get and update the current user
- Change password
- Account activation / deactivation
- Unique email validation

**Security**
- No plaintext passwords, ever
- Environment-based secrets
- Pydantic input validation
- Configurable CORS

---

## Tech Stack

| Layer | Tools |
| --- | --- |
| Framework | FastAPI, Uvicorn |
| Validation | Pydantic |
| Database | PostgreSQL, SQLAlchemy, Alembic |
| Auth | JWT, Passlib (Argon2) |
| Tooling | Docker |

---

## Architecture

```text
Client
  │
  ▼
FastAPI
  │
  ├── Routes
  │     ├── /auth    (register, login, refresh, logout)
  │     ├── /users   (current-user management)
  │     └── /admin   (admin-only management)
  │
  ├── Dependencies   (token validation, current user, role checks)
  │
  ├── Services       (auth, user management)
  │
  ├── Core           (password hashing, JWT, config)
  │
  ▼
PostgreSQL
```

Request lifecycle:

```text
Request → Validation → Authentication → Authorization → Business Logic → Database → Response
```

---

## Project Structure

```text
app/
├── main.py
│
├── api/
│   ├── dependencies.py
│   └── routes/
│       ├── auth.py
│       ├── users.py
│       └── admin.py
│
├── core/
│   ├── config.py
│   ├── security.py
│   └── database.py
│
├── models/
│   └── user.py
│
├── schemas/
│   ├── auth.py
│   └── user.py
│
├── services/
│   ├── auth.py
│   └── user.py
│
└── migrations/
```

---

## Authentication Flow

### 1. Register

```http
POST /auth/register
```

```json
{
  "email": "user@example.com",
  "password": "strong-password"
}
```

The password is hashed before it is stored. The plaintext password is never saved.

### 2. Login

```http
POST /auth/login
```

The server finds the user, verifies the password, and issues an access token and a refresh token:

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "bearer"
}
```

### 3. Access a protected route

```http
GET /users/me
Authorization: Bearer <access_token>
```

An authentication dependency validates the token and resolves the current user.

### 4. Refresh the access token

```http
POST /auth/refresh
```

When the short-lived access token expires, the refresh token is exchanged for a new one, so the user doesn't have to log in again.

---

## API Endpoints

### Authentication

| Method | Endpoint | Description | Auth |
| --- | --- | --- | --- |
| POST | `/auth/register` | Create account | None |
| POST | `/auth/login` | Log in | None |
| POST | `/auth/refresh` | Refresh access token | Refresh token |
| POST | `/auth/logout` | Log out | Required |

### Users

| Method | Endpoint | Description | Auth |
| --- | --- | --- | --- |
| GET | `/users/me` | Get current user | Required |
| PATCH | `/users/me` | Update profile | Required |
| PATCH | `/users/me/password` | Change password | Required |
| DELETE | `/users/me` | Delete account | Required |

### Admin

| Method | Endpoint | Description | Auth |
| --- | --- | --- | --- |
| GET | `/admin/users` | List users | Admin |
| GET | `/admin/users/{id}` | Get user | Admin |
| PATCH | `/admin/users/{id}` | Update user | Admin |
| DELETE | `/admin/users/{id}` | Delete user | Admin |

---

## Database Model

```text
User
├── id
├── email          (unique)
├── password_hash
├── role
├── is_active
├── created_at
└── updated_at
```

---

## JWT Details

A typical access token payload:

```json
{
  "sub": "user-id",
  "role": "user",
  "exp": 1790000000
}
```

On every protected request the API checks the token signature, expiration, user identity, user status (active), and required permissions.

---

## Authorization

Authentication answers **who are you?** Authorization answers **what are you allowed to do?**

| Endpoint | User | Admin |
| --- | :---: | :---: |
| `GET /users/me` | ✓ | ✓ |
| `PATCH /users/me` | ✓ | ✓ |
| `GET /admin/users` | ✗ | ✓ |

---

## Getting Started

### Environment variables

Create a `.env` file (see `.env.example`):

```env
DATABASE_URL=postgresql://user:password@localhost/auth_db
SECRET_KEY=your-secret-key
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7
```

> Never commit `.env` files or secrets to Git.

### Run locally

```bash
# Clone
git clone https://github.com/yourusername/fastapi-authentication.git
cd fastapi-authentication

# Virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# Dependencies
pip install -r requirements.txt

# Configuration
cp .env.example .env             # then edit the values

# Database migrations
alembic upgrade head

# Start the API
uvicorn app.main:app --reload
```

The API runs at `http://localhost:8000`.

---

## API Documentation

Interactive docs are generated automatically:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

---

## Security Considerations

- Passwords are hashed before storage and never kept in plaintext
- Access tokens are short-lived; refresh tokens provide longer sessions
- Protected routes require valid authentication
- Role-based authorization blocks unauthorized access
- Secrets live in environment variables
- Request data is validated with Pydantic
- Database access is kept separate from route handlers

For production, consider adding rate limiting, email verification, password reset, refresh-token rotation and revocation, audit logging, CSRF protection where applicable, and monitoring.

---

## What I Learned

**Backend:** REST API design with FastAPI, dependency injection, Pydantic validation, SQLAlchemy, PostgreSQL, and migrations.

**Security:** password hashing, JWT authentication, access vs. refresh tokens, authentication vs. authorization, RBAC, token expiry, and secret management.

**API design:** HTTP methods and status codes, authentication dependencies, error handling, schemas, and separation of concerns.

---

## Roadmap

- [ ] Email verification
- [ ] Forgot / reset password
- [ ] Refresh-token rotation
- [ ] Token revocation
- [ ] Login rate limiting
- [ ] Account lockout
- [ ] OAuth2 / Google sign-in
- [ ] Audit logs
- [ ] Redis-based session/token management
- [ ] Docker Compose
- [ ] Automated tests
- [ ] CI/CD
- [ ] Production deployment

---

## License

Released under the [MIT License](LICENSE).
