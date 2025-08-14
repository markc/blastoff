# [🔙](README.md) Forms and Fields in Filament 4.x

This guide covers advanced form building and validation in Filament 4.x, including complex field types, dynamic forms, and custom validation patterns.

## Form Builder Fundamentals

### 1. Form Schema Structure

**Basic Form Structure:**
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Section::make('Post Details')
                ->schema([
                    TextInput::make('title')
                        ->required()
                        ->maxLength(255)
                        ->live(onBlur: true)
                        ->afterStateUpdated(fn (string $operation, $state, callable $set) => 
                            $operation === 'create' ? $set('slug', Str::slug($state)) : null
                        ),
                        
                    TextInput::make('slug')
                        ->required()
                        ->unique(Post::class, 'slug', ignoreRecord: true)
                        ->alphaDash(),
                ])
                ->columns(2),
                
            Section::make('Content')
                ->schema([
                    RichEditor::make('content')
                        ->required()
                        ->toolbarButtons([
                            'attachFiles',
                            'blockquote',
                            'bold',
                            'bulletList',
                            'codeBlock',
                            'h2',
                            'h3',
                            'italic',
                            'link',
                            'orderedList',
                            'redo',
                            'strike',
                            'undo',
                        ])
                        ->fileAttachmentsDisk('public')
                        ->fileAttachmentsDirectory('attachments'),
                ])
                ->columnSpanFull(),
        ]);
}
```

### 2. Field Layout and Organization

**Advanced Layout Patterns:**
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Tabs::make('Tabs')
                ->tabs([
                    Tabs\Tab::make('Basic Info')
                        ->schema([
                            Grid::make(2)
                                ->schema([
                                    TextInput::make('title')->required(),
                                    TextInput::make('slug')->required(),
                                ]),
                                
                            RichEditor::make('content')->required(),
                        ]),
                        
                    Tabs\Tab::make('SEO')
                        ->schema([
                            TextInput::make('meta_title')->maxLength(60),
                            Textarea::make('meta_description')
                                ->rows(3)
                                ->maxLength(160),
                            TagsInput::make('keywords')
                                ->separator(','),
                        ]),
                        
                    Tabs\Tab::make('Media')
                        ->schema([
                            FileUpload::make('featured_image')
                                ->image()
                                ->imageEditor()
                                ->directory('posts/featured'),
                                
                            Repeater::make('gallery')
                                ->schema([
                                    FileUpload::make('image')
                                        ->image()
                                        ->required(),
                                    TextInput::make('caption'),
                                    TextInput::make('alt_text'),
                                ])
                                ->columns(3),
                        ]),
                ]),
        ]);
}
```

## Advanced Field Types

### 1. Dynamic and Conditional Fields

**Conditional Field Display:**
```php
Select::make('post_type')
    ->options([
        'article' => 'Article',
        'video' => 'Video',
        'gallery' => 'Gallery',
        'podcast' => 'Podcast',
    ])
    ->live()
    ->required(),

// Video-specific fields
TextInput::make('video_url')
    ->url()
    ->required()
    ->visible(fn (Get $get) => $get('post_type') === 'video'),

TextInput::make('video_duration')
    ->numeric()
    ->suffix('minutes')
    ->visible(fn (Get $get) => $get('post_type') === 'video'),

// Gallery-specific fields
Repeater::make('gallery_images')
    ->schema([
        FileUpload::make('image')->image()->required(),
        TextInput::make('caption'),
    ])
    ->visible(fn (Get $get) => $get('post_type') === 'gallery')
    ->minItems(1),

// Podcast-specific fields
FileUpload::make('audio_file')
    ->acceptedFileTypes(['audio/mpeg', 'audio/wav'])
    ->visible(fn (Get $get) => $get('post_type') === 'podcast'),
```

### 2. Complex Relationship Fields

**Advanced Relationship Management:**
```php
Select::make('category_id')
    ->relationship('category', 'name')
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
    ])
    ->required(),

CheckboxList::make('tags')
    ->relationship('tags', 'name')
    ->searchable()
    ->createOptionForm([
        TextInput::make('name')->required(),
        ColorPicker::make('color'),
    ])
    ->columns(3)
    ->gridDirection('row'),

Select::make('author_id')
    ->relationship('author', 'name')
    ->getOptionLabelFromRecordUsing(fn (User $record) => "{$record->name} ({$record->email})")
    ->searchable(['name', 'email'])
    ->preload(),
```

### 3. Custom Field Components

