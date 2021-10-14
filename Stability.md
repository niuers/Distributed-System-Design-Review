# Stability

## Stability Patterns
### Timeouts
### Circuit Breaker
### Let it Crash
1. Manage the crash as part of the life-cycle process
2. Process supervision
3. Supervisor hierarchies
   * 
5. Restart Strategy
   * One for One
   * All for One

### Fail Fast
1. Separate system (due resources) and application error
2. Verify resource availability before starting expensive task
3. Input validation immediately
4. 
### Bulkheads
1. Partition and tolerate failure in one part
2. Redundancy
3. 
### Steady State
1. Clean up after you
2. Logging
   * Always put log on separate disk
   * RollingFileAppender (log4j)
### Throttling
1. Maintaining a steady state
2. Count requets
   * If limit reached back-off (drop, raise error) 
3. Queue Requests
   * Used in Staged Event-Driven Architecture (SEDA) 
