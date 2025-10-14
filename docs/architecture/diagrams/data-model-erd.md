# Django Oscar Data Model - Entity Relationship Diagram

## Core Data Model Overview

This document provides entity relationship diagrams for Django Oscar's core data model, organized by domain.

## Complete Entity Relationship Diagram

```mermaid
erDiagram
    %% Product Catalogue Domain
    ProductClass ||--o{ Product : "has"
    ProductClass ||--o{ ProductAttribute : "defines"
    ProductClass ||--o{ Option : "has"

    Product ||--o{ Product : "parent-children"
    Product ||--o{ ProductImage : "has"
    Product ||--o{ ProductAttributeValue : "has"
    Product ||--o{ ProductRecommendation : "recommends"
    Product ||--o{ StockRecord : "has"
    Product }o--o{ Category : "belongs to"
    Product }o--o{ Option : "has"

    ProductAttribute ||--o{ ProductAttributeValue : "values"
    ProductAttribute }o--|| AttributeOptionGroup : "uses"

    AttributeOptionGroup ||--o{ AttributeOption : "contains"

    Category ||--o{ Category : "parent-children"

    %% Partner Domain
    Partner ||--o{ StockRecord : "manages"
    Partner ||--o{ PartnerAddress : "has"
    Partner }o--o{ User : "users"

    StockRecord ||--o{ StockAlert : "triggers"

    %% Basket Domain
    User ||--o{ Basket : "owns"
    Basket ||--o{ BasketLine : "contains"
    BasketLine }o--|| Product : "references"
    BasketLine ||--o{ BasketLineAttribute : "has"

    Basket }o--o{ Voucher : "applies"

    %% Order Domain
    User ||--o{ Order : "places"
    Basket ||--o| Order : "converts to"

    Order ||--o{ OrderLine : "contains"
    Order ||--o{ OrderNote : "has"
    Order ||--o{ OrderDiscount : "has"
    Order ||--o{ OrderStatusChange : "tracks"
    Order ||--o{ PaymentEvent : "has"
    Order ||--o{ ShippingEvent : "has"
    Order ||--o{ Surcharge : "has"
    Order }o--|| ShippingAddress : "ships to"
    Order }o--|| BillingAddress : "bills to"

    OrderLine }o--|| Product : "references"
    OrderLine }o--|| StockRecord : "fulfills from"
    OrderLine }o--|| Partner : "fulfilled by"
    OrderLine ||--o{ LineAttribute : "has"
    OrderLine ||--o{ OrderLineDiscount : "has"

    PaymentEvent }o--o{ OrderLine : "affects"
    ShippingEvent }o--o{ OrderLine : "affects"

    %% Offer and Voucher Domain
    ConditionalOffer ||--o{ OrderDiscount : "creates"
    Voucher ||--o{ OrderDiscount : "creates"
    Voucher }o--|| ConditionalOffer : "linked to"

    ConditionalOffer ||--|| Benefit : "provides"
    ConditionalOffer ||--|| Condition : "requires"

    Benefit }o--|| Range : "applies to"
    Condition }o--|| Range : "applies to"

    Range }o--o{ Product : "includes"

    %% Customer Domain
    User ||--o{ UserAddress : "has"
    User ||--o{ WishList : "has"

    WishList ||--o{ WishListLine : "contains"
    WishListLine }o--|| Product : "wants"

    %% Analytics Domain
    User ||--o{ UserRecord : "tracked by"
    Product ||--o{ ProductRecord : "tracked by"
    User ||--o{ UserProductView : "viewed"
    Product ||--o{ UserProductView : "viewed by"

    %% Product Domain Entities
    ProductClass {
        string name
        string slug
        boolean requires_shipping
        boolean track_stock
    }

    Product {
        string structure
        string upc
        string title
        string slug
        text description
        string meta_title
        text meta_description
        float rating
        boolean is_public
        boolean is_discountable
        datetime date_created
        datetime date_updated
    }

    ProductAttribute {
        string name
        string code
        string type
        boolean required
    }

    ProductAttributeValue {
        text value_text
        int value_integer
        boolean value_boolean
        float value_float
        date value_date
    }

    Category {
        string name
        string code
        text description
        string slug
        string path
        int depth
        boolean is_public
    }

    ProductImage {
        string code
        image original
        string caption
        int display_order
        datetime date_created
    }

    Option {
        string name
        string code
        string type
        boolean required
    }

    AttributeOptionGroup {
        string name
        string code
    }

    AttributeOption {
        string option
        string code
    }

    %% Partner Domain Entities
    Partner {
        string code
        string name
    }

    StockRecord {
        string partner_sku
        decimal price
        string price_currency
        int num_in_stock
        int num_allocated
        int low_stock_threshold
        datetime date_created
        datetime date_updated
    }

    StockAlert {
        int threshold
        string status
        datetime date_created
        datetime date_closed
    }

    PartnerAddress {
        text address
    }

    %% Basket Domain Entities
    Basket {
        string status
        datetime date_created
        datetime date_merged
        datetime date_submitted
    }

    BasketLine {
        int quantity
        decimal price_excl_tax
        decimal price_incl_tax
        datetime date_created
    }

    BasketLineAttribute {
        string option
        string value
    }

    %% Order Domain Entities
    Order {
        string number
        string currency
        decimal total_incl_tax
        decimal total_excl_tax
        decimal shipping_incl_tax
        decimal shipping_excl_tax
        string status
        string guest_email
        datetime date_placed
        boolean analytics_tracked
    }

    OrderLine {
        string partner_sku
        string title
        string upc
        int quantity
        decimal line_price_incl_tax
        decimal line_price_excl_tax
        decimal unit_price_incl_tax
        decimal unit_price_excl_tax
        string status
        int num_allocated
        boolean allocation_cancelled
    }

    OrderNote {
        string note_type
        text message
        datetime date_created
    }

    OrderDiscount {
        string category
        int offer_id
        string offer_name
        int voucher_id
        string voucher_code
        int frequency
        decimal amount
    }

    OrderStatusChange {
        string old_status
        string new_status
        datetime date_created
    }

    ShippingAddress {
        string title
        string first_name
        string last_name
        string line1
        string line2
        string line3
        string line4
        string state
        string postcode
        string country
    }

    BillingAddress {
        string title
        string first_name
        string last_name
        string line1
        string line2
        string line3
        string line4
        string state
        string postcode
        string country
    }

    PaymentEvent {
        decimal amount
        string reference
        datetime date_created
    }

    ShippingEvent {
        text notes
        datetime date_created
    }

    Surcharge {
        string name
        string code
        decimal incl_tax
        decimal excl_tax
    }

    %% Offer Domain Entities
    ConditionalOffer {
        string name
        string slug
        text description
        string offer_type
        string status
        int max_basket_applications
        int max_user_applications
        int max_global_applications
        date start_datetime
        date end_datetime
    }

    Benefit {
        string type
        decimal value
        int max_affected_items
    }

    Condition {
        string type
        decimal value
    }

    Range {
        string name
        text description
        boolean includes_all_products
    }

    Voucher {
        string name
        string code
        int usage
        date start_datetime
        date end_datetime
    }

    %% Customer Domain Entities
    User {
        string username
        string email
        string first_name
        string last_name
        boolean is_active
        datetime date_joined
    }

    UserAddress {
        string title
        string first_name
        string last_name
        string line1
        string line2
        string line3
        string line4
        string state
        string postcode
        string country
        boolean is_default_for_shipping
        boolean is_default_for_billing
    }

    WishList {
        string name
        string visibility
        datetime date_created
    }

    WishListLine {
        int quantity
        string title
        datetime date_added
    }

    %% Analytics Domain Entities
    ProductRecord {
        int num_views
        int num_basket_additions
        int num_purchases
        float score
    }

    UserRecord {
        int num_product_views
        int num_basket_additions
        int num_orders
        int num_order_lines
        int num_order_items
        decimal total_spent
        datetime date_last_order
    }

    UserProductView {
        datetime date_created
    }
```

