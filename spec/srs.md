## Part 1: Domain Glossary

| Term | Definition |
|---|---|
| **Product** | An item tracked by the system and displayed in the store catalog. A product has a name, price, quantity in stock, category, and optional expiration date. |
| **Admin** | An authorized user who can add, update, view, and permanently delete products in the inventory. |
| **Guest** | A user who can view and filter non-expired products in the public catalog without managing inventory. |
| **Inventory** | The collection of products managed by the Admin, including their quantities and product information. |
| **Quantity in Stock** | The current number of units of a product available in inventory. |
| **Stock Status** | A system-calculated status based on quantity: `Out of Stock`, `Running Low`, or `Available`. |
| **Out of Stock** | The stock status assigned when quantity is exactly `0`. |
| **Running Low** | The stock status assigned when quantity is between `1` and `10` units inclusive. |
| **Available** | The stock status assigned when quantity is `11` units or greater. |
| **Expiration Date** | An optional calendar date through which the product remains valid. |
| **Expired Flag** | A computed system state indicating that the current date is after the product’s Expiration Date. It is independent of Stock Status. |
| **Public Catalog** | The product listing accessible to Guests. Expired products are excluded from it. |
| **Admin Inventory Panel** | The inventory view accessible to Admins, including expired products and their expiration warnings. |
| **Category** | A classification assigned to a product and used for catalog filtering. |

## Part 2: Functional Requirements

### Product Management

**REQ-F-001**  
The system shall allow an Admin to add a product with a product name, price, quantity in stock, category, and optional expiration date.

**REQ-F-002**  
The system shall allow an Admin to update the stored information of an existing product.

**REQ-F-003**  
The system shall allow an Admin to permanently delete a product from the system.

**REQ-F-004**  
When a product is deleted, it shall no longer be available in the Admin Inventory Panel or the Public Catalog.

**REQ-F-005**  
The system shall reject any attempt to create or update a product with a negative quantity in stock.

### Stock Status

**REQ-F-006**  
The system shall automatically calculate and display `Out of Stock` when a product’s quantity in stock is exactly `0`.

**REQ-F-007**  
The system shall automatically calculate and display `Running Low` when a product’s quantity in stock is between `1` and `10` units inclusive.

**REQ-F-008**  
The system shall automatically calculate and display `Available` when a product’s quantity in stock is `11` units or greater.

**REQ-F-009**  
The system shall recalculate a product’s Stock Status immediately after any successful change to its quantity in stock.

**REQ-F-010**  
The system shall preserve the product’s calculated Stock Status independently of its Expired Flag.

### Expiration Date

**REQ-F-011**  
The system shall allow the Expiration Date to be empty.

**REQ-F-012**  
If a product has no Expiration Date, the system shall not assign the Expired Flag based on expiration.

**REQ-F-013**  
A product shall remain valid throughout its specified Expiration Date.

**REQ-F-014**  
The system shall assign the Expired Flag when the current date reaches 00:00 on the calendar day after the product’s Expiration Date.

**REQ-F-015**  
The Expired Flag shall be an additional computed state and shall not replace the product’s Stock Status.

**REQ-F-016**  
The Admin Inventory Panel shall display expired products with a clear `Expired` warning.

**REQ-F-017**  
The system shall automatically reevaluate the Expired Flag as the calendar date changes, without requiring the product data to be edited.

### Catalog and Filtering

**REQ-F-018**  
The system shall allow Guests to view products in the Public Catalog.

**REQ-F-019**  
The Public Catalog shall exclude all products with the Expired Flag.

**REQ-F-020**  
Expired products shall be absent from all Guest catalog search results.

**REQ-F-021**  
Expired products shall be absent from all Guest catalog filter results.

**REQ-F-022**  
The system shall allow Guests to filter visible products by Category.

**REQ-F-023**  
The system shall allow Guests to filter visible products by Stock Status.

**REQ-F-024**  
The system shall ensure that catalog filtering does not make an expired product visible to a Guest.

