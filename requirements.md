# 🏡 Airbnb Clone Backend - Feature Requirements

This document outlines the **technical and functional requirements** for the core backend features in the Airbnb Clone project.

---

## 🔐 Feature 1: User Authentication

### ✅ Functional Requirements
- Users can register as **Guest** or **Host**.
- Users can log in using **email/password** or **OAuth**.
- JWT token is returned on successful login.
- Support password reset and profile update.

### 🔧 API Endpoints

| Method | Endpoint                | Description                     |
|--------|-------------------------|---------------------------------|
| POST   | /api/auth/register/     | Register a new user             |
| POST   | /api/auth/login/        | Log in user                     |
| POST   | /api/auth/logout/       | Log out user (blacklist token)  |
| POST   | /api/auth/reset/        | Send password reset email       |
| PUT    | /api/auth/profile/      | Update user profile             |

### 📝 Input/Output Examples

**POST /api/auth/register/**  
Input (JSON):
```json
{
  "email": "guest@mail.com",
  "password": "StrongPass123",
  "role": "guest"
}
```

Output (JSON):
```json
{
  "token": "<JWT_TOKEN>",
  "user": {
    "id": 1,
    "email": "guest@mail.com",
    "role": "guest"
  }
}
```

### 🔐 Validation Rules
- Email must be unique.
- Password must be at least 8 characters, contain uppercase, lowercase, number.
- Role must be either `guest` or `host`.

### 🚀 Performance Criteria
- Auth endpoints should respond in under 300ms.
- JWT tokens expire after 1 hour, support refresh token rotation.

---

## 🏠 Feature 2: Property Management

### ✅ Functional Requirements
- Hosts can **create**, **edit**, **delete**, and **view** listings.
- Listings must include title, description, location, amenities, price, and availability.

### 🔧 API Endpoints

| Method | Endpoint                        | Description               |
|--------|----------------------------------|---------------------------|
| POST   | /api/properties/                | Create a property         |
| GET    | /api/properties/                | List all properties       |
| GET    | /api/properties/<id>/           | View specific property    |
| PUT    | /api/properties/<id>/           | Update a property         |
| DELETE | /api/properties/<id>/           | Delete a property         |

### 📝 Input/Output Examples

**POST /api/properties/**  
Input (JSON):
```json
{
  "title": "Beach House",
  "description": "A lovely place near the beach.",
  "location": "Lagos, Nigeria",
  "price_per_night": 200,
  "amenities": ["Wi-Fi", "Air Conditioning"],
  "availability": {
    "start_date": "2025-06-01",
    "end_date": "2025-06-30"
  }
}
```

Output (JSON):
```json
{
  "id": 23,
  "title": "Beach House",
  "host": 4
}
```

### 🔐 Validation Rules
- `price_per_night` must be a positive number.
- `availability` dates must be future dates.
- Only authenticated users with role `host` can post.

### 🚀 Performance Criteria
- Support pagination for property listing.
- Use indexing for `location`, `price` fields to optimize filtering/search.

---

## 📅 Feature 3: Booking System

### ✅ Functional Requirements
- Guests can book available properties.
- Prevent overlapping/double bookings.
- Allow cancellation of bookings.

### 🔧 API Endpoints

| Method | Endpoint                           | Description               |
|--------|-------------------------------------|---------------------------|
| POST   | /api/bookings/                     | Create a new booking      |
| GET    | /api/bookings/                     | View all bookings         |
| DELETE | /api/bookings/<id>/cancel/        | Cancel a booking          |
| PATCH  | /api/bookings/<id>/status/        | Update booking status     |

### 📝 Input/Output Examples

**POST /api/bookings/**  
Input (JSON):
```json
{
  "property_id": 23,
  "check_in": "2025-06-10",
  "check_out": "2025-06-15"
}
```

Output (JSON):
```json
{
  "id": 101,
  "guest_id": 1,
  "status": "pending"
}
```

### 🔐 Validation Rules
- `check_out` must be after `check_in`.
- Property must be available for selected dates.
- Only authenticated users with role `guest` can book.

### 🚀 Performance Criteria
- Booking creation must use atomic transactions to prevent double booking.
- Implement optimistic locking or date range validation on the backend.

---

## 💳 Feature 4: Payments

### ✅ Functional Requirements
- Users can pay for bookings through integrated payment gateway.
- System supports transaction success/failure handling.
- Webhook from payment provider confirms and updates status.

### 🔧 API Endpoints

| Method | Endpoint                          | Description                        |
|--------|------------------------------------|------------------------------------|
| POST   | /api/payments/                    | Initiate payment for a booking     |
| POST   | /api/payments/webhook/           | Handle Stripe webhook callback     |

### 📝 Input/Output Examples

**POST /api/payments/**  
Input (JSON):
```json
{
  "booking_id": 101,
  "payment_method": "card"
}
```

Output (JSON):
```json
{
  "payment_url": "https://checkout.stripe.com/..."
}
```

### 🔐 Validation Rules
- `booking_id` must be valid and unpaid.
- Must redirect to payment gateway like Stripe.
- Validate webhook signature to avoid fraud.

### 🚀 Performance Criteria
- Payment processing should complete within 5 seconds.
- Support automatic retry on webhook failures.

---

## 📬 Feature 5: Messaging (Optional / MVP+)

### ✅ Functional Requirements
- Guests and Hosts can message each other about listings.
- Messages are stored with timestamps and sender IDs.

### 🔧 API Endpoints

| Method | Endpoint                          | Description                        |
|--------|------------------------------------|------------------------------------|
| GET    | /api/messages/<user_id>/          | Fetch chat messages with a user    |
| POST   | /api/messages/                    | Send a message                     |

### 📝 Input/Output Examples

**POST /api/messages/**  
Input (JSON):
```json
{
  "receiver_id": 4,
  "text": "Hello, is the beach house available on June 12?"
}
```

Output (JSON):
```json
{
  "message_id": 200,
  "timestamp": "2025-06-01T13:45:00Z"
}
```

---

## 🛠️ Feature 6: Admin Management

### ✅ Functional Requirements
- Admin can deactivate users, remove listings, or review disputes.
- Access only allowed to staff/admin role.

### 🔧 API Endpoints

| Method | Endpoint                      | Description                        |
|--------|-------------------------------|------------------------------------|
| GET    | /api/admin/users/            | View all users                     |
| DELETE | /api/admin/users/<id>/       | Deactivate user                    |
| DELETE | /api/admin/properties/<id>/  | Remove a listing                   |

---

## 🔒 Security Considerations

- JWT authentication for all users.
- Admin-only routes use role-based access.
- Rate limiting on sensitive routes (login, register, payments).
- Passwords are hashed using `PBKDF2` or `bcrypt`.

---

## ⚙️ Technologies Used

- Django + Django REST Framework
- PostgreSQL or MySQL
- Redis (optional) for caching and session
- Stripe API for payments
- SendGrid or Mailgun for email
- Docker for containerization

---

## 📌 Notes

- All endpoints require authentication via JWT (except register/login).
- Use DRF Serializers for input validation.
- Add unit tests and integration tests for all endpoints.
- Use async where needed (e.g., sending emails).