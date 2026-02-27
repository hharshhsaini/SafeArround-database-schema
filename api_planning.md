# SafeAround - Complete API Documentation

## RESTful API & WebSocket Specification

---

## Table of Contents

1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [User Service APIs](#user-service-apis)
4. [Emergency Alert APIs](#emergency-alert-apis)
5. [Geofencing & Location APIs](#geofencing--location-apis)
6. [Crime Data & Heatmap APIs](#crime-data--heatmap-apis)
7. [Notification APIs](#notification-apis)
8. [WebSocket APIs](#websocket-apis)
9. [Rate Limiting](#rate-limiting)
10. [Error Handling](#error-handling)
11. [API Response Formats](#api-response-formats)

---

## API Overview

### Base URLs

```
Production:  https://api.safearound.com/v1
Staging:     https://api-staging.safearound.com/v1
Development: http://localhost:8000/v1

WebSocket:   wss://ws.safearound.com
```

### API Versioning

- Current Version: **v1**
- Version specified in URL path: `/v1/endpoint`
- Breaking changes require new version: `/v2/endpoint`

### Content Types

```
Request:  application/json
Response: application/json
```

### HTTP Methods

- **GET** - Retrieve resources
- **POST** - Create new resources
- **PUT** - Update entire resource
- **PATCH** - Partial update
- **DELETE** - Remove resource

---

## Authentication

### 1. JWT Token-Based Authentication

#### POST /auth/signup
Create new user account

**Request:**
```json
{
  "phone_number": "+1-555-123-4567",
  "email": "user@example.com",
  "full_name": "John Doe",
  "password": "SecurePass123!",
  "device_platform": "ios",
  "fcm_token": "FCM_DEVICE_TOKEN"
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "phone_number": "+1-555-123-4567",
    "email": "user@example.com",
    "full_name": "John Doe",
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 604800,
    "token_type": "Bearer"
  },
  "meta": {
    "timestamp": "2024-02-27T10:30:00Z"
  }
}
```

**Errors:**
- **400** - Invalid input (weak password, invalid phone)
- **409** - User already exists
- **500** - Server error

---

#### POST /auth/login
Authenticate existing user

**Request:**
```json
{
  "phone_number": "+1-555-123-4567",
  "password": "SecurePass123!",
  "device_id": "DEVICE_UUID",
  "fcm_token": "FCM_DEVICE_TOKEN"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 604800,
    "mfa_required": false,
    "user": {
      "full_name": "John Doe",
      "email": "user@example.com",
      "trust_score": 95,
      "location_sharing_enabled": true
    }
  }
}
```

**Errors:**
- **401** - Invalid credentials
- **403** - Account suspended
- **429** - Too many login attempts

---

#### POST /auth/refresh
Refresh access token

**Request:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 604800
  }
}
```

---

#### POST /auth/logout
Invalidate user session

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response: 204 No Content**

---

#### POST /auth/mfa/enable
Enable MFA for account

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "mfa_secret": "JBSWY3DPEHPK3PXP",
    "qr_code_url": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "backup_codes": [
      "ABCD-1234",
      "EFGH-5678",
      "IJKL-9012"
    ]
  }
}
```

---

#### POST /auth/mfa/verify
Verify MFA code

**Request:**
```json
{
  "code": "123456"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "verified": true,
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

## User Service APIs

### GET /users/me
Get current user profile

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "phone_number": "+1-555-123-4567",
    "email": "user@example.com",
    "full_name": "John Doe",
    "location_sharing_enabled": true,
    "alert_radius_preference": 100,
    "dnd_mode_enabled": false,
    "trust_score": 95,
    "account_status": "active",
    "created_at": "2024-01-15T08:30:00Z",
    "last_active_at": "2024-02-27T10:15:00Z",
    "app_version": "1.2.0",
    "device_platform": "ios"
  }
}
```

---

### PUT /users/me
Update user profile

**Request:**
```json
{
  "full_name": "John Michael Doe",
  "email": "john.doe@example.com",
  "location_sharing_enabled": true,
  "alert_radius_preference": 150,
  "dnd_mode_enabled": false
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "full_name": "John Michael Doe",
    "updated_at": "2024-02-27T10:30:00Z"
  }
}
```

---

### POST /users/me/emergency-contacts
Add emergency contact

**Request:**
```json
{
  "contact_name": "Mom",
  "phone_number": "+1-555-987-6543",
  "relationship": "Mother",
  "is_primary": true,
  "notify_on_alert": true,
  "priority_order": 1
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "contact_id": "660e8400-e29b-41d4-a716-446655440001",
    "contact_name": "Mom",
    "phone_number": "+1-555-987-6543",
    "relationship": "Mother",
    "is_primary": true,
    "created_at": "2024-02-27T10:30:00Z"
  }
}
```

---

### GET /users/me/emergency-contacts
List all emergency contacts

**Response: 200 OK**
```json
{
  "success": true,
  "data": [
    {
      "contact_id": "660e8400-e29b-41d4-a716-446655440001",
      "contact_name": "Mom",
      "phone_number": "+1-555-987-6543",
      "relationship": "Mother",
      "is_primary": true,
      "priority_order": 1,
      "created_at": "2024-02-27T10:30:00Z"
    },
    {
      "contact_id": "660e8400-e29b-41d4-a716-446655440002",
      "contact_name": "Dad",
      "phone_number": "+1-555-876-5432",
      "relationship": "Father",
      "is_primary": false,
      "priority_order": 2,
      "created_at": "2024-02-27T10:35:00Z"
    }
  ],
  "meta": {
    "total": 2
  }
}
```

---

### DELETE /users/me/emergency-contacts/{contact_id}
Remove emergency contact

**Response: 204 No Content**

---

### PATCH /users/me/password
Change password

**Request:**
```json
{
  "old_password": "OldSecurePass123!",
  "new_password": "NewSecurePass456!"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "message": "Password updated successfully"
}
```

---

## Emergency Alert APIs

### POST /alerts
Create emergency alert

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request:**
```json
{
  "alert_location": {
    "latitude": 37.7749,
    "longitude": -122.4194
  },
  "alert_type": "emergency",
  "silent_mode": false,
  "metadata": {
    "battery_level": "45%",
    "network_type": "4G",
    "trigger_method": "manual"
  }
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "alert_location": {
      "latitude": 37.7749,
      "longitude": -122.4194
    },
    "alert_status": "active",
    "current_radius": 100,
    "max_radius_reached": 100,
    "users_notified": 8,
    "silent_mode": false,
    "created_at": "2024-02-27T10:30:00Z",
    "timeline": [
      {
        "event_type": "alert_created",
        "radius": 100,
        "users_notified": 8,
        "occurred_at": "2024-02-27T10:30:00Z"
      }
    ]
  }
}
```

---

### GET /alerts/{alert_id}
Get alert details

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "user": {
      "full_name": "John Doe",
      "age": 28,
      "gender": "male"
    },
    "alert_location": {
      "latitude": 37.7749,
      "longitude": -122.4194,
      "address": "123 Main St, San Francisco, CA"
    },
    "alert_status": "responding",
    "current_radius": 250,
    "max_radius_reached": 250,
    "users_notified": 24,
    "responders_count": 2,
    "created_at": "2024-02-27T10:30:00Z",
    "elapsed_seconds": 45,
    "timeline": [
      {
        "event_type": "alert_created",
        "radius": 100,
        "users_notified": 8,
        "occurred_at": "2024-02-27T10:30:00Z"
      },
      {
        "event_type": "radius_expanded",
        "radius": 250,
        "users_notified": 16,
        "occurred_at": "2024-02-27T10:30:30Z"
      },
      {
        "event_type": "responder_accepted",
        "responder_id": "880e8400-e29b-41d4-a716-446655440004",
        "occurred_at": "2024-02-27T10:30:45Z"
      }
    ]
  }
}
```

---

### POST /alerts/{alert_id}/respond
Respond to alert (as helper)

**Request:**
```json
{
  "response_status": "accepted",
  "responder_location": {
    "latitude": 37.7751,
    "longitude": -122.4190
  },
  "estimated_arrival_minutes": 2
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "response_id": "880e8400-e29b-41d4-a716-446655440005",
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "responder_user_id": "550e8400-e29b-41d4-a716-446655440000",
    "response_status": "accepted",
    "distance_meters": 150.5,
    "estimated_arrival_minutes": 2,
    "responded_at": "2024-02-27T10:30:45Z"
  }
}
```

---

### PATCH /alerts/{alert_id}/status
Update alert status

**Request:**
```json
{
  "alert_status": "resolved",
  "resolution_type": "safe",
  "notes": "Situation resolved, user is safe"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "alert_status": "resolved",
    "resolved_at": "2024-02-27T10:35:00Z",
    "total_duration_seconds": 300
  }
}
```

---

### POST /alerts/{alert_id}/escalate
Escalate to emergency services

**Request:**
```json
{
  "escalation_type": "police",
  "reason": "situation_deteriorating",
  "additional_info": "Multiple people involved"
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "escalation_id": "990e8400-e29b-41d4-a716-446655440006",
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "escalation_type": "police",
    "escalation_status": "dispatched",
    "external_reference_id": "SFPD-2024-0227-1030",
    "escalated_at": "2024-02-27T10:32:00Z",
    "estimated_arrival_minutes": 5
  }
}
```

---

### GET /alerts/active
Get active alerts in area

**Query Parameters:**
```
latitude: 37.7749
longitude: -122.4194
radius: 1000 (meters)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": [
    {
      "alert_id": "770e8400-e29b-41d4-a716-446655440003",
      "distance_meters": 150.5,
      "alert_status": "active",
      "current_radius": 250,
      "responders_count": 2,
      "created_at": "2024-02-27T10:30:00Z",
      "elapsed_seconds": 120
    }
  ],
  "meta": {
    "total": 1,
    "search_radius": 1000
  }
}
```

---

## Geofencing & Location APIs

### POST /locations
Update user location

**Request:**
```json
{
  "latitude": 37.7749,
  "longitude": -122.4194,
  "accuracy": 10.5,
  "altitude": 15.0,
  "speed": 0.0,
  "heading": 0.0,
  "battery_level": "75%",
  "network_type": "4G",
  "location_source": "gps"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "location_id": "12345678",
    "recorded_at": "2024-02-27T10:30:00Z",
    "danger_zone_detected": true,
    "zone_info": {
      "zone_id": "aa0e8400-e29b-41d4-a716-446655440007",
      "risk_level": "medium",
      "crime_count": 12,
      "alert_sent": true
    }
  }
}
```

---

### GET /locations/check
Check danger zone at location

**Query Parameters:**
```
latitude: 37.7749
longitude: -122.4194
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "in_danger_zone": true,
    "zone": {
      "zone_id": "aa0e8400-e29b-41d4-a716-446655440007",
      "risk_level": "medium",
      "crime_count": 12,
      "density_score": 45.7,
      "recent_incidents": [
        {
          "incident_type": "theft",
          "severity": 2,
          "occurred_at": "2024-02-26T15:30:00Z",
          "distance_meters": 50.3
        }
      ]
    },
    "safety_tips": [
      "Stay in well-lit areas",
      "Be aware of surroundings",
      "Keep phone accessible"
    ]
  }
}
```

---

### GET /locations/nearby-users
Get nearby active users

**Query Parameters:**
```
latitude: 37.7749
longitude: -122.4194
radius: 500 (meters)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "total_users": 24,
    "active_users": 18,
    "recent_alerts": 1,
    "average_response_time_seconds": 45
  }
}
```

---

### POST /routes/safe
Calculate safe route

**Request:**
```json
{
  "origin": {
    "latitude": 37.7749,
    "longitude": -122.4194
  },
  "destination": {
    "latitude": 37.7849,
    "longitude": -122.4094
  },
  "mode": "walking"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "routes": [
      {
        "route_id": "route_1",
        "name": "Safest Route",
        "safety_score": 95,
        "distance_meters": 1200,
        "duration_minutes": 15,
        "risk_level": "low",
        "danger_zones_count": 0,
        "polyline": "encoded_polyline_string",
        "color": "#4CAF50"
      },
      {
        "route_id": "route_2",
        "name": "Balanced Route",
        "safety_score": 75,
        "distance_meters": 950,
        "duration_minutes": 12,
        "risk_level": "medium",
        "danger_zones_count": 1,
        "polyline": "encoded_polyline_string",
        "color": "#FFC107"
      },
      {
        "route_id": "route_3",
        "name": "Fastest Route",
        "safety_score": 50,
        "distance_meters": 800,
        "duration_minutes": 10,
        "risk_level": "high",
        "danger_zones_count": 3,
        "polyline": "encoded_polyline_string",
        "color": "#FF5722"
      }
    ]
  }
}
```

---

## Crime Data & Heatmap APIs

### GET /heatmap/tiles/{z}/{x}/{y}
Get heatmap tile

**Path Parameters:**
```
z: Zoom level (8-18)
x: Tile X coordinate
y: Tile Y coordinate
```

**Response: 200 OK**
```
Content-Type: image/png
Cache-Control: public, max-age=86400

