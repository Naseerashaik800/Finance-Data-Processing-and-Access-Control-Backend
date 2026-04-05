# Finance Backend - API Documentation

Complete API reference guide for the Finance Data Processing and Access Control Backend.

## Base URL

```
http://localhost:3000/api
```

## Authentication

All endpoints except `/auth/register` and `/auth/login` require JWT authentication via Bearer token:

```
Authorization: Bearer <JWT_TOKEN>
```

## Response Format

All responses follow a consistent JSON format:

```json
{
  "success": boolean,
  "message": string (optional),
  "data": any (optional),
  "error": string (optional)
}
```

## Endpoints Overview

### Authentication (`/auth`)
- `POST /register` - Create new user account
- `POST /login` - Authenticate and get JWT token

### Users (`/users`)
- `GET /profile` - Get current user profile
- `GET /` - Get all users (admin only)
- `GET /:userId` - Get specific user
- `PATCH /:userId/role` - Update user role (admin only)
- `PATCH /:userId/status` - Update user status (admin only)
- `DELETE /:userId` - Delete user (admin only)

### Records (`/records`)
- `POST /` - Create new financial record
- `GET /` - Get user's records with filtering
- `GET /search/:term` - Search records
- `GET /:recordId` - Get specific record
- `PATCH /:recordId` - Update record
- `DELETE /:recordId` - Delete record (soft delete)

### Dashboard (`/dashboard`)
- `GET /summary` - Get dashboard summary
- `GET /insights` - Get analytics insights (analyst/admin only)

---

## Detailed Endpoint Reference

### Auth Endpoints

#### POST /auth/register

Create a new user account.

**Request Body:**
```json
{
  "username": "string (3+ chars, unique)",
  "email": "string (valid email)",
  "password": "string (6+ chars)",
  "role": "viewer | analyst | admin (optional, defaults to viewer)"
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "string",
    "status": "active",
    "created_at": "ISO timestamp"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input (username too short, invalid email, weak password)
- `409 Conflict` - Username or email already exists

**Example:**
```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "securePass123"
  }'
```

---

#### POST /auth/login

Authenticate user and receive JWT token.

**Request Body:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "username": "string",
      "email": "string",
      "role": "string"
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Missing credentials
- `401 Unauthorized` - Invalid credentials or inactive user

**Token Usage:** Include token in Authorization header for subsequent requests:
```bash
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

### User Endpoints

#### GET /users/profile

Get the current authenticated user's profile.

**Authorization:** Required (any role)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "viewer | analyst | admin",
    "status": "active | inactive",
    "created_at": "ISO timestamp"
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Missing or invalid token
- `404 Not Found` - User not found

---

#### GET /users/?page=1&limit=10

Get paginated list of all users.

**Authorization:** Required (admin only)

