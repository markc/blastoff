# [🔙](README.md) Command-Line Development Workflow

This guide demonstrates efficient Laravel + Filament development using only command-line tools, nano editor, and Laravel Boost integration with Claude Code.

## Development Philosophy

Command-line development with Laravel Boost offers several advantages:
- **Immediate feedback** through Boost's real-time application access
- **Reduced context switching** between tools
- **Version-aware assistance** through Boost's documentation integration
- **Efficient file manipulation** with nano and command-line tools
- **Integrated debugging** without external tools

## Core Development Workflow

### 1. Session Initialization

Start every development session with application state analysis:

```bash
# Terminal 1: Start development environment
composer run dev

# Terminal 2: Development commands (keep this active)
# Ask Claude Code: "What's the current state of this application?"
```

This provides:
- Current package versions
- Database schema overview
- Available models and routes
- Recent errors or issues

### 2. Feature Planning

Before coding, gather information and plan:

```bash
# Ask Claude Code for implementation guidance:
# "How should I implement user roles with Filament 4.x in Laravel 12?"

# This triggers Boost documentation search for:
# - Laravel 12 authorization patterns
# - Filament 4.x role management
# - Best practices for your specific package versions
```

### 3. Implementation Cycle

Follow this efficient cycle for each feature:

**A. Model Creation**
```bash
# Create model with all related files
php artisan make:model Role -mfs

# Verify creation with Claude Code:
# "Test the new Role model and show me its structure"
```

**B. Edit with Nano**
```bash
# Edit migration
nano database/migrations/*_create_roles_table.php

# Nano shortcuts:
# Ctrl+W - Search within file
# Ctrl+O - Save file
# Ctrl+X - Exit
# Ctrl+G - Go to line number
```

**C. Test with Boost**
```bash
# Ask Claude Code to test via Tinker:
# "Run the migration and test the Role model"
```

**D. Iterate Rapidly**
```bash
# Format code
vendor/bin/pint

# Run tests
php artisan test

# Check for errors via Claude Code:
# "Any issues in the application logs?"
```

## Efficient Nano Usage

### Essential Nano Shortcuts

**File Operations:**
- `Ctrl+O` - Save file (WriteOut)
- `Ctrl+X` - Exit nano
- `Ctrl+R` - Read/Insert file

**Navigation:**
- `Ctrl+W` - Search (Where Is)
- `Ctrl+\` - Search and replace
- `Ctrl+G` - Go to line number
- `Ctrl+Y` - Previous page
- `Ctrl+V` - Next page

**Editing:**
- `Ctrl+K` - Cut line
- `Ctrl+U` - Paste (UnCut)
- `Ctrl+6` - Mark selection start
- `Alt+6` - Copy selection
- `Ctrl+T` - Check spelling

### Nano Configuration

Create `~/.nanorc` for enhanced editing:

```bash
# Improved nano configuration
nano ~/.nanorc
```

Add these settings:
```
# Show line numbers
set linenumbers

