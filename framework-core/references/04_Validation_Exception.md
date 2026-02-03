# 参数校验与异常处理规范

本项目使用 Spring Boot Validation 进行参数校验，统一异常处理。

## 1. 参数校验

### 1.1 Request DTO 校验

```java
package com.gxl.plancore.user.interfaces.dto;

import jakarta.validation.constraints.*;

/**
 * 注册请求
 */
public class RegisterRequest {
    
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    @Size(max = 100, message = "邮箱长度不能超过100个字符")
    private String email;
    
    @NotBlank(message = "密码不能为空")
    @Size(min = 6, max = 32, message = "密码长度必须在6-32个字符之间")
    private String password;
    
    @NotBlank(message = "昵称不能为空")
    @Size(min = 1, max = 20, message = "昵称长度必须在1-20个字符之间")
    private String nickname;
    
    // Getters and Setters
}
```

### 1.2 常用校验注解

| 注解 | 用途 | 示例 |
|------|------|------|
| `@NotNull` | 不能为 null | `@NotNull(message = "ID不能为空")` |
| `@NotBlank` | 不能为空字符串（含空格） | `@NotBlank(message = "名称不能为空")` |
| `@NotEmpty` | 集合/数组不能为空 | `@NotEmpty(message = "列表不能为空")` |
| `@Size` | 长度/大小限制 | `@Size(min = 1, max = 100)` |
| `@Min` / `@Max` | 数值范围 | `@Min(0) @Max(100)` |
| `@Email` | 邮箱格式 | `@Email(message = "邮箱格式不正确")` |
| `@Pattern` | 正则匹配 | `@Pattern(regexp = "^1[3-9]\\d{9}$")` |
| `@Past` / `@Future` | 日期校验 | `@Past(message = "必须是过去的日期")` |

### 1.3 Controller 中使用 @Valid

```java
@PostMapping("/register")
public ApiResponse<AuthResponse> register(
        @Valid @RequestBody RegisterRequest request  // 使用 @Valid 触发校验
) {
    // 校验通过后执行业务逻辑
}
```

### 1.4 嵌套对象校验

```java
public class OrderRequest {
    
    @NotNull(message = "收货地址不能为空")
    @Valid  // 嵌套对象需要加 @Valid
    private AddressRequest address;
}

public class AddressRequest {
    
    @NotBlank(message = "省份不能为空")
    private String province;
    
    @NotBlank(message = "城市不能为空")
    private String city;
}
```

## 2. 异常体系

### 2.1 业务异常

```java
package com.gxl.plancore.common.exception;

/**
 * 业务异常
 * 用于可预期的业务错误（如邮箱已注册、密码错误等）
 */
public class BusinessException extends RuntimeException {
    
    private final int code;
    
    public BusinessException(int code, String message) {
        super(message);
        this.code = code;
    }
    
    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
    }
    
    public int getCode() {
        return code;
    }
}
```

### 2.2 错误码枚举

```java
package com.gxl.plancore.common.response;

/**
 * 错误码枚举
 * 规范：
 * - 0: 成功
 * - 1xxx: 用户相关错误
 * - 2xxx: 任务相关错误
 * - 3xxx: 打卡相关错误
 * - 9xxx: 系统错误
 */
public enum ErrorCode {
    
    // 通用错误
    SUCCESS(0, "success"),
    BAD_REQUEST(400, "请求参数错误"),
    UNAUTHORIZED(401, "未授权"),
    FORBIDDEN(403, "禁止访问"),
    NOT_FOUND(404, "资源不存在"),
    INTERNAL_ERROR(500, "系统内部错误"),
    
    // 用户相关 (1xxx)
    USER_NOT_FOUND(1001, "用户不存在"),
    INVALID_TOKEN(1003, "无效的Token"),
    EMAIL_ALREADY_REGISTERED(1004, "邮箱已注册"),
    PASSWORD_FORMAT_ERROR(1005, "密码格式错误"),
    NICKNAME_INVALID(1006, "昵称违规"),
    LOGIN_FAILED(1007, "邮箱或密码错误"),
    
    // 任务相关 (2xxx)
    TASK_NOT_FOUND(2001, "任务不存在"),
    TASK_ALREADY_COMPLETED(2002, "任务已完成"),
    
    // 系统错误 (9xxx)
    DATABASE_ERROR(9001, "数据库错误"),
    NETWORK_ERROR(9002, "网络错误");
    
    private final int code;
    private final String message;
    
    ErrorCode(int code, String message) {
        this.code = code;
        this.message = message;
    }
    
    public int getCode() { return code; }
    public String getMessage() { return message; }
}
```

