# Django Oscar Architecture Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Principles](#architecture-principles)
3. [High-Level Architecture](#high-level-architecture)
4. [Core Subsystems](#core-subsystems)
5. [Dynamic Class Loading System](#dynamic-class-loading-system)
6. [Abstract Models Pattern](#abstract-models-pattern)
7. [Data Model Overview](#data-model-overview)
8. [Key Design Patterns](#key-design-patterns)
9. [Extension Points](#extension-points)

## System Overview

Django Oscar is a domain-driven e-commerce framework built on Django. It provides a complete e-commerce solution while maintaining flexibility for customization without requiring developers to fork the core codebase.

### Key Characteristics

- **Domain-Driven Design**: Organized around e-commerce business domains (catalogue, basket, order, partner, etc.)
- **Customization-First**: Built to be extended and customized at every layer
- **Abstract Models**: All data models are abstract base classes for easy extension
- **Dynamic Class Loading**: Runtime class resolution allows swapping implementations
- **Multi-Vendor**: Native support for multiple fulfillment partners
- **Flexible Pricing**: Strategy pattern for pricing and availability logic

## Architecture Principles

### 1. **Separation of Concerns**
Each app focuses on a specific business domain with clear boundaries.

### 2. **Open for Extension, Closed for Modification**
Core Oscar code should not need to be modified. All customization happens through:
- Subclassing abstract models
- Overriding views and forms
- Custom templates
- Dynamic class loading

### 3. **Convention Over Configuration**
Follows Django conventions with additional Oscar-specific patterns for predictable structure.

### 4. **Pluggability**
Apps can be used independently or together, with clear interfaces between them.

## High-Level Architecture

Django Oscar follows a layered architecture:

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│  (Templates, Views, Forms, Dashboard, Customer-facing)  │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────┐
│                   Application Layer                      │
│        (Business Logic, Workflows, Strategies)           │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────┐
│                     Domain Layer                         │
│        (Models, Services, Domain Logic)                  │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────┐
│                  Infrastructure Layer                    │
│      (Django ORM, Cache, Search, External APIs)          │
└─────────────────────────────────────────────────────────┘
```

## Core Subsystems

Django Oscar is organized into domain-specific apps:

### Catalogue Management
- **catalogue**: Product catalog, categories, product classes, attributes
- **partner**: Fulfillment partners, stock records, pricing

### Shopping Experience
- **basket**: Shopping cart functionality
- **checkout**: Checkout flow and session management
- **customer**: User accounts, wishlists, addresses

### Order Management
- **order**: Order processing, order lines, status management
- **payment**: Payment event tracking
- **shipping**: Shipping methods, events, and charges

### Promotions & Marketing
- **offer**: Conditional offers and promotions
- **voucher**: Voucher/coupon management
- **analytics**: Product views and tracking

### Administration
- **dashboard**: Admin interface for store management
- **communication**: Email templates and notifications

### Supporting
- **address**: Address management
- **search**: Search functionality integration
- **wishlists**: Customer wishlist management

## Dynamic Class Loading System

### Purpose
Allows Oscar to look up classes from your local apps first, then fall back to Oscar's core implementations.

### Key Functions

#### `get_class(module_path, class_name)`
Dynamically imports classes from either your local app or Oscar's core.

```python
from oscar.core.loading import get_class

ProductForm = get_class('catalogue.forms', 'ProductForm')
# Looks in:
# 1. your_project.catalogue.forms.ProductForm
# 2. oscar.apps.catalogue.forms.ProductForm
```

#### `get_model(app_label, model_name)`
Fetches models from Django's app registry.

```python
from oscar.core.loading import get_model

Product = get_model('catalogue', 'Product')
```

### Search Order
1. Check INSTALLED_APPS for a matching app with the class
2. Fall back to Oscar's core implementation
3. Raise ImportError if not found

This enables:
- Overriding individual classes without forking
- Gradual customization of the framework
- Maintaining compatibility with Oscar updates

## Abstract Models Pattern

### Structure
Every Oscar app follows this pattern:

```
oscar/apps/catalogue/
├── abstract_models.py     # Abstract base classes
├── models.py              # Concrete models (inherit from abstract)
├── views.py
├── forms.py
└── ...
```

### Example: Product Model

**oscar/apps/catalogue/abstract_models.py**
```python
class AbstractProduct(models.Model):
    title = models.CharField(max_length=255)
    slug = SlugField(max_length=255)
    description = models.TextField()
    # ... more fields and methods

    class Meta:
        abstract = True
```

**oscar/apps/catalogue/models.py**
```python
from oscar.apps.catalogue.abstract_models import AbstractProduct

class Product(AbstractProduct):
    pass  # Can add customizations here
```

### Customization Pattern

To customize a model in your project:

```python
# your_project/catalogue/models.py
from oscar.apps.catalogue.abstract_models import AbstractProduct

class Product(AbstractProduct):
    # Add custom fields
    custom_field = models.CharField(max_length=100)

    # Override methods
    def get_title(self):
        return f"Custom: {self.title}"

# CRITICAL: Import Oscar's other models
from oscar.apps.catalogue.models import *  # noqa
```

### Benefits
- Extend models without modifying Oscar code
- Maintain upgrade path to newer Oscar versions
- Add fields, methods, and behaviors specific to your business

## Data Model Overview

### Core Entities and Relationships

#### Product Domain
```
ProductClass ──┬── Product ──┬── ProductImage
               │             ├── ProductAttributeValue
               └── ProductAttribute

Product ─── ProductCategory ─── Category (Tree Structure)
     │
     └── StockRecord ─── Partner
```

#### Order Domain
```
User ──┬── Basket ──── BasketLine ──── Product
       │
       └── Order ──┬── OrderLine ──── StockRecord
                   ├── ShippingAddress
                   ├── BillingAddress
                   ├── OrderDiscount
                   └── ShippingEvent
```

#### Partner Domain
```
Partner ──┬── StockRecord ──┬── Product
          │                 ├── num_in_stock
          │                 ├── num_allocated
          │                 └── price
          │
          └── PartnerAddress
```

### Key Model Characteristics

#### Product
- Three structures: Standalone, Parent, Child
- Parent products have child products (variants)
- Attributes defined by ProductClass
- Categories via many-to-many relationship

#### StockRecord
- Links Product to Partner
- Manages inventory (num_in_stock, num_allocated)
- Pricing information per partner
- Two-stage stock allocation: allocate → consume

#### Order
- Immutable after creation
- Status pipeline for workflow management
- Separate billing and shipping addresses
- Links back to basket for audit trail

#### OrderLine
- Denormalized product information
- Partner and stockrecord references
- Price information (before/after discounts)
- Allocation tracking

## Key Design Patterns

### 1. Strategy Pattern
**Used for**: Pricing and availability logic

```python
class Strategy:
    def fetch_for_product(self, product):
        # Returns availability and pricing info
        pass
```

Different strategies can be swapped based on:
- Customer group
- Geographic location
- Business rules

### 2. Template Method Pattern
**Used for**: Checkout flow

Base checkout views define the flow, subclasses implement specific steps.

### 3. Repository Pattern
**Used for**: Complex queries and data access

Custom managers and querysets encapsulate data access logic.

```python
class ProductQuerySet(models.QuerySet):
    def browsable(self):
        return self.filter(is_public=True)

    def base_queryset(self):
        # Complex query optimization
        return self.select_related(...)
```

### 4. Pipeline Pattern
**Used for**: Order status transitions

```python
OSCAR_ORDER_STATUS_PIPELINE = {
    'Pending': ('Being processed', 'Cancelled'),
    'Being processed': ('Complete', 'Cancelled'),
    'Cancelled': (),
    'Complete': (),
}
```

### 5. Signals
**Used for**: Event-driven side effects

```python
order_placed = django.dispatch.Signal()
order_status_changed = django.dispatch.Signal()
```

## Extension Points

### 1. Model Customization
- Inherit from Abstract* models
- Add fields, methods, properties
- Maintain migrations from Oscar

### 2. View Overrides
- Subclass Oscar views
- Use get_class() for dynamic loading
- Override specific methods

### 3. Form Customization
- Subclass Oscar forms
- Add validation logic
- Customize widgets

### 4. Template Overrides
- Place templates at same path in your project
- Django's template loader finds yours first
- Can extend Oscar templates with `{% extends %}`

### 5. URL Configuration
- Oscar uses application instances
- Override per-app URL configuration
- Maintain URL patterns

### 6. Strategies
- Implement pricing strategies
- Custom availability logic
- Partner selection logic

### 7. Dashboard
- Add custom dashboard views
- Extend existing dashboard sections
- Custom permissions

## Architecture Diagrams

See the `diagrams/` directory for:
- C4 Context Diagram: System context and external actors
- C4 Container Diagram: High-level application architecture
- C4 Component Diagram: Internal component structure
- UML Class Diagram: Core data model relationships

## Best Practices

### When Extending Oscar

1. **Always use abstract models** - Never directly modify Oscar models
2. **Use get_class()** - For views, forms, and other classes
3. **Copy migrations** - When customizing models
4. **Follow Oscar's conventions** - For predictable behavior
5. **Test extensively** - Especially when overriding core logic
6. **Keep it simple** - Only customize what you need

### Migration Strategy

1. Fork the Oscar app to your project
2. Copy existing migrations from Oscar
3. Create new migrations for your changes
4. Update INSTALLED_APPS to use your app
5. Maintain migration dependencies

## Performance Considerations

### Database Queries
- Oscar uses select_related() and prefetch_related() extensively
- Custom querysets optimize common access patterns
- Be careful when adding fields that require joins

### Caching
- Category URLs are cached
- Product availability can be cached
- Dashboard uses caching for performance

### Search
- Uses Haystack for search abstraction
- Supports Whoosh, Solr, Elasticsearch
- Separate search indexes from database queries

## Security Considerations

- CSRF protection on all forms
- Permission-based dashboard access
- Order verification hashes
- No hard links between orders and product catalog
- Separate guest checkout flow

## Deployment Architecture

### Typical Production Setup
```
Load Balancer
     │
     ├── Web Server (nginx/Apache)
     │        │
     │        └── WSGI Server (Gunicorn/uWSGI)
     │                 │
     │                 └── Django Oscar
     │
     ├── Static Files (CDN/S3)
     ├── Media Files (CDN/S3)
     ├── Database (PostgreSQL)
     ├── Cache (Redis/Memcached)
     ├── Search (Elasticsearch/Solr)
     └── Task Queue (Celery + Redis)
```

## Further Reading

- [CLAUDE.md](../../CLAUDE.md) - Development guide for this project
- [Django Oscar Documentation](https://django-oscar.readthedocs.io/)
- [C4 Model](https://c4model.com/) - For understanding the diagrams
