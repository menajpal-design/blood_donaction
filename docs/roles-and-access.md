# User Roles and Access Control

## Role Hierarchy

1. Super Admin
2. District Admin
3. Upazila Admin
4. Union Leader
5. Donor

Higher roles can perform broader actions and manage lower roles.

## Role Scope

- Super Admin:
  - National scope (all districts/upazilas/unions).
- District Admin:
  - Own district only.
- Upazila Admin:
  - Own upazila only (inside own district).
- Union Leader:
  - Own union only (inside own upazila and district).
- Donor:
  - Self profile and self notifications.

## Access Model

The backend enforces three dimensions:

- Authentication:
  - JWT required for protected endpoints.
- Role hierarchy:
  - Minimum role checks for admin endpoints.
- Permission scope:
  - action:resource:scope style permissions with inheritance.

## Key Permission Families

- user:create:* , user:read:* , user:update:*
- donor:read:* , donor:update:* , donor:history:*
- report:read:*
- notification:read:* , notification:create:*

## Security Rules

- Admins cannot manage users of equal or higher role.
- Scope filters restrict all list/read operations.
- Target user access validates both hierarchy and geographic scope.
- Self-registration is restricted to donor role.

## Frontend Role Behavior

- Login redirects by role:
  - donor -> /donors
  - admin roles -> /dashboard
- Route guards enforce role-based page access.
- Navigation menu is filtered by role.
