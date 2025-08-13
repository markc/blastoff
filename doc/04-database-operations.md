# Database Operations with Laravel Boost

Laravel Boost provides powerful database tools that allow you to query, inspect, and manage your database directly through Claude Code without leaving the command line.

## Database Query Tool

### Overview

The `database-query` tool executes read-only SQL queries against your configured database. This is perfect for:
- Inspecting database schema
- Checking data integrity
- Debugging relationships
- Analyzing table contents
- Understanding database structure

### Basic Queries

**Show all tables:**
```
Ask Claude Code: "Show me all tables in the database"
```

Executes:
```sql
SELECT name FROM sqlite_master WHERE type='table';
```

**Count records:**
```
Ask Claude Code: "How many users are in the database?"
```

Executes:
```sql
SELECT COUNT(*) FROM users;
```

**Check table structure:**
```
Ask Claude Code: "Show me the structure of the posts table"
```

Executes:
```sql
PRAGMA table_info(posts);
```

### Advanced Database Inspection

**Migration Status:**
```
Ask Claude Code: "Show me the migration history"
```

Executes:
```sql
SELECT migration, batch FROM migrations ORDER BY batch DESC, id DESC LIMIT 10;
```

**Relationship Verification:**
```
Ask Claude Code: "Check if all posts have valid user_id references"
```

Executes:
```sql
SELECT p.id, p.title, p.user_id, u.name
FROM posts p
LEFT JOIN users u ON p.user_id = u.id
WHERE u.id IS NULL;
```

**Data Analysis:**
```
Ask Claude Code: "Show me post statistics by status"
```

Executes:
```sql
SELECT status, COUNT(*) as count, AVG(LENGTH(content)) as avg_content_length
FROM posts 
GROUP BY status;
```

### Schema Analysis Patterns

**Foreign Key Constraints:**
```sql
-- SQLite specific
SELECT sql FROM sqlite_master WHERE type='table' AND name='posts';

-- Check constraint effectiveness
SELECT 
    COUNT(*) as total_posts,
    COUNT(user_id) as posts_with_user,
    COUNT(*) - COUNT(user_id) as orphaned_posts
FROM posts;
```

**Index Analysis:**
```sql
-- Show indexes for a table
SELECT name, sql FROM sqlite_master 
WHERE type='index' AND tbl_name='posts';

-- Analyze query performance (SQLite)
EXPLAIN QUERY PLAN SELECT * FROM posts WHERE user_id = 1;
```

**Data Quality Checks:**
```sql
-- Check for duplicate slugs
SELECT slug, COUNT(*) as count 
FROM posts 
GROUP BY slug 
HAVING count > 1;

-- Validate email formats
SELECT id, email FROM users 
WHERE email NOT LIKE '%@%.%';

-- Check for empty required fields
SELECT id, title FROM posts 
WHERE title IS NULL OR title = '' OR TRIM(title) = '';
```

## Database Schema Tool

### Getting Complete Schema Information

```
Ask Claude Code: "Show me the complete database schema"
```

This provides:
- All table names and structures
- Column definitions with data types
- Primary and foreign keys
- Indexes and constraints
- Table relationships

### Filtering Schema Information

```
Ask Claude Code: "Show me schema for tables starting with 'user'"
```

Returns filtered results for user-related tables only.

## Practical Database Workflows

### 1. Development Verification Workflow

**After Migration:**
```bash
# Run migration
php artisan migrate

# Verify via Claude Code:
# "Check if the new table was created correctly"
# "Show me the structure of the new table"
# "Verify foreign key constraints were added"
```

**After Seeding:**
```bash
# Run seeder
php artisan db:seed

# Verify via Claude Code:
# "Count records in each table"
# "Show sample data from each table"
# "Check if relationships are properly populated"
```

### 2. Data Integrity Verification

**Check Relationships:**
```
# Ask Claude Code:
"Find any posts without valid users"
"Check for users without any posts"
"Verify all foreign key relationships are consistent"
```

**Validate Data:**
```
# Ask Claude Code:
"Find duplicate email addresses"
"Check for posts with invalid status values"
"Show any records with null required fields"
```

### 3. Performance Analysis

**Query Performance:**
```
# Ask Claude Code:
"Show me the most complex queries I should optimize"
"Analyze which tables need indexes"
"Find tables with the most records"
```

**Data Distribution:**
```
# Ask Claude Code:
"Show me data distribution across different tables"
"Find tables that are growing fastest"
"Check for tables with skewed data"
```

## Working with Different Database Systems

### SQLite (Development)

**Schema Queries:**
```sql
-- List all tables
SELECT name FROM sqlite_master WHERE type='table';

-- Table structure
PRAGMA table_info(table_name);

-- Indexes for a table
SELECT name, sql FROM sqlite_master 
WHERE type='index' AND tbl_name='table_name';

-- Foreign keys
PRAGMA foreign_key_list(table_name);
```

