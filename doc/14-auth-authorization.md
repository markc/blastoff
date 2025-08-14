# [🔙](README.md) Authentication and Authorization in Laravel and Filament

This guide covers implementing comprehensive authentication and authorization systems in Laravel 12 with Filament 4.x, including role-based access control, policies, and security best practices.

## Laravel Authentication Foundation

### 1. Authentication Setup

**Basic Authentication Configuration:**
```php
// config/auth.php
return [
    'defaults' => [
        'guard' => 'web',
        'passwords' => 'users',
    ],

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
    ],
];
```

**User Model Enhancement:**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'email_verified_at',
        'is_active',
        'last_login_at',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'last_login_at' => 'datetime',
            'is_active' => 'boolean',
            'password' => 'hashed',
        ];
    }

    // Role and permission relationships
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }

    public function permissions(): BelongsToMany
    {
        return $this->belongsToMany(Permission::class);
    }

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    // Authorization helper methods
    public function hasRole(string $role): bool
    {
        return $this->roles()->where('name', $role)->exists();
    }

    public function hasPermission(string $permission): bool
    {
        return $this->permissions()->where('name', $permission)->exists() ||
               $this->roles()->whereHas('permissions', function ($query) use ($permission) {
                   $query->where('name', $permission);
               })->exists();
    }

    public function hasAnyRole(array $roles): bool
    {
        return $this->roles()->whereIn('name', $roles)->exists();
    }

    public function canAccessFilament(): bool
    {
        return $this->hasRole('admin') || $this->hasRole('editor');
    }
}
```

### 2. Role-Based Access Control (RBAC)

**Role Model:**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    protected $fillable = [
        'name',
        'description',
        'is_active',
    ];

    protected function casts(): array
    {
        return [
            'is_active' => 'boolean',
        ];
    }

    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }

    public function permissions(): BelongsToMany
    {
        return $this->belongsToMany(Permission::class);
    }

    public function hasPermission(string $permission): bool
    {
        return $this->permissions()->where('name', $permission)->exists();
    }
}
```

**Permission Model:**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Permission extends Model
{
    protected $fillable = [
        'name',
        'description',
        'category',
    ];

    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }

    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

**Database Migrations:**
```php
// Migration: create_roles_table
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();
    $table->string('description')->nullable();
    $table->boolean('is_active')->default(true);
    $table->timestamps();
});

// Migration: create_permissions_table
Schema::create('permissions', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();
    $table->string('description')->nullable();
    $table->string('category')->nullable();
    $table->timestamps();
});

// Migration: create_role_user_table
Schema::create('role_user', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('role_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
});

// Migration: create_permission_role_table
Schema::create('permission_role', function (Blueprint $table) {
    $table->id();
    $table->foreignId('permission_id')->constrained()->cascadeOnDelete();
    $table->foreignId('role_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
});

// Migration: create_permission_user_table
Schema::create('permission_user', function (Blueprint $table) {
    $table->id();
    $table->foreignId('permission_id')->constrained()->cascadeOnDelete();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->timestamps();
});
```

## Laravel Policies

### 1. Policy Creation and Implementation

**Post Policy:**
```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function before(User $user, string $ability): bool|null
    {
        // Super admin can do anything
        if ($user->hasRole('super-admin')) {
            return true;
        }

        return null;
    }

    public function viewAny(User $user): bool
    {
        return $user->hasPermission('posts.view');
    }

    public function view(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.view') || 
               $this->owns($user, $post);
    }

    public function create(User $user): bool
    {
        return $user->hasPermission('posts.create');
    }

    public function update(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.update') || 
               ($user->hasPermission('posts.update-own') && $this->owns($user, $post));
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.delete') || 
               ($user->hasPermission('posts.delete-own') && $this->owns($user, $post));
    }

    public function restore(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.restore');
    }

    public function forceDelete(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.force-delete');
    }

    public function publish(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.publish');
    }

    public function unpublish(User $user, Post $post): bool
    {
        return $user->hasPermission('posts.unpublish');
    }

    private function owns(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

**Register Policies:**
```php
// app/Providers/AuthServiceProvider.php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Policies\PostPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    public function boot(): void
    {
        $this->registerPolicies();
    }
}
```

### 2. Advanced Policy Patterns

**Resource-Specific Policies:**
```php
class UserPolicy
{
    public function viewProfile(User $currentUser, User $targetUser): bool
    {
        // Users can view their own profile or admins can view any
        return $currentUser->id === $targetUser->id || 
               $currentUser->hasRole('admin');
    }

