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

### Empty States and Validation

**REQ-F-028**
When a Guest applies a Category or Stock Status filter and no non-expired products match, the system shall display an empty-state message stating `No products match the selected filters.` The catalog shall display no product cards, retain the selected filter values, and provide a `Clear filters` action that restores the unfiltered catalog.

**REQ-F-029**
The system shall require Product Name to contain between **1 and 100 characters**, inclusive, after leading and trailing whitespace is removed. Product Name shall allow Unicode letters, numbers, spaces, and the symbols hyphen (`-`), apostrophe (`'`), period (`.`), ampersand (`&`), and parentheses (`(` and `)`). The system shall reject an empty, whitespace-only, over-length, or otherwise invalid Product Name and shall identify the field-level validation error without persisting the product.

**REQ-F-030**
The system shall require Price to be a numeric value strictly greater than `0`, represented in the system’s configured currency with no more than two digits after the decimal separator. The system shall reject zero, negative values, text, non-finite values, and values with more than two decimal places, and shall not persist the product when validation fails.

**REQ-F-031**
When provided, Expiration Date shall use the exact `YYYY-MM-DD` format and shall represent a valid calendar date. The system shall reject other formats and impossible dates, including dates such as `2026-02-31`, with a field-level validation error and shall not persist the invalid value. An empty Expiration Date shall remain valid as defined by REQ-F-011.

### Category Management

**REQ-F-032**
Categories shall be managed data rather than an immutable hard-coded list. An Admin shall be able to create a category through category management and select an existing category when adding or editing a product. The system may offer category creation from the product form, but the new category shall be created and validated using the same category-management rules before the product is saved.

**REQ-F-033**
The system shall require each category name to be non-empty, unique case-insensitively, and no longer than 50 characters after trimming whitespace. The system shall reject duplicate or invalid category names and shall not create a partial category record.

**REQ-F-034**
The system shall prevent an Admin from deleting a category that is assigned to one or more active products. The system shall display the affected-product count and instruct the Admin to reassign those products before retrying deletion. A category deletion shall not modify or delete products.

### Exceptional and Concurrent Operations

**REQ-F-035**
If an Admin attempts to update or delete a product that no longer exists because it was permanently deleted by another session, the system shall reject the operation, display `Product no longer exists. Refresh the inventory and try again.`, and leave all remaining inventory unchanged.

**REQ-F-036**
If an inventory update cannot be persisted because of a database or network failure, the system shall report that the update was not saved, retain the Admin’s unsaved values in the form where possible, and leave the previously persisted product quantity, Stock Status, and other product fields unchanged.

**REQ-F-037**
The system shall prevent a failed or concurrent inventory operation from producing a partial update. A successful update shall persist all changed product fields and the corresponding calculated Stock Status together, or shall persist none of them.

### Catalog Pagination and Sorting

**REQ-F-038**
The Guest Public Catalog shall display at most **20 products per page**. The system shall provide page navigation when more than 20 eligible products exist, including the current page number, total page count, `Next` and `Previous` controls, and direct navigation to available page numbers. The `Previous` control shall be unavailable on the first page and the `Next` control shall be unavailable on the last page.

**REQ-F-039**
The system shall allow Guests to sort eligible catalog results by Product Name ascending (A-Z), Product Name descending (Z-A), Price ascending (lowest to highest), or Price descending (highest to lowest). The selected sort order shall remain applied while the Guest changes pages or applies Category or Stock Status filters. The default sort order shall be Product Name ascending. Products with equal sort values shall use a deterministic secondary order based on their product identifier.

### Concurrency and Access Security

**REQ-F-040**
The system shall use optimistic locking for product updates. Each product edit shall carry the version or modification state observed when editing began. If two Admins attempt to update the same product and the first update succeeds, the second stale update shall be rejected with `Product was changed by another Admin. Refresh the product and review the current values.` The second update shall not overwrite the first update, and the Admin shall be able to reload the current product before retrying.

**REQ-F-041**
The Admin Inventory Panel and every inventory-management API operation shall require an authenticated Admin identity. An unauthenticated Guest or a user without Admin authorization who accesses an Admin URL or calls an inventory API directly shall be denied access, shall not receive inventory data, and shall not be able to create, update, or delete products or categories. Public catalog access shall remain available to Guests as defined by REQ-F-018 and REQ-F-026.

### Modification Metadata

**REQ-F-042**
For every successfully persisted product creation or update, the system shall store and display the product’s `Last Modified` timestamp in an unambiguous ISO 8601 representation including timezone information. The timestamp shall be generated by the system, shall change after each successful update, and shall not change after a rejected, failed, or read-only operation. A permanent deletion shall remove the product and its modification metadata.


