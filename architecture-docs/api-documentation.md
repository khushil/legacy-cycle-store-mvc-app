# API Documentation

**Analysis Date**: 2025-10-30
**System**: Adventure Works Cycle Store MVC Application
**API Type**: MVC Controller Actions (Traditional Web UI)

## Executive Summary

The Adventure Works Cycle Store does **NOT** implement REST API endpoints despite having Web API routing configured. The application uses **TRADITIONAL MVC CONTROLLER ACTIONS** that return HTML views. There are **NO API CONTROLLERS** found in the codebase, making this purely a server-side rendered web application.

**Web API Status**: CONFIGURED BUT UNUSED (0 API controllers)

## Current State Architecture

### REST API Endpoints

**REST API CONTROLLERS**: NONE FOUND

**Evidence**:
- Grep search for "ApiController" returned NO results
- Grep search for "[HttpGet]", "[HttpPost]" with ApiController returned NONE
- Controllers namespace only contains MVC controllers
- No files in api/ folder

**Web API Configuration** (WebApiConfig.cs:10-16):
```csharp
config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
```

**Status**: Route registered but NO controllers to handle requests
**URL Pattern**: `http://localhost:55185/api/{controller}/{id}`
**Result**: 404 Not Found (no matching controllers)

### MVC Endpoints (Current Implementation)

The application uses traditional ASP.NET MVC controller actions that return HTML views.

#### 1. Home Controller Endpoints

**Controller**: HomeController.cs
**Namespace**: AdventureWorks.Controllers
**Base URL**: http://localhost:55185/Home

##### GET /Home/Default

**Method**: Default()
**Location**: HomeController.cs:10-14
**HTTP Verb**: GET (default)
**Authorization**: NONE (anonymous access)

**Request**:
```http
GET /Home/Default HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/Home/Default.cshtml
**Status Code**: 200 OK
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult Default()
{
    ViewBag.BodyClass = "homepage";
    return View();
}
```

**ViewBag Data**:
- BodyClass: "homepage"

**Purpose**: Main landing page / homepage

**Authentication Required**: NO
**Authorization**: NONE

#### 2. SiteLayout Controller Endpoints

**Controller**: SiteLayoutController.cs
**Namespace**: AdventureWorks.Controllers
**Base URL**: http://localhost:55185/SiteLayout

##### GET /SiteLayout/HeaderLayout

**Method**: HeaderLayout()
**Location**: SiteLayoutController.cs:20-24
**HTTP Verb**: GET (default)
**Authorization**: NONE

**Request**:
```http
GET /SiteLayout/HeaderLayout HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/SiteLayout/HeaderLayout.cshtml
**Status Code**: 200 OK
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult HeaderLayout()
{
    SiteLayoutModel Header = new SiteLayoutModel();
    return View(Header);
}
```

**Model**: SiteLayoutModel (empty initialization)

**Purpose**: Renders header partial view for site layout

**Authentication Required**: NO
**Authorization**: NONE

##### GET /SiteLayout/FooterLayout

**Method**: FooterLayout()
**Location**: SiteLayoutController.cs:26-31
**HTTP Verb**: GET (default)
**Authorization**: NONE

**Request**:
```http
GET /SiteLayout/FooterLayout HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/SiteLayout/FooterLayout.cshtml
**Status Code**: 200 OK
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult FooterLayout()
{
    SiteLayoutModel Footer = new SiteLayoutModel();
    Footer.ProductCategories = CategoryManager.GetMainCategories();
    return View(Footer);
}
```

**Model Data**:
- ProductCategories: List<ProductCategory> (all categories from database)

**Database Query**: YES
- Manager: CategoryManager.GetMainCategories()
- Query: SELECT * FROM ProductCategory

**Purpose**: Renders footer with product category navigation links

**Authentication Required**: NO
**Authorization**: NONE

##### GET /SiteLayout/ContentLayout

**Method**: ContentLayout()
**Location**: SiteLayoutController.cs:33-38
**HTTP Verb**: GET (default)
**Authorization**: NONE

**Request**:
```http
GET /SiteLayout/ContentLayout HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/SiteLayout/ContentLayout.cshtml
**Status Code**: 200 OK
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult ContentLayout()
{
    SiteLayoutModel Header = new SiteLayoutModel();
    Header.ProductCategories = CategoryManager.GetMainCategories();
    return View(Header);
}
```

