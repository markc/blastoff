# [🔙](README.md) Testing Strategies with Laravel Boost

This guide covers comprehensive testing strategies for Laravel + Filament applications using Pest 4.x and Laravel Boost integration for enhanced testing workflows.

## Testing Environment Setup

### 1. Pest 4.x Configuration

The project uses Pest 4.x for testing:
```bash
# Run tests
./vendor/bin/pest

# Run specific test
./vendor/bin/pest tests/Feature/PostTest.php

# Run with coverage
./vendor/bin/pest --coverage
```

### 2. Testing Database Configuration

**SQLite In-Memory Testing:**
```php
// phpunit.xml configuration
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

**Test Case Setup:**
```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication, RefreshDatabase;
    
    protected function setUp(): void
    {
        parent::setUp();
        
        // Seed essential data for every test
        $this->seed([
            UserSeeder::class,
            CategorySeeder::class,
        ]);
    }
}
```

## Model Testing with Boost Integration

### 1. Model Relationship Testing

**Test with Pest and Boost Verification:**
```php
<?php

use App\Models\User;
use App\Models\Post;

describe('Post Model', function () {
    it('belongs to a user', function () {
        $user = User::factory()->create();
        $post = Post::factory()->create(['user_id' => $user->id]);
        
        expect($post->user)->toBeInstanceOf(User::class);
        expect($post->user->id)->toBe($user->id);
    });
    
    it('can have many comments', function () {
        $post = Post::factory()
            ->hasComments(3)
            ->create();
            
        expect($post->comments)->toHaveCount(3);
    });
    
    it('can be published', function () {
        $post = Post::factory()->create(['status' => 'draft']);
        
        $post->publish();
        
        expect($post->fresh()->status)->toBe('published');
        expect($post->fresh()->published_at)->not->toBeNull();
    });
});

// Verify with Boost via Claude Code:
// "Test the Post model relationships and verify they work correctly"
```

### 2. Factory Testing

**Comprehensive Factory Tests:**
```php
describe('Post Factory', function () {
    it('creates valid posts', function () {
        $post = Post::factory()->create();
        
        expect($post->title)->not->toBeEmpty();
        expect($post->slug)->not->toBeEmpty();
        expect($post->content)->not->toBeEmpty();
        expect($post->user_id)->toBeGreaterThan(0);
    });
    
    it('creates featured posts', function () {
        $post = Post::factory()->featured()->create();
        
        expect($post->featured)->toBeTrue();
    });
    
    it('creates posts with specific status', function () {
        $post = Post::factory()->published()->create();
        
        expect($post->status)->toBe('published');
        expect($post->published_at)->not->toBeNull();
    });
});
```

## Filament Resource Testing

### 1. Resource CRUD Testing

**Test Resource Pages:**
```php
<?php

use App\Filament\Resources\PostResource;
use App\Models\User;
use App\Models\Post;