[PNG image binary data]
```

---

### GET /crimes/recent
Get recent crime incidents

**Query Parameters:**
```
latitude: 37.7749
longitude: -122.4194
radius: 1000 (meters)
limit: 50
since: 2024-02-20T00:00:00Z
severity: [1,2,3,4] (optional)
types: ["theft","assault"] (optional)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": [
    {
      "incident_id": "bb0e8400-e29b-41d4-a716-446655440008",
      "incident_type": "theft",
      "severity": 2,
      "location": {
        "latitude": 37.7750,
        "longitude": -122.4195,
        "address": "456 Oak St, San Francisco, CA"
      },
      "description": "Vehicle break-in reported",
      "occurred_at": "2024-02-26T15:30:00Z",
      "reported_at": "2024-02-26T16:00:00Z",
      "source": "SFPD",
      "verified": true,
      "distance_meters": 50.3
    }
  ],
  "meta": {
    "total": 42,
    "page": 1,
    "per_page": 50,
    "search_radius": 1000
  }
}
```

---

### GET /crimes/statistics
Get crime statistics for area

**Query Parameters:**
```
latitude: 37.7749
longitude: -122.4194
radius: 1000 (meters)
period: 30 (days)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "total_incidents": 156,
    "by_type": {
      "theft": 68,
      "assault": 24,
      "vandalism": 32,
      "robbery": 18,
      "other": 14
    },
    "by_severity": {
      "1": 45,
      "2": 62,
      "3": 34,
      "4": 15
    },
    "trend": "decreasing",
    "change_percentage": -12.5,
    "safest_hours": ["06:00-09:00", "14:00-16:00"],
    "riskiest_hours": ["22:00-02:00"],
    "comparison_to_city": {
      "city_average": 189,
      "percentage_difference": -17.5,
      "safer_than_city": true
    }
  }
}
```

---

### GET /zones/danger
Get danger zones in area

**Query Parameters:**
```
bounds: {
  "north": 37.7849,
  "south": 37.7649,
  "east": -122.4094,
  "west": -122.4294
}
risk_levels: ["high","critical"] (optional)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": [
    {
      "zone_id": "aa0e8400-e29b-41d4-a716-446655440007",
      "risk_level": "high",
      "crime_count": 45,
      "density_score": 78.5,
      "severity_sum": 112,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[...coordinates...]]]
      },
      "statistics": {
        "most_common_type": "theft",
        "peak_hours": ["22:00-02:00"],
        "last_incident": "2024-02-27T08:30:00Z"
      },
      "calculated_at": "2024-02-27T00:00:00Z"
    }
  ],
  "meta": {
    "total": 12
  }
}
```

---

## Notification APIs

### POST /notifications/send
Send notification to user

**Request:**
```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "notification_type": "geofence_alert",
  "title": "Danger Zone Alert",
  "body": "You have entered a high-risk area",
  "priority": "high",
  "data": {
    "zone_id": "aa0e8400-e29b-41d4-a716-446655440007",
    "risk_level": "high",
    "action": "show_zone_details"
  }
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "notification_id": "cc0e8400-e29b-41d4-a716-446655440009",
    "status": "sent",
    "sent_at": "2024-02-27T10:30:00Z",
    "delivery_channel": "fcm"
  }
}
```

---

### GET /notifications/history
Get notification history

**Query Parameters:**
```
limit: 50
offset: 0
type: "geofence_alert" (optional)
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": [
    {
      "notification_id": "cc0e8400-e29b-41d4-a716-446655440009",
      "notification_type": "geofence_alert",
      "title": "Danger Zone Alert",
      "body": "You have entered a high-risk area",
      "status": "delivered",
      "sent_at": "2024-02-27T10:30:00Z",
      "delivered_at": "2024-02-27T10:30:01Z",
      "read_at": "2024-02-27T10:30:15Z"
    }
  ],
  "meta": {
    "total": 245,
    "page": 1,
    "per_page": 50
  }
}
```

---

## WebSocket APIs

### Connection

**URL:** `wss://ws.safearound.com`

