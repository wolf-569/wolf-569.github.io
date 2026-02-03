---
title: "ACIT 2520 Session and Authentication Lab Writeup"
tags:
  - writeup
  - passportjs      
  - typescript
classes: "wide"
---


**Student:** Ashen Perera  
**Student ID:** A01455518  
**Date:** July 10, 2025

---

[https://github.com/wolf-569/acit2520-session-lab](https://github.com/wolf-569/acit2520-session-lab)

## Lab Objective

This lab demonstrates the implementation of secure session management and user authentication in a web application using Express.js and Passport.js. The primary goal is to understand how middleware, authentication strategies, and session handling work together to create a secure authentication system.

---

## Project Overview

This project is a **Node.js/Express web server** that implements user authentication with session management using Passport.js. The application supports multiple authentication strategies (local login and GitHub OAuth) and includes role-based access control (admin vs. regular users).

### Technology Stack

- **Runtime:** Node.js with TypeScript
- **Framework:** Express.js
- **Authentication:** Passport.js with Local and GitHub strategies
- **Session Management:** express-session
- **View Engine:** EJS (Embedded JavaScript Templates)
- **Testing:** Jest and Supertest

---

## Architecture and Components

### 1. **Application Entry Point** (`app.ts`)

The main application file configures:
- Express middleware (JSON parsing, URL encoding, static files)
- **Session configuration** with the following settings:
  - Secret key for session encryption
  - HTTP-only cookies (prevents JavaScript access)
  - Secure flag set to false (for local development)
  - 24-hour max age for sessions
- Passport initialization via custom middleware
- Route mounting for authentication and index routes
- Request logging middleware for debugging

### 2. **Authentication Middleware**

#### **checkAuth.ts**

Provides three authentication guards:

- **`ensureAuthenticated`**: Middleware that redirects unauthenticated users to `/auth/login`
- **`forwardAuthenticated`**: Prevents authenticated users from accessing login pages (redirects to dashboard)
- **`ensureAdmin`**: Role-based middleware that only allows admin users to proceed

```typescript
Example usage: router.get("/admin/dashboard", ensureAdmin, controller);
```

#### **PassportConfig.ts**

A configuration class that registers authentication strategies with Passport:
- Takes an array of strategy objects
- Registers each strategy using `passport.use()`
- Centralizes strategy registration logic

#### **passportMiddleware.ts**

Initializes Passport for the Express application by:
- Calling `passport.initialize()`
- Setting up `passport.session()`
- Configuring serialization/deserialization of user objects

### 3. **Authentication Strategies**

#### **Local Strategy** (`localStrategy.ts`)

Implements username/email and password authentication:
- Retrieves user from database by email
- Validates password
- Returns user object on success or failure message

#### **GitHub Strategy** (`githubStrategy.ts`)

Implements OAuth 2.0 authentication via GitHub:
- Uses GitHub credentials to authenticate users
- Allows social login without password storage

### 4. **User Model** (`models/userModel.ts`)

In-memory database containing sample users:
- **Admin User:** Jimmy Smith (jimmy123@gmail.com / jimmy123!)
- **Regular Users:** Johnny Doe, Jonathan Chen, test user
- Provides `findOne()` method to lookup users by email
- Provides `findById()` method to retrieve users by ID

### 5. **Routes**

#### **Auth Routes** (`routes/authRoute.ts`)

- **GET `/auth/login`**: Renders login page with any previous error messages
- **POST `/auth/login`**: Handles local authentication
  - Uses Passport's custom callback pattern for session message handling
  - Stores login errors in session for display on next request
  - Redirects admins to `/admin/dashboard` and regular users to `/dashboard`
- **GET `/auth/logout`**: Clears user session and redirects to login

#### **Index Routes** (`routes/indexRoute.ts`)

- **GET `/`**: Public welcome page
- **GET `/dashboard`**: User dashboard (requires authentication)
- **GET `/admin/dashboard`**: Admin dashboard (requires admin role)
- **GET `/admin/sessions`**: Session management page (admin only)

### 6. **Views** (EJS Templates)

- **`login.ejs`**: Login form with error message display
- **`layout.ejs`**: Base template with navigation
- **`dashboard.ejs`**: User dashboard
- **`admindashboard.ejs`**: Admin dashboard
- **`admin-sessions.ejs`**: Session monitoring interface

---

## Key Concepts Demonstrated

### Session Management

Sessions maintain user authentication state across requests:
- Each user receives a unique session ID stored in a cookie
- Session data is stored server-side and associated with the session ID
- The `maxAge` setting controls session expiration (24 hours)
- `httpOnly: true` prevents client-side JavaScript from accessing session cookies

### Authentication Flow

1. User submits credentials at `/auth/login`
2. Passport validates credentials using the Local strategy
3. On success, `req.logIn()` serializes the user into the session
4. User receives a session cookie
5. Subsequent requests include the session cookie
6. Passport deserializes the user from the session
7. User object is available via `req.user` throughout the request

### Role-Based Access Control

The `ensureAdmin` middleware checks:
```typescript
req.isAuthenticated() && req.user?.admin === true
```

This enables different views and functionality for admin vs. regular users.

---

## Authentication Strategy Patterns

### Custom Callback Pattern

The POST login endpoint uses the custom callback pattern:

```typescript
passport.authenticate("local", (err, user, info) => {
  // Custom logic here
})(req, res, next);
```

This approach allows:
- Custom error handling
- Session message storage
- Conditional redirection based on user role
- Fine-grained control over the authentication flow

---

## Testing

The project includes Jest and Supertest for testing:
- Unit tests for middleware
- Integration tests for authentication routes
- Session state verification tests
- Protected route access tests

---

## Security Considerations

1. **Session Secret**: Should be a strong, random value in production (currently "secret" for dev)
2. **HTTP-Only Cookies**: Prevents XSS attacks from accessing session tokens
3. **Password Hashing**: Current implementation stores passwords in plain text (not production-safe)
4. **CSRF Protection**: Should be added for form submissions
5. **HTTPS**: Secure flag should be `true` in production

---

## Learning Outcomes

This lab demonstrates:
- How Express middleware processes requests in order
- Session creation, storage, and retrieval mechanisms
- Passport.js strategy patterns and authentication flow
- Role-based authorization using middleware chains
- TypeScript interfaces for type-safe authentication
- Error handling in authentication flows
- Session persistence across HTTP requests

---

## Conclusion

This authentication system provides a foundational understanding of how modern web applications handle user sessions and authentication. The modular design with separate middleware, strategies, and controllers makes the codebase maintainable and extensible for adding new authentication methods or authorization rules.

The project successfully demonstrates the separation of concerns principle, with clear boundaries between authentication logic, session management, and route handling.
