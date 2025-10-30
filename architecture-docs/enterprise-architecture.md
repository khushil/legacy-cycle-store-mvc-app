# Enterprise Architecture

**Analysis Date**: 2025-10-30
**System**: Adventure Works Cycle Store MVC Application
**Codebase Size**: 19 C# files, 606 lines of code

## Executive Summary

The Adventure Works Cycle Store is a **MONOLITHIC** legacy ASP.NET MVC 4 application representing a single-domain e-commerce system for selling cycles and cycling accessories. The architecture follows a traditional three-tier pattern with a **SIMPLE** business domain centered around product catalog management.

**Architecture Style**: MONOLITHIC (CONFIRMED - single solution, no service boundaries detected)

## Current State Architecture

### Business Domain

**CONFIRMED**: Single Business Domain - **E-Commerce Product Catalog**

The application is organized around one primary business domain:

**Domain**: Product Catalog Management
- **Location**: `AdventureWorks.Business` namespace
- **File Path**: `AdventureWorksMVC/Business/`
- **Evidence**: All business logic confined to product and category management

### Bounded Context

**SINGLE BOUNDED CONTEXT IDENTIFIED**: Product Catalog Context

**Context Name**: Product Catalog
**Namespace**: `AdventureWorks.Business`
**Scope**: Product browsing, category management, product filtering

**Entities Within Context**:
1. `Product` - Core product entity (AdventureWorksMVC/Business/Product.cs:15)
2. `ProductCategory` - Top-level categories (AdventureWorksMVC/Business/ProductCategory.cs:15)
3. `ProductSubcategory` - Second-level categories (AdventureWorksMVC/Business/ProductSubcategory.cs:15)

**Context Boundaries**:
- **IN SCOPE**: Product catalog browsing, category navigation, product filtering
- **OUT OF SCOPE**: Shopping cart, checkout, payment processing, order management
- **UNKNOWN**: Whether shopping cart/order functionality exists in database but not in codebase

### Business Entities and Relationships

#### Core Business Entities (CONFIRMED)

**1. Product Entity**
**File**: `AdventureWorksMVC/Business/Product.cs`
**Properties** (CONFIRMED from line 17-41):
- `ProductID` (int, Primary Key)
- `Name` (string)
- `ProductNumber` (string)
- `Color`, `Size`, `Weight` (physical attributes)
- `StandardCost`, `ListPrice` (pricing)
- `ProductSubcategoryID` (foreign key)
- `SellStartDate`, `SellEndDate`, `DiscontinuedDate` (lifecycle dates)

**2. ProductCategory Entity**
**File**: `AdventureWorksMVC/Business/ProductCategory.cs`
**Properties** (CONFIRMED from line 22-25):
- `ProductCategoryID` (int, Primary Key)
- `Name` (string)
- `ProductSubcategories` (ICollection<ProductSubcategory>, navigation property)

**3. ProductSubcategory Entity**
**File**: `AdventureWorksMVC/Business/ProductSubcategory.cs`
**Properties** (CONFIRMED from line 17-23):
- `ProductSubcategoryID` (int, Primary Key)
- `ProductCategoryID` (int, Foreign Key)
- `Name` (string)
- `ProductCategory` (ProductCategory, navigation property)

#### Entity Relationships (CONFIRMED)

```
ProductCategory (1) ----< (Many) ProductSubcategory
                                  ^
                                  |
                                  | (Optional)
                                  |
Product (Many) -------------------+
```

**Evidence**:
- ProductCategory.ProductSubcategories (line 27, ProductCategory.cs) - Collection navigation
- ProductSubcategory.ProductCategory (line 23, ProductSubcategory.cs) - Reference navigation
- Product.ProductSubcategoryID (line 35, Product.cs) - Nullable foreign key

### Business Rules (DISCOVERED IN CODE)

#### Category Management Rules

**Rule 1**: All categories are retrieved without filtering
**Location**: `CategoryManager.GetMainCategories()` (CategoryManager.cs:19)
**Implementation**: Direct LINQ query with no business rules applied
```csharp
var cats = from cat in Common.DataEntities.ProductCategories select cat;
```

**Rule 2**: Category lookup is case-sensitive
**Location**: `CategoryManager.GetCategoryByName()` (CategoryManager.cs:31)
**Evidence**: Uses `==` comparison operator (line 34), no `.ToLower()` normalization

**Rule 3**: Subcategory lookup is case-sensitive
**Location**: `CategoryManager.GetProductSubcategoryByName()` (CategoryManager.cs:44)
**Evidence**: Uses `==` comparison operator (line 47), no `.ToLower()` normalization

#### Product Management Rules

