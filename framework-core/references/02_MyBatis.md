# MyBatis 规范

本项目使用 MyBatis 4.0.1，遵循以下规范。

## 1. Mapper 接口规范

### 1.1 位置与命名

```
{context}/infrastructure/persistence/mapper/
└── {Entity}Mapper.java
```

示例：`com.gxl.plancore.user.infrastructure.persistence.mapper.UserMapper`

### 1.2 接口定义

```java
package com.gxl.plancore.user.infrastructure.persistence.mapper;

import com.gxl.plancore.user.infrastructure.persistence.po.UserPO;
import org.apache.ibatis.annotations.*;

/**
 * 用户 MyBatis Mapper
 */
@Mapper
public interface UserMapper {
    
    @Select("SELECT id, user_id, email, password, nickname, avatar, " +
            "created_at, updated_at, deleted_at " +
            "FROM user WHERE user_id = #{userId} AND deleted_at IS NULL")
    UserPO findByUserId(@Param("userId") String userId);
    
    @Select("SELECT id, user_id, email, password, nickname, avatar, " +
            "created_at, updated_at, deleted_at " +
            "FROM user WHERE email = #{email} AND deleted_at IS NULL")
    UserPO findByEmail(@Param("email") String email);
    
    @Select("SELECT COUNT(1) > 0 FROM user WHERE email = #{email} AND deleted_at IS NULL")
    boolean existsByEmail(@Param("email") String email);
    
    @Insert("INSERT INTO user (user_id, email, password, nickname, avatar, " +
            "created_at, updated_at) " +
            "VALUES (#{userId}, #{email}, #{password}, #{nickname}, #{avatar}, " +
            "#{createdAt}, #{updatedAt})")
    int insert(UserPO userPO);
    
    @Update("UPDATE user SET nickname = #{nickname}, updated_at = #{updatedAt} " +
            "WHERE user_id = #{userId} AND deleted_at IS NULL")
    int updateNickname(@Param("userId") String userId, 
                       @Param("nickname") String nickname,
                       @Param("updatedAt") Instant updatedAt);
}
```

## 2. SQL 写法规范

### 2.1 简单 SQL 用注解

```java
// 适用于：单表简单查询、插入、更新
@Select("SELECT * FROM user WHERE id = #{id}")
UserPO findById(@Param("id") Long id);

@Insert("INSERT INTO user (email, nickname) VALUES (#{email}, #{nickname})")
int insert(UserPO userPO);

@Update("UPDATE user SET nickname = #{nickname} WHERE id = #{id}")
int update(UserPO userPO);

@Delete("DELETE FROM user WHERE id = #{id}")
int delete(@Param("id") Long id);
```

### 2.2 复杂 SQL 用 XML

当 SQL 包含动态条件、多表关联、复杂逻辑时，使用 XML 文件：

```xml
<!-- resources/mapper/UserMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.gxl.plancore.user.infrastructure.persistence.mapper.UserMapper">
    
    <!-- 动态条件查询 -->
    <select id="findByCondition" resultType="UserPO">
        SELECT * FROM user
        <where>
            <if test="email != null">
                AND email = #{email}
            </if>
            <if test="nickname != null">
                AND nickname LIKE CONCAT('%', #{nickname}, '%')
            </if>
            AND deleted_at IS NULL
        </where>
        ORDER BY created_at DESC
        LIMIT #{offset}, #{limit}
    </select>
    
    <!-- 批量插入 -->
    <insert id="batchInsert">
        INSERT INTO user (user_id, email, nickname, created_at)
        VALUES
        <foreach collection="list" item="item" separator=",">
            (#{item.userId}, #{item.email}, #{item.nickname}, #{item.createdAt})
        </foreach>
    </insert>
    
</mapper>
```

### 2.3 参数绑定

```java
// ✅ 使用 #{} 参数绑定（防 SQL 注入）
@Select("SELECT * FROM user WHERE email = #{email}")
UserPO findByEmail(@Param("email") String email);

// ❌ 禁止使用 ${} 字符串拼接
@Select("SELECT * FROM user WHERE email = '${email}'")  // SQL 注入风险！
UserPO findByEmail(@Param("email") String email);
```

### 2.4 多参数处理

