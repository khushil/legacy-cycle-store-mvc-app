# Data Architecture

**Analysis Date**: 2025-10-30
**Database**: CYCLE_STORE (SQL Server on AWS RDS)
**ORM**: Entity Framework 5.0.0 (Database-First with EDMX)

## Executive Summary

The Adventure Works Cycle Store uses a **DATABASE-FIRST** approach with Entity Framework 5 and EDMX models to interact with a SQL Server database hosted on AWS RDS. The data model is **SIMPLE** with only **THREE ENTITIES** visible in the codebase: Product, ProductCategory, and ProductSubcategory. The database likely contains additional tables (membership, descriptions, models) that are not fully exposed in the current EDMX model.

**Data Access Pattern**: New DbContext instance per operation via static factory (ANTI-PATTERN)

## Current State Architecture

### Database Overview

**Database Name**: CYCLE_STORE

**Location**: AWS RDS
**Host**: sqlrdsdb.xxxxx.us-east-1.rds.amazonaws.com (placeholder)
**Technology**: SQL Server (version UNKNOWN)
**Schema**: Production (inferred from entity names)

**Connection String** (Web.config:21):
```
metadata=res://*/Business.CycleModel.csdl|res://*/Business.CycleModel.ssdl|res://*/Business.CycleModel.msl;
provider=System.Data.SqlClient;
provider connection string="
  data source=sqlrdsdb.xxxxx.us-east-1.rds.amazonaws.com;
  initial catalog=CYCLE_STORE;
  user id=xxxx;
  password=xxxx;
  MultipleActiveResultSets=True;
  App=EntityFramework"
```

**Connection Status**: PLACEHOLDER CREDENTIALS (user id and password are xxxx)

### Entity Framework Configuration

**ORM**: Entity Framework 5.0.0
**Approach**: Database-First with EDMX

**EDMX Model**:
- **File**: CycleModel.edmx
- **Location**: AdventureWorksMVC/Business/CycleModel.edmx
- **Size**: 15 KB
- **Type**: XML-based model (EDMX)

**EDMX Components**:
1. **CSDL** (Conceptual Schema Definition Language) - Conceptual model
2. **SSDL** (Store Schema Definition Language) - Storage model (database schema)
3. **MSL** (Mapping Specification Language) - Mapping between conceptual and storage

**T4 Templates** (Code Generation):
1. **CycleModel.Context.tt** - Generates DbContext class
   - Output: CycleModel.Context.cs
   - Generated: CYCLE_STOREEntities class

2. **CycleModel.tt** - Generates entity classes
   - Output: Product.cs, ProductCategory.cs, ProductSubcategory.cs
   - Generated: POCO entity classes

### DbContext Organization

**DbContext Class**: CYCLE_STOREEntities

**File**: Business/CycleModel.Context.cs
**Namespace**: AdventureWorks.Business
**Base Class**: System.Data.Entity.DbContext

**Constructor** (CycleModel.Context.cs:18-20):
```csharp
public CYCLE_STOREEntities()
    : base("name=CYCLE_STOREEntities")
{
}
```

**DbSets** (CONFIRMED):

1. **Products** (line 28)
   - Type: DbSet<Product>
   - Table: Product (inferred)
   - Entity Count: UNKNOWN

2. **ProductCategories** (line 29)
   - Type: DbSet<ProductCategory>
   - Table: ProductCategory (inferred)
   - Entity Count: UNKNOWN

3. **ProductSubcategories** (line 30)
   - Type: DbSet<ProductSubcategory>
   - Table: ProductSubcategory (inferred)
   - Entity Count: UNKNOWN

**Additional Tables Referenced in Code**:
4. **ProductDescription** (referenced in ProductManager.cs:152)
   - NOT in DbSets
   - Accessed via Common.DataEntities.ProductDescription
   - **INCONSISTENCY**: Used but not defined in visible DbContext

**Code First Protection** (CycleModel.Context.cs:23-26):
```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    throw new UnintentionalCodeFirstException();
}
```
Purpose: Prevents accidental Code-First migrations

