---
name: java-engineering-structure
description: 遵循项目 DDD 分层、领域模型与防腐层的 Java 工程结构约束。适用于编写后端代码、新增模块、跨上下文协作、代码评审时。触发场景包括：新建 Controller/Service/Repository、领域模型设计、跨限界上下文调用、包结构调整。
---

# Java 工程结构规范

## 一、应用分层
asdasd

详见 [01_Layered_Architecture.md](references/01_Layered_Architecture.md)

| 层 | 职责 | 本项目对应包 | 依赖规则 |
|----|------|--------------|----------|
| 开放接口层 | RPC/HTTP 接口暴露、网关 | `interfaces/` | → Application |
| Web 层 | 路由、参数校验、简单转发 | `interfaces/controller/` | → Application |
| Service 层 | 业务流程编排、事务边界 | `application/service/` | → Domain |
| Manager 层 | 通用业务处理（可选） | `domain/service/` | → Domain |
| DAO 层 | 数据访问 | `infrastructure/persistence/` | 实现 Domain 接口 |
| 外部接口 | 第三方 RPC/HTTP | `infrastructure/acl/` | 经防腐层访问 |

## 二、DDD 四层职责

- **Interfaces**：接收 HTTP 请求、参数校验、调用 Application Service
- **Application**：编排业务流程、控制事务边界、调用 Domain Service
- **Domain**：核心业务逻辑、领域规则，**不依赖其他层**（纯 POJO）
- **Infrastructure**：持久化、外部服务，**实现 Domain 定义的接口**（依赖倒置）

## 三、防腐层（ACL）

详见 [03_ACL_Anti_Corruption_Layer.md](references/03_ACL_Anti_Corruption_Layer.md)

- 跨限界上下文访问时，**必须经防腐层转换**，禁止直接使用外部领域模型
- 本上下文定义自己的接口；防腐层负责将外部模型转为本上下文的 DTO/领域对象
- 本项目示例：Task/CheckIn/Focus/Membership 访问 User 时，通过 `UserFacade` 获取用户信息

## 四、包结构约定

详见 [02_Package_Structure.md](references/02_Package_Structure.md)

```text
com.gxl.plancore.{context}/
├── interfaces/controller/           # 控制器
├── interfaces/dto/                   # Request/Response DTO
├── application/service/              # 应用服务
├── application/command/              # 命令对象
├── application/dto/                  # 应用层 DTO
├── domain/entity/                    # 聚合根/实体
├── domain/valueobject/               # 值对象
├── domain/repository/                # 仓储接口
├── domain/service/                   # 领域服务（可选）
├── infrastructure/persistence/       # PO、Mapper、Converter、Repository 实现
└── infrastructure/acl/               # 防腐层实现
```

共享内核：`com.gxl.plancore.common/`（config、exception、response 等）

## 五、禁止事项

| 禁止行为 | 说明 |
|----------|------|
| Domain 依赖 Infrastructure | 领域层不能引用持久化类（PO、Mapper） |
| 跨上下文直接引用聚合根 | 必须通过 ACL 获取，不能直接 import 其他上下文的 Entity |
| Controller 直接操作数据库 | 必须经 Application Service 编排 |
| Application 包含核心业务逻辑 | 核心逻辑应放在 Domain 层 |
| 外部 DTO 泄露到 Domain 层 | 外部服务返回的 DTO 必须在 ACL 中转换 |

## 六、详细参考资料

- [01_Layered_Architecture.md](references/01_Layered_Architecture.md) - 各层职责详解与代码示例
- [02_Package_Structure.md](references/02_Package_Structure.md) - 完整目录结构树与包职责说明
- [03_ACL_Anti_Corruption_Layer.md](references/03_ACL_Anti_Corruption_Layer.md) - 防腐层实现步骤与最佳实践