**Connection Parameters:**
```javascript
const ws = new WebSocket('wss://ws.safearound.com', {
  headers: {
    'Authorization': 'Bearer {access_token}'
  }
});
```

---

### Events: Client → Server

#### 1. Join Room
```json
{
  "event": "join_room",
  "data": {
    "room_id": "alert_770e8400-e29b-41d4-a716-446655440003",
    "user_role": "responder"
  }
}
```

#### 2. Location Update
```json
{
  "event": "location_update",
  "data": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "accuracy": 10.5,
    "timestamp": "2024-02-27T10:30:00Z"
  }
}
```

#### 3. Chat Message
```json
{
  "event": "chat_message",
  "data": {
    "room_id": "alert_770e8400-e29b-41d4-a716-446655440003",
    "message": "I'm 2 minutes away",
    "message_type": "text"
  }
}
```

#### 4. Status Update
```json
{
  "event": "status_update",
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "status": "arrived"
  }
}
```

---

### Events: Server → Client

#### 1. Connection Established
```json
{
  "event": "connected",
  "data": {
    "connection_id": "conn_123456",
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "server_time": "2024-02-27T10:30:00Z"
  }
}
```

#### 2. Emergency Alert
```json
{
  "event": "emergency_alert",
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "user": {
      "full_name": "Sarah K.",
      "age": 28
    },
    "location": {
      "latitude": 37.7749,
      "longitude": -122.4194
    },
    "distance_meters": 150.5,
    "created_at": "2024-02-27T10:30:00Z"
  }
}
```

