# [🔙](README.md) Debugging Without IDE

This guide covers advanced debugging techniques for Laravel + Filament applications using only command-line tools, nano, and Laravel Boost integration.

## Command-Line Debugging Philosophy

### 1. Debugging Mindset

**Command-Line Advantages:**
- **Immediate feedback** through Laravel Boost
- **Direct application access** via Tinker integration
- **Real-time log monitoring** without switching tools
- **Database introspection** on-demand
- **Minimal context switching** between debugging tools

### 2. Debugging Workflow

**Systematic Approach:**
1. **Reproduce the issue** consistently
2. **Gather information** using Boost tools
3. **Form hypothesis** about the cause
4. **Test hypothesis** with targeted debugging
5. **Implement fix** and verify

## Laravel Boost Debugging Tools

### 1. Application State Inspection

**System Overview:**
```
Ask Claude Code: "What's the current state of the application?"
```

This provides:
- Package versions and compatibility
- Database connection status
- Configuration summary
- Recent errors or issues

**Configuration Debugging:**
```
Ask Claude Code: "Check the database configuration"
Ask Claude Code: "Show me the mail settings"
Ask Claude Code: "Verify the cache configuration"
```

### 2. Database Debugging

**Query Investigation:**
```
Ask Claude Code: "Show me the posts table structure"
Ask Claude Code: "Check for any orphaned relationships in posts"
Ask Claude Code: "Find posts with invalid user references"
```

**Data Integrity Checks:**
```
Ask Claude Code: "Verify all foreign key constraints are working"
Ask Claude Code: "Check for duplicate slugs in posts"
Ask Claude Code: "Show me records with null required fields"
```

### 3. Model and Relationship Debugging

**Relationship Testing:**
```
Ask Claude Code: "Test the User-Post relationship with actual data"
Ask Claude Code: "Verify the Post-Comments relationship works correctly"
Ask Claude Code: "Check if all models load their relationships properly"
```

**Factory and Seeder Debugging:**
```
Ask Claude Code: "Test the Post factory with different states"
Ask Claude Code: "Verify database seeders create valid data"
Ask Claude Code: "Check if factory relationships work correctly"
```

## Error Investigation Patterns

### 1. Application Error Debugging

**Error Context Gathering:**
```
Ask Claude Code: "What's the latest error in the application logs?"
Ask Claude Code: "Show me all errors from the last hour"
Ask Claude Code: "Find PHP fatal errors in recent logs"
```

**Stack Trace Analysis:**
```bash
# When errors occur, gather full context:
# 1. Check application logs via Boost
# 2. Examine the specific file mentioned in error
nano app/Models/Post.php +45  # Jump to specific line

# 3. Verify related files
nano app/Filament/Resources/PostResource.php

# 4. Test the specific functionality
# Ask Claude Code: "Test the failing method in isolation"
```

### 2. Database Error Debugging

**SQL Error Investigation:**
```
Ask Claude Code: "Show me recent database errors"
Ask Claude Code: "Check if the failing table exists"
Ask Claude Code: "Verify column names match the model"
```

**Migration Issues:**
```bash
# Check migration status
php artisan migrate:status

# Review specific migration
nano database/migrations/2024_01_01_000000_create_posts_table.php

# Test migration in isolation
php artisan migrate:rollback --step=1
php artisan migrate
```

### 3. Filament-Specific Debugging

**Resource Error Debugging:**
```
Ask Claude Code: "Check if the PostResource form validates correctly"
Ask Claude Code: "Test the PostResource table query"
Ask Claude Code: "Verify Filament resource permissions"
```

**Form Validation Issues:**
```bash
# Examine resource form definition
nano app/Filament/Resources/PostResource.php

# Focus on form() method - check for:
# - Correct field names
# - Validation rules
# - Relationship definitions

# Test with Claude Code:
# "Test form validation with invalid data"
```

## Performance Debugging

### 1. Query Performance Issues

