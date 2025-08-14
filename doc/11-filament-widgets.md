# [🔙](README.md) Widgets and Charts in Filament 4.x

This guide covers creating dashboard widgets and data visualization components in Filament 4.x, enabling rich admin dashboards with real-time data insights.

## Types of Filament Widgets

### 1. Stats Overview Widgets

**Basic Stats Widget:**
```bash
php artisan make:filament-widget PostStatsWidget --type=stats-overview
```

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use App\Models\User;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class PostStatsWidget extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Posts', Post::count())
                ->description('All posts in the system')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->chart([7, 2, 10, 3, 15, 4, 17])
                ->color('success'),
                
            Stat::make('Published Posts', Post::where('status', 'published')->count())
                ->description('Currently live')
                ->descriptionIcon('heroicon-m-eye')
                ->color('success'),
                
            Stat::make('Draft Posts', Post::where('status', 'draft')->count())
                ->description('Awaiting publication')
                ->descriptionIcon('heroicon-m-pencil')
                ->color('warning'),
                
            Stat::make('Featured Posts', Post::where('featured', true)->count())
                ->description('Highlighted content')
                ->descriptionIcon('heroicon-m-star')
                ->color('primary'),
        ];
    }
}
```

### 2. Chart Widgets

**Line Chart Widget:**
```bash
php artisan make:filament-widget PostsPerDayChart --type=chart
```

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Widgets\ChartWidget;
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

class PostsPerDayChart extends ChartWidget
{
    protected static ?string $heading = 'Posts Created Per Day';
    
    protected function getData(): array
    {
        $data = Trend::model(Post::class)
            ->between(
                start: now()->subDays(30),
                end: now(),
            )
            ->perDay()
            ->count();
            
        return [
            'datasets' => [
                [
                    'label' => 'Posts created',
                    'data' => $data->map(fn (TrendValue $value) => $value->aggregate),
                    'backgroundColor' => 'rgba(59, 130, 246, 0.1)',
                    'borderColor' => 'rgb(59, 130, 246)',
                    'pointBackgroundColor' => 'rgb(59, 130, 246)',
                    'tension' => 0.4,
                ],
            ],
            'labels' => $data->map(fn (TrendValue $value) => $value->date),
        ];
    }
    
    protected function getType(): string
    {
        return 'line';
    }
}
```

**Bar Chart Widget:**
```php
class PostsByStatusChart extends ChartWidget
{
    protected static ?string $heading = 'Posts by Status';
    
    protected function getData(): array
    {
        $data = Post::selectRaw('status, COUNT(*) as count')
            ->groupBy('status')
            ->pluck('count', 'status')
            ->toArray();
            
        return [
            'datasets' => [
                [
                    'label' => 'Posts',
                    'data' => array_values($data),
                    'backgroundColor' => [
                        'rgba(234, 179, 8, 0.8)',   // Draft - Yellow
                        'rgba(34, 197, 94, 0.8)',   // Published - Green
                        'rgba(239, 68, 68, 0.8)',   // Archived - Red
                    ],
                ],
            ],
            'labels' => array_keys($data),
        ];
    }
    
    protected function getType(): string
    {
        return 'bar';
    }
}
```

### 3. Table Widgets

**Recent Posts Widget:**
```bash
php artisan make:filament-widget RecentPostsWidget --type=table
```

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Widgets\TableWidget as BaseWidget;

class RecentPostsWidget extends BaseWidget
{
    protected static ?string $heading = 'Recent Posts';
    
    public function table(Table $table): Table
    {
        return $table
            ->query(
                Post::query()
                    ->with('user')
                    ->latest()
                    ->limit(5)
            )
            ->columns([
                Tables\Columns\TextColumn::make('title')
                    ->limit(50)
                    ->searchable(),
                    
                Tables\Columns\TextColumn::make('user.name')
                    ->label('Author'),
                    
                Tables\Columns\BadgeColumn::make('status')
                    ->colors([
                        'warning' => 'draft',
                        'success' => 'published',
                        'danger' => 'archived',
                    ]),
                    
                Tables\Columns\TextColumn::make('created_at')
                    ->since()
                    ->sortable(),
            ])
            ->actions([
                Tables\Actions\Action::make('edit')
                    ->icon('heroicon-m-pencil-square')
                    ->url(fn (Post $record): string => 
                        \App\Filament\Resources\PostResource::getUrl('edit', ['record' => $record])
                    ),
            ]);
    }
}
```

## Advanced Widget Features

### 1. Interactive Widgets

**Filterable Stats Widget:**
```php
class FilterableStatsWidget extends BaseWidget
{
    public ?string $filter = 'all';
    
    protected function getFilters(): ?array
    {
        return [
            'all' => 'All Time',
            'today' => 'Today',
            'week' => 'This Week',
            'month' => 'This Month',
        ];
    }
    
