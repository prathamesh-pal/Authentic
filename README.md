# FastAPI Authentication API

A complete authentication and authorization system built with **FastAPI**.

This project demonstrates how to design and build a secure, production-style authentication API from scratch — including user registration, password hashing, JWT authentication, protected routes, token handling, and role-based authorization.

> Built as a backend engineering project to demonstrate practical authentication, API design, security, and database skills with FastAPI.

---

## Features

### Authentication

* User registration
* User login
* Password hashing
* Password verification
* JWT access tokens
* Refresh token flow
* Token expiration
* Protected API routes
* Logout / token invalidation

### Authorization

* Role-based access control
* Protected user routes
* Admin-only routes
* Current-user dependency
* Permission checks

### User Management

* Get current authenticated user
* Update user information
* Change password
* Account activation/deactivation
* Unique email validation

### Security

* Secure password hashing
* JWT-based authentication
* Token expiration
* Protected endpoints
* Input validation
* Environment-based secrets
* CORS configuration
* Authentication dependencies

---

## Tech Stack

* **Python**
* **FastAPI**
* **Pydantic**
* **SQLAlchemy**
* **PostgreSQL**
* **JWT**
* **Passlib / Argon2**
* **Uvicorn**
* **Alembic**
* **Docker**

---

## Architecture

```text
Client
  │
  ▼
FastAPI
  │
  ├── Authentication
  │     ├── Register
  │     ├── Login
  │     ├── Access Token
  │     └── Refresh Token
  │
  ├── Authorization
  │     ├── User
  │     └── Admin
  │
  ├── API Routes
  │     ├── /auth
  │     ├── /users
  │     └── /admin
  │
  ├── Services
  │     ├── Password Hashing
  │     ├── JWT
  │     └── User Management
  │
  ▼
PostgreSQL
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

The client sends user credentials:

```http
POST /auth/register
```

```json
{
  "email": "user@example.com",
  "password": "strong-password"
}
```

The password is hashed before being stored in the database.

The plaintext password is never stored.

---

### 2. Login

```http
POST /auth/login
```

The server:

1. Finds the user
2. Verifies the password
3. Creates an access token
4. Creates a refresh token
5. Returns the authentication credentials

Example response:

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "bearer"
}
```

---

### 3. Access Protected Routes

The client sends the access token:

```http
GET /users/me
Authorization: Bearer <access_token>
```

FastAPI's authentication dependency validates the token and identifies the current user.

---

### 4. Refresh Token

When the access token expires:

```http
POST /auth/refresh
```

The refresh token can be exchanged for a new access token.

This avoids requiring the user to log in again every time a short-lived access token expires.

---

## API Endpoints

### Authentication

| Method | Endpoint         | Description          | Auth          |
| ------ | ---------------- | -------------------- | ------------- |
| POST   | `/auth/register` | Create account       | No            |
| POST   | `/auth/login`    | Login                | No            |
| POST   | `/auth/refresh`  | Refresh access token | Refresh Token |
| POST   | `/auth/logout`   | Logout               | Yes           |

### Users

| Method | Endpoint             | Description      | Auth |
| ------ | -------------------- | ---------------- | ---- |
| GET    | `/users/me`          | Get current user | Yes  |
| PATCH  | `/users/me`          | Update profile   | Yes  |
| PATCH  | `/users/me/password` | Change password  | Yes  |
| DELETE | `/users/me`          | Delete account   | Yes  |

### Admin

| Method | Endpoint            | Description | Auth  |
| ------ | ------------------- | ----------- | ----- |
| GET    | `/admin/users`      | List users  | Admin |
| GET    | `/admin/users/{id}` | Get user    | Admin |
| PATCH  | `/admin/users/{id}` | Update user | Admin |
| DELETE | `/admin/users/{id}` | Delete user | Admin |

---

## Database Model

Example `User` model:

```text
User
├── id
├── email
├── password_hash
├── role
├── is_active
├── created_at
└── updated_at
```

Passwords are stored as secure hashes rather than plaintext values.

---

## JWT Authentication

The API uses JWTs to authenticate requests.

A typical access token contains claims such as:

```json
{
  "sub": "user-id",
  "role": "user",
  "exp": 1790000000
}
```

The API validates:

* Token signature
* Token expiration
* User identity
* User status
* Required permissions

---

## Authorization

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to do?**

For example:

```text
Authenticated User
        │
        ├── GET /users/me       ✓
        ├── PATCH /users/me     ✓
        └── GET /admin/users    ✗

Admin
        │
        ├── GET /users/me       ✓
        ├── PATCH /users/me     ✓
        └── GET /admin/users    ✓
```

---

## Environment Variables

Create a `.env` file:

```env
DATABASE_URL=postgresql://user:password@localhost/auth_db

SECRET_KEY=your-secret-key

ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7
```

Never commit secrets or `.env` files to Git.

---

## Running Locally

### Clone the repository

```bash
git clone https://github.com/yourusername/fastapi-authentication.git

cd fastapi-authentication
```

### Create a virtual environment

```bash
python -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

### Install dependencies

```bash
pip install -r requirements.txt
```

### Configure environment variables

```bash
cp .env.example .env
```

Update the values inside `.env`.

### Run the API

```bash
uvicorn app.main:app --reload
```

The API will be available at:

```text
http://localhost:8000
```

---

## API Documentation

FastAPI automatically generates interactive API documentation.

### Swagger UI

```text
http://localhost:8000/docs
```

### ReDoc

```text
http://localhost:8000/redoc
```

---

## Example Authentication Flow

```text
Register
   │
   ▼
POST /auth/register
   │
   ▼
User Created
   │
   ▼
POST /auth/login
   │
   ▼
Access + Refresh Tokens
   │
   ▼
GET /users/me
   │
   ▼
JWT Validation
   │
   ▼
Authenticated User
```

---

## Security Considerations

This project focuses on several important authentication concepts:

* Passwords are never stored directly
* Password hashing is performed before database storage
* Access tokens are short-lived
* Refresh tokens provide longer-lived sessions
* Protected routes require valid authentication
* Role-based authorization prevents unauthorized access
* Secrets are stored through environment variables
* Request data is validated with Pydantic
* Database access is separated from API routes

For a production deployment, additional measures may include rate limiting, email verification, password reset flows, token rotation/revocation, audit logging, CSRF protections where applicable, and additional monitoring.

---

## What I Learned

Building this project helped me understand how authentication works beyond simply adding a login form.

### Backend

* Designing REST APIs with FastAPI
* Dependency injection
* Pydantic validation
* SQLAlchemy
* PostgreSQL
* Database relationships
* Database migrations

### Security

* Password hashing
* JWT authentication
* Access vs refresh tokens
* Authentication vs authorization
* Role-based access control
* Token expiration
* Secure secret management

### API Design

* HTTP methods and status codes
* Authentication middleware/dependencies
* Protected routes
* Error handling
* API schemas
* Separation of concerns

---

## Project Goals

The goal of this project is not to build a frontend authentication page.

The goal is to understand and demonstrate what happens **behind the login button**.

```text
User
 │
 ▼
API Request
 │
 ▼
Validation
 │
 ▼
Authentication
 │
 ▼
Authorization
 │
 ▼
Business Logic
 │
 ▼
Database
 │
 ▼
Response
```

---

## Future Improvements

* [ ] Email verification
* [ ] Forgot/reset password
* [ ] Refresh-token rotation
* [ ] Token revocation
* [ ] Login rate limiting
* [ ] Account lockout
* [ ] OAuth2 / Google authentication
* [ ] Audit logs
* [ ] Redis-based session/token management
* [ ] Docker Compose
* [ ] Automated tests
* [ ] CI/CD
* [ ] Production deployment

---

## License

This project is open source and available under the MIT License.

```

### Portfolio positioning

For your portfolio, I’d describe it as:

> **FastAPI Authentication API** — A production-style authentication and authorization backend implementing JWT access/refresh tokens, secure password hashing, protected routes, role-based access control, PostgreSQL, and API validation.

That makes it clear that the project is specifically demonstrating **backend engineering + security fundamentals**, rather than pretending it is a complete SaaS application.
```