**N+1 Query Detection:**
```
Ask Claude Code: "Check for N+1 queries in the Post resource"
Ask Claude Code: "Show me slow database queries from logs"
Ask Claude Code: "Test query performance for the posts listing"
```

**Optimization Testing:**
```bash
# Before optimization
# Ask Claude Code: "Time the post listing query performance"

# Edit model to add eager loading
nano app/Models/Post.php

# After optimization  
# Ask Claude Code: "Test the optimized query performance"
```

### 2. Memory Usage Debugging

**Memory Leak Detection:**
```
Ask Claude Code: "Check memory usage in application logs"
Ask Claude Code: "Test memory consumption for large datasets"
Ask Claude Code: "Monitor memory during bulk operations"
```

**Memory Optimization:**
```bash
# Check memory-intensive operations
nano app/Services/PostService.php

# Implement chunking for large datasets
# Ask Claude Code: "Test memory usage with chunked processing"
```

## Frontend Debugging

### 1. Browser-Side Issues

**JavaScript Error Monitoring:**
```
Ask Claude Code: "Check for JavaScript errors in browser logs"
Ask Claude Code: "Show me console errors from the admin panel"
Ask Claude Code: "Find network request failures"
```

**Asset Loading Issues:**
```bash
# Check asset compilation
npm run dev

# Verify asset files exist
ls -la public/build/

# Test asset loading
# Ask Claude Code: "Check if CSS and JS assets load correctly"
```

### 2. Livewire Component Debugging

**Component State Issues:**
```
Ask Claude Code: "Check for Livewire hydration errors"
Ask Claude Code: "Show me Livewire component errors in logs"
Ask Claude Code: "Test Livewire component functionality"
```

**Form Interaction Debugging:**
```bash
# Examine Livewire component
nano app/Livewire/PostForm.php

# Check for:
# - Property definitions
# - Validation rules  
# - Event handling

# Test component behavior:
# Ask Claude Code: "Test the PostForm component interaction"
```

## Advanced Debugging Techniques

### 1. Custom Debug Logging

**Strategic Logging Placement:**
```php
// Add to suspected problematic areas
\Log::debug('Post creation started', ['data' => $data]);
\Log::debug('Post saved successfully', ['post_id' => $post->id]);
\Log::error('Post creation failed', ['error' => $e->getMessage()]);
```

**Log Monitoring:**
```
Ask Claude Code: "Show me debug logs for post creation"
Ask Claude Code: "Filter logs to show only my debug messages"
```

### 2. Tinker-Based Debugging

**Interactive Problem Solving:**
```
Ask Claude Code: "Test the failing code in Tinker step by step"
```

Example debugging session:
```php
// Step 1: Test basic functionality
$user = User::first();
echo "User loaded: " . $user->name;

// Step 2: Test relationship
$posts = $user->posts;
echo "Posts count: " . $posts->count();

// Step 3: Test specific issue
$post = $user->posts()->create(['title' => 'Test']);
echo "Post created: " . $post->id;
```

### 3. Environment-Specific Debugging

**Development vs Production:**
```
Ask Claude Code: "Check which environment we're running in"
Ask Claude Code: "Verify debug mode is enabled for development"
Ask Claude Code: "Show me environment-specific configuration"
```

**Configuration Debugging:**
```bash
# Check environment file
nano .env

# Verify configuration loading
# Ask Claude Code: "Show me the loaded configuration values"
```

## Debugging Specific Scenarios

### 1. Authentication Issues

**User Authentication Debugging:**
```
Ask Claude Code: "Test user authentication in Filament"
Ask Claude Code: "Check if user sessions are working"
Ask Claude Code: "Verify password hashing is correct"
```

**Permission Debugging:**
```bash
# Check authorization logic
nano app/Policies/PostPolicy.php

# Test permissions:
# Ask Claude Code: "Test user permissions for post editing"
```

### 2. File Upload Issues