#### 3. Location Broadcast
```json
{
  "event": "location_broadcast",
  "data": {
    "user_id": "880e8400-e29b-41d4-a716-446655440004",
    "user_role": "responder",
    "location": {
      "latitude": 37.7751,
      "longitude": -122.4190
    },
    "distance_to_alert": 120.3,
    "estimated_arrival_minutes": 2,
    "timestamp": "2024-02-27T10:30:30Z"
  }
}
```

#### 4. Chat Message
```json
{
  "event": "chat_message",
  "data": {
    "message_id": "msg_123456",
    "sender_id": "880e8400-e29b-41d4-a716-446655440004",
    "sender_name": "John D.",
    "message": "I'm almost there",
    "timestamp": "2024-02-27T10:31:00Z"
  }
}
```

#### 5. Alert Status Change
```json
{
  "event": "alert_status_changed",
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "old_status": "active",
    "new_status": "responding",
    "responders_count": 2,
    "timestamp": "2024-02-27T10:30:45Z"
  }
}
```

#### 6. Radius Expansion
```json
{
  "event": "radius_expanded",
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "old_radius": 100,
    "new_radius": 250,
    "users_notified": 16,
    "elapsed_seconds": 30
  }
}
```

---

## Rate Limiting

### Rate Limit Tiers

