# API Documentation - Abbasi Hotel Service Application

## Base URL
```
http://localhost:5000
```

## Authentication
Currently uses JWT tokens in headers (to be implemented):
```
Authorization: Bearer <token>
```

## Response Format
All responses are JSON:
```json
{
  "data": {},
  "error": null,
  "timestamp": "2026-05-15T19:16:53Z"
}
```

## Endpoints

### Health Check
**GET** `/health`

Check server status.

```bash
curl http://localhost:5000/health
```

**Response:**
```json
{
  "status": "OK",
  "timestamp": "2026-05-15T19:16:53Z"
}
```

---

## Service Requests

### Create Service Request
**POST** `/api/requests`

Create a new service request from a guest.

**Request Body:**
```json
{
  "tableId": "T01",
  "guestId": "G001",
  "requestType": "water",
  "customRequest": ""
}
```

**Parameters:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| tableId | string | Yes | Table identifier |
| guestId | string | Yes | Guest identifier |
| requestType | string | Yes | Type: water, napkin, utensil_spoon, utensil_fork, utensil_knife, menu, assistance, custom |
| customRequest | string | No | Custom request text |

**Response (201):**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "tableId": "T01",
  "guestId": "G001",
  "requestType": "water",
  "status": "pending",
  "priority": "normal",
  "createdAt": "2026-05-15T19:16:53Z",
  "acknowledgedAt": null,
  "completedAt": null
}
```

**Example:**
```bash
curl -X POST http://localhost:5000/api/requests \
  -H "Content-Type: application/json" \
  -d '{
    "tableId": "T01",
    "guestId": "G001",
    "requestType": "water"
  }'
```

---

### Get Request Details
**GET** `/api/requests/:id`

Get details of a specific service request.

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| id | string | Request MongoDB ID |

**Response (200):**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "tableId": "T01",
  "guestId": "G001",
  "requestType": "water",
  "status": "in_progress",
  "assignedWaiter": "W001",
  "priority": "normal",
  "createdAt": "2026-05-15T19:16:53Z",
  "acknowledgedAt": "2026-05-15T19:17:00Z",
  "completedAt": null
}
```

**Example:**
```bash
curl http://localhost:5000/api/requests/507f1f77bcf86cd799439011
```

---

### Update Request Status
**PATCH** `/api/requests/:id/status`

Update the status of a service request.

**Request Body:**
```json
{
  "status": "in_progress",
  "assignedWaiter": "W001"
}
```

**Status Values:** pending, acknowledged, in_progress, completed, cancelled

**Response (200):**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "status": "in_progress",
  "assignedWaiter": "W001",
  "acknowledgedAt": "2026-05-15T19:17:00Z"
}
```

**Example:**
```bash
curl -X PATCH http://localhost:5000/api/requests/507f1f77bcf86cd799439011/status \
  -H "Content-Type: application/json" \
  -d '{
    "status": "in_progress",
    "assignedWaiter": "W001"
  }'
```

---

## Feedback

### Submit Feedback
**POST** `/api/feedback`

Submit guest feedback with rating and emoji.

**Request Body:**
```json
{
  "requestId": "507f1f77bcf86cd799439011",
  "guestId": "G001",
  "rating": 5,
  "emoji": "😊",
  "comment": "Excellent service!"
}
```

**Parameters:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| requestId | string | Yes | Service request ID |
| guestId | string | Yes | Guest identifier |
| rating | number | Yes | Rating 1-5 |
| emoji | string | No | Emoji representation |
| comment | string | No | Additional feedback text |

**Response (201):**
```json
{
  "_id": "507f191e810c19729de860ea",
  "requestId": "507f1f77bcf86cd799439011",
  "guestId": "G001",
  "rating": 5,
  "emoji": "😊",
  "comment": "Excellent service!",
  "createdAt": "2026-05-15T19:18:00Z"
}
```

**Emoji Scale:**
- 😞 = 1 star (Poor)
- 😐 = 2 stars (Fair)
- 🙂 = 3 stars (Good)
- 😊 = 4 stars (Very Good)
- 😄 = 5 stars (Excellent)

**Example:**
```bash
curl -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/json" \
  -d '{
    "requestId": "507f1f77bcf86cd799439011",
    "guestId": "G001",
    "rating": 5,
    "emoji": "😊",
    "comment": "Excellent service!"
  }'
```

---

### Get Request Feedback
**GET** `/api/feedback/:requestId`

Get all feedback for a specific request.

**Response (200):**
```json
[
  {
    "_id": "507f191e810c19729de860ea",
    "requestId": "507f1f77bcf86cd799439011",
    "rating": 5,
    "emoji": "😊",
    "comment": "Excellent service!",
    "createdAt": "2026-05-15T19:18:00Z"
  }
]
```

**Example:**
```bash
curl http://localhost:5000/api/feedback/507f1f77bcf86cd799439011
```

---

### Get Feedback Analytics
**GET** `/api/feedback/analytics/summary`

Get overall feedback statistics.

**Response (200):**
```json
{
  "total": 150,
  "averageRating": 4.5,
  "timestamp": "2026-05-15T19:20:00Z"
}
```

**Example:**
```bash
curl http://localhost:5000/api/feedback/analytics/summary
```

---

## Waiter Management

### Register Waiter Device
**POST** `/api/waiters/register`

Register a new waiter device.

**Request Body:**
```json
{
  "deviceId": "DEVICE-UUID-123",
  "userId": "W001",
  "deviceIP": "192.168.1.100",
  "deviceName": "Waiter iPad 1"
}
```

**Parameters:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| deviceId | string | Yes | Unique device identifier |
| userId | string | Yes | Waiter user ID |
| deviceIP | string | Yes | Device IP address |
| deviceName | string | Yes | Human-readable device name |

**Response (201):**
```json
{
  "_id": "507f1f77bcf86cd799439012",
  "deviceId": "DEVICE-UUID-123",
  "userId": "W001",
  "deviceIP": "192.168.1.100",
  "deviceName": "Waiter iPad 1",
  "isOnline": true,
  "registeredAt": "2026-05-15T19:16:53Z"
}
```

**Example:**
```bash
curl -X POST http://localhost:5000/api/waiters/register \
  -H "Content-Type: application/json" \
  -d '{
    "deviceId": "DEVICE-UUID-123",
    "userId": "W001",
    "deviceIP": "192.168.1.100",
    "deviceName": "Waiter iPad 1"
  }'
