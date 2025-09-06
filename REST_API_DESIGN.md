# InventoryPro - REST API Design Document

## Executive Summary
This document outlines the RESTful API architecture for InventoryPro, an inventory management system designed for 
African retail businesses. The design prioritizes simplicity, intuitiveness, and scalability, adhering to REST 
conventions with resource-oriented endpoints, proper HTTP semantics, and consistent error handling. The API is 
structured to support core inventory operations for products, categories, suppliers, and locations, with careful 
consideration given to the specific needs of multi-location retail management, such as tracking stock levels 
across different stores.

---
## 1. Business Domain Analysis
The core business domain of InventoryPro revolves around tracking inventory across multiple physical locations. 
The primary entities were identified by analyzing the key nouns and operations in a typical inventory workflow:

1. **Product**: The central entity. Represents a sellable item with a unique Stock Keeping Unit (SKU). Its 
attributes and stock levels are the primary data points.

2. **Category**: A logical grouping mechanism for products (e.g., "Electronics," "Food & Beverage," "Clothing," 
etc.). This enables organized browsing and reporting.

3. **Supplier**: The source from which products are procured. Critical for managing supply chains, reordering, and 
cost analysis.

4. **Location**: Represents a physical store or warehouse. This is a crucial differentiator, as stock levels are 
specific to each location, enabling businesses to manage inventory across a network of stores.

**Key Relationships**:

- A **Category** contains many **Products** (One-to-Many).

- A **Supplier** provides many **Products** (One-to-Many).

- A **Product** has a stock level at many **Locations** (Many-to-Many). This relationship is managed through a 
separate `inventory` resource.

**Critical Operations**: The API must support full CRUD (Create, Read, Update, Delete) for all entities. Beyond 
that, it must efficiently handle checking and updating stock levels, filtering products by category or supplier, 
and providing a consolidated view of inventory across all locations.

---
## 2. Resource Architecture Design
**Resource**: `Product`
- **Description**: A unique item available for sale across one or more locations.

- **Purpose**: To represent all sellable inventory items and their universal attributes.

- **Attributes**:
    - `id` (string, UUID) - Unique system identifier.
    - `sku` (string) - Unique stock-keeping unit, required.
    - `name` (string) - Human-readable product name, required.
    - `description` (string) - Detailed product information.
    - `price` (number) - Current selling price.
    - `cost` (number) - Procurement cost from the supplier.
    - `categoryId` (string, UUID) - Reference to the product's category.
    - `supplierId` (string, UUID) - Reference to the product's supplier.
    - `isActive` (boolean) - Status flag/soft delete.
    - `createdAt` (string, ISO timestamp) - Audit field.
    - `updatedAt` (string, ISO timestamp) - Audit field.

- **Relationships**:
    - Many-to-One with `Category` and `Supplier`.
    - Many-to-Many with `Location` (via the `inventory` sub-resource).

- **Constraints**: `sku` must be unique across the system.


**Resource**: `Category`
- **Description**: A logical group for organizing products (e.g., "Grains", "Mobile Phones", etc.).

- **Purpose**: To enable filtered product discovery and organized management.

- **Attributes**:
    - `id` (string, UUID) - Unique identifier.
    - `name` (string) - Category name, required, must be unique.
    - `description` (string) - Purpose of the category.
    - `createdAt` (string, ISO timestamp)
    - `updatedAt` (string, ISO timestamp)

- **Relationships**:
    - One-to-Many with `Product`.


**Resource**: `Supplier`
- **Description**: A company or entity that provides products to the business.

- **Purpose**: To manage vendor information and relationships.

- **Attributes**:
    - `id` (string, UUID) - Unique identifier.
    - `name` (string) - Company name, required.
    - `contactEmail` (string) - Primary contact email.
    - `contactPhone` (string) - Primary contact phone number.
    - `address` (object) - { street, city, country }.
    - `createdAt` (string, ISO timestamp)
    - `updatedAt` (string, ISO timestamp)

- **Relationships**:
    - One-to-Many with `Product`.


