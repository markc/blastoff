# [🔙](README.md) File Navigation and Project Organization

This guide covers efficient file navigation strategies and project organization patterns for Laravel + Filament development using command-line tools and nano.

## Laravel Project Structure Navigation

### 1. Understanding Laravel 12 Structure

**Core Directory Navigation:**
```bash
# Application structure overview
tree -I 'node_modules|vendor|storage/logs|storage/framework' -L 3

# Key directories for navigation:
app/                    # Application logic
├── Console/           # Artisan commands
├── Http/             # Controllers, middleware, requests
├── Models/           # Eloquent models
├── Providers/        # Service providers
└── Filament/         # Filament resources, pages, widgets

config/               # Configuration files
database/             # Migrations, factories, seeders
resources/            # Views, assets, language files
routes/               # Route definitions
tests/                # Test files
```

### 2. Quick Navigation Aliases

**Essential Bash Aliases:**
```bash
# Add to ~/.bashrc or ~/.zshrc

# Laravel core navigation
alias models='cd app/Models && ls -la'
alias controllers='cd app/Http/Controllers && ls -la'
alias middleware='cd app/Http/Middleware && ls -la'
alias providers='cd app/Providers && ls -la'
alias migrations='cd database/migrations && ls -la'
alias factories='cd database/factories && ls -la'
alias seeders='cd database/seeders && ls -la'

# Filament navigation
alias filament='cd app/Filament && find . -name "*.php" | head -10'
alias resources='cd app/Filament/Resources && ls -la'
alias pages='cd app/Filament/Pages && ls -la'
alias widgets='cd app/Filament/Widgets && ls -la'

# Configuration and routes
alias config='cd config && ls -la'
alias routes='cd routes && ls -la'
alias views='cd resources/views && ls -la'
alias assets='cd resources && ls -la'

# Testing navigation
alias tests='cd tests && find . -name "*.php" | head -10'
alias feature='cd tests/Feature && ls -la'
alias unit='cd tests/Unit && ls -la'

# Quick file access
alias env='nano .env'
alias composer='nano composer.json'
alias package='nano package.json'
alias vite='nano vite.config.js'
```

### 3. Smart Navigation Functions

**Context-Aware Navigation:**
```bash
# Add to ~/.bashrc or ~/.zshrc

# Navigate to model and related files
goto_model() {
    local model_name=$1
    if [[ -z $model_name ]]; then
        echo "Usage: goto_model <ModelName>"
        return 1
    fi
    
    echo "Model: app/Models/${model_name}.php"
    echo "Factory: database/factories/${model_name}Factory.php"
    echo "Migration: $(ls database/migrations/*_create_${model_name,,}*_table.php 2>/dev/null | head -1)"
    echo "Resource: app/Filament/Resources/${model_name}Resource.php"
    
    if [[ -f "app/Models/${model_name}.php" ]]; then
        cd app/Models
    else
        echo "Model not found!"
    fi
}

# Navigate to Filament resource and related files
goto_resource() {
    local resource_name=$1
    if [[ -z $resource_name ]]; then
        echo "Usage: goto_resource <ResourceName>"
        return 1
    fi
    
    local resource_file="app/Filament/Resources/${resource_name}Resource.php"
    local resource_dir="app/Filament/Resources/${resource_name}Resource"
    
    if [[ -f $resource_file ]]; then
        echo "Resource: $resource_file"
        
        if [[ -d $resource_dir ]]; then
            echo "Pages:"
            ls -la "$resource_dir/Pages/"
            echo "RelationManagers:"
            ls -la "$resource_dir/RelationManagers/" 2>/dev/null || echo "  None"
        fi
        
        cd "app/Filament/Resources"
    else
        echo "Resource not found!"
    fi
}

# Find files by pattern
find_laravel() {
    local pattern=$1
    local type=${2:-"php"}
    
    find . -name "*.${type}" -not -path "./vendor/*" -not -path "./node_modules/*" \
        -not -path "./storage/logs/*" -not -path "./storage/framework/*" \
        | xargs grep -l "$pattern" 2>/dev/null
}

# Find Laravel-specific patterns
find_method() {
    local method_name=$1
    find_laravel "function $method_name"
}

find_class() {
    local class_name=$1
    find_laravel "class $class_name"
}

find_route() {
    local route_pattern=$1
    find_laravel "$route_pattern" | grep -E "(routes|Resource\.php)"
}
```