```

---

### Get Online Waiters
**GET** `/api/waiters/status`

Get all currently online waiter devices.

**Response (200):**
```json
[
  {
    "_id": "507f1f77bcf86cd799439012",
    "deviceId": "DEVICE-UUID-123",
    "userId": "W001",
    "deviceIP": "192.168.1.100",
    "deviceName": "Waiter iPad 1",
    "isOnline": true,
    "lastSeenAt": "2026-05-15T19:20:00Z"
  },
  {
    "_id": "507f1f77bcf86cd799439013",
    "deviceId": "DEVICE-UUID-456",
    "userId": "W002",
    "deviceIP": "192.168.1.101",
    "deviceName": "Waiter iPad 2",
    "isOnline": true,
    "lastSeenAt": "2026-05-15T19:19:30Z"
  }
]
```

**Example:**
```bash
curl http://localhost:5000/api/waiters/status
```

---

## WebSocket Events

### Connection
```javascript
const socket = io('http://localhost:5000', {
  transports: ['websocket', 'polling']
});

socket.on('connect', () => {
  console.log('Connected to server');
});
```

### New Request Notification
```javascript
socket.on('newRequest', (request) => {
  console.log('New request:', request);
  // {
  //   _id: '...',
  //   tableId: 'T01',
  //   requestType: 'water',
  //   status: 'pending',
  //   createdAt: '2026-05-15T19:16:53Z'
  // }
});
```

### Request Status Update
```javascript
socket.on('requestStatusUpdated', (request) => {
  console.log('Request status changed:', request);
  // {
  //   _id: '...',
  //   status: 'in_progress',
  //   assignedWaiter: 'W001'
  // }
});
```

### Feedback Received
```javascript
socket.on('feedbackReceived', (feedback) => {
  console.log('New feedback:', feedback);
  // {
  //   _id: '...',
  //   requestId: '...',
  //   rating: 5,
  //   emoji: '😊'
  // }
});
```

### Waiter Status Change
```javascript
socket.on('waiterStatusChanged', (data) => {
  console.log('Waiter status:', data);
  // {
  //   deviceId: '...',
  //   status: 'online' or 'offline'
  // }
});
```

### Emit Waiter Online
```javascript
socket.emit('waiterOnline', {
  deviceId: 'DEVICE-UUID-123',
  userId: 'W001'
});
```

### Emit Waiter Offline
```javascript
socket.emit('waiterOffline', {
  deviceId: 'DEVICE-UUID-123',
  userId: 'W001'
});
```

---

## Error Responses

### 404 Not Found
```json
{
  "error": "Request not found"
}
```

### 400 Bad Request
```json
{
  "error": "Invalid request parameters"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal Server Error",
  "timestamp": "2026-05-15T19:16:53Z"
}
```

### 429 Too Many Requests
```json
{
  "error": "Too many requests from this IP, please try again later."
}
```

---

## Rate Limiting

- **Window**: 15 minutes
- **Limit**: 100 requests per IP
- **Header**: `X-RateLimit-Remaining`

---

## Testing

### Using Postman

1. Import collection from `docs/postman-collection.json`
2. Set environment variables
3. Run tests

### Using cURL Batch Script

```bash
#!/bin/bash

# Test health
echo "Testing health endpoint..."
curl -s http://localhost:5000/health | jq '.'

# Create request
echo "Creating request..."
REQUEST=$(curl -s -X POST http://localhost:5000/api/requests \
  -H "Content-Type: application/json" \
  -d '{
    "tableId": "T01",
    "guestId": "G001",
    "requestType": "water"
  }' | jq -r '._id')

echo "Request ID: $REQUEST"

# Get request
echo "Fetching request..."
curl -s http://localhost:5000/api/requests/$REQUEST | jq '.'

# Update status
echo "Updating status..."
curl -s -X PATCH http://localhost:5000/api/requests/$REQUEST/status \
  -H "Content-Type: application/json" \
  -d '{"status": "in_progress", "assignedWaiter": "W001"}' | jq '.'

# Submit feedback
echo "Submitting feedback..."
curl -s -X POST http://localhost:5000/api/feedback \
  -H "Content-Type: application/json" \
  -d "{
    \"requestId\": \"$REQUEST\",
    \"guestId\": \"G001\",
    \"rating\": 5,
    \"emoji\": \"😊\"
  }" | jq '.'

# Get analytics
echo "Fetching analytics..."
curl -s http://localhost:5000/api/feedback/analytics/summary | jq '.'
```

---

## Version
API Version: 1.0.0
Last Updated: 2026-05-15
