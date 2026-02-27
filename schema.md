# SafeAround - Database Schema Diagram (ERD)
## Complete Entity Relationship Diagram with Visual Representations

## 1. COMPLETE ERD DIAGRAM (Mermaid Format)
```mermaid
erDiagram
    users ||--o{ user_locations : tracks
    users ||--o{ emergency_contacts : has
    users ||--o{ emergency_alerts : creates
    users ||--o{ alert_responses : responds
    users ||--o{ user_reports : submits
    users ||--o{ notification_logs : receives
    users ||--o{ user_sessions : has

    emergency_alerts ||--o{ alert_responses : receives
    emergency_alerts ||--o{ alert_escalations : escalates
    emergency_alerts ||--o{ alert_timeline_events : has
    
    crime_incidents }o--|| crime_sources : from
    crime_incidents }o--|| crime_categories : categorized
    
    danger_zones ||--o{ zone_crime_history : contains
    danger_zones }o--|| zone_risk_levels : has
    
    users {
        UUID user_id PK
        VARCHAR phone_number UK
        VARCHAR email UK
        VARCHAR full_name
        BOOLEAN location_sharing_enabled
        INTEGER alert_radius_preference
        BOOLEAN dnd_mode_enabled
        VARCHAR firebase_uid UK
        VARCHAR account_status
        INTEGER trust_score
        TIMESTAMPTZ created_at
        TIMESTAMPTZ last_active_at
        VARCHAR app_version
        VARCHAR device_platform
    }

    user_locations {
        BIGSERIAL location_id PK
        UUID user_id FK
        GEOGRAPHY location
        FLOAT accuracy
        FLOAT altitude
        FLOAT speed
        FLOAT heading
        VARCHAR battery_level
        VARCHAR network_type
        TIMESTAMPTZ recorded_at
        VARCHAR location_source
    }

    emergency_contacts {
        UUID contact_id PK
        UUID user_id FK
        VARCHAR contact_name
        VARCHAR phone_number
        VARCHAR relationship
        BOOLEAN is_primary
        BOOLEAN notify_on_alert
        INTEGER priority_order
        TIMESTAMPTZ created_at
    }

    emergency_alerts {
        UUID alert_id PK
        UUID user_id FK
        GEOGRAPHY alert_location
        VARCHAR alert_type
        VARCHAR alert_status
        INTEGER current_radius
        INTEGER max_radius_reached
        JSONB metadata
        VARCHAR audio_recording_url
        BOOLEAN silent_mode
        TIMESTAMPTZ created_at
        TIMESTAMPTZ resolved_at
        TIMESTAMPTZ cancelled_at
        VARCHAR cancellation_reason
    }

    alert_responses {
        UUID response_id PK
        UUID alert_id FK
        UUID responder_user_id FK
        VARCHAR response_status
        GEOGRAPHY responder_location
        INTEGER estimated_arrival_minutes
        FLOAT distance_meters
        TIMESTAMPTZ responded_at
        TIMESTAMPTZ arrived_at
        INTEGER response_rating
        TEXT response_feedback
    }

    alert_escalations {
        UUID escalation_id PK
        UUID alert_id FK
        VARCHAR escalation_type
        VARCHAR escalation_status
        JSONB escalation_payload
        VARCHAR external_reference_id
        TIMESTAMPTZ escalated_at
        TIMESTAMPTZ acknowledged_at
    }

    alert_timeline_events {
        UUID event_id PK
        UUID alert_id FK
        VARCHAR event_type
        INTEGER radius_at_event
        INTEGER users_notified
        INTEGER responders_count
        JSONB event_data
        TIMESTAMPTZ occurred_at
    }

    crime_incidents {
        UUID incident_id PK
        UUID source_id FK
        UUID category_id FK
        GEOGRAPHY location
        VARCHAR incident_type
        INTEGER severity
        TEXT description
        VARCHAR address
        JSONB metadata
        BOOLEAN verified
        TIMESTAMPTZ occurred_at
        TIMESTAMPTZ reported_at
        TIMESTAMPTZ scraped_at
    }

    crime_sources {
        UUID source_id PK
        VARCHAR source_name
        VARCHAR source_type
        VARCHAR api_endpoint
        JSONB auth_config
        INTEGER reliability_score
        BOOLEAN is_active
        TIMESTAMPTZ last_successful_fetch
    }

    crime_categories {
        UUID category_id PK
        VARCHAR category_name
        VARCHAR category_code
        INTEGER default_severity
        VARCHAR icon_name
        VARCHAR color_hex
    }

    danger_zones {
        UUID zone_id PK
        GEOGRAPHY boundary
        VARCHAR risk_level
        INTEGER crime_count
        FLOAT density_score
        INTEGER severity_sum
        JSONB statistics
        TIMESTAMPTZ calculated_at
        TIMESTAMPTZ valid_until
    }

    zone_risk_levels {
        VARCHAR risk_level PK
        INTEGER min_score
        INTEGER max_score
        VARCHAR color_hex
        VARCHAR display_label
    }

    zone_crime_history {
        UUID history_id PK
        UUID zone_id FK
        DATE date
        INTEGER incident_count
        INTEGER severity_total
        JSONB crime_type_breakdown
    }

    user_reports {
        UUID report_id PK
        UUID user_id FK
        GEOGRAPHY report_location
        VARCHAR report_type
        TEXT description
        VARCHAR media_url
        VARCHAR status
        INTEGER upvote_count
        INTEGER downvote_count
        BOOLEAN verified
        TIMESTAMPTZ reported_at
        TIMESTAMPTZ verified_at
    }

    notification_logs {
        UUID notification_id PK
        UUID user_id FK
        VARCHAR notification_type
        VARCHAR delivery_channel
        JSONB payload
        VARCHAR status
        VARCHAR device_token
        TIMESTAMPTZ sent_at
        TIMESTAMPTZ delivered_at
        TIMESTAMPTZ read_at
    }

    user_sessions {
        UUID session_id PK
        UUID user_id FK
        VARCHAR device_id
        VARCHAR fcm_token
        VARCHAR apns_token
        JSONB device_info
        BOOLEAN is_active
        TIMESTAMPTZ created_at
        TIMESTAMPTZ expires_at
        TIMESTAMPTZ last_seen_at
    }
```

