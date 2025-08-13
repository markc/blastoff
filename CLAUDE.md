# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a demonstration project showcasing how to use Laravel Boost with Claude Code and Filament 4.x. The project demonstrates the enhanced development experience possible when combining:

- **Laravel 12.x** - Latest Laravel framework with streamlined structure
- **Filament 4.0** - Modern admin panel framework
- **Laravel Boost** - MCP server providing powerful development tools
- **Tailwind CSS 4.0** - Latest utility-first CSS framework
- **Claude Code** - AI-powered development without traditional IDEs

The project is specifically designed to demonstrate development workflows using only `nano` for text editing and command-line tools.

## Technology Stack

- **Backend**: Laravel 12.24.0 (PHP 8.4+)
- **Admin Interface**: Filament 4.0.0
- **Frontend**: Livewire 3.6.4 + Tailwind CSS 4.0.0
- **Database**: SQLite (for simplicity)
- **Development Tools**: Laravel Boost 1.0+, Laravel Pint 1.24.0
- **Testing**: PHPUnit 11.5.3+

## Development Commands

### Laravel Boost Integration
```bash
# Laravel Boost is integrated as an MCP server providing powerful development tools
# Available via Claude Code's MCP integration - no manual installation required
```

### Development Server
```bash
# Start full development environment (recommended)
composer run dev

# This starts concurrently:
# - Laravel development server (php artisan serve)
# - Queue worker (php artisan queue:listen --tries=1)
# - Log monitoring (php artisan pail --timeout=0)
# - Vite development server (npm run dev)

# Manual server start
php artisan serve
```

### Database Operations
```bash
# Create database (SQLite)
touch database/database.sqlite

# Run migrations
php artisan migrate

# Seed database
php artisan db:seed

# Fresh migration with seed
php artisan migrate:fresh --seed
```

### Asset Building
```bash
# Development build with watching
npm run dev

# Production build
npm run build

# Build assets only
vite build
```

### Code Quality
```bash
# Format code (required before commits)
vendor/bin/pint

# Format only dirty files
vendor/bin/pint --dirty

# Run tests
composer run test
# OR
php artisan test

# Run specific test
php artisan test --filter ExampleTest

# Run feature tests only
php artisan test tests/Feature

# Run unit tests only
php artisan test tests/Unit
```

### Filament Administration
```bash
# Access admin panel
# http://localhost:8000/admin

# Create Filament user
php artisan make:filament-user

# Create Filament resource
php artisan make:filament-resource ModelName

# Create Filament page
php artisan make:filament-page PageName

# Create Filament widget
php artisan make:filament-widget WidgetName
```

## Architecture Overview

### Laravel 12 Streamlined Structure
This project uses Laravel 12's new streamlined directory structure:

- **No `app/Http/Middleware/`** - Middleware registered in `bootstrap/app.php`
- **No `app/Console/Kernel.php`** - Console configuration in `bootstrap/app.php` or `routes/console.php`
- **Auto-discovery** - Commands in `app/Console/Commands/` auto-register
- **Simplified providers** - Located in `bootstrap/providers.php`

### Filament 4.0 Integration
- **Admin Panel Provider**: `app/Providers/Filament/AdminPanelProvider.php`
- **Default path**: `/admin` (customizable)
- **Authentication**: Built-in login system
- **Discovery paths**:
  - Resources: `app/Filament/Resources/`
  - Pages: `app/Filament/Pages/`
  - Widgets: `app/Filament/Widgets/`

### Database Configuration
- **Primary**: SQLite (`database/database.sqlite`)
- **Testing**: In-memory SQLite (`:memory:`)
- **Migrations**: Standard Laravel location (`database/migrations/`)

### Frontend Architecture
- **CSS Framework**: Tailwind CSS 4.0 with Vite plugin
- **JavaScript**: Vanilla JS with Laravel Vite plugin
- **Build Tool**: Vite 7.0+ with hot module replacement
- **Entry Points**: 
  - `resources/css/app.css`
  - `resources/js/app.js`

