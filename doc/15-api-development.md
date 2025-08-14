# [🔙](README.md) API Development with Laravel and Filament

This guide covers building robust APIs in Laravel 12 with Filament admin integration, including API authentication, resource transformation, testing strategies, and Laravel Boost integration.

## Laravel API Foundation

### 1. API Route Setup

**API Route Organization:**
```php
// routes/api.php
use App\Http\Controllers\Api\V1\PostController;
use App\Http\Controllers\Api\V1\AuthController;
use App\Http\Controllers\Api\V1\UserController;
use Illuminate\Support\Facades\Route;

// API versioning
Route::prefix('v1')->group(function () {
    // Public routes
    Route::post('/auth/login', [AuthController::class, 'login']);
    Route::post('/auth/register', [AuthController::class, 'register']);
    Route::post('/auth/forgot-password', [AuthController::class, 'forgotPassword']);
    
    // Public content endpoints
    Route::get('/posts', [PostController::class, 'index']);
    Route::get('/posts/{post:slug}', [PostController::class, 'show']);
    Route::get('/categories', [CategoryController::class, 'index']);
    
    // Protected routes
    Route::middleware('auth:sanctum')->group(function () {
        // Authentication
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/user', [AuthController::class, 'user']);
        
        // User management
        Route::apiResource('users', UserController::class);
        Route::get('/profile', [UserController::class, 'profile']);
        Route::put('/profile', [UserController::class, 'updateProfile']);
        
        // Content management
        Route::apiResource('posts', PostController::class)->except(['index', 'show']);
        Route::post('/posts/{post}/publish', [PostController::class, 'publish']);
        Route::post('/posts/{post}/unpublish', [PostController::class, 'unpublish']);
        
        // Comments
        Route::apiResource('posts.comments', CommentController::class);
        Route::post('/comments/{comment}/approve', [CommentController::class, 'approve']);
    });
});
```

### 2. API Controllers

**Base API Controller:**
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Response;

