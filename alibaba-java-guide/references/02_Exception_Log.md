# 第二章 异常日志

## (一) 异常处理
1.  **【强制】** Java 类库中定义的可以通过预检查方式规避的 `RuntimeException` 异常不应该通过 `catch` 的方式来处理。
    ```java
    // 正例
    if (obj != null) {
        ...
    }
    
    // 反例
    try { 
        obj.method(); 
    } catch (NullPointerException e) {
        ...
    }
    ```
2.  **【强制】** 异常不要用来做流程控制，条件控制。
3.  **【强制】** 捕获异常是为了处理它，不要捕获了却什么都不做而抛弃之。
4.  **【强制】** 事务场景中，抛出异常被 `catch` 后，如果需要回滚，一定要手动回滚事务。
    ```java
    // 正例
    try {
        // 业务逻辑
    } catch (Exception e) {
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        throw e;
    }
    ```
5.  **【强制】** `finally` 块必须对资源对象、流对象进行关闭，有异常也要做 `try-catch`。
    ```java
    // 正例
    // 推荐使用 try-with-resources
    try (Resource res = ...) {
        // 业务处理
    }
    ```

## (二) 日志规约
1.  **【强制】** 应用中不可直接使用日志系统（Log4j、Logback）中的 API，而应依赖使用日志框架 SLF4J 中的 API。
    ```java
    // 正例
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    private static final Logger logger = LoggerFactory.getLogger(Abc.class);
    ```
2.  **【强制】** 在日志输出时，字符串变量之间的拼接使用占位符的方式。
    ```java
    // 正例
    logger.debug("Processing trade with id: {} and symbol: {}", id, symbol);
    ```
3.  **【强制】** 生产环境禁止直接使用 `System.out` 或 `System.err` 输出日志或使用 `e.printStackTrace()` 打印异常堆栈。
4.  **【强制】** 异常信息应该包括两类信息：案发现场信息和异常堆栈信息。
    ```java
    // 正例
    logger.error("各类参数或者对象toString + 异常信息: ", e);
    ```
