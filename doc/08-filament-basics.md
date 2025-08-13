# Filament 4.x Basics

This guide covers the fundamentals of building admin interfaces with Filament 4.x, emphasizing development patterns that work well with Laravel Boost and command-line workflows.

## Filament 4.x Overview

Filament 4.x is a modern admin panel framework for Laravel that provides:
- **Rapid admin interface creation** with minimal code
- **Built-in form builders** with extensive field types
- **Advanced table components** with filtering and sorting
- **Dashboard widgets** for data visualization
- **Authentication and authorization** integration
- **Modern UI** with Tailwind CSS

## Core Concepts

### 1. Panel Architecture

Filament organizes functionality into panels. The main admin panel is configured in:
`app/Providers/Filament/AdminPanelProvider.php`

### 2. Resource System

Resources are the primary building blocks:
- **Resources** - Full CRUD interfaces for models
- **Pages** - Custom admin pages
- **Widgets** - Dashboard components
- **Forms** - Reusable form components

### 3. Discovery Convention

Filament auto-discovers components in:
- `app/Filament/Resources/` - Model resources
- `app/Filament/Pages/` - Custom pages  
- `app/Filament/Widgets/` - Dashboard widgets

## Initial Filament Setup

### 1. Installation and Configuration

```bash
# Install Filament 4.x
composer require filament/filament:"^4.0"

# Install admin panel
php artisan filament:install --panels

# Create admin user
php artisan make:filament-user
```

### 2. Verify Installation

```bash
# Start development server
php artisan serve

# Visit admin panel
# http://localhost:8000/admin
```

Test with Claude Code:
```
"Check if the Filament admin panel is accessible and working"
```

## Creating Your First Resource

### 1. Create Model and Migration

```bash
# Create Post model with migration, factory, seeder
php artisan make:model Post -mfs
```

Edit the migration with nano:
```bash
nano database/migrations/*_create_posts_table.php
```

Add fields:
```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('slug')->unique();
        $table->text('content');
        $table->string('status')->default('draft');
        $table->boolean('featured')->default(false);
        $table->timestamp('published_at')->nullable();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->timestamps();
    });
}
```

### 2. Update Model

```bash
nano app/Models/Post.php
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'slug',
        'content', 
        'status',
        'featured',
        'published_at',
        'user_id',
    ];

    protected function casts(): array
    {
        return [
            'featured' => 'boolean',
            'published_at' => 'datetime',
        ];
    }

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

### 3. Run Migration and Test

```bash
# Run migration
php artisan migrate

# Test via Claude Code:
# "Test the Post model and verify the user relationship works"
```

### 4. Create Filament Resource

```bash
# Create resource with all CRUD operations
php artisan make:filament-resource Post --generate --view
```

This creates:
- `app/Filament/Resources/PostResource.php` - Main resource
- `app/Filament/Resources/PostResource/Pages/` - CRUD pages
- Resource pages for Create, Edit, List, View

## Understanding Filament Resources

### 1. Resource Structure

```bash
nano app/Filament/Resources/PostResource.php
```

Key methods in a resource:
- `form()` - Defines create/edit forms
- `table()` - Defines list view table
- `getRelations()` - Defines relationship managers
- `getPages()` - Defines available pages

### 2. Form Configuration

The `form()` method defines the create/edit interface:

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Forms\Components\TextInput::make('title')
                ->required()
                ->live(onBlur: true)
                ->afterStateUpdated(fn (string $operation, $state, callable $set) => 
                    $operation === 'create' ? $set('slug', Str::slug($state)) : null
                ),
                
            Forms\Components\TextInput::make('slug')
                ->required()
                ->unique(Post::class, 'slug', ignoreRecord: true),
                
            Forms\Components\RichEditor::make('content')
                ->required()
                ->columnSpanFull(),
                
            Forms\Components\Select::make('status')
                ->options([
                    'draft' => 'Draft',
                    'published' => 'Published',
                    'archived' => 'Archived',
                ])
                ->required(),
                
            Forms\Components\Toggle::make('featured'),
            
            Forms\Components\DateTimePicker::make('published_at'),
            
            Forms\Components\Select::make('user_id')
                ->relationship('user', 'name')
                ->required(),
        ]);
}
```

### 3. Table Configuration