## Domain-Specific ERDs

### 1. Catalogue Domain

```mermaid
erDiagram
    ProductClass ||--o{ Product : "categorizes"
    ProductClass ||--o{ ProductAttribute : "defines attributes"
    Product ||--o{ Product : "parent/child"
    Product ||--o{ ProductImage : "has images"
    Product ||--o{ ProductAttributeValue : "has values"
    Product }o--o{ Category : "belongs to"
    ProductAttribute ||--o{ ProductAttributeValue : "stores values"
    Category ||--o{ Category : "hierarchical"

    ProductClass {
        int id PK
        string name
        string slug UK
        boolean requires_shipping
        boolean track_stock
    }

    Product {
        int id PK
        string structure
        int parent_id FK
        int product_class_id FK
        string upc UK
        string title
        string slug
        text description
        boolean is_public
        boolean is_discountable
        datetime date_created
        datetime date_updated
    }

    Category {
        int id PK
        string name
        string code UK
        string slug
        string path
        int depth
        boolean is_public
        boolean ancestors_are_public
    }

    ProductImage {
        int id PK
        int product_id FK
        string code UK
        image original
        string caption
        int display_order
    }

    ProductAttribute {
        int id PK
        int product_class_id FK
        string name
        string code
        string type
        boolean required
    }

    ProductAttributeValue {
        int id PK
        int product_id FK
        int attribute_id FK
        text value_text
        int value_integer
        float value_float
        boolean value_boolean
    }
```