## DETAILED TABLE STRUCTURES WITH RELATIONSHIPS
 Core User Management Tables
```mermaid
graph TB
    subgraph User Management
        Users[Users Table<br/>- user_id PK<br/>- phone_number UNIQUE<br/>- email UNIQUE<br/>- trust_score<br/>- location_sharing_enabled]
        
        UserLocations[User Locations<br/>- location_id PK<br/>- user_id FK<br/>- location GEOGRAPHY<br/>- accuracy<br/>- recorded_at]
        
        EmergencyContacts[Emergency Contacts<br/>- contact_id PK<br/>- user_id FK<br/>- contact_name<br/>- phone_number<br/>- is_primary]
        
        Sessions[User Sessions<br/>- session_id PK<br/>- user_id FK<br/>- fcm_token<br/>- device_id<br/>- is_active]
    end
    
    Users -->|1:N| UserLocations
    Users -->|1:N| EmergencyContacts
    Users -->|1:N| Sessions
    
    style Users fill:#E3F2FD,stroke:#1976D2,stroke-width:3px
    style UserLocations fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style EmergencyContacts fill:#FFF9C4,stroke:#F57C00,stroke-width:2px
    style Sessions fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px
```

 Emergency Alert System Tables

```mermaid
graph TB
    subgraph Emergency System
        Alerts[Emergency Alerts<br/>- alert_id PK<br/>- user_id FK<br/>- alert_location GEOGRAPHY<br/>- alert_status<br/>- current_radius<br/>- silent_mode]
        
        Responses[Alert Responses<br/>- response_id PK<br/>- alert_id FK<br/>- responder_user_id FK<br/>- response_status<br/>- distance_meters<br/>- arrived_at]
        
        Escalations[Alert Escalations<br/>- escalation_id PK<br/>- alert_id FK<br/>- escalation_type<br/>- external_reference_id<br/>- escalated_at]
        
        Timeline[Alert Timeline Events<br/>- event_id PK<br/>- alert_id FK<br/>- event_type<br/>- radius_at_event<br/>- users_notified<br/>- occurred_at]
    end
    
    Alerts -->|1:N| Responses
    Alerts -->|1:N| Escalations
    Alerts -->|1:N| Timeline
    
    style Alerts fill:#FFEBEE,stroke:#D32F2F,stroke-width:3px
    style Responses fill:#E8F5E9,stroke:#43A047,stroke-width:2px
    style Escalations fill:#FFF3E0,stroke:#FB8C00,stroke-width:2px
    style Timeline fill:#E1F5FE,stroke:#039BE5,stroke-width:2px
```
Crime Data & Heatmap Tables
```mermaid
graph TB
    subgraph Crime Intelligence
        Incidents[Crime Incidents<br/>- incident_id PK<br/>- source_id FK<br/>- location GEOGRAPHY<br/>- incident_type<br/>- severity<br/>- verified]
        
        Sources[Crime Sources<br/>- source_id PK<br/>- source_name<br/>- source_type<br/>- reliability_score<br/>- is_active]
        
        Categories[Crime Categories<br/>- category_id PK<br/>- category_name<br/>- default_severity<br/>- color_hex]
        
        DangerZones[Danger Zones<br/>- zone_id PK<br/>- boundary GEOGRAPHY<br/>- risk_level<br/>- crime_count<br/>- density_score]
        
        ZoneHistory[Zone Crime History<br/>- history_id PK<br/>- zone_id FK<br/>- date<br/>- incident_count<br/>- severity_total]
    end
    
    Incidents -->|N:1| Sources
    Incidents -->|N:1| Categories
    DangerZones -->|1:N| ZoneHistory
    
    style Incidents fill:#FFF9C4,stroke:#FBC02D,stroke-width:3px
    style Sources fill:#E0F2F1,stroke:#00897B,stroke-width:2px
    style Categories fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px
    style DangerZones fill:#FFECB3,stroke:#FFA000,stroke-width:3px
    style ZoneHistory fill:#E8EAF6,stroke:#5C6BC0,stroke-width:2px
```