## Part 3: Non-Functional Requirements

**REQ-NF-001: Catalog Response Time**  
For normal operating conditions, the system shall return the Guest Public Catalog and requested filters within **2 seconds** of the request.

**REQ-NF-002: Stock Update Consistency**  
When an Admin successfully changes a product’s quantity, the system shall persist the new quantity and corresponding Stock Status as one consistent update.

**REQ-NF-003: Data Persistence Reliability**  
After the system confirms a product change, the change shall remain available after the Admin refreshes the inventory view or reconnects to the system.

**REQ-NF-004: Validation Integrity**  
The system shall not persist a product quantity that is negative or a product update that violates the defined product data rules.

**REQ-NF-005: Input Format and Constraint Enforcement**
The system shall enforce the Product Name, Price, Category, and Expiration Date constraints at the user-interface and persistence boundaries, using the same rules for create and update operations. Validation failures shall be deterministic, field-specific where applicable, and shall not write invalid data.

**REQ-NF-006: Failure Transparency and Recovery**
For database, network, validation, and missing-record failures during inventory operations, the system shall provide an actionable error to the Admin, shall not report success, and shall preserve the last successfully persisted inventory state. Retrying after a transient connection failure shall not duplicate or partially apply the operation.

**REQ-NF-007: Concurrent Data Integrity**
The system shall handle concurrent product and category operations deterministically. A stale update or deletion shall be rejected when its target has already been permanently deleted, and no operation shall overwrite a missing record or silently discard another completed operation.

**REQ-NF-008: Catalog Paging Performance and Stability**
Under normal operating conditions, each Guest catalog page, sort operation, and filtered-page request shall be returned within **2 seconds**. Pagination shall be applied after expired-product exclusion and the selected filters and sort order, and the system shall not return duplicate or missing products solely because the Guest navigated between pages during one catalog retrieval.

**REQ-NF-009: Authentication and Authorization Enforcement**
The system shall enforce Admin authentication and authorization on the server or API boundary, not only through UI navigation controls. Unauthorized requests shall return the repository-standard authorization response, shall not disclose whether protected products or categories exist, and shall leave inventory data unchanged.

**REQ-NF-010: Modification Metadata Integrity**
System-generated `Last Modified` timestamps shall use a synchronized server-side clock, be stored with timezone information, and remain consistent with the successful persistence transaction. A timestamp shall not be presented as updated unless the corresponding product change was committed successfully.

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

### 4.3 Empty Results and Field Validation

#### Acceptance Criteria

- A zero-result Category or Stock Status filter displays `No products match the selected filters.`
- No product cards are displayed in the empty state.
- Selected filters remain visible and a `Clear filters` action is available.
- Clearing filters restores the eligible, non-expired catalog.
- Product Name accepts 1-100 trimmed characters only and rejects unsupported symbols, blank input, and over-length input.
- Price accepts only finite values greater than zero with at most two decimal places.
- Price rejects zero, negative, text, and excessive-precision values.
- Expiration Date accepts either an empty value or a real date in `YYYY-MM-DD` format.
- Invalid calendar dates, including February 31st, are rejected and are not persisted.
- Validation errors identify the affected field and leave the last persisted product unchanged.

#### BDD Scenario: Guest Receives an Empty Filter State

```gherkin
Scenario: Selected filters match no visible products
  Given the Public Catalog contains no non-expired products in the category "Bakery"
  When the Guest selects the category "Bakery"
  Then the system shall display "No products match the selected filters."
  And the system shall display no product cards
  And the category filter shall remain selected
  And the system shall provide a "Clear filters" action
  When the Guest selects "Clear filters"
  Then the system shall restore the unfiltered eligible catalog
```

#### BDD Scenario: Invalid Product Fields Are Rejected

```gherkin
Scenario: Admin submits invalid product data
  Given an Admin is adding a product
  When the Admin enters a blank Product Name
  And the Admin enters a Price of -1.005
  And the Admin enters an Expiration Date of "2026-02-31"
  And the Admin submits the product
  Then the system shall reject the submission
  And the system shall identify the Product Name, Price, and Expiration Date errors
  And the system shall not persist the product
```

### 4.4 Category Lifecycle

#### Acceptance Criteria

- An Admin can create a category through category management and use it for a product.
- Category names are trimmed, non-empty, case-insensitively unique, and at most 50 characters.
- Invalid or duplicate categories are rejected without creating a partial record.
- A category assigned to active products cannot be deleted.
- A rejected category deletion reports the number of affected products and leaves all products unchanged.
- After all affected products are reassigned, the now-unused category can be deleted.