**Model Data**:
- ProductCategories: List<ProductCategory> (all categories from database)

**Database Query**: YES
- Manager: CategoryManager.GetMainCategories()
- Query: SELECT * FROM ProductCategory

**Purpose**: Renders main content layout with category navigation

**Authentication Required**: NO
**Authorization**: NONE

#### 3. Error Controller Endpoints

**Controller**: ErrorController.cs
**Namespace**: AdventureWorks.Controllers
**Base URL**: http://localhost:55185/Error

##### GET /Error/Default

**Method**: Default()
**Location**: ErrorController.cs:10-14
**HTTP Verb**: GET (default)
**Authorization**: NONE

**Request**:
```http
GET /Error/Default HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/Error/Default.cshtml
**Status Code**: 200 OK (Note: Should be error code but returns 200)
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult Default()
{
    ViewBag.ErrMsg = "An error occurred while processing your request.";
    return View();
}
```

**ViewBag Data**:
- ErrMsg: "An error occurred while processing your request."

**Purpose**: Generic error page for 404 and 500 errors

**Triggered By**:
- 404 errors (Web.config:25)
- 500 errors (Web.config:26)

**Authentication Required**: NO
**Authorization**: NONE

##### GET /Error/GenericError

**Method**: GenericError()
**Location**: ErrorController.cs:16-20
**HTTP Verb**: GET (default)
**Authorization**: NONE

**Request**:
```http
GET /Error/GenericError HTTP/1.1
Host: localhost:55185
```

**Parameters**: NONE

**Response Type**: ViewResult
**View**: Views/Error/GenericError.cshtml (likely) or default view
**Status Code**: 200 OK
**Content-Type**: text/html

**Implementation**:
```csharp
public ActionResult GenericError()
{
    ViewBag.ErrMsg = "An error occurred while processing your request.";
    return View();
}
```

**ViewBag Data**:
- ErrMsg: "An error occurred while processing your request."

**Purpose**: Alternative generic error handler

**Authentication Required**: NO
**Authorization**: NONE

### Routing Configuration

**MVC Routing** (RouteConfig.cs:16-20):

```csharp
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Default", id = UrlParameter.Optional }
);
```

**Route Pattern**: `{controller}/{action}/{id}`

**Defaults**:
- Controller: "Home"
- Action: "Default"
- ID: Optional

**Examples**:
- `/` → Home/Default
- `/Home` → Home/Default
- `/Home/Default` → Home/Default
- `/SiteLayout/FooterLayout` → SiteLayout/FooterLayout
- `/Error` → Error/Default

**Route Constraints**: NONE

**Attribute Routing**: NOT USED (ASP.NET MVC 4 supports attribute routing but not configured)

### Request/Response DTOs

**DTOs**: MINIMAL

#### SiteLayoutModel

**File**: Models/SiteLayoutModel.cs
**Namespace**: AdventureWorks.Models
**Purpose**: View model for layout components

**Properties**:
- AnonymousTemplateVisibility: string
- LoggedInTemplateVisibility: string
- LoggedInEmailID: string
- ShoppingCartItemsCount: string
- ProductCategories: List<ProductCategory>

**Usage**:
- SiteLayoutController.HeaderLayout() (line 22)
- SiteLayoutController.FooterLayout() (line 28)
- SiteLayoutController.ContentLayout() (line 35)

**Populated Data**:
- ProductCategories populated from CategoryManager.GetMainCategories()
- Other properties NOT USED in analyzed code

### Authorization Attributes

**AUTHORIZATION**: NOT IMPLEMENTED

**Search Results**:
- NO [Authorize] attributes found
- NO [AllowAnonymous] attributes found
- NO role-based security
- NO custom authorization filters (beyond HandleErrorAttribute)

**Current State**: All endpoints publicly accessible (anonymous)

**Forms Authentication Configured**: YES (Web.config:28-30)
- Login URL: ~/Home/Login
- However, NO Login action found in HomeController

**Assessment**: Authentication configured but NOT enforced on any actions

### API Versioning

**API VERSIONING**: NOT APPLICABLE

**Reason**: No API endpoints exist

**MVC Versioning**: NOT IMPLEMENTED
- No version numbers in URLs
- No Accept header versioning
- No query string versioning

### Content Negotiation

