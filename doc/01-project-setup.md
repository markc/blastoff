# [🔙](README.md) Project Setup Guide

This guide demonstrates setting up a Laravel + Filament 4.x project optimized for development with Laravel Boost and Claude Code.

## Prerequisites

- PHP 8.2 or higher
- Composer
- Node.js and npm
- SQLite (or your preferred database)
- Claude Code with Laravel Boost MCP server

## Initial Project Setup

### 1. Create New Laravel Project

```bash
# Create new Laravel project
composer create-project laravel/laravel boost-demo
cd boost-demo
```

### 2. Install Core Dependencies

```bash
# Install Filament 4.x
composer require filament/filament:"^4.0"

# Install Laravel Boost (development only)
composer require --dev laravel/boost:"^1.0"

# Install additional development tools
composer require --dev laravel/pint laravel/pail
```

### 3. Environment Configuration

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Configure database (SQLite for simplicity)
nano .env
```

Update `.env` for SQLite:
```bash
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/your/project/database/database.sqlite
# Remove other DB_* variables
```

### 4. Database Setup

```bash
# Create SQLite database file
touch database/database.sqlite

# Run initial migrations
php artisan migrate
```

### 5. Install Filament Admin Panel

```bash
# Install Filament admin panel
php artisan filament:install --panels

# Create admin user
php artisan make:filament-user
```

### 6. Frontend Dependencies

```bash
# Install Node.js dependencies
npm install

# Install Tailwind CSS 4.x
npm install -D tailwindcss@^4.0.0 @tailwindcss/vite@^4.0.0

# Build frontend assets
npm run build
```

### 7. Configure Vite for Tailwind CSS 4.x

Update `vite.config.js`:
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
});
```

Update `resources/css/app.css`:
```css
@import "tailwindcss";

/* Your custom styles here */
```

## Laravel Boost Integration

### 1. Verify Boost Installation

Using Claude Code, run:
```bash
# Check Laravel Boost is available
composer show laravel/boost
```

### 2. Test Boost Functionality

With Claude Code, test these Boost features:

**Application Info:**
Use Claude Code's `application-info` tool to verify installation:
- PHP version
- Laravel version  
- Installed packages
- Available models

**Database Query:**
Use Claude Code's `database-query` tool to test database connectivity:
```sql
SELECT name FROM sqlite_master WHERE type='table';
```

**Tinker Integration:**
Use Claude Code's `tinker` tool to test PHP execution:
```php
echo "Laravel Boost is working!";
App\Models\User::count();
```

## Development Environment Setup

### 1. Configure Development Scripts

Add to `composer.json` scripts section:
```json
{
    "scripts": {
        "dev": [
            "Composer\\Config::disableProcessTimeout",
            "npx concurrently -c \"#93c5fd,#c4b5fd,#fb7185,#fdba74\" \"php artisan serve\" \"php artisan queue:listen --tries=1\" \"php artisan pail --timeout=0\" \"npm run dev\" --names=server,queue,logs,vite"
        ],
        "test": [
            "@php artisan config:clear --ansi",
            "@php artisan test"
        ]
    }
}
```

### 2. Install Concurrently for Development

```bash
npm install -D concurrently
```

### 3. Start Development Environment

```bash
# Start all development services
composer run dev
```

This command starts:
- Laravel development server (`localhost:8000`)
- Queue worker for background jobs
- Log monitoring with Pail
- Vite development server for asset compilation

## Verify Installation

### 1. Test Laravel Application

```bash
# Visit Laravel welcome page
curl http://localhost:8000
```

### 2. Test Filament Admin Panel

```bash
# Visit Filament admin (should redirect to login)
curl http://localhost:8000/admin
```

### 3. Test Asset Compilation

```bash
# Build assets
npm run build

# Verify compiled files exist
ls -la public/build/
```

### 4. Run Tests

```bash
# Run test suite
php artisan test

# Format code
vendor/bin/pint
```

## Project Structure Overview

After setup, your project structure should include:

```
boost-demo/
├── app/
│   ├── Filament/           # Filament resources, pages, widgets
│   ├── Http/Controllers/   # Laravel controllers
│   ├── Models/            # Eloquent models
│   └── Providers/         # Service providers
├── database/
│   ├── database.sqlite    # SQLite database file
│   ├── migrations/        # Database migrations
│   └── seeders/          # Database seeders
├── resources/
│   ├── css/app.css       # Tailwind CSS entry point
│   ├── js/app.js         # JavaScript entry point
│   └── views/            # Blade templates
├── routes/
│   ├── web.php           # Web routes
│   └── console.php       # Console routes
└── tests/                # PHPUnit tests
```

## Next Steps

1. **Read [Laravel Boost Basics](02-boost-basics.md)** to understand MCP server integration
2. **Follow [Command-Line Development](03-command-line-development.md)** for efficient workflows
3. **Build your first admin panel** with [Filament Basics](08-filament-basics.md)

## Common Issues

### Database Connection Errors
- Ensure SQLite file exists: `touch database/database.sqlite`
- Use absolute paths in `.env` file
- Check file permissions

### Asset Compilation Issues
- Run `npm install` to ensure dependencies are installed
- Check Node.js version compatibility
- Verify Vite configuration matches documentation

### Filament Installation Issues
- Ensure Laravel 12.x compatibility
- Run `php artisan filament:upgrade` after updates
- Check Filament panel provider is registered

### Laravel Boost Not Working
- Verify Boost is installed: `composer show laravel/boost`
- Ensure you're using Claude Code with MCP server support
- Check MCP server configuration in Claude Code settings