#### BDD Scenario: Category Deletion Is Blocked by Active Products

```gherkin
Scenario: Admin attempts to delete a category in use
  Given the category "Bakery" is assigned to 3 active products
  When the Admin attempts to delete the category "Bakery"
  Then the system shall reject the deletion
  And the system shall report that 3 active products use the category
  And the system shall instruct the Admin to reassign those products
  And the category and all 3 products shall remain unchanged
```

### 4.5 Missing Records and Persistence Failures

#### Acceptance Criteria

- Updating or deleting a product deleted by another session is rejected with the defined missing-record message.
- A missing-record operation does not create, restore, or modify any product.
- A database or network failure reports that the inventory update was not saved.
- The last persisted product values remain visible after a failed update or refresh.
- A failed update does not change quantity, Stock Status, or any other product field.
- Product updates are atomic: all changed fields and derived Stock Status are persisted together or none are persisted.
- Retrying a transient failure does not duplicate a product or apply fields more than once.

#### BDD Scenario: Admin Uses a Stale Product Record

```gherkin
Scenario: Product was deleted by another session
  Given Admin A is editing product "Milk"
  And Admin B permanently deletes product "Milk"
  When Admin A attempts to save the edit
  Then the system shall reject the update
  And the system shall display "Product no longer exists. Refresh the inventory and try again."
  And the system shall not recreate or modify product "Milk"
```

#### BDD Scenario: Inventory Update Cannot Be Persisted

```gherkin
Scenario: Database or network failure occurs during an inventory update
  Given a product has a persisted quantity of 8 and Stock Status "Running Low"
  When an Admin changes the quantity to 15
  And persistence fails because the database or network is unavailable
  Then the system shall report that the update was not saved
  And the persisted quantity shall remain 8
  And the persisted Stock Status shall remain "Running Low"
  And the system shall not report a successful update
```

### 4.6 Catalog Pagination and Sorting

#### Acceptance Criteria

- The Guest Public Catalog displays no more than 20 products on a page.
- When more than 20 eligible products exist, the system displays the current page, total page count, and page navigation controls.
- `Previous` is unavailable on the first page and `Next` is unavailable on the last page.
- Pagination is applied only after expired products are excluded and filters and sorting are applied.
- Guests can sort by Product Name A-Z, Product Name Z-A, Price ascending, and Price descending.
- Product Name ascending is the default sort order.
- The selected sort remains active across page navigation and filtering.
- Equal sort values use a deterministic product-identifier tie-breaker.
- Catalog page, sort, and filtered-page requests complete within 2 seconds under normal conditions.

#### BDD Scenario: Guest Navigates a Paginated Catalog

```gherkin
Scenario: Guest views the second page of catalog results
  Given 21 non-expired products match the current catalog filters
  And the default sort is Product Name ascending
  When the Guest opens the Public Catalog
  Then the system shall display 20 products on page 1 of 2
  And the Previous control shall be unavailable
  And the Next control shall be available
  When the Guest selects page 2
  Then the system shall display the remaining 1 product
  And the Next control shall be unavailable
  And the Previous control shall be available
```

#### BDD Scenario: Guest Sorts Filtered Results

```gherkin
Scenario: Guest sorts products by price descending
  Given the Guest has selected the Stock Status "Available"
  When the Guest selects the Price descending sort
  Then the system shall display only non-expired Available products
  And the products shall be ordered from highest price to lowest price
  And the selected Price descending sort shall remain active when the Guest changes page
```

### 4.7 Concurrent Updates and Admin Authorization

#### Acceptance Criteria

- Product updates include the version or modification state read when editing began.
- The first successful update of a product is committed normally.
- A stale concurrent update is rejected with the defined conflict message.
- A rejected stale update does not overwrite any field or Stock Status from the successful update.
- The Admin can refresh the product and retry using current values.
- The Admin Inventory Panel requires authentication and Admin authorization.
- Direct unauthenticated or unauthorized URL and API access is denied without protected data disclosure.
- Guest access to the Public Catalog remains available.

#### BDD Scenario: Concurrent Stock Updates Produce a Conflict

```gherkin
Scenario: Two Admins edit the same product
  Given Admin A and Admin B both open product "Milk" with quantity 8
  When Admin A saves a quantity of 12
  And Admin B saves a quantity of 15 using the stale product version
  Then Admin A's update shall succeed
  And Admin B's update shall be rejected
  And the system shall display "Product was changed by another Admin. Refresh the product and review the current values."
  And the persisted quantity shall remain 12
  And the persisted Stock Status shall remain "Available"
```