    protected function getStats(): array
    {
        $query = Post::query();
        
        match ($this->filter) {
            'today' => $query->whereDate('created_at', today()),
            'week' => $query->whereBetween('created_at', [now()->startOfWeek(), now()->endOfWeek()]),
            'month' => $query->whereMonth('created_at', now()->month),
            default => $query,
        };
        
        return [
            Stat::make('Posts', $query->count())
                ->description('In selected period'),
                
            Stat::make('Published', $query->clone()->where('status', 'published')->count())
                ->description('Live posts'),
                
            Stat::make('Draft', $query->clone()->where('status', 'draft')->count())
                ->description('Unpublished'),
        ];
    }
}
```

### 2. Real-Time Widgets

**Polling Widget:**
```php
class LiveStatsWidget extends BaseWidget
{
    protected static ?string $pollingInterval = '10s';
    
    protected function getStats(): array
    {
        return [
            Stat::make('Online Users', $this->getOnlineUsersCount())
                ->description('Active in last 5 minutes')
                ->color('success'),
                
            Stat::make('Today\'s Posts', Post::whereDate('created_at', today())->count())
                ->description('Posts created today')
                ->color('primary'),
        ];
    }
    
    private function getOnlineUsersCount(): int
    {
        // Implementation depends on your session tracking
        return cache()->remember('online_users_count', 60, function () {
            return rand(10, 50); // Mock data
        });
    }
}
```

### 3. Custom Widgets

**Custom Calendar Widget:**
```bash
php artisan make:filament-widget PostCalendarWidget
```

```php
class PostCalendarWidget extends BaseWidget
{
    protected static string $view = 'filament.widgets.post-calendar-widget';
    
    public function getViewData(): array
    {
        $posts = Post::whereMonth('published_at', now()->month)
            ->whereYear('published_at', now()->year)
            ->with('user')
            ->get()
            ->groupBy(function ($post) {
                return $post->published_at->format('Y-m-d');
            });
            
        return [
            'posts' => $posts,
            'currentMonth' => now()->format('F Y'),
        ];
    }
}
```

**Widget Blade Template:**
```blade
{{-- resources/views/filament/widgets/post-calendar-widget.blade.php --}}
<x-filament-widgets::widget>
    <x-filament::section>
        <x-slot name="heading">
            Post Calendar - {{ $currentMonth }}
        </x-slot>
        
        <div class="grid grid-cols-7 gap-2">
            @for ($day = 1; $day <= now()->daysInMonth; $day++)
                @php
                    $date = now()->startOfMonth()->addDays($day - 1)->format('Y-m-d');
                    $dayPosts = $posts->get($date, collect());
                @endphp
                
                <div class="p-2 border rounded {{ $dayPosts->count() ? 'bg-blue-50' : 'bg-gray-50' }}">
                    <div class="font-semibold">{{ $day }}</div>
                    @if ($dayPosts->count())
                        <div class="text-xs text-blue-600">
                            {{ $dayPosts->count() }} posts
                        </div>
                    @endif
                </div>
            @endfor
        </div>
    </x-filament::section>
</x-filament-widgets::widget>
```

## Widget Layout and Organization

### 1. Dashboard Organization

**Admin Panel Provider Configuration:**
```php
// app/Providers/Filament/AdminPanelProvider.php
public function panel(Panel $panel): Panel
{
    return $panel
        ->default()
        ->id('admin')
        ->path('/admin')
        ->widgets([
            PostStatsWidget::class,
            PostsPerDayChart::class,
            PostsByStatusChart::class,
            RecentPostsWidget::class,
        ]);
}
```

### 2. Widget Positioning

**Control Widget Order:**
```php
class PostStatsWidget extends BaseWidget
{
    protected static ?int $sort = 1;
}

class PostsPerDayChart extends ChartWidget
{
    protected static ?int $sort = 2;
}

class RecentPostsWidget extends TableWidget
{
    protected static ?int $sort = 3;
}
```

### 3. Conditional Widget Display

**Permission-Based Widgets:**
```php
class AdminOnlyWidget extends BaseWidget
{
    public static function canView(): bool
    {
        return auth()->user()->hasRole('admin');
    }
}
```

**Resource-Specific Widgets:**
```php
// In PostResource.php
public static function getWidgets(): array
{
    return [
        PostStatsWidget::class,
        PostsPerDayChart::class,
    ];
}