**Repeater with Complex Fields:**
```php
Repeater::make('sections')
    ->schema([
        Select::make('type')
            ->options([
                'text' => 'Text Block',
                'image' => 'Image Block',
                'quote' => 'Quote Block',
                'code' => 'Code Block',
            ])
            ->live()
            ->required(),
            
        TextInput::make('title')
            ->visible(fn (Get $get) => in_array($get('type'), ['text', 'quote'])),
            
        RichEditor::make('content')
            ->visible(fn (Get $get) => $get('type') === 'text'),
            
        FileUpload::make('image')
            ->image()
            ->visible(fn (Get $get) => $get('type') === 'image'),
            
        Textarea::make('quote_text')
            ->visible(fn (Get $get) => $get('type') === 'quote'),
            
        TextInput::make('quote_author')
            ->visible(fn (Get $get) => $get('type') === 'quote'),
            
        Textarea::make('code')
            ->rows(10)
            ->visible(fn (Get $get) => $get('type') === 'code'),
            
        Select::make('language')
            ->options([
                'php' => 'PHP',
                'javascript' => 'JavaScript',
                'python' => 'Python',
                'sql' => 'SQL',
            ])
            ->visible(fn (Get $get) => $get('type') === 'code'),
    ])
    ->columns(2)
    ->collapsible()
    ->itemLabel(fn (array $state): ?string => $state['title'] ?? $state['type'] ?? null)
    ->addActionLabel('Add Section')
    ->reorderable(),
```

### 4. File Upload Management

**Advanced File Uploads:**
```php
FileUpload::make('featured_image')
    ->image()
    ->imageEditor()
    ->imageEditorAspectRatios([
        '16:9',
        '4:3',
        '1:1',
    ])
    ->imageResizeMode('cover')
    ->imageCropAspectRatio('16:9')
    ->imageResizeTargetWidth('1920')
    ->imageResizeTargetHeight('1080')
    ->directory('posts/featured')
    ->visibility('public')
    ->downloadable()
    ->openable()
    ->acceptedFileTypes(['image/jpeg', 'image/png', 'image/webp'])
    ->maxSize(2048)
    ->rules(['dimensions:min_width=800,min_height=600']),

FileUpload::make('attachments')
    ->multiple()
    ->directory('posts/attachments')
    ->acceptedFileTypes(['application/pdf', 'application/msword', 'text/plain'])
    ->maxFiles(5)
    ->maxSize(5120)
    ->downloadable()
    ->previewable(false)
    ->reorderable()
    ->appendFiles(),
```

## Form Validation

### 1. Built-in Validation Rules

**Comprehensive Validation:**
```php
TextInput::make('title')
    ->required()
    ->minLength(10)
    ->maxLength(255)
    ->rules(['string', 'regex:/^[a-zA-Z0-9\s\-_]+$/'])
    ->validationMessages([
        'regex' => 'Title can only contain letters, numbers, spaces, hyphens, and underscores.',
    ]),

TextInput::make('slug')
    ->required()
    ->unique(Post::class, 'slug', ignoreRecord: true)
    ->alphaDash()
    ->minLength(3)
    ->maxLength(100),

TextInput::make('email')
    ->email()
    ->unique(User::class, 'email', ignoreRecord: true)
    ->endsWith(['@company.com', '@partner.com'])
    ->rules(['confirmed']),

TextInput::make('email_confirmation')
    ->email()
    ->same('email')
    ->label('Confirm Email'),

DateTimePicker::make('published_at')
    ->after('today')
    ->before('2025-12-31')
    ->timezone('UTC'),

TextInput::make('price')
    ->numeric()
    ->minValue(0)
    ->maxValue(999999.99)
    ->step(0.01)
    ->prefix('$'),
```

### 2. Custom Validation Rules

**Custom Validation Logic:**
```php
TextInput::make('slug')
    ->required()
    ->rules([
        function () {
            return function (string $attribute, $value, Closure $fail) {
                if (Post::where('slug', $value)->exists()) {
                    $fail('This slug is already taken.');
                }
                
                if (in_array($value, ['admin', 'api', 'www'])) {
                    $fail('This slug is reserved.');
                }
                
                if (!preg_match('/^[a-z0-9-]+$/', $value)) {
                    $fail('Slug must only contain lowercase letters, numbers, and hyphens.');
                }
            };
        },
    ]),

Select::make('category_id')
    ->relationship('category', 'name')
    ->rules([
        function () {
            return function (string $attribute, $value, Closure $fail) {
                $category = Category::find($value);
                if ($category && !$category->is_active) {
                    $fail('Selected category is not active.');
                }
            };
        },
    ]),
```

### 3. Cross-Field Validation

**Dependent Field Validation:**
```php
DateTimePicker::make('event_start')
    ->required()
    ->live()
    ->afterStateUpdated(function (callable $set, $state) {
        $set('event_end', null); // Clear end date when start changes
    }),

DateTimePicker::make('event_end')
    ->required()
    ->rules([
        function (Get $get) {
            return function (string $attribute, $value, Closure $fail) use ($get) {
                $startDate = $get('event_start');
                if ($startDate && $value <= $startDate) {
                    $fail('End date must be after start date.');
                }
            };
        },
    ]),

TextInput::make('discount_percentage')
    ->numeric()
    ->minValue(0)
    ->maxValue(100)
    ->visible(fn (Get $get) => $get('has_discount')),

TextInput::make('discount_amount')
    ->numeric()
    ->minValue(0)
    ->visible(fn (Get $get) => $get('has_discount'))
    ->rules([
        function (Get $get) {
            return function (string $attribute, $value, Closure $fail) use ($get) {
                $price = $get('price');
                if ($price && $value >= $price) {
                    $fail('Discount amount must be less than the price.');
                }
            };
        },
    ]),
```

