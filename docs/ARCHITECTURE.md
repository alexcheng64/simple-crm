# Simple CRM System Architecture

## 1. Overview

This document describes the technical architecture for the Simple CRM system, a lightweight contact management application.

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           Client Layer                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│   │   Browser   │    │   Mobile    │    │   Desktop   │            │
│   │  (React SPA)│    │  (PWA/App)  │    │   (Future)  │            │
│   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘            │
│          │                  │                  │                    │
└──────────┼──────────────────┼──────────────────┼────────────────────┘
           │                  │                  │
           └──────────────────┼──────────────────┘
                              │ HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         API Gateway Layer                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    REST API Server                           │   │
│   │                    (Node.js/Express)                         │   │
│   ├─────────────────────────────────────────────────────────────┤   │
│   │  • Authentication Middleware (JWT)                          │   │
│   │  • Request Validation                                        │   │
│   │  • Rate Limiting                                             │   │
│   │  • CORS Handling                                             │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Service Layer                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌───────────────┐  ┌───────────────┐  ┌───────────────┐          │
│   │    Auth       │  │   Contact     │  │     Tag       │          │
│   │   Service     │  │   Service     │  │   Service     │          │
│   └───────────────┘  └───────────────┘  └───────────────┘          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Data Layer                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌───────────────┐  ┌───────────────────────────────────────┐     │
│   │     ORM       │  │              Database                  │     │
│   │  (Prisma/     │──│  ┌─────────┐  ┌─────────┐            │     │
│   │   Sequelize)  │  │  │ SQLite  │  │PostgreSQL│            │     │
│   └───────────────┘  │  │  (Dev)  │  │  (Prod)  │            │     │
│                      │  └─────────┘  └─────────┘            │     │
│                      └───────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Component Architecture

### 3.1 Frontend Architecture

```
src/
├── components/           # Reusable UI components
│   ├── common/          # Buttons, inputs, modals
│   ├── contacts/        # Contact-specific components
│   └── layout/          # Header, sidebar, footer
├── pages/               # Route-level components
│   ├── Login.tsx
│   ├── Register.tsx
│   ├── ContactList.tsx
│   ├── ContactDetail.tsx
│   └── ContactForm.tsx
├── services/            # API client functions
│   ├── api.ts           # Axios instance & interceptors
│   ├── auth.ts          # Auth API calls
│   └── contacts.ts      # Contact API calls
├── store/               # State management
│   ├── authSlice.ts
│   └── contactSlice.ts
├── hooks/               # Custom React hooks
├── utils/               # Helper functions
└── types/               # TypeScript type definitions
```

**Technology Stack:**
- React 18+ with TypeScript
- React Router for navigation
- Redux Toolkit or Zustand for state management
- Axios for HTTP requests
- Tailwind CSS for styling

### 3.2 Backend Architecture

```
src/
├── config/              # Configuration management
│   ├── database.ts
│   └── environment.ts
├── controllers/         # Request handlers
│   ├── authController.ts
│   └── contactController.ts
├── middleware/          # Express middleware
│   ├── auth.ts          # JWT verification
│   ├── validation.ts    # Request validation
│   └── errorHandler.ts  # Global error handling
├── models/              # Database models
│   ├── User.ts
│   ├── Contact.ts
│   └── Tag.ts
├── routes/              # Route definitions
│   ├── auth.ts
│   └── contacts.ts
├── services/            # Business logic
│   ├── authService.ts
│   └── contactService.ts
├── utils/               # Helper functions
│   ├── hash.ts          # Password hashing
│   └── token.ts         # JWT utilities
└── app.ts               # Express app setup
```

**Technology Stack:**
- Node.js 18+ with TypeScript
- Express.js web framework
- Prisma ORM (or Sequelize)
- JWT for authentication
- bcrypt for password hashing
- Zod for validation

---

## 4. Data Flow

### 4.1 Authentication Flow

```
┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐
│ Client │     │  API   │     │ Auth   │     │   DB   │
└───┬────┘     └───┬────┘     └───┬────┘     └───┬────┘
    │              │              │              │
    │ POST /login  │              │              │
    │─────────────▶│              │              │
    │              │ validate     │              │
    │              │─────────────▶│              │
    │              │              │ find user    │
    │              │              │─────────────▶│
    │              │              │◀─────────────│
    │              │              │              │
    │              │ verify password             │
    │              │◀─────────────│              │
    │              │              │              │
    │ JWT token    │ generate JWT │              │
    │◀─────────────│◀─────────────│              │
    │              │              │              │
```

### 4.2 Contact CRUD Flow

```
┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐
│ Client │     │  API   │     │Service │     │   DB   │
└───┬────┘     └───┬────┘     └───┬────┘     └───┬────┘
    │              │              │              │
    │ GET /contacts│              │              │
    │ + JWT Header │              │              │
    │─────────────▶│              │              │
    │              │ verify JWT   │              │
    │              │──────┐       │              │
    │              │◀─────┘       │              │
    │              │              │              │
    │              │ getContacts  │              │
    │              │─────────────▶│              │
    │              │              │ SELECT *     │
    │              │              │─────────────▶│
    │              │              │◀─────────────│
    │              │◀─────────────│              │
    │ JSON response│              │              │
    │◀─────────────│              │              │
```

---

## 5. Database Architecture

