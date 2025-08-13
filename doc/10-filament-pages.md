# [🔙](README.md) Custom Pages in Filament 4.x

Custom pages in Filament allow you to create admin interfaces that go beyond standard CRUD operations, enabling dashboards, reports, settings, and specialized workflows.

## Types of Custom Pages

### 1. Dashboard Pages

**Analytics Dashboard:**
```bash
php artisan make:filament-page Analytics
```

```php
class Analytics extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-chart-bar';
    protected static string $view = 'filament.pages.analytics';
    protected static ?string $navigationGroup = 'Reports';
    
    public function getViewData(): array
    {
        return [
            'totalPosts' => Post::count(),
            'publishedPosts' => Post::where('status', 'published')->count(),
            'totalUsers' => User::count(),
            'recentPosts' => Post::latest()->limit(5)->get(),
        ];
    }
    
    protected function getHeaderWidgets(): array
    {
        return [
            AnalyticsStatsWidget::class,
            PostsChartWidget::class,
        ];
    }
}
```

### 2. Settings Pages

**Application Settings:**
```bash
php artisan make:filament-page Settings
```

```php
class Settings extends Page implements HasForms
{
    use InteractsWithForms;
    
    protected static ?string $navigationIcon = 'heroicon-o-cog-6-tooth';
    protected static string $view = 'filament.pages.settings';
    
    public ?array $data = [];
    
    public function mount(): void
    {
        $this->form->fill([
            'site_name' => config('app.name'),
            'site_description' => config('app.description'),
            'posts_per_page' => config('app.posts_per_page', 10),
            'enable_comments' => config('app.enable_comments', true),
        ]);
    }
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                Section::make('Site Information')
                    ->schema([
                        TextInput::make('site_name')
                            ->required()
                            ->maxLength(255),
                        Textarea::make('site_description')
                            ->rows(3)
                            ->maxLength(500),
                    ]),
                    
                Section::make('Content Settings')
                    ->schema([
                        TextInput::make('posts_per_page')
                            ->numeric()
                            ->required()
                            ->minValue(1)
                            ->maxValue(100),
                        Toggle::make('enable_comments')
                            ->label('Allow comments on posts'),
                    ]),
            ])
            ->statePath('data');
    }
    
    public function save(): void
    {
        $data = $this->form->getState();
        
        // Save to config or database
        foreach ($data as $key => $value) {
            Setting::updateOrCreate(
                ['key' => $key],
                ['value' => $value]
            );
        }
        
        Notification::make()
            ->title('Settings saved successfully')
            ->success()
            ->send();
    }
    
    protected function getFormActions(): array
    {
        return [
            Action::make('save')
                ->label('Save Settings')
                ->submit('save'),
        ];
    }
}
```

### 3. Report Pages

**Content Report:**
```bash
php artisan make:filament-page ContentReport
```

```php
class ContentReport extends Page implements HasTable
{
    use InteractsWithTable;
    
    protected static ?string $navigationIcon = 'heroicon-o-document-chart-bar';
    protected static string $view = 'filament.pages.content-report';
    protected static ?string $navigationGroup = 'Reports';
    
    public function table(Table $table): Table
    {
        return $table
            ->query(
                Post::withCount(['comments', 'views'])
                    ->with('user')
                    ->orderBy('created_at', 'desc')
            )
            ->columns([
                TextColumn::make('title')
                    ->searchable()
                    ->sortable(),
                TextColumn::make('user.name')
                    ->label('Author')
                    ->sortable(),
                TextColumn::make('status')
                    ->badge(),
                TextColumn::make('comments_count')
                    ->label('Comments')
                    ->sortable(),
                TextColumn::make('views_count')
                    ->label('Views')
                    ->sortable(),
                TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable(),
            ])
            ->filters([
                SelectFilter::make('status'),
                DateFilter::make('created_at'),
            ])
            ->actions([
                Action::make('view')
                    ->url(fn (Post $record): string => PostResource::getUrl('view', ['record' => $record]))
                    ->openUrlInNewTab(),
            ]);
    }
}
```