**Key Relationships**:
- Product can be standalone, parent (with children), or child (of parent)
- ProductClass defines the type of product and its attributes
- Categories form a tree structure (using django-treebeard)
- ProductAttributeValue uses EAV pattern for flexible attributes

### 2. Partner and Inventory Domain

```mermaid
erDiagram
    Partner ||--o{ StockRecord : "manages stock"
    Product ||--o{ StockRecord : "has stock records"
    StockRecord ||--o{ StockAlert : "triggers alerts"
    Partner ||--o{ PartnerAddress : "has addresses"

    Partner {
        int id PK
        string code UK
        string name
    }

    StockRecord {
        int id PK
        int product_id FK
        int partner_id FK
        string partner_sku
        decimal price
        string price_currency
        int num_in_stock
        int num_allocated
        int low_stock_threshold
        datetime date_updated
    }

    StockAlert {
        int id PK
        int stockrecord_id FK
        int threshold
        string status
        datetime date_created
        datetime date_closed
    }

    PartnerAddress {
        int id PK
        int partner_id FK
        text address_data
    }
```

**Key Concepts**:
- **Two-Stage Stock Allocation**:
  - `num_in_stock`: Total physical inventory
  - `num_allocated`: Reserved for orders not yet shipped
  - `net_stock_level = num_in_stock - num_allocated`
- Multiple partners can stock the same product
- Each partner has their own SKU and pricing

### 3. Basket Domain

```mermaid
erDiagram
    User ||--o{ Basket : "owns"
    Basket ||--o{ BasketLine : "contains"
    BasketLine }o--|| Product : "references"
    BasketLine ||--o{ BasketLineAttribute : "has attributes"
    Basket }o--o{ Voucher : "applies"

    User {
        int id PK
        string username UK
        string email
    }

    Basket {
        int id PK
        int owner_id FK
        string status
        datetime date_created
        datetime date_merged
        datetime date_submitted
    }

    BasketLine {
        int id PK
        int basket_id FK
        int product_id FK
        int stockrecord_id FK
        int quantity
        decimal price_excl_tax
        decimal price_incl_tax
        datetime date_created
    }

    BasketLineAttribute {
        int id PK
        int line_id FK
        int option_id FK
        string value
    }

    Voucher {
        int id PK
        string code UK
        int usage
        date start_datetime
        date end_datetime
    }
```

**Key Features**:
- Anonymous users have baskets stored in session
- Baskets merge when user logs in
- Line attributes store product options (e.g., personalized text)
- Vouchers applied at basket level

### 4. Order Domain

```mermaid
erDiagram
    User ||--o{ Order : "places"
    Basket ||--o| Order : "becomes"
    Order ||--o{ OrderLine : "contains"
    Order ||--o{ OrderDiscount : "has"
    Order ||--o{ PaymentEvent : "tracks payments"
    Order ||--o{ ShippingEvent : "tracks shipments"
    Order }o--|| ShippingAddress : "ships to"
    Order }o--|| BillingAddress : "bills to"

    OrderLine }o--|| Product : "references"
    OrderLine }o--|| StockRecord : "from"
    OrderLine }o--|| Partner : "fulfilled by"

    PaymentEvent }o--o{ OrderLine : "affects"
    ShippingEvent }o--o{ OrderLine : "affects"

    Order {
        int id PK
        string number UK
        int user_id FK
        int basket_id FK
        int shipping_address_id FK
        int billing_address_id FK
        string currency
        decimal total_incl_tax
        decimal total_excl_tax
        decimal shipping_incl_tax
        decimal shipping_excl_tax
        string status
        datetime date_placed
    }

    OrderLine {
        int id PK
        int order_id FK
        int product_id FK
        int stockrecord_id FK
        int partner_id FK
        string partner_sku
        string title
        int quantity
        decimal line_price_incl_tax
        decimal line_price_excl_tax
        string status
        int num_allocated
    }

    PaymentEvent {
        int id PK
        int order_id FK
        int event_type_id FK
        decimal amount
        string reference
        datetime date_created
    }

    ShippingEvent {
        int id PK
        int order_id FK
        int event_type_id FK
        text notes
        datetime date_created
    }
```