## File Organization Strategies

### 1. Logical Grouping by Feature

**Feature-Based Organization:**
```bash
# Example: Blog feature organization
app/
├── Models/
│   ├── Post.php
│   ├── Category.php
│   └── Tag.php
├── Http/Controllers/
│   ├── PostController.php
│   └── CategoryController.php
├── Filament/Resources/
│   ├── PostResource.php
│   ├── PostResource/
│   │   ├── Pages/
│   │   │   ├── CreatePost.php
│   │   │   ├── EditPost.php
│   │   │   └── ListPosts.php
│   │   └── RelationManagers/
│   │       └── CommentsRelationManager.php
│   └── CategoryResource.php

# Navigation helper for features
goto_feature() {
    local feature=$1
    echo "=== $feature Feature Files ==="
    echo "Models:"
    ls -la app/Models/ | grep -i "$feature" || echo "  None found"
    echo "Controllers:"
    ls -la app/Http/Controllers/ | grep -i "$feature" || echo "  None found"
    echo "Resources:"
    ls -la app/Filament/Resources/ | grep -i "$feature" || echo "  None found"
    echo "Migrations:"
    ls -la database/migrations/ | grep -i "$feature" || echo "  None found"
}
```

### 2. File Naming Conventions

**Consistent Naming Patterns:**
```bash
# Model conventions
app/Models/
├── User.php                    # Singular, PascalCase
├── Post.php
├── BlogCategory.php            # Descriptive names for clarity
└── UserProfile.php

# Filament resource conventions
app/Filament/Resources/
├── UserResource.php            # Model + Resource suffix
├── PostResource.php
├── BlogCategoryResource.php
└── UserProfileResource.php

# Migration conventions  
database/migrations/
├── 2024_01_01_000000_create_users_table.php        # Plural table names
├── 2024_01_02_000000_create_posts_table.php
├── 2024_01_03_000000_add_featured_to_posts_table.php   # Descriptive modifications
└── 2024_01_04_000000_create_post_tag_pivot_table.php   # Clear pivot naming
```

### 3. Directory Structure Navigation

**Hierarchical Navigation:**
```bash
# Create navigation bookmark system
create_bookmarks() {
    mkdir -p ~/.laravel_bookmarks
    
    # Create symbolic links for quick access
    ln -sf "$(pwd)/app/Models" ~/.laravel_bookmarks/models
    ln -sf "$(pwd)/app/Filament/Resources" ~/.laravel_bookmarks/resources
    ln -sf "$(pwd)/database/migrations" ~/.laravel_bookmarks/migrations
    ln -sf "$(pwd)/tests" ~/.laravel_bookmarks/tests
    
    echo "Bookmarks created in ~/.laravel_bookmarks/"
}

# Quick bookmark access
bm() {
    local bookmark=$1
    if [[ -z $bookmark ]]; then
        echo "Available bookmarks:"
        ls -la ~/.laravel_bookmarks/
        return
    fi
    
    if [[ -L ~/.laravel_bookmarks/$bookmark ]]; then
        cd ~/.laravel_bookmarks/$bookmark
    else
        echo "Bookmark '$bookmark' not found"
    fi
}
```

## Advanced File Search Techniques

### 1. Content-Based Search

**Laravel-Specific Search Patterns:**
```bash
# Function to search Laravel patterns
search_laravel() {
    local pattern=$1
    local context=${2:-3}
    
    grep -r -n --include="*.php" \
        --exclude-dir=vendor \
        --exclude-dir=node_modules \
        --exclude-dir=storage \
        -C $context "$pattern" .
}

# Common Laravel search patterns
alias find_models='search_laravel "extends Model"'
alias find_resources='search_laravel "extends Resource"'
alias find_migrations='search_laravel "Schema::"'
alias find_routes='search_laravel "Route::"'
alias find_policies='search_laravel "extends.*Policy"'
alias find_middleware='search_laravel "extends.*Middleware"'

# Filament-specific searches
alias find_forms='search_laravel "Forms\\\\Components"'
alias find_tables='search_laravel "Tables\\\\Columns"'
alias find_actions='search_laravel "Tables\\\\Actions"'
alias find_widgets='search_laravel "extends.*Widget"'
```

### 2. File Type Navigation