**Rule 4**: Products can have optional colors, sizes, and weights
**Evidence**: Nullable properties in Product entity (Product.cs:22,27,30)
**Location**: Filter methods check for `!= null` (ProductManager.cs:87,103,138)

**Rule 5**: Products are filtered by subcategory, not category
**Location**: `ProductManager.GetProductByCategory()` (ProductManager.cs:46)
**Evidence**: Method accepts `ProductSubcategory` parameter, filters on `ProductSubcategoryID`

**Rule 6**: Product colors are distinct and sorted alphabetically
**Location**: `ProductManager.GetProductColor()` (ProductManager.cs:83)
**Evidence**: `.Distinct().ToList()` and `orderby p.Color` (line 88-89)

**Rule 7**: Product weights are distinct and sorted numerically
**Location**: `ProductManager.GetProductWeight()` (ProductManager.cs:98)
**Evidence**: `.Distinct().ToList()` and `orderby p.Weight` (line 105)

**Rule 8**: Product sizes are distinct and sorted
**Location**: `ProductManager.GetProductSize()` (ProductManager.cs:134)
**Evidence**: `.Distinct().ToList()` and `orderby p.Size` (line 139)

### System Actors

#### External Actors (CONFIRMED)

**1. Web User (End Customer)**
- **Role**: Anonymous or authenticated user browsing products
- **Authentication**: Forms-based authentication (Web.config:28-30)
- **Session Duration**: 30 days (43200 minutes, Web.config:29)
- **Evidence**: Forms authentication configuration at Web.config:28
- **Interaction**: HTTPS requests to MVC application

**2. System Administrator**
- **Role**: UNKNOWN (no admin functionality found in codebase)
- **Evidence**: NOT FOUND in current codebase scan

#### Internal Actors (CONFIRMED)

**1. SQL Membership Provider**
- **Type**: Authentication Service
- **Configuration**: Web.config:31-36
- **Provider**: SqlMembershipProvider
- **Database**: ASPNetDB connection string (referenced but NOT DEFINED in connectionStrings section)
- **Application Name**: C1AdventureWorks

### External Dependencies

#### Primary External System (CONFIRMED)

**1. CYCLE_STORE Database (SQL Server on AWS RDS)**
- **Type**: Data Store
- **Technology**: SQL Server
- **Location**: AWS RDS (sqlrdsdb.xxxxx.us-east-1.rds.amazonaws.com)
- **Connection String**: Web.config:21
- **Access Method**: Entity Framework 5 (EDMX, database-first)
- **Schema**: Production schema (inferred from entity names)
- **Status**: Credentials placeholder (user id=xxxx, password=xxxx)

#### Secondary Dependencies (DETECTED)

**2. ASPNetDB Database (Membership Database)**
- **Type**: Authentication Data Store
- **Connection String**: REFERENCED but NOT DEFINED in Web.config
- **Status**: MISCONFIGURATION - ConnectionStringName "ASPNetDB" used but not present
- **Evidence**: Web.config:34 references "ASPNetDB" connection string

#### External Services (SEARCHED - NONE FOUND)

- **SOAP/WCF Services**: NOT FOUND
- **REST API Integrations**: NOT FOUND
- **Email Services**: NOT FOUND (SMTP configuration absent)
- **Payment Gateways**: NOT FOUND
- **File Integrations**: NOT FOUND
- **Message Queuing**: NOT FOUND

### Business Capabilities

**CAPABILITY MAPPING** (based on codebase analysis):

#### Implemented Capabilities (CONFIRMED)

1. **Product Catalog Browsing**
   - Status: IMPLEMENTED
   - Components: CategoryManager, ProductManager
   - Evidence: CategoryManager.cs, ProductManager.cs

2. **Category Navigation**
   - Status: IMPLEMENTED
   - Components: CategoryManager.GetMainCategories()
   - Evidence: CategoryManager.cs:19, used in SiteLayoutController.cs:29,36

3. **Product Filtering by Attributes**
   - Status: IMPLEMENTED
   - Filters: Color, Size, Weight
   - Evidence: ProductManager methods at lines 83, 98, 134

4. **Product Search by Name**
   - Status: IMPLEMENTED
   - Component: ProductManager.GetProductByName()
   - Evidence: ProductManager.cs:33

5. **User Authentication**
   - Status: CONFIGURED (implementation not visible in codebase)
   - Method: Forms Authentication + SQL Membership
   - Evidence: Web.config:28-36

#### Missing Capabilities (NOT FOUND)