**Data Analysis:**
```sql
-- Database file size
SELECT page_count * page_size as size 
FROM pragma_page_count(), pragma_page_size();

-- Table sizes
SELECT 
    name,
    SUM("pgsize") as size
FROM "dbstat" 
WHERE name NOT LIKE 'sqlite_%'
GROUP BY name
ORDER BY size DESC;
```

### MySQL/MariaDB (Production)

**Schema Queries:**
```sql
-- Show all tables
SHOW TABLES;

-- Table structure
DESCRIBE table_name;
SHOW CREATE TABLE table_name;

-- Indexes
SHOW INDEX FROM table_name;

-- Foreign keys
SELECT 
    TABLE_NAME,
    COLUMN_NAME,
    CONSTRAINT_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE REFERENCED_TABLE_NAME IS NOT NULL;
```

## Integration with Filament Development

### 1. Resource Development Workflow

**Before Creating Resource:**
```
# Ask Claude Code:
"Show me the structure of the posts table"
"What relationships exist for the Post model?"
"Check sample data in the posts table"
```

**After Creating Resource:**
```
# Ask Claude Code:
"Verify the Filament resource can access all required data"
"Check if the table has enough sample data for testing"
"Ensure all foreign keys have valid references"
```

### 2. Form Field Validation

**Check Available Options:**
```
# Before creating select fields:
"Show me all distinct values in the status column"
"List all categories available for posts"
"Check what user roles exist in the database"
```

**Validate Constraints:**
```
# Ensure form validation matches database constraints:
"Show me the maximum length for the title column"
"Check if email field has unique constraint"
"Verify which fields allow null values"
```

### 3. Relationship Debugging

**Verify Relationships Work:**
```
# Ask Claude Code:
"Test the user-posts relationship with actual data"
"Show me posts with their associated users"
"Check if all posts have valid category references"
```

**Find Orphaned Records:**
```
# Ask Claude Code:
"Find posts without valid users"
"Check for categories with no posts"
"Identify any broken relationship data"
```

## Database Connections

### Multiple Database Support

```
# Ask Claude Code:
"What database connections are configured?"
"Show me tables from the reporting database"
"Query the analytics database for user statistics"
```

### Environment-Specific Queries

**Development vs Production:**
```bash
# Check current database configuration
# Ask Claude Code: "What database am I currently connected to?"

# Verify environment-specific data
# Ask Claude Code: "Show me sample users (limit 5)"
```

## Performance Considerations

### 1. Query Optimization

**Identify Slow Operations:**
```
# Ask Claude Code:
"Find tables that might need indexes"
"Show me complex joins that could be optimized"
"Check for N+1 query patterns in the data"
```

**Index Recommendations:**
```
# Ask Claude Code:
"Analyze which columns are frequently searched"
"Suggest indexes for the posts table"
"Check if existing indexes are being used"
```

### 2. Data Volume Analysis

**Growth Patterns:**
```
# Ask Claude Code:
"Show me data growth over the last month"
"Which tables are growing fastest?"
"Check if we need to implement data archiving"
```

**Cleanup Opportunities:**
```
# Ask Claude Code:
"Find old data that could be archived"
"Check for duplicate or unnecessary records"
"Identify tables with soft-deleted records"
```

## Error Diagnosis with Database Queries

### 1. Application Error Investigation

**When Filament Shows Errors:**
```
# Ask Claude Code:
"Check if the database connection is working"
"Verify all required tables exist"
"Test if the failing query has valid data"
```

**Migration Issues:**
```
# Ask Claude Code:
"Show me the last migration that ran"
"Check if migration created the expected schema"
"Verify foreign key constraints were created correctly"
```

### 2. Data Consistency Issues

**Form Submission Failures:**
```
# Ask Claude Code:
"Check if all required foreign key references exist"
"Verify field lengths match form expectations"
"Test if unique constraints are causing conflicts"
```

**Display Issues:**
```
# Ask Claude Code:
"Check if relationship data exists for all records"
"Verify that display columns have valid data"
"Test if joins are returning expected results"
```

## Best Practices

### 1. Safe Query Patterns

- Always use read-only queries via the database-query tool
- Use LIMIT clauses when inspecting large tables
- Prefer specific column selection over SELECT *
- Use table aliases for complex joins

### 2. Efficient Data Inspection

- Start with COUNT(*) queries before SELECT * queries
- Use sampling (LIMIT with RANDOM) for large datasets
- Check constraints before examining data
- Verify relationships exist before complex joins

### 3. Development Integration

- Query database state before writing code
- Verify assumptions about data structure
- Test relationship queries match your models
- Use database insights to inform Filament resource design

## Next Steps

1. **Master [Tinker Integration](05-tinker-integration.md)** for interactive PHP and Eloquent testing
2. **Learn [Log Monitoring](06-log-monitoring.md)** for real-time debugging
3. **Practice [Filament Resources](09-filament-resources.md)** with database-informed design

The database query tool in Laravel Boost provides immediate insights into your application's data layer, enabling more informed development decisions and faster debugging of database-related issues.