    public function updateProfile(User $currentUser, User $targetUser): bool
    {
        // Users can only update their own profile
        return $currentUser->id === $targetUser->id;
    }

    public function changeRole(User $currentUser, User $targetUser): bool
    {
        // Only super admins can change roles
        return $currentUser->hasRole('super-admin') && 
               $currentUser->id !== $targetUser->id;
    }

    public function viewSensitiveData(User $currentUser, User $targetUser): bool
    {
        // HR and admins can view sensitive data
        return $currentUser->hasAnyRole(['admin', 'hr']) && 
               $currentUser->id !== $targetUser->id;
    }
}
```

## Filament Authentication Integration

### 1. Panel Authentication

**Admin Panel Provider:**
```php
<?php

namespace App\Providers\Filament;

use Filament\Http\Middleware\Authenticate;
use Filament\Http\Middleware\DisableBladeIconComponents;
use Filament\Http\Middleware\DispatchServingFilamentEvent;
use Filament\Pages;
use Filament\Panel;
use Filament\PanelProvider;
use Filament\Support\Colors\Color;
use Filament\Widgets;
use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\AuthenticateSession;
use Illuminate\Session\Middleware\StartSession;
use Illuminate\View\Middleware\ShareErrorsFromSession;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->default()
            ->id('admin')
            ->path('/admin')
            ->login()
            ->colors([
                'primary' => Color::Amber,
            ])
            ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
            ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
            ->pages([
                Pages\Dashboard::class,
            ])
            ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets')
            ->widgets([
                Widgets\AccountWidget::class,
                Widgets\FilamentInfoWidget::class,
            ])
            ->middleware([
                EncryptCookies::class,
                AddQueuedCookiesToResponse::class,
                StartSession::class,
                AuthenticateSession::class,
                ShareErrorsFromSession::class,
                VerifyCsrfToken::class,
                SubstituteBindings::class,
                DisableBladeIconComponents::class,
                DispatchServingFilamentEvent::class,
            ])
            ->authMiddleware([
                Authenticate::class,
            ])
            ->authGuard('web')
            ->loginRouteSlug('login')
            ->registration()
            ->passwordReset()
            ->emailVerification()
            ->profile()
            ->tenantMiddleware([
                // Custom tenant middleware
            ]);
    }
}
```

### 2. Multi-Panel Authentication

**Editor Panel Provider:**
```php
class EditorPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('editor')
            ->path('/editor')
            ->login()
            ->colors([
                'primary' => Color::Blue,
            ])
            ->authMiddleware([
                Authenticate::class,
                EnsureUserHasRole::class.':editor',
            ])
            ->resources([
                // Limited resources for editors
                PostResource::class,
                MediaResource::class,
            ])
            ->pages([
                Pages\Dashboard::class,
            ]);
    }
}

// Custom middleware
class EnsureUserHasRole
{
    public function handle($request, Closure $next, ...$roles)
    {
        if (!auth()->user()->hasAnyRole($roles)) {
            abort(403, 'Unauthorized access.');
        }

        return $next($request);
    }
}
```

### 3. Resource-Level Authorization

**Post Resource with Authorization:**
```php
<?php

namespace App\Filament\Resources;