**CONTENT NEGOTIATION**: NOT APPLICABLE

**Reason**:
- MVC controllers return ViewResult (HTML only)
- NO JSON/XML serialization
- NO Accept header processing
- NO multiple response formats

**Fixed Response Format**: text/html (Razor views)

### Rate Limiting

**RATE LIMITING**: NOT IMPLEMENTED

**Evidence**:
- NO throttling attributes
- NO rate limiting middleware
- NO IP-based restrictions
- NO request quotas

### CORS Configuration

**CORS**: NOT APPLICABLE

**Reason**:
- Traditional server-side rendered application
- No JavaScript API calls from external domains
- Web API not used

### Pagination

**PAGINATION**: NOT IMPLEMENTED

**Evidence**:
- CategoryManager.GetMainCategories() returns ALL categories (CategoryManager.cs:21-23)
- NO Take() or Skip() in queries
- NO page parameters in actions
- Full result sets loaded: `.ToList()`

**Impact**: Potential performance issue with large datasets

### Filtering and Sorting

**FILTERING**: IMPLEMENTED (Business Logic Layer)

**Available Filters** (ProductManager):
- GetProductByName(string) - Filter by product name
- GetProductByCategory(ProductSubcategory) - Filter by subcategory
- GetProductColor(int) - Get distinct colors for subcategory
- GetProductWeight(int) - Get distinct weights for subcategory
- GetProductSize(int) - Get distinct sizes for subcategory

**SORTING**: IMPLEMENTED

**Evidence**:
- ProductManager.GetProductColor() includes `orderby p.Color` (ProductManager.cs:88)
- ProductManager.GetProductWeight() includes `orderby p.Weight` (ProductManager.cs:104)
- ProductManager.GetProductSize() includes `orderby p.Size` (ProductManager.cs:139)

**Sorting Method**: Database-side (LINQ orderby translates to SQL ORDER BY)

### Caching Headers

**CACHING HEADERS**: NOT CONFIGURED

**Evidence**:
- NO OutputCache attributes on actions
- NO Cache-Control headers set
- NO ETag headers
- NO Last-Modified headers

**Current Behavior**: No browser caching, server queries database on every request

### Error Responses

**Error Handling**:

**Global Exception Handling**:
- HandleErrorAttribute (FilterConfig.cs:10)
- Catches unhandled exceptions globally

**Custom Error Pages**:
- 404 → ~/Error/Default (Web.config:25)
- 500 → ~/Error/Default (Web.config:26)

**Error Response Format**:
- Content-Type: text/html
- Status Code: 200 OK (INCORRECT - should return 404 or 500)
- Body: HTML error page

**API Error Responses**: NOT APPLICABLE (no API)

### Security Headers

**SECURITY HEADERS**: NOT CONFIGURED

**Missing Headers**:
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY or SAMEORIGIN
- X-XSS-Protection: 1; mode=block
- Content-Security-Policy
- Strict-Transport-Security (HSTS)

**Impact**: Potential security vulnerabilities (clickjacking, XSS)

## Diagrams

**MVC Request Sequence**: [See mvc-request-sequence.mermaid](./diagrams/mvc-request-sequence.mermaid)

## Endpoint Summary Table

| Endpoint | Method | Controller | Action | Auth | Database | Purpose |
|----------|--------|------------|--------|------|----------|---------|
| / | GET | Home | Default | No | No | Landing page |
| /Home/Default | GET | Home | Default | No | No | Landing page |
| /SiteLayout/HeaderLayout | GET | SiteLayout | HeaderLayout | No | No | Header partial |
| /SiteLayout/FooterLayout | GET | SiteLayout | FooterLayout | No | Yes | Footer with categories |
| /SiteLayout/ContentLayout | GET | SiteLayout | ContentLayout | No | Yes | Content with categories |
| /Error/Default | GET | Error | Default | No | No | Error page (404/500) |
| /Error/GenericError | GET | Error | GenericError | No | No | Generic error |

**Total Endpoints**: 7 MVC actions
**API Endpoints**: 0
**Authenticated Endpoints**: 0
**Database Queries**: 2 actions

## API Architecture Assessment

### What Should Be API Endpoints (But Aren't)

**MISSING API ENDPOINTS** (Common e-commerce patterns):

