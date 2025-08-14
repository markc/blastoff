# [🔙](README.md) Model Relationships in Laravel and Filament

This guide covers implementing and managing complex Eloquent relationships in Laravel, with specific focus on how they integrate with Filament 4.x admin interfaces.

## Eloquent Relationship Types

### 1. One-to-One Relationships

**User Profile Relationship:**
```php
// User Model
class User extends Authenticatable
{
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }
}

// Profile Model
class Profile extends Model
{
    protected $fillable = [
        'user_id',
        'first_name',
        'last_name',
        'bio',
        'avatar',
        'phone',
        'address',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

**Filament Integration:**
```php
// In UserResource.php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Account Information')
                ->schema([
                    TextInput::make('name')->required(),
                    TextInput::make('email')->email()->required(),
                ]),
                
            Section::make('Profile Information')
                ->relationship('profile')
                ->schema([
                    TextInput::make('first_name'),
                    TextInput::make('last_name'),
                    Textarea::make('bio')->rows(3),
                    FileUpload::make('avatar')->image(),
                    TextInput::make('phone'),
                    Textarea::make('address')->rows(2),
                ]),
        ]);
}
```

### 2. One-to-Many Relationships

**Post and Comments:**
```php
// Post Model
class Post extends Model
{
    protected $fillable = [
        'title',
        'slug',
        'content',
        'status',
        'published_at',
        'user_id',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
    
    public function approvedComments(): HasMany
    {
        return $this->hasMany(Comment::class)->where('status', 'approved');
    }
}

// Comment Model
class Comment extends Model
{
    protected $fillable = [
        'post_id',
        'author_name',
        'author_email',
        'content',
        'status',
        'parent_id', // For nested comments
    ];

    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
    
    public function parent(): BelongsTo
    {
        return $this->belongsTo(Comment::class, 'parent_id');
    }
    
    public function replies(): HasMany
    {
        return $this->hasMany(Comment::class, 'parent_id');
    }
}
```

**Filament Relation Manager:**
```php
// Create: php artisan make:filament-relation-manager PostResource comments content
class CommentsRelationManager extends RelationManager
{
    protected static string $relationship = 'comments';

    public function form(Form $form): Form
    {
        return $form
            ->schema([
                TextInput::make('author_name')
                    ->required()
                    ->maxLength(255),
                    
                TextInput::make('author_email')
                    ->email()
                    ->required(),
                    
                Textarea::make('content')
                    ->required()
                    ->rows(4),
                    
                Select::make('status')
                    ->options([
                        'pending' => 'Pending',
                        'approved' => 'Approved',
                        'spam' => 'Spam',
                    ])
                    ->default('pending')
                    ->required(),
                    
                Select::make('parent_id')
                    ->label('Reply to Comment')
                    ->relationship('parent', 'content')
                    ->getOptionLabelFromRecordUsing(fn (Comment $record): string => 
                        Str::limit($record->content, 50)
                    )
                    ->searchable(),
            ]);
    }

    public function table(Table $table): Table
    {
        return $table
            ->recordTitleAttribute('content')
            ->columns([
                TextColumn::make('author_name')
                    ->searchable()
                    ->sortable(),
                    
                TextColumn::make('content')
                    ->limit(50)
                    ->searchable(),
                    
                BadgeColumn::make('status')
                    ->colors([
                        'warning' => 'pending',
                        'success' => 'approved',
                        'danger' => 'spam',
                    ]),
                    
                TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable(),
            ])
            ->filters([
                SelectFilter::make('status')
                    ->options([
                        'pending' => 'Pending',
                        'approved' => 'Approved',
                        'spam' => 'Spam',
                    ]),
            ])
            ->headerActions([
                CreateAction::make(),
            ])
            ->actions([
                EditAction::make(),
                Action::make('approve')
                    ->action(fn (Comment $record) => $record->update(['status' => 'approved']))
                    ->requiresConfirmation()
                    ->visible(fn (Comment $record) => $record->status !== 'approved'),
                DeleteAction::make(),
            ])
            ->bulkActions([
                BulkActionGroup::make([
                    DeleteBulkAction::make(),
                    BulkAction::make('approve')
                        ->action(fn (Collection $records) => $records->each->update(['status' => 'approved']))
                        ->requiresConfirmation(),
                ]),
            ]);
    }
}
```

### 3. Many-to-Many Relationships

**Posts and Tags:**
```php
// Post Model
class Post extends Model
{
    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class)
            ->withTimestamps()
            ->withPivot(['order', 'featured']);
    }
}

// Tag Model
class Tag extends Model
{
    protected $fillable = [
        'name',
        'slug',
        'description',
        'color',
    ];

    public function posts(): BelongsToMany
    {
        return $this->belongsToMany(Post::class)
            ->withTimestamps()
            ->withPivot(['order', 'featured']);
    }
}