**Resource**: `Location`
- **Description**: A physical store or warehouse where inventory is held.

- **Purpose**: To manage inventory levels across a business's physical presence.

- **Attributes**:
    - `id` (string, UUID) - Unique identifier.
    - `name` (string) - Location name (e.g., "Lagos Main Store"), required.
    - `address` (object) - { street, city, country }, required.
    - `isActive` (boolean) - Whether the location is operational.
    - `createdAt` (string, ISO timestamp)
    - `updatedAt` (string, ISO timestamp)

- **Relationships**:
    - Many-to-Many with `Product` (via the `inventory` sub-resource).

---
## 3. Complete Endpoint Specification
**Products Endpoints**


|    Resource    |    Operation    |  HTTP Method  |     URI     |  Request Body  |        Success Response          |       Error Response      |
|:--------------:|:---------------:|:-------------:|:-----------:|:--------------:|:--------------------------------:|:-------------------------:|
|   Products     |  List Products  |     `GET`     | `/products` |        -       |`200 OK`(Array of Product objects)|`500 Internal Server Error`|
|                |  Greate Product |     `POST`    | `/products` |Product object (without id)|`201 Created` (Created Product object)|`400 Bad Request` (Validation error), `409 Conflict` (SKU exists)|
|    Product     |   Get Product   |      `GET`    |`/products/{productId}`|       -      | `200 OK` (Product object) |   `404 Not Found`        |
|                |  Update Product |      `PUT`    |`/products/{productId}`|Product object (full update)|`200 OK` (Updated Product object)|`400 Bad Request`, `404 Not Found`|
|                |  Delete Product |    `DELETE`   |`/products/{productId}`|       -      |      `204 No Content`     |      `404 Not Found`     |


**Query Parameters for** `GET /products`:

- `categoryId`: Filter products by category.
- `supplierId`: Filter products by supplier.
- `page`, `limit`: For pagination.


**Categories Endpoints**


|    Resource    |    Operation    |  HTTP Method  |     URI     |  Request Body  |        Success Response          |       Error Response      |
|:--------------:|:---------------:|:-------------:|:-----------:|:--------------:|:--------------------------------:|:-------------------------:|
|   Categories   | List Categories |     `GET`     |`/categories`|        -       |`200 OK`(Array of Category objects)|`500 Internal Server Error`|
|                | Greate Category |     `POST`    |`/categories`|Category object (without id)|`201 Created` (Created Category object)|`400 Bad Request`|
|   Category     |  Get Category   |    `GET`    |`/categories/{categoryId}`|       -      | `200 OK` (Category object) |  `404 Not Found`        |
|                | Update Category |     `PUT`   |`/categories/{categoryId}`|Category object (full update)|`200 OK` (Updated Category object)|`400 Bad Request`, `404 Not Found`|
|                | Delete Category |    `DELETE` |`/categories/{categoryId}`|       -      |    `204 No Content`   |    `404 Not Found`, `409 Conflict` (if category has products)   |


**Suppliers Endpoints**


|    Resource    |    Operation    |  HTTP Method  |     URI     |  Request Body  |        Success Response          |       Error Response      |
|:--------------:|:---------------:|:-------------:|:-----------:|:--------------:|:--------------------------------:|:-------------------------:|
|  Suppliers     | List Suppliers  |     `GET`     |`/suppliers` |        -       |`200 OK`(Array of Supplier objects)|`500 Internal Server Error`|
|                | Greate Supplier |     `POST`    |`/suppliers` |Supplier object (without id)|`201 Created` (Created Supplier object)|`400 Bad Request` (Validation error)|
|   Supplier     |   Get Supplier  |      `GET`    |`/suppliers/{supplierId}`|       -      | `200 OK` (Supplier object) |  `404 Not Found`    |
|                | Update Supplier |      `PUT`    |`/suppliers/{supplierId}`|Supplier object (full update)|`200 OK` (Updated Supplier object)|`400 Bad Request`, `404 Not Found`|
|                | Delete Supplier |    `DELETE`   |`/suppliers/{supplierId}`|       -      |      `204 No Content`     |      `404 Not Found`, `409 Conflict` (if supplier has associated products)     |

