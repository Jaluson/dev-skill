# Thread Pool and Resource Standards

> Applicable mode: Development mode + Review mode
> Rule prefix: R-POOL

## R-POOL-01 Use pooled resources

Resource types that must be pooled:

| Resource Type | Pooling Method | Config Location |
|---------------|----------------|-----------------|
| Database connection | HikariCP (Spring Boot default) | `spring.datasource.hikari.*` |
| Async task threads | `ThreadPoolTaskExecutor` | Custom config class |
| HTTP connections | Apache HttpClient connection pool | Custom config class |

**Prohibited behaviors**:
- Creating new threads per request
- Manually managing database connection lifecycle (handled by HikariCP)

## R-POOL-02 Thread pool parameters auto-tuning

Thread pool configuration classes must support auto-tuning: when the user has not filled in core parameters, calculate automatically based on system resources.

**Configuration class template**:
```java
@Configuration
public class ThreadPoolConfig {

    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        
        // Auto-tuning: calculate based on CPU core count when not configured
        int processors = Runtime.getRuntime().availableProcessors();
        executor.setCorePoolSize(processors);
        executor.setMaxPoolSize(processors * 2);
        executor.setQueueCapacity(500);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        executor.initialize();
        
        return executor;
    }
}
```

**Tuning formulas**:
- `corePoolSize` = CPU core count
- `maxPoolSize` = CPU core count × 2
- `queueCapacity` = set based on business peak, default 500

Users can override via config file:
```yaml
thread:
  pool:
    core-size: 4
    max-size: 8
    queue-capacity: 1000
```

## R-POOL-03 Async tasks must be submitted to thread pool

All async tasks must be submitted to a thread pool for execution; directly calling `new Thread()` is prohibited.

**Correct example**:
```java
@Service
public class OrderService {
    @Autowired
    @Qualifier("taskExecutor")
    private Executor executor;
    
    public void asyncNotify(Order order) {
        CompletableFuture.runAsync(() -> {
            // send notification
        }, executor);
    }
}
```

Or with `@Async` annotation:
```java
@Async("taskExecutor")
public void asyncNotify(Order order) {
    // send notification
}
```

**Incorrect example**:
```java
public void asyncNotify(Order order) {
    new Thread(() -> {
        // send notification
    }).start();
}
```

Review checkpoints:
- Scan `new Thread(` calls
- Scan whether `@Async` specifies a thread pool name
- Check whether `CompletableFuture.runAsync` passes a custom Executor