## 3. DATABASE SCHEMA OVERVIEW WITH STATISTICS

```mermaid
graph LR
    subgraph Core Tables
        T1[Users<br/>8 indexes<br/>12 columns]
        T2[User Locations<br/>5 indexes<br/>11 columns]
        T3[Emergency Alerts<br/>6 indexes<br/>14 columns]
    end
    
    subgraph Crime Data
        T4[Crime Incidents<br/>7 indexes<br/>13 columns]
        T5[Danger Zones<br/>4 indexes<br/>9 columns]
    end
    
    subgraph Support Tables
        T6[Emergency Contacts<br/>3 indexes<br/>9 columns]
        T7[Notification Logs<br/>4 indexes<br/>10 columns]
        T8[Alert Responses<br/>5 indexes<br/>11 columns]
    end
    
    T1 -.->|Foreign Key| T2
    T1 -.->|Foreign Key| T3
    T1 -.->|Foreign Key| T6
    T3 -.->|Foreign Key| T8
    T4 -.->|Spatial| T5
    
    style T1 fill:#E3F2FD,stroke:#1976D2,stroke-width:3px
    style T2 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style T3 fill:#FFEBEE,stroke:#D32F2F,stroke-width:3px
    style T4 fill:#FFF9C4,stroke:#FBC02D,stroke-width:2px
    style T5 fill:#FFECB3,stroke:#FFA000,stroke-width:2px
    style T6 fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px
    style T7 fill:#E0F2F1,stroke:#00897B,stroke-width:2px
    style T8 fill:#E8F5E9,stroke:#43A047,stroke-width:2px
```
## 4. KEY RELATIONSHIPS & CARDINALITY
```mermaid
graph TD
    U[Users] -->|1 to Many| UL[User Locations]
    U -->|1 to Many| EC[Emergency Contacts]
    U -->|1 to Many| EA[Emergency Alerts]
    U -->|1 to Many| AR[Alert Responses as Responder]
    U -->|1 to Many| UR[User Reports]
    U -->|1 to Many| NL[Notification Logs]
    
    EA -->|1 to Many| AR2[Alert Responses]
    EA -->|1 to Many| AE[Alert Escalations]
    EA -->|1 to Many| ATE[Alert Timeline Events]
    
    CI[Crime Incidents] -->|Many to 1| CS[Crime Sources]
    CI -->|Many to 1| CC[Crime Categories]
    
    DZ[Danger Zones] -->|1 to Many| ZH[Zone Crime History]
    
    style U fill:#1976D2,stroke:#0D47A1,stroke-width:4px,color:#FFF
    style EA fill:#D32F2F,stroke:#B71C1C,stroke-width:4px,color:#FFF
    style CI fill:#F57C00,stroke:#E65100,stroke-width:3px,color:#FFF
    style DZ fill:#FFA000,stroke:#FF6F00,stroke-width:3px,color:#FFF
```
## 5. SPATIAL INDEXES & GEOSPATIAL TABLES
```mermaid
graph TB
    subgraph PostGIS Spatial Tables
        UL[user_locations<br/>GIST location]
        EA[emergency_alerts<br/>GIST alert_location]
        CI[crime_incidents<br/>GIST location]
        DZ[danger_zones<br/>GIST boundary]
        UR[user_reports<br/>GIST report_location]
    end
    
    subgraph Spatial Operations
        STContains[ST_Contains<br/>Point in Polygon]
        STDistance[ST_Distance<br/>Proximity Search]
        STDWithin[ST_DWithin<br/>Radius Query]
        STBuffer[ST_Buffer<br/>Create Zones]
    end
    
    UL -.->|Query| STDWithin
    EA -.->|Query| STDWithin
    CI -.->|Query| STContains
    DZ -.->|Query| STContains
    
    style UL fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style EA fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
    style CI fill:#FFF9C4,stroke:#F57C00,stroke-width:2px
    style DZ fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style STContains fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style STDistance fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style STDWithin fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style STBuffer fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
```

