# 应用分层架构详解

本项目遵循 DDD（领域驱动设计）四层架构规范，强调**依赖倒置**和**关注点分离**。

## 1. 接口层 (Interfaces)

**职责**：处理外部请求（HTTP/RPC），解析参数，权限校验，调用应用层服务。

**注意**：
- 只做参数校验和请求转发，不包含业务逻辑
- 使用构造器注入（Spring Boot 推荐）
- 统一返回 `ApiResponse<T>` 结构

```java
// user/interfaces/controller/AuthController.java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    
    private final AuthApplicationService authApplicationService;

    // 构造器注入（推荐）
    public AuthController(AuthApplicationService authApplicationService) {
        this.authApplicationService = authApplicationService;
    }

    @PostMapping("/register")
    public ApiResponse<AuthResponse> register(@RequestBody @Valid RegisterRequest request) {
        // 1. 参数校验（由 @Valid 处理）
        // 2. 转换请求对象为 Command
        RegisterCommand command = new RegisterCommand(
            request.getEmail(),
            request.getPassword(),
            request.getNickname(),
            request.getDeviceInfo()
        );
        
        // 3. 调用应用服务
        AuthResponse response = authApplicationService.register(command);
        
        // 4. 返回统一响应
        return ApiResponse.success(response);
    }
}
```

## 2. 应用层 (Application)

**职责**：编排业务流程，控制事务边界，**不包含核心业务逻辑**。

**注意**：
- 事务边界在此层控制（`@Transactional`）
- 调用领域服务处理复杂业务逻辑
- 负责 DTO 与领域对象的转换

```java
// user/application/service/AuthApplicationService.java
@Service
public class AuthApplicationService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthApplicationService(UserRepository userRepository, 
                                   PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Transactional
    public AuthResponse register(RegisterCommand command) {
        // 1. 校验邮箱是否已注册
        Email email = Email.of(command.getEmail());
        if (userRepository.findByEmail(email) != null) {
            throw new BusinessException(ErrorCode.EMAIL_ALREADY_REGISTERED);
        }
        
        // 2. 创建领域对象（值对象封装校验逻辑）
        Password password = Password.fromPlainText(command.getPassword(), passwordEncoder);
        Nickname nickname = Nickname.of(command.getNickname());
        
        // 3. 构建用户聚合根
        User user = User.create(email, password, nickname);
        
        // 4. 持久化
        userRepository.save(user);
        
        // 5. 生成 Token 并返回
        return buildAuthResponse(user);
    }
}
```

## 3. 领域层 (Domain)

**职责**：包含核心业务逻辑、领域模型（实体、值对象）、领域服务。**不依赖 Infrastructure 层**。

**注意**：
- 领域层是纯 POJO，不依赖 Spring/MyBatis 等框架
- 使用值对象封装业务规则（如邮箱格式、密码强度）
- 聚合根包含业务行为（充血模型）

```java
// user/domain/entity/User.java（聚合根）
public class User {
    private UserId userId;
    private Email email;
    private Password password;
    private Nickname nickname;
    private Instant createdAt;

    // 工厂方法：封装创建逻辑
    public static User create(Email email, Password password, Nickname nickname) {
        User user = new User();
        user.userId = UserId.generate();
        user.email = email;
        user.password = password;
        user.nickname = nickname;
        user.createdAt = Instant.now();
        return user;
    }

    // 业务行为：修改昵称（封装7天内最多2次的规则）
    public void updateNickname(Nickname newNickname, Instant now) {
        // 业务规则校验...
        this.nickname = newNickname;
    }
}

// user/domain/valueobject/Email.java（值对象）
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
    
    // equals, hashCode, toString...
}

// user/domain/repository/UserRepository.java（仓储接口）
public interface UserRepository {
    void save(User user);
    User findByUserId(UserId userId);
    User findByEmail(Email email);
}
```

## 4. 基础设施层 (Infrastructure)

**职责**：提供技术实现（数据库、缓存、消息队列、外部接口）。**实现 Domain 层定义的接口**。

**注意**：
- 实现领域层定义的 Repository 接口
- 负责 PO（持久化对象）与领域对象的转换
- 外部服务调用放在 `acl/` 包中

```java
// user/infrastructure/persistence/repository/UserRepositoryImpl.java
@Repository
public class UserRepositoryImpl implements UserRepository {

    private final UserMapper userMapper;
    private final UserConverter userConverter;

    public UserRepositoryImpl(UserMapper userMapper, UserConverter userConverter) {
        this.userMapper = userMapper;
        this.userConverter = userConverter;
    }

    @Override
    public void save(User user) {
        // 1. 将领域对象转换为持久化对象 (PO)
        UserPO userPO = userConverter.toPO(user);
        
        // 2. 数据库操作
        if (userMapper.selectById(userPO.getId()) == null) {
            userMapper.insert(userPO);
        } else {
            userMapper.updateById(userPO);
        }
    }

    @Override
    public User findByEmail(Email email) {
        UserPO userPO = userMapper.selectByEmail(email.getValue());
        return userPO == null ? null : userConverter.toDomain(userPO);
    }
}
```

## 5. 依赖关系图

```
Interfaces ──────→ Application ──────→ Domain ←────── Infrastructure
(Controller)       (Service)          (Entity)        (Repository Impl)
    │                  │                 │                   │
    │                  │                 │                   │
    ▼                  ▼                 ▼                   ▼
 Request/          Command/         Entity/             PO/Mapper/
 Response DTO      App DTO         ValueObject         Converter
```

**依赖方向**：外层依赖内层，Infrastructure 实现 Domain 定义的接口（依赖倒置）。
