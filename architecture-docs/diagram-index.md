# Diagram Index

**Analysis Date**: 2025-10-30
**System**: Adventure Works Cycle Store MVC Application

## Overview

This index provides a complete catalog of all architectural diagrams created during the analysis. Diagrams are organized by category and type, with references to their locations and the documentation that uses them.

**Total Diagrams**: 9
- C4 Model Diagrams: 4
- Data Architecture Diagrams: 2
- Integration Diagrams: 1
- Sequence Diagrams: 1
- Flow Diagrams: 1

---

## C4 Model Diagrams (MANDATORY)

### Level 1: System Context

**Diagram**: c4-level1-context.puml
**Type**: PlantUML
**Location**: `architecture-docs/diagrams/c4-level1-context.puml`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows the Adventure Works Cycle Store system and its external actors/systems

**Components Shown**:
- Web User (Customer)
- Adventure Works Cycle Store (ASP.NET MVC 4 Application)
- CYCLE_STORE Database (SQL Server on AWS RDS)

**Relationships**:
- Web User → System (HTTPS)
- System → Database (Entity Framework 5)

**Referenced In**:
- enterprise-architecture.md

**Key Insights**:
- Single external system (database)
- Forms authentication configured
- IIS Express hosting on port 55185

---

### Level 2: Container Diagram

**Diagram**: c4-level2-containers.puml
**Type**: PlantUML
**Location**: `architecture-docs/diagrams/c4-level2-containers.puml`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows the high-level technology containers of the system

**Containers Shown**:
- Web Application (ASP.NET MVC 4 Container)
- CYCLE_STORE Database (SQL Server on AWS RDS)
- ASPNetDB Database (SQL Server, location unknown)

**Technologies**:
- ASP.NET MVC 4
- .NET Framework 4.5
- Razor View Engine
- Entity Framework 5 (EDMX)
- Forms Authentication
- SQL Membership Provider

**Referenced In**:
- solution-architecture.md

**Key Insights**:
- Two database containers (one misconfigured)
- 30-day session timeout
- Custom error pages for 404 and 500
- Placeholder credentials detected

---

### Level 3: Component Diagrams

#### Web Application Components

**Diagram**: c4-level3-web-components.puml
**Type**: PlantUML
**Location**: `architecture-docs/diagrams/c4-level3-web-components.puml`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows internal components of the ASP.NET MVC Web Application

**Layers Shown**:
1. **Presentation Layer**:
   - HomeController
   - SiteLayoutController
   - ErrorController
   - Razor Views
   - View Models

2. **Business Logic Layer**:
   - CategoryManager (static methods)
   - ProductManager (static methods)

3. **Data Access Layer**:
   - Common (static factory)
   - CYCLE_STOREEntities (EF DbContext)
   - Entity Models

**Referenced In**:
- technical-architecture.md
- solution-architecture.md

**Key Insights**:
- Three-tier architecture within single container
- Static method pattern throughout
- DbContext anti-pattern (new instance per operation)
- Namespace inconsistency in ProductManager

#### MVC Pipeline Components

**Diagram**: c4-level3-mvc-pipeline.puml
**Type**: PlantUML
**Location**: `architecture-docs/diagrams/c4-level3-mvc-pipeline.puml`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows ASP.NET MVC pipeline and request flow components

**Components Shown**:

**HTTP Pipeline**:
- Global.asax (Application Entry)
- RouteConfig (URL Routing)
- FilterConfig (Global Filters)
- WebApiConfig (API Routing - unused)

**Authentication & Security**:
- Forms Authentication Module
- SQL Membership Provider
- HandleError Attribute

**MVC Controllers**:
- Controller Factory
- Action Invoker
- View Engine (Razor)

**Referenced In**:
- technical-architecture.md

**Key Insights**:
- Standard ASP.NET MVC 4 pipeline
- Forms authentication with 30-day timeout
- HandleErrorAttribute for global exception handling
- Web API configured but unused