abstract class BaseApiController extends Controller
{
    protected function successResponse($data = null, string $message = 'Success', int $code = 200): JsonResponse
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data' => $data,
        ], $code);
    }

    protected function errorResponse(string $message = 'Error', int $code = 400, array $errors = []): JsonResponse
    {
        $response = [
            'success' => false,
            'message' => $message,
        ];

        if (!empty($errors)) {
            $response['errors'] = $errors;
        }

        return response()->json($response, $code);
    }

    protected function paginatedResponse($data, string $message = 'Success'): JsonResponse
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data' => $data->items(),
            'pagination' => [
                'current_page' => $data->currentPage(),
                'per_page' => $data->perPage(),
                'total' => $data->total(),
                'last_page' => $data->lastPage(),
                'next_page_url' => $data->nextPageUrl(),
                'prev_page_url' => $data->previousPageUrl(),
            ],
        ]);
    }

    protected function createdResponse($data, string $message = 'Created successfully'): JsonResponse
    {
        return $this->successResponse($data, $message, 201);
    }

    protected function noContentResponse(string $message = 'No content'): JsonResponse
    {
        return $this->successResponse(null, $message, 204);
    }

    protected function unauthorizedResponse(string $message = 'Unauthorized'): JsonResponse
    {
        return $this->errorResponse($message, 401);
    }

    protected function forbiddenResponse(string $message = 'Forbidden'): JsonResponse
    {
        return $this->errorResponse($message, 403);
    }

    protected function notFoundResponse(string $message = 'Not found'): JsonResponse
    {
        return $this->errorResponse($message, 404);
    }

    protected function validationErrorResponse(array $errors, string $message = 'Validation failed'): JsonResponse
    {
        return $this->errorResponse($message, 422, $errors);
    }
}
```

**Post API Controller:**
```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Api\BaseApiController;
use App\Http\Requests\Api\StorePostRequest;
use App\Http\Requests\Api\UpdatePostRequest;
use App\Http\Resources\PostCollection;
use App\Http\Resources\PostResource;
use App\Models\Post;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class PostController extends BaseApiController
{
    public function index(Request $request): JsonResponse
    {
        $posts = Post::with(['user:id,name', 'category:id,name', 'tags:id,name'])
            ->when($request->search, function ($query, $search) {
                $query->where('title', 'like', "%{$search}%")
                      ->orWhere('content', 'like', "%{$search}%");
            })
            ->when($request->category, function ($query, $category) {
                $query->whereHas('category', function ($q) use ($category) {
                    $q->where('slug', $category);
                });
            })
            ->when($request->tag, function ($query, $tag) {
                $query->whereHas('tags', function ($q) use ($tag) {
                    $q->where('slug', $tag);
                });
            })
            ->when($request->status, function ($query, $status) {
                $query->where('status', $status);
            })
            ->when($request->sort, function ($query, $sort) {
                match ($sort) {
                    'title' => $query->orderBy('title'),
                    'date' => $query->orderByDesc('published_at'),
                    'popular' => $query->withCount('comments')->orderByDesc('comments_count'),
                    default => $query->latest(),
                };
            }, function ($query) {
                $query->latest();
            })
            ->where('status', 'published')
            ->paginate($request->per_page ?? 15);

        return $this->paginatedResponse(new PostCollection($posts));
    }

    public function show(Post $post): JsonResponse
    {
        if ($post->status !== 'published' && $post->user_id !== auth()->id()) {
            return $this->notFoundResponse();
        }

        $post->load(['user:id,name,email', 'category', 'tags', 'comments.user:id,name']);

        return $this->successResponse(new PostResource($post));
    }

    public function store(StorePostRequest $request): JsonResponse
    {
        $validated = $request->validated();
        $validated['user_id'] = auth()->id();

        $post = Post::create($validated);
        $post->load(['user:id,name', 'category', 'tags']);

        return $this->createdResponse(new PostResource($post), 'Post created successfully');
    }

    public function update(UpdatePostRequest $request, Post $post): JsonResponse
    {
        $this->authorize('update', $post);

        $post->update($request->validated());
        $post->load(['user:id,name', 'category', 'tags']);

        return $this->successResponse(new PostResource($post), 'Post updated successfully');
    }

    public function destroy(Post $post): JsonResponse
    {
        $this->authorize('delete', $post);

        $post->delete();

        return $this->noContentResponse('Post deleted successfully');
    }

    public function publish(Post $post): JsonResponse
    {
        $this->authorize('publish', $post);

        $post->update([
            'status' => 'published',
            'published_at' => now(),
        ]);

        return $this->successResponse(new PostResource($post), 'Post published successfully');
    }

    public function unpublish(Post $post): JsonResponse
    {
        $this->authorize('unpublish', $post);

        $post->update([
            'status' => 'draft',
            'published_at' => null,
        ]);

        return $this->successResponse(new PostResource($post), 'Post unpublished successfully');
    }
}
```

## API Resources and Transformation

### 1. Eloquent API Resources

**Post Resource:**
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => $this->excerpt,
            'content' => $this->when($request->routeIs('api.v1.posts.show'), $this->content),
            'status' => $this->status,
            'featured' => $this->featured,
            'published_at' => $this->published_at?->toISOString(),
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
            
            // Relationships
            'author' => new UserResource($this->whenLoaded('user')),
            'category' => new CategoryResource($this->whenLoaded('category')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'comments' => CommentResource::collection($this->whenLoaded('comments')),
            
            // Computed attributes
            'reading_time' => $this->reading_time,
            'comment_count' => $this->when(
                $this->relationLoaded('comments'),
                $this->comments->count()
            ),
            
            // Conditional fields based on permissions
            'can_edit' => $this->when(
                auth('sanctum')->check(),
                auth('sanctum')->user()->can('update', $this->resource)
            ),
            'can_delete' => $this->when(
                auth('sanctum')->check(),
                auth('sanctum')->user()->can('delete', $this->resource)
            ),
            'can_publish' => $this->when(
                auth('sanctum')->check(),
                auth('sanctum')->user()->can('publish', $this->resource)
            ),
            
            // URLs
            'url' => route('posts.show', $this->slug),
            'api_url' => route('api.v1.posts.show', $this->resource),
            
            // Media
            'featured_image' => $this->when(
                $this->featured_image,
                [
                    'url' => $this->featured_image_url,
                    'alt' => $this->featured_image_alt,
                    'caption' => $this->featured_image_caption,
                ]
            ),
        ];
    }

    public function with(Request $request): array
    {
        return [
            'version' => '1.0',
            'api_documentation' => route('api.documentation'),
        ];
    }
}
```