**Query Parameters:**
- `page` (integer) - Page number, default: 1
- `limit` (integer, 1-100) - Records per page, default: 10

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "uuid",
        "username": "string",
        "email": "string",
        "role": "string",
        "status": "string",
        "created_at": "ISO timestamp"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 25,
      "pages": 3
    }
  }
}
```

---

#### GET /users/:userId

Get specific user's profile.

**Authorization:** Required
- Admin: Can view any user
- Other roles: Can only view own profile

**URL Parameters:**
- `userId` (string) - UUID of the user

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "username": "string",
    "email": "string",
    "role": "string",
    "status": "string",
    "created_at": "ISO timestamp"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - User not found

---

#### PATCH /users/:userId/role

Update a user's role.

**Authorization:** Required (admin only)

**Request Body:**
```json
{
  "role": "viewer | analyst | admin"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "User role updated successfully",
  "data": { ... }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid role
- `403 Forbidden` - Not admin
- `404 Not Found` - User not found

---

#### PATCH /users/:userId/status

Update a user's status (active/inactive).

**Authorization:** Required (admin only)

**Request Body:**
```json
{
  "status": "active | inactive"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "User status updated successfully",
  "data": { ... }
}
```

---

#### DELETE /users/:userId

Delete a user (hard delete).

**Authorization:** Required (admin only)

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "User deleted successfully"
}
```

---

### Record Endpoints

#### POST /records

Create a new financial record.

**Authorization:** Required (create:records permission)
- Analyst, Admin only (not Viewer)

**Request Body:**
```json
{
  "type": "income | expense",
  "amount": number (positive),
  "category": "string",
  "description": "string (optional)",
  "date": "YYYY-MM-DD"
}
```

**Success Response (201 Created):**
```json
{
  "success": true,
  "message": "Record created successfully",
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "type": "income | expense",
    "amount": number,
    "category": "string",
    "description": "string | null",
    "date": "YYYY-MM-DD",
    "is_deleted": false,
    "created_at": "ISO timestamp",
    "updated_at": "ISO timestamp"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input
- `403 Forbidden` - Insufficient permissions (viewer role)

**Example:**
```bash
curl -X POST http://localhost:3000/api/records \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "income",
    "amount": 5000,
    "category": "Salary",
    "description": "Monthly salary",
    "date": "2024-04-01"
  }'
```

---

#### GET /records/?page=1&limit=10&type=income&category=Salary&startDate=2024-01-01&endDate=2024-12-31

Get user's financial records with optional filtering.

**Authorization:** Required (view:records permission - all roles)

**Query Parameters:**
- `page` (integer) - Page number, default: 1
- `limit` (integer, 1-100) - Records per page, default: 10
- `type` (string) - Filter by "income" or "expense" (optional)
- `category` (string) - Filter by category (case-insensitive, optional)
- `startDate` (string, YYYY-MM-DD) - Filter records from this date (optional)
- `endDate` (string, YYYY-MM-DD) - Filter records until this date (optional)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "records": [
      {
        "id": "uuid",
        "user_id": "uuid",
        "type": "income | expense",
        "amount": number,
        "category": "string",
        "description": "string | null",
        "date": "YYYY-MM-DD",
        "is_deleted": false,
        "created_at": "ISO timestamp",
        "updated_at": "ISO timestamp"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 45,
      "pages": 5
    }
  }
}
```

**Example:**
```bash
curl -X GET "http://localhost:3000/api/records?page=1&limit=10&type=income" \
  -H "Authorization: Bearer TOKEN"
```

---

#### GET /records/search/:term?page=1&limit=10

Search records by category or description.

**Authorization:** Required (Analyst/Admin only)

**URL Parameters:**
- `term` (string) - Search term for category or description

**Query Parameters:**
- `page` (integer) - Page number, default: 1
- `limit` (integer, 1-100) - Records per page, default: 10

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "records": [...],
    "pagination": { ... }
  }
}
```

**Example:**
```bash
curl -X GET "http://localhost:3000/api/records/search/Salary?page=1&limit=10" \
  -H "Authorization: Bearer TOKEN"
```

---

#### GET /records/:recordId

Get a specific financial record.

**Authorization:** Required (view:records permission)

**URL Parameters:**
- `recordId` (string) - UUID of the record

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "user_id": "uuid",
    "type": "income | expense",
    "amount": number,
    "category": "string",
    "description": "string | null",
    "date": "YYYY-MM-DD",
    "is_deleted": false,
    "created_at": "ISO timestamp",
    "updated_at": "ISO timestamp"
  }
}
```

**Error Responses:**
- `404 Not Found` - Record not found or belongs to different user

---

#### PATCH /records/:recordId

Update a financial record.

**Authorization:** Required (update:records permission - Analyst/Admin only)

**URL Parameters:**
- `recordId` (string) - UUID of the record

**Request Body:** (all fields optional, at least one required)
```json
{
  "type": "income | expense",
  "amount": number (positive),
  "category": "string",
  "description": "string",
  "date": "YYYY-MM-DD"
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Record updated successfully",
  "data": { ... }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input or no fields provided
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Record not found

---

#### DELETE /records/:recordId

Delete a financial record (soft delete - marks as deleted).

**Authorization:** Required (delete:records permission - Admin only)

**URL Parameters:**
- `recordId` (string) - UUID of the record

**Success Response (200 OK):**
```json
{
  "success": true,
  "message": "Record deleted successfully"
}
```

**Error Responses:**
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Record not found

**Note:** Uses soft delete - record is marked as deleted but not removed from database.

---

### Dashboard Endpoints

#### GET /dashboard/summary

Get dashboard summary with financial overview.

**Authorization:** Required (view:dashboard permission - all roles)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_income": number,
    "total_expense": number,
    "net_balance": number,
    "record_count": integer,
    "category_breakdown": [
      {
        "category": "string",
        "amount": number,
        "type": "income | expense"
      }
    ],
    "recent_records": [
      {
        "id": "uuid",
        "type": "income | expense",
        "amount": number,
        "category": "string",
        "date": "YYYY-MM-DD"
      }
    ],
    "monthly_trend": [
      {
        "month": "YYYY-MM",
        "income": number,
        "expense": number
      }
    ]
  }
}
```

**Example:**
```bash
curl -X GET http://localhost:3000/api/dashboard/summary \
  -H "Authorization: Bearer TOKEN"
