# [🔙](README.md) Performance Optimization for Laravel and Filament

This guide covers comprehensive performance optimization strategies for Laravel 12 + Filament 4.x applications, with emphasis on command-line monitoring and Laravel Boost integration.

## Database Performance Optimization

### 1. Query Optimization

**Eloquent Query Optimization:**
```php
// Inefficient - N+1 Query Problem
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name; // N+1 queries
}

// Optimized - Eager Loading
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name; // 2 queries total
}

// Advanced Eager Loading
$posts = Post::with([
    'user:id,name,email',          // Select specific columns
    'comments' => function ($query) {
        $query->select('id', 'post_id', 'content')
              ->where('status', 'approved')
              ->latest()
              ->limit(3);
    },
    'tags:id,name',
])->get();
```

**Optimized Filament Resource Queries:**
```php
class PostResource extends Resource
{
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->with(['user', 'category'])           // Eager load relationships
            ->withCount(['comments', 'likes'])     // Efficient counting
            ->select([                             // Select only needed columns
                'id',
                'title', 
                'slug',
                'status',
                'user_id',
                'category_id',
                'created_at',
                'updated_at'
            ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->query(
                static::getEloquentQuery()
                    ->when(
                        request()->has('search'),
                        fn ($query) => $query->searchable()
                    )
            )
            ->columns([
                TextColumn::make('title')
                    ->searchable(isIndividual: true)   // Individual search optimization
                    ->sortable(),
                    
                TextColumn::make('user.name')
                    ->sortable(query: function (Builder $query, string $direction): Builder {
                        return $query
                            ->join('users', 'posts.user_id', '=', 'users.id')
                            ->orderBy('users.name', $direction);
                    }),
                    
                TextColumn::make('comments_count')
                    ->counts('comments')
                    ->sortable(),
            ])
            ->defaultSort('created_at', 'desc')
            ->paginated([10, 25, 50, 100]);        // Reasonable pagination options
    }
}
```

### 2. Database Indexing

**Strategic Index Creation:**
```php
// Migration with performance indexes
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->string('slug')->unique();
    $table->text('content');
    $table->enum('status', ['draft', 'published', 'archived'])->default('draft');
    $table->timestamp('published_at')->nullable();
    $table->foreignId('user_id')->constrained();
    $table->foreignId('category_id')->constrained();
    $table->boolean('featured')->default(false);
    $table->timestamps();

    // Performance indexes
    $table->index(['status', 'published_at']);           // For published posts query
    $table->index(['user_id', 'status']);                // For user's posts
    $table->index(['category_id', 'status', 'featured']); // For category filtering
    $table->index(['created_at']);                       // For sorting
    $table->fullText(['title', 'content']);              // For search functionality
});
```

**Index Monitoring with Laravel Boost:**
```
Ask Claude Code: "Show me slow database queries from the logs"
Ask Claude Code: "Check if the posts table has proper indexes"
Ask Claude Code: "Find queries that are doing table scans"
```

### 3. Query Caching

**Model-Level Caching:**
```php
class Post extends Model
{
    // Cache expensive computed properties
    public function getFormattedPublishedAtAttribute(): string
    {
        return cache()->remember(
            "post.{$this->id}.formatted_published_at",
            3600,
            fn () => $this->published_at?->format('F j, Y') ?? 'Not published'
        );
    }

    // Cache relationship counts
    public function getCachedCommentsCountAttribute(): int
    {
        return cache()->remember(
            "post.{$this->id}.comments_count",
            1800,
            fn () => $this->comments()->count()
        );
    }

    // Cache complex queries
    public static function getPopularPosts($limit = 10): Collection
    {
        return cache()->remember(
            "popular_posts_{$limit}",
            3600,
            fn () => static::withCount(['comments', 'likes'])
                ->where('status', 'published')
                ->orderByDesc('likes_count')
                ->orderByDesc('comments_count')
                ->limit($limit)
                ->get()
        );
    }

    // Invalidate cache on model changes
    protected static function booted(): void
    {
        static::saved(function ($post) {
            cache()->forget("post.{$post->id}.comments_count");
            cache()->forget("post.{$post->id}.formatted_published_at");
            cache()->tags(['posts'])->flush();
        });
    }
}
```