---

## Data Architecture Diagrams

### Entity Relationship Diagram

**Diagram**: entity-relationship.mermaid
**Type**: Mermaid
**Location**: `architecture-docs/diagrams/entity-relationship.mermaid`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows database entity relationships from Entity Framework EDMX

**Entities**:
1. **ProductCategory** (4 properties)
   - ProductCategoryID (PK)
   - Name
   - rowguid
   - ModifiedDate

2. **ProductSubcategory** (5 properties)
   - ProductSubcategoryID (PK)
   - ProductCategoryID (FK)
   - Name
   - rowguid
   - ModifiedDate

3. **Product** (25 properties)
   - ProductID (PK)
   - Name, ProductNumber
   - Color, Size, Weight (nullable)
   - Pricing, inventory, lifecycle dates
   - ProductSubcategoryID (FK, nullable)
   - And 16 more properties

**Relationships**:
- ProductCategory (1) → (Many) ProductSubcategory
- ProductSubcategory (1) → (Many) Product (optional)

**Referenced In**:
- data-models.md
- enterprise-architecture.md

**Key Insights**:
- Simple hierarchical category structure
- Product can optionally belong to subcategory
- All entities have rowguid and ModifiedDate
- Many nullable properties on Product entity

### Data Flow Diagram

**Diagram**: data-flow.mermaid
**Type**: Mermaid
**Location**: `architecture-docs/diagrams/data-flow.mermaid`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows how data flows through the application layers

**Flow Path**:
1. Web Browser → User Action
2. IIS Express → Forms Authentication → MVC Routing
3. MVC Controller (Presentation)
4. Business Managers (CategoryManager, ProductManager)
5. Common.DataEntities (Static Factory)
6. CYCLE_STOREEntities (EF DbContext)
7. LINQ to Entities (Query Translation)
8. SQL Server Database
9. Return path through same layers
10. Razor View Engine → HTML Response

**Warnings Highlighted**:
- New DbContext created per request
- All queries use LINQ to Entities
- No caching implemented

**Referenced In**:
- data-models.md

**Key Insights**:
- Clear layer separation
- Every request queries database
- Static method calls throughout
- No optimization or caching

---

## Integration Diagrams

### Integration Landscape

**Diagram**: integration-landscape.puml
**Type**: PlantUML
**Location**: `architecture-docs/diagrams/integration-landscape.puml`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows all external integration points

**Systems/Actors**:
- Web User
- Web Application
- Forms Authentication
- SQL Membership Provider
- CYCLE_STORE Database (AWS RDS)
- ASPNetDB Database (Unknown Location)

**Integration Points**:
1. **CONFIRMED**: Application → CYCLE_STORE via Entity Framework 5
2. **MISCONFIGURED**: Application → ASPNetDB (connection string missing)

**NO Legacy Integrations Found**:
- ❌ No FTP/File integrations
- ❌ No SOAP/WCF services
- ❌ No REST API calls
- ❌ No Message Queuing
- ❌ No Windows Services
- ❌ No Batch processes
- ❌ No Email integration
- ❌ No Third-party APIs

**Referenced In**:
- integration-architecture.md

**Key Insights**:
- Minimal integration architecture (1 confirmed)
- Isolated application with database-only dependency
- ASPNetDB misconfiguration will cause runtime failures

---

## Sequence Diagrams

### MVC Request Sequence

**Diagram**: mvc-request-sequence.mermaid
**Type**: Mermaid
**Location**: `architecture-docs/diagrams/mvc-request-sequence.mermaid`
**Status**: ✅ COMPLETED
**Syntax Verified**: 2025-10-30

**Purpose**: Shows typical request/response flow through the MVC pipeline

**Example Flow**: GET /SiteLayout/FooterLayout

**Participants**:
- Browser
- IIS Express
- Forms Auth Module
- MVC Router
- SiteLayoutController
- CategoryManager
- Common.DataEntities
- CYCLE_STOREEntities
- SQL Server
- Razor Engine

