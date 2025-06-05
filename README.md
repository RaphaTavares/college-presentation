# Workshop: Building Your First Login Backend

This project demonstrates how to build a complete authentication backend using Node.js, Express.js, and PostgreSQL with REST API principles.

## 🎯 Learning Objectives

By the end of this workshop, you will understand:
- How to build REST API endpoints
- User authentication with JWT tokens
- Password hashing with bcrypt
- Database operations with PostgreSQL
- Input validation and error handling
- Docker for database management

## 🛠 Technology Stack

- **Node.js** - JavaScript runtime
- **Express.js** - Web framework
- **PostgreSQL** - Database (running in Docker)
- **JWT** - Authentication tokens
- **bcryptjs** - Password hashing
- **express-validator** - Input validation

## 📋 Prerequisites

- Node.js (v14 or higher)
- Docker and Docker Compose
- Basic knowledge of JavaScript and REST APIs

## 🚀 Getting Started

### 1. Install Dependencies

```bash
npm install
```

### 2. Start PostgreSQL Database

```bash
npm run db:up
```

This will start a PostgreSQL container with the database already initialized.

### 3. Start the Server

```bash
npm start
```

Or for development with auto-reload:

```bash
npm run dev
```

The server will start on `http://localhost:3000`

## 📡 API Endpoints

### Authentication Endpoints

#### 1. Sign Up
- **POST** `/api/auth/signup`
- **Description**: Register a new user
- **Body**:
```json
{
  "email": "user@example.com",
  "password": "password123",
  "firstName": "John",
  "lastName": "Doe"
}
```

#### 2. Sign In
- **POST** `/api/auth/signin`
- **Description**: Login with email and password
- **Body**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

### Profile Endpoints (Protected)

#### 3. View Profile
- **GET** `/api/profile`
- **Description**: Get current user's profile
- **Headers**: `Authorization: Bearer <token>`

#### 4. Update Profile
- **PUT** `/api/profile`
- **Description**: Update user's profile information
- **Headers**: `Authorization: Bearer <token>`
- **Body**:
```json
{
  "firstName": "Jane",
  "lastName": "Smith"
}
```

## 🧪 Testing the API

### Using curl

1. **Sign up a new user**:
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123","firstName":"Test","lastName":"User"}'
```

2. **Sign in** (save the token from response):
```bash
curl -X POST http://localhost:3000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'
```

3. **View profile** (replace TOKEN with actual token):
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer TOKEN"
```

4. **Update profile**:
```bash
curl -X PUT http://localhost:3000/api/profile \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"firstName":"Updated","lastName":"Name"}'
```

## 🗂 Project Structure

```
├── config/
│   └── database.js          # PostgreSQL connection
├── middleware/
│   └── auth.js              # JWT authentication middleware
├── models/
│   └── User.js              # User model and database operations
├── routes/
│   ├── auth.js              # Authentication routes
│   └── profile.js           # Profile routes
├── docker-compose.yml       # PostgreSQL Docker setup
├── init.sql                 # Database initialization
├── server.js                # Main server file
├── .env                     # Environment variables
└── README.md                # This file
```

## 🔧 Useful Commands

- `npm run db:up` - Start PostgreSQL container
- `npm run db:down` - Stop PostgreSQL container
- `npm run db:logs` - View database logs
- `npm start` - Start the server
- `npm run dev` - Start server with auto-reload

## 🔐 Security Features

- Password hashing with bcrypt (12 salt rounds)
- JWT token-based authentication
- Input validation and sanitization
- SQL injection protection with parameterized queries
- CORS enabled for cross-origin requests

## 📚 Key Concepts Covered

1. **REST API Design** - Proper HTTP methods and status codes
2. **Authentication Flow** - Registration, login, and protected routes
3. **Database Design** - User table with proper indexing
4. **Security Best Practices** - Password hashing, JWT tokens
5. **Error Handling** - Consistent error responses
6. **Input Validation** - Server-side validation with express-validator

## 🎓 Workshop Exercises

Try these exercises to deepen your understanding:

1. Add email verification during signup
2. Implement password reset functionality
3. Add user roles and permissions
4. Create additional profile fields
5. Add rate limiting to prevent abuse
6. Implement refresh tokens

## 🐛 Troubleshooting

- **Database connection issues**: Make sure Docker is running and PostgreSQL container is up
- **Port conflicts**: Change the PORT in `.env` if 3000 is already in use
- **JWT errors**: Check that JWT_SECRET is set in `.env`

Happy coding! 🚀
