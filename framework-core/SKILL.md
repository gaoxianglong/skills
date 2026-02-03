---
name: spring-mybatis-framework
description: 项目基础框架规范，涵盖 Spring Boot 4.0、MyBatis 4.0、Druid 连接池、参数校验、异常处理等。适用于编写 Controller、Service、Mapper、配置类、异常处理时。触发场景包括：新建 REST API、数据库操作、参数校验、全局异常处理、配置管理。
---

# Spring + MyBatis 框架规范

## 一、技术栈版本

| 组件 | 版本 | 说明 |
|------|------|------|
| JDK | 21 | Java 运行时 |
| Spring Boot | 4.0.0 | Web 框架 |
| MyBatis | 4.0.1 | ORM 框架 |
| Druid | 1.2.27 | 数据库连接池 |
| MySQL | 9.2.0 | 数据库 |
| Fastjson2 | 2.0.43 | JSON 处理 |
| Validation | Spring Boot 内置 | 参数校验 |

## 二、Controller 规范

详见 [01_Spring_Boot.md](references/01_Spring_Boot.md)

```java
@RestController
@RequestMapping("/api/v1/{module}")
public class XxxController {
    
    private static final Logger log = LoggerFactory.getLogger(XxxController.class);
    
    private final XxxApplicationService xxxApplicationService;
    
    // 构造器注入（推荐）
    public XxxController(XxxApplicationService xxxApplicationService) {
        this.xxxApplicationService = xxxApplicationService;
    }
    
    @PostMapping("/action")
    public ApiResponse<XxxResponse> action(@Valid @RequestBody XxxRequest request) {
        // 1. 日志记录
        // 2. 构建 Command
        // 3. 调用 Application Service
        // 4. 返回 ApiResponse.success(data)
    }
}
```

## 三、MyBatis 规范

详见 [02_MyBatis.md](references/02_MyBatis.md)

| 规范 | 说明 |
|------|------|
| Mapper 接口 | 使用 `@Mapper` 注解，放在 `infrastructure/persistence/mapper/` |
| SQL 写法 | 简单 SQL 用注解，复杂 SQL 用 XML |
| 驼峰映射 | 已开启 `map-underscore-to-camel-case=true` |
| PO 类 | 放在 `infrastructure/persistence/po/`，对应数据库表 |

## 四、配置规范

详见 [03_Configuration.md](references/03_Configuration.md)

- 配置文件：`application.properties`（不使用 YAML）
- 数据源：Druid 连接池
- 日志：Logback（`logback.xml`）

## 五、参数校验与异常处理

详见 [04_Validation_Exception.md](references/04_Validation_Exception.md)

| 类型 | 处理方式 |
|------|----------|
| 参数校验 | `@Valid` + `@NotNull`/`@Size`/`@Email` 等 |
| 业务异常 | 抛出 `BusinessException`，由全局处理器捕获 |
| 系统异常 | 由 `GlobalExceptionHandler` 统一处理 |
| 统一响应 | 使用 `ApiResponse<T>` 包装 |

## 六、禁止事项

| 禁止行为 | 正确做法 |
|----------|----------|
| 使用 `@Autowired` 字段注入 | 使用构造器注入 |
| Controller 直接调用 Mapper | 必须经过 Application Service → Repository |
| 在 Controller 中写业务逻辑 | 业务逻辑放在 Application/Domain 层 |
| 使用 `System.out.println` | 使用 `LoggerFactory.getLogger()` |
| 手动拼接 SQL（字符串拼接） | 使用 MyBatis 参数绑定 `#{}` |
| 捕获异常后不处理 | 必须记录日志或重新抛出 |

## 七、详细参考资料

- [01_Spring_Boot.md](references/01_Spring_Boot.md) - Spring Boot 控制器、依赖注入、日志规范
- [02_MyBatis.md](references/02_MyBatis.md) - Mapper 接口、SQL 写法、PO 类规范
- [03_Configuration.md](references/03_Configuration.md) - 配置文件、数据源、日志配置
- [04_Validation_Exception.md](references/04_Validation_Exception.md) - 参数校验、异常体系、统一响应
