# InventoryPro - REST API Design Document

## Executive Summary

**Context**: InventoryPro is an API-driven inventory management system designed for African retail businesses operating across multiple physical locations. The primary clients are a modern web dashboard for managers and a mobile application for floor staff to perform stock checks and updates.

**Approach**: This design follows a strict resource-oriented REST architecture. It uses HTTP methods semantically, JSON for data exchange, and standard HTTP status codes. The initial version is placed under a `/v1` path prefix for clear versioning. The design prioritizes clarity, predictability, and robustness for client developers.

**Scope**: This document covers the identification and modeling of four core resources (products, categories, suppliers, locations) and their relationships. It provides full CRUD endpoint specifications, a consistent error handling model, and designs for pagination, filtering, and key domain-specific operations like stock transfer and low-stock alerts.

---

## Business Domain Analysis

**Business Goals**

- Reduce stock-out incidents by 25% within the first year of use.
- Enable rapid onboarding of new suppliers and products into the system.
- Provide accurate, real-time visibility of inventory levels across all store and warehouse locations.
- Support efficient inventory auditing and reconciliation processes.
- Facilitate data-driven decisions on restocking and product placement.

**Primary Entities (identification & justification)**

- **Product**: The central entity of the entire system. It represents a unique, sellable item. Accurate product data is critical for all business operations, from sales to supply chain management.
- **Category**: A logical grouping entity. Categories are essential for organizing thousands of products, enabling efficient browsing, filtering, and generating structured reports (e.g., "sales performance in the Electronics category").
- **Supplier**: A partner entity representing the source of products. Managing supplier data is vital for procurement, cost analysis, and building resilient supply chains, which is a key challenge for retailers.
- **Location**: A physical place entity. This is non-negotiable for businesses with more than one store or a warehouse. Inventory levels are meaningless without being tied to a specific location, making this a core entity.

**Key Relationships (narrative)**

- A Product must belong to exactly one Category (e.g., a Samsung phone belongs to "Electronics").
- A Product is sourced from one primary Supplier.
- A Product can be stocked in multiple Locations (e.g., a product can be in both the Accra and Kumasi stores).
- Conversely, a Location contains stock for many Products. This is a many-to-many relationship.

**Critical Operations (verbs in business language)**

- Onboard new products and suppliers.
- Adjust stock levels up (after new shipment) or down (after a sale).
- Transfer stock between locations to balance inventory.
- Search and filter products by various criteria.
- Identify products that are low in stock or out of stock.

**Data Requirements (high-level)**

- **Product**: Unique identifier (SKU - Stock Keeping Unit), name, description, cost price, selling price, category, supplier.
- **Category**: Name, description.
- **Supplier**: Company name, contact information (email, phone), physical address.
- **Location**: Store name, physical address, operational status.

---

## Resource Specifications

### Resource: Product

**Purpose**: Represents a unique, sellable item in the inventory. It holds the universal attributes of an item that are consistent across all locations where it is stocked.

**Attributes**

| Attribute    | Type        | Required | Constraints/Format    | Notes                                   |
|--------------|-------------|:--------:|-----------------------|-----------------------------------------|
| id           | string/uuid |    ✓     | immutable             | Server-generated unique identifier      |
| sku          | string      |    ✓     | 3-50 chars, alphanumeric | Unique across all products           |
| name         | string      |    ✓     | 1-120 chars           |                                         |
| description  | string      |          | 0-500 chars           |                                         |
| price        | number      |    ✓     | positive              | Current selling price                   |
| cost         | number      |          | positive              | Procurement cost from supplier          |
| categoryId   | string/uuid |    ✓     |                       | Must reference an existing category     |
| supplierId   | string/uuid |    ✓     |                       | Must reference an existing supplier     |
| isActive     | boolean     |    ✓     |                       | Default: true (soft delete flag)        |
| createdAt    | datetime    |    ✓     | ISO 8601              | Server-generated                        |
| updatedAt    | datetime    |    ✓     | ISO 8601              | Server-generated                        |

**Relationships**

- Many-to-One with Category (via categoryId).
- Many-to-One with Supplier (via supplierId).
- Many-to-Many with Location. This relationship is managed through an inventory association resource that tracks the quantity at each location.

**Business Rules & Constraints**

- The `sku` must be unique across the entire system.
- `price` must be greater than or equal to `cost`.
- A product cannot be deleted if it has historical transactions or current stock; it must be soft-deleted (`isActive: false`).