The `table()` method defines the list view:

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            Tables\Columns\TextColumn::make('title')
                ->searchable()
                ->sortable(),
                
            Tables\Columns\TextColumn::make('slug')
                ->searchable()
                ->toggleable(isToggledHiddenByDefault: true),
                
            Tables\Columns\BadgeColumn::make('status')
                ->colors([
                    'warning' => 'draft',
                    'success' => 'published', 
                    'danger' => 'archived',
                ]),
                
            Tables\Columns\IconColumn::make('featured')
                ->boolean(),
                
            Tables\Columns\TextColumn::make('user.name')
                ->sortable(),
                
            Tables\Columns\TextColumn::make('published_at')
                ->dateTime()
                ->sortable(),
                
            Tables\Columns\TextColumn::make('created_at')
                ->dateTime()
                ->sortable()
                ->toggleable(isToggledHiddenByDefault: true),
        ])
        ->filters([
            Tables\Filters\SelectFilter::make('status')
                ->options([
                    'draft' => 'Draft',
                    'published' => 'Published',
                    'archived' => 'Archived',
                ]),
                
            Tables\Filters\Filter::make('featured')
                ->query(fn (Builder $query): Builder => $query->where('featured', true)),
                
            Tables\Filters\Filter::make('published')
                ->query(fn (Builder $query): Builder => $query->whereNotNull('published_at')),
        ])
        ->actions([
            Tables\Actions\ViewAction::make(),
            Tables\Actions\EditAction::make(),
            Tables\Actions\DeleteAction::make(),
        ])
        ->bulkActions([
            Tables\Actions\BulkActionGroup::make([
                Tables\Actions\DeleteBulkAction::make(),
            ]),
        ]);
}
```

## Testing Filament Resources

### 1. Verify Resource Registration

Test with Claude Code:
```
"Check if the Post resource is registered and accessible in Filament"
```

### 2. Test CRUD Operations

```bash
# Ask Claude Code to verify:
# "Test creating a new post through the Filament interface"
# "Verify the post list displays correctly with filters"
# "Check if the user relationship works in the form"
```

### 3. Database Verification

```bash
# Use Claude Code to check:
# "Show me the posts table structure"
# "Create a test post and verify it was saved correctly"
```

## Common Filament Patterns

### 1. Form Field Types

**Text Inputs:**
```php
TextInput::make('title')->required(),
TextInput::make('slug')->unique(Post::class, 'slug'),
Textarea::make('excerpt')->rows(3),
RichEditor::make('content')->columnSpanFull(),
```

**Selection Fields:**
```php
Select::make('category_id')
    ->relationship('category', 'name')
    ->required(),
    
Radio::make('status')
    ->options(['draft' => 'Draft', 'published' => 'Published']),
    
CheckboxList::make('tags')
    ->relationship('tags', 'name'),
```

**Date and Time:**
```php
DatePicker::make('published_date'),
DateTimePicker::make('published_at'),
TimePicker::make('event_time'),
```

**Boolean and Toggle:**
```php
Toggle::make('featured'),
Checkbox::make('agree_to_terms'),
```

### 2. Table Column Types

**Display Columns:**
```php
TextColumn::make('title')->searchable()->sortable(),
BadgeColumn::make('status')->colors(['success' => 'published']),
IconColumn::make('featured')->boolean(),
ImageColumn::make('featured_image'),
```

**Relationship Columns:**
```php
TextColumn::make('user.name')->sortable(),
TextColumn::make('category.name')->searchable(),
TextColumn::make('tags.name')->badge(),
```

### 3. Filters and Actions

**Table Filters:**
```php
SelectFilter::make('status')->options([...]),
Filter::make('featured')->query(fn ($query) => $query->where('featured', true)),
DateFilter::make('created_at'),
```

**Table Actions:**
```php
ViewAction::make(),
EditAction::make(),
DeleteAction::make(),
Action::make('duplicate')->action(fn (Post $record) => $record->replicate()->save()),
```

## Customizing Filament Resources

### 1. Custom Form Layouts

```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Post Details')
                ->schema([
                    TextInput::make('title')->required(),
                    TextInput::make('slug')->required(),
                ])
                ->columns(2),
                
            Section::make('Content')
                ->schema([
                    RichEditor::make('content')->required(),
                ])
                ->columnSpanFull(),
                
            Section::make('Publishing')
                ->schema([
                    Select::make('status')->options([...]),
                    Toggle::make('featured'),
                    DateTimePicker::make('published_at'),
                ])
                ->columns(3),
        ]);
}
```

### 2. Advanced Table Features

```php
public static function table(Table $table): Table
{
    return $table
        ->columns([...])
        ->defaultSort('created_at', 'desc')
        ->paginated([10, 25, 50, 100])
        ->poll('30s') // Auto-refresh every 30 seconds
        ->striped()
        ->actions([
            ActionGroup::make([
                ViewAction::make(),
                EditAction::make(),
                DeleteAction::make(),
            ])->dropdownPlacement('bottom-start'),
        ]);
}
```

## Development Workflow with Boost

### 1. Iterative Development

```bash
# 1. Create/modify resource
nano app/Filament/Resources/PostResource.php

# 2. Test immediately via Claude Code:
# "Check if the Post resource form displays correctly"

# 3. Fix issues and repeat
vendor/bin/pint  # Format code
```

### 2. Documentation-Driven Development

Before implementing complex features:
```
# Ask Claude Code:
# "How do I add file uploads to a Filament resource in version 4.x?"
# "What's the best way to handle image galleries in Filament forms?"
```

### 3. Database-First Validation

```bash
# Use Claude Code to verify:
# "Test if the form validation works correctly"
# "Check if the database constraints match the form rules"
```

## Next Steps

1. **Learn [Resource Creation](09-filament-resources.md)** for advanced resource patterns
2. **Explore [Custom Pages](10-filament-pages.md)** for non-CRUD interfaces
3. **Master [Forms and Fields](12-filament-forms.md)** for complex form handling

Filament 4.x provides a powerful foundation for admin interfaces, and when combined with Laravel Boost's real-time feedback, enables rapid development of sophisticated admin panels.