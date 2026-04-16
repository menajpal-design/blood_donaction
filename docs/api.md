# API Documentation

Base URL: /api/v1

## Authentication

### POST /auth/register

Register a donor account and return JWT + profile.

Request body:

```json
{
  "name": "Nusrat Jahan",
  "email": "nusrat@example.com",
  "password": "StrongPass123",
  "bloodGroup": "O-",
  "districtId": "6658e7...",
  "upazilaId": "6658e8...",
  "unionId": "6658e9...",
  "phone": "+88017...",
  "location": "Dhanmondi, Dhaka"
}
```

### POST /auth/login

Authenticate user and return JWT + profile.

Request body:

```json
{
  "email": "nusrat@example.com",
  "password": "StrongPass123"
}
```

### GET /auth/me

Get current authenticated user profile.

Auth: Bearer token required.

## User Management

### POST /users

Create user by admin based on hierarchy scope.

Auth: Union Leader and above.

### GET /users

Get users in caller scope.

Auth: Union Leader and above.

### GET /users/:userId

Get user details by ID in caller scope.

Auth: Union Leader and above.

## Locations

Public location lookup endpoints for the Bangladesh hierarchy.

### GET /locations/divisions

Get all divisions.

### GET /locations/divisions/:divisionId/districts

Get districts by division ID.

### GET /locations/districts/:districtId/upazilas

Get upazilas by district ID.

### GET /locations/upazilas/:upazilaId/unions

Get unions / pouroshava by upazila ID.

Response shape:

```json
{
  "success": true,
  "data": [
    {
      "id": "...",
      "name": "Dhaka",
      "bnName": "ঢাকা",
      "code": "DIV-0001",
      "externalId": 1
    }
  ]
}
```

## Donor Profiles

### GET /donor-profiles/me

Get current donor profile.

### PUT /donor-profiles/me

Create/update donor profile.

Request body:

```json
{
  "bloodGroup": "A+",
  "lastDonationDate": "2026-04-01",
  "availabilityStatus": "available"
}
```

### POST /donor-profiles/me/history

Add donation history record.

Request body:

```json
{
  "donationDate": "2026-04-08",
  "location": "Dhaka Medical",
  "notes": "Emergency request"
}
```

### GET /donor-profiles/me/history

Get current donor history list.

### GET /donor-profiles/search

Advanced donor search.

Query params:

- bloodGroup
- districtId
- upazilaId
- unionId
- availabilityStatus
- page
- limit

### GET /donor-profiles/user/:userId

Get donor profile by user ID for admin in-scope.

## Reports

### GET /reports/monthly/donors

Monthly donor activity report.

Query params:

- year: number
- month: number (1-12)
- format: json | csv

Response includes:

- donorActivity metrics
- donationFrequency metrics
- inactiveDonors list

## Notifications

Notification types:

- donation_request
- donation_approval
- admin_update

### GET /notifications/me

List current user notifications.

Query params:

- page
- limit
- unreadOnly
- type

### PATCH /notifications/me/:notificationId/read

Mark single notification as read.

### PATCH /notifications/me/read-all

Mark all notifications as read.

### POST /notifications

Create notification for scoped target user.

Auth: Union Leader and above.

Request body:

```json
{
  "recipientUserId": "6658ff...",
  "type": "admin_update",
  "title": "Schedule Updated",
  "message": "Blood drive schedule updated for your union.",
  "metadata": {
    "campaignId": "CMP-APR-26"
  }
}
```

### POST /notifications/me/seed-demo

Create demo notifications for current user.

## Health

### GET /health

Service and DB health status.