### Resource: Category

**Purpose**: Provides a logical hierarchy for grouping products, making inventory easier to navigate and manage.

**Attributes**

| Attribute   | Type        | Required | Constraints/Format | Notes                              |
|-------------|-------------|:--------:|--------------------|------------------------------------|
| id          | string/uuid |    ✓     | immutable          | Server-generated unique identifier |
| name        | string      |    ✓     | 1-60 chars         | Unique across all categories       |
| description | string      |          | 0-255 chars        |                                    |
| createdAt   | datetime    |    ✓     | ISO 8601           | Server-generated                   |
| updatedAt   | datetime    |    ✓     | ISO 8601           | Server-generated                   |

**Relationships**

- One-to-Many with Product (a category has many products).

**Business Rules & Constraints**

- The `name` must be unique.
- A category cannot be deleted if it has products associated with it.

### Resource: Supplier

**Purpose**: Represents a vendor or provider from whom products are procured. Critical for managing the supply chain.

**Attributes**

| Attribute      | Type        | Required | Constraints/Format | Notes                              |
|----------------|-------------|:--------:|--------------------|------------------------------------|
| id             | string/uuid |    ✓     | immutable          | Server-generated unique identifier |
| name           | string      |    ✓     | 1-120 chars        |                                    |
| contactEmail   | string      |          | email format       |                                    |
| contactPhone   | string      |          |                    |                                    |
| addressStreet  | string      |          |                    |                                    |
| addressCity    | string      |          |                    |                                    |
| addressCountry | string      |          |                    |                                    |
| createdAt      | datetime    |    ✓     | ISO 8601           | Server-generated                   |
| updatedAt      | datetime    |    ✓     | ISO 8601           | Server-generated                   |

**Relationships**

- One-to-Many with Product (a supplier provides many products).

**Business Rules & Constraints**

- A supplier cannot be deleted if it has products associated with it.

### Resource: Location

**Purpose**: Represents a physical store or warehouse where inventory is stored and managed.

**Attributes**

| Attribute      | Type        | Required | Constraints/Format | Notes                              |
|----------------|-------------|:--------:|--------------------|------------------------------------|
| id             | string/uuid |    ✓     | immutable          | Server-generated unique identifier |
| name           | string      |    ✓     | 1-120 chars        |                                    |
| addressStreet  | string      |    ✓     |                    |                                    |
| addressCity    | string      |    ✓     |                    |                                    |
| addressCountry | string      |    ✓     |                    |                                    |
| isActive       | boolean     |    ✓     |                    | Default: true                      |
| createdAt      | datetime    |    ✓     | ISO 8601           | Server-generated                   |
| updatedAt      | datetime    |    ✓     | ISO 8601           | Server-generated                   |

**Relationships**

- Many-to-Many with Product via the inventory association resource.

**Business Rules & Constraints**

- A location can be deactivated (`isActive: false`) but not deleted if it has historical inventory records.

---

## Endpoint Documentation (CRUD)

**Conventions**

- Base path: `/v1`
- Media type: `application/json; charset=utf-8`
- Timestamps: ISO 8601 (UTC)
- Pagination: `?page` & `?page_size` (default: page=1, page_size=25)
- Filtering: `?field=value` (e.g., `?categoryId=uuid`, `?supplierId=uuid`)
- Sorting: `?sort=field` (asc) or `?sort=-field` (desc) (e.g., `?sort=-createdAt`)

### Resource: Products

| Resource | Operation        | HTTP   | URI                                   | Request Body                           | Success Response                    | Error Responses                        |
|----------|------------------|--------|---------------------------------------|----------------------------------------|-------------------------------------|----------------------------------------|
| Products | Create           | POST   | `/v1/products`                        | `{ "sku": "A123", "name": "Phone", ... }` | **201 Created**; body + Location header | **400** Bad Request, **409** Conflict (SKU) |
| Products | Retrieve (list)  | GET    | `/v1/products?page=1&page_size=25`    | —                                      | **200 OK**; paginated list          | **400** Bad Request (invalid query)   |
| Product  | Retrieve (by id) | GET    | `/v1/products/{id}`                   | —                                      | **200 OK**; single product          | **404** Not Found                     |
| Product  | Update (full)    | PUT    | `/v1/products/{id}`                   | `{ "sku": "A123", "name": "Phone", ... }` | **200 OK**; updated product         | **400**, **404**, **409**             |
| Product  | Update (partial) | PATCH  | `/v1/products/{id}`                   | `{ "price": 299.99 }`                 | **200 OK**; updated product         | **400**, **404**, **409**             |
| Product  | Delete           | DELETE | `/v1/products/{id}`                   | —                                      | **204 No Content**                  | **404** Not Found                     |