**Post Collection Resource:**
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class PostCollection extends ResourceCollection
{
    public function toArray(Request $request): array
    {
        return [
            'posts' => $this->collection->transform(function ($post) {
                return [
                    'id' => $post->id,
                    'title' => $post->title,
                    'slug' => $post->slug,
                    'excerpt' => $post->excerpt,
                    'status' => $post->status,
                    'featured' => $post->featured,
                    'published_at' => $post->published_at?->toISOString(),
                    'reading_time' => $post->reading_time,
                    'author' => [
                        'id' => $post->user->id,
                        'name' => $post->user->name,
                    ],
                    'category' => [
                        'id' => $post->category->id,
                        'name' => $post->category->name,
                        'slug' => $post->category->slug,
                    ],
                    'tags' => $post->tags->map(function ($tag) {
                        return [
                            'id' => $tag->id,
                            'name' => $tag->name,
                            'slug' => $tag->slug,
                        ];
                    }),
                    'url' => route('posts.show', $post->slug),
                    'api_url' => route('api.v1.posts.show', $post),
                ];
            }),
        ];
    }

    public function with(Request $request): array
    {
        return [
            'meta' => [
                'total_posts' => $this->collection->count(),
                'filters_applied' => array_filter([
                    'search' => $request->search,
                    'category' => $request->category,
                    'tag' => $request->tag,
                    'status' => $request->status,
                ]),
                'available_filters' => [
                    'categories' => route('api.v1.categories.index'),
                    'tags' => route('api.v1.tags.index'),
                    'statuses' => ['published', 'draft'],
                ],
            ],
        ];
    }
}
```

### 2. Request Validation

**Store Post Request:**
```php
<?php

