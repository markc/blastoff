# [🔙](README.md) Resource Creation with Filament 4.x

This guide covers advanced patterns for building and customizing Filament resources, going beyond the basics to create sophisticated admin interfaces with complex relationships and custom functionality.

## Advanced Resource Patterns

### 1. Complex Form Layouts

**Multi-Section Forms:**
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Basic Information')
                ->schema([
                    TextInput::make('title')->required(),
                    TextInput::make('slug')->unique(Post::class, 'slug'),
                    Select::make('category_id')
                        ->relationship('category', 'name')
                        ->createOptionForm([
                            TextInput::make('name')->required(),
                            TextInput::make('slug')->required(),
                        ])
                        ->required(),
                ])
                ->columns(2),
                
            Section::make('Content')
                ->schema([
                    RichEditor::make('content')
                        ->required()
                        ->toolbarButtons([
                            'bold', 'italic', 'link', 'bulletList', 'orderedList',
                        ]),
                    TagsInput::make('tags')
                        ->separator(',')
                        ->placeholder('Add tags...'),
                ])
                ->columnSpanFull(),
                
            Section::make('Publishing Options')
                ->schema([
                    Select::make('status')
                        ->options([
                            'draft' => 'Draft',
                            'review' => 'Under Review', 
                            'published' => 'Published',
                            'archived' => 'Archived',
                        ])
                        ->default('draft')
                        ->required(),
                        
                    Toggle::make('featured')
                        ->label('Featured Post'),
                        
                    DateTimePicker::make('published_at')
                        ->label('Publish Date')
                        ->timezone('UTC'),
                        
                    Select::make('user_id')
                        ->relationship('user', 'name')
                        ->searchable()
                        ->preload()
                        ->required(),
                ])
                ->columns(2),
        ]);
}
```

### 2. Advanced Table Configurations

**Enhanced Table with Custom Columns:**
```php
public static function table(Table $table): Table
{
    return $table
        ->columns([
            Stack::make([
                TextColumn::make('title')
                    ->weight(FontWeight::Bold)
                    ->searchable()
                    ->sortable(),
                TextColumn::make('user.name')
                    ->color('gray')
                    ->prefix('by '),
            ]),
            
            BadgeColumn::make('status')
                ->colors([
                    'gray' => 'draft',
                    'warning' => 'review',
                    'success' => 'published',
                    'danger' => 'archived',
                ])
                ->icons([
                    'heroicon-o-pencil' => 'draft',
                    'heroicon-o-clock' => 'review',
                    'heroicon-o-eye' => 'published',
                    'heroicon-o-archive-box' => 'archived',
                ]),
                
            IconColumn::make('featured')
                ->boolean()
                ->trueIcon('heroicon-o-star')
                ->falseIcon('heroicon-o-star')
                ->trueColor('warning')
                ->falseColor('gray'),
                
            TextColumn::make('category.name')
                ->badge()
                ->searchable(),
                
            TextColumn::make('published_at')
                ->dateTime()
                ->sortable()
                ->toggleable(),
                
            TextColumn::make('created_at')
                ->since()
                ->sortable()
                ->toggleable(isToggledHiddenByDefault: true),
        ])
        ->defaultSort('created_at', 'desc')
        ->filters([
            SelectFilter::make('status')
                ->multiple()
                ->options([
                    'draft' => 'Draft',
                    'review' => 'Under Review',
                    'published' => 'Published', 
                    'archived' => 'Archived',
                ]),
                
            SelectFilter::make('category')
                ->relationship('category', 'name')
                ->searchable()
                ->preload(),
                
            Filter::make('featured')
                ->query(fn (Builder $query): Builder => $query->where('featured', true))
                ->toggle(),
                
            Filter::make('published_this_week')
                ->query(fn (Builder $query): Builder => 
                    $query->whereBetween('published_at', [now()->startOfWeek(), now()->endOfWeek()])
                ),
        ])
        ->actions([
            ActionGroup::make([
                ViewAction::make(),
                EditAction::make(),
                Action::make('publish')
                    ->icon('heroicon-o-eye')
                    ->action(function (Post $record) {
                        $record->update([
                            'status' => 'published',
                            'published_at' => now(),
                        ]);
                    })
                    ->requiresConfirmation()
                    ->visible(fn (Post $record) => $record->status !== 'published'),
                DeleteAction::make(),
            ]),
        ])
        ->bulkActions([
            BulkActionGroup::make([
                BulkAction::make('publish')
                    ->icon('heroicon-o-eye')
                    ->action(function (Collection $records) {
                        $records->each(function (Post $record) {
                            $record->update([
                                'status' => 'published',
                                'published_at' => now(),
                            ]);
                        });
                    })
                    ->requiresConfirmation(),
                DeleteBulkAction::make(),
            ]),
        ]);
}
```

## Relationship Management

### 1. HasMany Relationships

**Post Comments Relation Manager:**
```php
// In PostResource.php
public static function getRelations(): array
{
    return [
        CommentsRelationManager::class,
    ];
}

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
                Textarea::make('content')
                    ->required()
                    ->rows(3),
                Select::make('status')
                    ->options([
                        'pending' => 'Pending',
                        'approved' => 'Approved',
                        'spam' => 'Spam',
                    ])
                    ->default('pending'),
            ]);
    }
    
    public function table(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('author_name'),
                TextColumn::make('content')->limit(50),
                BadgeColumn::make('status'),
                TextColumn::make('created_at')->dateTime(),
            ])
            ->filters([
                SelectFilter::make('status'),
            ])
            ->actions([
                EditAction::make(),
                DeleteAction::make(),
            ])
            ->bulkActions([
                DeleteBulkAction::make(),
            ]);
    }
}
```

### 2. BelongsToMany Relationships

**Post Tags Management:**
```php
// In form schema
CheckboxList::make('tags')
    ->relationship('tags', 'name')
    ->columns(3)
    ->gridDirection('row')
    ->bulkToggleable(),