## Laravel Boost Development Workflow

### Core Philosophy
Laravel Boost enhances development by providing:
- **Direct database querying** without leaving the command line
- **Laravel Tinker integration** for immediate PHP execution
- **Real-time log monitoring** for debugging
- **Documentation search** for version-specific guidance
- **Application introspection** for understanding structure

### Recommended Development Pattern
1. **Start with Boost application info** to understand the current state
2. **Use Boost documentation search** before implementing features
3. **Test code with Boost Tinker** before writing to files
4. **Monitor logs with Boost** during development
5. **Query database directly with Boost** for data verification

### Integration with Claude Code
- **No IDE required** - Pure command-line development
- **MCP server integration** - Boost tools available natively
- **Version-aware documentation** - Always current for installed packages
- **Seamless debugging** - Direct access to application state

## Common Development Tasks

### Creating New Features
```bash
# 1. Create model with migration, factory, and seeder
php artisan make:model ModelName -mfs

# 2. Create Filament resource
php artisan make:filament-resource ModelName

# 3. Run migration
php artisan migrate

# 4. Format code
vendor/bin/pint

# 5. Test
php artisan test
```

### Debugging Workflow
```bash
# Check application logs
php artisan pail

# Query database directly (via Boost)
# Use Claude Code's database-query tool

# Execute PHP code (via Boost)  
# Use Claude Code's tinker tool

# Monitor real-time logs (via Boost)
# Use Claude Code's read-log-entries tool
```

### Adding Filament Components
```bash
# Resource with all options
php artisan make:filament-resource Post --generate --view

# Custom page
php artisan make:filament-page Settings --type=custom

# Widget
php artisan make:filament-widget StatsOverview --type=stats-overview

# Form component
php artisan make:filament-form-component CustomField
```

## Testing Strategy

### Test Structure
- **Feature Tests**: `tests/Feature/` - Full application tests
- **Unit Tests**: `tests/Unit/` - Isolated component tests
- **Database**: In-memory SQLite for speed
- **Configuration**: `phpunit.xml` with proper environment isolation

### Running Tests
```bash
# All tests
php artisan test

# With coverage (if xdebug available)
php artisan test --coverage

# Specific test file
php artisan test tests/Feature/ExampleTest.php

# Specific test method
php artisan test --filter test_example_method
```

## Environment Configuration

### Required Environment Variables
```bash
APP_ENV=local
APP_DEBUG=true
APP_KEY=base64:... # Generated with php artisan key:generate
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database/database.sqlite
```

### Development Setup
```bash
# 1. Copy environment file
cp .env.example .env

# 2. Generate application key
php artisan key:generate

# 3. Create database
touch database/database.sqlite

# 4. Run migrations
php artisan migrate

# 5. Install dependencies
composer install
npm install

# 6. Start development
composer run dev
```

## Code Conventions

### Laravel Standards
- **Models**: Singular, PascalCase (`User`, `BlogPost`)
- **Controllers**: Singular with Controller suffix (`UserController`)
- **Resources**: Plural (`app/Filament/Resources/UsersResource.php`)
- **Migrations**: Descriptive snake_case (`create_users_table`)

### Filament Conventions
- **Resources**: Follow Laravel naming with Resource suffix
- **Pages**: Descriptive names for custom pages
- **Widgets**: Descriptive names ending in Widget
- **Forms**: Use Filament's form builder patterns

### Code Formatting
- **Laravel Pint**: Must run before committing (`vendor/bin/pint`)
- **Configuration**: Uses default Laravel standards
- **Automation**: Integrated into development workflow

## Documentation Structure

The `doc/` directory contains comprehensive tutorials:
- **Getting Started**: Basic setup and first steps
- **Laravel Boost Integration**: MCP server usage patterns
- **Filament Development**: Admin panel creation and customization
- **Command-Line Workflows**: Development without traditional IDEs
- **Advanced Patterns**: Complex development scenarios

Each tutorial includes practical examples and step-by-step instructions for nano-based development.