## Interactive Pages with Forms

### 1. Data Import Page

**Bulk Import Interface:**
```php
class ImportPosts extends Page implements HasForms
{
    use InteractsWithForms;
    
    protected static string $view = 'filament.pages.import-posts';
    
    public ?array $data = [];
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                FileUpload::make('import_file')
                    ->label('CSV File')
                    ->acceptedFileTypes(['text/csv', 'application/csv'])
                    ->required()
                    ->directory('imports')
                    ->visibility('private'),
                    
                Checkbox::make('has_headers')
                    ->label('File has headers')
                    ->default(true),
                    
                Select::make('default_status')
                    ->label('Default Status for Imported Posts')
                    ->options([
                        'draft' => 'Draft',
                        'published' => 'Published',
                    ])
                    ->default('draft')
                    ->required(),
                    
                Select::make('default_author')
                    ->label('Default Author')
                    ->relationship('users', 'name')
                    ->searchable()
                    ->preload()
                    ->required(),
            ])
            ->statePath('data');
    }
    
    public function import(): void
    {
        $data = $this->form->getState();
        
        $filePath = storage_path('app/' . $data['import_file']);
        $csv = Reader::createFromPath($filePath, 'r');
        
        if ($data['has_headers']) {
            $csv->setHeaderOffset(0);
        }
        
        $imported = 0;
        foreach ($csv as $record) {
            Post::create([
                'title' => $record['title'] ?? 'Untitled',
                'slug' => Str::slug($record['title'] ?? 'untitled'),
                'content' => $record['content'] ?? '',
                'status' => $data['default_status'],
                'user_id' => $data['default_author'],
            ]);
            $imported++;
        }
        
        Notification::make()
            ->title("Imported {$imported} posts successfully")
            ->success()
            ->send();
            
        // Clear the form
        $this->form->fill();
    }
    
    protected function getFormActions(): array
    {
        return [
            Action::make('import')
                ->label('Import Posts')
                ->action('import')
                ->requiresConfirmation(),
        ];
    }
}
```

### 2. Bulk Operations Page

**Mass Update Interface:**
```php
class BulkUpdate extends Page implements HasForms, HasTable
{
    use InteractsWithForms, InteractsWithTable;
    
    protected static string $view = 'filament.pages.bulk-update';
    
    public ?array $data = [];
    
    public function table(Table $table): Table
    {
        return $table
            ->query(Post::query())
            ->columns([
                TextColumn::make('title'),
                TextColumn::make('status')->badge(),
                TextColumn::make('user.name'),
            ])
            ->bulkActions([
                BulkAction::make('updateSelected')
                    ->label('Update Selected')
                    ->form([
                        Select::make('status')
                            ->options([
                                'draft' => 'Draft',
                                'published' => 'Published',
                                'archived' => 'Archived',
                            ]),
                        Select::make('category_id')
                            ->relationship('categories', 'name')
                            ->searchable(),
                    ])
                    ->action(function (Collection $records, array $data) {
                        $updates = array_filter($data);
                        if (!empty($updates)) {
                            $records->each->update($updates);
                            
                            Notification::make()
                                ->title('Updated ' . $records->count() . ' posts')
                                ->success()
                                ->send();
                        }
                    }),
            ]);
    }
}
```

## Page Widgets and Components

### 1. Dashboard Widgets

**Quick Stats Widget:**
```bash
php artisan make:filament-widget QuickStats --type=stats-overview
```

```php
class QuickStats extends StatsOverviewWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Posts', Post::count())
                ->description('32k increase')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->chart([7, 2, 10, 3, 15, 4, 17])
                ->color('success'),
                
            Stat::make('Published Today', Post::whereDate('published_at', today())->count())
                ->description('7% increase')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->color('success'),
                
            Stat::make('Draft Posts', Post::where('status', 'draft')->count())
                ->description('3% decrease')
                ->descriptionIcon('heroicon-m-arrow-trending-down')
                ->color('danger'),
        ];
    }
}
```