// In resource pages
class ListPosts extends ListRecords
{
    protected function getHeaderWidgets(): array
    {
        return PostResource::getWidgets();
    }
}
```

## Data Visualization Patterns

### 1. Trend Analysis

**Growth Metrics:**
```php
class GrowthWidget extends ChartWidget
{
    protected function getData(): array
    {
        $currentMonth = Post::whereMonth('created_at', now()->month)->count();
        $lastMonth = Post::whereMonth('created_at', now()->subMonth()->month)->count();
        
        $growth = $lastMonth > 0 ? (($currentMonth - $lastMonth) / $lastMonth) * 100 : 0;
        
        return [
            'datasets' => [
                [
                    'label' => 'Monthly Posts',
                    'data' => [$lastMonth, $currentMonth],
                    'backgroundColor' => $growth >= 0 ? 'rgba(34, 197, 94, 0.8)' : 'rgba(239, 68, 68, 0.8)',
                ],
            ],
            'labels' => ['Last Month', 'This Month'],
        ];
    }
}
```

### 2. Comparative Analytics

**Multi-Dataset Charts:**
```php
class ComparativeChart extends ChartWidget
{
    protected function getData(): array
    {
        $posts = Trend::model(Post::class)
            ->between(start: now()->subDays(30), end: now())
            ->perDay()
            ->count();
            
        $users = Trend::model(User::class)
            ->between(start: now()->subDays(30), end: now())
            ->perDay()
            ->count();
            
        return [
            'datasets' => [
                [
                    'label' => 'Posts',
                    'data' => $posts->map(fn (TrendValue $value) => $value->aggregate),
                    'borderColor' => 'rgb(59, 130, 246)',
                ],
                [
                    'label' => 'Users',
                    'data' => $users->map(fn (TrendValue $value) => $value->aggregate),
                    'borderColor' => 'rgb(34, 197, 94)',
                ],
            ],
            'labels' => $posts->map(fn (TrendValue $value) => $value->date),
        ];
    }
}
```

## Testing Widgets

### 1. Widget Data Testing

**Test with Claude Code:**
```
"Test the PostStatsWidget data accuracy"
"Verify the PostsPerDayChart displays correctly"
"Check if the RecentPostsWidget shows latest posts"
```

### 2. Widget Performance Testing

**Monitor Widget Performance:**
```
"Check widget loading times on dashboard"
"Test widget performance with large datasets"
"Monitor memory usage of chart widgets"
```

### 3. Widget Integration Testing

**Full Dashboard Testing:**
```
"Test complete dashboard with all widgets"
"Verify widget permissions work correctly"
"Check widget responsiveness on different screen sizes"
```

## Widget Optimization

### 1. Caching Strategies

**Cache Expensive Queries:**
```php
class OptimizedStatsWidget extends BaseWidget
{
    protected function getStats(): array
    {
        $cacheKey = 'post_stats_' . auth()->id();
        
        return cache()->remember($cacheKey, 300, function () {
            return [
                Stat::make('Total Posts', Post::count()),
                Stat::make('Your Posts', auth()->user()->posts()->count()),
                Stat::make('Published', auth()->user()->posts()->published()->count()),
            ];
        });
    }
}
```

### 2. Lazy Loading

**Load Widgets On-Demand:**
```php
class LazyWidget extends BaseWidget
{
    protected static bool $isLazy = true;
    
    protected function getStats(): array
    {
        // This will only load when the widget comes into view
        return [
            Stat::make('Heavy Calculation', $this->performExpensiveCalculation()),
        ];
    }
}
```

### 3. Database Optimization

**Efficient Queries:**
```php
class EfficientWidget extends BaseWidget
{
    protected function getStats(): array
    {
        // Single query with aggregates
        $stats = Post::selectRaw('
            COUNT(*) as total,
            COUNT(CASE WHEN status = "published" THEN 1 END) as published,
            COUNT(CASE WHEN status = "draft" THEN 1 END) as draft,
            COUNT(CASE WHEN featured = 1 THEN 1 END) as featured
        ')->first();
        
        return [
            Stat::make('Total Posts', $stats->total),
            Stat::make('Published', $stats->published),
            Stat::make('Draft', $stats->draft),
            Stat::make('Featured', $stats->featured),
        ];
    }
}
```

## Best Practices

### 1. Widget Design

- Keep widgets focused on single metrics
- Use appropriate chart types for data
- Provide meaningful descriptions and context
- Implement proper error handling

### 2. Performance

- Cache expensive calculations
- Use database aggregates instead of collection methods
- Implement lazy loading for heavy widgets
- Monitor widget loading times

### 3. User Experience

- Group related widgets logically
- Use consistent color schemes
- Provide filtering options where appropriate
- Ensure widgets are responsive

## Next Steps

1. **Master [Forms and Fields](12-filament-forms.md)** for advanced form handling
2. **Learn [Model Relationships](13-model-relationships.md)** for complex data relationships
3. **Practice [Performance Optimization](20-performance-optimization.md)** for widget optimization

Filament widgets provide powerful data visualization capabilities that transform raw data into actionable insights for admin users, enhancing the overall admin panel experience.