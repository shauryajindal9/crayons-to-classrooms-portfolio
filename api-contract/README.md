# API Contract (Draft)

This document outlines the *interface* the UI expects from a backend service.

## Roles (Example)
- Admin
- Warehouse Staff
- Volunteer (read-only)

## Inventory

### GET /inventory
Purpose: list inventory items with filtering + pagination

Inputs (query params):
- q (search)
- category
- status
- page, limit

Outputs (response shape):
- items: [{ id, name, category, quantity, status, updated_at }]
- page, limit, total

### POST /inventory
Purpose: create a new item (Admin/Staff)

### PATCH /inventory/{id}
Purpose: update quantity/status/metadata (Admin/Staff)

### GET /inventory/{id}/history
Purpose: item audit trail (receipts/distributions/adjustments)
