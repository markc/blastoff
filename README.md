# :rocket: Blast Off!

A demonstration project showcasing how Laravel Boost accelerates Filament 4 development with Claude Code at the helm.

## Project Overview

This project demonstrates the enhanced development experience possible when combining:

- **Laravel 12.x** - Latest Laravel framework with streamlined structure
- **Filament 4.0** - Modern admin panel framework
- **Laravel Boost** - MCP server providing powerful development tools
- **Tailwind CSS 4.0** - Latest utility-first CSS framework
- **Claude Code** - AI-powered development without traditional IDEs

The project is specifically designed to demonstrate development workflows using only command-line tools and Claude Code's intelligent assistance.

## Technology Stack

- **Backend**: Laravel 12.24.0 (PHP 8.4+)
- **Admin Interface**: Filament 4.0.0
- **Frontend**: Livewire 3.6.4 + Tailwind CSS 4.0.0
- **Database**: SQLite (for simplicity)
- **Development Tools**: Laravel Boost 1.0+, Laravel Pint 1.24.0
- **Testing**: Pest 4.0.0-beta.2

## Quick Start

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Create database
touch database/database.sqlite

# Run migrations
php artisan migrate

# Start development environment
composer run dev
```

## Laravel Boost Integration

Laravel Boost enhances development by providing:
- **Direct database querying** without leaving the command line
- **Laravel Tinker integration** for immediate PHP execution
- **Real-time log monitoring** for debugging
- **Documentation search** for version-specific guidance
- **Application introspection** for understanding structure

## Development Workflow

### Core Commands
```bash
# Start full development environment
composer run dev

# Format code (required before commits)
vendor/bin/pint

# Run tests
php artisan test

# Access admin panel
# http://localhost:8000/admin
```

### With Laravel Boost + Claude Code
1. **Start with Boost application info** to understand the current state
2. **Use Boost documentation search** before implementing features
3. **Test code with Boost Tinker** before writing to files
4. **Monitor logs with Boost** during development
5. **Query database directly with Boost** for data verification

## Features

- **Streamlined Laravel 12 structure** - No legacy middleware or kernel files
- **Filament 4.0 admin panel** - Modern, responsive interface
- **MCP server integration** - Seamless development tools
- **Command-line first** - Pure terminal-based development
- **Version-aware documentation** - Always current for installed packages

## License

MIT License - See LICENSE file for details
