# 第一章 编程规约

## (一) 命名风格
1.  **【强制】** 代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。
    ```java
    // 反例
    String _name;
    String __name;
    String $Object;
    String name_;
    String name$;
    String object$;
    ```
2.  **【强制】** 类名使用 `UpperCamelCase` 风格，但 DO / BO / DTO / VO / AO / PO / UID 等情形例外。
    ```java
    // 正例
    class UserDO {}
    class XmlService {}
    class TcpUdpDeal {}
    ```
3.  **【强制】** 方法名、参数名、成员变量、局部变量都统一使用 `lowerCamelCase` 风格。
    ```java
    // 正例
    localValue / getHttpMessage() / inputUserId
    ```
4.  **【强制】** 常量命名全部大写，单词间用下划线隔开。
    ```java
    // 正例
    public static final String ACCESS_TOKEN_CACHE_KEY = "access_token";
    ```
5.  **【强制】** 抽象类命名使用 `Abstract` 或 `Base` 开头；异常类命名使用 `Exception` 结尾；测试类命名以 `Test` 结尾。
6.  **【强制】** POJO 类中布尔类型的变量，都不要加 `is` 前缀。
    ```java
    // 正例
    private Boolean success; // RPC 框架解析时生成 isSuccess() 方法
    ```
7.  **【强制】** 包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。
    ```java
    // 正例
    package com.alibaba.ai.util;
    ```

## (二) 常量定义
1.  **【强制】** 不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。
    ```java
    // 反例
    String key = "Id#taobao_" + tradeId;
    
    // 正例
    String key = CACHE_PREFIX + tradeId;
    ```
2.  **【推荐】** 如果变量值仅在一个范围内变化，且带有名称之外的延伸属性，定义为枚举类。

## (三) 代码格式
1.  **【强制】** 采用 4 个空格缩进，禁止使用 tab 字符。
2.  **【强制】** 单行字符数限制不超过 120 个，超出需要换行。
3.  **【强制】** IDE文件编码设置为UTF-8; 换行符使用Unix格式。

## (四) OOP 规约
1.  **【强制】** 避免通过一个类的对象引用访问此类的静态变量或静态方法，直接用类名访问即可。
2.  **【强制】** 所有的覆写方法，必须加 `@Override` 注解。
3.  **【强制】** 外部正在调用或者二方库依赖的接口，不允许修改方法签名。
4.  **【强制】** 使用常量或确定有值的对象来调用 `equals`，避免空指针异常。
    ```java
    // 正例
    "test".equals(object);
    
    // 反例
    object.equals("test");
    ```
5.  **【强制】** 所有整型包装类对象之间值的比较，全部使用 `equals` 方法比较。
    ```java
    Integer a = 235;
    Integer b = 235;
    if (a.equals(b)) { // true }
    ```
6.  **【强制】** 定义 DO/DTO/VO 等 POJO 类时，不要设定任何属性默认值。

## (五) 日期时间
1.  **【强制】** 日期格式化字符串 `yyyy-MM-dd HH:mm:ss`。
2.  **【强制】** 获取当前毫秒数：`System.currentTimeMillis()`。
3.  **【推荐】** 使用 JDK8 的 `Instant`、`LocalDateTime`、`DateTimeFormatter`。
    ```java
    // 正例
    LocalDateTime now = LocalDateTime.now();
    DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String nowStr = now.format(format);
    ```

## (六) 集合处理
1.  **【强制】** 只要重写 `equals`，就必须重写 `hashCode`。
2.  **【强制】** 使用集合转数组的方法，必须使用 `collection.toArray(new T[0])`。
    ```java
    List<String> list = new ArrayList<>(2);
    list.add("guan");
    list.add("bao");
    String[] array = list.toArray(new String[0]);
    ```
3.  **【强制】** `Arrays.asList()` 返回的集合禁止修改。
4.  **【强制】** 不要在 `foreach` 循环里进行元素的 `remove/add` 操作，请使用 `Iterator`。
    ```java
    // 正例
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
        String item = iterator.next();
        if (删除条件) {
            iterator.remove();
        }
    }
    ```

## (七) 并发处理
1.  **【强制】** 单例对象需要保证线程安全。
2.  **【强制】** 线程资源必须通过线程池提供，不允许显式创建线程。
3.  **【强制】** 线程池不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 的方式。
    ```java
    // 正例
    ThreadPoolExecutor executor = new ThreadPoolExecutor(
        corePoolSize, 
        maximumPoolSize, 
        keepAliveTime, 
        TimeUnit.SECONDS, 
        new LinkedBlockingQueue<Runnable>(capacity), 
        new ThreadPoolExecutor.AbortPolicy()
    );
    ```
4.  **【强制】** `SimpleDateFormat` 是线程不安全的类，需要加锁或使用 `ThreadLocal`。

## (八) 控制语句
1.  **【强制】** `switch` 中必须包含 `default`。
2.  **【强制】** 在 `if/else/for/while/do` 语句中必须使用大括号。
    ```java
    // 正例
    if (flag) {
        System.out.println("hello");
    }
    ```

## (九) 注释规约
1.  **【强制】** 类、类属性、类方法的注释必须使用 Javadoc 规范。
    ```java
    /**
     * 这是一个示例方法
     * @param id 用户ID
     * @return 用户对象
     */
    public User getUser(Long id) { ... }
    ```
2.  **【强制】** 所有的抽象方法（包括接口中的方法）必须要用 Javadoc 注释。
