# Django Oscar - Architectural Analysis

This document provides a comprehensive architectural analysis of Django Oscar, an e-commerce framework for Django. It includes C4 diagrams, workflow diagrams, and sequence diagrams for critical areas of the system.

## Table of Contents

1. [C4 Context Diagram](#c4-context-diagram)
2. [C4 Container Diagram](#c4-container-diagram)
3. [C4 Component Diagrams](#c4-component-diagrams)
4. [Workflow Diagrams](#workflow-diagrams)
5. [Sequence Diagrams](#sequence-diagrams)

---

## C4 Context Diagram

The Context diagram shows Django Oscar within its broader ecosystem, including external systems and users.

```mermaid
C4Context
    title System Context Diagram for Django Oscar E-Commerce Framework

    Person(customer, "Customer", "A person browsing and purchasing products")
    Person(admin, "Admin/Merchant", "Manages products, orders, and site configuration")

    System(oscar, "Django Oscar", "E-commerce framework providing catalog, basket, checkout, and order management")

    System_Ext(payment_gateway, "Payment Gateway", "Processes payments (Stripe, PayPal, etc.)")
    System_Ext(search_engine, "Search Engine", "Haystack-compatible search backend (Elasticsearch, Solr, Whoosh)")
    System_Ext(email_service, "Email Service", "Sends transactional emails")
    System_Ext(shipping_provider, "Shipping Provider", "Calculates shipping rates and tracks shipments")
    System_Ext(analytics, "Analytics Service", "Tracks customer behavior and conversions")

    Rel(customer, oscar, "Browses products, adds to basket, checks out")
    Rel(admin, oscar, "Manages catalog, processes orders, configures site")

    Rel(oscar, payment_gateway, "Processes payments", "HTTPS/API")
    Rel(oscar, search_engine, "Indexes and searches products", "Haystack API")
    Rel(oscar, email_service, "Sends order confirmations, notifications", "SMTP")
    Rel(oscar, shipping_provider, "Calculates shipping, tracks packages", "API")
    Rel(oscar, analytics, "Sends event data", "JavaScript/API")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

---

## C4 Container Diagram

The Container diagram shows the major technical components within Django Oscar.

```mermaid
C4Container
    title Container Diagram for Django Oscar

    Person(customer, "Customer")
    Person(admin, "Admin")

    Container_Boundary(oscar_boundary, "Django Oscar Application") {
        Container(web_app, "Web Application", "Django", "Handles HTTP requests, renders templates, manages sessions")
        Container(dashboard, "Admin Dashboard", "Django Apps", "Backend interface for managing catalog, orders, customers")
        Container(api, "API Layer", "Django REST Framework (optional)", "RESTful API for headless commerce")

        ContainerDb(db, "Database", "PostgreSQL/MySQL", "Stores products, orders, customers, baskets")
        ContainerDb(cache, "Cache", "Redis/Memcached", "Session storage, basket caching")
        Container(search_index, "Search Index", "Haystack", "Product search and filtering")
        Container(static_files, "Static Assets", "CSS/JS/Images", "Frontend assets built with Gulp/SASS")
    }

    System_Ext(payment_gateway, "Payment Gateway")
    System_Ext(search_backend, "Search Backend", "Elasticsearch/Solr")

    Rel(customer, web_app, "Uses", "HTTPS")
    Rel(admin, dashboard, "Manages", "HTTPS")
    Rel(customer, api, "Uses (optional)", "HTTPS/JSON")

    Rel(web_app, db, "Reads/Writes", "SQL")
    Rel(dashboard, db, "Reads/Writes", "SQL")
    Rel(web_app, cache, "Reads/Writes", "Redis Protocol")
    Rel(web_app, search_index, "Queries", "Haystack API")
    Rel(search_index, search_backend, "Indexes/Queries", "Native Protocol")
    Rel(web_app, static_files, "Serves")
    Rel(web_app, payment_gateway, "Processes payments", "HTTPS")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

---

## C4 Component Diagrams

### Catalogue Component Diagram

Shows the internal structure of the Catalogue app, which manages products and categories.

```mermaid
C4Component
    title Component Diagram - Catalogue App

    Container_Boundary(catalogue_boundary, "Catalogue App") {
        Component(product_model, "Product Model", "Abstract Model", "Represents products with variants, attributes, and relationships")
        Component(category_model, "Category Model", "Treebeard Model", "Hierarchical category structure")
        Component(product_class, "Product Class", "Abstract Model", "Defines product types and their attributes")
        Component(product_attribute, "Product Attribute", "EAV Pattern", "Flexible product attributes")

        Component(product_views, "Product Views", "Django Views", "Product detail, list, and search views")
        Component(category_views, "Category Views", "Django Views", "Category browsing and filtering")

        Component(product_manager, "Product Manager", "Django Manager", "Custom querysets for product retrieval")
        Component(availability_policy, "Availability Policy", "Strategy Pattern", "Determines product availability and pricing")
    }

    ContainerDb(db, "Database")
    Container(search_index, "Search Index")
    Container(partner_app, "Partner App")

    Rel(product_views, product_model, "Queries")
    Rel(category_views, category_model, "Queries")
    Rel(product_model, product_class, "Has a")
    Rel(product_model, product_attribute, "Has many")
    Rel(product_model, product_manager, "Uses")
    Rel(product_views, availability_policy, "Checks availability")

    Rel(product_model, db, "Persists to")
    Rel(category_model, db, "Persists to")
    Rel(product_manager, search_index, "Indexes products")
    Rel(availability_policy, partner_app, "Gets stock info from")

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

### Checkout & Order Component Diagram

Shows the checkout and order placement flow components.

```mermaid
C4Component
    title Component Diagram - Checkout & Order Processing

    Container_Boundary(checkout_boundary, "Checkout & Order Apps") {
        Component(checkout_views, "Checkout Views", "Django Views", "Multi-step checkout flow (gateway, shipping, payment, preview)")
        Component(checkout_session, "Checkout Session", "Session Mixin", "Manages checkout state across steps")
        Component(order_placement, "Order Placement Mixin", "Business Logic", "Creates orders from baskets")

        Component(order_model, "Order Model", "Abstract Model", "Stores completed orders with lines, shipping, billing")
        Component(order_creator, "Order Creator", "Service", "Converts basket to order, handles stock allocation")
        Component(payment_handler, "Payment Handler", "Strategy Pattern", "Processes payments via gateways")

        Component(shipping_methods, "Shipping Methods", "Repository Pattern", "Available shipping options and costs")
        Component(shipping_calculator, "Shipping Calculator", "Service", "Calculates shipping charges")
    }

    Container(basket_app, "Basket App")
    Container(partner_app, "Partner App")
    System_Ext(payment_gateway, "Payment Gateway")
    ContainerDb(db, "Database")

    Rel(checkout_views, checkout_session, "Uses")
    Rel(checkout_views, order_placement, "Uses")
    Rel(order_placement, order_creator, "Calls")
    Rel(order_creator, order_model, "Creates")
    Rel(checkout_views, payment_handler, "Processes payment")
    Rel(checkout_views, shipping_methods, "Gets options")
    Rel(shipping_methods, shipping_calculator, "Calculates costs")

    Rel(order_creator, basket_app, "Reads from")
    Rel(order_creator, partner_app, "Allocates stock in")
    Rel(payment_handler, payment_gateway, "Calls")
    Rel(order_model, db, "Persists to")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

### Dynamic Class Loading Component Diagram

Shows Oscar's customization mechanism through dynamic class loading.

```mermaid
C4Component
    title Component Diagram - Dynamic Class Loading System

    Container_Boundary(loading_boundary, "Class Loading System") {
        Component(get_class, "get_class()", "Core Function", "Dynamically loads classes from app modules")
        Component(get_model, "get_model()", "Core Function", "Dynamically loads models from apps")
        Component(class_loader, "Class Loader", "Strategy Pattern", "Configurable loader (default or custom)")

        Component(app_registry, "App Registry", "Django Apps", "Maintains list of installed apps")
        Component(forked_apps, "Forked Apps", "Custom Apps", "Project-specific app overrides")
        Component(core_apps, "Core Apps", "Oscar Apps", "Default Oscar implementations")
    }

    Container(views, "Views/Forms/Services")

    Rel(views, get_class, "Imports classes via")
    Rel(views, get_model, "Imports models via")
    Rel(get_class, class_loader, "Delegates to")
    Rel(get_model, class_loader, "Delegates to")

    Rel(class_loader, app_registry, "Queries")
    Rel(class_loader, forked_apps, "Tries first")
    Rel(class_loader, core_apps, "Falls back to")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

---

## Workflow Diagrams

### Complete Checkout Process

End-to-end workflow from basket to order confirmation.

```mermaid
flowchart TD
    Start([Customer has items in basket]) --> CheckAuth{Authenticated?}

    CheckAuth -->|No| GuestEmail[Enter email or sign in]
    CheckAuth -->|Yes| ShippingAddr
    GuestEmail --> CreateAccount{Create account?}
    CreateAccount -->|Yes| Register[Register account]
    CreateAccount -->|No| ShippingAddr
    Register --> ShippingAddr

    ShippingAddr[Enter/Select Shipping Address] --> ValidateAddr{Valid address?}
    ValidateAddr -->|No| ShippingAddr
    ValidateAddr -->|Yes| ShippingMethod

    ShippingMethod[Select Shipping Method] --> GetRates[Calculate shipping rates]
    GetRates --> DisplayRates[Display available methods]
    DisplayRates --> SelectMethod{Method selected?}
    SelectMethod -->|No| DisplayRates
    SelectMethod -->|Yes| PaymentMethod

    PaymentMethod[Enter Payment Details] --> ValidatePayment{Valid payment?}
    ValidatePayment -->|No| PaymentMethod
    ValidatePayment -->|Yes| OrderPreview

    OrderPreview[Review Order Summary] --> ApplyVoucher{Apply voucher?}
    ApplyVoucher -->|Yes| ValidateVoucher{Valid?}
    ValidateVoucher -->|No| OrderPreview
    ValidateVoucher -->|Yes| RecalculateTotal
    ApplyVoucher -->|No| Confirm
    RecalculateTotal[Recalculate total] --> OrderPreview

    Confirm{Confirm order?} -->|No| OrderPreview
    Confirm -->|Yes| FreezeBasket[Freeze basket]

    FreezeBasket --> ProcessPayment[Process payment]
    ProcessPayment --> PaymentSuccess{Payment OK?}

    PaymentSuccess -->|No| PaymentError[Display error]
    PaymentError --> UnfreezeBasket[Unfreeze basket]
    UnfreezeBasket --> PaymentMethod

    PaymentSuccess -->|Yes| CreateOrder[Create order from basket]
    CreateOrder --> AllocateStock[Allocate stock]
    AllocateStock --> StockCheck{Stock available?}

    StockCheck -->|No| StockError[Handle stock error]
    StockError --> OrderFailed[Order placement failed]
    OrderFailed --> UnfreezeBasket

    StockCheck -->|Yes| SubmitBasket[Mark basket as submitted]
    SubmitBasket --> SendConfirmation[Send order confirmation email]
    SendConfirmation --> TriggerSignals[Trigger order_placed signals]
    TriggerSignals --> ThankYou([Redirect to thank you page])

    style Start fill:#e1f5e1
    style ThankYou fill:#e1f5e1
    style PaymentError fill:#ffe1e1
    style StockError fill:#ffe1e1
    style OrderFailed fill:#ffe1e1
```

### Product Customization Flow (App Forking)

Workflow for customizing Oscar apps using the fork mechanism.

```mermaid
flowchart TD
    Start([Need to customize Oscar app]) --> Identify[Identify app to customize]

    Identify --> CreateRoot{Root module exists?}
    CreateRoot -->|No| MakeRoot[Create yourappsfolder/__init__.py]
    CreateRoot -->|Yes| ForkCommand
    MakeRoot --> ForkCommand

    ForkCommand[Run: oscar_fork_app <app> <folder>] --> GenerateFiles[Generate app structure]

    GenerateFiles --> FilesCreated[Files created:<br/>- admin.py<br/>- apps.py<br/>- models.py<br/>- migrations/]

    FilesCreated --> UpdateSettings[Update INSTALLED_APPS]
    UpdateSettings --> ReplaceApp[Replace Oscar app config with forked app]

    ReplaceApp --> DashboardCheck{Customizing dashboard app?}
    DashboardCheck -->|Yes| ForkCoreDashboard[Also fork core dashboard app]
    DashboardCheck -->|No| CustomizeCode
    ForkCoreDashboard --> CustomizeCode

    CustomizeCode[Add customizations] --> CustomizeType{What to customize?}

    CustomizeType -->|Models| ImportAbstract[Import abstract model from Oscar]
    CustomizeType -->|Views| ImportView[Import view from Oscar]
    CustomizeType -->|Forms| ImportForm[Import form from Oscar]

    ImportAbstract --> ExtendClass[Extend with custom fields/methods]
    ImportView --> ExtendClass
    ImportForm --> ExtendClass

    ExtendClass --> MakeMigrations{Models changed?}
    MakeMigrations -->|Yes| RunMakeMigrations[python manage.py makemigrations]
    MakeMigrations -->|No| TestCustomization
    RunMakeMigrations --> RunMigrate[python manage.py migrate]
    RunMigrate --> TestCustomization

    TestCustomization[Test customization] --> Works{Works correctly?}
    Works -->|No| DebugIssue[Debug issue]
    DebugIssue --> CustomizeCode
    Works -->|Yes| Complete([Customization complete])

    style Start fill:#e1f5e1
    style Complete fill:#e1f5e1
    style DebugIssue fill:#fff4e1
```

### Offer and Voucher Application

How promotions and discounts are applied to baskets.

```mermaid
flowchart TD
    Start([Basket modified or voucher added]) --> LoadBasket[Load basket with lines]

    LoadBasket --> GetOffers[Query active offers]
    GetOffers --> OfferTypes{Offer types}

    OfferTypes --> SiteOffers[Site-wide offers]
    OfferTypes --> VoucherOffers[Voucher-based offers]
    OfferTypes --> UserOffers[User-specific offers]

    SiteOffers --> EvaluateSite[Evaluate site offer conditions]
    VoucherOffers --> EvaluateVoucher[Evaluate voucher conditions]
    UserOffers --> EvaluateUser[Evaluate user offer conditions]

    EvaluateSite --> SiteMatch{Conditions met?}
    EvaluateVoucher --> VoucherMatch{Conditions met?}
    EvaluateUser --> UserMatch{Conditions met?}

    SiteMatch -->|Yes| ApplySiteBenefit[Apply benefit]
    SiteMatch -->|No| NextOffer1
    VoucherMatch -->|Yes| ApplyVoucherBenefit[Apply benefit]
    VoucherMatch -->|No| NextOffer2
    UserMatch -->|Yes| ApplyUserBenefit[Apply benefit]
    UserMatch -->|No| NextOffer3

    ApplySiteBenefit --> CalculateDiscount1[Calculate discount amount]
    ApplyVoucherBenefit --> CalculateDiscount2[Calculate discount amount]
    ApplyUserBenefit --> CalculateDiscount3[Calculate discount amount]

    CalculateDiscount1 --> ApplyToLines1[Apply to basket lines]
    CalculateDiscount2 --> ApplyToLines2[Apply to basket lines]
    CalculateDiscount3 --> ApplyToLines3[Apply to basket lines]

    ApplyToLines1 --> NextOffer1{More offers?}
    ApplyToLines2 --> NextOffer2{More offers?}
    ApplyToLines3 --> NextOffer3{More offers?}

    NextOffer1 -->|Yes| GetOffers
    NextOffer2 -->|Yes| GetOffers
    NextOffer3 -->|Yes| GetOffers

    NextOffer1 -->|No| RecalculateTotal
    NextOffer2 -->|No| RecalculateTotal
    NextOffer3 -->|No| RecalculateTotal

    RecalculateTotal[Recalculate basket total] --> ShippingDiscount{Shipping discounts?}
    ShippingDiscount -->|Yes| ApplyShippingDiscount[Reduce shipping cost]
    ShippingDiscount -->|No| UpdateBasket
    ApplyShippingDiscount --> UpdateBasket

    UpdateBasket[Update basket with discounts] --> End([Return discounted basket])

    style Start fill:#e1f5e1
    style End fill:#e1f5e1
```

---

## Sequence Diagrams

### Add Product to Basket

Interaction flow when a customer adds a product to their basket.

```mermaid
sequenceDiagram
    actor Customer
    participant View as Product Detail View
    participant Form as AddToBasketForm
    participant Strategy as Pricing Strategy
    participant Basket as Basket Model
    participant Line as Basket Line
    participant Stock as Stock Record
    participant DB as Database

    Customer->>View: Click "Add to Basket"
    View->>Form: Validate form (quantity, product)

    alt Invalid Form
        Form-->>View: Validation errors
        View-->>Customer: Display errors
    else Valid Form
        Form->>Strategy: Get product pricing
        Strategy->>Stock: Check availability
        Stock-->>Strategy: Return stock level & price
        Strategy-->>Form: Return price info

        Form->>Basket: Get or create basket for user
        Basket->>DB: Query basket
        DB-->>Basket: Return basket or create new

        Basket->>Line: Check if product already in basket

        alt Product exists in basket
            Line->>Line: Increase quantity
            Line->>Stock: Check sufficient stock
            Stock-->>Line: Confirm stock available
            Line->>DB: Update quantity
        else New product
            Basket->>Line: Create new basket line
            Line->>Stock: Check sufficient stock
            Stock-->>Line: Confirm stock available
            Line->>DB: Insert line
        end

        Line->>Basket: Recalculate basket total
        Basket->>DB: Save basket
        DB-->>Basket: Confirmed

        Basket-->>View: Success
        View->>View: Add success message
        View-->>Customer: Show basket widget updated
    end
```

### Complete Checkout and Order Placement

Detailed sequence of placing an order through checkout.

```mermaid
sequenceDiagram
    actor Customer
    participant CheckoutView as Checkout View
    participant Session as Checkout Session
    participant PaymentHandler as Payment Handler
    participant Gateway as Payment Gateway
    participant OrderCreator as Order Creator
    participant Basket as Basket Model
    participant Order as Order Model
    participant StockRecord as Stock Record
    participant Email as Email Service
    participant Signals as Django Signals

    Customer->>CheckoutView: Submit order (preview page)
    CheckoutView->>Session: Validate checkout session
    Session-->>CheckoutView: Session valid

    CheckoutView->>Basket: Freeze basket
    Basket->>Basket: Set status = FROZEN

    CheckoutView->>PaymentHandler: Process payment
    PaymentHandler->>Gateway: Charge customer

    alt Payment Failed
        Gateway-->>PaymentHandler: Payment declined
        PaymentHandler-->>CheckoutView: Payment error
        CheckoutView->>Basket: Unfreeze basket
        CheckoutView-->>Customer: Display error message
    else Payment Successful
        Gateway-->>PaymentHandler: Payment confirmed
        PaymentHandler-->>CheckoutView: Payment successful

        CheckoutView->>OrderCreator: Create order from basket
        OrderCreator->>Basket: Get basket lines
        Basket-->>OrderCreator: Return lines

        OrderCreator->>Order: Create order instance
        Order->>Order: Generate order number

        loop For each basket line
            OrderCreator->>Order: Create order line
            OrderCreator->>StockRecord: Allocate stock

            alt Insufficient Stock
                StockRecord-->>OrderCreator: Stock error
                OrderCreator->>Order: Rollback order
                OrderCreator->>Basket: Unfreeze basket
                OrderCreator-->>CheckoutView: Stock allocation failed
                CheckoutView-->>Customer: Display stock error
            else Stock Allocated
                StockRecord-->>OrderCreator: Stock allocated
            end
        end

        OrderCreator->>Order: Save order with lines
        Order->>Basket: Mark basket as SUBMITTED
        Order-->>OrderCreator: Order created

        OrderCreator->>Email: Send order confirmation
        Email-->>Customer: Email sent

        OrderCreator->>Signals: Trigger order_placed signal
        Signals->>Signals: Notify listeners (analytics, etc.)

        OrderCreator-->>CheckoutView: Order successful
        CheckoutView->>Session: Clear checkout session
        CheckoutView-->>Customer: Redirect to thank you page
    end
```

### Dynamic Class Loading Mechanism

How Oscar's dynamic class loading works to support customization.

```mermaid
sequenceDiagram
    participant Code as Application Code
    participant GetClass as get_class()
    participant Loader as Class Loader
    participant AppRegistry as Django App Registry
    participant ForkedApp as Forked App Module
    participant CoreApp as Oscar Core App
    participant Cache as LRU Cache

    Code->>GetClass: get_class('catalogue.forms', 'ProductForm')
    GetClass->>Cache: Check cache for loader

    alt Cache Hit
        Cache-->>GetClass: Return cached loader
    else Cache Miss
        GetClass->>GetClass: Import from settings.OSCAR_DYNAMIC_CLASS_LOADER
        GetClass->>Cache: Cache loader
    end

    GetClass->>Loader: Load class
    Loader->>AppRegistry: Find app matching 'catalogue'
    AppRegistry-->>Loader: Return app config list

    Loader->>Loader: Parse module path (app='catalogue', module='forms')

    Loader->>ForkedApp: Try import from 'yourproject.catalogue.forms'

    alt Class Found in Forked App
        ForkedApp-->>Loader: Return ProductForm class
        Loader-->>GetClass: Return class
        GetClass-->>Code: Return ProductForm (forked version)
    else Not Found in Forked App
        ForkedApp-->>Loader: ImportError or AttributeError
        Loader->>CoreApp: Fallback to 'oscar.apps.catalogue.forms'

        alt Class Found in Core
            CoreApp-->>Loader: Return ProductForm class
            Loader-->>GetClass: Return class
            GetClass-->>Code: Return ProductForm (core version)
        else Not Found Anywhere
            CoreApp-->>Loader: ImportError
            Loader-->>GetClass: Raise ClassNotFoundError
            GetClass-->>Code: Exception raised
        end
    end
```

### Product Search Flow

How product search works through Haystack integration.

```mermaid
sequenceDiagram
    actor Customer
    participant SearchView as Search View
    participant SearchForm as Search Form
    participant Haystack as Haystack
    participant SearchBackend as Search Backend (ES/Solr)
    participant ProductIndex as Product Search Index
    participant Strategy as Pricing Strategy
    participant DB as Database

    Customer->>SearchView: Enter search query
    SearchView->>SearchForm: Validate search parameters
    SearchForm-->>SearchView: Valid query

    SearchView->>Haystack: Build search query
    Haystack->>ProductIndex: Get indexed fields
    ProductIndex-->>Haystack: Return searchable fields

    Haystack->>Haystack: Build query with filters:<br/>- Text search<br/>- Category filter<br/>- Price range<br/>- Facets

    Haystack->>SearchBackend: Execute search query
    SearchBackend->>SearchBackend: Full-text search on:<br/>- Title<br/>- Description<br/>- Attributes<br/>- Category

    SearchBackend->>SearchBackend: Apply filters and facets
    SearchBackend-->>Haystack: Return search results (product IDs)

    Haystack->>DB: Query full product objects
    DB-->>Haystack: Return product instances

    Haystack->>Strategy: Annotate with pricing/availability
    loop For each product
        Strategy->>DB: Get stock records
        DB-->>Strategy: Return stock info
        Strategy->>Strategy: Calculate price & availability
    end
    Strategy-->>Haystack: Products with pricing

    Haystack->>Haystack: Paginate results
    Haystack-->>SearchView: Return SearchQuerySet

    SearchView->>SearchView: Render template with:<br/>- Products<br/>- Facets<br/>- Pagination
    SearchView-->>Customer: Display search results
```

### Stock Allocation and Order Fulfillment

Managing inventory during order placement and fulfillment.

```mermaid
sequenceDiagram
    participant OrderCreator as Order Creator
    participant Order as Order Model
    participant OrderLine as Order Line
    participant StockRecord as Stock Record
    participant Partner as Partner
    participant EventHandler as Event Handler
    participant DB as Database

    OrderCreator->>Order: Create order from basket

    loop For each basket line
        OrderCreator->>OrderLine: Create order line
        OrderLine->>StockRecord: Allocate stock (quantity)

        StockRecord->>DB: Begin transaction
        StockRecord->>StockRecord: Check num_allocated + quantity <= num_in_stock

        alt Sufficient Stock
            StockRecord->>StockRecord: num_allocated += quantity
            StockRecord->>DB: UPDATE stock_record
            DB-->>StockRecord: Success

            StockRecord->>EventHandler: Trigger stock_allocated event
            EventHandler->>EventHandler: Log allocation
            EventHandler-->>StockRecord: Event processed

            StockRecord-->>OrderLine: Stock allocated
            OrderLine-->>OrderCreator: Line created
        else Insufficient Stock
            StockRecord->>DB: Rollback transaction
            StockRecord-->>OrderLine: Raise InvalidStockAdjustment
            OrderLine-->>OrderCreator: Stock allocation failed
            OrderCreator->>Order: Delete partially created order
            OrderCreator->>OrderCreator: Raise UnableToPlaceOrder
        end
    end

    OrderCreator->>Order: Save complete order
    Order->>DB: Commit transaction
    DB-->>Order: Order saved

    Note over Order,Partner: Later: Order fulfillment process

    Partner->>Order: Mark order line as shipped
    Order->>OrderLine: Update status = 'Shipped'
    OrderLine->>StockRecord: Consume allocated stock

    StockRecord->>DB: Begin transaction
    StockRecord->>StockRecord: num_allocated -= quantity
    StockRecord->>StockRecord: num_in_stock -= quantity
    StockRecord->>DB: UPDATE stock_record
    DB-->>StockRecord: Success

    StockRecord->>EventHandler: Trigger stock_consumed event
    EventHandler-->>StockRecord: Event logged

    StockRecord-->>OrderLine: Stock consumed
    OrderLine-->>Order: Line updated
    Order->>DB: Commit transaction
```

---

## Key Architectural Patterns

### 1. Abstract Model Pattern
Oscar uses abstract base models that can be extended by forking apps. This allows adding custom fields without modifying core code.

**Example**: `AbstractProduct` in `catalogue.abstract_models` → Extended in forked app → Concrete `Product` model

### 2. Dynamic Class Loading
Almost all classes are loaded dynamically using `get_class()` and `get_model()`, enabling customization by module path resolution.

**Resolution order**: Forked App → Core App → Raise Exception

### 3. Strategy Pattern
Used extensively for:
- **Pricing Strategy**: How product prices are determined
- **Availability Policy**: How stock availability is checked
- **Shipping Methods**: How shipping costs are calculated
- **Payment Handlers**: How payments are processed

### 4. Repository Pattern
The Shipping Repository provides available shipping methods based on basket and customer context.

### 5. Signal-Based Events
Django signals are used throughout for extensibility:
- `order_placed` - When an order is successfully created
- `start_checkout` - When checkout process begins
- `basket_addition` - When product added to basket
- `order_line_status_changed` - When order line status changes

### 6. Session-Based State Management
Checkout process uses session storage to maintain state across multiple request/response cycles.

---

## Database Schema Overview

### Core Entities and Relationships

```mermaid
erDiagram
    PRODUCT ||--o{ PRODUCT_ATTRIBUTE_VALUE : has
    PRODUCT ||--o{ STOCK_RECORD : has
    PRODUCT }o--|| PRODUCT_CLASS : "belongs to"
    PRODUCT }o--o{ CATEGORY : "in categories"
    PRODUCT ||--o{ PRODUCT_IMAGE : has

    BASKET ||--o{ BASKET_LINE : contains
    BASKET }o--o| USER : "owned by"
    BASKET }o--o{ VOUCHER : "has applied"
    BASKET_LINE }o--|| PRODUCT : references

    ORDER ||--o{ ORDER_LINE : contains
    ORDER }o--o| USER : "placed by"
    ORDER }o--|| BASKET : "created from"
    ORDER }o--o| SHIPPING_ADDRESS : has
    ORDER }o--o| BILLING_ADDRESS : has
    ORDER_LINE }o--|| PRODUCT : references
    ORDER_LINE }o--o| STOCK_RECORD : "allocated from"

    STOCK_RECORD }o--|| PARTNER : "managed by"
    STOCK_RECORD }o--|| PRODUCT : "for product"

    VOUCHER }o--o{ CONDITIONAL_OFFER : "triggers"
    CONDITIONAL_OFFER ||--|| CONDITION : has
    CONDITIONAL_OFFER ||--|| BENEFIT : has

    USER ||--o{ USER_ADDRESS : has

    PRODUCT {
        int id PK
        string upc
        string title
        text description
        int product_class_id FK
        datetime date_created
    }

    BASKET {
        int id PK
        int owner_id FK
        string status
        datetime date_created
    }

    ORDER {
        int id PK
        string number UK
        int user_id FK
        int basket_id FK
        decimal total_incl_tax
        string status
        datetime date_placed
    }

    STOCK_RECORD {
        int id PK
        int product_id FK
        int partner_id FK
        int num_in_stock
        int num_allocated
        decimal price
    }
```

---

## Conclusion

Django Oscar's architecture is built around flexibility and customization. The dynamic class loading system, abstract models, and forking mechanism allow developers to customize any part of the framework without modifying core code. This makes it suitable for a wide range of e-commerce scenarios, from simple shops to complex B2B platforms.

Key architectural strengths:
- **Extensibility**: Fork and customize any app without touching core code
- **Modularity**: Clear separation of concerns across apps
- **Flexibility**: Strategy pattern for pricing, shipping, payment
- **Domain-driven**: Models reflect e-commerce domain concepts
- **Django-native**: Leverages Django's app system, signals, and ORM

For implementation details, refer to the source code in `src/oscar/` and the official documentation.
