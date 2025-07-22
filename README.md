
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

#### `asset_categories` NO CHANGE `Main Category like Electronics, etc`

```sql
CREATE TABLE m_asset_categories (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID NOT NULL,
    company_type    VARCHAR(255) NOT NULL,
    name            VARCHAR(255) NOT NULL,
    status          INT NOT NULL,          -- 0=inactive,1=active,2=deleted
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


#### `asset_products` NO CHANGE

```sql

CREATE TABLE m_asset_products (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name                VARCHAR(255) NOT NULL, --name will act as sub category because of the current data
    status              INT NOT NULL,          -- 0=inactive,1=active,2=deleted 
    company_id          UUID NOT NULL,
    company_type        VARCHAR(255) NOT NULL,
    category_id         UUID NOT NULL REFERENCES asset_categories(id),
    purchase_from varchar(255) [not null],
    purchase_date timestamp [not null],
    brand               VARCHAR(255),
    model               VARCHAR(255),
    serial_no varchar(255) [not null], --Unique for all products,
    warranty timestamp [not null],
    createdAt timestamp [not null],
    updatedAt timestamp [not null],
    image               TEXT,
    
);
```


#### `locations` NEW TABLE

```sql
CREATE TABLE m_locations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID NOT NULL,
    company_type    VARCHAR(255) NOT NULL,
    status          INT NOT NULL,          -- 0=inactive,1=active,2=deleted 
    location_name   VARCHAR(255) NOT NULL,

    --location information, helpful in case of tracking location of assets like AC, 
    building        VARCHAR(100),
    floor           VARCHAR(50),
    room            VARCHAR(50),

    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
);
```


### Assignment \& Logs Tables

#### `assets` NEW COLUMN `assigned_to_type` to check whether the asset is assigned to department or location (eg. RO, AC)

```sql

CREATE TABLE m_assets (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id varchar(255) [not null],
    product_id varchar(255) [not null],
    assigned_to varchar(255), --id of user, department, or location
    status               INT NOT NULL DEFAULT 1,  -- 0=inactive,1=active,2=deleted
    assigned_date timestamp [not null],
    assigned_by varchar(255) [not null],
    createdAt timestamp [not null],
    updatedAt timestamp [not null],
    company_type varchar(255),
    serial_no varchar(255),   -- why we are using ?
    asset_category_id int,
    unassigned_date timestamp,
    
    -- new columns

    assigned_to_type     VARCHAR(20),        -- 'user','department','location'
);
```


#### `asset_history` NEW TABLE to keep logs of asset ownership history and maintenance history

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
// Maintenance Requests
```



## 1. Asset Registration \& Cataloging (AM-001)

```
User requests to assign an asset (select Category, Product, assignee)
          ↓
Query: Get available product selection list:
          ↓
    [Fetch all products p where
        (
          (p.id NOT IN (SELECT product_id FROM m_assets))    // products not yet assets
        OR
          (p.id IN (SELECT product_id FROM m_assets WHERE assigned_to IS NULL)) // assets unassigned
        )
     ]
          ↓
User selects product(s)
          ↓
For each selected product:
    Check: Does asset exist for this product?
          ↓                   ↓
   [No, create asset]      [Yes, asset exists]
   INSERT into             Get asset record with assigned_to IS NULL
   m_assets (product_id, etc, set assigned_to)
          ↓                   ↓
Update asset record: set
    assigned_to = (user/department/location ID)
    assigned_to_type = 'user'/'department'/'location'
    assigned_date = now
    status = 1   // Active
          ↓
Insert entry in asset_history:
    action_type = 'assigned'
    user_id = assigner
    action_details = assignment details (who, when, to whom)
          ↓
Done. Asset assigned and tracked.

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
    
User requests to unassign asset (asset_id)
          ↓
Fetch asset record, check assigned_to IS NOT NULL
          ↓
Update asset record:
    assigned_to = NULL
    assigned_to_type = NULL
    unassigned_date = now
    status = 1 // Open/Available
          ↓
Insert entry in asset_history:
    action_type = 'unassigned'
    user_id = operator
    action_details = unassignment details
          ↓
Asset successfully unassigned, now appears as available for next assignment.

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

