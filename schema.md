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

