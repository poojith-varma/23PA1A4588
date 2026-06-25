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



# Stage 3

The original SQL statement is incomplete because the `SELECT` clause does not specify which columns should be returned. A corrected version is shown below:

```sql
SELECT *
FROM notifications
WHERE student_id = 1042
  AND is_read = false
ORDER BY created_at DESC;
```

This query may perform poorly when the `notifications` table contains nearly 5 million records. Without an appropriate index, the database engine must scan a significant portion of the table before sorting the matching rows, which increases execution time.

A good optimization is to create a composite index covering the columns used for filtering and ordering.

```sql
CREATE INDEX idx_notifications
ON notifications (student_id, is_read, created_at DESC);
```

With this index in place, the database can locate the required records more efficiently while minimizing the amount of data that needs to be scanned and sorted.

Creating indexes on every column is generally not recommended. Although indexes improve read performance, they also require additional storage and increase the cost of `INSERT`, `UPDATE`, and `DELETE` operations. Therefore, indexes should only be created for columns that are commonly involved in search conditions, joins, or sorting.

The following query retrieves the unique IDs of students who have received placement-related notifications within the past seven days:

```sql
SELECT DISTINCT student_id
FROM notifications
WHERE type = 'Placement'
  AND created_at >= NOW() - INTERVAL '7 days';
```



# Stage 4

Getting notifications from the database every time a page loads is not efficient. It increases the number of database requests and makes the page slower.

To make the system faster, I would:

- **Use Redis Cache:** Save frequently used notifications and unread counts in Redis so the database is not queried every time.
- **Use Pagination:** Load only a small number of notifications instead of loading all of them at once.
- **Use WebSockets:** Send new notifications to users instantly instead of checking the server again and again.
- **Add Database Indexes:** Create indexes on columns that are searched often to make queries faster.

## Trade-offs

- **Redis Cache:** Makes the application faster, but keeping the cache updated can be difficult.
- **Pagination:** Reduces database load, but users need to load more pages to see older notifications.
- **WebSockets:** Give real-time updates, but they need a permanent connection between the client and server.
- **Indexes:** Improve read speed, but they make insert, update, and delete operations a little slower.