# Enable syntax highlighting
include /usr/share/nano/*.nanorc

# Set tab size
set tabsize 4

# Convert tabs to spaces
set tabstospaces

# Show whitespace
set whitespace

# Enable mouse support
set mouse

# Auto-indent
set autoindent

# Smooth scrolling
set smooth

# Word wrap
set softwrap
```

### Project-Specific Navigation

**Quick File Access:**
```bash
# Create aliases for common files
alias routes='nano routes/web.php'
alias env='nano .env'
alias composer='nano composer.json'

# Navigate to model
alias model='cd app/Models && ls'
alias migration='cd database/migrations && ls'
```

## Boost-Enhanced Development Patterns

### 1. Database-First Development

Use Boost to understand schema before coding:

```bash
# Ask Claude Code: "Show me the current database schema"
# Then: "What relationships exist between models?"

# Plan new features based on existing structure
# Ask: "How should I add comments to the blog post system?"
```

### 2. Documentation-Driven Implementation

Search for patterns before implementation:

```bash
# Before creating Filament resources:
# "Show me examples of Filament resources with file uploads in version 4.x"

# Before model relationships:
# "What's the Laravel 12 way to handle polymorphic relationships?"
```

### 3. Test-As-You-Go Development

Use Boost for continuous testing:

```bash
# After each change, ask Claude Code:
# "Test the new User factory"
# "Verify the relationship works correctly"
# "Check if the migration ran successfully"
```

## Advanced Command-Line Patterns

### 1. Multiple Terminal Workflow

**Terminal 1: Development Server**
```bash
composer run dev
# Runs: server, queue, logs, vite
```

**Terminal 2: Development Commands**
```bash
# Active development terminal
# Use for artisan commands, git, testing
```

**Terminal 3: File Editing**
```bash
# Dedicated nano terminal
# Keep nano open for active file editing
```

**Terminal 4: Claude Code Interaction**
```bash
# Terminal for Claude Code commands
# Use for Boost tool interactions
```

### 2. Efficient File Management

**Quick File Finding:**
```bash
# Find files by type
find . -name "*.php" -path "./app/*" | head -10

# Find specific patterns
grep -r "class.*Resource" app/Filament/

# Use Boost to understand structure:
# Ask Claude Code: "What Filament resources exist in this project?"
```

**Git Integration:**
```bash
# Frequent commits with descriptive messages
git add .
git commit -m "Add Role model with migration and factory"

# Use git for navigation
git log --oneline -10
git diff HEAD~1
```

### 3. Debugging Without IDE

**Log Analysis:**
```bash
# Real-time log monitoring (via composer run dev)
# Plus Claude Code: "Show me recent application errors"

# Database debugging via Claude Code:
# "Check if the user_roles table exists"
# "Show me the last 5 migrations that ran"
```

**Application State Checking:**
```bash
# Via Claude Code Boost integration:
# "Test if the authentication system works"
# "Verify all routes are accessible"
# "Check configuration for mail service"
```

## Filament-Specific Command-Line Workflow

### 1. Resource Creation Pipeline

```bash
# 1. Create model
php artisan make:model Post -mfs

# 2. Edit migration (nano)
nano database/migrations/*_create_posts_table.php

# 3. Run migration
php artisan migrate

# 4. Test via Claude Code:
# "Test the Post model and factory"

# 5. Create Filament resource
php artisan make:filament-resource Post --generate

# 6. Verify via browser and Claude Code:
# "Check if the Post resource appears in Filament admin"
```

### 2. Customization Workflow

```bash
# 1. Identify customization needs via Claude Code:
# "How do I add custom fields to the Post resource?"

# 2. Edit resource file
nano app/Filament/Resources/PostResource.php

# 3. Test changes immediately:
# "Verify the new fields appear in the Filament form"

# 4. Iterate rapidly
vendor/bin/pint  # Format
php artisan test # Test
```

## Performance Optimization

### 1. Reduce Context Switching

- Keep nano open in dedicated terminal
- Use Claude Code for application state instead of manual checking
- Leverage Boost's immediate feedback vs. manual file inspection

### 2. Automate Repetitive Tasks

**Create Development Aliases:**
```bash
# Add to ~/.bashrc
alias pa='php artisan'
alias tinker='php artisan tinker'
alias migrate='php artisan migrate'
alias test='php artisan test'
alias pint='vendor/bin/pint'
alias serve='php artisan serve'

# Filament-specific
alias make-resource='php artisan make:filament-resource'
alias make-page='php artisan make:filament-page'
alias make-widget='php artisan make:filament-widget'
```

**Development Scripts:**
```bash
# Create dev-helper.sh
#!/bin/bash
case $1 in
  "new-model")
    php artisan make:model $2 -mfs
    echo "Created model $2 with migration, factory, seeder"
    ;;
  "new-resource")
    php artisan make:filament-resource $2 --generate
    echo "Created Filament resource for $2"
    ;;
  "fresh")
    php artisan migrate:fresh --seed
    echo "Database refreshed with seed data"
    ;;
esac
```

## Troubleshooting Command-Line Development

### Common Issues

**1. File Not Found Errors**
```bash
# Use Claude Code to locate files:
# "Where is the User model located?"
# "Find all files containing 'UserResource'"
```

**2. Permission Issues**
```bash
# Fix storage permissions
chmod -R 755 storage/
chmod -R 755 bootstrap/cache/
```

**3. Asset Compilation Issues**
```bash
# Check if Vite is running (composer run dev should handle this)
npm run dev

# Or build for production
npm run build
```

### Boost Debugging Integration

**Application Errors:**
```bash
# Ask Claude Code:
# "What's the latest error in the logs?"
# "Test if the database connection works"
# "Verify all services are running correctly"
```

**Performance Issues:**
```bash
# Use Claude Code to check:
# "Are there any slow database queries in the logs?"
# "Check if queue jobs are processing correctly"
```

## Next Steps

1. **Master [Database Operations](04-database-operations.md)** for advanced data management
2. **Learn [Tinker Integration](05-tinker-integration.md)** for interactive PHP development  
3. **Practice [Filament Basics](08-filament-basics.md)** to build admin interfaces efficiently

Command-line development with Laravel Boost provides a streamlined, efficient alternative to traditional IDE-based development, especially when enhanced with Claude Code's intelligent assistance.