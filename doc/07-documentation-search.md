# [🔙](README.md) Documentation Search with Laravel Boost

Laravel Boost's documentation search tool provides version-specific documentation access for the Laravel ecosystem, ensuring you always get relevant information for your installed package versions.

## Documentation Search Tool

### Overview

The `search-docs` tool provides:
- **Version-specific results** based on your installed packages
- **Laravel ecosystem coverage** including Laravel, Filament, Livewire, Pest, etc.
- **Contextual search** with multiple related queries
- **Real-time package version detection**
- **Comprehensive documentation access**

### Basic Documentation Search

**General Framework Questions:**
```
Ask Claude Code: "How do I create middleware in Laravel 12?"
```

This searches Laravel 12-specific documentation for middleware creation patterns.

**Package-Specific Queries:**
```
Ask Claude Code: "How do I add file uploads to Filament resources?"
```

Searches Filament 4.x documentation for file upload implementation.

**Multi-Package Integration:**
```
Ask Claude Code: "How do I test Filament resources with Pest?"
```

Searches both Filament and Pest documentation for testing patterns.

## Advanced Search Strategies

### 1. Multiple Query Approach

For complex topics, use multiple related queries:
```
# Ask Claude Code with multiple angles:
"How do I implement user roles and permissions in Laravel with Filament?"

# This might search for:
# - "laravel authorization roles permissions"
# - "filament user management policies"
# - "spatie permission package laravel"
```

### 2. Package-Specific Searches

Focus on specific packages when needed:
```
# Ask Claude Code:
"Show me Livewire 3 component patterns for forms"
"How do I customize Tailwind CSS in Filament 4?"
"What are the new features in Laravel 12?"
```

### 3. Version-Aware Development

Always get current version information:
```
# Ask Claude Code:
"What's new in the Laravel framework version I'm using?"
"Show me Filament 4 migration guide from version 3"
"How has Pest changed in the latest version?"
```

## Integration with Development Workflow

### 1. Pre-Implementation Research

Before building features, search for best practices:
```
# Ask Claude Code:
"How should I structure a blog application with Filament 4?"
"What's the recommended way to handle file uploads in Laravel 12?"
"How do I implement real-time notifications with Livewire 3?"
```

### 2. Problem-Solving Workflow

When encountering issues:
```
# Ask Claude Code:
"How do I fix Laravel validation errors in Filament forms?"
"Why might Livewire components not be updating?"
"How do I debug queue job failures in Laravel?"
```

### 3. Optimization and Best Practices

Improve existing code:
```
# Ask Claude Code:
"How do I optimize Eloquent queries in Filament resources?"
"What are performance best practices for Laravel 12?"
"How do I implement caching in Filament applications?"
```

## Common Documentation Searches

### Laravel Framework

**Core Features:**
```
# Ask Claude Code:
"Laravel 12 routing best practices"
"How to use Laravel 12 model factories"
"Laravel validation rules and custom rules"
"Database migration patterns in Laravel 12"
```

**Advanced Topics:**
```
# Ask Claude Code:
"Laravel event sourcing implementation"
"How to build APIs with Laravel Sanctum"
"Laravel queue worker optimization"
"Custom Artisan commands in Laravel 12"
```

### Filament Administration

**Resource Development:**
```
# Ask Claude Code:
"Filament 4 resource relationship management"
"How to customize Filament table columns"
"Filament form builder advanced fields"
"Custom actions in Filament resources"
```

**UI Customization:**
```
# Ask Claude Code:
"How to customize Filament 4 theme"
"Filament navigation menu customization"
"Adding custom CSS to Filament panels"
"Filament widget development patterns"
```

### Testing and Quality

**Testing Frameworks:**
```
# Ask Claude Code:
"Pest testing best practices for Laravel"
"How to test Filament resources with Pest"
"Laravel feature testing patterns"
"Database testing with Pest and Laravel"
```

**Code Quality:**
```
# Ask Claude Code:
"Laravel Pint configuration options"
"Static analysis tools for Laravel projects"
"Code coverage setup with Pest"
"Laravel debugging techniques"
```

## Documentation Search Tips

### 1. Be Specific

Instead of vague queries, be specific:
- ❌ "How do forms work?"
- ✅ "How do I validate file uploads in Filament 4 forms?"

### 2. Include Package Context

Mention specific packages when relevant:
- ❌ "How do I test components?"
- ✅ "How do I test Livewire components with Pest?"

### 3. Version Awareness

Trust that Boost will find version-specific information:
- The tool automatically uses your installed package versions
- Results are filtered for compatibility with your setup
- No need to specify versions unless comparing different versions

### 4. Multiple Perspectives

Ask related questions for comprehensive understanding:
```
# Ask Claude Code:
"How do I implement user authentication in Filament?"
# Follow up with:
"How do I customize the Filament login page?"
"How do I add role-based access to Filament resources?"
```

## Troubleshooting with Documentation

### 1. Error Resolution

When you encounter errors:
```
# Ask Claude Code:
"How do I fix 'Class not found' errors in Laravel?"
"Debugging Filament form validation failures"
"Resolving Livewire component hydration issues"
```

### 2. Migration and Upgrades

When upgrading packages:
```
# Ask Claude Code:
"How do I migrate from Filament 3 to Filament 4?"
"Laravel 11 to Laravel 12 upgrade guide"
"Breaking changes in Livewire 3 from version 2"
```

### 3. Performance Issues

For optimization guidance:
```
# Ask Claude Code:
"Laravel application performance optimization"
"How to optimize Filament resource loading times"
"Database query optimization in Laravel applications"
```

## Integration with Other Boost Tools

### 1. Combined with Tinker

```
# Ask Claude Code:
"How do I test model relationships?" 
# Then test with Tinker:
"Test the User-Post relationship using the documented approach"
```

### 2. Combined with Database Queries

```
# Ask Claude Code:
"How should I structure database indexes for this use case?"
# Then verify with database queries:
"Show me the current indexes on the posts table"
```

### 3. Combined with Log Monitoring

```
# Ask Claude Code:
"How do I debug Laravel queue failures?"
# Then monitor logs:
"Show me recent queue-related errors"
```

## Next Steps

1. **Practice [Filament Resources](09-filament-resources.md)** using documentation-informed patterns
2. **Learn [Testing Strategies](16-testing-strategies.md)** with comprehensive test documentation
3. **Master [Performance Optimization](20-performance-optimization.md)** using Laravel best practices

Documentation search with Laravel Boost ensures you're always working with current, version-specific information, making your development more efficient and following current best practices.