use App\Models\Post;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class PostResource extends Resource
{
    protected static ?string $model = Post::class;

    protected static ?string $navigationIcon = 'heroicon-o-document-text';

    public static function canViewAny(): bool
    {
        return auth()->user()->can('viewAny', Post::class);
    }

    public static function canCreate(): bool
    {
        return auth()->user()->can('create', Post::class);
    }

    public static function canEdit($record): bool
    {
        return auth()->user()->can('update', $record);
    }

    public static function canDelete($record): bool
    {
        return auth()->user()->can('delete', $record);
    }

    public static function getEloquentQuery(): Builder
    {
        $query = parent::getEloquentQuery();

        // Limit query based on user role
        if (!auth()->user()->hasRole('admin')) {
            $query->where('user_id', auth()->id());
        }

        return $query;
    }

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('title')
                    ->required()
                    ->maxLength(255),

                Forms\Components\Textarea::make('content')
                    ->required(),

                Forms\Components\Select::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                    ])
                    ->default('draft')
                    ->visible(fn () => auth()->user()->can('publish', Post::class)),

                Forms\Components\Hidden::make('user_id')
                    ->default(auth()->id()),
            ]);
    }

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('title')
                    ->searchable(),

                Tables\Columns\TextColumn::make('user.name')
                    ->label('Author')
                    ->visible(fn () => auth()->user()->hasRole('admin')),

                Tables\Columns\BadgeColumn::make('status')
                    ->colors([
                        'warning' => 'draft',
                        'success' => 'published',
                    ]),

                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable(),
            ])
            ->filters([
                Tables\Filters\SelectFilter::make('status')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                    ]),
            ])
            ->actions([
                Tables\Actions\EditAction::make()
                    ->visible(fn ($record) => auth()->user()->can('update', $record)),

                Tables\Actions\DeleteAction::make()
                    ->visible(fn ($record) => auth()->user()->can('delete', $record)),

                Tables\Actions\Action::make('publish')
                    ->action(fn (Post $record) => $record->update(['status' => 'published']))
                    ->requiresConfirmation()
                    ->visible(fn (Post $record) => 
                        $record->status === 'draft' && 
                        auth()->user()->can('publish', $record)
                    ),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make()
                        ->visible(fn () => auth()->user()->hasPermission('posts.delete')),
                ]),
            ]);
    }
}
```

## Advanced Authorization Patterns

### 1. Field-Level Permissions

**Conditional Field Display:**
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Forms\Components\TextInput::make('title')
                ->required(),

            Forms\Components\Select::make('category_id')
                ->relationship('category', 'name')
                ->visible(fn () => auth()->user()->hasPermission('posts.edit-category')),

            Forms\Components\TextInput::make('slug')
                ->visible(fn () => auth()->user()->hasRole('admin')),

            Forms\Components\DateTimePicker::make('published_at')
                ->visible(fn () => auth()->user()->hasPermission('posts.schedule')),

            Forms\Components\Toggle::make('featured')
                ->visible(fn () => auth()->user()->hasPermission('posts.feature')),

            Forms\Components\Section::make('SEO Settings')
                ->schema([
                    Forms\Components\TextInput::make('meta_title'),
                    Forms\Components\Textarea::make('meta_description'),
                ])
                ->visible(fn () => auth()->user()->hasPermission('posts.seo')),
        ]);
}
```

### 2. Dynamic Navigation

**Role-Based Navigation:**
```php
// In resource class
protected static function shouldRegisterNavigation(): bool
{
    return auth()->user()->hasPermission('posts.view');
}

protected static ?int $navigationSort = 10;

public static function getNavigationBadge(): ?string
{
    if (auth()->user()->hasRole('admin')) {
        return static::getModel()::count();
    }

    return static::getModel()::where('user_id', auth()->id())->count();
}

public static function getNavigationBadgeColor(): ?string
{
    return static::getModel()::where('status', 'draft')->count() > 0 ? 'warning' : 'success';
}
```

### 3. Tenant-Based Authorization

**Multi-Tenant Resource:**
```php
class OrganizationPostResource extends Resource
{
    protected static ?string $model = Post::class;

    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()
            ->where('organization_id', auth()->user()->organization_id);
    }

    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Hidden::make('organization_id')
                    ->default(auth()->user()->organization_id),

                Forms\Components\TextInput::make('title')
                    ->required(),

                Forms\Components\Select::make('user_id')
                    ->relationship('user', 'name')
                    ->options(fn () => 
                        User::where('organization_id', auth()->user()->organization_id)
                            ->pluck('name', 'id')
                    ),
            ]);
    }
}
```

## Security Best Practices

### 1. Authentication Security

