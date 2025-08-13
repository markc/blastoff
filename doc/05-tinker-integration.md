# [🔙](README.md) Tinker Integration with Laravel Boost

Laravel Boost's Tinker integration allows you to execute PHP code directly in your Laravel application context through Claude Code. This provides immediate feedback for testing models, relationships, services, and application logic.

## Understanding Tinker Integration

### What is Laravel Tinker?

Laravel Tinker is an interactive REPL (Read-Eval-Print Loop) for Laravel applications. Boost's integration allows you to:
- Execute PHP code in full Laravel application context
- Test Eloquent models and relationships
- Verify service functionality
- Debug application state
- Prototype code before writing files

### How Boost Enhances Tinker

Traditional Tinker requires:
```bash
php artisan tinker
>>> App\Models\User::count()
```

With Boost via Claude Code:
```
"Test if the User model works correctly"
```

Claude Code automatically executes appropriate PHP code and shows results.

## Basic Tinker Operations

### Model Testing

**Count Records:**
```
Ask Claude Code: "How many users are in the database?"
```

Executes:
```php
App\Models\User::count();
```

**Create Test Data:**
```
Ask Claude Code: "Create a test user"
```

Executes:
```php
$user = App\Models\User::factory()->create();
echo "Created user: {$user->name} ({$user->email})";
```

**Test Model Attributes:**
```
Ask Claude Code: "Show me a sample user with all attributes"
```

Executes:
```php
$user = App\Models\User::first();
if ($user) {
    echo "User: {$user->name}\n";
    echo "Email: {$user->email}\n";
    echo "Created: {$user->created_at}\n";
    var_dump($user->toArray());
}
```

### Relationship Testing

**Test Relationships:**
```
Ask Claude Code: "Test the user-posts relationship"
```

Executes:
```php
$user = App\Models\User::with('posts')->first();
if ($user) {
    echo "User: {$user->name}\n";
    echo "Posts count: {$user->posts->count()}\n";
    foreach ($user->posts as $post) {
        echo "- {$post->title}\n";
    }
}
```

**Verify Inverse Relationships:**
```
Ask Claude Code: "Test if posts correctly reference their users"
```

Executes:
```php
$post = App\Models\Post::with('user')->first();
if ($post && $post->user) {
    echo "Post: {$post->title}\n";
    echo "Author: {$post->user->name}\n";
} else {
    echo "No post found or post has no user\n";
}
```

## Advanced Tinker Patterns

### 1. Factory and Seeder Testing

**Test Model Factories:**
```
Ask Claude Code: "Test the Post factory"
```

Executes:
```php
// Create single post
$post = App\Models\Post::factory()->create();
echo "Created post: {$post->title}\n";

// Create multiple posts
$posts = App\Models\Post::factory()->count(3)->create();
echo "Created {$posts->count()} posts\n";

// Test factory states if they exist
$featured = App\Models\Post::factory()->featured()->create();
echo "Created featured post: {$featured->title}\n";
```

**Test Relationships in Factories:**
```
Ask Claude Code: "Create a user with posts using factories"
```

Executes:
```php
$user = App\Models\User::factory()
    ->has(App\Models\Post::factory()->count(3))
    ->create();
    
echo "User: {$user->name}\n";
echo "Posts created: {$user->posts->count()}\n";
foreach ($user->posts as $post) {
    echo "- {$post->title}\n";
}
```

### 2. Service and Configuration Testing

**Test Configuration:**
```
Ask Claude Code: "Check if mail configuration is working"
```

Executes:
```php
echo "Mail driver: " . config('mail.default') . "\n";
echo "Mail host: " . config('mail.mailers.smtp.host') . "\n";
echo "App name: " . config('app.name') . "\n";
echo "App URL: " . config('app.url') . "\n";
```

**Test Services:**
```
Ask Claude Code: "Test if the cache service works"
```

Executes:
```php
// Test cache
Cache::put('test_key', 'test_value', 60);
$cached = Cache::get('test_key');
echo "Cache test: " . ($cached === 'test_value' ? 'PASS' : 'FAIL') . "\n";

// Test database connection
try {
    DB::connection()->getPdo();
    echo "Database connection: PASS\n";
} catch (Exception $e) {
    echo "Database connection: FAIL - " . $e->getMessage() . "\n";
}
```

### 3. Eloquent Query Testing

**Complex Queries:**
```
Ask Claude Code: "Test complex post queries with filters"
```