```

---

#### GET /dashboard/insights

Get detailed analytics and insights.

**Authorization:** Required (Analyst/Admin only)

**Success Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "category_insights": [
      {
        "type": "income | expense",
        "category": "string",
        "count": integer,
        "avg_amount": number,
        "max_amount": number,
        "min_amount": number
      }
    ],
    "top_categories": [
      {
        "category": "string",
        "type": "income | expense",
        "total_amount": number,
        "count": integer
      }
    ],
    "spending_patterns": [
      {
        "day_of_week": integer (0-6),
        "month": integer (1-12),
        "avg_amount": number,
        "count": integer
      }
    ]
  }
}
```

---

## Error Handling

### Standard Error Response

```json
{
  "success": false,
  "error": "Error description"
}
```

### HTTP Status Codes

| Code | Meaning | Common Cause |
|------|---------|-------------|
| 200 | OK | Successful GET, PATCH, DELETE |
| 201 | Created | Successful POST |
| 400 | Bad Request | Invalid input, validation error |
| 401 | Unauthorized | Missing/invalid token, inactive user |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate username/email |
| 500 | Server Error | Internal server error |

---

## Rate Limiting

Currently no rate limiting is implemented. Future versions may include:
- IP-based rate limiting
- Per-user rate limiting
- Endpoint-specific limits

---

## Pagination

For endpoints that return lists, pagination is applied:

**Query Parameters:**
- `page` - Page number (1-indexed), default: 1
- `limit` - Records per page (1-100), default: 10

**Response Includes:**
```json
{
  "pagination": {
    "page": integer,
    "limit": integer,
    "total": integer,
    "pages": integer
  }
}
```

---

## Common Scenarios

### Scenario 1: User Registration and Login
```bash
# 1. Register
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newuser",
    "email": "newuser@example.com",
    "password": "password123"
  }'

# 2. Login
RESPONSE=$(curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "newuser",
    "password": "password123"
  }')

# 3. Extract token and use in subsequent requests
TOKEN=$(echo $RESPONSE | jq -r '.data.token')
```

### Scenario 2: Create and View Records
```bash
# 1. Create record
curl -X POST http://localhost:3000/api/records \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "income",
    "amount": 1000,
    "category": "Freelance",
    "date": "2024-04-05"
  }'

# 2. View all records
curl -X GET "http://localhost:3000/api/records?page=1&limit=10" \
  -H "Authorization: Bearer $TOKEN"

# 3. Filter by category
curl -X GET "http://localhost:3000/api/records?category=Freelance" \
  -H "Authorization: Bearer $TOKEN"
```

### Scenario 3: Admin User Management
```bash
# 1. Get all users (admin only)
curl -X GET "http://localhost:3000/api/users?page=1&limit=10" \
  -H "Authorization: Bearer $ADMIN_TOKEN"

# 2. Update user role
curl -X PATCH http://localhost:3000/api/users/{userId}/role \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "role": "analyst"
  }'

# 3. Deactivate user
curl -X PATCH http://localhost:3000/api/users/{userId}/status \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "inactive"
  }'
```

---

## Notes

- All timestamps are in ISO 8601 format (UTC)
- Monetary amounts are stored as decimals with 2 decimal places
- Dates are in YYYY-MM-DD format
- All string fields are case-sensitive except where noted (category is case-insensitive)
- Deleted records (soft delete) are excluded from listing but can be restored by database queries
