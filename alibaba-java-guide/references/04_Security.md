# 第四章 安全规约

1.  **【强制】** 隶属于用户个人的页面或者功能必须进行权限控制校验。
2.  **【强制】** 用户敏感数据禁止直接展示，必须对展示数据进行脱敏。
    ```java
    // 正例
    // 手机号脱敏：139****1234
    public static String maskMobile(String mobile) {
        if (StringUtils.isBlank(mobile)) {
            return "";
        }
        return mobile.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
    }
    ```
3.  **【强制】** 用户输入的 SQL 参数严格使用参数绑定或者 METADATA 字段值限定，防止 SQL 注入，禁止字符串拼接 SQL 访问数据库。
    ```java
    // 正例：使用 MyBatis 的 #{}
    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);
    
    // 反例：使用 ${} 拼接
    @Select("SELECT * FROM user WHERE name = '${name}'")
    User findByName(@Param("name") String name);
    ```
4.  **【强制】** 用户请求传入的任何参数必须做有效性验证。
5.  **【强制】** 禁止向 HTML 页面输出未经安全过滤或未正确转义的用户数据。
6.  **【强制】** 表单、AJAX 提交必须执行 CSRF 安全验证。