## Application Performance

### 1. Configuration Optimization

**Production Configuration:**
```php
// config/app.php - Production settings
return [
    'debug' => false,                    // Always false in production
    'log_level' => 'warning',           // Reduce log verbosity
    'providers' => [
        // Remove development-only providers in production
    ],
];

// config/cache.php - Optimized caching
return [
    'default' => 'redis',               // Use Redis for better performance
    
    'stores' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
            'lock_connection' => 'default',
        ],
    ],
];

// config/session.php - Session optimization
return [
    'driver' => 'redis',                // Use Redis for sessions
    'lifetime' => 120,                  // Reasonable session lifetime
    'expire_on_close' => false,
    'encrypt' => true,
    'files' => storage_path('framework/sessions'),
    'connection' => null,
    'table' => 'sessions',
    'store' => null,
    'lottery' => [2, 100],              // Session cleanup frequency
    'cookie' => 'laravel_session',
    'path' => '/',
    'domain' => null,
    'secure' => true,                   // HTTPS only in production
    'http_only' => true,
    'same_site' => 'lax',
];
```

### 2. Optimized Service Providers

**Efficient Service Provider:**
```php
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Defer expensive operations
        if ($this->app->environment('production')) {
            $this->app->bind('expensive-service', function () {
                return new ExpensiveService();
            });
        }
    }

    public function boot(): void
    {
        // Only perform necessary operations
        if (!$this->app->runningInConsole()) {
            // View composers only for web requests
            View::composer('layouts.app', AppLayoutComposer::class);
        }

        // Optimize Eloquent
        Model::preventLazyLoading(!$this->app->isProduction());
        Model::preventSilentlyDiscardingAttributes(!$this->app->isProduction());

        // Optimize database queries
        DB::whenQueryingForLongerThan(2000, function ($connection, $event) {
            Log::warning('Long query detected', [
                'sql' => $event->sql,
                'time' => $event->time,
                'connection' => $connection->getName(),
            ]);
        });
    }
}
```

### 3. Memory Optimization

**Efficient Data Processing:**
```php
class BulkPostProcessor
{
    public function processPosts(): void
    {
        // Chunk large datasets to prevent memory exhaustion
        Post::chunk(1000, function ($posts) {
            foreach ($posts as $post) {
                $this->processPost($post);
            }
            
            // Clear memory periodically
            gc_collect_cycles();
        });
    }

    public function exportPosts(): void
    {
        // Use lazy collections for large exports
        Post::with('user:id,name')
            ->select('id', 'title', 'content', 'user_id', 'created_at')
            ->lazy()
            ->each(function ($post) {
                $this->writeToFile($post);
            });
    }

    public function updatePostStatuses(): void
    {
        // Batch updates instead of individual saves
        Post::where('status', 'draft')
            ->where('created_at', '<', now()->subDays(30))
            ->update(['status' => 'archived']);
    }
}
```

## Filament Performance Optimization

### 1. Resource Optimization

