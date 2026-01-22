# Simple CRM System Specification

## 1. Overview

A lightweight Customer Relationship Management system focused on contact management. The system allows users to store, organize, search, and manage customer/contact information.

---

## 2. Functional Requirements

### 2.1 Contact Management

| Feature | Description |
|---------|-------------|
| Create Contact | Add new contacts with required and optional fields |
| View Contact | Display full contact details |
| Edit Contact | Modify existing contact information |
| Delete Contact | Remove contacts (with confirmation) |
| List Contacts | Display paginated list of all contacts |
| Search Contacts | Find contacts by name, email, phone, or company |

### 2.2 Contact Data Model

**Required Fields:**
- `id` - Unique identifier
- `first_name` - Contact's first name
- `last_name` - Contact's last name
- `email` - Primary email address
- `created_at` - Timestamp of creation
- `updated_at` - Timestamp of last update

**Optional Fields:**
- `phone` - Phone number
- `company` - Company/organization name
- `job_title` - Position/role
- `address` - Street address
- `city` - City
- `state` - State/province
- `postal_code` - ZIP/postal code
- `country` - Country
- `notes` - Free-form text notes
- `tags` - Array of labels for categorization

### 2.3 User Authentication

| Feature | Description |
|---------|-------------|
| Register | Create new user account |
| Login | Authenticate with email/password |
| Logout | End user session |
| Password Reset | Reset forgotten password via email |

### 2.4 User Data Model

- `id` - Unique identifier
- `email` - Login email (unique)
- `password_hash` - Securely hashed password
- `name` - Display name
- `created_at` - Account creation timestamp

---

## 3. Non-Functional Requirements

### 3.1 Performance
- Page load time: < 2 seconds
- Search results: < 500ms
- Support up to 10,000 contacts per user

### 3.2 Security
- Passwords hashed using bcrypt or Argon2
- HTTPS for all communications
- Session tokens with expiration
- Input validation and sanitization
- Protection against SQL injection and XSS

### 3.3 Usability
- Responsive design (desktop and mobile)
- Intuitive navigation
- Form validation with clear error messages

---

## 4. System Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│    Frontend     │────▶│   Backend API   │────▶│    Database     │
│   (Web Client)  │     │    (REST/JSON)  │     │   (Relational)  │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### 4.1 Recommended Database: SQLite or PostgreSQL
- SQLite for single-user/small deployments
- PostgreSQL for multi-user/production deployments

---

## 5. API Endpoints

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Create new user |
| POST | `/api/auth/login` | Authenticate user |
| POST | `/api/auth/logout` | End session |
| POST | `/api/auth/reset-password` | Request password reset |

### Contacts
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/contacts` | List all contacts (paginated) |
| GET | `/api/contacts/:id` | Get single contact |
| POST | `/api/contacts` | Create new contact |
| PUT | `/api/contacts/:id` | Update contact |
| DELETE | `/api/contacts/:id` | Delete contact |
| GET | `/api/contacts/search?q=` | Search contacts |

### Query Parameters for List
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20, max: 100)
- `sort` - Sort field (default: last_name)
- `order` - Sort order: asc/desc (default: asc)

---

## 6. Database Schema

```sql
-- Users table
CREATE TABLE users (
    id          INTEGER PRIMARY KEY,
    email       VARCHAR(255) UNIQUE NOT NULL,
    password    VARCHAR(255) NOT NULL,
    name        VARCHAR(100),
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Contacts table
CREATE TABLE contacts (
    id          INTEGER PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id),
    first_name  VARCHAR(100) NOT NULL,
    last_name   VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL,
    phone       VARCHAR(50),
    company     VARCHAR(200),
    job_title   VARCHAR(100),
    address     VARCHAR(255),
    city        VARCHAR(100),
    state       VARCHAR(100),
    postal_code VARCHAR(20),
    country     VARCHAR(100),
    notes       TEXT,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tags table
CREATE TABLE tags (
    id          INTEGER PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id),
    name        VARCHAR(50) NOT NULL,
    UNIQUE(user_id, name)
);

-- Contact-Tags junction table
CREATE TABLE contact_tags (
    contact_id  INTEGER REFERENCES contacts(id) ON DELETE CASCADE,
    tag_id      INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (contact_id, tag_id)
);

-- Indexes
CREATE INDEX idx_contacts_user ON contacts(user_id);
CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_name ON contacts(last_name, first_name);
```

---

## 7. User Interface Screens

1. **Login/Register** - Authentication forms
2. **Contact List** - Searchable, sortable table of contacts
3. **Contact Detail** - Full view of single contact
4. **Contact Form** - Create/edit contact form
5. **Settings** - User profile and preferences

---

## 8. Future Considerations (Out of Scope)

- Deal/opportunity tracking
- Task and activity management
- Email integration
- Import/export (CSV)
- Reporting and analytics
- Multi-user organizations
- API rate limiting

---

## 9. Verification

To verify the implementation:
1. Create a user account and log in
2. Create, view, edit, and delete contacts
3. Test search functionality with various queries
4. Verify pagination works correctly
5. Test form validation with invalid inputs
6. Confirm responsive design on mobile viewport