## 6. TABLE SIZES & GROWTH ESTIMATES
```mermaid
graph LR
    subgraph High Volume Tables
        T1[user_locations<br/>~10M rows/month<br/>Partitioned by month]
        T2[notification_logs<br/>~5M rows/month<br/>Partitioned by week]
        T3[alert_timeline_events<br/>~500K rows/month<br/>Partitioned by month]
    end
    
    subgraph Medium Volume
        T4[emergency_alerts<br/>~100K rows/month<br/>Standard]
        T5[crime_incidents<br/>~50K rows/month<br/>Standard]
    end
    
    subgraph Low Volume
        T6[users<br/>~10K rows/month<br/>Standard]
        T7[danger_zones<br/>~1K rows/month<br/>Standard]
    end
    
    style T1 fill:#FFEBEE,stroke:#C62828,stroke-width:3px
    style T2 fill:#FFEBEE,stroke:#C62828,stroke-width:3px
    style T3 fill:#FFEBEE,stroke:#C62828,stroke-width:3px
    style T4 fill:#FFF9C4,stroke:#F57C00,stroke-width:2px
    style T5 fill:#FFF9C4,stroke:#F57C00,stroke-width:2px
    style T6 fill:#E8F5E9,stroke:#43A047,stroke-width:2px
    style T7 fill:#E8F5E9,stroke:#43A047,stroke-width:2px
```
# 7. INDEXING STRATEGY
## Primary Indexes (BTREE)
users:
  - PRIMARY KEY (user_id)
  - UNIQUE INDEX (phone_number)
  - UNIQUE INDEX (email)
  - INDEX (account_status) WHERE status = 'active'
  - INDEX (last_active_at) WHERE last_active > NOW() - 7 days

