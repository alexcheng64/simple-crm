# Simple CRM System Architecture

## 1. Overview

This document describes the technical architecture for the Simple CRM system, a lightweight contact management application built with React and Supabase.

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
│                         Supabase Platform                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                      Auth Service                            │   │
│   │  • Email/Password authentication                            │   │
│   │  • JWT session management                                   │   │
│   │  • Password reset flows                                     │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    PostgREST API                             │   │
│   │  • Auto-generated REST endpoints                            │   │
│   │  • Query filtering, sorting, pagination                     │   │
│   │  • Request validation                                       │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    PostgreSQL Database                       │   │
│   │  • Row Level Security (RLS) policies                        │   │
│   │  • User data isolation                                      │   │
│   │  • Automatic timestamps                                     │   │
│   └─────────────────────────────────────────────────────────────┘   │
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
├── services/            # Supabase client functions
│   ├── supabase.ts      # Supabase client initialization
│   ├── auth.ts          # Auth service functions
│   └── contacts.ts      # Contact CRUD functions
├── store/               # State management
│   ├── authSlice.ts
│   └── contactSlice.ts
├── hooks/               # Custom React hooks
│   ├── useAuth.ts       # Auth state and methods
│   └── useContacts.ts   # Contact data fetching
├── utils/               # Helper functions
└── types/               # TypeScript type definitions
```

**Technology Stack:**
- React 18+ with TypeScript
- React Router for navigation
- Redux Toolkit or Zustand for state management
- `@supabase/supabase-js` for backend communication
- Tailwind CSS for styling

### 3.2 Supabase Client Setup

```typescript
// src/services/supabase.ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

---

## 4. Data Flow

### 4.1 Authentication Flow

```
┌────────┐                    ┌─────────────────────────────────────┐
│ Client │                    │            Supabase                  │
└───┬────┘                    │  ┌────────┐         ┌────────────┐  │
    │                         │  │  Auth  │         │  Database  │  │
    │                         │  └───┬────┘         └─────┬──────┘  │
    │                         │      │                    │         │
    │ signInWithPassword()    │      │                    │         │
    │────────────────────────▶│      │                    │         │
    │                         │      │ verify credentials │         │
    │                         │      │───────────────────▶│         │
    │                         │      │◀───────────────────│         │
    │                         │      │                    │         │
    │                         │      │ generate JWT       │         │
    │                         │      │──────┐             │         │
    │                         │      │◀─────┘             │         │
    │                         │      │                    │         │
    │ { user, session }       │      │                    │         │
    │◀────────────────────────│      │                    │         │
    │                         │      │                    │         │
    │ Store session in        │      │                    │         │
    │ localStorage            │      │                    │         │
    │                         │      │                    │         │
    └─────────────────────────┴──────┴────────────────────┴─────────┘
```

### 4.2 Contact CRUD Flow

```
┌────────┐                    ┌─────────────────────────────────────┐
│ Client │                    │            Supabase                  │
└───┬────┘                    │  ┌─────────┐        ┌────────────┐  │
    │                         │  │PostgREST│        │ PostgreSQL │  │
    │                         │  └────┬────┘        │   + RLS    │  │
    │                         │       │             └─────┬──────┘  │
    │                         │       │                   │         │
    │ supabase.from('contacts')       │                   │         │
    │   .select('*')          │       │                   │         │
    │   + JWT in header       │       │                   │         │
    │────────────────────────▶│       │                   │         │
    │                         │       │ Extract user_id   │         │
    │                         │       │ from JWT          │         │
    │                         │       │                   │         │
    │                         │       │ SELECT * FROM     │         │
    │                         │       │ contacts WHERE    │         │
    │                         │       │ RLS policy passes │         │
    │                         │       │──────────────────▶│         │
    │                         │       │                   │         │
    │                         │       │ Only user's       │         │
    │                         │       │ contacts returned │         │
    │                         │       │◀──────────────────│         │
    │                         │       │                   │         │
    │ { data: contacts[] }    │       │                   │         │
    │◀────────────────────────│       │                   │         │
    │                         │       │                   │         │
    └─────────────────────────┴───────┴───────────────────┴─────────┘
```

---

## 5. Database Architecture

### 5.1 Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐
│   auth.users    │       │    contacts     │
│   (Supabase)    │       ├─────────────────┤
├─────────────────┤       │ id (PK, UUID)   │
│ id (PK, UUID)   │◀──┐   │ user_id (FK)    │───┐
│ email           │   └───│ first_name      │   │
│ encrypted_pass  │       │ last_name       │   │
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
                          │ id (PK, UUID)   │◀──┘
                          │ user_id (FK)    │
                          │ name            │
                          └─────────────────┘