// Migration for pivot table
Schema::create('post_tag', function (Blueprint $table) {
    $table->id();
    $table->foreignId('post_id')->constrained()->cascadeOnDelete();
    $table->foreignId('tag_id')->constrained()->cascadeOnDelete();
    $table->integer('order')->default(0);
    $table->boolean('featured')->default(false);
    $table->timestamps();
});
```

**Filament Many-to-Many Management:**
```php
// In PostResource form
CheckboxList::make('tags')
    ->relationship('tags', 'name')
    ->columns(3)
    ->gridDirection('row')
    ->bulkToggleable()
    ->searchable(),

// Alternative with Select for many tags
Select::make('tags')
    ->relationship('tags', 'name')
    ->multiple()
    ->searchable()
    ->preload()
    ->createOptionForm([
        TextInput::make('name')->required(),
        TextInput::make('slug')->required(),
        Textarea::make('description'),
        ColorPicker::make('color'),
    ])
    ->editOptionForm([
        TextInput::make('name')->required(),
        TextInput::make('slug')->required(),
        Textarea::make('description'),
        ColorPicker::make('color'),
    ]),
```

### 4. Polymorphic Relationships

**Comments on Multiple Models:**
```php
// Comment Model (polymorphic)
class Comment extends Model
{
    protected $fillable = [
        'commentable_id',
        'commentable_type',
        'author_name',
        'author_email',
        'content',
        'status',
    ];

    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

// Post Model
class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

// Video Model
class Video extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

**Filament Polymorphic Relations:**
```php
// In CommentResource
Select::make('commentable')
    ->label('Content')
    ->options(function () {
        $posts = Post::pluck('title', 'id')->mapWithKeys(fn ($title, $id) => 
            ["App\\Models\\Post::{$id}" => "Post: {$title}"]
        );
        
        $videos = Video::pluck('title', 'id')->mapWithKeys(fn ($title, $id) => 
            ["App\\Models\\Video::{$id}" => "Video: {$title}"]
        );
        
        return $posts->merge($videos);
    })
    ->afterStateUpdated(function (callable $set, $state) {
        if ($state) {
            [$type, $id] = explode('::', $state);
            $set('commentable_type', $type);
            $set('commentable_id', $id);
        }
    })
    ->searchable(),
```

### 5. Has-Many-Through Relationships

**Countries, Users, and Posts:**
```php
// Country Model
class Country extends Model
{
    public function users(): HasMany
    {
        return $this->hasMany(User::class);
    }

    public function posts(): HasManyThrough
    {
        return $this->hasManyThrough(Post::class, User::class);
    }
}

// User Model
class User extends Model
{
    public function country(): BelongsTo
    {
        return $this->belongsTo(Country::class);
    }

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

## Advanced Relationship Patterns

### 1. Self-Referencing Relationships

**Hierarchical Categories:**
```php
class Category extends Model
{
    protected $fillable = [
        'name',
        'slug',
        'description',
        'parent_id',
        'sort_order',
    ];

    public function parent(): BelongsTo
    {
        return $this->belongsTo(Category::class, 'parent_id');
    }

    public function children(): HasMany
    {
        return $this->hasMany(Category::class, 'parent_id')
            ->orderBy('sort_order');
    }

    public function descendants(): HasMany
    {
        return $this->hasMany(Category::class, 'parent_id')
            ->with('descendants');
    }

    public function ancestors(): Collection
    {
        $ancestors = collect();
        $parent = $this->parent;
        
        while ($parent) {
            $ancestors->push($parent);
            $parent = $parent->parent;
        }
        
        return $ancestors;
    }
}
```

**Filament Tree Structure:**
```php
// In CategoryResource
Select::make('parent_id')
    ->label('Parent Category')
    ->relationship('parent', 'name')
    ->searchable()
    ->nullable()
    ->rules([
        function () {
            return function (string $attribute, $value, Closure $fail) {
                // Prevent circular references
                if ($value && $this->record && $this->record->descendants->contains('id', $value)) {
                    $fail('Cannot select a descendant as parent.');
                }
            };
        },
    ]),

// Display hierarchical structure in table
TextColumn::make('full_path')
    ->getStateUsing(function (Category $record): string {
        $path = collect([$record->name]);
        $parent = $record->parent;
        
        while ($parent) {
            $path->prepend($parent->name);
            $parent = $parent->parent;
        }
        
        return $path->join(' > ');
    })
    ->searchable(),
```

### 2. Conditional Relationships

**Dynamic Relationships Based on Type:**
```php
class Product extends Model
{
    public function variants(): HasMany
    {
        return $this->hasMany(ProductVariant::class);
    }

    public function digitalAssets(): HasMany
    {
        return $this->hasMany(DigitalAsset::class)
            ->where('type', 'digital');
    }

