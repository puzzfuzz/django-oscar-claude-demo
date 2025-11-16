# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Django Oscar is a domain-driven e-commerce framework for Django. It's designed as a highly customizable framework where core functionality can be extended and customized to suit specific business domains.

## Development Commands

### Environment Setup
```bash
make install          # Install development and production requirements
make venv             # Create virtual environment and install requirements
npm install           # Install Node.js dependencies
make assets           # Install and build static assets
```

### Sandbox Environment
The sandbox is a working Oscar site used for local testing and development:
```bash
make sandbox          # Install requirements and create a complete sandbox
make build_sandbox    # Creates sandbox from scratch (cleans DB, loads fixtures)
make sandbox_clean    # Clean sandbox database and media files
```

### Testing
```bash
make test             # Run all tests using pytest
make retest           # Run only previously failed tests
make coverage         # Generate coverage report
pytest                # Run tests directly (venv required)
pytest -k test_name   # Run specific test by name
pytest path/to/test_file.py  # Run tests in specific file
tox                   # Run tests across multiple Python/Django versions
make test_migrations  # Test migrations against PostgreSQL
```

### Code Quality
```bash
make lint             # Run black, pylint, and other linters (check mode)
make black            # Format code with black
npm run eslint        # Run ESLint on JavaScript files
```

### Static Assets
```bash
npm run build         # Build CSS/JS assets using gulp
npm run watch         # Watch for changes and rebuild automatically
```

### Translations
```bash
make extract_translations  # Extract strings and create .po files
make compile_translations  # Compile .po files to .mo files
```

### Documentation
```bash
make docs             # Build documentation using Sphinx
```

### Migrations
Create migrations using the sandbox:
```bash
make sandbox
sandbox/manage.py makemigrations
```

## Architecture

### Core Design Principles

1. **Abstract Models**: Core models are defined as abstract classes in `abstract_models.py` files within each app. Concrete implementations in `models.py` can be overridden by forking apps.

2. **Dynamic Class Loading**: Almost all classes (models, views, forms, etc.) are dynamically loaded, allowing customization by overriding classes without modifying Oscar's core code.

3. **App-Based Structure**: Oscar is organized into discrete Django apps, each handling a specific domain:
   - `catalogue` - Product catalog and categories
   - `order` - Order management
   - `basket` - Shopping basket/cart
   - `checkout` - Checkout process
   - `customer` - Customer accounts
   - `partner` - Partner/vendor management
   - `shipping` - Shipping methods
   - `payment` - Payment handling
   - `offer` - Promotions and offers
   - `voucher` - Voucher codes
   - `address` - Address handling
   - `wishlists` - Customer wishlists
   - `search` - Product search (Haystack integration)
   - `analytics` - Analytics and reporting
   - `communication` - Email and notifications
   - `dashboard` - Admin dashboard (with sub-apps for each domain)

4. **Forking Apps for Customization**: Use `./manage.py oscar_fork_app <app_name> <target_folder>` to create a customizable copy of an Oscar app. Replace the Oscar app with your forked version in `INSTALLED_APPS`.

### Source Code Organization

```
src/oscar/
├── apps/              # Core Oscar applications
│   ├── catalogue/     # Product models, views, managers
│   ├── order/         # Order models and processing
│   ├── basket/        # Shopping basket functionality
│   ├── dashboard/     # Admin interface (has sub-apps)
│   └── ...
├── core/              # Core utilities and base classes
├── views/             # Base view classes
├── forms/             # Base form classes
├── models/            # Base model utilities
├── templates/         # Default templates
├── test/              # Test utilities and factories
└── utils/             # Utility functions
```

### Testing Structure

```
tests/
├── unit/              # Unit tests (isolated component testing)
├── integration/       # Integration tests (multiple components)
└── functional/        # Functional tests (end-to-end workflows)
```

Each test directory mirrors the app structure (e.g., `tests/unit/catalogue/`, `tests/integration/order/`).

### Static Assets

- Source files: `src/oscar/static_src/`
- Compiled output: `src/oscar/static/`
- Build tool: Gulp (configured in `gulpfile.js/`)
- CSS: Written in SASS, compiled to CSS
- Dependencies: Bootstrap 4, jQuery, Select2, TinyMCE

## Important Conventions

### Code Style
- Python: Black formatting (79 char line length for imports via isort)
- JavaScript: ESLint configuration in `.eslintrc.json`
- Migrations: Excluded from linting and formatting
- Flake8 max line length: 119 characters

### Model Customization
When customizing models:
1. Fork the app using `oscar_fork_app`
2. Import from `oscar.apps.<app>.abstract_models`
3. Extend the abstract model with your custom fields
4. Update `INSTALLED_APPS` to use your forked app
5. Create and run migrations

### Dashboard App Dependencies
Dashboard applications depend on core applications. When forking:
- Fork core apps before dashboard apps
- If customizing a dashboard sub-app (e.g., `dashboard.catalogue`), also fork the core `dashboard` app

### Test Database
- Default: SQLite (`sandbox/db.sqlite`)
- PostgreSQL: Use `sandbox/settings_postgres.py` for migration testing
- Fixtures located in: `sandbox/fixtures/`

## Common Development Workflows

### Adding a New Feature to an Existing App
1. Fork the relevant Oscar app if not already forked
2. Add your changes to the forked app
3. Write tests in the appropriate test directory (unit/integration/functional)
4. Run tests: `make test`
5. Format code: `make black`
6. Lint: `make lint`

### Running a Single Test
```bash
pytest tests/unit/catalogue/test_models.py::TestProductModel::test_method_name
```

### Debugging in the Sandbox
```bash
make sandbox
cd sandbox
./manage.py runserver
# Visit http://localhost:8000
# Dashboard: http://localhost:8000/dashboard/ (superuser credentials in fixtures)
```

### Working with Migrations
All migrations are created against the sandbox:
```bash
make sandbox
sandbox/manage.py makemigrations <app_name>
sandbox/manage.py migrate
```

## Configuration Files

- `pyproject.toml` - Python package configuration and dependencies
- `tox.ini` - Testing matrix configuration
- `setup.cfg` - Pytest, flake8, and isort configuration
- `package.json` - Node.js dependencies and scripts
- `.pre-commit-config.yaml` - Pre-commit hooks (trailing whitespace, flake8, isort)
- `Makefile` - Development task automation

## Key Dependencies

- Django 4.2+ or 5.2+
- Haystack (search)
- Treebeard (category trees)
- Pillow (image handling)
- Babel (currency formatting)
- django-tables2 (table rendering)
- Bootstrap 4 (UI framework)
