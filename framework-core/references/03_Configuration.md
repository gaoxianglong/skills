# 配置规范

本项目使用 `application.properties` 配置文件，不使用 YAML 格式。

## 1. 配置文件结构

```properties
# ============================================
# 服务配置
# ============================================
server.port=8080
spring.application.name=plan-core

# ============================================
# MySQL 数据源配置
# ============================================
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/PlanCore?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ============================================
# Druid 连接池配置
# ============================================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.druid.initial-size=5
spring.datasource.druid.min-idle=5
spring.datasource.druid.max-active=20
spring.datasource.druid.max-wait=60000
spring.datasource.druid.time-between-eviction-runs-millis=60000
spring.datasource.druid.min-evictable-idle-time-millis=300000
spring.datasource.druid.validation-query=SELECT 1
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false

# ============================================
# MyBatis 配置
# ============================================
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.gxl.plancore.domain.entity
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.log-impl=org.apache.ibatis.logging.slf4j.Slf4jImpl

# ============================================
# 日志配置
# ============================================
logging.config=classpath:logback.xml
```

## 2. Druid 连接池参数说明

| 参数 | 值 | 说明 |
|------|-----|------|
| `initial-size` | 5 | 初始化连接数 |
| `min-idle` | 5 | 最小空闲连接数 |
| `max-active` | 20 | 最大活跃连接数 |
| `max-wait` | 60000 | 获取连接最大等待时间（毫秒） |
| `time-between-eviction-runs-millis` | 60000 | 检测间隔时间（毫秒） |
| `min-evictable-idle-time-millis` | 300000 | 连接最小空闲时间（毫秒） |
| `validation-query` | SELECT 1 | 验证连接的 SQL |
| `test-while-idle` | true | 空闲时检测连接有效性 |
| `test-on-borrow` | false | 借用时不检测（性能考虑） |
| `test-on-return` | false | 归还时不检测（性能考虑） |

## 3. Logback 配置

```xml
<!-- resources/logback.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- INFO 日志文件 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/plan-core-info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/plan-core-info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- ERROR 日志文件 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/plan-core-error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/plan-core-error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
    
    <!-- 根日志级别 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
    
    <!-- 项目包日志级别 -->
    <logger name="com.gxl.plancore" level="DEBUG"/>
    
    <!-- MyBatis SQL 日志 -->
    <logger name="com.gxl.plancore.**.mapper" level="DEBUG"/>
    
</configuration>
```

## 4. Configuration 类规范

### 4.1 Bean 配置类

```java
package com.gxl.plancore.common.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * 安全配置
 */
@Configuration
public class SecurityConfig {
    
    /**
     * 密码编码器
     * 使用 BCrypt 加密算法
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 4.2 配置属性类

```java
package com.gxl.plancore.common.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * JWT 配置属性
 */
@Component
@ConfigurationProperties(prefix = "app.jwt")
public class JwtProperties {
    
    private String secret;
    private long accessTokenExpireSeconds = 7200;
    private long refreshTokenExpireSeconds = 604800;
    
    // Getters and Setters
    public String getSecret() { return secret; }
    public void setSecret(String secret) { this.secret = secret; }
    
    public long getAccessTokenExpireSeconds() { return accessTokenExpireSeconds; }
    public void setAccessTokenExpireSeconds(long accessTokenExpireSeconds) { 
        this.accessTokenExpireSeconds = accessTokenExpireSeconds; 
    }
    
    public long getRefreshTokenExpireSeconds() { return refreshTokenExpireSeconds; }
    public void setRefreshTokenExpireSeconds(long refreshTokenExpireSeconds) { 
        this.refreshTokenExpireSeconds = refreshTokenExpireSeconds; 
    }
}
```

对应配置：

```properties
# application.properties
app.jwt.secret=your-secret-key
app.jwt.access-token-expire-seconds=7200
app.jwt.refresh-token-expire-seconds=604800
```

## 5. 多环境配置

### 5.1 配置文件命名

| 文件 | 用途 |
|------|------|
| `application.properties` | 通用配置 |
| `application-dev.properties` | 开发环境 |
| `application-test.properties` | 测试环境 |
| `application-prod.properties` | 生产环境 |

### 5.2 激活方式

```properties
# application.properties
spring.profiles.active=dev
```

或启动参数：

```bash
java -jar plan-core.jar --spring.profiles.active=prod
```

## 6. 敏感配置处理

### 6.1 环境变量方式

```properties
# application.properties
spring.datasource.password=${DB_PASSWORD:123456}
```

### 6.2 禁止事项

| 禁止行为 | 正确做法 |
|----------|----------|
| 生产密码写在配置文件中 | 使用环境变量或配置中心 |
| 提交 `application-prod.properties` 到 Git | 加入 `.gitignore` |
| 硬编码敏感信息 | 使用 `@Value` 或 `@ConfigurationProperties` |