**Optimized Resource Configuration:**
```php
class PostResource extends Resource
{
    // Limit query scope
    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->with(['user:id,name', 'category:id,name'])
            ->withCount('comments')
            ->select([
                'posts.id',
                'posts.title',
                'posts.slug', 
                'posts.status',
                'posts.user_id',
                'posts.category_id',
                'posts.created_at'
            ]);
    }

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Section::make('Basic Information')
                    ->schema([
                        TextInput::make('title')
                            ->required()
                            ->maxLength(255)
                            ->live(debounce: 1000)        // Debounce live updates
                            ->afterStateUpdated(function (callable $set, $state) {
                                $set('slug', Str::slug($state));
                            }),
                            
                        TextInput::make('slug')
                            ->required()
                            ->unique(ignoreRecord: true),
                    ]),
                    
                RichEditor::make('content')
                    ->lazy()                              // Lazy load heavy components
                    ->required(),
                    
                Select::make('category_id')
                    ->relationship('category', 'name')
                    ->searchable()
                    ->preload()                           // Preload small datasets
                    ->native(false),
                    
                Select::make('tags')
                    ->relationship('tags', 'name')
                    ->multiple()
                    ->searchable()
                    ->optionsLimit(50)                    // Limit search results
                    ->createOptionForm([
                        TextInput::make('name')->required(),
                    ]),
            ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('title')
                    ->limit(50)                          // Limit text length
                    ->searchable(isIndividual: true)
                    ->sortable(),
                    
                TextColumn::make('user.name')
                    ->toggleable(isToggledHiddenByDefault: true), // Hidden by default
                    
                BadgeColumn::make('status')
                    ->colors([
                        'warning' => 'draft',
                        'success' => 'published',
                    ]),
                    
                TextColumn::make('comments_count')
                    ->counts('comments')
                    ->sortable()
                    ->toggleable(),
                    
                TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->since()
                    ->toggleable(),
            ])
            ->filters([
                SelectFilter::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                        'archived' => 'Archived',
                    ])
                    ->multiple(),
                    
                DateFilter::make('created_at'),
            ])
            ->defaultSort('created_at', 'desc')
            ->paginated([25, 50, 100])                   // Reasonable pagination
            ->poll(null);                                // Disable auto-polling
    }
}
```

### 2. Widget Performance

**Efficient Widget Implementation:**
```php
class PostStatsWidget extends StatsOverviewWidget
{
    protected static ?string $pollingInterval = null;    // Disable auto-polling

    protected function getStats(): array
    {
        // Cache expensive calculations
        $cacheKey = 'post_stats_' . auth()->id();
        
        return cache()->remember($cacheKey, 300, function () {
            // Single query with multiple aggregates
            $stats = Post::selectRaw('
                COUNT(*) as total,
                COUNT(CASE WHEN status = "published" THEN 1 END) as published,
                COUNT(CASE WHEN status = "draft" THEN 1 END) as draft,
                COUNT(CASE WHEN featured = 1 THEN 1 END) as featured
            ')->first();

            return [
                Stat::make('Total Posts', $stats->total)
                    ->description('All posts in system')
                    ->color('primary'),
                    
                Stat::make('Published', $stats->published)
                    ->description('Live posts')
                    ->color('success'),
                    
                Stat::make('Draft', $stats->draft)
                    ->description('Unpublished')
                    ->color('warning'),
                    
                Stat::make('Featured', $stats->featured)
                    ->description('Highlighted')
                    ->color('info'),
            ];
        });
    }

    // Cache invalidation
    public static function getStats(): array
    {
        $instance = new static();
        return $instance->getStats();
    }

    public static function invalidateCache(): void
    {
        cache()->forget('post_stats_' . auth()->id());
    }
}
```

### 3. Form Performance

**Optimized Form Loading:**
```php
class CreatePost extends CreateRecord
{
    protected static string $resource = PostResource::class;

    protected function mutateFormDataBeforeCreate(array $data): array
    {
        // Add user ID without loading user relationship
        $data['user_id'] = auth()->id();
        
        return $data;
    }

    protected function afterCreate(): void
    {
        // Invalidate relevant caches after creation
        cache()->tags(['posts'])->flush();
        PostStatsWidget::invalidateCache();
        
        // Queue heavy operations instead of doing them immediately
        ProcessNewPost::dispatch($this->record);
    }
}
```

## Frontend Performance

### 1. Asset Optimization