### Resource: Categories

| Resource   | Operation        | HTTP   | URI                     | Request Body                      | Success Response            | Error Responses                                   |
|------------|------------------|--------|-------------------------|-----------------------------------|-----------------------------|---------------------------------------------------|
| Categories | Create           | POST   | `/v1/categories`        | `{ "name": "Electronics", ... }`  | **201 Created**             | **400** Bad Request                               |
| Categories | Retrieve (list)  | GET    | `/v1/categories`        | —                                 | **200 OK**; list            | —                                                 |
| Category   | Retrieve (by id) | GET    | `/v1/categories/{id}`   | —                                 | **200 OK**; single category | **404** Not Found                                 |
| Category   | Update (full)    | PUT    | `/v1/categories/{id}`   | `{ "name": "Electronics", ... }`  | **200 OK**; updated category | **400**, **404**                                 |
| Category   | Update (partial) | PATCH  | `/v1/categories/{id}`   | `{ "description": "New desc" }`   | **200 OK**; updated category | **400**, **404**                                 |
| Category   | Delete           | DELETE | `/v1/categories/{id}`   | —                                 | **204 No Content**          | **404** Not Found, **409** Conflict (has products) |

### Resource: Suppliers

| Resource  | Operation        | HTTP   | URI                    | Request Body                         | Success Response           | Error Responses                                   |
|-----------|------------------|--------|------------------------|--------------------------------------|----------------------------|---------------------------------------------------|
| Suppliers | Create           | POST   | `/v1/suppliers`        | `{ "name": "Supplier Co", ... }`     | **201 Created**            | **400** Bad Request                               |
| Suppliers | Retrieve (list)  | GET    | `/v1/suppliers`        | —                                    | **200 OK**; list           | —                                                 |
| Supplier  | Retrieve (by id) | GET    | `/v1/suppliers/{id}`   | —                                    | **200 OK**; single supplier | **404** Not Found                                 |
| Supplier  | Update (full)    | PUT    | `/v1/suppliers/{id}`   | `{ "name": "New Name", ... }`        | **200 OK**; updated supplier | **400**, **404**                                 |
| Supplier  | Update (partial) | PATCH  | `/v1/suppliers/{id}`   | `{ "contactEmail": "new@email.com" }` | **200 OK**; updated supplier | **400**, **404**                                 |
| Supplier  | Delete           | DELETE | `/v1/suppliers/{id}`   | —                                    | **204 No Content**         | **404** Not Found, **409** Conflict (has products) |

### Resource: Locations

| Resource  | Operation        | HTTP   | URI                    | Request Body                  | Success Response            | Error Responses                                  |
|-----------|------------------|--------|------------------------|-------------------------------|-----------------------------|-------------------------------------------------|
| Locations | Create           | POST   | `/v1/locations`        | `{ "name": "Lagos Store", ... }` | **201 Created**             | **400** Bad Request                              |
| Locations | Retrieve (list)  | GET    | `/v1/locations`        | —                             | **200 OK**; list            | —                                               |
| Location  | Retrieve (by id) | GET    | `/v1/locations/{id}`   | —                             | **200 OK**; single location | **404** Not Found                               |
| Location  | Update (full)    | PUT    | `/v1/locations/{id}`   | `{ "name": "New Name", ... }` | **200 OK**; updated location | **400**, **404**                               |
| Location  | Update (partial) | PATCH  | `/v1/locations/{id}`   | `{ "isActive": false }`       | **200 OK**; updated location | **400**, **404**                               |
| Location  | Delete           | DELETE | `/v1/locations/{id}`   | —                             | **204 No Content**          | **404** Not Found, **409** Conflict (has history) |

---

## Advanced Features

### Associations & Views

- **List products by category**: `GET /v1/categories/{categoryId}/products`
- **List products by supplier**: `GET /v1/suppliers/{supplierId}/products`
- **Get stock for a product**: `GET /v1/products/{productId}/inventory` (returns array of `{locationId, quantity, locationName}`)
- **Get stock at a location**: `GET /v1/locations/{locationId}/inventory` (returns array of `{productId, quantity, productName}`)
- **Update stock level**: `PUT /v1/products/{productId}/inventory/{locationId}` Body: `{ "quantity": 50 }`