1. **Product Catalog API**:
   - GET /api/products - List products
   - GET /api/products/{id} - Get product details
   - GET /api/categories - List categories
   - GET /api/categories/{id}/products - Products by category

2. **Shopping Cart API**:
   - GET /api/cart - Get cart contents
   - POST /api/cart/items - Add item to cart
   - PUT /api/cart/items/{id} - Update cart item
   - DELETE /api/cart/items/{id} - Remove cart item

3. **Order API**:
   - POST /api/orders - Create order
   - GET /api/orders/{id} - Get order details

**Current Reality**: All functionality via server-side rendered HTML

### Why No API?

**HISTORICAL CONTEXT**:
- ASP.NET MVC 4 (2012) predates modern SPA frameworks
- Web applications primarily server-side rendered
- REST APIs less common for traditional web apps
- Web API included but often unused in MVC projects

**PROJECT SCOPE**:
- Small application (606 LOC)
- No JavaScript-heavy frontend
- No mobile app integration
- No third-party integrations

### REST API Maturity Level

**Richardson Maturity Model**: N/A

**Reason**: No REST API endpoints exist

**If APIs Existed, Expected Level**: Level 2
- Level 0: Not applicable
- Level 1: Resources (not implemented)
- Level 2: HTTP Verbs (likely implementation)
- Level 3: HATEOAS (unlikely)

## Missing Endpoints (Inferred from Business Logic)

**BUSINESS LOGIC EXISTS BUT NO ENDPOINTS**:

These methods exist in business managers but have NO corresponding controller actions:

### Product Endpoints (Not Exposed)

1. **Get Product by Name** (ProductManager.cs:33)
   - Method: GetProductByName(string name)
   - Missing Endpoint: GET /Products/Details?name={name}

2. **Get Products by Category** (ProductManager.cs:46)
   - Method: GetProductByCategory(ProductSubcategory category)
   - Missing Endpoint: GET /Products/ByCategory/{subcategoryId}

3. **Get Product by ID** (ProductManager.cs:60)
   - Method: GetProductByProductId(int prodId)
   - Missing Endpoint: GET /Products/Details/{id}

4. **Get Product Colors** (ProductManager.cs:83)
   - Method: GetProductColor(int categoryId)
   - Missing Endpoint: GET /Products/Colors?categoryId={id}

5. **Get Product Sizes** (ProductManager.cs:134)
   - Method: GetProductSize(int categoryId)
   - Missing Endpoint: GET /Products/Sizes?categoryId={id}

6. **Get Product Weights** (ProductManager.cs:98)
   - Method: GetProductWeight(int categoryId)
   - Missing Endpoint: GET /Products/Weights?categoryId={id}

7. **Get Product Description** (ProductManager.cs:149)
   - Method: GetProductDesc(int descId)
   - Missing Endpoint: GET /Products/Description/{descId}

### Category Endpoints (Partially Exposed)

1. **Get All Categories** (CategoryManager.cs:19)
   - Method: GetMainCategories()
   - Exposed: Via FooterLayout and ContentLayout (indirect)
   - Missing Endpoint: GET /api/categories

2. **Get Category by Name** (CategoryManager.cs:31)
   - Method: GetCategoryByName(string name)
   - Missing Endpoint: GET /api/categories?name={name}

3. **Get Subcategory by Name** (CategoryManager.cs:44)
   - Method: GetProductSubcategoryByName(string name)
   - Missing Endpoint: GET /api/subcategories?name={name}

### Authentication Endpoints (Configured but Missing)

**Login URL Configured**: ~/Home/Login (Web.config:29)
**Status**: NO Login action found in HomeController

**Missing Endpoints**:
- POST /Home/Login - User authentication
- POST /Home/Logout - User sign out
- POST /Home/Register - User registration

## Summary

**REST API Endpoints**: 0 (NONE)
**MVC Controller Actions**: 7
**Web API Configured**: YES (but unused)
**Authorization Implemented**: NO
**API Documentation**: NOT APPLICABLE (no APIs)
**API Versioning**: NOT APPLICABLE
**Rate Limiting**: NONE
**CORS**: NOT APPLICABLE
**Authentication**: Configured but not enforced

**Conclusion**: This is a **TRADITIONAL SERVER-SIDE RENDERED WEB APPLICATION** with NO REST API implementation despite Web API framework being included and configured. All functionality is delivered via HTML views generated by MVC controllers.
