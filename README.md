# ER Model — Shopping Malls Management System

Notation used: **(PK)** = primary key, **(FK)** = foreign key (only relevant for weak entities), *italic* = composite attribute (sub-parts listed indented), `{multivalued}` = multivalued attribute, `/derived/` = derived attribute.

---

## 1. Entities and Attributes

### 1. Mall (strong entity)
- **mall_id (PK)**
- mall_name
- *address*
  - latitue
  - longitude
  - street
  - city
  - state
  - pincode
- contact_number 
- opening_date
- total_area_sqft
- `/occupancy_rate/` — derived from (occupied stores ÷ total stores)

### 2. Enterprise_Executive (strong entity)
- **executive_id (PK)**
- *name*
  - first_name
  - last_name
- email
- phone_number
- date_joined

### 3. Mall_Manager (strong entity)
- **manager_id (PK)**
- *name*
  - first_name
  - last_name
- email
- phone_number
- date_joined

### 4. Tenant (strong entity)
- **tenant_id (PK)**
- business_name
- email
- phone_number
- business_type
- date_registered

### 5. Employee (strong entity)
- **employee_id (PK)**
- *name*
  - first_name
  - last_name
- email
- phone_number
- current_designation
- date_of_joining
- base_salary
- `/years_of_service/` — derived from date_of_joining

### 6. User (strong entity)
- **user_id (PK)**
- *name*
  - first_name
  - last_name
- email
- phone_number

### 7. Store (strong entity)
- **store_id (PK)**
- store_name
- shop_number
- floor
- area_sqft
- category
- status (occupied / vacant)
- `{listing_media}` — pictures and videos (relevant while vacant, for bidders)

### 8. Product (weak entity — identifying owner: Store)
- **product_id (partial key)**
- product_name
- price

### 9. Discount_Offer (weak entity — identifying owner: Store)
- **offer_id (partial key)**
- description
- discount_percentage
- start_date
- end_date
- `/is_active/` — derived from current date vs. start_date/end_date

### 10. Bid (weak entity — identifying owner: Store)
- **bid_id (partial key)**
- round_number
- bid_amount
- bid_date
- status (pending / accepted / rejected)

### 11. Financial_Transaction (weak entity — identifying owner: Tenant)
- **transaction_id (partial key)**
- transaction_type (rent / fee / annual_revenue_report / annual_profit_report / other)
- amount
- {debit/credit}
- transaction_date
- remarks

### 12. Leave_Request (weak entity — identifying owner: Employee)
- **request_id (partial key)**
- start_date
- end_date
- reason
- status (pending / approved / rejected)

### 13. Payroll_Record (weak entity — identifying owner: Employee)
- **record_id (partial key)**
- record_type (salary / bonus / insurance)
- amount
- issue_date

### 14. Attendance (weak entity — identifying owner: Employee)
- **date (partial key)**
- check_in_time
- check_out_time
- `/status/` — derived (present / absent / late, computed from times)

### 15. Bid Event ( weak entity - identifying owner: store)
 - event_id
 - start_date
 - end_date
 - minimum_bid_amount
 - current_maximum_bid - derived
 - participating fee
 - final_allocation
---

## 2. Relationships (all binary)

