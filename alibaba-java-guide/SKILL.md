---
name: "alibaba-java-guide"
description: "执行阿里巴巴Java开发手册（嵩山版）规则。在编写、重构或审查Java代码时调用，以确保符合行业标准。"
---

# 阿里巴巴Java开发手册（嵩山版）指南

本 Skill 旨在帮助你编写符合《阿里巴巴Java开发手册（嵩山版）》规范的 Java 代码。为了方便查阅和维护，手册内容已拆分为以下模块：

## 1. 核心规约索引

### [第一章 编程规约](references/01_Programming.md)
涵盖命名风格、常量定义、代码格式、OOP 规约、日期时间、集合处理、并发处理、控制语句及注释规约。

### [第二章 异常日志](references/02_Exception_Log.md)
涵盖异常处理原则（如禁止捕获 RuntimeException、事务回滚）及日志输出规范（如 SLF4J 使用、禁止 System.out）。

### [第三章 单元测试](references/03_Unit_Test.md)
涵盖 AIR 原则（自动化、独立性、可重复）及 BCDE 测试编写原则。

### [第四章 安全规约](references/04_Security.md)
涵盖权限控制、敏感数据脱敏、SQL 注入防护、参数验证及 CSRF 防护。

### [第五章 MySQL 数据库](references/05_MySQL.md)
涵盖建表规约（字段命名、类型）、索引规约（唯一索引、Join 限制）、SQL 语句及 ORM 映射规范。

### [第六章 依赖管理](references/06_Dependency.md)
二方库依赖管理（GAV 规则）及服务器参数配置建议。

### [第七章 设计规约](references/07_Design.md)
涵盖设计文档沉淀、UML 图使用规范（用例图、状态图、时序图、类图、活动图）。

## 2. 原始资料
- [阿里巴巴Java开发手册（嵩山版）.pdf](references/AlibabaJavaGuide_Songshan.pdf)
