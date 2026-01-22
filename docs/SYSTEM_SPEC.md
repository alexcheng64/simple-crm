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
- `id` - Unique identifier (UUID)
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

User data is managed by Supabase Auth (`auth.users` table):
- `id` - Unique identifier (UUID)
- `email` - Login email (unique)
- `created_at` - Account creation timestamp
- Passwords are securely hashed and managed by Supabase

---

## 3. Non-Functional Requirements

### 3.1 Performance
- Page load time: < 2 seconds
- Search results: < 500ms
- Support up to 10,000 contacts per user

### 3.2 Security
- Passwords securely hashed and managed by Supabase Auth
- HTTPS for all communications (enforced by Supabase)
- Session management handled by Supabase Auth (JWT-based)
- Row Level Security (RLS) for data isolation at database level
- Input validation and sanitization on frontend
- Protection against SQL injection via parameterized queries (PostgREST)

### 3.3 Usability
- Responsive design (desktop and mobile)
- Intuitive navigation
- Form validation with clear error messages

---

## 4. System Architecture

```
┌─────────────────┐         ┌─────────────────────────────┐
│                 │         │         Supabase            │
│    Frontend     │  HTTPS  │  ┌─────────────────────┐   │
│   (React SPA)   │────────▶│  │   Auth Service      │   │
│                 │         │  ├─────────────────────┤   │
│                 │         │  │   PostgREST API     │   │
└─────────────────┘         │  ├─────────────────────┤   │
                            │  │   PostgreSQL + RLS  │   │
                            │  └─────────────────────┘   │
                            └─────────────────────────────┘
```

### 4.1 Backend: Supabase (Backend-as-a-Service)

Supabase provides all backend functionality:
- **PostgreSQL Database** - Hosted, managed PostgreSQL
- **Authentication** - Built-in auth with email/password, OAuth, magic links
- **Auto-generated REST API** - PostgREST provides instant CRUD APIs
- **Row Level Security (RLS)** - Database-level authorization
- **Realtime** - Optional WebSocket subscriptions for live updates

No custom backend server is required.

---

## 5. Data Access (Supabase Client SDK)

### Authentication

```typescript
// Register
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123'
})

// Login
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password123'
})

// Logout
const { error } = await supabase.auth.signOut()

// Password Reset
const { error } = await supabase.auth.resetPasswordForEmail('user@example.com')
```

### Contacts

```typescript
// List contacts (paginated)
const { data, error } = await supabase
  .from('contacts')
  .select('*')
  .order('last_name', { ascending: true })
  .range(0, 19)  // First 20 items

// Get single contact
const { data, error } = await supabase
  .from('contacts')
  .select('*')
  .eq('id', contactId)
  .single()

// Create contact
const { data, error } = await supabase
  .from('contacts')
  .insert({ first_name, last_name, email, ... })
  .select()
  .single()

// Update contact
const { data, error } = await supabase
  .from('contacts')
  .update({ first_name, last_name, ... })
  .eq('id', contactId)
  .select()
  .single()

// Delete contact
const { error } = await supabase
  .from('contacts')
  .delete()
  .eq('id', contactId)

// Search contacts
const { data, error } = await supabase
  .from('contacts')
  .select('*')
  .or(`first_name.ilike.%${query}%,last_name.ilike.%${query}%,email.ilike.%${query}%,company.ilike.%${query}%`)
```

---

## 6. Database Schema

```sql
-- Contacts table (users managed by Supabase Auth)
CREATE TABLE contacts (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
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
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Tags table
CREATE TABLE tags (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    name        VARCHAR(50) NOT NULL,
    UNIQUE(user_id, name)
);

-- Contact-Tags junction table
CREATE TABLE contact_tags (
    contact_id  UUID REFERENCES contacts(id) ON DELETE CASCADE,
    tag_id      UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (contact_id, tag_id)
);

-- Indexes
CREATE INDEX idx_contacts_user ON contacts(user_id);
CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_contacts_name ON contacts(last_name, first_name);

-- Row Level Security Policies
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;
ALTER TABLE tags ENABLE ROW LEVEL SECURITY;
ALTER TABLE contact_tags ENABLE ROW LEVEL SECURITY;

-- Contacts: Users can only access their own contacts
CREATE POLICY "Users can view own contacts"
    ON contacts FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Users can create own contacts"
    ON contacts FOR INSERT
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own contacts"
    ON contacts FOR UPDATE
    USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own contacts"
    ON contacts FOR DELETE
    USING (auth.uid() = user_id);

-- Tags: Users can only access their own tags
CREATE POLICY "Users can view own tags"
    ON tags FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Users can create own tags"
    ON tags FOR INSERT
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own tags"
    ON tags FOR UPDATE
    USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own tags"
    ON tags FOR DELETE
    USING (auth.uid() = user_id);

-- Contact Tags: Users can only access tags for their own contacts
CREATE POLICY "Users can view own contact tags"
    ON contact_tags FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM contacts
            WHERE contacts.id = contact_tags.contact_id
            AND contacts.user_id = auth.uid()
        )
    );

CREATE POLICY "Users can create own contact tags"
    ON contact_tags FOR INSERT
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM contacts
            WHERE contacts.id = contact_tags.contact_id
            AND contacts.user_id = auth.uid()
        )
    );

CREATE POLICY "Users can delete own contact tags"
    ON contact_tags FOR DELETE
    USING (
        EXISTS (
            SELECT 1 FROM contacts
            WHERE contacts.id = contact_tags.contact_id
            AND contacts.user_id = auth.uid()
        )
    );

-- Function to auto-update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER contacts_updated_at
    BEFORE UPDATE ON contacts
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();
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
