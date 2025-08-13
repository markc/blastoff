# [🔙](README.md) Log Monitoring with Laravel Boost

Laravel Boost provides powerful log monitoring capabilities that allow you to monitor application logs in real-time through Claude Code without leaving the command line.

## Log Monitoring Tools

### Overview

Laravel Boost offers several tools for log monitoring:
- **Real-time log reading** with proper PSR-3 formatting
- **Browser log monitoring** for frontend debugging
- **Error detection and analysis** 
- **Log level filtering and parsing**
- **Integration with Laravel Pail** for enhanced log viewing

### Application Log Monitoring

**Basic Log Reading:**
```
Ask Claude Code: "Show me the latest application logs"
```

This uses the `read-log-entries` tool to display recent log entries with proper formatting.

**Error-Specific Monitoring:**
```
Ask Claude Code: "Show me recent application errors"
```

Filters logs to show only error-level entries for focused debugging.

**Real-Time Monitoring:**
```bash
# Start Pail in development environment
composer run dev
# This includes: php artisan pail --timeout=0
```

### Browser Log Integration

**Frontend Error Monitoring:**
```
Ask Claude Code: "Check for JavaScript errors in the browser"
```

Uses the `browser-logs` tool to show:
- Console errors and warnings
- JavaScript exceptions
- Network request failures
- Performance issues

## Development Workflow Integration

### 1. Continuous Monitoring

During development, keep log monitoring active:
```bash
# Terminal 1: Development server with logs
composer run dev

# Terminal 2: Ask Claude Code periodically:
# "Any new errors in the application logs?"
# "Check browser logs for JavaScript issues"
```

### 2. Error Investigation

When issues occur:
```
# Ask Claude Code:
"What's the latest error in the logs?"
"Show me database-related errors from today"
"Check for failed queue jobs"
```

### 3. Performance Monitoring

Track performance issues:
```
# Ask Claude Code:
"Show me slow query logs"
"Check for memory usage warnings"
"Find performance bottlenecks in the logs"
```

## Advanced Log Analysis

### 1. Log Level Filtering

Monitor specific log levels:
```
# Ask Claude Code:
"Show me warning-level logs from the last hour"
"Filter logs to show only critical errors"
"Display debug information for authentication"
```

### 2. Component-Specific Monitoring

Focus on specific application components:
```
# Ask Claude Code:
"Show logs related to database queries"
"Check Filament-specific log entries"
"Monitor cache-related log messages"
```

### 3. Pattern Recognition

Identify recurring issues:
```
# Ask Claude Code:
"Find repeated error patterns in the logs"
"Show me all 404 errors from today"
"Check for failed login attempts"
```

## Integration with Filament Development

### 1. Resource Development Debugging

When building Filament resources:
```
# Ask Claude Code:
"Check for Filament resource errors"
"Show validation errors from form submissions"
"Monitor table loading issues"
```

### 2. Form and Validation Monitoring

Track form-related issues:
```
# Ask Claude Code:
"Show form validation errors"
"Check for file upload failures"
"Monitor relationship loading issues"
```

### 3. Performance Optimization

Optimize based on log insights:
```
# Ask Claude Code:
"Find N+1 query warnings in logs"
"Check for slow Filament page loads"
"Monitor memory usage during bulk operations"
```

## Best Practices

### 1. Proactive Monitoring

- Check logs regularly during development
- Monitor both application and browser logs
- Use log levels appropriately in your code

### 2. Error Context

- Always check full error context, not just error messages
- Look for patterns in error timing and frequency
- Correlate errors with specific user actions

### 3. Development Integration

- Keep log monitoring active during feature development
- Test error scenarios and verify proper logging
- Use logs to validate fix effectiveness

## Next Steps

1. **Learn [Documentation Search](07-documentation-search.md)** for finding solutions to logged issues
2. **Master [Filament Resources](09-filament-resources.md)** with log-informed debugging
3. **Practice [Testing Strategies](16-testing-strategies.md)** that include log validation

Log monitoring with Laravel Boost provides immediate visibility into application behavior, enabling faster debugging and more reliable development.