**Vite Configuration:**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['@alpinejs/alpinejs'],
                    filament: ['@filament/forms', '@filament/tables'],
                },
            },
        },
        chunkSizeWarningLimit: 1000,
        sourcemap: false,                    // Disable sourcemaps in production
    },
    server: {
        hmr: {
            host: 'localhost',
        },
    },
});
```

### 2. CSS Optimization

**Tailwind Configuration:**
```javascript
// tailwind.config.js
module.exports = {
    content: [
        './app/**/*.php',
        './resources/**/*.html',
        './resources/**/*.js',
        './resources/**/*.vue',
        './resources/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
    ],
    theme: {
        extend: {},
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
    corePlugins: {
        preflight: false,                    // Disable if conflicts with Filament
    },
}
```

### 3. JavaScript Optimization

**Alpine.js Performance:**
```javascript
// resources/js/app.js
import Alpine from 'alpinejs'
import collapse from '@alpinejs/collapse'
import intersect from '@alpinejs/intersect'

// Only register needed plugins
Alpine.plugin(collapse)
Alpine.plugin(intersect)

// Optimize directive loading
Alpine.directive('tooltip', (el, { expression }, { evaluate }) => {
    // Lazy load tooltip functionality
})

// Start Alpine
Alpine.start()
```

## Caching Strategies

### 1. Multi-Level Caching

**Comprehensive Caching Strategy:**
```php
class CacheManager
{
    // Application-level caching
    public static function getPostsForCategory(int $categoryId, int $perPage = 15): LengthAwarePaginator
    {
        $cacheKey = "category.{$categoryId}.posts.page." . request('page', 1);
        
        return cache()->remember($cacheKey, 1800, function () use ($categoryId, $perPage) {
            return Post::with(['user:id,name', 'tags:id,name'])
                ->where('category_id', $categoryId)
                ->where('status', 'published')
                ->orderByDesc('published_at')
                ->paginate($perPage);
        });
    }

    // Model-level caching
    public static function getPopularTags(int $limit = 20): Collection
    {
        return cache()->remember("popular_tags_{$limit}", 3600, function () use ($limit) {
            return Tag::withCount('posts')
                ->orderByDesc('posts_count')
                ->limit($limit)
                ->get();
        });
    }

    // Database-level caching (query result cache)
    public static function getCategoryStats(): array
    {
        return cache()->remember('category_stats', 1800, function () {
            return DB::table('posts')
                ->join('categories', 'posts.category_id', '=', 'categories.id')
                ->selectRaw('categories.name, COUNT(*) as post_count')
                ->where('posts.status', 'published')
                ->groupBy('categories.id', 'categories.name')
                ->orderByDesc('post_count')
                ->get()
                ->toArray();
        });
    }

    // Cache invalidation
    public static function invalidatePostCaches(Post $post): void
    {
        cache()->forget("category.{$post->category_id}.posts.page.1");
        cache()->tags(['posts', "category.{$post->category_id}"])->flush();
    }
}
```

### 2. Redis Optimization

**Redis Configuration:**
```php
// config/database.php
'redis' => [
    'client' => 'phpredis',              // Use phpredis for better performance
    
    'options' => [
        'cluster' => 'redis',
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
        'read_write_timeout' => 60,
        'context' => [
            'auth' => [env('REDIS_PASSWORD', null)],
        ],
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

    'session' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_SESSION_DB', '2'),
    ],
],
```

## Performance Monitoring

### 1. Laravel Boost Monitoring

**Performance Testing Commands:**
```
"Check for slow database queries in the application"
"Monitor memory usage during bulk operations"  
"Test query performance for the posts listing"
"Check if caching is working properly"
"Find N+1 query problems in the admin panel"
"Monitor widget loading times on dashboard"
```

### 2. Performance Metrics

**Custom Performance Monitoring:**
```php
class PerformanceMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage(true);
        
        $response = $next($request);
        
        $endTime = microtime(true);
        $endMemory = memory_get_usage(true);
        
        $executionTime = ($endTime - $startTime) * 1000; // Convert to milliseconds
        $memoryUsage = $endMemory - $startMemory;
        
        // Log slow requests
        if ($executionTime > 1000) { // Requests slower than 1 second
            Log::warning('Slow request detected', [
                'url' => $request->fullUrl(),
                'method' => $request->method(),
                'execution_time' => $executionTime,
                'memory_usage' => $memoryUsage,
                'user_id' => auth()->id(),
            ]);
        }
        
        // Add performance headers for debugging
        if (config('app.debug')) {
            $response->headers->set('X-Execution-Time', $executionTime);
            $response->headers->set('X-Memory-Usage', $memoryUsage);
        }
        
        return $response;
    }
}
```

### 3. Database Query Monitoring

**Query Performance Tracking:**
```php
class DatabaseQueryMonitor
{
    public static function enable(): void
    {
        DB::listen(function ($query) {
            if ($query->time > 1000) { // Queries slower than 1 second
                Log::warning('Slow database query', [
                    'sql' => $query->sql,
                    'bindings' => $query->bindings,
                    'time' => $query->time,
                    'connection' => $query->connectionName,
                ]);
            }
        });
    }

