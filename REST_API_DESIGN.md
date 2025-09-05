# InventoryPro - REST API Design Document

## Executive Summary
This document outlines the RESTful API architecture for InventoryPro, an inventory management system designed for 
African retail businesses. The design prioritizes simplicity, intuitiveness, and scalability, adhering to REST 
conventions with resource-oriented endpoints, proper HTTP semantics, and consistent error handling. The API is 
structured to support core inventory operations for products, categories, suppliers, and locations, with careful 
consideration given to the specific needs of multi-location retail management, such as tracking stock levels 
across different stores.

### Task 1: Business Domain Analysis
The core business domain of InventoryPro revolves around tracking inventory across multiple physical locations. 
The primary entities were identified by analyzing the key nouns and operations in a typical inventory workflow:

1. `Product`: The central entity. Represents a sellable item with a unique Stock Keeping Unit (SKU). Its 
attributes and stock levels are the primary data points.

2. `Category`: A logical grouping mechanism for products (e.g., "Electronics," "Food & Beverage," "Clothing," 
etc.). This enables organized browsing and reporting.

3. `Supplier`: The source from which products are procured. Critical for managing supply chains, reordering, and cost analysis.

4. `Location`: Represents a physical store or warehouse. This is a crucial differentiator, as stock levels are 
specific to each location, enabling businesses to manage inventory across a network of stores.

Key Relationships:

- A Category contains many Products (One-to-Many).

- A Supplier provides many Products (One-to-Many).

- A Product has a stock level at many Locations (Many-to-Many). This relationship is managed through a separate 
`inventory` resource.

Critical Operations: The API must support full CRUD (Create, Read, Update, Delete) for all entities. Beyond that, 
it must efficiently handle checking and updating stock levels, filtering products by category or supplier, and 
providing a consolidated view of inventory across all locations.