# 第五章 MySQL 数据库

## (一) 建表规约
1.  **【强制】** 表达是与否概念的字段，必须使用 `is_xxx` 的方式命名，数据类型是 `unsigned tinyint`（1表示是，0表示否）。
2.  **【强制】** 表名、字段名必须使用小写字母或数字，禁止出现数字开头，禁止两个下划线中间只出现数字。
3.  **【强制】** 表名不使用复数名词。
4.  **【强制】** 禁用保留字，如 `desc`, `range`, `match`, `delayed` 等。
5.  **【强制】** 主键索引名为 `pk_字段名`；唯一索引名为 `uk_字段名`；普通索引名则为 `idx_字段名`。
6.  **【强制】** 小数类型为 `decimal`，禁止使用 `float` 和 `double`。
7.  **【强制】** 表必备三字段：`id`, `create_time`, `update_time`。
    ```sql
    CREATE TABLE `user_info` (
        `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
        `user_name` varchar(32) NOT NULL COMMENT '用户名',
        `is_deleted` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '是否删除 1-是 0-否',
        `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
        `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
        PRIMARY KEY (`id`),
        UNIQUE KEY `uk_user_name` (`user_name`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
    ```

## (二) 索引规约
1.  **【强制】** 业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引。
2.  **【强制】** 超过三个表禁止 join。需要 join 的字段，数据类型必须绝对一致；多表关联查询时，保证被关联的字段需要有索引。
3.  **【强制】** 在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引。
4.  **【强制】** 页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎解决。
    ```sql
    -- 反例：索引失效
    SELECT * FROM user WHERE name LIKE '%Alibaba%';
    ```

## (三) SQL 语句
1.  **【强制】** 不要使用 `count(列名)` 或 `count(常量)` 来替代 `count(*)`，`count(*)` 是 SQL92 定义的标准统计行数的语法。
    ```sql
    -- 正例
    SELECT count(*) FROM user;
    ```
2.  **【强制】** `count(distinct col)` 计算该列除 NULL 之外的不重复行数。
3.  **【强制】** 当某一列的值全是 NULL 时，`count(col)` 的返回结果为 0，但 `sum(col)` 的返回结果为 NULL。
4.  **【强制】** 代码中写分页查询逻辑时，若 count 为 0 应直接返回，避免执行后面的分页语句。
5.  **【强制】** 禁止使用存储过程。

## (四) ORM 映射
1.  **【强制】** 在表查询中，一律不要使用 `*` 作为查询的字段列表，需要哪些字段必须明确写明。
2.  **【强制】** POJO 类的布尔属性不能加 `is`，而数据库字段必须加 `is_`，要求在 resultMap 中进行字段与属性之间的映射。
    ```xml
    <!-- MyBatis Mapper 示例 -->
    <resultMap id="BaseResultMap" type="com.alibaba.mmp.dob.UserDO">
        <result column="is_deleted" property="deleted" jdbcType="TINYINT" />
    </resultMap>
    ```
3.  **【强制】** 不要用 resultClass 当返回参数，即使所有类属性名与数据库字段一一对应，也需要定义；反过来，每一个表也必然有一个 POJO 类与之对应。
