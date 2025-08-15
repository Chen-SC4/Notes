# MyBatis

## 1. 执行流程

### 加载 MyBatis 的配置

在这个阶段中，MyBatis 会读取配置文件或者使用配置类。配置中包含有数据源、事务管理、SQL 映射关系、注解信息等

### 构建 SqlSessionFactory

构建 SqlSessionFactory 是为了下一步创建 SqlSession 而服务的，Factory 会根据第一步中读取到的配置来创建不同类型的 SqlSession 对象

### 创建 SqlSession 对象

SqlSession 是 MyBatis 中用于执行 SQL 语句的核心接口。SqlSession 包含了执行 SQL 所需的所有方法，比如增删改查。

### 通过 Mapper 调用 SQL

用户通常会写一个 Mapper.class 或者 Mapper.xml 来执行 SQL 调用操作，此时 MyBatis 会通过动态代理技术，把 SQL 语句和 Mapper 进行关联。调用 Mapper 时，实际上是通过代理对象去执行 SQL 语句。

### 执行 SQL 语句

进行动态 SQL 处理，然后执行。

### 结果映射

执行完成之后，MyBatis 会把结果映射成为 Java 对象。

## 2. `#{}` 和 `${}` 的区别

在 MyBatis 中，#{} 和 ${} 是两种不同的参数占位符，它们在处理方式、安全性和使用场景上有重要区别。

### `#{}` 预编译占位符

在执行 SQL 之前，MyBatis 会将 #{} 替换为 `?`，然后将实际的参数值作为预编译语句进行参数设置。在数据库预编译了 SQL 语句之后，SQL 语句的结构已经确定，此时再进行 SQL 注入就无效了。

```java
// 原始 SQL
SELECT * FROM users WHERE id = #{id} AND name = #{name}

// 预编译后的 SQL
SELECT * FROM users WHERE id = ? AND name = ?

// 参数设置
preparedStatement.setLong(1, 123L);
preparedStatement.setString(2, "张三");
```

### `${}` 字符串直接替换

```java
// 原始 SQL
SELECT * FROM users WHERE name = '${name}' ORDER BY ${column}

// 直接字符串替换后的 SQL
SELECT * FROM users WHERE name = '张三' ORDER BY create_time

// 无预编译，直接执行最终 SQL
```

