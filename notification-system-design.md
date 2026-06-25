# Notification System Design
# Stage 1

## Base URL

```
/api
```

## Headers

```
Content-Type: application/json
Authorization: Bearer <token>
```

---

## 1. Get Notifications

**GET** `/notifications`

### Response

```json
[
  {
    "id": 1,
    "title": "Placement Drive",
    "message": "XYZ drive starts tomorrow",
    "type": "placement",
    "isRead": false
  }
]
```

---

## 2. Create Notification

**POST** `/notifications`

### Request

```json
{
  "title": "Placement Drive",
  "message": "XYZ drive starts tomorrow",
  "type": "placement"
}
```

### Response

```json
{
  "message": "Notification created"
}
```

---

## 3. Mark Notification as Read

**PUT** `/notifications/{id}/read`

### Response

```json
{
  "message": "Notification marked as read"
}
```

---

## 4. Delete Notification

**DELETE** `/notifications/{id}`

### Response

```json
{
  "message": "Notification deleted"
}
```

---

# Real-Time Notifications

Use **WebSocket**.

When a new notification is created, the server sends it instantly to all connected users.

Example:

```json
{
  "event": "new_notification",
  "title": "Placement Drive",
  "message": "xyz drive starts tomorrow"
}
```

---

## Status Codes

- 200 OK
- 201 Created
- 400 Bad Request
- 401 Unauthorized
- 404 Not Found
- 500 Internal Server Error





# Stage 2

## Recommended Database

For storing notification data, **PostgreSQL** is a my preferabble option. It offers reliable transactions (ACID support), fast querying through indexes, and scales well for structured data.

## Database Design

### Students Table

```sql
CREATE TABLE students (
  id BIGINT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
```

### Notifications Table

```sql
CREATE TYPE notification_type AS ENUM ('Event', 'Result', 'Placement');

CREATE TABLE notifications (
  id UUID PRIMARY KEY,
  student_id BIGINT REFERENCES students(id),
  type notification_type,
  message TEXT,
  is_read BOOLEAN DEFAULT false,
  created_at TIMESTAMP
);
```

## Challenges with Large Datasets

* Query execution may become slower as the notification count increases.
* Higher storage usage and greater database workload over time.

## Optimization Techniques

* Create indexes on commonly filtered fields such as `student_id`, `is_read`, and `created_at`.
* Use **Redis** to cache unread notification counts and frequently requested information.

## Example SQL Queries

### Retrieve All Notifications

```sql
SELECT *
FROM notifications
WHERE student_id = 1042
ORDER BY created_at DESC;
```

### Retrieve Only Unread Notifications

```sql
SELECT *
FROM notifications
WHERE student_id = 1042
  AND is_read = false
ORDER BY created_at DESC;
```

### Update Notification Status

```sql
UPDATE notifications
SET is_read = true
WHERE id = 'notification_id';
```