// Or with Select for many options
Select::make('tags')
    ->relationship('tags', 'name')
    ->multiple()
    ->searchable()
    ->preload()
    ->createOptionForm([
        TextInput::make('name')->required(),
        ColorPicker::make('color'),
    ]),
```

## Custom Actions and Widgets

### 1. Custom Resource Actions

**Duplicate Post Action:**
```php
// In table actions
Action::make('duplicate')
    ->icon('heroicon-o-document-duplicate')
    ->action(function (Post $record) {
        $newPost = $record->replicate();
        $newPost->title = $record->title . ' (Copy)';
        $newPost->slug = $record->slug . '-copy-' . time();
        $newPost->status = 'draft';
        $newPost->published_at = null;
        $newPost->save();
        
        // Copy relationships
        $newPost->tags()->attach($record->tags->pluck('id'));
        
        Notification::make()
            ->title('Post duplicated successfully')
            ->success()
            ->send();
    }),
```

**Bulk Status Update:**
```php
// In bulk actions
BulkAction::make('updateStatus')
    ->label('Update Status')
    ->icon('heroicon-o-pencil-square')
    ->form([
        Select::make('status')
            ->options([
                'draft' => 'Draft',
                'published' => 'Published',
                'archived' => 'Archived',
            ])
            ->required(),
    ])
    ->action(function (Collection $records, array $data) {
        $records->each(function (Post $record) use ($data) {
            $record->update(['status' => $data['status']]);
        });
        
        Notification::make()
            ->title('Status updated for ' . $records->count() . ' posts')
            ->success()
            ->send();
    }),
```

### 2. Resource Widgets

**Create Post Stats Widget:**
```bash
php artisan make:filament-widget PostStatsWidget --resource=PostResource
```

```php
class PostStatsWidget extends BaseWidget
{
    protected static string $resource = PostResource::class;
    