### Access and Scope

**REQ-F-025**  
The system shall permit Admins to manage products and view the complete inventory, including expired products.

**REQ-F-026**  
The system shall permit Guests to view and filter the Public Catalog but shall not permit them to create, modify, or delete products.

**REQ-F-027**  
The system shall not provide shopping-cart, checkout, payment-processing, supplier-management, user-registration, or multi-location-inventory functionality.


## Part 3: Non-Functional Requirements

**REQ-NF-001: Catalog Response Time**  
For normal operating conditions, the system shall return the Guest Public Catalog and requested filters within **2 seconds** of the request.

**REQ-NF-002: Stock Update Consistency**  
When an Admin successfully changes a product’s quantity, the system shall persist the new quantity and corresponding Stock Status as one consistent update.

**REQ-NF-003: Data Persistence Reliability**  
After the system confirms a product change, the change shall remain available after the Admin refreshes the inventory view or reconnects to the system.

**REQ-NF-004: Validation Integrity**  
The system shall not persist a product quantity that is negative or a product update that violates the defined product data rules.

## Part 4: Acceptance Criteria & BDD Scenarios

### 4.1 Stock Status Transition

#### Acceptance Criteria

- An Admin can update a product’s quantity in stock.
- The system rejects negative quantities.
- The system recalculates Stock Status immediately after a successful quantity update.
- A quantity of `0` results in `Out of Stock`.
- A quantity from `1` through `10` inclusive results in `Running Low`.
- A quantity of `11` or greater results in `Available`.
- The updated quantity and Stock Status are both persisted.

#### BDD Scenario: Transition to Running Low

```gherkin
Scenario: Product changes from Available to Running Low
  Given an Admin views a product with a quantity of 12
  And the product has the Stock Status "Available"
  When the Admin changes the quantity to 8
  And the Admin saves the change
  Then the system shall accept the quantity
  And the system shall immediately set the Stock Status to "Running Low"
  And the system shall display the quantity as 8
  And the system shall persist both the quantity and Stock Status
```

#### BDD Scenario: Negative Quantity Rejection

```gherkin
Scenario: Admin attempts to enter a negative quantity
  Given an Admin is editing a product
  When the Admin enters a quantity of -1
  Then the system shall reject the value
  And the system shall not save the product with a negative quantity
  And the product’s previously persisted quantity and Stock Status shall remain unchanged
```

### 4.2 Expiration Date Handling

#### Acceptance Criteria

- A product with an expiration date remains valid through the entire specified date.
- The system sets the Expired Flag at `00:00` on the day after the Expiration Date.
- The Expired Flag is independent of the product’s Stock Status.
- The Admin Inventory Panel continues to display expired products.
- The Admin Inventory Panel displays a clear `Expired` warning.
- Expired products are completely excluded from the Guest Public Catalog.
- Expired products do not appear in Guest search results or filter results.
- Products without an Expiration Date are not flagged as expired.

#### BDD Scenario: Product Becomes Expired

```gherkin
Scenario: Product expiration date passes
  Given a product has an Expiration Date of 2026-10-02
  And the product has a quantity of 15
  And the product has the Stock Status "Available"
  And the product is visible in the Guest Public Catalog on 2026-10-02
  When the system date becomes 2026-10-03 at 00:00
  Then the system shall set the product’s Expired Flag
  And the product shall retain the Stock Status "Available"
  And the Admin Inventory Panel shall display the product
  And the Admin Inventory Panel shall display an "Expired" warning
  And the product shall be absent from the Guest Public Catalog
```

#### BDD Scenario: Expired Product Excluded from Guest Filtering

```gherkin
Scenario: Guest filters the catalog after a product expires
  Given a product has the Expired Flag
  And the product matches the Guest’s selected category and Stock Status filters
  When the Guest applies those filters
  Then the system shall not return the expired product
  And the expired product shall not appear in Guest search results
```

