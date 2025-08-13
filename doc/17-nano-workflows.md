# [🔙](README.md) Efficient Nano Usage for Laravel Development

This guide covers advanced nano usage patterns specifically optimized for Laravel + Filament development, enabling efficient code editing without traditional IDEs.

## Advanced Nano Configuration

### 1. Optimized .nanorc Setup

**Enhanced Configuration:**
```bash
nano ~/.nanorc
```

```bash
# Essential settings for Laravel development
set linenumbers
set mouse
set autoindent
set tabsize 4
set tabstospaces
set smooth
set softwrap
set whitespace
set titlecolor brightwhite,blue
set statuscolor brightwhite,green
set keycolor cyan
set functioncolor green

# Syntax highlighting
include /usr/share/nano/*.nanorc

# Custom bindings for Laravel development
bind ^S writeout main
bind ^Q exit main
bind ^F whereis main
bind ^G gotoline main
bind ^R replace main

# Multi-buffer support
set multibuffer
bind M-< prevbuffer main
bind M-> nextbuffer main

# Enable backup files
set backup
set backupdir ~/.nano/backups

# Better search
set casesensitive
set regexp
```

### 2. Laravel-Specific Shortcuts

**Create Custom Key Bindings:**
```bash
# Add to ~/.nanorc

# Quick Laravel file navigation
bind F1 "cd app/Models && ls" main
bind F2 "cd app/Http/Controllers && ls" main  
bind F3 "cd database/migrations && ls" main
bind F4 "cd resources/views && ls" main

# Common Laravel commands
bind ^L "php artisan list" main
bind ^M "php artisan migrate" main
bind ^T "php artisan test" main
```

## File Navigation Patterns

### 1. Project Structure Navigation

**Quick Directory Access:**
```bash
# Create navigation aliases
alias models='cd app/Models && ls -la'
alias controllers='cd app/Http/Controllers && ls -la'
alias migrations='cd database/migrations && ls -la'
alias resources='cd app/Filament/Resources && ls -la'
alias views='cd resources/views && ls -la'
alias routes='nano routes/web.php'
alias config='nano config/app.php'
alias env='nano .env'

# Filament-specific navigation
alias filament='cd app/Filament && find . -name "*.php" | head -10'
alias pages='cd app/Filament/Pages && ls -la'
alias widgets='cd app/Filament/Widgets && ls -la'
```

### 2. Smart File Opening

**Context-Aware File Access:**
```bash
# Open related files quickly
function edit_model() {
    local model_name=$1
    nano "app/Models/${model_name}.php"
}

function edit_resource() {
    local resource_name=$1
    nano "app/Filament/Resources/${resource_name}Resource.php"
}

function edit_migration() {
    local table_name=$1
    local migration_file=$(ls database/migrations/*_create_${table_name}_table.php 2>/dev/null | head -1)
    if [[ -n $migration_file ]]; then
        nano "$migration_file"
    else
        echo "Migration for $table_name not found"
    fi
}

# Usage examples:
# edit_model Post
# edit_resource Post  
# edit_migration posts
```

## Efficient Editing Workflows

### 1. Multi-Buffer Management

**Working with Multiple Files:**
```bash
# Open multiple related files simultaneously
nano app/Models/Post.php app/Filament/Resources/PostResource.php database/migrations/*_create_posts_table.php

# Navigate between buffers:
# Alt+< - Previous buffer
# Alt+> - Next buffer
# Ctrl+T - Show all open buffers
```

### 2. Search and Replace Patterns

**Laravel-Specific Search Patterns:**
```bash
# Common Laravel search patterns in nano:
# Ctrl+W then enter:

# Find all methods:
function.*\(

# Find class definitions:
class.*{

# Find use statements:
^use.*

# Find route definitions:
Route::

# Find Filament form fields:
Forms\\Components\\

# Find Eloquent relationships:
public function.*\(\).*BelongsTo|HasMany|HasOne
```

### 3. Code Snippets and Templates

**Quick Code Generation:**
```bash
# Create snippet files for common patterns
mkdir -p ~/.nano/snippets

# Laravel Model snippet
cat > ~/.nano/snippets/model.php << 'EOF'
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class {{MODEL_NAME}} extends Model
{
    use HasFactory;

    protected $fillable = [
        //
    ];

    protected function casts(): array
    {
        return [
            //
        ];
    }
}
EOF

# Filament Resource snippet
cat > ~/.nano/snippets/resource.php << 'EOF'
<?php

namespace App\Filament\Resources;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class {{RESOURCE_NAME}}Resource extends Resource
{
    protected static ?string $model = {{MODEL_NAME}}::class;
    
    protected static ?string $navigationIcon = 'heroicon-o-rectangle-stack';

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                //
            ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                //
            ])
            ->filters([
                //
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }
}
EOF
```

## Laravel Development Workflows

### 1. Model-First Development

**Complete Model Creation Workflow:**
```bash
# 1. Create model with related files
php artisan make:model Post -mfs

# 2. Edit migration
nano database/migrations/*_create_posts_table.php
# Add fields, relationships, indexes

# 3. Edit model  
nano app/Models/Post.php
# Add fillable, casts, relationships

# 4. Edit factory
nano database/factories/PostFactory.php
# Define factory attributes

# 5. Run migration
php artisan migrate

# 6. Test via Claude Code:
# "Test the Post model and verify relationships work"
```

### 2. Filament Resource Development

