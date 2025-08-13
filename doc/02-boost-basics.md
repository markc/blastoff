# 🔙 [Back to Documentation](README.md) | Laravel Boost Basics

Laravel Boost is an MCP (Model Context Protocol) server that provides powerful development tools specifically designed for Laravel applications. This guide covers the essential Boost features and how to use them effectively with Claude Code.

## What is Laravel Boost?

Laravel Boost is an MCP server that enhances Laravel development by providing:
- Direct database access and querying
- Interactive PHP execution (Tinker integration)
- Real-time log monitoring and analysis
- Version-specific documentation search
- Application introspection and debugging tools

## MCP Server Integration

### Understanding MCP with Claude Code

MCP (Model Context Protocol) allows Claude Code to directly interact with your Laravel application through specialized tools. Laravel Boost provides these tools natively - no manual configuration required.

### Available Boost Tools

When using Claude Code with Laravel Boost, you have access to:

1. **Application Info** - Get comprehensive project details
2. **Database Operations** - Query and inspect database
3. **Tinker Integration** - Execute PHP code interactively
4. **Log Monitoring** - Real-time log analysis
5. **Documentation Search** - Version-specific Laravel ecosystem docs
6. **Configuration Access** - Read application configuration
7. **Route Inspection** - List and analyze application routes

## Core Boost Tools

### 1. Application Information

**Purpose**: Get comprehensive overview of your Laravel application

**Example Usage with Claude Code**:
```
Tell me about this Laravel application
```

Claude Code will use the `application-info` tool to show:
- PHP version (8.4.1)
- Laravel version (12.24.0)
- Database engine (SQLite)
- Installed packages with versions
- Available Eloquent models

### 2. Database Query Tool

**Purpose**: Execute read-only SQL queries directly

**Example Usage**:
```
Show me all tables in the database
```

Claude Code executes:
```sql
SELECT name FROM sqlite_master WHERE type='table';
```

**Advanced Examples**:
```
# Count users
SELECT COUNT(*) FROM users;

# Show recent migrations
SELECT * FROM migrations ORDER BY batch DESC LIMIT 5;

# Analyze table structure
PRAGMA table_info(users);
```

### 3. Tinker Integration

**Purpose**: Execute PHP code in Laravel application context

**Example Usage**:
```
Test if the User model works correctly
```

Claude Code executes:
```php
$userCount = App\Models\User::count();
echo "Total users: $userCount";

// Create test user
$user = App\Models\User::factory()->make();
echo "Generated user: " . $user->name;
```

**Advanced Examples**:
```php
// Test model relationships
$user = User::with('posts')->first();
echo $user->posts->count() . " posts found";

// Check configuration
echo config('app.name');

// Test service container
$service = app('some.service');
var_dump($service);
```

### 4. Documentation Search

**Purpose**: Search version-specific Laravel ecosystem documentation

**Example Usage**:
```
How do I create a Filament resource with relationships?
```

Claude Code searches for:
- `['filament resources', 'relationships', 'hasMany']`

Returns documentation specific to:
- Laravel 12.24.0
- Filament 4.0.0
- Other installed package versions

### 5. Log Monitoring

**Purpose**: Read application logs in real-time

**Example Usage**:
```
Show me the latest application errors
```

Claude Code uses `read-log-entries` to show recent log entries with proper PSR-3 formatting.

**Browser Log Monitoring**:
```
Check for JavaScript errors in the browser
```

Claude Code uses `browser-logs` to show frontend errors and console output.

## Boost-Enhanced Development Workflow

### 1. Project Analysis Phase

Start every development session by understanding your application:

```
# With Claude Code, ask:
"What's the current state of this Laravel application?"
```

Boost provides:
- Package versions and compatibility
- Database schema overview  
- Available models and relationships
- Current configuration state

### 2. Feature Planning Phase

Before implementing features, search for best practices:

```
# Example: Planning a blog feature
"How should I implement a blog with categories using Filament 4.x?"
```

Boost searches version-specific documentation for:
- Laravel 12.x model patterns
- Filament 4.x resource creation
- Relationship best practices

### 3. Development Phase

Use Boost tools during active development:

**Database Schema Design**:
```
"Show me the current database schema"
```

**Model Testing**:
```
"Test the User model factory and relationships"
```

**Configuration Verification**:
```
"Check if mail configuration is correct"
```

### 4. Debugging Phase

Boost excels at debugging without leaving the command line:

**Error Investigation**:
```
"What's the latest error in the application logs?"
```

**Database State Checking**:
```
"Check if the migration ran correctly"
```

**Model State Verification**:
```
"Test if the relationship between User and Post works"
```

## Best Practices with Boost

### 1. Start with Application Info

Always begin development sessions by asking Claude Code about the application state. This ensures you're working with current package versions and understanding the project structure.

### 2. Use Documentation Search Proactively

Before implementing features, search for documentation:
- Multiple related queries work better than single specific ones
- Include package names when relevant
- Search for both general concepts and specific implementation details

### 3. Test Code with Tinker Before Writing

Use Tinker integration to test:
- Model relationships and queries
- Service functionality
- Configuration values
- Factory and seeder behavior

### 4. Monitor Logs During Development

Keep log monitoring active during development:
- Watch for database query issues
- Monitor for unexpected errors
- Track performance bottlenecks

### 5. Verify Database State Regularly

Use database queries to verify:
- Migration success
- Seed data integrity
- Relationship consistency
- Index performance

## Nano Integration Tips

### Efficient File Navigation

When Claude Code suggests file locations based on Boost analysis:

```bash
# Navigate directly to suggested files
nano app/Models/User.php

# Use nano's search functionality
# Ctrl+W to search for specific methods or properties
```

### Quick Configuration Checks

Instead of opening config files, ask Claude Code:
```
"What's the current database configuration?"
"Show me the mail settings"
```

### Model Verification

Before manually inspecting model files:
```
"Test the User model relationships and show me the structure"
```

## Advanced Boost Patterns

### 1. Schema-Driven Development

```
# 1. Analyze current schema
"Show me the database schema for users and posts"

# 2. Test relationships
"Verify the user-posts relationship works correctly"

# 3. Create new features based on analysis
"Create a Filament resource for Posts with user relationship"
```

### 2. Documentation-First Implementation

```
# 1. Search for implementation patterns
"How do I implement role-based permissions in Filament 4.x?"

# 2. Understand Laravel 12 patterns
"What's the Laravel 12 way to handle authorization?"

# 3. Implement with guidance
"Create the permission system using the documented approach"
```

### 3. Continuous Verification

```
# During development, continuously verify:
"Test the new model factory"
"Check if the migration created the correct indexes"
"Verify the Filament resource displays correctly"
```

## Troubleshooting Boost Issues

### MCP Server Not Available

If Boost tools aren't working:
1. Verify Laravel Boost is installed: `composer show laravel/boost`
2. Check Claude Code MCP server configuration
3. Restart Claude Code if necessary

### Database Connection Issues

If database queries fail:
```
"Check the database configuration and test connectivity"
```

### Application Errors

If Tinker execution fails:
```
"Show me the latest application errors and help debug"
```

## Next Steps

1. **Practice with [Command-Line Development](03-command-line-development.md)** to integrate Boost into efficient workflows
2. **Learn [Database Operations](04-database-operations.md)** for advanced database management
3. **Master [Tinker Integration](05-tinker-integration.md)** for interactive development

Laravel Boost transforms Laravel development by providing immediate access to application state and version-specific documentation, making command-line development more productive than traditional IDE-based workflows.