#### BDD Scenario: Guest Is Denied Admin Inventory Access

```gherkin
Scenario: Guest attempts direct Admin access
  Given the user is not authenticated as an Admin
  When the user requests the Admin Inventory Panel URL or an inventory-management API
  Then the system shall deny the request
  And the system shall not return inventory data
  And the system shall not create, update, or delete any product or category
```

### 4.8 Modification Metadata

#### Acceptance Criteria

- A successfully created or updated product has a system-generated ISO 8601 `Last Modified` timestamp with timezone information.
- A successful product update changes the timestamp.
- Rejected validation, conflict, missing-record, and persistence-failure operations do not change the timestamp.
- Read-only operations do not change the timestamp.
- The timestamp is committed only with the corresponding successful product change.
- Permanent deletion removes the product and its modification metadata.

#### BDD Scenario: Last Modified Changes Only After Success

```gherkin
Scenario: Product modification timestamp reflects successful updates only
  Given product "Milk" has a stored Last Modified timestamp T1
  When an Admin successfully changes its price
  Then the system shall store a Last Modified timestamp later than T1 in ISO 8601 format with timezone information
  When a subsequent update fails to persist
  Then the system shall retain the previously stored product values
  And the system shall retain the successful update timestamp
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
| REQ-F-028 | Empty filtered results show the defined message, preserve filters, and offer `Clear filters`. | Guest receives an empty filter state |  |  |  |
| REQ-F-029 | Product Name is trimmed, 1-100 characters, and limited to the defined symbols. | Invalid product fields are rejected |  |  |  |
| REQ-F-030 | Price is finite, strictly greater than zero, and has at most two decimal places. | Invalid product fields are rejected |  |  |  |
| REQ-F-031 | Expiration Date is empty or a valid `YYYY-MM-DD` calendar date. | Invalid product fields are rejected |  |  |  |
| REQ-F-032 | Admin-managed categories can be created and selected during product management. | N/A |  |  |  |
| REQ-F-033 | Category names are valid, trimmed, and case-insensitively unique. | N/A |  |  |  |
| REQ-F-034 | Categories assigned to active products cannot be deleted. | Category deletion is blocked by active products |  |  |  |
| REQ-F-035 | Updates/deletions of missing products are rejected with an actionable message. | Admin uses a stale product record |  |  |  |
| REQ-F-036 | Database/network failures do not save inventory changes and report failure. | Inventory update cannot be persisted |  |  |  |
| REQ-F-037 | Inventory updates are atomic and cannot partially apply. | Inventory update cannot be persisted |  |  |  |
| REQ-F-038 | Guest catalog uses 20-item pages with navigable page controls. | Guest navigates a paginated catalog |  |  |  |
| REQ-F-039 | Guests can select the defined sort orders with stable tie-breaking. | Guest sorts filtered results |  |  |  |
| REQ-F-040 | Stale same-product updates are rejected using optimistic locking. | Concurrent stock updates produce a conflict |  |  |  |
| REQ-F-041 | Admin inventory UI and APIs require authenticated Admin authorization. | Guest is denied Admin inventory access |  |  |  |
| REQ-F-042 | Successful product changes store system-generated Last Modified metadata. | Last Modified changes only after success |  |  |  |
| REQ-NF-001 | Public Catalog and filter responses complete within 2 seconds under normal conditions. | N/A |  |  |  |
| REQ-NF-002 | Quantity and Stock Status are persisted as one consistent update. | Product changes from Available to Running Low |  |  |  |
| REQ-NF-003 | Confirmed product changes survive refresh or reconnection. | N/A |  |  |  |
| REQ-NF-004 | Invalid quantities and invalid product updates are never persisted. | Admin attempts to enter a negative quantity |  |  |  |
| REQ-NF-005 | Validation constraints are enforced consistently at UI and persistence boundaries. | Invalid product fields are rejected |  |  |  |
| REQ-NF-006 | Failures are transparent, actionable, and preserve the last persisted state. | Inventory update cannot be persisted |  |  |  |
| REQ-NF-007 | Concurrent operations are deterministic and preserve data integrity. | Admin uses a stale product record |  |  |  |
| REQ-NF-008 | Catalog page, sort, and filtered-page requests meet the 2-second target and remain stable. | Guest navigates a paginated catalog |  |  |  |
| REQ-NF-009 | Admin authorization is enforced at protected UI and API boundaries without data disclosure. | Guest is denied Admin inventory access |  |  |  |
| REQ-NF-010 | Last Modified metadata is server-generated and transactionally consistent. | Last Modified changes only after success |  |  |  |