Executes:
```php
// Test various query patterns
$publishedPosts = App\Models\Post::where('status', 'published')->count();
echo "Published posts: {$publishedPosts}\n";

$featuredPosts = App\Models\Post::where('featured', true)->count();
echo "Featured posts: {$featuredPosts}\n";

$recentPosts = App\Models\Post::where('created_at', '>=', now()->subWeek())->count();
echo "Recent posts (last week): {$recentPosts}\n";

// Test with relationships
$postsWithAuthors = App\Models\Post::has('user')->count();
echo "Posts with authors: {$postsWithAuthors}\n";
```

**Query Performance Testing:**
```
Ask Claude Code: "Test query performance for post listing"
```

Executes:
```php
// Measure query performance
$start = microtime(true);

$posts = App\Models\Post::with('user')
    ->where('status', 'published')
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();

$time = microtime(true) - $start;
echo "Query time: " . round($time * 1000, 2) . "ms\n";
echo "Posts fetched: {$posts->count()}\n";
echo "First post: {$posts->first()->title}\n";
```

## Filament-Specific Testing

### 1. Resource Data Testing

**Test Resource Query:**
```
Ask Claude Code: "Test the data that PostResource will display"
```

Executes:
```php
// Simulate Filament resource query
$query = App\Models\Post::query();

// Apply typical Filament eager loading
$posts = $query->with(['user'])->get();

echo "Total posts for resource: {$posts->count()}\n";
foreach ($posts->take(3) as $post) {
    echo "- {$post->title} by {$post->user->name}\n";
}
```

**Test Form Options:**
```
Ask Claude Code: "Test options for post status select field"
```

Executes:
```php
// Test distinct values for select fields
$statuses = App\Models\Post::distinct('status')->pluck('status');
echo "Available statuses:\n";
foreach ($statuses as $status) {
    $count = App\Models\Post::where('status', $status)->count();
    echo "- {$status}: {$count} posts\n";
}

// Test relationship options
$users = App\Models\User::pluck('name', 'id');
echo "\nAvailable authors: {$users->count()}\n";
```

### 2. Form Validation Testing

**Test Validation Rules:**
```
Ask Claude Code: "Test post validation scenarios"
```

Executes:
```php
// Test unique slug validation
$existingPost = App\Models\Post::first();
if ($existingPost) {
    echo "Existing slug: {$existingPost->slug}\n";
    
    $duplicate = App\Models\Post::where('slug', $existingPost->slug)
        ->where('id', '!=', $existingPost->id)
        ->exists();
    echo "Slug uniqueness test: " . ($duplicate ? 'FAIL' : 'PASS') . "\n";
}

// Test required relationships
$orphanedPosts = App\Models\Post::whereNull('user_id')->count();
echo "Posts without users: {$orphanedPosts}\n";
```

### 3. Authorization Testing

**Test Policy Methods:**
```
Ask Claude Code: "Test post authorization policies"
```

Executes:
```php
$user = App\Models\User::first();
$post = App\Models\Post::first();

if ($user && $post) {
    // Test if policies exist and work
    try {
        $canView = $user->can('view', $post);
        $canUpdate = $user->can('update', $post);
        $canDelete = $user->can('delete', $post);
        
        echo "User: {$user->name}\n";
        echo "Can view post: " . ($canView ? 'YES' : 'NO') . "\n";
        echo "Can update post: " . ($canUpdate ? 'YES' : 'NO') . "\n";
        echo "Can delete post: " . ($canDelete ? 'YES' : 'NO') . "\n";
    } catch (Exception $e) {
        echo "Policy test failed: " . $e->getMessage() . "\n";
    }
}
```

## Development Workflow Integration

### 1. Pre-Development Testing

**Before Creating Models:**
```bash
# Ask Claude Code:
"What models exist in this application?"
"Test the existing User model functionality"
"Check what relationships are already defined"
```

**Before Creating Resources:**
```bash
# Ask Claude Code:
"Test the Post model with sample data"
"Check what fields are available on the Post model"
"Verify relationships work correctly"
```

### 2. During Development Testing

**After Model Changes:**
```bash
# Edit model with nano
nano app/Models/Post.php

# Test via Claude Code:
"Test the updated Post model"
"Verify new relationships work"
"Check if new methods function correctly"
```

**After Migration:**
```bash
# Run migration
php artisan migrate

# Test via Claude Code:
"Test if the new fields are accessible on the model"
"Create a test record with the new structure"
"Verify constraints work as expected"
```

### 3. Debugging with Tinker

**When Filament Shows Errors:**
```
Ask Claude Code: "Debug the Post model for any issues"
```