| Tier | Requests/Minute | Requests/Day | Burst |
|------|-----------------|--------------|-------|
| Free | 100 | 10,000 | 20 |
| Pro | 1,000 | 100,000 | 200 |
| Enterprise | 10,000 | Unlimited | 2,000 |

### Rate Limit Headers

**Response Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 945
X-RateLimit-Reset: 1709034000
Retry-After: 45
```

### Rate Limit Response

**429 Too Many Requests:**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 45 seconds.",
    "retry_after": 45,
    "limit": 1000,
    "remaining": 0,
    "reset_at": "2024-02-27T11:00:00Z"
  }
}
```

---

## Error Handling

### Standard Error Response

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    },
    "timestamp": "2024-02-27T10:30:00Z",
    "request_id": "req_123456789"
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| INVALID_REQUEST | 400 | Malformed request body |
| VALIDATION_ERROR | 400 | Input validation failed |
| UNAUTHORIZED | 401 | Invalid or missing token |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| SERVER_ERROR | 500 | Internal server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |

### Validation Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input validation failed",
    "validation_errors": [
      {
        "field": "phone_number",
        "message": "Invalid phone number format",
        "code": "INVALID_FORMAT"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters",
        "code": "TOO_SHORT"
      }
    ]
  }
}
```

---

## API Response Formats

### Success Response Structure

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2024-02-27T10:30:00Z",
    "version": "1.0",
    "request_id": "req_123456789"
  }
}
```