```java
// 方式1：@Param 注解（推荐）
@Select("SELECT * FROM user WHERE email = #{email} AND status = #{status}")
UserPO findByEmailAndStatus(@Param("email") String email, @Param("status") Integer status);

// 方式2：Map 参数
@Select("SELECT * FROM user WHERE email = #{email} AND status = #{status}")
UserPO findByCondition(Map<String, Object> params);

// 方式3：对象参数
@Select("SELECT * FROM user WHERE email = #{email} AND status = #{status}")
UserPO findByCondition(UserQuery query);
```

## 3. PO 类规范

### 3.1 位置与命名

```
{context}/infrastructure/persistence/po/
└── {Entity}PO.java
```

### 3.2 类定义

```java
package com.gxl.plancore.user.infrastructure.persistence.po;

import java.time.Instant;
import java.time.LocalDate;

/**
 * 用户持久化对象
 * 对应数据库 user 表
 */
public class UserPO {
    
    private Long id;                    // 主键（自增）
    private String userId;              // 业务 ID（UUID）
    private String email;
    private String password;
    private String nickname;
    private String avatar;
    private String ipLocation;
    private Integer consecutiveDays;
    private LocalDate lastCheckInDate;
    private Instant createdAt;
    private Instant updatedAt;
    private Instant deletedAt;          // 软删除标记
    
    // Getters and Setters（不使用 Lombok）
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    // ... 其他字段
}
```

### 3.3 字段映射

| 数据库字段 | Java 字段 | 类型映射 |
|------------|-----------|----------|
| `user_id` | `userId` | 驼峰自动映射 |
| `created_at` | `createdAt` | `Instant` |
| `last_check_in_date` | `lastCheckInDate` | `LocalDate` |
| `deleted_at` | `deletedAt` | `Instant`（null 表示未删除） |

**注意**：已开启驼峰映射 `map-underscore-to-camel-case=true`，无需手动配置。

## 4. Converter 规范

### 4.1 PO ↔ Entity 转换

```java
package com.gxl.plancore.user.infrastructure.persistence.converter;

import com.gxl.plancore.user.domain.entity.User;
import com.gxl.plancore.user.domain.valueobject.*;
import com.gxl.plancore.user.infrastructure.persistence.po.UserPO;
import org.springframework.stereotype.Component;

/**
 * 用户 PO ↔ 领域对象 转换器
 */
@Component
public class UserConverter {
    
    /**
     * PO → 领域对象
     */
    public User toDomain(UserPO po) {
        if (po == null) {
            return null;
        }
        return User.reconstitute(
            UserId.of(po.getUserId()),
            Email.of(po.getEmail()),
            Password.fromHashedValue(po.getPassword()),
            Nickname.of(po.getNickname()),
            po.getAvatar(),
            po.getIpLocation(),
            po.getCreatedAt()
        );
    }
    
    /**
     * 领域对象 → PO
     */
    public UserPO toPO(User user) {
        if (user == null) {
            return null;
        }
        UserPO po = new UserPO();
        po.setUserId(user.getUserId().getValue());
        po.setEmail(user.getEmail().getValue());
        po.setPassword(user.getPassword().getHashedValue());
        po.setNickname(user.getNickname().getValue());
        po.setAvatar(user.getAvatar());
        po.setIpLocation(user.getIpLocation());
        po.setCreatedAt(user.getCreatedAt());
        po.setUpdatedAt(Instant.now());
        return po;
    }
}
```

## 5. 配置说明

```properties
# application.properties

# Mapper XML 位置
mybatis.mapper-locations=classpath:mapper/*.xml

# 类型别名包（简化 XML 中的类型引用）
mybatis.type-aliases-package=com.gxl.plancore.domain.entity

# 驼峰映射（user_id → userId）
mybatis.configuration.map-underscore-to-camel-case=true

# SQL 日志输出
mybatis.configuration.log-impl=org.apache.ibatis.logging.slf4j.Slf4jImpl
```

## 6. 禁止事项

| 禁止行为 | 正确做法 |
|----------|----------|
| 使用 `${}` 拼接 SQL | 使用 `#{}` 参数绑定 |
| 在 Mapper 中写业务逻辑 | Mapper 只负责 SQL，业务逻辑放 Service |
| 返回 `Map<String, Object>` | 定义明确的 PO 类 |
| 使用 `SELECT *` | 明确列出需要的字段 |
| 在 Controller 直接调用 Mapper | 经过 Repository → Application Service |