## Form State Management

### 1. Live Updates and Reactivity

**Reactive Form Behavior:**
```php
TextInput::make('quantity')
    ->numeric()
    ->live(debounce: 500)
    ->afterStateUpdated(function (callable $set, callable $get, $state) {
        $price = $get('unit_price') ?? 0;
        $total = $price * ($state ?? 0);
        $set('total_price', number_format($total, 2));
    }),

TextInput::make('unit_price')
    ->numeric()
    ->prefix('$')
    ->live(debounce: 500)
    ->afterStateUpdated(function (callable $set, callable $get, $state) {
        $quantity = $get('quantity') ?? 0;
        $total = ($state ?? 0) * $quantity;
        $set('total_price', number_format($total, 2));
    }),

TextInput::make('total_price')
    ->prefix('$')
    ->disabled()
    ->dehydrated(false),
```

### 2. Form Actions and Buttons

**Custom Form Actions:**
```php
protected function getFormActions(): array
{
    return [
        Action::make('save')
            ->label('Save Post')
            ->action('save')
            ->keyBindings(['mod+s']),
            
        Action::make('saveAndPublish')
            ->label('Save & Publish')
            ->action('saveAndPublish')
            ->color('success')
            ->requiresConfirmation()
            ->modalHeading('Publish Post')
            ->modalDescription('Are you sure you want to publish this post?'),
            
        Action::make('preview')
            ->label('Preview')
            ->url(fn (Post $record) => route('posts.preview', $record))
            ->openUrlInNewTab()
            ->color('gray'),
            
        Action::make('cancel')
            ->label('Cancel')
            ->url(PostResource::getUrl('index'))
            ->color('gray'),
    ];
}

public function saveAndPublish(): void
{
    $data = $this->form->getState();
    $data['status'] = 'published';
    $data['published_at'] = now();
    
    $this->record->update($data);
    
    Notification::make()
        ->title('Post published successfully')
        ->success()
        ->send();
        
    $this->redirect(PostResource::getUrl('index'));
}
```

### 3. Form Wizards

**Multi-Step Forms:**
```php
public static function form(Form $form): Form
{
    return $form
        ->schema([
            Wizard::make([
                Wizard\Step::make('Basic Information')
                    ->schema([
                        TextInput::make('title')->required(),
                        TextInput::make('slug')->required(),
                        Textarea::make('excerpt'),
                    ]),
                    
                Wizard\Step::make('Content')
                    ->schema([
                        RichEditor::make('content')->required(),
                        TagsInput::make('keywords'),
                    ]),
                    
                Wizard\Step::make('Media')
                    ->schema([
                        FileUpload::make('featured_image')->image(),
                        Repeater::make('gallery')
                            ->schema([
                                FileUpload::make('image')->image(),
                                TextInput::make('caption'),
                            ]),
                    ]),
                    
                Wizard\Step::make('Publishing')
                    ->schema([
                        Select::make('status')->options([
                            'draft' => 'Draft',
                            'published' => 'Published',
                        ]),
                        DateTimePicker::make('published_at'),
                        Select::make('category_id')
                            ->relationship('category', 'name'),
                    ]),
            ])
            ->submitAction(new HtmlString(Blade::render(<<<BLADE
                <x-filament::button
                    type="submit"
                    size="sm"
                >
                    Create Post
                </x-filament::button>
            BLADE))),
        ]);
}
```

## Form Testing and Debugging

### 1. Form Validation Testing

**Test with Claude Code:**
```
"Test the post form validation with invalid data"
"Verify the form handles file uploads correctly"
"Check if conditional fields show/hide properly"
```

### 2. Form State Testing

**State Management Testing:**
```
"Test the reactive form behavior with live updates"
"Verify form actions work correctly"
"Check if form wizard progresses properly"
```

### 3. Performance Testing

**Form Performance:**
```
"Test form loading time with large datasets"
"Check form responsiveness with many fields"
"Monitor memory usage during file uploads"
```

## Best Practices

### 1. Form Design

**User Experience:**
- Group related fields logically
- Use appropriate field types for data
- Provide clear labels and help text
- Implement progressive disclosure for complex forms

### 2. Validation

**Robust Validation:**
- Validate on both client and server side
- Provide clear error messages
- Use appropriate validation rules
- Implement cross-field validation where needed

### 3. Performance

**Optimization:**
- Use lazy loading for expensive options
- Implement debouncing for live updates
- Cache relationship options
- Optimize file upload handling

### 4. Accessibility

**Inclusive Design:**
- Use semantic form elements
- Provide appropriate ARIA labels
- Ensure keyboard navigation works
- Test with screen readers

## Next Steps

1. **Learn [Model Relationships](13-model-relationships.md)** for complex data relationships
2. **Master [Authentication & Authorization](14-auth-authorization.md)** for form security
3. **Practice [Performance Optimization](20-performance-optimization.md)** for form optimization

Advanced Filament forms provide powerful, flexible interfaces for data entry and management, enabling complex workflows while maintaining excellent user experience.