**Navigate by File Purpose:**
```bash
# Navigate to specific file types
goto_type() {
    local type=$1
    
    case $type in
        "model"|"models")
            cd app/Models && ls -la
            ;;
        "resource"|"resources")
            cd app/Filament/Resources && ls -la
            ;;
        "migration"|"migrations")
            cd database/migrations && ls -lt
            ;;
        "test"|"tests")
            cd tests && find . -name "*.php" | head -20
            ;;
        "config")
            cd config && ls -la
            ;;
        "route"|"routes")
            cd routes && ls -la
            ;;
        "view"|"views")
            cd resources/views && find . -name "*.blade.php" | head -20
            ;;
        *)
            echo "Unknown type: $type"
            echo "Available types: model, resource, migration, test, config, route, view"
            ;;
    esac
}
```

### 3. Recent Files Navigation

**Track Recently Modified Files:**
```bash
# Show recently modified Laravel files
recent() {
    local hours=${1:-24}
    
    echo "=== Recently modified files (last $hours hours) ==="
    find . -name "*.php" \
        -not -path "./vendor/*" \
        -not -path "./node_modules/*" \
        -not -path "./storage/*" \
        -mtime -$(echo "$hours/24" | bc -l) \
        -type f \
        -exec ls -lt {} + | head -20
}

# Show recently modified by category
recent_models() {
    find app/Models -name "*.php" -mtime -1 -exec ls -lt {} +
}

recent_resources() {
    find app/Filament/Resources -name "*.php" -mtime -1 -exec ls -lt {} +
}

recent_migrations() {
    find database/migrations -name "*.php" -mtime -1 -exec ls -lt {} +
}
```

## IDE-Alternative Navigation

### 1. Multi-File Editing with Nano

**Session Management:**
```bash
# Create editing sessions for related files
edit_model_set() {
    local model_name=$1
    
    if [[ -z $model_name ]]; then
        echo "Usage: edit_model_set <ModelName>"
        return 1
    fi
    
    local model_file="app/Models/${model_name}.php"
    local resource_file="app/Filament/Resources/${model_name}Resource.php"
    local migration_file=$(ls database/migrations/*_create_${model_name,,}*_table.php 2>/dev/null | head -1)
    local factory_file="database/factories/${model_name}Factory.php"
    
    local files_to_edit=()
    
    [[ -f $model_file ]] && files_to_edit+=("$model_file")
    [[ -f $resource_file ]] && files_to_edit+=("$resource_file")
    [[ -f $migration_file ]] && files_to_edit+=("$migration_file")
    [[ -f $factory_file ]] && files_to_edit+=("$factory_file")
    
    if [[ ${#files_to_edit[@]} -gt 0 ]]; then
        echo "Opening files: ${files_to_edit[*]}"
        nano "${files_to_edit[@]}"
    else
        echo "No files found for model: $model_name"
    fi
}

# Edit feature-related files
edit_feature() {
    local feature=$1
    
    local files=$(find . -name "*.php" \
        -not -path "./vendor/*" \
        -not -path "./storage/*" \
        | xargs grep -l "$feature" 2>/dev/null | head -5)
    
    if [[ -n $files ]]; then
        echo "Opening feature files: $files"
        nano $files
    else
        echo "No files found for feature: $feature"
    fi
}
```

### 2. Project-Wide Search and Replace

**Safe Search and Replace:**
```bash
# Search and replace with backup
safe_replace() {
    local search_pattern=$1
    local replace_pattern=$2
    local file_pattern=${3:-"*.php"}
    
    if [[ -z $search_pattern || -z $replace_pattern ]]; then
        echo "Usage: safe_replace <search> <replace> [file_pattern]"
        return 1
    fi
    
    # Create backup
    local backup_dir="backup_$(date +%Y%m%d_%H%M%S)"
    mkdir -p $backup_dir
    
    # Find and preview changes
    echo "Files that would be changed:"
    find . -name "$file_pattern" \
        -not -path "./vendor/*" \
        -not -path "./node_modules/*" \
        -exec grep -l "$search_pattern" {} \;
    
    read -p "Continue with replacement? (y/N): " confirm
    if [[ $confirm =~ ^[Yy]$ ]]; then
        # Create backups and perform replacement
        find . -name "$file_pattern" \
            -not -path "./vendor/*" \
            -not -path "./node_modules/*" \
            -exec grep -l "$search_pattern" {} \; \
            | while read file; do
                cp "$file" "$backup_dir/$(basename $file)"
                sed -i "s/$search_pattern/$replace_pattern/g" "$file"
                echo "Updated: $file"
            done
            
        echo "Backups saved in: $backup_dir"
    fi
}
```

