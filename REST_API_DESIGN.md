# InventoryPro - REST API Design Document

## Executive Summary
This document outlines the RESTful API architecture for InventoryPro, an inventory management system designed for 
African retail businesses. The design prioritizes simplicity, intuitiveness, and scalability, adhering to REST 
conventions with resource-oriented endpoints, proper HTTP semantics, and consistent error handling. The API is 
structured to support core inventory operations for products, categories, suppliers, and locations, with careful 
consideration given to the specific needs of multi-location retail management, such as tracking stock levels 
across different stores.

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