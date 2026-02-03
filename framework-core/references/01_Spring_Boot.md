# Spring Boot 规范

本项目使用 Spring Boot 4.0，遵循以下规范。

## 1. Controller 规范

### 1.1 类定义

```java
package com.gxl.plancore.{context}.interfaces.controller;

import com.gxl.plancore.common.response.ApiResponse;
import jakarta.validation.Valid;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;

/**
 * 功能描述
 */
@RestController
@RequestMapping("/api/v1/{module}")
public class XxxController {
    
    // 1. 日志对象（类级别）
    private static final Logger log = LoggerFactory.getLogger(XxxController.class);
    
    // 2. 依赖（final）
    private final XxxApplicationService xxxApplicationService;
    
    // 3. 构造器注入（推荐）
    public XxxController(XxxApplicationService xxxApplicationService) {
        this.xxxApplicationService = xxxApplicationService;
    }
    
    // 4. 接口方法
}
```

### 1.2 接口方法

```java
/**
 * 接口描述
 * POST /api/v1/auth/register
 */
@PostMapping("/register")
public ApiResponse<AuthResponse> register(
        @Valid @RequestBody RegisterRequest request,
        HttpServletRequest httpRequest
) {
    // 1. 记录请求日志（脱敏敏感信息）
    log.info("收到注册请求: email={}", request.getEmail());
    
    // 2. 构建 Command 对象
    RegisterCommand command = new RegisterCommand(
            request.getEmail(),
            request.getPassword(),
            request.getNickname()
    );
    
    // 3. 调用 Application Service
    UserDTO userDTO = authApplicationService.register(command);
    
    // 4. 构建响应并返回
    AuthResponse response = buildAuthResponse(userDTO);
    return ApiResponse.success(response);
}
```

### 1.3 URL 规范

| 规范 | 示例 |
|------|------|
| 版本前缀 | `/api/v1/` |
| 资源命名 | 小写复数：`/users`、`/tasks` |
| 动作命名 | 动词：`/register`、`/login`、`/logout` |

### 1.4 HTTP 方法

| 方法 | 用途 | 示例 |
|------|------|------|
| GET | 查询 | `GET /api/v1/tasks/{id}` |
| POST | 创建 | `POST /api/v1/tasks` |
| PUT | 全量更新 | `PUT /api/v1/tasks/{id}` |
| PATCH | 部分更新 | `PATCH /api/v1/tasks/{id}` |
| DELETE | 删除 | `DELETE /api/v1/tasks/{id}` |

## 2. 依赖注入规范

### 2.1 构造器注入（推荐）

```java
// ✅ 推荐：构造器注入
@Service
public class AuthApplicationService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthApplicationService(UserRepository userRepository, 
                                   PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
}
```

### 2.2 禁止字段注入

```java
// ❌ 禁止：@Autowired 字段注入
@Service
public class AuthApplicationService {
    
    @Autowired
    private UserRepository userRepository;  // ❌ 不推荐
}
```

### 2.3 为什么推荐构造器注入

| 优点 | 说明 |
|------|------|
| 不可变性 | 字段可以声明为 `final` |
| 显式依赖 | 依赖关系一目了然 |
| 易于测试 | 可以通过构造器传入 Mock 对象 |
| 循环依赖检测 | 编译时即可发现循环依赖 |

## 3. 日志规范

### 3.1 Logger 定义

```java
// 使用 SLF4J + Logback
private static final Logger log = LoggerFactory.getLogger(XxxController.class);
```

### 3.2 日志级别

| 级别 | 用途 | 示例 |
|------|------|------|
| ERROR | 系统错误、需要立即关注 | `log.error("数据库连接失败", e)` |
| WARN | 业务异常、可预期的错误 | `log.warn("用户登录失败: userId={}", userId)` |
| INFO | 重要业务事件 | `log.info("用户注册成功: userId={}", userId)` |
| DEBUG | 调试信息（生产环境关闭） | `log.debug("查询结果: {}", result)` |

### 3.3 日志格式

```java
// ✅ 使用占位符（推荐）
log.info("用户注册成功: userId={}, email={}", userId, email);

// ❌ 禁止字符串拼接
log.info("用户注册成功: userId=" + userId);  // 性能差
```

### 3.4 敏感信息脱敏

```java
// ✅ 脱敏敏感信息
log.info("收到注册请求: email={}", request.getEmail());

// ❌ 禁止打印密码
log.info("注册: email={}, password={}", email, password);  // 绝对禁止
```

## 4. Configuration 规范

### 4.1 配置类

```java
package com.gxl.plancore.common.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 配置类描述
 */
@Configuration
public class XxxConfig {
    
    @Bean
    public XxxBean xxxBean() {
        return new XxxBean();
    }
}
```

### 4.2 配置属性类

```java
package com.gxl.plancore.common.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "app.xxx")
public class XxxProperties {
    
    private String property1;
    private int property2;
    
    // Getters and Setters
}
```

## 5. 获取客户端 IP

```java
/**
 * 获取客户端真实 IP（兼容代理）
 */
private String getClientIp(HttpServletRequest request) {
    String ip = request.getHeader("X-Forwarded-For");
    if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("X-Real-IP");
    }
    if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("Proxy-Client-IP");
    }
    if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }
    if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getRemoteAddr();
    }
    // 多个代理时取第一个
    if (ip != null && ip.contains(",")) {
        ip = ip.split(",")[0].trim();
    }
    return ip;
}
```
