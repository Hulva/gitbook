# Kafka Schedule
> A Schedule for running jobs 

> This interface controls a job scheduler that allows scheduling either repeating background jobs that execute periodically or delayed one-time actions that are scheduled in the future.

1. Initialize a thread pool
2. Provide a method `schedule()` to create and execute one-shot or periodic actions

## Method
- `void startup()`
- `void shutdown()`
- `void schedule(String name, Runnable fun, long delay, long period, TimeUnit unit)`