- **Shopping Cart**: NOT FOUND in codebase
- **Checkout Process**: NOT FOUND
- **Order Management**: NOT FOUND
- **Payment Processing**: NOT FOUND
- **Inventory Management**: NOT FOUND (Product entity has inventory fields but no management logic)
- **Product Administration**: NOT FOUND
- **User Registration**: NOT FOUND (despite membership provider configuration)
- **Email Notifications**: NOT FOUND

**Note**: Shopping cart mentioned in SiteLayoutModel.ShoppingCartItemsCount property (SiteLayoutModel.cs:12) but implementation NOT FOUND in analyzed code files.

### Domain Events

**DOMAIN EVENTS**: NOT IMPLEMENTED

**Evidence**: No event sourcing, no event handlers, no pub/sub patterns detected in codebase.

### Business Processes

**PROCESS ORCHESTRATION**: NOT FOUND

**Current Implementation**: Stateless request/response pattern only
**Evidence**: Controllers directly call static manager methods (SiteLayoutController.cs:29,36)

## Diagrams

**C4 Level 1 - System Context**: [See c4-level1-context.puml](./diagrams/c4-level1-context.puml)

## File References

### Primary Business Logic Files

1. **CategoryManager.cs** - Category and subcategory operations
   - Path: `AdventureWorksMVC/Business/CategoryManager.cs`
   - Lines: 53 lines total
   - Methods: 3 (GetMainCategories, GetCategoryByName, GetProductSubcategoryByName)

2. **ProductManager.cs** - Product operations and filtering
   - Path: `AdventureWorksMVC/Business/ProductManager.cs`
   - Lines: 160 lines total
   - Methods: 9 (GetCategory, GetProductByName, GetProductByCategory, etc.)
   - **WARNING**: Uses inconsistent namespace `EpicAdventureWorks` (line 7)

3. **Common.cs** - Data access factory
   - Path: `AdventureWorksMVC/Business/Common.cs`
   - Lines: 21 lines total
   - Pattern: Static factory returning new DbContext instances

### Entity Model Files

4. **Product.cs** - Product entity (auto-generated)
   - Path: `AdventureWorksMVC/Business/Product.cs`
   - Lines: 44 lines total
   - Generated: Entity Framework T4 template

5. **ProductCategory.cs** - Category entity (auto-generated)
   - Path: `AdventureWorksMVC/Business/ProductCategory.cs`
   - Lines: 30 lines total
   - Generated: Entity Framework T4 template

6. **ProductSubcategory.cs** - Subcategory entity (auto-generated)
   - Path: `AdventureWorksMVC/Business/ProductSubcategory.cs`
   - Lines: 26 lines total
   - Generated: Entity Framework T4 template

### Configuration Files

7. **Web.config** - Application configuration
   - Path: `AdventureWorksMVC/Web.config`
   - Authentication: Lines 28-30
   - Membership: Lines 31-36
   - Connection Strings: Line 21

8. **CycleModel.edmx** - Entity Framework model
   - Path: `AdventureWorksMVC/Business/CycleModel.edmx`
   - Size: 15KB
   - Type: Database-first EDMX model

## Architecture Assessment

### Strengths (OBSERVED)

1. **Simple Domain Model**: Easy to understand with clear entity relationships
2. **Separation of Concerns**: Business logic separated from presentation (controllers)
3. **Static Helper Pattern**: Predictable access pattern via static methods

### Constraints (OBSERVED)

1. **Single Bounded Context**: All domain logic in one namespace/project
2. **Stateless Operations**: No long-running business processes or workflows
3. **Read-Heavy**: Most operations are read queries (filtering, browsing)
4. **Limited Business Rules**: Minimal domain logic, mostly data retrieval

### Technical Debt Indicators (OBSERVED)

1. **Namespace Inconsistency**: ProductManager uses `EpicAdventureWorks` namespace (ProductManager.cs:7)
2. **Missing Connection String**: ASPNetDB referenced but not defined (Web.config:34)
3. **Incomplete Implementation**: Shopping cart property exists but no implementation
4. **Placeholder Credentials**: Database connection uses `xxxx` placeholders (Web.config:21)
5. **Mixed DbContext References**: ProductManager references both `Common.DataEntities.Products` and `Common.DataEntities.Product` (ProductManager.cs:35,62)

## Summary

**Domain Complexity**: LOW
**Bounded Contexts**: 1 (Product Catalog)
**Core Entities**: 3 (Product, ProductCategory, ProductSubcategory)
**Business Rules**: 8 explicit rules identified
**External Systems**: 1 (SQL Server database)
**Architecture Style**: Monolithic, Three-Tier

This is a **focused, single-domain application** with a simple product catalog management bounded context. The business logic is straightforward with minimal complexity, centered around browsing and filtering products by categories and attributes.