### Paginated Response Structure

```json
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "total": 500,
    "page": 1,
    "per_page": 50,
    "total_pages": 10,
    "has_next": true,
    "has_previous": false
  },
  "links": {
    "first": "/api/v1/resource?page=1",
    "last": "/api/v1/resource?page=10",
    "next": "/api/v1/resource?page=2",
    "prev": null
  }
}
```

---

## Security

### HTTPS Only
All API endpoints require HTTPS. HTTP requests are automatically redirected to HTTPS.

### CORS Policy
```
Access-Control-Allow-Origin: https://safearound.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400
```

### API Key (Optional)
For server-to-server communication:
```
X-API-Key: your_api_key_here
```

---

## Webhooks

### Webhook Events

SafeAround can send webhook notifications for the following events:

| Event | Description |
|-------|-------------|
| alert.created | New emergency alert created |
| alert.responded | Someone responded to alert |
| alert.resolved | Alert marked as resolved |
| alert.escalated | Alert escalated to 911 |
| user.entered_danger_zone | User entered high-risk area |

### Webhook Payload

```json
{
  "event": "alert.created",
  "timestamp": "2024-02-27T10:30:00Z",
  "data": {
    "alert_id": "770e8400-e29b-41d4-a716-446655440003",
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "alert_location": {
      "latitude": 37.7749,
      "longitude": -122.4194
    },
    "alert_status": "active"
  },
  "signature": "sha256_signature_here"
}
```

---

## Complete API Endpoints Summary

### Authentication (7 endpoints)
- POST /auth/signup
- POST /auth/login
- POST /auth/refresh
- POST /auth/logout
- POST /auth/mfa/enable
- POST /auth/mfa/verify
- POST /auth/password/reset

### Users (6 endpoints)
- GET /users/me
- PUT /users/me
- PATCH /users/me/password
- GET /users/me/emergency-contacts
- POST /users/me/emergency-contacts
- DELETE /users/me/emergency-contacts/{id}

### Emergency Alerts (7 endpoints)
- POST /alerts
- GET /alerts/{id}
- PATCH /alerts/{id}/status
- POST /alerts/{id}/respond
- POST /alerts/{id}/escalate
- GET /alerts/active
- GET /alerts/history

### Locations & Geofencing (4 endpoints)
- POST /locations
- GET /locations/check
- GET /locations/nearby-users
- POST /routes/safe

### Crime Data & Heatmap (4 endpoints)
- GET /heatmap/tiles/{z}/{x}/{y}
- GET /crimes/recent
- GET /crimes/statistics
- GET /zones/danger

### Notifications (2 endpoints)
- POST /notifications/send
- GET /notifications/history

### WebSocket (1 connection + 8 event types)
- WS Connection: wss://ws.safearound.com
- Client Events: join_room, location_update, chat_message, status_update
- Server Events: connected, emergency_alert, location_broadcast, chat_message, alert_status_changed, radius_expanded

**Total REST Endpoints:** 30  
**WebSocket Events:** 8  
**Total API Surface:** 38 endpoints

---

## SDK Examples

### JavaScript/TypeScript
```javascript
import SafeAroundSDK from '@safearound/sdk';

const client = new SafeAroundSDK({
  apiKey: 'your_api_key',
  environment: 'production'
});

// Create alert
const alert = await client.alerts.create({
  location: { latitude: 37.7749, longitude: -122.4194 },
  silentMode: false
});

// Connect WebSocket
const ws = client.websocket.connect();
ws.on('emergency_alert', (data) => {
  console.log('New alert:', data);
});
```

### Python
```python
from safearound import SafeAroundClient

client = SafeAroundClient(api_key='your_api_key')

# Create alert
alert = client.alerts.create(
    location={'latitude': 37.7749, 'longitude': -122.4194},
    silent_mode=False
)

# Get recent crimes
crimes = client.crimes.recent(
    latitude=37.7749,
    longitude=-122.4194,
    radius=1000
)
```

---

**API Documentation Complete!**  
Total: 30 REST endpoints, 8 WebSocket events, comprehensive authentication, rate limiting, and error handling.