### Entity Models and Configurations

#### 1. Product Entity

**File**: Business/Product.cs
**Namespace**: AdventureWorks.Business
**Type**: Partial class (allows extension)
**Generated**: Yes (Auto-generated from EDMX)

**Properties** (25 total):

**Identity**:
- ProductID: int (Primary Key)
- ProductNumber: string

**Basic Information**:
- Name: string
- MakeFlag: bool
- FinishedGoodsFlag: bool

**Physical Attributes** (Nullable):
- Color: string (nullable)
- Size: string (nullable)
- Weight: decimal? (nullable)
- SizeUnitMeasureCode: string (nullable)
- WeightUnitMeasureCode: string (nullable)

**Inventory**:
- SafetyStockLevel: short
- ReorderPoint: short

**Pricing**:
- StandardCost: decimal
- ListPrice: decimal

**Manufacturing**:
- DaysToManufacture: int

**Classification** (Nullable):
- ProductLine: string (nullable)
- Class: string (nullable)
- Style: string (nullable)

**Relationships**:
- ProductSubcategoryID: int? (Foreign Key, nullable)
- ProductModelID: int? (Foreign Key, nullable)

**Lifecycle**:
- SellStartDate: DateTime
- SellEndDate: DateTime? (nullable)
- DiscontinuedDate: DateTime? (nullable)

**Metadata**:
- rowguid: Guid (SQL Server uniqueidentifier)
- ModifiedDate: DateTime

**Navigation Properties**: NONE in current entity
**Note**: ProductSubcategoryID exists but no navigation property defined

#### 2. ProductCategory Entity

**File**: Business/ProductCategory.cs
**Namespace**: AdventureWorks.Business
**Type**: Partial class
**Generated**: Yes (Auto-generated from EDMX)

**Properties** (4 total):

**Identity**:
- ProductCategoryID: int (Primary Key)

**Data**:
- Name: string

**Metadata**:
- rowguid: Guid
- ModifiedDate: DateTime

**Navigation Properties** (ProductCategory.cs:27):
- ProductSubcategories: ICollection<ProductSubcategory>

**Constructor** (ProductCategory.cs:17-20):
```csharp
public ProductCategory()
{
    this.ProductSubcategories = new HashSet<ProductSubcategory>();
}
```
Purpose: Initializes navigation collection to prevent null reference

**Relationship**: One-to-Many with ProductSubcategory

#### 3. ProductSubcategory Entity

**File**: Business/ProductSubcategory.cs
**Namespace**: AdventureWorks.Business
**Type**: Partial class
**Generated**: Yes (Auto-generated from EDMX)

**Properties** (5 total):

**Identity**:
- ProductSubcategoryID: int (Primary Key)

**Relationship**:
- ProductCategoryID: int (Foreign Key)

**Data**:
- Name: string

**Metadata**:
- rowguid: Guid
- ModifiedDate: DateTime

**Navigation Properties** (ProductSubcategory.cs:23):
- ProductCategory: ProductCategory (Reference to parent)

**Relationship**: Many-to-One with ProductCategory

### Database Schema Relationships

**CONFIRMED RELATIONSHIPS**:

```
ProductCategory (1) ----< (Many) ProductSubcategory
                                  ^
                                  |
                                  | (Optional)
                                  |
Product (Many) -------------------+
```

**Relationship Details**:

1. **ProductCategory → ProductSubcategory**
   - Type: One-to-Many
   - Parent: ProductCategory
   - Child: ProductSubcategory
   - Foreign Key: ProductSubcategory.ProductCategoryID
   - Navigation: ProductCategory.ProductSubcategories (collection)
   - Reverse Navigation: ProductSubcategory.ProductCategory (reference)
   - Delete Cascade: UNKNOWN (not visible in entity classes)

2. **ProductSubcategory → Product**
   - Type: One-to-Many (Optional)
   - Parent: ProductSubcategory
   - Child: Product
   - Foreign Key: Product.ProductSubcategoryID (nullable)
   - Navigation: NONE (no navigation property on ProductSubcategory)
   - Reverse Navigation: NONE (no navigation property on Product)
   - **ISSUE**: Relationship exists but navigation properties missing

