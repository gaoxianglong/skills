# 第六章 工程结构

## (一) 二方库依赖
1.  **【强制】** 定义 GAV 遵从以下规则：
    *   GroupID 格式：`com.{公司/BU}.业务线 [.子业务线]`。
    *   ArtifactID 格式：`产品线名-模块名`。
    *   Version：`主版本号.次版本号.修订号`。
    
    ```xml
    <!-- 示例 -->
    <dependency>
        <groupId>com.alibaba.tair</groupId>
        <artifactId>tair-client</artifactId>
        <version>1.0.0</version>
    </dependency>
    ```
2.  **【强制】** 二方库里可以定义枚举类型，参数可以使用枚举类型，但是接口返回值不允许使用枚举类型或者包含枚举类型的 POJO 对象。
3.  **【强制】** 依赖于一个二方库群时，必须定义一个统一的版本变量，避免版本不一致。

## (二) 服务器
1.  **【推荐】** 高并发服务器建议调小 TCP 协议的 time_wait 超时时间。
2.  **【推荐】** 调大服务器所支持的最大文件句柄数（File Descriptor，简写为 fd）。
3.  **【推荐】** 给 JVM 环境参数设置 -XX:+HeapDumpOnOutOfMemoryError 参数，让 JVM 碰到 OOM 场景时输出 dump 信息。
    ```bash
    # JVM 启动参数示例
    java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof -jar app.jar
    ```