**Upload Debugging:**
```
Ask Claude Code: "Check file upload permissions"
Ask Claude Code: "Verify storage disk configuration"
Ask Claude Code: "Test file upload functionality"
```

**Storage Investigation:**
```bash
# Check storage directories
ls -la storage/app/public/

# Verify symbolic link
ls -la public/storage

# Test storage configuration
# Ask Claude Code: "Test file storage and retrieval"
```

### 3. Queue and Job Issues

**Job Debugging:**
```
Ask Claude Code: "Show me failed queue jobs"
Ask Claude Code: "Check queue worker status"
Ask Claude Code: "Test job processing manually"
```

**Queue Monitoring:**
```bash
# Check queue configuration
nano config/queue.php

# Monitor queue processing
php artisan queue:work --verbose

# Test specific job
# Ask Claude Code: "Execute a test job and monitor results"
```

## Debugging Tools Integration

### 1. Laravel Pail Integration

**Real-Time Log Monitoring:**
```bash
# Start Pail for log monitoring
php artisan pail --timeout=0

# In another terminal, trigger the issue
# Watch logs in real-time for debugging information
```

### 2. Artisan Commands for Debugging

**Built-in Debugging Commands:**
```bash
# Check application status
php artisan about

# Verify configuration
php artisan config:show

# Test database connection
php artisan db:show

# Check routes
php artisan route:list

# Verify queue configuration
php artisan queue:monitor
```

### 3. Custom Debug Commands

**Create Debugging Artisan Commands:**
```bash
php artisan make:command DebugPost
```

```php
class DebugPost extends Command
{
    protected $signature = 'debug:post {id}';
    protected $description = 'Debug specific post issues';
    
    public function handle()
    {
        $postId = $this->argument('id');
        $post = Post::with(['user', 'comments'])->find($postId);
        
        if (!$post) {
            $this->error("Post {$postId} not found");
            return;
        }
        
        $this->info("Post: {$post->title}");
        $this->info("Author: {$post->user->name}");
        $this->info("Comments: {$post->comments->count()}");
        
        // Test specific functionality
        $this->info("Status: {$post->status}");
        $this->info("Published: " . ($post->isPublished() ? 'Yes' : 'No'));
    }
}
```

## Debugging Best Practices

### 1. Systematic Approach

**Organized Investigation:**
1. **Document the issue** clearly
2. **Reproduce consistently** in development
3. **Isolate the problem** to specific components
4. **Test incrementally** with small changes
5. **Verify the fix** thoroughly

### 2. Information Gathering

**Comprehensive Context:**
- Application state via Boost tools
- Error logs and stack traces
- Database state and relationships
- Configuration values
- Environment variables

### 3. Incremental Testing

**Step-by-Step Verification:**
```bash
# Test each layer independently:
# 1. Database layer
# Ask Claude Code: "Test database queries directly"

# 2. Model layer  
# Ask Claude Code: "Test model methods in isolation"

# 3. Resource layer
# Test Filament resource components individually

# 4. Frontend layer
# Check browser logs and network requests
```

## Recovery and Prevention

### 1. Error Recovery

**Quick Fix Strategies:**
```bash
# Clear caches when configuration changes
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Reset database if data issues
php artisan migrate:fresh --seed

# Rebuild assets if frontend issues  
npm run build
```

### 2. Prevention Strategies

**Proactive Debugging:**
- Regular log monitoring during development
- Comprehensive testing before deployment
- Documentation of known issues and solutions
- Regular database integrity checks

## Next Steps

1. **Master [Performance Optimization](20-performance-optimization.md)** for debugging performance issues
2. **Learn [File Navigation](18-file-navigation.md)** for efficient debugging workflows
3. **Practice [Testing Strategies](16-testing-strategies.md)** to prevent debugging needs

Command-line debugging with Laravel Boost provides powerful, immediate feedback for identifying and resolving issues efficiently without the overhead of traditional debugging tools.