### 5.1 Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐
│      users      │       │    contacts     │
├─────────────────┤       ├─────────────────┤
│ id (PK)         │       │ id (PK)         │
│ email           │◀──┐   │ user_id (FK)    │───┐
│ password_hash   │   └───│ first_name      │   │
│ name            │       │ last_name       │   │
│ created_at      │       │ email           │   │
└─────────────────┘       │ phone           │   │
                          │ company         │   │
                          │ job_title       │   │
                          │ address         │   │
                          │ city            │   │
                          │ state           │   │
                          │ postal_code     │   │
                          │ country         │   │
                          │ notes           │   │
                          │ created_at      │   │
                          │ updated_at      │   │
                          └─────────────────┘   │
                                    │           │
                                    │           │
                          ┌─────────────────┐   │
                          │  contact_tags   │   │
                          ├─────────────────┤   │
                          │ contact_id (FK) │───┘
                          │ tag_id (FK)     │───┐
                          └─────────────────┘   │
                                                │
                          ┌─────────────────┐   │
                          │      tags       │   │
                          ├─────────────────┤   │
                          │ id (PK)         │◀──┘
                          │ user_id (FK)    │
                          │ name            │
                          └─────────────────┘
```

### 5.2 Indexing Strategy

| Table | Index | Columns | Purpose |
|-------|-------|---------|---------|
| users | idx_users_email | email | Fast login lookup |
| contacts | idx_contacts_user | user_id | Filter by owner |
| contacts | idx_contacts_email | email | Search by email |
| contacts | idx_contacts_name | last_name, first_name | Sort by name |
| tags | idx_tags_user_name | user_id, name | Unique tag per user |

---

## 6. Security Architecture

### 6.1 Authentication & Authorization

```
┌─────────────────────────────────────────────────────────────┐
│                    Security Layers                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 1: Transport Security                         │    │
│  │ • HTTPS/TLS 1.3 encryption                         │    │
│  │ • HSTS headers                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 2: Authentication                             │    │
│  │ • JWT tokens (access + refresh)                    │    │
│  │ • Token expiration (15min access, 7d refresh)      │    │
│  │ • Secure httpOnly cookies                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 3: Authorization                              │    │
│  │ • User can only access own contacts                │    │
│  │ • Resource ownership validation                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 4: Input Validation                           │    │
│  │ • Schema validation (Zod)                          │    │
│  │ • SQL injection prevention (parameterized queries) │    │
│  │ • XSS prevention (output encoding)                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Password Storage

```
User Password → bcrypt(password, salt_rounds=12) → password_hash → Database
```

---

## 7. Deployment Architecture

### 7.1 Development Environment

```
┌─────────────────────────────────────────┐
│           Local Development             │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────┐    ┌─────────────┐    │
│  │   Vite      │    │   Node.js   │    │
│  │ Dev Server  │    │   Express   │    │
│  │  :3000      │───▶│   :8080     │    │
│  └─────────────┘    └──────┬──────┘    │
│                            │           │
│                     ┌──────▼──────┐    │
│                     │   SQLite    │    │
│                     │  (file DB)  │    │
│                     └─────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

### 7.2 Production Environment

```
┌──────────────────────────────────────────────────────────────────┐
│                     Production Deployment                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│    ┌─────────┐      ┌─────────────────────────────────────┐     │
│    │   CDN   │      │         Cloud Platform               │     │
│    │(Static) │      │     (Railway/Render/Fly.io)         │     │
│    └────┬────┘      ├─────────────────────────────────────┤     │
│         │           │                                      │     │
│         │           │  ┌───────────┐    ┌───────────┐     │     │
│         │           │  │  Node.js  │    │ PostgreSQL│     │     │
│         └──────────▶│  │  Server   │───▶│  Database │     │     │
│                     │  │           │    │           │     │     │
│                     │  └───────────┘    └───────────┘     │     │
│                     │                                      │     │
│                     └─────────────────────────────────────┘     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 8. API Design

### 8.1 RESTful Conventions

| Operation | HTTP Method | Endpoint | Response |
|-----------|-------------|----------|----------|
| List | GET | /api/contacts | 200 + array |
| Read | GET | /api/contacts/:id | 200 + object |
| Create | POST | /api/contacts | 201 + object |
| Update | PUT | /api/contacts/:id | 200 + object |
| Delete | DELETE | /api/contacts/:id | 204 |

### 8.2 Response Format

**Success Response:**
```json
{
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}
```

**Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      { "field": "email", "message": "Must be a valid email" }
    ]
  }
}
```

---

## 9. Technology Decisions

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Frontend | React + TypeScript | Component reusability, type safety |
| Backend | Node.js + Express | JavaScript ecosystem, fast development |
| Database | PostgreSQL | Reliability, JSON support, scalability |
| ORM | Prisma | Type safety, migrations, good DX |
| Auth | JWT | Stateless, scalable |
| Validation | Zod | TypeScript integration, runtime safety |
| Styling | Tailwind CSS | Rapid UI development, consistency |

---

## 10. Scalability Considerations

### Current Design (Single Instance)
- Handles up to 10,000 contacts per user
- Supports ~100 concurrent users

### Future Scaling Path
1. **Horizontal Scaling**: Add load balancer + multiple API instances
2. **Database**: Read replicas for search-heavy workloads
3. **Caching**: Redis for session storage and frequent queries
4. **Search**: Elasticsearch for advanced contact search