### 2. Chart Widgets

**Posts Over Time Chart:**
```bash
php artisan make:filament-widget PostsChart --type=chart
```

```php
class PostsChart extends ChartWidget
{
    protected static ?string $heading = 'Posts Created Over Time';
    
    protected function getData(): array
    {
        $data = Post::selectRaw('DATE(created_at) as date, COUNT(*) as count')
            ->whereBetween('created_at', [now()->subDays(30), now()])
            ->groupBy('date')
            ->orderBy('date')
            ->pluck('count', 'date')
            ->toArray();
            
        return [
            'datasets' => [
                [
                    'label' => 'Posts created',
                    'data' => array_values($data),
                    'backgroundColor' => 'rgba(59, 130, 246, 0.1)',
                    'borderColor' => 'rgb(59, 130, 246)',
                ],
            ],
            'labels' => array_keys($data),
        ];
    }
    
    protected function getType(): string
    {
        return 'line';
    }
}
```

## Page Navigation and Organization

### 1. Navigation Groups

**Organize Related Pages:**
```php
// In each page class
protected static ?string $navigationGroup = 'Content Management';
protected static ?int $navigationSort = 10;
```

### 2. Navigation Icons and Labels

**Custom Navigation:**
```php
protected static ?string $navigationIcon = 'heroicon-o-chart-bar';
protected static ?string $navigationLabel = 'Analytics Dashboard';
protected static ?string $slug = 'analytics-dashboard';
```

### 3. Conditional Navigation

**Role-Based Navigation:**
```php
public static function shouldRegisterNavigation(): bool
{
    return auth()->user()->can('view-analytics');
}
```

## Page Actions and Header

### 1. Header Actions

**Page-Level Actions:**
```php
protected function getHeaderActions(): array
{
    return [
        Action::make('export')
            ->label('Export Data')
            ->icon('heroicon-o-arrow-down-tray')
            ->action(function () {
                return Excel::download(new PostsExport, 'posts.xlsx');
            }),
            
        Action::make('refresh')
            ->label('Refresh Data')
            ->icon('heroicon-o-arrow-path')
            ->action(function () {
                // Refresh page data
                $this->redirect(request()->header('Referer'));
            }),
    ];
}
```

### 2. Page Breadcrumbs

**Custom Breadcrumbs:**
```php
protected function getBreadcrumbs(): array
{
    return [
        url()->previous() => 'Back',
        '' => 'Analytics Dashboard',
    ];
}
```

## Testing Custom Pages

### 1. Page Accessibility

Test with Claude Code:
```
"Test if the Analytics page loads correctly"
"Verify the Settings form validation works"
"Check if the Import page handles file uploads properly"
```

### 2. Widget Functionality

Verify widgets display correctly:
```
"Test the dashboard widgets for accurate data"
"Check if the chart widgets render properly"
"Verify the stats widgets update in real-time"
```

### 3. Form Interactions

Test form behavior:
```
"Test the settings form save functionality"
"Verify the import form handles CSV files correctly"
"Check if bulk update operations work as expected"
```

## Best Practices

### 1. Page Organization

- Group related pages using navigation groups
- Use descriptive icons and labels
- Implement proper authorization checks

### 2. Performance

- Cache expensive queries in dashboard widgets
- Use pagination for large datasets
- Implement efficient data loading strategies

### 3. User Experience

- Provide clear feedback for user actions
- Use loading states for long operations
- Implement proper error handling

## Next Steps

1. **Learn [Widgets and Charts](11-filament-widgets.md)** for advanced dashboard components
2. **Master [Forms and Fields](12-filament-forms.md)** for complex form handling
3. **Practice [Authentication & Authorization](14-auth-authorization.md)** for page security

Custom pages in Filament provide unlimited flexibility for creating specialized admin interfaces that perfectly match your application's needs.