### 2.3 抛出业务异常

```java
// 在 Application Service 或 Domain 中抛出
if (userRepository.findByEmail(email) != null) {
    throw new BusinessException(ErrorCode.EMAIL_ALREADY_REGISTERED);
}

// 或自定义消息
throw new BusinessException(1004, "邮箱 " + email + " 已被注册");
```

## 3. 全局异常处理

```java
package com.gxl.plancore.common.exception;

import com.gxl.plancore.common.response.ApiResponse;
import com.gxl.plancore.common.response.ErrorCode;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.validation.BindException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.stream.Collectors;

/**
 * 全局异常处理器
 */
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    /**
     * 处理业务异常
     */
    @ExceptionHandler(BusinessException.class)
    @ResponseStatus(HttpStatus.OK)  // 业务异常返回 200，通过 code 区分
    public ApiResponse<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: code={}, message={}", e.getCode(), e.getMessage());
        return ApiResponse.error(e.getCode(), e.getMessage());
    }
    
    /**
     * 处理参数校验异常（@Valid）
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining(", "));
        log.warn("参数校验失败: {}", message);
        return ApiResponse.error(ErrorCode.BAD_REQUEST.getCode(), message);
    }
    
    /**
     * 处理绑定异常
     */
    @ExceptionHandler(BindException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleBindException(BindException e) {
        String message = e.getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining(", "));
        log.warn("参数绑定失败: {}", message);
        return ApiResponse.error(ErrorCode.BAD_REQUEST.getCode(), message);
    }
    
    /**
     * 处理非法参数异常（值对象校验）
     */
    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleIllegalArgumentException(IllegalArgumentException e) {
        log.warn("非法参数: {}", e.getMessage());
        return ApiResponse.error(ErrorCode.BAD_REQUEST.getCode(), e.getMessage());
    }
    
    /**
     * 处理其他未知异常
     */
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse<Void> handleException(Exception e) {
        log.error("系统异常", e);  // 记录完整堆栈
        return ApiResponse.error(ErrorCode.INTERNAL_ERROR);
    }
}
```

## 4. 统一响应结构

```java
package com.gxl.plancore.common.response;

import java.time.Instant;
import java.util.UUID;

/**
 * 统一响应结构
 */
public class ApiResponse<T> {
    
    private int code;           // 错误码，0 表示成功
    private String message;     // 错误信息
    private T data;             // 响应数据
    private String traceId;     // 请求追踪 ID
    private long timestamp;     // 时间戳
    
    private ApiResponse() {
        this.traceId = UUID.randomUUID().toString();
        this.timestamp = Instant.now().toEpochMilli();
    }
    
    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> response = new ApiResponse<>();
        response.code = 0;
        response.message = "success";
        response.data = data;
        return response;
    }
    
    public static <T> ApiResponse<T> success() {
        return success(null);
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        ApiResponse<T> response = new ApiResponse<>();
        response.code = code;
        response.message = message;
        response.data = null;
        return response;
    }
    
    public static <T> ApiResponse<T> error(ErrorCode errorCode) {
        return error(errorCode.getCode(), errorCode.getMessage());
    }
    
    // Getters...
}
```

## 5. 响应示例

### 5.1 成功响应

```json
{
    "code": 0,
    "message": "success",
    "data": {
        "userId": "550e8400-e29b-41d4-a716-446655440000",
        "nickname": "张三"
    },
    "traceId": "abc123",
    "timestamp": 1706860800000
}
```

### 5.2 业务错误响应

```json
{
    "code": 1004,
    "message": "邮箱已注册",
    "data": null,
    "traceId": "abc123",
    "timestamp": 1706860800000
}
```

### 5.3 参数校验错误响应

```json
{
    "code": 400,
    "message": "邮箱不能为空, 密码长度必须在6-32个字符之间",
    "data": null,
    "traceId": "abc123",
    "timestamp": 1706860800000
}
```

## 6. 禁止事项

| 禁止行为 | 正确做法 |
|----------|----------|
| 捕获异常后不处理 | 记录日志并重新抛出或返回错误 |
| 在 Controller 中 try-catch 所有异常 | 由 GlobalExceptionHandler 统一处理 |
| 使用魔法数字作为错误码 | 定义 ErrorCode 枚举 |
| 返回 null 或原始 Map | 使用 ApiResponse 包装 |
| 打印敏感信息到日志 | 脱敏处理（如密码、token） |
