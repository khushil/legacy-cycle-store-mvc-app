# Architectural Analysis Plan
**Date**: 2025-10-30
**Project**: Adventure Works MVC Cycle Store
**Codebase Size**: 19 C# files, 606 lines of code

## Project Overview
This is a legacy ASP.NET MVC 4 application built on .NET Framework 4.5, representing a relatively small, focused web application for a cycle store. The codebase follows a traditional three-tier architecture pattern.

## Codebase Structure Discovery

### Solution Structure
- **Solution**: AdventureWorksMVC_2013.sln
- **Project**: Single ASP.NET MVC 4 web application
- **Total Files**: 19 C# source files
- **Total Lines of Code**: 606 LOC
- **Framework**: .NET Framework 4.5, ASP.NET MVC 4

### Directory Organization
```
AdventureWorksMVC/
├── App_Start/          # Application startup configuration (3 files)
├── Business/           # Business logic and data models (9 files)
├── Controllers/        # MVC Controllers (3 files)
├── Models/             # View Models (1 file)
├── Views/              # Razor views
├── Helpers/            # Utility helpers (1 file)
├── Properties/         # Assembly info and publish profiles
└── Web.config          # Application configuration
```

### Technology Stack Identified
- ASP.NET MVC 4
- Entity Framework 5 (Database-First with EDMX)
- SQL Server (AWS RDS hosted)
- Razor View Engine
- ASP.NET Web API
- ComponentOne Wijmo Controls
- Forms Authentication with SQL Membership Provider

### Integration Points Discovered
- **Database**: SQL Server on AWS RDS (`CYCLE_STORE` database)
- **No Legacy Integrations Found**:
  - No SOAP/WCF services
  - No file-based integrations (FTP, file shares)
  - No message queuing (MSMQ, RabbitMQ)
  - No Windows Services
  - No external API calls detected in initial scan

## Analysis Strategy

Given the small codebase size (~600 LOC), we can perform a **comprehensive single-pass analysis** rather than chunked approach.

### Analysis Sequence

1. **Entry Point Analysis** (CURRENT)
   - Global.asax.cs - Application startup
   - App_Start configuration files
   - Web.config examination

2. **Data Architecture Analysis**
   - Entity Framework EDMX model
   - Entity classes (Product, ProductCategory, ProductSubcategory)
   - Database context and connection strings

3. **Business Logic Analysis**
   - CategoryManager.cs
   - ProductManager.cs
   - Common.cs (data access patterns)

4. **Presentation Layer Analysis**
   - Controllers (HomeController, SiteLayoutController, ErrorController)
   - View Models
   - Routing configuration

5. **Cross-Cutting Concerns**
   - Authentication/Authorization setup
   - Error handling
   - Helpers and utilities

## Documentation Deliverables

### Required Documents
1. ✅ analysis-plan.md (this document)
2. ⏳ enterprise-architecture.md
3. ⏳ solution-architecture.md
4. ⏳ technical-architecture.md
5. ⏳ data-models.md
6. ⏳ api-documentation.md
7. ⏳ integration-architecture.md
8. ⏳ technical-debt-register.md
9. ⏳ cross-references.md
10. ⏳ analysis-index.md
11. ⏳ diagram-index.md

### Required Diagrams

#### C4 Model Diagrams (MANDATORY)
1. ⏳ c4-level1-context.puml - System Context
2. ⏳ c4-level2-containers.puml - Container Diagram
3. ⏳ c4-level3-web-components.puml - Web Application Components
4. ⏳ c4-level3-business-components.puml - Business Layer Components

#### Additional Diagrams
5. ⏳ entity-relationship.mermaid - Data Model
6. ⏳ data-flow.mermaid - Request/Response Flow
7. ⏳ auth-sequence.mermaid - Authentication Flow
8. ⏳ deployment.puml - Deployment Architecture
9. ⏳ middleware-pipeline.mermaid - ASP.NET MVC Pipeline

### Integration Architecture Notes
Based on initial scan, this application has:
- **Primary Integration**: SQL Server Database (AWS RDS)
- **No Legacy Integrations**: Clean modern web application pattern
- **Simple Architecture**: Monolithic web application with direct database access

## File Analysis Priority

### High Priority (Core Architecture)
1. `Global.asax.cs` - Application lifecycle
2. `Web.config` - Configuration and connection strings
3. `Business/CycleModel.edmx` - Data model
4. `Business/Common.cs` - Data access pattern
5. `Business/CategoryManager.cs` & `ProductManager.cs` - Business logic
6. `App_Start/RouteConfig.cs` - Routing
7. `Controllers/*.cs` - MVC controllers

### Medium Priority
8. `Models/SiteLayoutModel.cs` - View models
9. `App_Start/FilterConfig.cs` - Global filters
10. `App_Start/WebApiConfig.cs` - Web API configuration
11. `Helpers/WebResource.cs` - Utilities

### Low Priority (Supporting Files)
12. Entity classes (auto-generated from EDMX)
13. `Properties/AssemblyInfo.cs`

## Token Management Strategy
Given the small codebase size, token management should not be an issue. We can:
- Analyze all files in a single session
- Create all diagrams without checkpointing
- Complete comprehensive documentation in one pass

## Known Challenges
1. **Namespace Inconsistency**: ProductManager uses `EpicAdventureWorks` namespace
2. **Placeholder Credentials**: Database connection string has placeholder values
3. **Legacy Framework**: .NET Framework 4.5 (not .NET Core) - limited to Windows hosting
4. **DbContext Pattern**: New context created per operation via static property

## Next Steps
1. Read and analyze Global.asax.cs and App_Start files
2. Examine Entity Framework model and data access patterns
3. Document business logic layer
4. Create all C4 model diagrams
5. Generate supporting architectural diagrams
6. Compile findings into structured documentation