### 3. Code Navigation Shortcuts

**Quick Code Jumping:**
```bash
# Jump to specific code elements
goto_class() {
    local class_name=$1
    local file=$(grep -r "class $class_name" app/ --include="*.php" | head -1 | cut -d: -f1)
    
    if [[ -n $file ]]; then
        local line=$(grep -n "class $class_name" "$file" | cut -d: -f1)
        echo "Opening $file at line $line"
        nano "+$line" "$file"
    else
        echo "Class $class_name not found"
    fi
}

goto_function() {
    local function_name=$1
    local file=$(grep -r "function $function_name" app/ --include="*.php" | head -1 | cut -d: -f1)
    
    if [[ -n $file ]]; then
        local line=$(grep -n "function $function_name" "$file" | cut -d: -f1)
        echo "Opening $file at line $line"
        nano "+$line" "$file"
    else
        echo "Function $function_name not found"
    fi
}

goto_route() {
    local route_name=$1
    local file=$(grep -r "$route_name" routes/ --include="*.php" | head -1 | cut -d: -f1)
    
    if [[ -n $file ]]; then
        local line=$(grep -n "$route_name" "$file" | cut -d: -f1)
        echo "Opening $file at line $line"
        nano "+$line" "$file"
    else
        echo "Route $route_name not found"
    fi
}
```

## Integration with Laravel Boost

### 1. Context-Aware File Access

**Use Boost for File Discovery:**
```bash
# Function to get file recommendations from Claude Code
get_file_context() {
    local search_term=$1
    echo "Ask Claude Code: 'Show me files related to $search_term'"
    echo "Ask Claude Code: 'Where should I look for $search_term implementation?'"
    echo "Ask Claude Code: 'Find files containing $search_term'"
}

# Get model relationship files
explore_relationships() {
    local model=$1
    echo "Ask Claude Code: 'Show me all files related to $model model relationships'"
    echo "Ask Claude Code: 'Find migration files for $model'"
    echo "Ask Claude Code: 'Show me Filament resources using $model'"
}
```

### 2. Boost-Guided Navigation

**Smart File Suggestions:**
```bash
# Before editing, understand the context
before_edit() {
    local file=$1
    
    echo "=== File Context for $file ==="
    echo "Ask Claude Code: 'Explain the purpose of $file'"
    echo "Ask Claude Code: 'Show me related files to $file'"
    echo "Ask Claude Code: 'What should I be careful about when editing $file?'"
    
    if [[ -f $file ]]; then
        echo ""
        echo "=== Current file structure ==="
        grep -n "class\|function\|public\|protected\|private" "$file" | head -10
    fi
}
```

## File Organization Best Practices

### 1. Consistent Structure

**Maintain Organization:**
- Group related files by feature/domain
- Use consistent naming conventions
- Keep similar file types together
- Organize imports and use statements

### 2. Documentation Integration

**File-Level Documentation:**
```bash
# Create file documentation
document_file() {
    local file=$1
    local doc_file="${file%.php}.md"
    
    echo "# $(basename $file)" > "$doc_file"
    echo "" >> "$doc_file"
    echo "## Purpose" >> "$doc_file"
    echo "## Dependencies" >> "$doc_file"
    echo "## Usage" >> "$doc_file"
    echo "## Notes" >> "$doc_file"
    
    nano "$doc_file"
}
```

### 3. Navigation Efficiency

**Quick Access Patterns:**
- Create shortcuts for frequently accessed directories
- Use pattern-based file finding
- Leverage bash history for repeated commands
- Maintain bookmarks for project areas

## Next Steps

1. **Master [Debugging Techniques](19-debugging-techniques.md)** for navigation during troubleshooting
2. **Learn [Performance Optimization](20-performance-optimization.md)** for efficient file organization
3. **Practice [Nano Workflows](17-nano-workflows.md)** for optimized editing

Efficient file navigation and organization are crucial for productive Laravel development, especially when working without traditional IDEs and relying on command-line tools and Laravel Boost integration.