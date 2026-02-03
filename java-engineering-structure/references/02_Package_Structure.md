# 包结构约定详解

本项目的包结构设计旨在清晰分离各层职责，方便维护和扩展。

## 目录结构树（本项目实际结构）

```text
com.gxl.plancore
├── common                       // [共享内核] 跨上下文共享
│   ├── config                   // 全局配置类（如 SecurityConfig）
│   ├── exception                // 异常体系（BusinessException、GlobalExceptionHandler）
│   └── response                 // 统一响应（ApiResponse、ErrorCode）
│
├── user                         // [用户上下文]
│   ├── interfaces               // 接口层
│   │   ├── controller           // Web 控制器（AuthController）
│   │   └── dto                  // Request/Response DTO
│   ├── application              // 应用层
│   │   ├── service              // 应用服务（AuthApplicationService）
│   │   ├── command              // 命令对象（RegisterCommand）
│   │   └── dto                  // 应用层 DTO（UserDTO）
│   ├── domain                   // 领域层
│   │   ├── entity               // 聚合根（User）
│   │   ├── valueobject          // 值对象（UserId、Email、Password、Nickname）
│   │   ├── service              // 领域服务（可选）
│   │   └── repository           // 仓储接口（UserRepository）
│   └── infrastructure           // 基础设施层
│       ├── persistence          // 持久化
│       │   ├── mapper           // MyBatis Mapper（UserMapper）
│       │   ├── po               // 持久化对象（UserPO）
│       │   ├── converter        // PO ↔ Entity 转换（UserConverter）
│       │   └── repository       // 仓储实现（UserRepositoryImpl）
│       └── acl                  // 防腐层（若有外部依赖）
│
├── task                         // [任务上下文] 结构同上
├── checkin                      // [打卡上下文] 结构同上
├── focus                        // [专注上下文] 结构同上
└── membership                   // [会员上下文] 结构同上
```

## 核心包说明

### 1. `common/`（共享内核）

跨所有限界上下文共享的基础设施：

| 包 | 职责 | 示例 |
|----|------|------|
| `config/` | 全局配置 | `SecurityConfig`（密码加密器） |
| `exception/` | 异常体系 | `BusinessException`、`GlobalExceptionHandler` |
| `response/` | 统一响应 | `ApiResponse<T>`、`ErrorCode` 枚举 |

### 2. `domain/repository` vs `infrastructure/persistence/repository`

| 位置 | 职责 | 示例 |
|------|------|------|
| `domain/repository/` | **仅定义接口**，方法签名使用领域对象 | `interface UserRepository { User findByEmail(Email email); }` |
| `infrastructure/.../repository/` | **实现接口**，内部调用 Mapper | `class UserRepositoryImpl implements UserRepository` |

### 3. `interfaces/dto` vs `application/dto`

| 位置 | 职责 | 注解 |
|------|------|------|
| `interfaces/dto/` | HTTP 契约，Request/Response | 可包含 `@NotNull`、`@Size` 等校验注解 |
| `application/dto/` | 应用服务内部传递的数据结构 | 一般不包含校验注解 |

**简单场景**：Controller 可直接返回 `application/dto`  
**复杂场景**：Controller 将 `application/dto` 转换为 `interfaces/dto`（如版本兼容）

### 4. `infrastructure/acl/`（防腐层）

存放所有对**外部系统**或**其他上下文**的调用逻辑：

- 外部系统的 DTO **不应该泄露**到 Domain 或 Application 层
- 在 ACL 中完成模型转换

## 代码示例

### 值对象（不可变，封装业务规则）

```java
package com.gxl.plancore.user.domain.valueobject;

public class Email {
    private final String value;
    private static final String EMAIL_REGEX = "^[A-Za-z0-9+_.-]+@(.+)$";

    private Email(String value) {
        this.value = value;
    }

    public static Email of(String value) {
        if (value == null || !value.matches(EMAIL_REGEX)) {
            throw new IllegalArgumentException("邮箱格式不正确");
        }
        return new Email(value.toLowerCase().trim());
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Email email = (Email) o;
        return Objects.equals(value, email.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

### 统一响应结构

```java
package com.gxl.plancore.common.response;

public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
    private long timestamp;

    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> response = new ApiResponse<>();
        response.code = 0;
        response.message = "success";
        response.data = data;
        response.timestamp = System.currentTimeMillis();
        return response;
    }

    public static <T> ApiResponse<T> error(ErrorCode errorCode) {
        ApiResponse<T> response = new ApiResponse<>();
        response.code = errorCode.getCode();
        response.message = errorCode.getMessage();
        response.timestamp = System.currentTimeMillis();
        return response;
    }
}
```
