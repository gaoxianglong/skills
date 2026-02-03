# 防腐层 (ACL) 模式详解

## 为什么需要防腐层？

在 DDD 架构中，一个上下文（如"任务"）经常需要访问其他上下文（如"用户"）。如果直接引入用户上下文的实体或 DTO，会导致：

1. **强耦合**：用户上下文的模型变更会直接影响任务上下文代码
2. **概念混淆**：外部的 `User` 实体可能包含大量任务不关心的字段，污染任务领域模型
3. **边界模糊**：上下文之间的依赖关系变得不清晰

## 本项目 ACL 场景

根据领域模型设计，本项目的上下文依赖关系如下：

```
┌─────────────────────────────────────────────────────────────┐
│                      用户上下文 (User)                       │
│         提供：用户身份验证、账号状态查询                       │
└─────────────────────────────────────────────────────────────┘
       ▲                  ▲                   ▲                ▲
       │ ACL              │ ACL               │ ACL            │ ACL
       │                  │                   │                │
┌──────┴──────┐  ┌────────┴─────┐  ┌──────────┴──────┐  ┌────┴──────┐
│  任务上下文  │  │ 打卡上下文   │  │  专注上下文      │  │会员上下文 │
│   (Task)    │  │  (CheckIn)   │  │   (Focus)        │  │(Membership)│
└─────────────┘  └──────────────┘  └──────────────────┘  └───────────┘
```

**Task/CheckIn/Focus/Membership 上下文**需要获取用户信息时，**必须通过 ACL** 访问，不能直接引用 User 聚合根。

## ACL 实现步骤

### 1. 在领域层定义接口 (Domain Layer)

任务上下文只需要用户的 ID 和昵称，不需要知道用户的密码或邮箱。

```java
// task/domain/acl/UserFacade.java
package com.gxl.plancore.task.domain.acl;

/**
 * 用户防腐层接口
 * 任务上下文通过此接口获取用户信息，避免直接依赖用户上下文的领域模型
 */
public interface UserFacade {
    /**
     * 获取任务所需的用户信息
     * @param userId 用户ID
     * @return 任务上下文关注的用户信息
     */
    TaskUser getUserInfo(String userId);
    
    /**
     * 检查用户是否存在
     */
    boolean userExists(String userId);
}

// task/domain/acl/TaskUser.java
package com.gxl.plancore.task.domain.acl;

/**
 * 任务上下文的用户模型（仅包含任务关注的字段）
 */
public class TaskUser {
    private final String userId;
    private final String nickname;

    public TaskUser(String userId, String nickname) {
        this.userId = userId;
        this.nickname = nickname;
    }

    public String getUserId() { return userId; }
    public String getNickname() { return nickname; }
}
```

### 2. 在基础设施层实现接口 (Infrastructure Layer)

这里负责调用用户上下文，并进行模型转换。

```java
// task/infrastructure/acl/UserFacadeImpl.java
package com.gxl.plancore.task.infrastructure.acl;

import com.gxl.plancore.task.domain.acl.TaskUser;
import com.gxl.plancore.task.domain.acl.UserFacade;
import com.gxl.plancore.user.domain.entity.User;
import com.gxl.plancore.user.domain.repository.UserRepository;
import com.gxl.plancore.user.domain.valueobject.UserId;
import org.springframework.stereotype.Service;

@Service
public class UserFacadeImpl implements UserFacade {

    private final UserRepository userRepository;

    public UserFacadeImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public TaskUser getUserInfo(String userId) {
        // 1. 调用用户上下文的仓储
        User user = userRepository.findByUserId(UserId.of(userId));
        
        if (user == null) {
            throw new IllegalArgumentException("用户不存在: " + userId);
        }
        
        // 2. 核心：防腐转换
        // 将用户上下文的领域对象转换为任务上下文的模型
        return new TaskUser(
            user.getUserId().getValue(),
            user.getNickname().getValue()
        );
    }

    @Override
    public boolean userExists(String userId) {
        return userRepository.findByUserId(UserId.of(userId)) != null;
    }
}
```

### 3. 在应用层/领域服务中使用

业务代码只依赖 `UserFacade` 接口，完全不知道底层实现。

```java
// task/application/service/TaskApplicationService.java
package com.gxl.plancore.task.application.service;

@Service
public class TaskApplicationService {
    
    private final TaskRepository taskRepository;
    private final UserFacade userFacade;  // 通过 ACL 访问用户上下文

    public TaskApplicationService(TaskRepository taskRepository, UserFacade userFacade) {
        this.taskRepository = taskRepository;
        this.userFacade = userFacade;
    }

    @Transactional
    public TaskDTO createTask(CreateTaskCommand command) {
        // 1. 通过 ACL 校验用户是否存在
        if (!userFacade.userExists(command.getUserId())) {
            throw new BusinessException(ErrorCode.USER_NOT_FOUND);
        }
        
        // 2. 创建任务...
        Task task = Task.create(
            UserId.of(command.getUserId()),
            TaskTitle.of(command.getTitle()),
            Priority.valueOf(command.getPriority())
        );
        
        taskRepository.save(task);
        return TaskConverter.toDTO(task);
    }
}
```

## 最佳实践

| 原则 | 说明 |
|------|------|
| **接口定义在 Domain 层** | 接口是为了支持领域逻辑而存在的 |
| **实现在 Infrastructure 层** | 因为涉及外部 I/O 和跨上下文依赖 |
| **模型转换在 ACL 中完成** | 外部模型不应泄露到 Domain 或 Application 层 |
| **只暴露需要的字段** | TaskUser 只包含 userId 和 nickname，不包含 password/email 等 |
| **保持简单** | 如果只需要 userId，不需要过度封装；但涉及字段映射时，必须使用 ACL |

## 禁止事项

```java
// ❌ 错误：直接在任务上下文中引用用户上下文的聚合根
import com.gxl.plancore.user.domain.entity.User;

public class TaskDomainService {
    private final UserRepository userRepository;  // ❌ 直接依赖用户仓储
    
    public void createTask(String userId) {
        User user = userRepository.findByUserId(userId);  // ❌ 获取完整用户实体
        // ...
    }
}

// ✅ 正确：通过 ACL 接口访问
import com.gxl.plancore.task.domain.acl.UserFacade;

public class TaskDomainService {
    private final UserFacade userFacade;  // ✅ 依赖 ACL 接口
    
    public void createTask(String userId) {
        TaskUser user = userFacade.getUserInfo(userId);  // ✅ 只获取需要的信息
        // ...
    }
}
```

## 何时需要 ACL

| 场景 | 是否需要 ACL |
|------|-------------|
| 同一上下文内的服务调用 | ❌ 不需要 |
| 跨上下文访问其他上下文的数据 | ✅ 需要 |
| 调用外部 RPC/HTTP 服务 | ✅ 需要 |
| 只需要传递 ID（不需要其他字段） | ⚠️ 可选（建议使用值对象包装） |
