---
title: "Passport.js TypeScript Lab Writeup"
tags:
  - writeup 
  - passportjs
  - typescript    
classes: "wide"
---


## Course Information
- **Course**: ACIT 2520
- **Student**: Ashen Perera
- **Student ID**: A01455518
- **Date**: August 2025

---
[https://github.com/wolf-569/acit2520-passport](https://github.com/wolf-569/acit2520-passport)
## Overview

This lab demonstrates the implementation of a secure web server using **Express.js** with **Passport.js** for authentication and session management. The application is built entirely in **TypeScript**, showcasing modern web development practices with proper type safety.

The primary objectives of this lab are to:
1. Understand HTTP session handling and session persistence
2. Implement middleware-based authentication flows
3. Configure multiple authentication strategies using Passport.js
4. Manage user authentication states and role-based access control
5. Secure routes with authentication and authorization middleware

---

## Project Architecture

### Technology Stack
- **Runtime**: Node.js
- **Framework**: Express.js 4.21.2
- **Authentication**: Passport.js 0.7.0
- **Language**: TypeScript
- **Template Engine**: EJS (Embedded JavaScript)
- **Session Store**: Express-session (in-memory)
- **Authentication Strategies**:
  - Local Strategy (Email/Password)
  - GitHub OAuth2 Strategy

### Directory Structure
```
├── app.ts                          # Main application entry point
├── controllers/
│   └── userController.ts          # User business logic and validation
├── middleware/
│   ├── checkAuth.ts               # Authentication guard middleware
│   ├── PassportConfig.ts          # Passport strategy configuration
│   ├── passportMiddleware.ts      # Express-Passport integration
│   └── passportStrategies/
│       ├── localStrategy.ts       # Email/password authentication
│       └── githubStrategy.ts      # OAuth2 GitHub authentication
├── models/
│   └── userModel.ts               # User data model with mock database
├── routes/
│   ├── authRoute.ts               # Authentication routes
│   └── indexRoute.ts              # Application routes
├── interfaces/
│   └── index.ts                   # TypeScript interfaces
├── views/
│   ├── login.ejs                  # Login page
│   ├── dashboard.ejs              # User dashboard
│   ├── admindashboard.ejs         # Admin dashboard
│   ├── admin-sessions.ejs         # Session management view
│   └── layout.ejs                 # Layout template
└── public/                         # Static assets
```

---

## Key Components

### 1. Authentication Middleware (`middleware/checkAuth.ts`)

Three guard middleware functions control access to protected routes:

```typescript
ensureAuthenticated()    // Redirects to login if not authenticated
forwardAuthenticated()   // Redirects to dashboard if already authenticated
ensureAdmin()           // Ensures user is authenticated AND has admin role
```

These middleware functions check `req.isAuthenticated()` and `req.user?.admin` properties set by Passport.js.

### 2. User Model (`models/userModel.ts`)

The user model provides:
- An in-memory mock database with 4 sample users
- `findOne(email)` - Retrieve user by email
- `findById(id)` - Retrieve user by ID

Sample users include admin and regular users with credentials for testing.

### 3. Local Authentication Strategy (`middleware/passportStrategies/localStrategy.ts`)

Implements email/password authentication:
- Accepts email and password fields from login form
- Validates credentials against the user model
- Provides specific error messages for failed attempts (invalid email, wrong password)
- Uses `serializeUser()` and `deserializeUser()` to manage session state

**Key Functions**:
- `serializeUser()` - Stores user ID in session (minimal data)
- `deserializeUser()` - Retrieves user from database using session ID

### 4. User Controller (`controllers/userController.ts`)

Business logic layer that:
- Validates email/password combinations
- Checks user existence by email
- Verifies password correctness
- Retrieves user information by ID

### 5. Routes

#### Authentication Routes (`routes/authRoute.ts`)
- `GET /auth/login` - Display login form
- `POST /auth/login` - Process login with Passport
- `GET /auth/logout` - Destroy session and logout
- `GET /auth/github` - Initiate GitHub OAuth
- `GET /auth/github/callback` - Handle GitHub OAuth callback

#### Index Routes (`routes/indexRoute.ts`)
- `GET /` - Welcome page
- `GET /dashboard` - User dashboard (requires authentication)
- `GET /admin/dashboard` - Admin dashboard (requires authentication + admin role)
- `GET /admin/sessions` - View all active sessions (admin only)

### 6. Session Management (`app.ts`)

Express-session configuration:
- Uses in-memory store (suitable for development/testing)
- 24-hour session expiration
- HttpOnly cookies for security
- Session ID logged for debugging

---

## Authentication Flow

### Local Authentication Flow
1. User submits login form with email/password
2. Passport's local strategy validates credentials
3. On success, `req.logIn()` creates session and serializes user ID
4. User redirected to appropriate dashboard (admin vs regular)
5. On subsequent requests, `deserializeUser()` reconstructs user object from session

### Session Persistence
- Session ID stored in httpOnly cookie
- Session data persists in memory store during server runtime
- Admin users can view all active sessions via `/admin/sessions`

---

## Security Features

1. **Session Security**
   - HttpOnly cookies prevent XSS attacks
   - Secure flag can be enabled for HTTPS
   - Session expiration (24 hours)

2. **Authentication Layers**
   - Multiple authentication strategies
   - Email validation before password check
   - Role-based access control (admin flag)

3. **Route Protection**
   - Middleware guards prevent unauthorized access
   - Redirects to login for unauthenticated requests
   - Admin routes require both authentication and admin status

---

## Testing & Credentials

### Sample Users (from `models/userModel.ts`)
```
1. Admin User
   Email: jimmy123@gmail.com
   Password: jimmy123!
   
2. Regular User
   Email: johnny123@gmail.com
   Password: johnny123!
   
3. Regular User
   Email: jonathan123@gmail.com
   Password: jonathan123!
   
4. Admin User (Test)
   Email: test@mail.com
   Password: test
```

### Testing Local Authentication
1. Navigate to `/auth/login`
2. Enter test credentials from above
3. On success, redirected to appropriate dashboard
4. Admin users see admin dashboard with session management

---

## Learning Outcomes

This lab demonstrates:

✅ **Express.js Middleware Pattern**
- Custom middleware for authentication guards
- Express-session for state management

✅ **Passport.js Architecture**
- Strategy pattern for pluggable authentication methods
- Serialization/deserialization for session handling
- Multiple authentication strategies

✅ **TypeScript Best Practices**
- Custom interfaces for type safety
- Extending Express types via declaration merging
- Type-safe middleware functions

✅ **Security Concepts**
- Session-based authentication
- Role-based access control (RBAC)
- Secure cookie handling
- Input validation

✅ **MVC Architecture**
- Controllers for business logic
- Models for data access
- Routes for request handling
- Middleware for cross-cutting concerns

---

## Potential Enhancements

1. **Database Integration**: Replace in-memory model with actual database (PostgreSQL, MongoDB)
2. **Password Hashing**: Implement bcrypt for secure password storage
3. **CSRF Protection**: Add express-csrf middleware
4. **Rate Limiting**: Implement login attempt throttling
5. **Email Verification**: Add email confirmation for registration
6. **Refresh Tokens**: Implement token-based authentication alongside sessions
7. **Logging**: Add structured logging for authentication events
8. **Error Handling**: Centralized error handling middleware
9. **Testing**: Expand Jest test suite with integration tests
10. **Production Deployment**: Use persistent session store (Redis, MongoDB)

---

## Conclusion

This Passport.js TypeScript lab successfully demonstrates enterprise-grade authentication patterns. The application showcases proper separation of concerns, type safety, and security best practices. The implementation serves as a solid foundation for building secure web applications with Node.js and Express.js.