    public function physicalInventory(): HasMany
    {
        return $this->hasMany(Inventory::class)
            ->where('type', 'physical');
    }

    public function getRelatedItemsAttribute()
    {
        return match ($this->type) {
            'digital' => $this->digitalAssets,
            'physical' => $this->physicalInventory,
            'variable' => $this->variants,
            default => collect(),
        };
    }
}
```

### 3. Eager Loading Strategies

**Optimized Relationship Loading:**
```php
// In PostResource table query
public function table(Table $table): Table
{
    return $table
        ->query(
            Post::query()
                ->with(['user', 'category', 'tags'])
                ->withCount(['comments', 'likes'])
        )
        ->columns([
            TextColumn::make('title'),
            TextColumn::make('user.name'),
            TextColumn::make('category.name'),
            TextColumn::make('tags.name')->badge(),
            TextColumn::make('comments_count')->counts('comments'),
            TextColumn::make('likes_count')->counts('likes'),
        ]);
}

// Conditional eager loading
public function scopeWithRelations(Builder $query, array $relations = []): Builder
{
    $defaultRelations = ['user', 'category'];
    
    if (in_array('comments', $relations)) {
        $defaultRelations[] = 'comments.author';
    }
    
    if (in_array('tags', $relations)) {
        $defaultRelations[] = 'tags';
    }
    
    return $query->with($defaultRelations);
}
```

## Testing Relationships with Laravel Boost

### 1. Relationship Integrity Testing

**Test with Claude Code:**
```
"Test the User-Post relationship with actual data"
"Verify the Post-Comments relationship loads correctly"
"Check if the many-to-many Post-Tags relationship works"
"Test polymorphic comments on different models"
```

### 2. Performance Testing

**Query Performance:**
```
"Check for N+1 queries in Post resource with relationships"
"Test eager loading performance for complex relationships"
"Monitor memory usage with deep relationship loading"
```

### 3. Data Integrity Testing

**Relationship Consistency:**
```
"Find any orphaned comments without valid posts"
"Check for posts without valid user references"
"Verify all pivot table relationships are consistent"
```

## Relationship Performance Optimization

### 1. Query Optimization

**Efficient Relationship Queries:**
```php
// Instead of lazy loading
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name; // N+1 problem
}

// Use eager loading
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name; // Single query
}

// Conditional eager loading
$posts = Post::when($includeComments, function ($query) {
    return $query->with('comments.author');
})->get();
```

### 2. Relationship Caching

**Cache Expensive Relationships:**
```php
class Post extends Model
{
    public function getCachedCommentsCountAttribute(): int
    {
        return cache()->remember(
            "post.{$this->id}.comments_count",
            3600,
            fn () => $this->comments()->count()
        );
    }

    public function getPopularTagsAttribute(): Collection
    {
        return cache()->remember(
            "post.{$this->id}.popular_tags",
            3600,
            fn () => $this->tags()
                ->withCount('posts')
                ->orderByDesc('posts_count')
                ->limit(5)
                ->get()
        );
    }
}
```

### 3. Database Indexing

**Optimize Relationship Queries:**
```php
// Migration indexes for relationships
Schema::table('posts', function (Blueprint $table) {
    $table->index('user_id'); // Foreign key index
    $table->index(['status', 'published_at']); // Compound index
    $table->index(['category_id', 'created_at']); // Category posts
});

Schema::table('comments', function (Blueprint $table) {
    $table->index(['commentable_type', 'commentable_id']); // Polymorphic index
    $table->index(['post_id', 'status']); // Approved comments
    $table->index('parent_id'); // Nested comments
});
```

## Best Practices

### 1. Relationship Design

**Design Principles:**
- Use appropriate relationship types for your data model
- Consider performance implications of deep relationships
- Implement proper foreign key constraints
- Use meaningful relationship method names

### 2. Filament Integration

**Admin Interface:**
- Use relation managers for complex relationships
- Implement proper validation for relationship fields
- Provide intuitive interfaces for managing relationships
- Consider user permissions for relationship access

### 3. Performance

**Optimization:**
- Always eager load relationships when needed
- Use database indexes on foreign keys
- Cache expensive relationship queries
- Monitor query performance with Laravel Boost

### 4. Data Integrity

**Consistency:**
- Use database foreign key constraints
- Implement model observers for relationship changes
- Validate relationship data before saving
- Handle cascading deletes appropriately

## Next Steps

1. **Master [Authentication & Authorization](14-auth-authorization.md)** for relationship security
2. **Learn [Performance Optimization](20-performance-optimization.md)** for relationship optimization
3. **Practice [Testing Strategies](16-testing-strategies.md)** for relationship testing

Complex model relationships are the foundation of sophisticated Laravel applications, and proper implementation with Filament provides powerful admin interfaces for managing interconnected data.