Executes diagnostic code:
```php
// Check model exists and is accessible
try {
    $model = new App\Models\Post();
    echo "Model instantiated successfully\n";
    
    // Check database table exists
    $count = App\Models\Post::count();
    echo "Table exists with {$count} records\n";
    
    // Check relationships
    $post = App\Models\Post::with('user')->first();
    if ($post) {
        echo "Relationships working: " . ($post->user ? 'YES' : 'NO') . "\n";
    }
    
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## Performance and Optimization Testing

### 1. N+1 Query Detection

**Test for N+1 Problems:**
```
Ask Claude Code: "Check for N+1 query issues in post listing"
```

Executes:
```php
// Enable query logging
DB::enableQueryLog();

// Simulate typical resource loading
$posts = App\Models\Post::all();
foreach ($posts as $post) {
    echo $post->user->name . "\n"; // This would cause N+1
}

$queries = DB::getQueryLog();
echo "Total queries executed: " . count($queries) . "\n";

// Better approach with eager loading
DB::flushQueryLog();
$posts = App\Models\Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name . "\n";
}

$queries = DB::getQueryLog();
echo "Queries with eager loading: " . count($queries) . "\n";
```

### 2. Memory Usage Testing

**Test Memory Consumption:**
```
Ask Claude Code: "Test memory usage for large data sets"
```

Executes:
```php
$startMemory = memory_get_usage(true);

// Test chunk processing
$processedCount = 0;
App\Models\Post::chunk(100, function ($posts) use (&$processedCount) {
    $processedCount += $posts->count();
});

$endMemory = memory_get_usage(true);
$memoryUsed = $endMemory - $startMemory;

echo "Processed {$processedCount} posts\n";
echo "Memory used: " . round($memoryUsed / 1024 / 1024, 2) . " MB\n";
```

## Error Handling and Debugging

### 1. Exception Testing

**Test Error Scenarios:**
```
Ask Claude Code: "Test error handling for invalid data"
```

Executes:
```php
try {
    // Test with invalid data
    $post = new App\Models\Post();
    $post->title = null; // Should fail validation
    $post->save();
    echo "Unexpected success\n";
} catch (Exception $e) {
    echo "Caught expected error: " . $e->getMessage() . "\n";
}

// Test with valid data
try {
    $user = App\Models\User::first();
    $post = new App\Models\Post([
        'title' => 'Test Post',
        'slug' => 'test-post-' . time(),
        'content' => 'Test content',
        'status' => 'draft',
        'user_id' => $user->id,
    ]);
    $post->save();
    echo "Valid post created successfully\n";
} catch (Exception $e) {
    echo "Unexpected error: " . $e->getMessage() . "\n";
}
```

### 2. Constraint Testing

**Test Database Constraints:**
```
Ask Claude Code: "Test database constraints for the Post model"
```

Executes:
```php
// Test unique constraints
try {
    $existingPost = App\Models\Post::first();
    if ($existingPost) {
        $duplicate = new App\Models\Post([
            'title' => 'Duplicate Test',
            'slug' => $existingPost->slug, // Should fail
            'content' => 'Test content',
            'status' => 'draft',
            'user_id' => $existingPost->user_id,
        ]);
        $duplicate->save();
        echo "ERROR: Duplicate slug allowed\n";
    }
} catch (Exception $e) {
    echo "Unique constraint working: " . $e->getMessage() . "\n";
}

// Test foreign key constraints
try {
    $post = new App\Models\Post([
        'title' => 'Orphan Test',
        'slug' => 'orphan-test-' . time(),
        'content' => 'Test content',
        'status' => 'draft',
        'user_id' => 99999, // Invalid user ID
    ]);
    $post->save();
    echo "ERROR: Invalid foreign key allowed\n";
} catch (Exception $e) {
    echo "Foreign key constraint working: " . $e->getMessage() . "\n";
}
```

## Best Practices

### 1. Safe Testing Patterns

- Always check if records exist before accessing them
- Use try-catch blocks for operations that might fail
- Clean up test data when appropriate
- Use factories for creating test data

### 2. Efficient Testing

- Test small samples before large operations
- Use specific queries instead of loading all data
- Verify assumptions before complex operations
- Use eager loading when testing relationships

### 3. Development Integration

- Test models immediately after creation
- Verify relationships before building Filament resources
- Test edge cases and error scenarios
- Use Tinker to prototype complex logic

## Next Steps

1. **Learn [Log Monitoring](06-log-monitoring.md)** for real-time debugging
2. **Master [Documentation Search](07-documentation-search.md)** for finding solutions
3. **Practice [Filament Resources](09-filament-resources.md)** with model insights from Tinker

Laravel Boost's Tinker integration provides immediate feedback on your application's behavior, enabling rapid development and debugging without leaving the command line interface.