namespace App\Http\Requests\Api;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;
use Illuminate\Contracts\Validation\Validator;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return auth('sanctum')->check() && auth('sanctum')->user()->can('create', Post::class);
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'slug' => 'nullable|string|max:255|unique:posts,slug|alpha_dash',
            'excerpt' => 'nullable|string|max:500',
            'content' => 'required|string',
            'status' => 'in:draft,published',
            'featured' => 'boolean',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'array',
            'tags.*' => 'exists:tags,id',
            'published_at' => 'nullable|date|after:now',
            'featured_image' => 'nullable|image|max:2048',
            'meta_title' => 'nullable|string|max:60',
            'meta_description' => 'nullable|string|max:160',
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Post title is required',
            'content.required' => 'Post content is required',
            'category_id.required' => 'Please select a category',
            'category_id.exists' => 'Selected category does not exist',
            'tags.*.exists' => 'One or more selected tags do not exist',
            'published_at.after' => 'Published date must be in the future',
            'featured_image.image' => 'Featured image must be a valid image file',
            'featured_image.max' => 'Featured image size cannot exceed 2MB',
        ];
    }

    protected function prepareForValidation(): void
    {
        if (!$this->has('slug') && $this->has('title')) {
            $this->merge([
                'slug' => \Str::slug($this->title),
            ]);
        }

        if (!$this->has('status')) {
            $this->merge([
                'status' => 'draft',
            ]);
        }
    }

    protected function failedValidation(Validator $validator): void
    {
        $response = response()->json([
            'success' => false,
            'message' => 'Validation failed',
            'errors' => $validator->errors(),
        ], 422);

        throw new HttpResponseException($response);
    }
}
```

## API Authentication

### 1. Sanctum Setup

**Authentication Controller:**
```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Api\BaseApiController;
use App\Http\Requests\Api\LoginRequest;
use App\Http\Requests\Api\RegisterRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends BaseApiController
{
    public function register(RegisterRequest $request): JsonResponse
    {
        $validated = $request->validated();
        $validated['password'] = Hash::make($validated['password']);

        $user = User::create($validated);

        $token = $user->createToken('api-token', ['*'])->plainTextToken;

        return $this->createdResponse([
            'user' => new UserResource($user),
            'token' => $token,
            'token_type' => 'Bearer',
        ], 'Registration successful');
    }

    public function login(LoginRequest $request): JsonResponse
    {
        $credentials = $request->only('email', 'password');

        $user = User::where('email', $credentials['email'])->first();

        if (!$user || !Hash::check($credentials['password'], $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        if (!$user->is_active) {
            return $this->forbiddenResponse('Account is deactivated');
        }

        // Revoke existing tokens if requested
        if ($request->revoke_existing_tokens) {
            $user->tokens()->delete();
        }

        $token = $user->createToken('api-token', $this->getTokenAbilities($user))->plainTextToken;

        // Update last login
        $user->update(['last_login_at' => now()]);

        return $this->successResponse([
            'user' => new UserResource($user),
            'token' => $token,
            'token_type' => 'Bearer',
            'expires_at' => now()->addDays(30)->toISOString(),
        ], 'Login successful');
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return $this->successResponse(null, 'Logout successful');
    }

    public function user(Request $request): JsonResponse
    {
        return $this->successResponse(new UserResource($request->user()));
    }

    public function refreshToken(Request $request): JsonResponse
    {
        $user = $request->user();
        
        // Delete current token
        $request->user()->currentAccessToken()->delete();
        
        // Create new token
        $token = $user->createToken('api-token', $this->getTokenAbilities($user))->plainTextToken;

        return $this->successResponse([
            'token' => $token,
            'token_type' => 'Bearer',
            'expires_at' => now()->addDays(30)->toISOString(),
        ], 'Token refreshed successfully');
    }

    private function getTokenAbilities(User $user): array
    {
        $abilities = ['read'];

        if ($user->hasRole('admin')) {
            $abilities = ['*'];
        } elseif ($user->hasRole('editor')) {
            $abilities = ['read', 'write', 'publish'];
        } elseif ($user->hasRole('author')) {
            $abilities = ['read', 'write'];
        }

        return $abilities;
    }
}
```

### 2. Token Management

**Token Scopes and Abilities:**
```php
// In AuthServiceProvider
use Laravel\Sanctum\Sanctum;

public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
    
    // Define token abilities
    Gate::define('posts:read', function ($user) {
        return $user->tokenCan('read') || $user->tokenCan('*');
    });

    Gate::define('posts:write', function ($user) {
        return $user->tokenCan('write') || $user->tokenCan('*');
    });

    Gate::define('posts:publish', function ($user) {
        return $user->tokenCan('publish') || $user->tokenCan('*');
    });
}
```

**Custom Token Middleware:**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckTokenAbility
{
    public function handle(Request $request, Closure $next, string $ability): Response
    {
        if (!$request->user() || !$request->user()->tokenCan($ability)) {
            return response()->json([
                'success' => false,
                'message' => 'Insufficient token permissions',
            ], 403);
        }

        return $next($request);
    }
}
```

## API Testing

### 1. Feature Tests

**Post API Tests:**
```php
<?php

namespace Tests\Feature\Api;

use App\Models\Post;
use App\Models\User;
use App\Models\Category;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Laravel\Sanctum\Sanctum;
use Tests\TestCase;

class PostApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_get_published_posts(): void
    {
        Post::factory()
            ->published()
            ->count(5)
            ->create();

        $response = $this->getJson('/api/v1/posts');

        $response->assertOk()
                ->assertJsonStructure([
                    'success',
                    'message',
                    'data' => [
                        'posts' => [
                            '*' => [
                                'id',
                                'title',
                                'slug',
                                'excerpt',
                                'status',
                                'published_at',
                                'author' => ['id', 'name'],
                                'category' => ['id', 'name', 'slug'],
                                'tags',
                            ],
                        ],
                    ],
                    'pagination',
                ]);
    }

    public function test_can_get_single_post(): void
    {
        $post = Post::factory()->published()->create();

        $response = $this->getJson("/api/v1/posts/{$post->slug}");

        $response->assertOk()
                ->assertJsonFragment([
                    'title' => $post->title,
                    'content' => $post->content,
                ]);
    }

    public function test_authenticated_user_can_create_post(): void
    {
        $user = User::factory()->create();
        $category = Category::factory()->create();
        
        Sanctum::actingAs($user, ['write']);

        $postData = [
            'title' => 'Test Post',
            'content' => 'This is test content',
            'category_id' => $category->id,
        ];

        $response = $this->postJson('/api/v1/posts', $postData);

        $response->assertCreated()
                ->assertJsonFragment([
                    'title' => 'Test Post',
                    'status' => 'draft',
                ]);

        $this->assertDatabaseHas('posts', [
            'title' => 'Test Post',
            'user_id' => $user->id,
        ]);
    }

    public function test_unauthenticated_user_cannot_create_post(): void
    {
        $category = Category::factory()->create();

        $postData = [
            'title' => 'Test Post',
            'content' => 'This is test content',
            'category_id' => $category->id,
        ];

        $response = $this->postJson('/api/v1/posts', $postData);

        $response->assertUnauthorized();
    }

    public function test_user_can_only_update_own_posts(): void
    {
        $user = User::factory()->create();
        $otherUser = User::factory()->create();
        $post = Post::factory()->create(['user_id' => $otherUser->id]);

        Sanctum::actingAs($user, ['write']);

        $response = $this->putJson("/api/v1/posts/{$post->id}", [
            'title' => 'Updated Title',
        ]);

        $response->assertForbidden();
    }

    public function test_can_search_posts(): void
    {
        Post::factory()->published()->create(['title' => 'Laravel Tutorial']);
        Post::factory()->published()->create(['title' => 'Vue.js Guide']);
        Post::factory()->published()->create(['title' => 'PHP Best Practices']);

        $response = $this->getJson('/api/v1/posts?search=Laravel');

        $response->assertOk();
        
        $posts = $response->json('data.posts');
        $this->assertCount(1, $posts);
        $this->assertStringContainsString('Laravel', $posts[0]['title']);
    }

    public function test_can_filter_posts_by_category(): void
    {
        $category = Category::factory()->create(['slug' => 'tutorials']);
        
        Post::factory()->published()->create(['category_id' => $category->id]);
        Post::factory()->published()->create(); // Different category

        $response = $this->getJson('/api/v1/posts?category=tutorials');

        $response->assertOk();
        
        $posts = $response->json('data.posts');
        $this->assertCount(1, $posts);
        $this->assertEquals('tutorials', $posts[0]['category']['slug']);
    }
}
```

### 2. API Testing with Laravel Boost

**Test API Endpoints:**
```
"Test the posts API endpoint with different filters"
"Verify API authentication works with Sanctum tokens"
"Check API response structure for posts endpoint"
"Test API validation errors for post creation"
"Verify API permissions work correctly"
```

**Performance Testing:**
```
"Test API response times for large datasets"
"Check API memory usage during bulk operations"
"Monitor API query performance"
"Test API rate limiting functionality"
```

## Filament Integration with API

### 1. API Management in Filament

**API Token Resource:**
```php
<?php

namespace App\Filament\Resources;

use App\Models\PersonalAccessToken;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;

class ApiTokenResource extends Resource
{
    protected static ?string $model = PersonalAccessToken::class;
    
    protected static ?string $navigationIcon = 'heroicon-o-key';
    
    protected static ?string $navigationGroup = 'API Management';

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('name')
                    ->required()
                    ->maxLength(255),
                    
                Forms\Components\Select::make('tokenable_id')
                    ->label('User')
                    ->relationship('tokenable', 'name')
                    ->required(),
                    
                Forms\Components\CheckboxList::make('abilities')
                    ->options([
                        'read' => 'Read',
                        'write' => 'Write',
                        'publish' => 'Publish',
                        'admin' => 'Admin',
                        '*' => 'All Permissions',
                    ])
                    ->columns(2),
                    
                Forms\Components\DateTimePicker::make('expires_at')
                    ->label('Expires At')
                    ->nullable(),
            ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('name')
                    ->searchable(),
                    
                Tables\Columns\TextColumn::make('tokenable.name')
                    ->label('User')
                    ->searchable(),
                    
                Tables\Columns\BadgeColumn::make('abilities')
                    ->getStateUsing(fn ($record) => implode(', ', $record->abilities ?? []))
                    ->colors([
                        'success' => fn ($state) => str_contains($state, '*'),
                        'warning' => fn ($state) => str_contains($state, 'admin'),
                        'primary' => fn ($state) => true,
                    ]),
                    
                Tables\Columns\TextColumn::make('last_used_at')
                    ->dateTime()
                    ->sortable()
                    ->since(),
                    
                Tables\Columns\TextColumn::make('expires_at')
                    ->dateTime()
                    ->sortable(),
                    
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable(),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('tokenable_id')
                    ->label('User')
                    ->relationship('tokenable', 'name'),
                    
                Tables\Filters\Filter::make('expired')
                    ->query(fn ($query) => $query->where('expires_at', '<', now()))
                    ->label('Expired Tokens'),
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
                
                Tables\Actions\Action::make('revoke')
                    ->action(fn ($record) => $record->delete())
                    ->requiresConfirmation()
                    ->color('danger')
                    ->icon('heroicon-o-x-mark'),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }
}
```

### 2. API Statistics Widget

**API Usage Widget:**
```php
<?php

namespace App\Filament\Widgets;

use App\Models\PersonalAccessToken;
use App\Models\Post;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;
use Illuminate\Support\Facades\DB;

class ApiStatsWidget extends BaseWidget
{
    protected function getStats(): array
    {
        // Get API usage statistics
        $activeTokens = PersonalAccessToken::whereNull('expires_at')
            ->orWhere('expires_at', '>', now())
            ->count();

        $recentApiRequests = DB::table('personal_access_tokens')
            ->where('last_used_at', '>', now()->subDay())
            ->count();

        $totalApiPosts = Post::whereNotNull('api_created_at')->count();

        return [
            Stat::make('Active API Tokens', $activeTokens)
                ->description('Currently valid tokens')
                ->descriptionIcon('heroicon-m-key')
                ->color('success'),
                
            Stat::make('Recent API Requests', $recentApiRequests)
                ->description('Last 24 hours')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->color('primary'),
                
            Stat::make('API Created Posts', $totalApiPosts)
                ->description('Posts created via API')
                ->descriptionIcon('heroicon-m-document-text')
                ->color('info'),
        ];
    }
}
```

## API Documentation

### 1. OpenAPI/Swagger Integration

**API Documentation Controller:**
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;

class DocumentationController extends Controller
{
    public function index(): JsonResponse
    {
        $documentation = [
            'openapi' => '3.0.0',
            'info' => [
                'title' => 'Blog API',
                'version' => '1.0.0',
                'description' => 'RESTful API for blog management',
                'contact' => [
                    'name' => 'API Support',
                    'email' => 'support@example.com',
                ],
            ],
            'servers' => [
                [
                    'url' => url('/api/v1'),
                    'description' => 'Production server',
                ],
            ],
            'paths' => $this->getPaths(),
            'components' => $this->getComponents(),
        ];

        return response()->json($documentation);
    }

    private function getPaths(): array
    {
        return [
            '/posts' => [
                'get' => [
                    'summary' => 'Get all posts',
                    'parameters' => [
                        [
                            'name' => 'search',
                            'in' => 'query',
                            'schema' => ['type' => 'string'],
                            'description' => 'Search in title and content',
                        ],
                        [
                            'name' => 'category',
                            'in' => 'query',
                            'schema' => ['type' => 'string'],
                            'description' => 'Filter by category slug',
                        ],
                        [
                            'name' => 'per_page',
                            'in' => 'query',
                            'schema' => ['type' => 'integer', 'default' => 15],
                            'description' => 'Number of items per page',
                        ],
                    ],
                    'responses' => [
                        '200' => [
                            'description' => 'Successful response',
                            'content' => [
                                'application/json' => [
                                    'schema' => [
                                        '$ref' => '#/components/schemas/PostCollection',
                                    ],
                                ],
                            ],
                        ],
                    ],
                ],
                'post' => [
                    'summary' => 'Create a new post',
                    'security' => [['sanctum' => []]],
                    'requestBody' => [
                        'content' => [
                            'application/json' => [
                                'schema' => [
                                    '$ref' => '#/components/schemas/CreatePostRequest',
                                ],
                            ],
                        ],
                    ],
                    'responses' => [
                        '201' => [
                            'description' => 'Post created successfully',
                            'content' => [
                                'application/json' => [
                                    'schema' => [
                                        '$ref' => '#/components/schemas/PostResource',
                                    ],
                                ],
                            ],
                        ],
                        '422' => [
                            'description' => 'Validation error',
                        ],
                    ],
                ],
            ],
        ];
    }

    private function getComponents(): array
    {
        return [
            'schemas' => [
                'PostResource' => [
                    'type' => 'object',
                    'properties' => [
                        'id' => ['type' => 'integer'],
                        'title' => ['type' => 'string'],
                        'slug' => ['type' => 'string'],
                        'content' => ['type' => 'string'],
                        'status' => ['type' => 'string', 'enum' => ['draft', 'published']],
                        'published_at' => ['type' => 'string', 'format' => 'date-time'],
                        'author' => ['$ref' => '#/components/schemas/UserResource'],
                        'category' => ['$ref' => '#/components/schemas/CategoryResource'],
                    ],
                ],
                'CreatePostRequest' => [
                    'type' => 'object',
                    'required' => ['title', 'content', 'category_id'],
                    'properties' => [
                        'title' => ['type' => 'string', 'maxLength' => 255],
                        'content' => ['type' => 'string'],
                        'category_id' => ['type' => 'integer'],
                        'status' => ['type' => 'string', 'enum' => ['draft', 'published']],
                    ],
                ],
            ],
            'securitySchemes' => [
                'sanctum' => [
                    'type' => 'http',
                    'scheme' => 'bearer',
                    'bearerFormat' => 'JWT',
                ],
            ],
        ];
    }
}
```

## API Performance and Security

### 1. Rate Limiting

**Custom Rate Limiting:**
```php
// In RouteServiceProvider
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return $request->user()
            ? Limit::perMinute(100)->by($request->user()->id)
            : Limit::perMinute(20)->by($request->ip());
    });

    RateLimiter::for('auth', function (Request $request) {
        return Limit::perMinute(5)->by($request->ip());
    });
}
```

### 2. API Security Middleware

**API Security Headers:**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ApiSecurityHeaders
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        
        // CORS headers for API
        if ($request->is('api/*')) {
            $response->headers->set('Access-Control-Allow-Origin', config('cors.allowed_origins')[0] ?? '*');
            $response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
            $response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
        }

        return $response;
    }
}
```

## Best Practices

### 1. API Design Principles

**RESTful Design:**
- Use appropriate HTTP methods (GET, POST, PUT, DELETE)
- Return meaningful HTTP status codes
- Implement consistent URL patterns
- Provide proper error messages
- Use resource-based URLs

### 2. Performance Optimization

**API Optimization:**
- Implement caching for expensive operations
- Use eager loading to prevent N+1 queries
- Paginate large result sets
- Compress responses when appropriate
- Monitor API performance metrics

### 3. Security Best Practices

**API Security:**
- Implement proper authentication and authorization
- Use HTTPS in production
- Validate all input data
- Implement rate limiting
- Log security events and API access

## Next Steps

1. **Master [Testing Strategies](16-testing-strategies.md)** for comprehensive API testing
2. **Learn [Authentication & Authorization](14-auth-authorization.md)** for advanced API security
3. **Practice [Performance Optimization](20-performance-optimization.md)** for API scaling

API development with Laravel and Filament provides powerful tools for building robust, secure, and performant APIs while maintaining excellent admin interfaces for API management and monitoring.