**Query Parameters for** `GET /suppliers`:

- `page`, `limit`: For pagination.


**Inventory Sub-Resource Endpoints (Linking Products & Locations)**


|    Resource    |    Operation    |  HTTP Method  |     URI     |  Request Body  |        Success Response          |       Error Response      |
|:--------------:|:---------------:|:-------------:|:-----------:|:--------------:|:--------------------------------:|:-------------------------:|
|Location Inventory|Get Stock Level|     `GET`     |`/products/{productId}/inventory/{locationId}`|        -       |`200 OK``{ productId, locationId, quantity }`|`404 Not Found`|
|                |Update Stock Level|     `PUT`    |`/products/{productId}/inventory/{locationId}`|`{ quantity: 66 }`|`200 OK` (Updated inventory object)|`400 Bad Request`, `404 Not Found`|
|Product Inventory|List All Stock|   `GET`   |`/products/{productId}/inventory`|       -      | `200 OK` (Array of {locationId, quantity, locationName}) |   `404 Not Found`(if product not found)   |
| Location Stock |List All Products|      `GET`    |`/locations/{locationId}/inventory`|   -   |`200 OK` (Array of `{productId, quantity, productName}`)|`404 Not Found` (if location not found)|

---
## 4. Advanced API Features
**1. Association Endpoints**
- `GET /categories/{categoryId}/products`: Retrieves all products belonging to a specific category. Supports pagination and filtering.

- `GET /suppliers/{supplierId}/products`: Retrieves all products provided by a specific supplier.

**2. Bulk Operations**
- `POST /inventory/bulk-update`: Allows updating stock levels for multiple product-location pairs in a single atomic request to prevent race conditions.
    - **Request Body**: `[ { productId, locationId, quantity }, ... ]`
    - **Response**: `207 Multi-Status` with individual statuses for each update.

**3. Search & Filtering**
- `GET /products/search?q={query}`: A global product search endpoint that searches through product `name`, `description`, and `sku` fields.

- `GET /products?lowStock=true`: A special filter parameter on the list products endpoint to only return products with a stock level below a defined threshold at any location. This is crucial for restocking alerts.

**4. Domain-Specific Operation**
- `POST /inventory/transfer`: Facilitates transferring stock between two locations (e.g., from a central warehouse to a store).
    - **Request Body**: `{ productId, fromLocationId, toLocationId, quantity }`
    - **Logic**: Atomically decreases stock at `fromLocationId` and increases it at `toLocationId`.
    - **Error**: `400 Bad Request` if `fromLocation` has insufficient stock.

---
## 5. Design Rationale
1. **Resource Modeling**: The `inventory` is modeled as a sub-resource of a `Product` because a stock level has no meaning without its associated 
product and location. This leads to intuitive URIs like `/products/{id}/inventory`.

2. **HTTP Methods**: Strict adherence to REST semantics: `POST` for creation, `GET` for retrieval, `PUT` for full updates, and `DELETE` for 
removal. `PATCH` was avoided for simplicity in this initial version.

3. **Status Codes**: Meaningful codes are used: `201 Created` upon successful creation, `204 No Content` for successful deletions (no content to 
return), and `409 Conflict` for unique constraint violations, providing clear signals to the client. `409 Conflict` is a critical business rule. 
For example, a supplier cannot be deleted if there are still products associated with it in the system. This prevents orphaned records and 
maintains referential integrity. The API should respond with a clear error message in the response body, e.g., `{"error": "Cannot delete supplier; 
one or more products are still associated."}`

4. **Multi-Location Focus**: The entire design is built around the core requirement of managing inventory across multiple `Location` resources, 
which is a common and critical need for growing African retail businesses.

5. **Filtering & Search**: The advanced features like `lowStock=true` and the dedicated search endpoint are designed directly from user stories,
ensuring the API solves real business problems like identifying items for restocking or quickly finding products.