3. **Product → ProductModel** (Inferred)
   - Type: Many-to-One (Optional)
   - Foreign Key: Product.ProductModelID (nullable)
   - Entity: NOT in current EDMX model
   - Status: Referenced but not modeled

### Migration History

**MIGRATIONS**: NOT APPLICABLE

**Reason**: Database-First approach with EDMX does not use Code-First migrations

**Database Changes**:
- Changes made directly to database
- EDMX updated from database ("Update Model from Database")
- T4 templates regenerate entity classes
- NO migration files or history table

**Evidence**:
- NO Migrations folder in project
- NO __MigrationHistory table
- UnintentionalCodeFirstException thrown to prevent Code-First (CycleModel.Context.cs:25)

### Stored Procedures and Raw SQL

**STORED PROCEDURES**: NOT FOUND

**Evidence**:
- NO stored procedure calls in manager classes
- NO FromSqlRaw or ExecuteSqlCommand usage
- All database access via LINQ to Entities

**RAW SQL**: NOT FOUND

**Search Results**:
- NO SqlCommand usage
- NO ADO.NET direct access
- NO database.ExecuteSqlCommand()
- NO FromSqlRaw() calls

**Query Method**: 100% LINQ to Entities

**Example** (CategoryManager.cs:21-23):
```csharp
var cats = from cat in Common.DataEntities.ProductCategories
           select cat;
return cats.ToList();
```

### Data Access Patterns

**PRIMARY PATTERN**: Static Factory with Direct DbContext Access

#### Pattern: Static Factory

**Implementation**: Common.DataEntities

**File**: Business/Common.cs
**Method**: Property getter (lines 13-18)

```csharp
public static CYCLE_STOREEntities DataEntities
{
    get
    {
        return new CYCLE_STOREEntities();
    }
}
```

**Usage Examples**:

1. **CategoryManager** (CategoryManager.cs:21):
```csharp
var cats = from cat in Common.DataEntities.ProductCategories select cat;
```

2. **ProductManager** (ProductManager.cs:35):
```csharp
var product = from p in Common.DataEntities.Products
              where p.Name == name
              select p;
```

**Assessment**: ANTI-PATTERN

**Issues**:
1. **New Context Per Call**: Creates new DbContext on every access
2. **No Connection Pooling**: Cannot reuse connections efficiently
3. **No Transaction Support**: Each operation in separate context
4. **Memory Overhead**: Unnecessary context creation
5. **No Unit of Work**: Cannot batch changes
6. **Difficult to Test**: Static dependency

#### Query Patterns Observed

**1. Select All Pattern**
- Implementation: CategoryManager.GetMainCategories() (line 19)
- Example: `from cat in Common.DataEntities.ProductCategories select cat`
- Returns: All records as List<T>

**2. Filter by Name Pattern**
- Implementation: CategoryManager.GetCategoryByName() (line 31)
- Example: `where cat.Name == name`
- Returns: FirstOrDefault()
- **Issue**: Case-sensitive comparison

**3. Filter by Subcategory Pattern**
- Implementation: ProductManager.GetProductByCategory() (line 46)
- Example: `where p.ProductSubcategory.ProductSubcategoryID == category.ProductSubcategoryID`
- Returns: IQueryable<Product> (deferred execution)

**4. Distinct Value Pattern**
- Implementation: ProductManager.GetProductColor() (line 83)
- Example: `select p.Color).Distinct().ToList()`
- Returns: List of unique values, sorted
- Used for: Filter options (Color, Size, Weight)

**5. Null Filtering Pattern**
- Implementation: ProductManager.GetProductColor() (line 87)
- Example: `where p.Color != null`
- Purpose: Exclude products without values

**6. Single Entity Pattern**
- Implementation: ProductManager.GetProductByProductId() (line 60)
- Example: `where p.ProductID == prodId`
- Returns: FirstOrDefault()

#### Inconsistencies in Data Access