**Enhanced User Model Security:**
```php
class User extends Authenticatable
{
    // Rate limiting for login attempts
    public function canAttemptLogin(): bool
    {
        $attempts = cache()->get("login_attempts_{$this->email}", 0);
        return $attempts < 5;
    }

    public function recordFailedLogin(): void
    {
        $key = "login_attempts_{$this->email}";
        $attempts = cache()->get($key, 0) + 1;
        cache()->put($key, $attempts, now()->addMinutes(15));
    }

    public function clearFailedLogins(): void
    {
        cache()->forget("login_attempts_{$this->email}");
    }

    // Session security
    public function logActivity(string $action, array $data = []): void
    {
        activity()
            ->performedOn($this)
            ->causedBy($this)
            ->withProperties($data)
            ->log($action);
    }

    // Password security
    public function mustChangePassword(): bool
    {
        return $this->password_changed_at?->lt(now()->subDays(90)) ?? true;
    }
}
```

### 2. Authorization Middleware

**Custom Authorization Middleware:**
```php
class EnsureUserCanAccess
{
    public function handle($request, Closure $next, $permission)
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }

        if (!auth()->user()->hasPermission($permission)) {
            abort(403, 'Insufficient permissions.');
        }

        // Log access for audit trail
        auth()->user()->logActivity("accessed_{$permission}", [
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
        ]);

        return $next($request);
    }
}
```

### 3. API Authentication

**API Token Management:**
```php
class ApiTokenController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'abilities' => 'array',
        ]);

        $token = auth()->user()->createToken(
            $request->name,
            $request->abilities ?? []
        );

        return response()->json([
            'token' => $token->plainTextToken,
            'abilities' => $token->accessToken->abilities,
        ]);
    }

    public function destroy(Request $request, $tokenId)
    {
        auth()->user()->tokens()->where('id', $tokenId)->delete();

        return response()->json(['message' => 'Token deleted successfully']);
    }
}
```

## Testing Authentication and Authorization

### 1. Authentication Testing

**Test with Claude Code:**
```
"Test user login with valid credentials"
"Verify user cannot access admin panel without proper role"
"Check if password reset functionality works"
"Test account lockout after failed login attempts"
```

### 2. Authorization Testing

**Policy Testing:**
```
"Test if user can only edit their own posts"
"Verify admin can view all posts regardless of owner"
"Check if editor role has correct permissions"
"Test permission inheritance from roles"
```

### 3. Security Testing

**Security Validation:**
```
"Test session security and timeout handling"
"Verify CSRF protection on all forms"
"Check if unauthorized API access is blocked"
"Test rate limiting for login attempts"
```

## Performance Optimization

### 1. Permission Caching

**Cache User Permissions:**
```php
class User extends Authenticatable
{
    public function getCachedPermissions(): Collection
    {
        return cache()->remember(
            "user_permissions_{$this->id}",
            3600,
            fn () => $this->getAllPermissions()
        );
    }

    public function getAllPermissions(): Collection
    {
        return $this->permissions()
            ->get()
            ->merge(
                $this->roles()
                    ->with('permissions')
                    ->get()
                    ->pluck('permissions')
                    ->flatten()
            )
            ->unique('id');
    }

    public function hasPermission(string $permission): bool
    {
        return $this->getCachedPermissions()
            ->contains('name', $permission);
    }
}
```

### 2. Efficient Authorization Queries

**Optimized Policy Queries:**
```php
class PostPolicy
{
    public function viewAny(User $user): bool
    {
        // Use eager-loaded permissions
        return $user->relationLoaded('permissions') 
            ? $user->permissions->contains('name', 'posts.view')
            : $user->hasPermission('posts.view');
    }
}
```

## Next Steps

1. **Master [File Navigation](18-file-navigation.md)** for organizing auth-related files
2. **Learn [Testing Strategies](16-testing-strategies.md)** for comprehensive auth testing
3. **Practice [Performance Optimization](20-performance-optimization.md)** for auth system efficiency

Comprehensive authentication and authorization provide the foundation for secure Laravel applications, with Filament offering powerful tools for managing user access and permissions in admin interfaces.