emergency_alerts:
  - PRIMARY KEY (alert_id)
  - INDEX (user_id, created_at DESC)
  - INDEX (alert_status, created_at DESC)
  - INDEX (created_at) WHERE alert_status IN ('active', 'responding')

crime_incidents:
  - PRIMARY KEY (incident_id)
  - INDEX (occurred_at DESC)
  - INDEX (incident_type, severity)
  - INDEX (verified) WHERE verified = true

## Spatial Indexes (GIST)
user_locations:
  - GIST INDEX (location)
  - Supports: ST_DWithin, ST_Distance queries

emergency_alerts:
  - GIST INDEX (alert_location)
  - Supports: Radius expansion queries

crime_incidents:
  - GIST INDEX (location)
  - Supports: Heatmap aggregation

danger_zones:
  - GIST INDEX (boundary)
  - Supports: ST_Contains queries
Composite Indexes

## alert_responses:
  - INDEX (alert_id, response_status, responded_at)
  - INDEX (responder_user_id, responded_at DESC)

notification_logs:
  - INDEX (user_id, sent_at DESC)
  - INDEX (notification_type, status, sent_at)

## 8. TABLE PARTITIONING STRATEGY

```mermaid
graph TB
    subgraph Partitioned Tables
        UL[user_locations<br/>Partition by: recorded_at<br/>Strategy: Monthly<br/>Retention: 90 days]
        
        NL[notification_logs<br/>Partition by: sent_at<br/>Strategy: Weekly<br/>Retention: 30 days]
        
        ATE[alert_timeline_events<br/>Partition by: occurred_at<br/>Strategy: Monthly<br/>Retention: 365 days]
    end
    
    subgraph Partition Management
        AutoDrop[Auto-drop old partitions<br/>pg_cron scheduled job]
        AutoCreate[Auto-create future partitions<br/>7 days ahead]
    end
    
    UL --> AutoDrop
    NL --> AutoDrop
    ATE --> AutoDrop
    
    UL --> AutoCreate
    NL --> AutoCreate
    ATE --> AutoCreate
    
    style UL fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style NL fill:#FFF3E0,stroke:#F57C00,stroke-width:2px
    style ATE fill:#E8F5E9,stroke:#43A047,stroke-width:2px
    style AutoDrop fill:#FFEBEE,stroke:#D32F2F,stroke-width:2px
    style AutoCreate fill:#E8F5E9,stroke:#388E3C,stroke-width:2px
```
## 9. MATERIALIZED VIEWS FOR PERFORMANCE
```sql
-- Danger Zones Materialized View
CREATE MATERIALIZED VIEW mv_danger_zones AS
SELECT
    ST_SnapToGrid(location::geometry, 0.01) as grid_cell,
    COUNT(*) as incident_count,
    SUM(severity) as severity_total,
    AVG(severity) as avg_severity,
    MAX(occurred_at) as last_incident
FROM crime_incidents
WHERE occurred_at > NOW() - INTERVAL '30 days'
GROUP BY grid_cell;

CREATE INDEX ON mv_danger_zones USING GIST(grid_cell);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_danger_zones;

-- User Activity Summary
CREATE MATERIALIZED VIEW mv_user_activity AS
SELECT
    user_id,
    COUNT(DISTINCT DATE(recorded_at)) as active_days,
    MAX(recorded_at) as last_location_update,
    COUNT(*) as total_locations
FROM user_locations
WHERE recorded_at > NOW() - INTERVAL '30 days'
GROUP BY user_id;

CREATE INDEX ON mv_user_activity(user_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_user_activity;
```
## 10. DATABASE CONSTRAINTS & BUSINESS RULES
```mermaid
graph TB
    subgraph Data Integrity Rules
        C1[Users<br/>trust_score: 0-100<br/>account_status: enum]
        C2[Emergency Alerts<br/>current_radius <= max_radius<br/>status transitions valid]
        C3[Crime Incidents<br/>severity: 1-4<br/>occurred_at <= scraped_at]
        C4[Danger Zones<br/>risk_level: enum<br/>density_score >= 0]
    end
    
    subgraph Cascade Rules
        D1[User deletion<br/>CASCADE: locations, contacts, sessions<br/>RESTRICT: active alerts]
        D2[Alert deletion<br/>CASCADE: responses, escalations, timeline<br/>ARCHIVE: after 90 days]
    end
    
    C1 -.-> D1
    C2 -.-> D2
    
    style C1 fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    style C2 fill:#FFEBEE,stroke:#D32F2F,stroke-width:2px
    style C3 fill:#FFF9C4,stroke:#F57C00,stroke-width:2px
    style C4 fill:#FFE0B2,stroke:#FB8C00,stroke-width:2px
    style D1 fill:#FFCDD2,stroke:#C62828,stroke-width:2px
    style D2 fill:#FFCDD2,stroke:#C62828,stroke-width:2px
```
## 11. COMPLETE TABLE SUMMARY