### Bulk Operations

- **Bulk update inventory**: `POST /v1/inventory:batchUpdate`
  - Body: `{ "updates": [ {"productId": "uuid", "locationId": "uuid", "quantity": 100}, ... ] }`
  - Returns **207 Multi-Status** with outcomes for each update.

### Search & Filtering

- **Global product search**: `GET /v1/products/search?q=samsung` (searches name, description, sku)
- **Low stock alert filter**: `GET /v1/products?lowStock=true` (returns products where quantity <= lowStockThreshold at any location)
- **Filter products by category/supplier**: `GET /v1/products?categoryId=uuid&supplierId=uuid`

### Domain-Specific Operations

- **Stock Transfer**: `POST /v1/inventory/transfers`
  - Body: `{ "productId": "uuid", "fromLocationId": "uuid", "toLocationId": "uuid", "quantity": 10 }`
  - Atomically decreases stock at fromLocation and increases it at toLocation.
  - Returns **201 Created** on success. Errors with **400 Bad Request** if insufficient stock.

---

## Error Handling & Problem Details

**Error Response Envelope (RFC 7807-inspired)**

```json
{
  "type": "/problems/invalid-request",
  "title": "Your request parameters are invalid.",
  "status": 400,
  "detail": "The field 'sku' must be unique.",
  "instance": "/v1/products",
  "errors": [
    { "field": "sku", "message": "must be unique", "value": "A123" }
  ]
}
```

**Common HTTP Status Codes:**

- **400 Bad Request**: General validation failure (details in errors array).
- **404 Not Found**: Resource does not exist.
- **409 Conflict**: Violation of a business rule (e.g., duplicate SKU, deleting a resource in use).
- **422 Unprocessable Entity**: Semantic validation error (e.g., insufficient stock for a transfer).

---

## Pagination & Query Conventions

- **Strategy**: Page-based (`page` & `page_size`)
- **Default page_size**: 25
- **Max page_size**: 100

**Paginated List Response Wrapper**

```json
{
  "data": [ /* array of resource objects */ ],
  "pagination": {
    "page": 1,
    "page_size": 25,
    "total_records": 124,
    "total_pages": 5
  },
  "links": {
    "self": "/v1/products?page=1&page_size=25",
    "first": "/v1/products?page=1&page_size=25",
    "prev": null,
    "next": "/v1/products?page=2&page_size=25",
    "last": "/v1/products?page=5&page_size=25"
  }
}
```

---

## Design Rationale

- **Resource naming**: Plural kebab-case nouns (`/v1/products`). Clear and consistent.
- **Versioning**: Path-based versioning (`/v1`). Simple, explicit, and easy to debug.
- **Id strategy**: UUIDs for all resources. Opaque, globally unique, and avoids revealing business data or sequence.
- **Consistency rules**: All timestamps are ISO 8601 in UTC. All id fields are UUIDs. Soft delete via `isActive` flag is preferred over hard delete.
- **Security posture**: Assumes OAuth2 authentication will be implemented later. All endpoints require auth. No sensitive Personally Identifiable Informaton(PII) is stored in these core resources beyond business contact info.
- **Trade-offs**: Chose PUT for full updates and PATCH for partial for flexibility, understanding it adds complexity for clients. The inventory sub-resource pattern was chosen over a top-level resource for more intuitive URIs.

---

## Appendix A — Glossary

- **Resource**: An object with a type, associated data, and relationships to other resources. It is the fundamental concept in REST, accessible via a URI.
- **Collection**: A server-managed directory of resources. Clients can create new resources in a collection (e.g., POST /v1/products adds to the products collection).
- **Representation**: The current state of a resource, formatted in JSON for this API, delivered from server to client or vice versa.

---

## Appendix B — AI Usage Statement

I used an AI assistant to clarify general REST principles, status code meanings, and Markdown structuring. I did **not** use AI to generate InventoryPro's resource designs, endpoint specifications, or business domain analysis. All domain modeling, attribute decisions, relationship mapping, and endpoint design in this document are my own, based on the provided business context and assignment requirements. The AI was used as a tool for formatting (e.g. grammar, layout, etc.) and ensuring technical accuracy of standard conventions, not for creative design work.