    protected function getStats(): array
    {
        return [
            Stat::make('Total Posts', Post::count())
                ->description('All posts in the system')
                ->descriptionIcon('heroicon-m-document-text')
                ->color('success'),
                
            Stat::make('Published Posts', Post::where('status', 'published')->count())
                ->description('Currently published')
                ->descriptionIcon('heroicon-m-eye')
                ->color('success'),
                
            Stat::make('Draft Posts', Post::where('status', 'draft')->count())
                ->description('Awaiting publication')
                ->descriptionIcon('heroicon-m-pencil')
                ->color('warning'),
                
            Stat::make('This Week', Post::whereBetween('created_at', [now()->startOfWeek(), now()->endOfWeek()])->count())
                ->description('Posts created this week')
                ->descriptionIcon('heroicon-m-calendar')
                ->color('primary'),
        ];
    }
}
```

## Advanced Form Fields

### 1. File Upload Handling

**Image Upload with Preview:**
```php
FileUpload::make('featured_image')
    ->image()
    ->imageEditor()
    ->imageEditorAspectRatios([
        '16:9',
        '4:3',
        '1:1',
    ])
    ->directory('posts/featured')
    ->visibility('public')
    ->maxSize(2048)
    ->acceptedFileTypes(['image/jpeg', 'image/png', 'image/webp']),
```

**Multiple File Uploads:**
```php
FileUpload::make('attachments')
    ->multiple()
    ->directory('posts/attachments')
    ->maxFiles(5)
    ->maxSize(5120)
    ->downloadable()
    ->previewable(false),
```

### 2. Dynamic Form Fields

**Conditional Field Display:**
```php
Select::make('post_type')
    ->options([
        'article' => 'Article',
        'video' => 'Video',
        'gallery' => 'Gallery',
    ])
    ->live()
    ->required(),

TextInput::make('video_url')
    ->url()
    ->visible(fn (Get $get) => $get('post_type') === 'video'),

FileUpload::make('gallery_images')
    ->image()
    ->multiple()
    ->visible(fn (Get $get) => $get('post_type') === 'gallery'),
```

## Resource Customization

### 1. Custom Resource Pages

**Create Custom View Page:**
```bash
php artisan make:filament-page ViewPost --resource=PostResource --type=custom
```

```php
class ViewPost extends Page implements HasRecord
{
    use InteractsWithRecord;
    
    protected static string $resource = PostResource::class;
    protected static string $view = 'filament.resources.post-resource.pages.view-post';
    
    public function mount(int | string $record): void
    {
        $this->record = $this->resolveRecord($record);
    }
    
    protected function getHeaderActions(): array
    {
        return [
            EditAction::make(),
            DeleteAction::make(),
        ];
    }
}
```

### 2. Custom Resource List Page

**Enhanced List Page with Custom Header:**
```php
class ListPosts extends ListRecords
{
    protected static string $resource = PostResource::class;
    
    protected function getHeaderActions(): array
    {
        return [
            CreateAction::make(),
            Action::make('import')
                ->label('Import Posts')
                ->icon('heroicon-o-arrow-down-tray')
                ->url(route('admin.posts.import')),
        ];
    }
    
    protected function getHeaderWidgets(): array
    {
        return [
            PostStatsWidget::class,
        ];
    }
}
```

## Testing Resource Implementation

### 1. Basic Resource Testing

Test with Claude Code:
```
"Test the Post resource functionality"
"Verify the post form validation works correctly"
"Check if all table filters function properly"
```

### 2. Relationship Testing

Verify relationships work:
```
"Test the post-category relationship in Filament"
"Verify the comments relation manager displays correctly"
"Check if tag assignment works in the post form"
```

### 3. Performance Testing

Monitor resource performance:
```
"Check for N+1 queries in the Post resource table"
"Test the performance of bulk actions on large datasets"
"Verify image upload processing time"
```

## Best Practices

### 1. Resource Organization

- Keep form schemas readable with proper sectioning
- Use consistent naming conventions for fields and relationships
- Implement proper validation rules matching database constraints

### 2. User Experience

- Provide clear field labels and help text
- Use appropriate field types for data (Toggle vs Checkbox vs Select)
- Implement logical field ordering and grouping

### 3. Performance Considerations

- Use `preload()` judiciously on Select fields
- Implement proper eager loading for relationships
- Consider pagination and filtering for large datasets

## Next Steps

1. **Learn [Custom Pages](10-filament-pages.md)** for non-CRUD interfaces
2. **Master [Widgets and Charts](11-filament-widgets.md)** for dashboard components
3. **Practice [Forms and Fields](12-filament-forms.md)** for advanced form handling

Advanced Filament resources provide powerful admin interfaces that can handle complex business logic while maintaining an intuitive user experience.