| Table Name | Primary Key | Foreign Keys | Spatial | Partitioned | Indexes |
|------------|-------------|--------------|---------|-------------|---------|
| users | user_id | - | No | No | 8 |
| user_locations | location_id | user_id | Yes (GEOGRAPHY) | Yes (monthly) | 5 |
| emergency_contacts | contact_id | user_id | No | No | 3 |
| emergency_alerts | alert_id | user_id | Yes (GEOGRAPHY) | No | 6 |
| alert_responses | response_id | alert_id, responder_user_id | Yes (GEOGRAPHY) | No | 5 |
| alert_escalations | escalation_id | alert_id | No | No | 3 |
| alert_timeline_events | event_id | alert_id | No | Yes (monthly) | 4 |
| crime_incidents | incident_id | source_id, category_id | Yes (GEOGRAPHY) | No | 7 |
| crime_sources | source_id | - | No | No | 2 |
| crime_categories | category_id | - | No | No | 2 |
| danger_zones | zone_id | - | Yes (GEOGRAPHY) | No | 4 |
| zone_crime_history | history_id | zone_id | No | No | 3 |
| user_reports | report_id | user_id | Yes (GEOGRAPHY) | No | 4 |
| notification_logs | notification_id | user_id | No | Yes (weekly) | 4 |
| user_sessions | session_id | user_id | No | No | 3 |

**Total Tables:** 15  
**Total Indexes:** 67 (including 5 spatial GIST indexes)  
**Partitioned Tables:** 3  
**Tables with Spatial Data:** 6  

---

## 12. QUERY PERFORMANCE OPTIMIZATION

### Most Frequent Queries:
```sql
-- 1. Find nearby users (Geofencing)
SELECT user_id, ST_Distance(location, $1) as distance
FROM user_locations
WHERE ST_DWithin(location, $1, $2)
AND recorded_at > NOW() - INTERVAL '5 minutes'
ORDER BY distance
LIMIT 50;

-- 2. Get danger zone for location
SELECT zone_id, risk_level, crime_count
FROM danger_zones
WHERE ST_Contains(boundary, $1)
LIMIT 1;

-- 3. Active alerts in area
SELECT alert_id, user_id, alert_status, current_radius
FROM emergency_alerts
WHERE alert_status IN ('active', 'responding')
AND ST_DWithin(alert_location, $1, $2)
ORDER BY created_at DESC;
```