    public static function logQueryCounts(): void
    {
        $queryCount = DB::getQueryLog() ? count(DB::getQueryLog()) : 0;
        
        if ($queryCount > 50) { // More than 50 queries per request
            Log::warning('High query count detected', [
                'query_count' => $queryCount,
                'url' => request()->fullUrl(),
            ]);
        }
    }
}
```

## Production Optimization

### 1. Deployment Optimization

**Optimized Deployment Commands:**
```bash
#!/bin/bash
# deploy-optimize.sh

# Clear and cache configuration
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

# Optimize Composer autoloader
composer install --optimize-autoloader --no-dev

# Build optimized assets
npm ci --only=production
npm run build

# Queue optimizations
php artisan queue:restart

# Clear application cache if needed
php artisan cache:clear

# Warm up application
php artisan route:list > /dev/null
php artisan config:show > /dev/null
```

### 2. Server-Level Optimization

**PHP Configuration (php.ini):**
```ini
; Memory optimization
memory_limit = 512M
max_execution_time = 300

; OPcache optimization
opcache.enable = 1
opcache.memory_consumption = 256
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 20000
opcache.validate_timestamps = 0
opcache.revalidate_freq = 0
opcache.save_comments = 0

; File uploads
upload_max_filesize = 100M
post_max_size = 100M
max_file_uploads = 20

; Session optimization
session.save_handler = redis
session.save_path = "tcp://127.0.0.1:6379"
```

### 3. Queue Optimization

**Efficient Queue Processing:**
```php
class ProcessPostAnalytics implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $timeout = 300;
    public $maxExceptions = 3;
    public $backoff = [30, 60, 120];

    public function handle(): void
    {
        // Batch process analytics
        Post::chunk(1000, function ($posts) {
            $analytics = [];
            
            foreach ($posts as $post) {
                $analytics[] = $this->calculateAnalytics($post);
            }
            
            // Bulk insert analytics
            DB::table('post_analytics')->insert($analytics);
        });
    }

    public function failed(\Throwable $exception): void
    {
        Log::error('Analytics processing failed', [
            'exception' => $exception->getMessage(),
            'job' => static::class,
        ]);
    }
}
```

## Best Practices Summary

### 1. Database Optimization
- Use eager loading to prevent N+1 queries
- Create strategic database indexes
- Implement query result caching
- Use database chunking for large datasets

### 2. Application Performance
- Cache expensive computations
- Use lazy loading for heavy components
- Implement proper memory management
- Optimize service providers

### 3. Filament Optimization
- Limit resource query scope
- Use pagination appropriately
- Implement widget caching
- Optimize form loading

### 4. Monitoring and Testing
- Use Laravel Boost for performance insights
- Implement performance middleware
- Monitor slow queries and requests
- Regular performance testing

## Next Steps

1. **Master [Debugging Techniques](19-debugging-techniques.md)** for performance troubleshooting
2. **Review [File Navigation](18-file-navigation.md)** for efficient development workflows
3. **Practice [Testing Strategies](16-testing-strategies.md)** for performance validation

Performance optimization is an ongoing process that requires continuous monitoring, testing, and refinement to ensure Laravel + Filament applications scale effectively and provide excellent user experience.