**INCONSISTENCY 1**: DbSet Name Mismatch

**Evidence** (ProductManager.cs):
- Line 35: Uses `Common.DataEntities.Products` (correct, plural)
- Line 62: Uses `Common.DataEntities.Product` (incorrect, singular)
- Line 86: Uses `Common.DataEntities.Product` (incorrect, singular)

**Actual DbSet Name**: `Products` (CycleModel.Context.cs:28)

**Impact**: Code will fail at runtime if line 62 is executed

**INCONSISTENCY 2**: Additional DbSet Reference

**Evidence** (ProductManager.cs:152):
```csharp
var descList = from pd in Common.DataEntities.ProductDescription
               where pd.ProductDescriptionID == descId
               select pd.Description;
```

**Status**: ProductDescription NOT in DbSets (CycleModel.Context.cs:28-30)

**Possible Explanations**:
1. EDMX model incomplete (not all tables imported)
2. Code written before EDMX update
3. ProductDescription manually added to DbContext but not visible

**INCONSISTENCY 3**: Overloaded Method with Different Pattern

**Method**: ProductManager.GetProductByProductId() (two overloads)

**Overload 1** (line 60): Uses Common.DataEntities (static factory)
**Overload 2** (line 68): Accepts Entities parameter (different context)

**Impact**: Suggests multiple data access patterns coexist

### Database Tables (Inferred)

**TABLES VISIBLE IN EDMX** (Confirmed):
1. Product
2. ProductCategory
3. ProductSubcategory

**TABLES REFERENCED IN CODE** (Likely exist):
4. ProductDescription (ProductManager.cs:152)
5. ProductModel (Product.ProductModelID foreign key)

**TABLES FOR MEMBERSHIP** (Configured but not in EDMX):
6. aspnet_Users
7. aspnet_Membership
8. aspnet_Applications
(Standard ASP.NET Membership schema)

**ESTIMATED TOTAL TABLES**: 8-15 tables (based on typical Adventure Works schema)

**UNKNOWN TABLES** (May exist in full Adventure Works):
- SalesOrderHeader
- SalesOrderDetail
- Customer
- Address
- ShoppingCartItem
- ProductReview
- ProductPhoto
- Vendor
- PurchaseOrder

### Data Validation and Constraints

**ENTITY VALIDATION**: NOT IMPLEMENTED

**Evidence**:
- NO DataAnnotations attributes on entities
- NO [Required], [MaxLength], [Range] attributes
- NO IValidatableObject implementations

**DATABASE CONSTRAINTS**: UNKNOWN

**Likely Constraints** (Standard SQL Server):
- Primary Key constraints on *ID fields
- Foreign Key constraints (ProductSubcategory.ProductCategoryID)
- Unique constraints (rowguid likely unique)
- NOT NULL constraints (non-nullable properties)

**Cannot Confirm Without Database Access**

### Lazy Loading vs Eager Loading

**LAZY LOADING**: ENABLED (Default in EF 5)

**Evidence**:
- Navigation properties declared as `virtual` would enable lazy loading
- Current entities do NOT use `virtual` keyword
- Navigation property: `public virtual ICollection<ProductSubcategory>` NOT FOUND
- Actual: `public ICollection<ProductSubcategory>` (ProductCategory.cs:27)

**Assessment**: Lazy loading DISABLED (navigation properties not virtual)

**EAGER LOADING**: NOT USED

**Evidence**:
- NO `.Include()` calls in manager methods
- NO `var cats = context.ProductCategories.Include(c => c.ProductSubcategories)`

**Current Behavior**: Only requested entities loaded, no automatic navigation

### Connection Management

**CONNECTION STRING**: CYCLE_STOREEntities

**Provider**: System.Data.SqlClient (SQL Server)
**Connection Type**: Entity Framework EntityClient provider
**MARS**: Enabled (MultipleActiveResultSets=True)

**Connection Pooling**:
- **Enabled**: Yes (ADO.NET default)
- **Min Pool Size**: Not specified (default: 0)
- **Max Pool Size**: Not specified (default: 100)