**Resource Creation Workflow:**
```bash
# 1. Create Filament resource
php artisan make:filament-resource Post --generate

# 2. Edit resource form
nano app/Filament/Resources/PostResource.php
# Customize form schema

# 3. Test in browser
php artisan serve
# Visit /admin to test

# 4. Iterate and improve
# Use nano to refine form fields, table columns, filters
```

### 3. Database-Driven Development

**Migration-First Approach:**
```bash
# 1. Create migration
php artisan make:migration create_posts_table

# 2. Design schema in nano
nano database/migrations/*_create_posts_table.php

# 3. Run migration
php artisan migrate

# 4. Verify with Claude Code:
# "Show me the posts table structure"

# 5. Create model to match
nano app/Models/Post.php
```

## Advanced Nano Techniques

### 1. Code Navigation

**Efficient Code Movement:**
```bash
# Navigation shortcuts in nano:
# Ctrl+Y - Previous page
# Ctrl+V - Next page  
# Ctrl+G - Go to specific line number
# Alt+G - Go to beginning of file
# Alt+T - Go to end of file

# Search-based navigation:
# Ctrl+W - Search forward
# Alt+W - Search backward
# Ctrl+\ - Search and replace

# Bookmarking (if supported):
# Alt+Ins - Set bookmark
# Alt+PgUp/PgDn - Go to previous/next bookmark
```

### 2. Code Selection and Manipulation

**Text Selection and Editing:**
```bash
# Selection in nano:
# Ctrl+6 - Start/stop text selection
# Alt+6 - Copy selection
# Ctrl+K - Cut line or selection
# Ctrl+U - Paste

# Indentation:
# Alt+} - Indent selected text
# Alt+{ - Unindent selected text

# Case conversion:
# Alt+U - Convert to uppercase
# Alt+L - Convert to lowercase
```

### 3. Multi-Line Editing

**Efficient Multi-Line Operations:**
```bash
# Comment multiple lines:
# 1. Select lines with Ctrl+6
# 2. Alt+3 to comment
# 3. Alt+3 again to uncomment

# Duplicate lines:
# 1. Position cursor on line
# 2. Ctrl+K (cut)
# 3. Ctrl+U (paste)
# 4. Ctrl+U (paste again)

# Move lines:
# 1. Cut line with Ctrl+K
# 2. Move to destination
# 3. Paste with Ctrl+U
```

## Integration with Laravel Boost

### 1. Edit-Test Cycle

**Rapid Development Loop:**
```bash
# 1. Edit file in nano
nano app/Models/Post.php

# 2. Test immediately via Claude Code:
# "Test the Post model changes"

# 3. Fix issues and repeat
# No need to exit nano - use another terminal

# 4. Format code when done
vendor/bin/pint app/Models/Post.php
```

### 2. Database-Informed Editing

**Use Boost for Context:**
```bash
# Before editing models, check database:
# Ask Claude Code: "Show me the posts table structure"

# Edit model based on actual schema
nano app/Models/Post.php

# Verify changes work:
# Ask Claude Code: "Test the updated Post model"
```

### 3. Documentation-Driven Development

**Research Before Coding:**
```bash
# Before implementing features:
# Ask Claude Code: "How do I add file uploads to Filament resources?"

# Then implement with nano:
nano app/Filament/Resources/PostResource.php

# Add the documented approach
```

## Productivity Tips

### 1. Session Management

**Persistent Work Sessions:**
```bash
# Use tmux/screen for persistent sessions
tmux new-session -d -s laravel

# Multiple nano sessions
tmux split-window -h 'nano app/Models/Post.php'
tmux split-window -v 'nano app/Filament/Resources/PostResource.php'

# Attach to session
tmux attach -t laravel
```

### 2. Backup and Recovery

**Automatic Backup Strategy:**
```bash
# Enable nano backups
echo 'set backup' >> ~/.nanorc
echo 'set backupdir ~/.nano/backups' >> ~/.nanorc

# Create backup directory
mkdir -p ~/.nano/backups

# Additional git-based backup
alias save='git add . && git commit -m "WIP: nano session save"'
```

### 3. Template Integration

**Quick Template Access:**
```bash
# Function to use templates
function use_template() {
    local template_name=$1
    local target_file=$2
    
    if [[ -f ~/.nano/snippets/${template_name}.php ]]; then
        cp ~/.nano/snippets/${template_name}.php "$target_file"
        nano "$target_file"
    else
        echo "Template $template_name not found"
    fi
}

# Usage:
# use_template model app/Models/Product.php
# use_template resource app/Filament/Resources/ProductResource.php
```

## Troubleshooting Common Issues

### 1. File Encoding Issues

**Handle Different Encodings:**
```bash
# Check file encoding
file -i filename.php

# Convert encoding if needed
iconv -f iso-8859-1 -t utf-8 input.php > output.php
```

### 2. Large File Handling

**Optimize for Large Files:**
```bash
# For large files, disable certain features
nano --restricted --tempfile large-migration.php

# Or use view mode for read-only
nano --view large-log-file.txt
```

### 3. Syntax Highlighting Issues

**Fix Highlighting Problems:**
```bash
# Update nanorc files
sudo nano /usr/share/nano/php.nanorc

# Or create custom highlighting
nano ~/.nano/laravel.nanorc
```

## Next Steps

1. **Master [File Navigation](18-file-navigation.md)** for advanced project organization
2. **Learn [Debugging Techniques](19-debugging-techniques.md)** for efficient problem-solving
3. **Practice [Performance Optimization](20-performance-optimization.md)** with nano-based profiling

Mastering nano for Laravel development enables efficient, focused coding without the overhead of traditional IDEs, especially when combined with Laravel Boost's immediate feedback capabilities.