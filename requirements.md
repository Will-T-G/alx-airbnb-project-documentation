# Airbnb Clone Backend — Requirement Specifications

## 1. User Authentication System

### Overview

The User Authentication module manages registration, login, and secure access control for all users (guests, hosts, and admins).

---

### API Endpoints

| Method   | Endpoint                | Description                          | Access        |
| -------- | ----------------------- | ------------------------------------ | ------------- |
| **POST** | `/api/v1/auth/register` | Registers a new user account         | Public        |
| **POST** | `/api/v1/auth/login`    | Authenticates a user and returns JWT | Public        |
| **GET**  | `/api/v1/users/profile` | Retrieves logged-in user details     | Authenticated |
| **PUT**  | `/api/v1/users/profile` | Updates user profile details         | Authenticated |

---

### Input Specifications

**Registration (`POST /register`)**

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "StrongPassword123!",
  "role": "host"
}
```

**Login (`POST /login`)**

```json
{
  "email": "john@example.com",
  "password": "StrongPassword123!"
}
```

---

### Output Specifications

**Successful Login**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR...",
  "user": {
    "id": 1,
    "email": "john@example.com",
    "role": "host"
  }
}
```

**Failed Login**

```json
{
  "error": "Invalid email or password"
}
```

---

### Validation Rules

* `email`: Must be unique, valid email format.
* `password`: Minimum 8 characters, must include uppercase, lowercase, number, and special character.
* `role`: Must be either `guest` or `host`.
* All required fields must be present.

---

### Performance Criteria

* Authentication requests should complete within **300ms** under normal load.
* Token validation must be stateless and scalable using **JWT**.
* Secure password hashing using **bcrypt** or **Argon2**.

---

## 2. Property Management System

### Overview

This module allows hosts to manage property listings, including creation, updates, deletions, and availability.

---

### API Endpoints

| Method     | Endpoint                 | Description                            | Access            |
| ---------- | ------------------------ | -------------------------------------- | ----------------- |
| **POST**   | `/api/v1/properties`     | Create new property listing            | Host              |
| **GET**    | `/api/v1/properties`     | Retrieve all properties (with filters) | Public            |
| **GET**    | `/api/v1/properties/:id` | Retrieve a single property             | Public            |
| **PUT**    | `/api/v1/properties/:id` | Update property details                | Host (owner only) |
| **DELETE** | `/api/v1/properties/:id` | Delete a property listing              | Host (owner only) |

---

### Input Specifications

**Create Listing (`POST /properties`)**

```json
{
  "title": "Modern Apartment",
  "description": "A spacious apartment with sea view.",
  "price": 120,
  "location": "Monrovia, Liberia",
  "amenities": ["WiFi", "Air Conditioning"],
  "max_guests": 4,
  "availability": {
    "start_date": "2025-10-01",
    "end_date": "2025-12-31"
  }
}
```

---

### Output Specifications

**Successful Creation**

```json
{
  "id": 42,
  "title": "Modern Apartment",
  "status": "listed",
  "host_id": 12
}
```

---

### Validation Rules

* `title`: Required, 5–100 characters.
* `price`: Must be a positive integer or float.
* `availability`: Valid date format, end date must be after start date.
* Only the property **owner (host)** can update or delete.

---

### Performance Criteria

* API should handle up to **1,000 property queries/sec** with caching (Redis recommended).
* Property search queries should complete within **500ms**.
* Image uploads handled asynchronously to file storage (e.g., AWS S3).

---

## 3. Booking System

### Overview

The Booking System allows guests to reserve available properties and ensures there are no double bookings.

---

### API Endpoints

| Method   | Endpoint                      | Description              | Access        |
| -------- | ----------------------------- | ------------------------ | ------------- |
| **POST** | `/api/v1/bookings`            | Create a new booking     | Guest         |
| **GET**  | `/api/v1/bookings`            | Retrieve user’s bookings | Authenticated |
| **PUT**  | `/api/v1/bookings/:id/cancel` | Cancel a booking         | Guest or Host |
| **GET**  | `/api/v1/bookings/:id`        | Retrieve booking details | Authenticated |

---

### Input Specifications

**Create Booking (`POST /bookings`)**

```json
{
  "property_id": 42,
  "check_in": "2025-11-05",
  "check_out": "2025-11-10",
  "guests": 2,
  "payment_method": "stripe"
}
```

---

### Output Specifications

**Successful Booking**

```json
{
  "booking_id": 205,
  "property_id": 42,
  "status": "confirmed",
  "total_price": 600,
  "payment_status": "paid"
}
```

**Error (Double Booking)**

```json
{
  "error": "Property not available for selected dates"
}
```

---

### Validation Rules

* `check_in` < `check_out`.
* Property must be **available** for selected dates.
* Guest count must not exceed property’s `max_guests`.
* Payment method must be valid and authorized.

---

### Performance Criteria

* Booking creation should validate and confirm availability in **<400ms**.
* Database transactions must ensure **atomicity** (no overlapping bookings).
* Use background workers for email confirmations and payment settlement.