describe('PostResource', function () {
    beforeEach(function () {
        $this->admin = User::factory()->admin()->create();
        $this->actingAs($this->admin);
    });
    
    it('can render index page', function () {
        $posts = Post::factory(3)->create();
        
        $response = $this->get(PostResource::getUrl('index'));
        
        $response->assertSuccessful();
    });
    
    it('can render create page', function () {
        $response = $this->get(PostResource::getUrl('create'));
        
        $response->assertSuccessful();
    });
    
    it('can create new posts', function () {
        $postData = [
            'title' => 'Test Post',
            'slug' => 'test-post',
            'content' => 'This is test content',
            'status' => 'draft',
            'user_id' => $this->admin->id,
        ];
        
        $response = $this->post(PostResource::getUrl('store'), $postData);
        
        $this->assertDatabaseHas('posts', $postData);
    });
    
    it('validates required fields', function () {
        $response = $this->post(PostResource::getUrl('store'), []);
        
        $response->assertSessionHasErrors(['title', 'content', 'user_id']);
    });
});
```

### 2. Form Validation Testing

**Complex Validation Scenarios:**
```php
describe('Post Form Validation', function () {
    beforeEach(function () {
        $this->admin = User::factory()->admin()->create();
        $this->actingAs($this->admin);
    });
    
    it('requires unique slugs', function () {
        $existingPost = Post::factory()->create(['slug' => 'existing-slug']);
        
        $response = $this->post(PostResource::getUrl('store'), [
            'title' => 'New Post',
            'slug' => 'existing-slug',
            'content' => 'Content',
            'user_id' => $this->admin->id,
        ]);
        
        $response->assertSessionHasErrors(['slug']);
    });
    
    it('validates status options', function () {
        $response = $this->post(PostResource::getUrl('store'), [
            'title' => 'Test Post',
            'slug' => 'test-post',
            'content' => 'Content',
            'status' => 'invalid-status',
            'user_id' => $this->admin->id,
        ]);
        
        $response->assertSessionHasErrors(['status']);
    });
});
```

## Feature Testing with Boost

### 1. End-to-End Feature Tests

**Complete Workflow Testing:**
```php
describe('Blog Management Workflow', function () {
    beforeEach(function () {
        $this->admin = User::factory()->admin()->create();
        $this->author = User::factory()->author()->create();
    });
    
    it('allows admin to manage posts', function () {
        $this->actingAs($this->admin);
        
        // Create post
        $response = $this->post(PostResource::getUrl('store'), [
            'title' => 'Admin Post',
            'slug' => 'admin-post',
            'content' => 'Admin content',
            'status' => 'published',
            'user_id' => $this->author->id,
        ]);
        
        $post = Post::where('slug', 'admin-post')->first();
        expect($post)->not->toBeNull();
        
        // Update post
        $this->put(PostResource::getUrl('update', ['record' => $post]), [
            'title' => 'Updated Admin Post',
            'slug' => 'updated-admin-post',
            'content' => 'Updated content',
            'status' => 'published',
            'user_id' => $this->author->id,
        ]);
        
        expect($post->fresh()->title)->toBe('Updated Admin Post');
    });
    
    it('restricts author permissions', function () {
        $this->actingAs($this->author);
        $otherAuthorPost = Post::factory()->create();
        
        $response = $this->get(PostResource::getUrl('edit', ['record' => $otherAuthorPost]));
        
        $response->assertForbidden();
    });
});

// Verify with Boost:
// "Test the complete post creation workflow"
// "Verify user permissions work correctly in Filament"
```

### 2. API Testing (if applicable)

**API Endpoint Testing:**
```php
describe('Post API', function () {
    it('returns posts list', function () {
        $posts = Post::factory(3)->published()->create();
        
        $response = $this->getJson('/api/posts');
        
        $response->assertSuccessful()
            ->assertJsonCount(3, 'data');
    });
    
    it('filters posts by status', function () {
        Post::factory(2)->published()->create();
        Post::factory(3)->draft()->create();
        
        $response = $this->getJson('/api/posts?status=published');
        
        $response->assertSuccessful()
            ->assertJsonCount(2, 'data');
    });
});
```

## Database Testing Strategies

### 1. Migration Testing

**Test Database Schema:**
```php
describe('Database Migrations', function () {
    it('creates posts table with correct structure', function () {
        expect(Schema::hasTable('posts'))->toBeTrue();
        expect(Schema::hasColumn('posts', 'title'))->toBeTrue();
        expect(Schema::hasColumn('posts', 'slug'))->toBeTrue();
        expect(Schema::hasColumn('posts', 'content'))->toBeTrue();
        expect(Schema::hasColumn('posts', 'status'))->toBeTrue();
        expect(Schema::hasColumn('posts', 'user_id'))->toBeTrue();
    });
    
    it('has proper foreign key constraints', function () {
        // Test via Boost
        // "Check if the posts table has proper foreign key constraints"
    });
});
```

### 2. Seeder Testing

**Verify Data Seeding:**
```php
describe('Database Seeders', function () {
    it('creates admin user', function () {
        $this->seed(UserSeeder::class);
        
        expect(User::where('email', 'admin@example.com')->exists())->toBeTrue();
    });
    
    it('creates sample posts', function () {
        $this->seed([UserSeeder::class, PostSeeder::class]);
        
        expect(Post::count())->toBeGreaterThan(0);
    });
});
```

## Performance Testing

### 1. Query Performance Testing

**N+1 Query Detection:**
```php
describe('Query Performance', function () {
    it('avoids N+1 queries in post listing', function () {
        $users = User::factory(3)->create();
        Post::factory(10)->create();
        
        DB::enableQueryLog();
        
        // Test the actual resource query
        $posts = Post::with('user')->get();
        foreach ($posts as $post) {
            $authorName = $post->user->name; // This should not trigger additional queries
        }
        
        $queries = DB::getQueryLog();
        
        // Should be 2 queries: posts + users (with eager loading)
        expect(count($queries))->toBeLessThanOrEqualTo(2);
    });
});