**Steps** (11 total):
1. Browser sends GET request
2. IIS checks authentication
3. Router matches route
4. Controller invoked
5. Business manager called
6. Static factory creates DbContext
7. LINQ query built
8. SQL executed
9. Results materialized
10. View model populated
11. Razor view rendered

**Timing**: ~50-200ms (no caching, DB query every request)

**Referenced In**:
- api-documentation.md

**Key Insights**:
- Every layout render queries database
- New DbContext per request
- No caching layer
- Authentication check on every request

---

## Diagram Categories

### By Type

**PlantUML Diagrams** (5):
- c4-level1-context.puml
- c4-level2-containers.puml
- c4-level3-web-components.puml
- c4-level3-mvc-pipeline.puml
- integration-landscape.puml

**Mermaid Diagrams** (4):
- entity-relationship.mermaid
- data-flow.mermaid
- mvc-request-sequence.mermaid

### By Architecture Layer

**System Level** (2):
- c4-level1-context.puml
- c4-level2-containers.puml

**Application Level** (3):
- c4-level3-web-components.puml
- c4-level3-mvc-pipeline.puml
- mvc-request-sequence.mermaid

**Data Level** (2):
- entity-relationship.mermaid
- data-flow.mermaid

**Integration Level** (2):
- integration-landscape.puml
- data-flow.mermaid (also shows integration)

### By Documentation Reference

**enterprise-architecture.md**:
- c4-level1-context.puml
- entity-relationship.mermaid

**solution-architecture.md**:
- c4-level2-containers.puml
- c4-level3-web-components.puml

**technical-architecture.md**:
- c4-level3-web-components.puml
- c4-level3-mvc-pipeline.puml

**data-models.md**:
- entity-relationship.mermaid
- data-flow.mermaid

**api-documentation.md**:
- mvc-request-sequence.mermaid

**integration-architecture.md**:
- integration-landscape.puml

---

## Diagram Rendering

### PlantUML Diagrams

**Status**: ✅ ALL DIAGRAMS FIXED AND TESTED (2025-10-30)

All PlantUML diagrams have been rewritten with proper syntax and tested successfully with PlantUML 1.2020.2 on Ubuntu 24.04. See [PLANTUML-FIXES.md](./PLANTUML-FIXES.md) for details.

**Rendering Tools**:
- PlantUML command-line: `plantuml diagram-name.puml`
- PlantUML online server: http://www.plantuml.com/plantuml/
- Visual Studio Code with PlantUML extension
- IntelliJ IDEA with PlantUML plugin

**Format**: `.puml` files (text-based)

**Syntax**: Standard PlantUML syntax (no external dependencies required)
**Generated Files**: PNG images available in diagrams/ folder

### Mermaid Diagrams

**Rendering Tools**:
- Mermaid Live Editor: https://mermaid.live/
- Visual Studio Code with Mermaid extension
- GitHub/GitLab (native support)
- Markdown viewers with Mermaid support

**Format**: `.mermaid` files (text-based)

---

## Diagram Quality Checklist

**All diagrams verified for**:
✅ Syntax correctness (verified date noted in each file)
✅ Completeness (all required elements present)
✅ Accuracy (matches codebase analysis)
✅ Readability (clear labels and layout)
✅ Documentation integration (referenced in appropriate docs)
✅ Evidence-based (annotations with file:line references)

---

## Summary

**Diagram Coverage**:
- ✅ C4 Level 1 (System Context)
- ✅ C4 Level 2 (Container)
- ✅ C4 Level 3 (Component) - 2 diagrams
- ✅ Entity Relationship
- ✅ Data Flow
- ✅ Integration Landscape
- ✅ Sequence Diagram (MVC Request)

**Completeness**: All mandatory C4 diagrams completed plus additional supporting diagrams for data, integration, and request flow analysis.

**Status**: ✅ COMPLETE - All planned diagrams created and verified
