# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a legacy ASP.NET MVC 4 application for a cycle store (Adventure Works), built with .NET Framework 4.5. The application uses Entity Framework 5 with a database-first approach to interact with a SQL Server database named `CYCLE_STORE`.

## Build and Run

**Prerequisites:**
- Visual Studio 2013 or later with ASP.NET development tools
- .NET Framework 4.5
- IIS Express (configured on port 55185)
- SQL Server connection (currently configured for AWS RDS)

**Build:**
```bash
# Using MSBuild (from Visual Studio Developer Command Prompt)
msbuild AdventureWorksMVC_2013.sln /p:Configuration=Debug
msbuild AdventureWorksMVC_2013.sln /p:Configuration=Release
```

**Run:**
The application is configured to run with IIS Express on `http://localhost:55185/`. Open the solution in Visual Studio and press F5 to run.

**Publish:**
The application has a publish profile at `AdventureWorksMVC/Properties/PublishProfiles/AdventureWorks-MVC.pubxml` configured for MSDeploy to IIS.

## Architecture

### Database Layer

- **Entity Framework Model:** `AdventureWorksMVC/Business/CycleModel.edmx`
  - Database-first EDMX model mapping to `CYCLE_STORE` database
  - Main entities: `Product`, `ProductCategory`, `ProductSubcategory`
  - Schema: Production schema with relationships between categories, subcategories, and products
  - Context class: `CYCLE_STOREEntities`

- **Common Data Access:** `AdventureWorksMVC/Business/Common.cs`
  - Provides `DataEntities` property that returns a new instance of `CYCLE_STOREEntities`
  - Used throughout the application for database access

### Business Logic Layer

Located in `AdventureWorksMVC/Business/`:

- **CategoryManager.cs:** Handles product category and subcategory operations
  - `GetMainCategories()` - Retrieves all top-level categories
  - `GetCategoryByName()` - Finds category by name
  - `GetProductSubcategoryByName()` - Finds subcategory by name

- **ProductManager.cs:** Handles product-related operations
  - Note: Uses namespace `EpicAdventureWorks` (inconsistent with rest of app)
  - `GetProductByName()` - Product lookup
  - `GetProductByCategory()` - Products by subcategory
  - `GetProductByProductId()` - Product by ID (has two overloads)
  - `GetProductColor()`, `GetProductWeight()`, `GetProductSize()` - Filter helpers

### Presentation Layer

**Controllers** (`AdventureWorksMVC/Controllers/`):
- **HomeController.cs:** Main landing page (`Default` action)
- **SiteLayoutController.cs:** Layout components
- **ErrorController.cs:** Error handling

**Views** (`AdventureWorksMVC/Views/`):
- Razor views with `.cshtml` extension
- Shared layout: `Views/Shared/_SiteLayout.cshtml`
- Home view: `Views/Home/Default.cshtml`
- Error view: `Views/Error/Default.cshtml`

**Routing:**
- Default route: `{controller}/{action}/{id}`
- Default controller: `Home`, Default action: `Default`

### Configuration

**Web.config:**
- Connection string: `CYCLE_STOREEntities` (Entity Framework connection to AWS RDS SQL Server)
- Forms authentication enabled with 30-day timeout
- Custom error pages for 404 and 500 errors
- SQL membership provider configured for user management

**Dependencies:**
- ASP.NET MVC 4
- Entity Framework 5
- ASP.NET Web API
- ComponentOne Wijmo Controls (C1.Web.Wijmo.Controls)
- Newtonsoft.Json

## Important Notes

1. **Namespace Inconsistency:** ProductManager uses `EpicAdventureWorks` namespace while most code uses `AdventureWorks` or `AdventureWorks.Business`

2. **Database Connection:** The connection string in Web.config contains placeholder credentials (`user id=xxxx;password=xxxx`) that need to be updated for actual database access

3. **Entity Framework Pattern:** The application creates a new DbContext instance for each operation via `Common.DataEntities`. This is not ideal for performance but is the established pattern in this codebase.

4. **Legacy Technology:** This is .NET Framework 4.5 with MVC 4, not .NET Core. Use appropriate syntax and patterns for legacy ASP.NET.
