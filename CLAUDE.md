# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Django Oscar is a domain-driven e-commerce framework for Django. It uses abstract models and dynamic class loading to allow extensive customization without forking the core codebase.

## Architecture

### Dynamic Class Loading System

Oscar uses a sophisticated class loading mechanism (`oscar.core.loading`) that allows customization without forking:

- **get_class()**: Dynamically imports classes from either your local app or Oscar's core
- **get_model()**: Fetches models from the Django app registry
- Classes are looked up first in your local apps, then fall back to Oscar's core implementations

Example: `get_class('catalogue.forms', 'ProductForm')` will import from your custom `catalogue/forms.py` if it exists, otherwise from `oscar.apps.catalogue.forms`.

### Abstract Models Pattern

All Oscar models are defined as abstract base classes in `abstract_models.py` files within each app:

- Abstract models in `oscar/apps/{app}/abstract_models.py` define base functionality
- Concrete models in `oscar/apps/{app}/models.py` inherit from these abstracts
- To customize: create your own app with the same label, subclass the abstract model, add fields/methods, then import Oscar's remaining models with `from oscar.apps.{app}.models import *` at the bottom

**Critical**: When customizing models, always import Oscar's models AFTER defining your custom ones to ensure Django registers yours first.

### App Structure

Core e-commerce apps in `src/oscar/apps/`:
- **catalogue**: Products, categories, product classes, product attributes
- **partner**: Partners, stock records, stock alerts
- **basket**: Shopping basket functionality
- **order**: Order processing and management
- **customer**: User accounts, addresses, wishlists
- **checkout**: Checkout flow and session management
- **payment**: Payment event tracking
- **shipping**: Shipping methods and charges
- **offer**: Conditional offers and vouchers
- **voucher**: Voucher management
- **analytics**: Product analytics and tracking
- **dashboard**: Admin interface for managing the store
- **search**: Search functionality (uses Haystack)

### URL and View Configuration

Oscar apps use `OscarConfig` (extends Django's AppConfig) to handle:
- URL routing at the app level
- Permissions configuration
- View registration

See `oscar/config.py` for the main URL assembly.

## Development Commands

### Setup and Installation
```bash
# Create virtual environment and install dependencies
make venv

# Install for local development (without venv setup)
make install

# Install static assets (npm dependencies and build)
make assets

# Alternative: Just install requirements
pip install -e .[test]
```

### Running Tests
```bash
# Run all tests using pytest
make test
# Or directly:
./venv/bin/py.test

# Run specific test file
./venv/bin/py.test tests/integration/partner/test_stock_allocation_distributed.py -v

# Run failed tests only
make retest

# Generate coverage report
make coverage

# Test with specific database (default is PostgreSQL)
# For SQLite in memory:
export DATABASE_ENGINE=django.db.backends.sqlite3
export DATABASE_NAME=:memory:
./venv/bin/py.test
```

**Test Configuration**: Tests use `tests/settings.py` and `tests/conftest.py`. Default database is PostgreSQL but can be overridden with environment variables.

### Code Quality
```bash
# Run all linters (black, pylint, eslint)
make lint

# Auto-format Python code with black
make black

# Test migrations
make test_migrations
```

### Sandbox Environment
```bash
# Create a full sandbox site from scratch
make sandbox

# Clean and rebuild sandbox
make build_sandbox

# Individual sandbox setup steps:
make sandbox_clean        # Remove database and media
make sandbox_load_user    # Load user fixtures
make sandbox_load_data    # Import catalog and other fixtures

# Run sandbox server
cd sandbox && python manage.py runserver
```

The sandbox is a demo e-commerce site in `sandbox/` with fixtures for testing.

### Frontend Assets
```bash
# Build CSS and JavaScript
npm run build

# Watch for changes and rebuild
npm run watch

# Run ESLint on JavaScript
npm run eslint
```

**Build Process**: Oscar uses Gulp to compile SCSS and copy frontend assets from `src/oscar/static_src/` to `src/oscar/static/`.

### Documentation
```bash
# Build documentation
make docs
```

### Translations
```bash
# Extract translatable strings
make extract_translations

# Compile translation files
make compile_translations
```

## Customizing Oscar

### Forking an App

1. Create a new app directory with the same label: `myproject/{app_label}/`
2. Create `models.py`, `admin.py`, and any other modules you want to override
3. Update `INSTALLED_APPS` to use your app instead of Oscar's
4. For models: inherit from `oscar.apps.{app}.abstract_models.Abstract{Model}`, add your customizations, then `from oscar.apps.{app}.models import *` at the end
5. Copy migrations from Oscar's app and create new migrations for your changes

### Overriding Views and Forms

Use `get_class()` to import and subclass:
```python
from oscar.core.loading import get_class

ProductForm = get_class('dashboard.catalogue.forms', 'ProductForm')

class CustomProductForm(ProductForm):
    # Your customizations
```

### Template Customization

Django's template loader looks in your project's template directories first, so create templates at the same path to override Oscar's.

## Testing Patterns

- Tests are organized in `tests/` with subdirectories: `unit/`, `integration/`, `functional/`
- Uses pytest with pytest-django plugin
- Factory Boy is used for test factories (`factory-boy` package)
- Test settings in `tests/settings.py` use custom apps from `tests/_site/apps/` to test model customization

## Database

- Primary support: PostgreSQL
- Also works with: SQLite (for development/testing), MySQL
- Uses Django migrations for all schema changes
- When customizing models, copy Oscar's migrations to maintain dependency chain

## Key Settings

Important Oscar settings (see `oscar/defaults.py` for full list):
- `OSCAR_SHOP_NAME`: Your shop name
- `OSCAR_DEFAULT_CURRENCY`: Default currency code
- `OSCAR_ALLOW_ANON_CHECKOUT`: Allow anonymous checkout
- `OSCAR_INITIAL_ORDER_STATUS`: Initial order status code
- `OSCAR_ORDER_STATUS_PIPELINE`: Valid order status transitions
- `OSCAR_HIDDEN_FEATURES`: List of Oscar features to disable
- `OSCAR_DYNAMIC_CLASS_LOADER`: Custom class loader (advanced)

## Deployment Notes

- Static files must be built with `npm run build` before deploying
- Set `DEBUG = False` in production
- Configure `ALLOWED_HOSTS`, `SECRET_KEY`, and database settings
- Use `python manage.py collectstatic` to gather static files
- Oscar uses Babel for currency formatting (requires locale data)
- Search requires Haystack configuration (Whoosh, Solr, or Elasticsearch)

## Linear Integration

**Default Project**: Oscar-Demo-Decomp-Test1
- Project ID: `0de7b52f-bec5-4d21-93e3-9f2763e3e2f9`
- Project URL: https://linear.app/cpuzzo/project/oscar-demo-decomp-test1-69d4ac454491

When creating or resolving issues for this repository, use the Oscar-Demo-Decomp-Test1 project by default unless otherwise specified.
