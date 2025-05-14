Backend Requirement Specifications
1. User Authentication
Objective
Provide secure user registration, login, and session management for hosts and guests.
Functional Requirements

Allow users to register with their name, email, and password.
Enable users to log in using email and password.
Differentiate between hosts (is_host = true) and guests (is_host = false).

API Endpoints
POST /api/auth/register

Description: Register a new user.
Input:
name (string, required, 2-50 characters)
email (string, required, valid email format)
password (string, required, 8-50 characters, must include at least one uppercase, one lowercase, one number, and one special character)
is_host (boolean, required)


Validation Rules:
name: No special characters except spaces, hyphens, or apostrophes.
email: Must be unique in the User table, follow standard email format (e.g., user@domain.com).
password: Must meet complexity requirements.


Output:
Success: { "user_id": <id>, "message": "User registered successfully" } (HTTP 201)
Error: { "error": "Email already exists" } (HTTP 400)


Performance Criteria:
Response time: < 500ms for 95% of requests.
Password hashing: Use bcrypt with a cost factor of 12.



POST /api/auth/login

Description: Authenticate a user and return a JWT token.
Input:
email (string, required, valid email format)
password (string, required)


Validation Rules:
email: Must exist in the User table.
password: Must match the hashed password in the database.


Output:
Success: { "token": <jwt_token>, "user_id": <id>, "is_host": <boolean> } (HTTP 200)
Error: { "error": "Invalid credentials" } (HTTP 401)


Performance Criteria:
Response time: < 300ms for 95% of requests.
JWT token expiration: 24 hours.




2. Property Management
Objective
Allow hosts to create, update, and delete properties, and allow guests to view available properties.
Functional Requirements

Hosts can add new properties with details like title, location, and price per night.
Hosts can update or delete their own properties.
Guests can view properties with filters for location and price.

API Endpoints
POST /api/properties

Description: Create a new property (host only).
Input:
user_id (integer, required, authenticated host’s ID from JWT)
title (string, required, 5-100 characters)
location (string, required, 2-100 characters)
price_per_night (decimal, required, > 0)


Validation Rules:
user_id: Must belong to a user with is_host = true.
price_per_night: Must be a positive number with up to 2 decimal places.


Output:
Success: { "property_id": <id>, "message": "Property created successfully" } (HTTP 201)
Error: { "error": "Unauthorized: Only hosts can create properties" } (HTTP 403)


Performance Criteria:
Response time: < 400ms for 95% of requests.



GET /api/properties

Description: Retrieve a list of properties with optional filters.
Input (Query Parameters):
location (string, optional)
max_price (decimal, optional)


Validation Rules:
max_price: If provided, must be a positive number.


Output:
Success: [ { "id": <id>, "title": <string>, "location": <string>, "price_per_night": <decimal>, "user_id": <id> }, ... ] (HTTP 200)
Error: { "error": "Invalid max_price" } (HTTP 400)


Performance Criteria:
Response time: < 500ms for 95% of requests with up to 1000 records.



PUT /api/properties/{id}

Description: Update a property (host only).
Input:
title (string, optional, 5-100 characters)
location (string, optional, 2-100 characters)
price_per_night (decimal, optional, > 0)


Validation Rules:
id: Must exist in the Properties table.
user_id: Must match the property’s user_id (only the owner can update).


Output:
Success: { "message": "Property updated successfully" } (HTTP 200)
Error: { "error": "Property not found" } (HTTP 404)


Performance Criteria:
Response time: < 400ms for 95% of requests.




3. Booking System
Objective
Enable guests to book properties and process payments, ensuring availability based on check-in/check-out dates.
Functional Requirements

Guests can book a property for specific dates.
System checks property availability by cross-referencing existing bookings.
System creates a payment record upon successful booking.

API Endpoints
POST /api/bookings

Description: Create a new booking.
Input:
user_id (integer, required, authenticated guest’s ID from JWT)
property_id (integer, required)
check_in (date, required, format: YYYY-MM-DD)
check_out (date, required, format: YYYY-MM-DD)


Validation Rules:
user_id: Must belong to a user with is_host = false.
property_id: Must exist in the Properties table.
check_in: Must be today or a future date.
check_out: Must be after check_in.
Availability: No overlapping bookings for the same property_id (check Bookings table for conflicts).


Output:
Success: { "booking_id": <id>, "message": "Booking created successfully" } (HTTP 201)
Error: { "error": "Property not available for selected dates" } (HTTP 400)


Performance Criteria:
Response time: < 600ms for 95% of requests (includes availability check).



POST /api/payments

Description: Process payment for a booking.
Input:
booking_id (integer, required)
amount (decimal, required, > 0)


Validation Rules:
booking_id: Must exist in the Bookings table and not have an associated payment.
amount: Must match the calculated total based on price_per_night and duration of stay.


Output:
Success: { "payment_id": <id>, "status": "completed", "message": "Payment processed successfully" } (HTTP 201)
Error: { "error": "Invalid booking_id" } (HTTP 400)


Performance Criteria:
Response time: < 500ms for 95% of requests.




General Performance and Security Requirements

All endpoints must use HTTPS.
Rate limiting: Maximum 100 requests per minute per user.
Database queries should be optimized with indexes on frequently queried fields (e.g., Properties.location, Bookings.check_in, Bookings.check_out).
Error logging: Log all errors with timestamps and request details for debugging.