// Verify with Boost:
// "Check for N+1 query issues in the Post resource"
```

### 2. Memory Usage Testing

**Large Dataset Handling:**
```php
describe('Memory Performance', function () {
    it('handles large datasets efficiently', function () {
        Post::factory(1000)->create();
        
        $memoryBefore = memory_get_usage();
        
        // Use chunking for large datasets
        $processedCount = 0;
        Post::chunk(100, function ($posts) use (&$processedCount) {
            $processedCount += $posts->count();
        });
        
        $memoryAfter = memory_get_usage();
        $memoryUsed = $memoryAfter - $memoryBefore;
        
        expect($processedCount)->toBe(1000);
        expect($memoryUsed)->toBeLessThan(50 * 1024 * 1024); // Less than 50MB
    });
});
```

## Integration Testing with Laravel Boost

### 1. Boost Tool Testing

**Test Boost Integration:**
```php
describe('Laravel Boost Integration', function () {
    it('can query database through boost', function () {
        $posts = Post::factory(5)->create();
        
        // This would be tested via Claude Code:
        // "Count posts in the database and verify the result"
        
        expect(Post::count())->toBe(5);
    });
    
    it('can execute tinker commands', function () {
        // Test via Claude Code:
        // "Create a test post using Tinker and verify it exists"
        
        $testPost = Post::factory()->create(['title' => 'Tinker Test Post']);
        expect($testPost->title)->toBe('Tinker Test Post');
    });
});
```

### 2. Documentation Testing

**Verify Documentation Examples:**
```php
describe('Documentation Examples', function () {
    it('validates code examples work correctly', function () {
        // Test examples from documentation
        $user = User::factory()->create();
        $post = Post::factory()->create(['user_id' => $user->id]);
        
        // Test relationship as documented
        expect($post->user->id)->toBe($user->id);
        expect($user->posts->contains($post))->toBeTrue();
    });
});
```

## Testing Best Practices

### 1. Test Organization

**Structure Tests Logically:**
```php
// tests/Feature/PostManagementTest.php - Full workflow tests
// tests/Unit/PostTest.php - Model-specific tests
// tests/Unit/PostServiceTest.php - Service layer tests
```

### 2. Test Data Management

**Use Factories Consistently:**
```php
// Always use factories for test data
$post = Post::factory()->published()->create();

// Use specific states for different scenarios
$draftPost = Post::factory()->draft()->create();
$featuredPost = Post::factory()->featured()->create();
```

### 3. Assertion Patterns

**Clear and Descriptive Assertions:**
```php
// Good assertions
expect($post->isPublished())->toBeTrue();
expect($response)->toHaveStatus(200);
expect($user->can('edit', $post))->toBeTrue();

// Avoid unclear assertions
expect($post->status)->toBe('published'); // Less clear intent
```

## Continuous Integration

### 1. Test Commands

**Automated Testing Scripts:**
```bash
# composer.json scripts
"test": [
    "@php artisan config:clear --ansi",
    "./vendor/bin/pest"
],
"test-coverage": [
    "@php artisan config:clear --ansi", 
    "./vendor/bin/pest --coverage --min=80"
]
```

### 2. GitHub Actions Integration

**Basic CI Configuration:**
```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.4'
        
    - name: Install dependencies
      run: composer install
      
    - name: Run tests
      run: composer run test
```

## Next Steps

1. **Master [Performance Optimization](20-performance-optimization.md)** for testing performance improvements
2. **Learn [Debugging Techniques](19-debugging-techniques.md)** for test debugging
3. **Practice [Model Relationships](13-model-relationships.md)** with comprehensive test coverage

Comprehensive testing with Pest 4.x and Laravel Boost integration ensures reliable, maintainable applications with confidence in every feature.