**Key Characteristics**:
- Orders are immutable after creation
- Product information denormalized (preserves historical data)
- Status pipeline controls order state transitions
- Many-to-many between events and order lines (via "through" tables)

### 5. Offer and Voucher Domain

```mermaid
erDiagram
    ConditionalOffer ||--|| Benefit : "provides"
    ConditionalOffer ||--|| Condition : "requires"
    ConditionalOffer ||--o{ OrderDiscount : "creates"

    Voucher }o--|| ConditionalOffer : "triggers"
    Voucher ||--o{ OrderDiscount : "creates"

    Benefit }o--|| Range : "applies to"
    Condition }o--|| Range : "tests"
    Range }o--o{ Product : "includes"

    ConditionalOffer {
        int id PK
        string name
        string slug UK
        string offer_type
        string status
        int max_basket_applications
        int max_user_applications
        date start_datetime
        date end_datetime
    }

    Benefit {
        int id PK
        int range_id FK
        string type
        decimal value
        int max_affected_items
    }

    Condition {
        int id PK
        int range_id FK
        string type
        decimal value
    }

    Range {
        int id PK
        string name
        boolean includes_all_products
    }

    Voucher {
        int id PK
        int offer_id FK
        string code UK
        int usage
        date start_datetime
        date end_datetime
    }

    OrderDiscount {
        int id PK
        int order_id FK
        int offer_id FK
        int voucher_id FK
        string category
        decimal amount
    }
```

**Offer Types**:
- **Site offers**: Apply automatically to all qualifying baskets
- **Voucher offers**: Require a voucher code
- **Session offers**: Temporary, for current session only

**Benefit Types**:
- Percentage discount
- Absolute discount
- Fixed price
- Free shipping
- Multibuy (e.g., 3 for 2)

## Key Indexing Strategy

### High-Priority Indexes

**Product**:
- `structure` - Filtering by product type
- `is_public`, `date_updated` - Public product queries
- `parent_id` - Parent-child lookups

**StockRecord**:
- `product_id`, `partner_id` - Joint lookup
- `date_updated` - Finding recently updated stock

**Order**:
- `number` - Order lookup
- `user_id`, `date_placed` - User order history
- `status`, `date_placed` - Order processing queries

**OrderLine**:
- `order_id` - Order lines lookup
- `product_id` - Product sales reports
- `partner_id` - Partner fulfillment queries

**Category**:
- `path` - Tree traversal (treebeard)
- `slug` - URL lookups

## Data Integrity Constraints

### Unique Constraints
- `Product.upc` - Universal Product Code
- `StockRecord.(partner, partner_sku)` - Partner's SKU
- `Order.number` - Order identifier
- `Voucher.code` - Voucher code

### Check Constraints
- Stock quantities >= 0
- Prices >= 0
- Order totals consistent with line totals

### Foreign Key Cascades
- `Product` deletion → CASCADE to `StockRecord`
- `Order` deletion → CASCADE to `OrderLine`
- `Product` deletion → SET_NULL in `OrderLine` (preserves orders)
- `User` deletion → SET_NULL in `Order` (preserves orders)

## Historical Data Preservation

### Denormalized Data in Orders
To preserve historical information, orders store copies of:
- Product title and SKU
- Partner name and SKU
- Prices at time of purchase
- Discount information

This ensures:
- Order display remains accurate even if products/partners are deleted
- Accurate historical reporting
- Audit compliance

## Performance Optimizations

### Select Related Usage
```python
Order.objects.select_related(
    'user', 'billing_address', 'shipping_address'
)
```

### Prefetch Related Usage
```python
Order.objects.prefetch_related(
    'lines__product__images',
    'shipping_events__event_type'
)
```

### Denormalization
- Order totals stored (not calculated on each view)
- Product rating cached
- Category full_slug cached