#### BDD Scenario: Product Without Expiration Date

```gherkin
Scenario: Product has no Expiration Date
  Given a product has no Expiration Date
  When the system evaluates the product’s expiration state
  Then the system shall not set the Expired Flag
  And the product shall remain subject only to its Stock Status rules
```

## Initial Traceability Matrix

| Requirement ID | Acceptance Criteria / Description | BDD Scenario | Ticket ID | Test ID | Code Ref |
|---|---|---|---|---|---|
| REQ-F-001 | Admin can create a product with required product data and optional Expiration Date. | N/A |  |  |  |
| REQ-F-002 | Admin can update an existing product. | N/A |  |  |  |
| REQ-F-003 | Admin can permanently delete a product. | N/A |  |  |  |
| REQ-F-004 | Deleted products are absent from inventory and catalog views. | N/A |  |  |  |
| REQ-F-005 | Negative quantities are rejected and not persisted. | Admin attempts to enter a negative quantity |  |  |  |
| REQ-F-006 | Quantity `0` produces `Out of Stock`. | N/A |  |  |  |
| REQ-F-007 | Quantity `1-10` produces `Running Low`. | Product changes from Available to Running Low |  |  |  |
| REQ-F-008 | Quantity `11+` produces `Available`. | N/A |  |  |  |
| REQ-F-009 | Stock Status recalculates immediately after quantity changes. | Product changes from Available to Running Low |  |  |  |
| REQ-F-010 | Expired Flag does not replace Stock Status. | Product expiration date passes |  |  |  |
| REQ-F-011 | Expiration Date may be empty. | Product has no Expiration Date |  |  |  |
| REQ-F-012 | Products without an Expiration Date are not flagged as expired. | Product has no Expiration Date |  |  |  |
| REQ-F-013 | Product remains valid throughout its Expiration Date. | Product expiration date passes |  |  |  |
| REQ-F-014 | Expired Flag is set at 00:00 on the day after Expiration Date. | Product expiration date passes |  |  |  |
| REQ-F-015 | Expired Flag is independent of Stock Status. | Product expiration date passes |  |  |  |
| REQ-F-016 | Admin inventory displays expired products with an `Expired` warning. | Product expiration date passes |  |  |  |
| REQ-F-017 | Expiration state is reevaluated as the calendar date changes. | Product expiration date passes |  |  |  |
| REQ-F-018 | Guests can view the Public Catalog. | N/A |  |  |  |
| REQ-F-019 | Public Catalog excludes expired products. | Product expiration date passes |  |  |  |
| REQ-F-020 | Expired products are absent from Guest search results. | Guest filters the catalog after a product expires |  |  |  |
| REQ-F-021 | Expired products are absent from Guest filter results. | Guest filters the catalog after a product expires |  |  |  |
| REQ-F-022 | Guests can filter products by Category. | N/A |  |  |  |
| REQ-F-023 | Guests can filter products by Stock Status. | N/A |  |  |  |
| REQ-F-024 | Catalog filtering cannot expose expired products. | Guest filters the catalog after a product expires |  |  |  |
| REQ-F-025 | Admin can manage and view the complete inventory, including expired products. | Product expiration date passes |  |  |  |
| REQ-F-026 | Guests can view/filter products but cannot modify inventory. | N/A |  |  |  |
| REQ-F-027 | Shopping cart, checkout, payment, supplier management, registration, and multi-location inventory are excluded. | N/A |  |  |  |
| REQ-NF-001 | Public Catalog and filter responses complete within 2 seconds under normal conditions. | N/A |  |  |  |
| REQ-NF-002 | Quantity and Stock Status are persisted as one consistent update. | Product changes from Available to Running Low |  |  |  |
| REQ-NF-003 | Confirmed product changes survive refresh or reconnection. | N/A |  |  |  |
| REQ-NF-004 | Invalid quantities and invalid product updates are never persisted. | Admin attempts to enter a negative quantity |  |  |  |