**Connection Lifetime**:
- **Pattern**: New DbContext per operation
- **Duration**: Short-lived (request scope)
- **Disposal**: Automatic (garbage collection)
- **Issue**: No explicit using statements for DbContext

**Best Practice Violation**:
Should wrap DbContext in `using` statement:
```csharp
using (var context = new CYCLE_STOREEntities())
{
    // operations
}
```

Current: No disposal management

### Indexing Strategy

**INDEXES**: UNKNOWN

**Cannot Determine From Code**:
- Index definitions in database, not visible in EDMX
- Likely indexes on:
  - Primary Keys (automatic)
  - Foreign Keys (ProductSubcategoryID, ProductCategoryID)
  - Name columns (for searching)
  - rowguid (unique identifier)

**Query Performance Indicators**:
- Name lookups (CategoryManager.cs:34) - Would benefit from index on Name
- SubcategoryID filters (ProductManager.cs:50) - Would benefit from index on ProductSubcategoryID

### Data Seeding

**SEED DATA**: NOT FOUND

**Evidence**:
- NO Seed() method in DbContext
- NO data seeding configuration
- NO SQL scripts in project

**Assumption**: Database contains existing data from Adventure Works sample database

## Diagrams

**Entity Relationship Diagram**: [See entity-relationship.mermaid](./diagrams/entity-relationship.mermaid)
**Data Flow Diagram**: [See data-flow.mermaid](./diagrams/data-flow.mermaid)

## Data Architecture Assessment

### Strengths (OBSERVED)

1. **Simple Model**: Easy to understand with 3 core entities
2. **LINQ Queries**: SQL injection protection via parameterized queries
3. **Partial Classes**: Allows entity extension without modifying generated code
4. **MARS Enabled**: Supports multiple active result sets on single connection

### Weaknesses (OBSERVED)

1. **DbContext Anti-Pattern**: New context per operation
2. **No Disposal Management**: DbContext not properly disposed
3. **Static Dependencies**: Difficult to test and maintain
4. **Missing Navigation Properties**: Product entity lacks relationships
5. **Inconsistent DbSet Names**: Plural/singular mismatch in code
6. **No Eager Loading**: Potential N+1 query issues
7. **No Caching**: Database queried on every request
8. **Incomplete EDMX**: ProductDescription referenced but not in DbSets

### Data Quality Issues (DETECTED)

1. **Placeholder Credentials**: Connection string has xxxx for user/password
2. **Missing Connection String**: ASPNetDB not defined
3. **Mixed Entity References**: Products vs Product usage
4. **Case-Sensitive Queries**: Name lookups are case-sensitive

### Performance Concerns

**CURRENT IMPACT**:
- ❌ New DbContext per operation (overhead)
- ❌ No query result caching
- ❌ No compiled queries
- ❌ No eager loading (potential N+1)
- ❌ ToList() on all queries (no paging)

**POTENTIAL ISSUES**:
- High database query volume (no caching)
- Connection pool churn (new context each time)
- Memory pressure (loading full result sets)

### Security Assessment

**SQL INJECTION**: PROTECTED
- All queries use LINQ to Entities (parameterized)
- NO string concatenation in queries
- NO raw SQL commands

**DATA ACCESS CONTROL**: NOT IMPLEMENTED
- NO row-level security
- NO entity-level permissions
- ALL data accessible to authenticated users

## Summary

**Data Model Complexity**: SIMPLE - 3 entities, 2 relationships
**ORM**: Entity Framework 5 (Database-First EDMX)
**Data Access Pattern**: Static Factory (ANTI-PATTERN)
**Query Method**: 100% LINQ to Entities
**Migrations**: NOT APPLICABLE (Database-First)
**Stored Procedures**: NONE
**Lazy Loading**: DISABLED
**Eager Loading**: NOT USED
**Connection Management**: POOR (no disposal, new context per operation)
**Caching**: NONE
**Performance**: CONCERNS (no optimization)

The data architecture is **FUNCTIONAL BUT INEFFICIENT** with significant room for optimization in data access patterns, connection management, and query strategies.