| # | Relationship | Entities | Cardinality | Participation | Notes |
|---|---|---|---|---|---|
| 1 | **Oversees** | Enterprise_Executive – Mall | M:N | Total on neither side (partial-partial) | An executive can oversee multiple malls; a mall can be reviewed by multiple executives |
| 2 | **Manages** | Mall_Manager – Mall | 1:1 | Total on Mall side | Every mall has exactly one manager |
| 3 | **Employs (Mall)** | Mall – Employee | 1:N | Total on Employee side (for mall-hired staff) | Direct mall staff |
| 4 | **Employs (Tenant)** | Store – Employee | 1:N | Total on Employee side (for tenant-hired staff) | Tenant's own staff, using the same portal |
| 5 | **Has_Store** | Mall – Store | 1:N | Total on Store side | Every store belongs to exactly one mall |
| 6 | **Owns** | Tenant – Store | 1:N | Partial on Store side (vacant stores have no tenant) | A tenant may hold more than one store across the chain |
| 7 | **Places** | Tenant – Bid | 1:N | Total on Bid side | Identifying relationship for Bid (weak entity) |
| 8 | **Receives_Bid** | Store – Bid | 1:N | Total on Bid side | Identifying relationship for Bid; a vacant store receives multiple bids across rounds |
| 9 | **Has_Transaction** | Tenant – Financial_Transaction | 1:N | Total on Financial_Transaction side | Identifying relationship (weak entity) |
| 10 | **Requests_Leave** | Employee – Leave_Request | 1:N | Total on Leave_Request side | Identifying relationship (weak entity) |
| 11 | **Has_Payroll_Record** | Employee – Payroll_Record | 1:N | Total on Payroll_Record side | Identifying relationship (weak entity) |
| 12 | **Has_Attendance** | Employee – Attendance | 1:N | Total on Attendance side | Identifying relationship (weak entity); date is partial key |
| 13 | **Lists_Product** | Store – Product | 1:N | Total on Product side | Identifying relationship (weak entity) |
| 14 | **Offers_Discount** | Store – Discount_Offer | 1:N | Total on Discount_Offer side | Identifying relationship (weak entity) |

---

## 3. Design Notes / Assumptions

- **Employee** is a single entity, connected via two separate 1:N relations — one to `Mall` (mall-hired staff) and one to `Store` (tenant-hired staff). Each employee record participates in only one of the two relations in practice, but both are modeled since the same self-service portal structure (leave, payroll, attendance) applies to both groups.
- **Financial_Transaction** is a single unified weak entity covering both routine transactions (rent, fees) and yearly compliance submissions (revenue/profit reports), distinguished by `transaction_type`.
- **Store** is a single entity with a `status` attribute (occupied/vacant) rather than two separate entities — a store's lifecycle (vacant → bid → occupied → vacant again) is handled as a state change rather than a type change.
- **Customer** browsing/searching (finding malls, viewing stores, checking discounts/top products) is treated as **read-only querying** over existing entities (Mall, Store, Product, Discount_Offer) per the RBAC section, not as a stored relationship. No transactional relationship is created for Customers unless you want to track things like favorites/wishlists — happy to add that if needed.
- Redundant relationships were deliberately avoided where a link is transitively derivable (e.g., Mall_Manager to Tenant, or Enterprise_Executive to Employee) via existing 1:1/1:N chains (Manager→Mall→Store→Tenant), to keep the model at binary relations without duplicating information.
- No new weak entities were introduced beyond what's implied by the requirements (Bid, Financial_Transaction, Leave_Request, Payroll_Record, Attendance, Product, Discount_Offer) — each has exactly one identifying relationship, consistent with your instruction to avoid creating weak entities that don't need multiple relations.
- **Financial_Transaction vs. Payroll_Record**: kept as two separate weak entities on purpose, not redundant. `Financial_Transaction` tracks money between Tenant and Mall (rent, fees, annual revenue/profit reports); `Payroll_Record` tracks money from Mall/Tenant to Employee (salary, bonus, insurance). Different owners, different attributes, different portals — merging them would force irrelevant fields onto both.
- **Product stays a weak entity scoped to one Store** (not a shared catalog with an M:N "Sells" relationship to Store). The proposal only requires customers to browse a store's own top-selling products — there's no requirement for cross-store product search or price comparison. Since the spec never needs "which stores sell iPhone 15," a shared-catalog model would add a relationship and junction complexity the requirements don't call for. If a future requirement introduces cross-store product search/comparison, this can be upgraded to a strong `Product` entity with an M:N `Sells` relationship (price and units_sold moving to relationship attributes).
