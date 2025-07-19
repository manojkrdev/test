
# Asset Management System: Core Service Specifications

**Date:** Saturday, July 19, 2025, 4:20 PM IST

Old Table Schema [Click Here](https://dbdiagram.io/d/required-tables-for-ams-schema-6879e5a2f413ba35087ebca9)

## 1. List of Features

### Core Asset Management Features

| **Feature ID** | **Feature Name** | **Description** |
| :-- | :-- | :-- |
| **AM-001** | Asset Registration \& Cataloging | Create, update, and delete asset records with full metadata: serial numbers, specifications, and categories. |
| **AM-002** | Asset Location Tracking | Real-time tracking of asset physical locations across multiple sites with history log. |
| **AM-003** | Asset Assignment Management | Assign/unassign assets to users, departments, or locations; record dates, actors, and history. |
| **AM-004** | Barcode/QR Code Generation \& Scanning | Generate barcodes/QR codes from `serial_number`; enable mobile scanning for asset lookup. |
| **AM-005** | Asset Status Management | Manage lifecycle states—available, assigned, maintenance, retired, disposed—via `status`. |
| **AM-006** | Asset Category Management | Organize assets hierarchically by type, department, or function via `asset_categories`. |

### Inventory \& Stock Management

| **Feature ID** | **Feature Name** | **Description** |
| :-- | :-- | :-- |
| **IM-001** | Stock Level Monitoring | Track product stock levels and consumption in `asset_products`. |
| **IM-002** | Automated Stock Alerts | Trigger notifications when stock falls below thresholds. |
| **IM-003** | Bulk Asset Operations | Import/export via CSV/Excel with field mapping; log outcomes. |

### Maintenance Management

| **Feature ID** | **Feature Name** | **Description** |
| :-- | :-- | :-- |
| **MT-001** | Maintenance Request Creation from Tickets | Log maintenance issues via tickets; assign and prioritize tasks. |
| **MT-003** | Maintenance History Tracking | Store full history with costs and technician notes in `maintenance_requests`. |

### Financial Tracking

| **Feature ID** | **Feature Name** | **Description** |
| :-- | :-- | :-- |
| **FT-001** | Cost Tracking | Record purchase price, maintenance costs, and total cost of ownership in assets and maintenance tables. |

### Reporting \& Analytics

| **Feature ID** | **Feature Name** | **Description** |
| :-- | :-- | :-- |
| **RP-001** | Asset Utilization Reports | Analyze usage patterns and utilization metrics. |
| **RP-003** | Custom Dashboards | Role-based dashboards with configurable widgets and KPIs. |
| **RP-004** | Audit Trail Reports | Comprehensive logs with user and timestamp details. |
| **RP-005** | Advanced Filtering \& Search | Multi-parameter filters, saved views, and custom searches. |

## 2. Database Schema

### Core Asset Tables

#### `asset_categories`

```sql
CREATE TABLE asset_categories (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID NOT NULL,
    company_type    VARCHAR(255) NOT NULL,
    name            VARCHAR(255) NOT NULL,
    status          INT NOT NULL,          -- 0=inactive,1=active,2=deleted
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


#### `asset_products`

```sql
CREATE TABLE asset_products (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id          UUID NOT NULL,
    company_type        VARCHAR(255) NOT NULL,
    category_id         UUID NOT NULL REFERENCES asset_categories(id),
    name                VARCHAR(255) NOT NULL,
    description         TEXT,
    brand               VARCHAR(255),
    model_number        VARCHAR(255),
    image               TEXT,
    purchase_price      DECIMAL(15,2),
    purchase_date       DATE,
    warranty_expiry     DATE,
    product_status      VARCHAR(20) DEFAULT 'available',  -- available, assigned, retired
    condition_status    VARCHAR(20) DEFAULT 'good',
    status              INT NOT NULL,          -- 0=inactive,1=active,2=deleted
    stock_quantity      INT DEFAULT 0,
    reserved_quantity   INT DEFAULT 0,
    stock_alert_threshold INT DEFAULT 5,
    unit_cost           DECIMAL(15,2),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by          UUID NOT NULL,
    updated_by          UUID NOT NULL
);
```


#### `locations`

```sql
CREATE TABLE locations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID NOT NULL,
    company_type    VARCHAR(255) NOT NULL,
    name            VARCHAR(255) NOT NULL,
    location_code   VARCHAR(50) UNIQUE,
    description     TEXT,
    latitude        DECIMAL(10,8),
    longitude       DECIMAL(11,8),
    building        VARCHAR(100),
    floor           VARCHAR(50),
    room            VARCHAR(50),
    status          INT NOT NULL DEFAULT 1,
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by      UUID NOT NULL,
    updated_by      UUID NOT NULL
);
```


### Assignment \& Logs Tables

#### `assets`

```sql
CREATE TABLE assets (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id           UUID NOT NULL,
    company_type         VARCHAR(255) NOT NULL,
    asset_product_id     UUID NOT NULL REFERENCES asset_products(id),
    serial_number       VARCHAR(255) UNIQUE NOT NULL,
    assigned_to_type     VARCHAR(20),        -- 'user','department','location'
    assigned_to_id       UUID,
    assigned_by          UUID,
    assigned_date        TIMESTAMP,
    expected_return_date TIMESTAMP,
    location_id          UUID REFERENCES locations(id),
    status               INT NOT NULL DEFAULT 1,  -- 0=inactive,1=active,2=deleted
    created_at           TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at           TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


#### `asset_history`

```sql
CREATE TABLE asset_history (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id       UUID NOT NULL,
    company_type     VARCHAR(255) NOT NULL,
    asset_id         UUID NOT NULL REFERENCES assets(id),
    action_type      VARCHAR(50) NOT NULL,  -- assigned, unassigned, location_changed, etc.
    action_details   JSONB NOT NULL,
    user_id          UUID NOT NULL,
    notes            TEXT,
    reference_id     UUID,
    reference_type   VARCHAR(50),
    created_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
#### `action_details Examples`

```
//Assignment

{
  "action_type": "assigned",
  "action_details": {
    "assigned_to_type": "user",
    "assigned_to_id": "user-234",
    "assigned_by": "admin-123",
    "assignment_reason": "Issued for remote work",
    "location_id": "loc-delhi-floor2"
  },
  "user_id": "admin-123",
  "notes": "Assigned for Q3 project"
}

// Unassignment

{
  "action_type": "unassigned",
  "action_details": {
    "unassigned_by": "admin-123",
    "return_notes": "Returned in good condition",
    "previous_user": "user-234"
  },
  "user_id": "admin-123",
  "notes": "Returned after 3 months"
} 

// Location Change

{
  "action_type": "location_changed",
  "action_details": {
    "from_location_id": "loc-delhi-floor2",
    "to_location_id": "loc-mumbai-floor5",
    "reason": "Transferred to Mumbai branch"
  },
  "user_id": "admin-456",
  "notes": "Office relocation"
}
```

### Maintenance Tables

#### `maintenance_requests`

```sql
CREATE TABLE maintenance_requests (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id          UUID NOT NULL,
    company_type        VARCHAR(255) NOT NULL,
    asset_id            UUID NOT NULL REFERENCES assets(id),
    issue_description   TEXT NOT NULL,
    priority            VARCHAR(20) DEFAULT 'medium',
    maintenance_status  VARCHAR(20) DEFAULT 'pending',
    requested_by        UUID NOT NULL,
    requested_at        TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    approved_by         UUID,
    approved_at         TIMESTAMP,
    rejection_reason    TEXT,
    repair_start_date   TIMESTAMP,
    repair_end_date     TIMESTAMP,
    assigned_to_user_id UUID,
    performed_by        UUID,
    vendor_name         VARCHAR(255),
    actual_cost         DECIMAL(12,2),
    ticket_id           UUID,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


## 1. Asset Registration \& Cataloging (AM-001)

```
User Submits Asset Creation Request
    ↓ Validate: serial_number unique, product exists
    ↓ CREATE use existing asset_products entry 
    ↓ CREATE assets record {
           company_id, company_type, asset_product_id,
           status=1, created_by, timestamps
       }
    ↓ INSERT asset_history {
           asset_id, action_type="created",
           action_details:{...}, user_id, notes
       }
    ↓ Publish RabbitMQ 
    ↓ Return success response with asset details
```


## 2. Asset Category Management (AM-006)

```
Admin/Manager Requests Category Create/Update/Delete
    ↓ Validate inputs (name, company context)
    ↓ On Create:
        • INSERT INTO asset_categories
    ↓ On Update:
        • UPDATE asset_categories SET name/status
    ↓ On Delete:
        • UPDATE asset_categories SET status=2
    ↓ Publish RabbitMQ 
    ↓ Return updated category list
```


## 3. Asset Location Tracking (AM-002)

```
User/Admin Updates Asset Location
    ↓ Validate new location_id exists in locations
    ↓ UPDATE assets SET location_id, updated_at
    ↓ INSERT asset_history {action_type="location_changed", details:{from, to}, user_id}
    ↓ Publish “asset.location_changed”
    ↓ Return updated asset record
```


## 4. Asset Assignment Management (AM-003)

```
Manager Calls Assign API
    ↓ Validate asset.status=1 & asset exists
    ↓ Determine assigned_to_type & assigned_to_id
    ↓ UPDATE assets SET assigned_to_type, assigned_to_id, assigned_by, assigned_date
    ↓ INSERT asset_history {action_type="assigned", details, user_id}
    ↓ Publish “asset.assigned”
    ↓ Notify assignee
```


## 5. Barcode/QR Code Generation \& Scanning (AM-004)

```
Generate Barcode/QR:
    ↓ Fetch assets.serial_number
    ↓ Encode into barcode/QR image
    ↓ Return image to client

Scan Barcode/QR:
    ↓ Decode serial_number
    ↓ SELECT assets JOIN asset_products WHERE serial_number
    ↓ Return asset and product details
```


## 6. Asset Status Management (AM-005)

```
Any lifecycle action (assign, maintenance, retire):
    ↓ UPDATE assets.status to new state
    ↓ INSERT asset_history {action_type=status_change, details, user_id}
    ↓ Publish “asset.status_changed”
```

## 7. Stock Level Monitoring (IM-001)

```
Scheduled Job Runs
    ↓ SELECT stock_quantity FROM asset_products
    ↓ Compute available_quantity
    ↓ Publish “stock.snapshot”
```


## 8. Automated Stock Alerts (IM-002)

```
Scheduled Cron
    ↓ SELECT asset_products WHERE stock_quantity < stock_alert_threshold
    ↓ For each:
        • Publish “stock.low”
        • Notify inventory managers
```


## 9. Bulk Asset Operations (IM-003)

```
User Uploads CSV/Excel
    ↓ Parse & map fields
    ↓ For each record:
        • Validate data
        • INSERT/UPDATE asset_products or assets
        • Record success/failure in m_asset_import_logs.details
    ↓ UPDATE m_asset_import_logs.status
    ↓ Publish “import.completed”
    ↓ Return import summary
```


## 10. Maintenance Request Creation from Tickets (MT-001)

```
Ticket Service Publishes “ticket.maintenance.requested”
    ↓ Validate asset_id exists
    ↓ INSERT maintenance_requests {
           asset_id, issue_description, priority,
           maintenance_status="pending", requested_by, requested_at, ticket_id
       }
    ↓ INSERT asset_history {action="maintenance_requested", details, user_id}
    ↓ Publish “maintenance.requested”
    ↓ Notify maintenance team
```


## 11. Maintenance History Tracking (MT-003)

```
Technician Updates Maintenance
    ↓ Validate request status transition
    ↓ UPDATE maintenance_requests SET maintenance_status, repair_dates, actual_cost, performed_by
    ↓ INSERT asset_history {action="maintenance_completed", details, user_id}
    ↓ If completed:
        • UPDATE assets.status=1
        • UPDATE corresponding t_tickets.status="closed"
        • Publish “maintenance.completed”
        • Notify requester
```


## 12. Cost Tracking (FT-001)

```
On Asset Creation/Update or Maintenance Completion
    ↓ Capture purchase_price or actual_cost
    ↓ INSERT asset_history {action="cost_recorded", details, user_id}
    ↓ INSERT m_asset_logs {action="cost_recorded", details, user_id}
    ↓ Publish “cost.updated”
```


## 13. Reporting \& Analytics (RP-001/RP-003/RP-004/RP-005)

```
User Requests Report/Dashboard
    ↓ Authenticate & Authorize
    ↓ Build dynamic query across assets, asset_products, maintenance_requests
    ↓ Apply filters, aggregations, pagination
    ↓ Execute query
    ↓ Cache results (if needed)
    ↓ Return JSON payload for charts/tables
```


## 14. System Integration

### RabbitMQ Messaging (SI-001)

```
On key events (asset.created, asset.assigned, maintenance.requested…)
    ↓ Publish to exchange with routing key
Consumers:
    ↓ Consume and update downstream systems (notifications, search index)
```


### User Service Integration (SI-002)

```
On user.created/updated/deactivated events
    ↓ If deactivated:
        • UPDATE assets SET assigned_to_id=NULL
        • INSERT asset_history & m_asset_logs {action="unassigned", details}
        • Publish “asset.unassigned”
```


### Ticket Service Integration (SI-003)

```
On ticket.asset.* events
    ↓ If maintenance requested: trigger Maintenance Request Creation
    ↓ If ticket closed and linked to asset workflow: finalize and close related processes
```


## 4. Permission Matrix (Scoped by Role \& Table)

| Permission | Tables Affected | Roles | Scopes |
| :-- | :-- | :-- | :-- |
| asset.create | assets | asset.admin, asset.manager | global, department, category |
| asset.read | assets, asset_history | all roles (filtered by scope) | global, department, location |
| asset.update | assets | asset.admin, asset.manager, dept.manager | global, department, assigned |
| asset.delete | assets | asset.admin, asset.manager | global, department |
| asset.assign | assets, asset_history | asset.admin, asset.manager, dept.manager | global, department |
| stock.read | asset_products | inventory.manager, asset.manager | global, department |
| stock.adjust | asset_products, asset_history | inventory.manager | global, department |
| maintenance.request | maintenance_requests | all authenticated users | global, department, assigned |
| maintenance.update | maintenance_requests | maintenance.technician, asset.manager | global, department, assigned |
| report.view | all reporting tables | asset.admin, asset.manager, auditor | global, department |