```

### 5.2 Row Level Security (RLS)

RLS policies ensure data isolation at the database level. Each user can only access their own data.

| Table | Policy | Rule |
|-------|--------|------|
| contacts | SELECT | `auth.uid() = user_id` |
| contacts | INSERT | `auth.uid() = user_id` |
| contacts | UPDATE | `auth.uid() = user_id` |
| contacts | DELETE | `auth.uid() = user_id` |
| tags | SELECT/INSERT/UPDATE/DELETE | `auth.uid() = user_id` |
| contact_tags | ALL | Contact must belong to current user |

### 5.3 Indexing Strategy

| Table | Index | Columns | Purpose |
|-------|-------|---------|---------|
| contacts | idx_contacts_user | user_id | Filter by owner |
| contacts | idx_contacts_email | email | Search by email |
| contacts | idx_contacts_name | last_name, first_name | Sort by name |

---

## 6. Security Architecture

### 6.1 Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Security Layers                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 1: Transport Security                         │    │
│  │ • HTTPS/TLS encryption (enforced by Supabase)      │    │
│  │ • Secure WebSocket connections                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 2: Authentication (Supabase Auth)             │    │
│  │ • JWT tokens with automatic refresh                 │    │
│  │ • Secure password hashing (bcrypt)                  │    │
│  │ • Session management                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 3: Authorization (Row Level Security)         │    │
│  │ • Database-level access control                    │    │
│  │ • User can only access own data                    │    │
│  │ • Policies enforced on every query                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Layer 4: Input Validation                           │    │
│  │ • Frontend validation (Zod/Yup)                    │    │
│  │ • PostgREST request validation                     │    │
│  │ • Database constraints                              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Authentication Flow

1. User submits credentials to Supabase Auth
2. Supabase validates and returns JWT + refresh token
3. Client stores session (managed by Supabase client)
4. JWT automatically included in subsequent API requests
5. PostgREST extracts `auth.uid()` from JWT for RLS

---

## 7. Deployment Architecture

### 7.1 Development Environment

```
┌─────────────────────────────────────────────────────────────┐
│                   Local Development                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────┐                                        │
│  │   Vite Dev      │                                        │
│  │   Server        │                                        │
│  │   :5173         │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           │ HTTPS                                            │
│           ▼                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Supabase (Cloud or Local)                 │    │
│  │  • Auth Service                                      │    │
│  │  • PostgREST API                                    │    │
│  │  • PostgreSQL Database                               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Production Environment

```
┌──────────────────────────────────────────────────────────────────┐
│                     Production Deployment                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌─────────────────────────┐                                    │
│   │   Static Hosting        │                                    │
│   │   (Vercel/Netlify/      │                                    │
│   │    Cloudflare Pages)    │                                    │
│   │                         │                                    │
│   │  • React SPA bundle     │                                    │
│   │  • CDN distribution     │                                    │
│   │  • Automatic SSL        │                                    │
│   └───────────┬─────────────┘                                    │
│               │                                                   │
│               │ HTTPS                                             │
│               ▼                                                   │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │              Supabase Cloud (Fully Managed)              │    │
│   │                                                          │    │
│   │  ┌────────────┐  ┌────────────┐  ┌────────────────┐     │    │
│   │  │   Auth     │  │  PostgREST │  │   PostgreSQL   │     │    │
│   │  │  Service   │  │    API     │  │   Database     │     │    │
│   │  │            │  │            │  │                │     │    │
│   │  │ • Sign up  │  │ • REST API │  │ • Managed DB   │     │    │
│   │  │ • Sign in  │  │ • Realtime │  │ • Backups      │     │    │
│   │  │ • Sessions │  │ • Storage  │  │ • RLS Policies │     │    │
│   │  └────────────┘  └────────────┘  └────────────────┘     │    │
│   │                                                          │    │
│   └─────────────────────────────────────────────────────────┘    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Deployment is simplified:**
- Frontend: Deploy static React build to any CDN/static host
- Backend: Supabase is fully managed (no servers to deploy)

---

## 8. API Design

### 8.1 Supabase Client Operations

| Operation | Supabase Client Method |
|-----------|------------------------|
| List | `supabase.from('contacts').select('*')` |
| Read | `supabase.from('contacts').select('*').eq('id', id).single()` |
| Create | `supabase.from('contacts').insert(data).select().single()` |
| Update | `supabase.from('contacts').update(data).eq('id', id).select().single()` |
| Delete | `supabase.from('contacts').delete().eq('id', id)` |

### 8.2 Query Features

```typescript
// Pagination
.range(0, 19)  // Items 0-19 (first 20)

// Sorting
.order('last_name', { ascending: true })

// Filtering
.eq('company', 'Acme Inc')
.ilike('email', '%@gmail.com')

// Search (multiple fields)
.or('first_name.ilike.%john%,last_name.ilike.%john%,email.ilike.%john%')

// Select specific columns
.select('id, first_name, last_name, email')
```

### 8.3 Response Format

**Success Response:**
```typescript
{
  data: Contact[] | Contact | null,
  error: null,
  count: number | null  // When using .select('*', { count: 'exact' })
}
```

**Error Response:**
```typescript
{
  data: null,
  error: {
    message: "Row not found",
    details: null,
    hint: null,
    code: "PGRST116"
  }
}
```

---

## 9. Technology Decisions

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Frontend | React + TypeScript | Component reusability, type safety |
| Backend | Supabase | Managed PostgreSQL, auth, instant APIs, RLS |
| Database | PostgreSQL (Supabase) | Reliability, JSON support, RLS capabilities |
| Auth | Supabase Auth | Built-in, secure, handles JWT/sessions |
| API | PostgREST (via Supabase) | Auto-generated from schema, no code needed |
| Authorization | Row Level Security | Database-level, cannot be bypassed |
| Client SDK | @supabase/supabase-js | Type-safe, handles auth state |
| Styling | Tailwind CSS | Rapid UI development, consistency |

---

## 10. Scalability Considerations

### Current Design
- Handles up to 10,000 contacts per user
- Supports ~100 concurrent users
- Supabase free tier: 500MB database, 50,000 monthly active users

### Scaling with Supabase
1. **Database**: Upgrade Supabase plan for more storage and connections
2. **Performance**: Add database indexes, optimize queries
3. **Realtime**: Enable Supabase Realtime for live updates (optional)
4. **Edge Functions**: Add Supabase Edge Functions for complex logic if needed
5. **Search**: Use PostgreSQL full-text search or integrate external search service
