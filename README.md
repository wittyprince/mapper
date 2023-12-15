![logo](logo.png)

# MyBatis Mapper

基于 **mybatis-mapper/provider**( [gitee](https://gitee.com/mybatis-mapper/provider)
| [GitHub](https://github.com/mybatis-mapper/provider) ) 实现的通用 Mapper。

项目文档： https://mapper.mybatis.io

[加入QQ群: 277256950](https://qm.qq.com/cgi-bin/qm/qr?k=ks1Y2WVNI1xjsqMlBzAkWmBMuYqDhVqZ&jump_from=webapi&authKey=U0CL8SDMS4wsJ8H9njVcNMbOW6Gb9AsxLfFB4jbxSKa0w0ChBX3wCms0+Cjzhekw)

## 支持 jakarta.persistence-api

添加依赖：
```xml
<dependency>
  <groupId>io.mybatis</groupId>
  <artifactId>mybatis-jakarta-jpa</artifactId>
  <version>2.2.1</version>
</dependency>
```

## 1. 快速入门

这是一个不需要任何配置就可以直接使用的通用 Mapper，通过简单的学习就可以直接在项目中使用。

### 1.1 主要目标

1. 开箱即用，无需任何配置，继承基类 Mapper 即可获得大量通用方法；
2. 随心所欲，通过复制粘贴的方式可以组建自己的基类 Mapper；
3. 全面贴心，提供 Service 层的封装方便业务使用和理解 Mapper；
4. 简单直观，提供 ActiveRecord 模式，结合 Spring Boot 自动配置直接上手用；
5. 自定义方法，简单几行代码即可实现自定义通用方法；
6. 轻松扩展，通过 Java SPI 轻松扩展。

### 1.2 系统要求

MyBatis Mapper 要求 MyBatis 最低版本为
3.5.1，推荐使用最新版本 <a href="https://maven-badges.herokuapp.com/maven-central/org.mybatis/mybatis"><img src="https://maven-badges.herokuapp.com/maven-central/org.mybatis/mybatis/badge.svg"/></a>
。

和 MyBatis 框架一样，最低需要 Java 8。

### 1.3 安装

<CodeGroup>
<CodeGroupItem title="Maven" active>

```xml
<dependencies>
  <dependency>
    <groupId>io.mybatis</groupId>
    <artifactId>mybatis-mapper</artifactId>
    <version>2.2.1</version>
  </dependency>
  <!-- 使用 Service 层封装时 -->
  <dependency>
    <groupId>io.mybatis</groupId>
    <artifactId>mybatis-service</artifactId>
    <version>2.2.1</version>
  </dependency>
  <!-- 使用 ActiveRecord 模式时 -->
  <dependency>
    <groupId>io.mybatis</groupId>
    <artifactId>mybatis-activerecord</artifactId>
    <version>2.2.1</version>
  </dependency>
</dependencies>
```

</CodeGroupItem>
<CodeGroupItem title="Gradle">

```groovy
dependencies {
  compile("io.mybatis:mybatis-mapper:2.2.1")
  // 使用 Service 层封装时
  compile("io.mybatis:mybatis-service:2.2.1")
  // 使用 ActiveRecord 模式时
  compile("io.mybatis:mybatis-activerecord:2.2.1")
}
```

</CodeGroupItem>
</CodeGroup>

### 1.4 快速设置

MyBatis Mapper 的基本原理是将实体类映射为数据库中的表和字段信息，因此实体类需要通过注解配置基本的元数据，配置好实体后， 只需要创建一个继承基础接口的 Mapper 接口就可以开始使用了。

#### 1.4.1 实体类配置

假设有一个表：

```sql
create table user
(
  id   INTEGER GENERATED BY DEFAULT AS IDENTITY (START WITH 1) PRIMARY KEY,
  name VARCHAR(32) DEFAULT 'DEFAULT',
  sex  VARCHAR(2)
);
```

对应的实体类：

```java
import io.mybatis.provider.Entity;

@Entity.Table("user")
public class User {
  @Entity.Column(id = true)
  private Long   id;
  @Entity.Column("name")
  private String username;
  @Entity.Column
  private String sex;

  //省略set和get方法
}
```

实体类上 **必须添加** `@Entity.Table` 注解指定实体类对应的表名，建议明确指定表名，不提供表名的时候，使用类名作为表名。 所有属于表中列的字段，**必须添加** `@Entity.Column`
注解，不指定列名时，使用字段名（不做任何转换），通过 `id=true` 可以标记字段为主键。

> `@Entity` 中包含的这两个注解提供了大量的配置属性，想要使用更多的配置，参考下面 **3. @Entity 注解** 的内容，
> 下面是一个简单示例：
>```java
>@Entity.Table(value = "sys_user", remark = "系统用户", autoResultMap = true)
>public class User {
>  @Entity.Column(id = true, remark = "主键", updatable = false, insertable = false)
>  private Long   id;
>  @Entity.Column(value = "name", remark = "帐号")
>  private String userName;
>  //省略其他
>}
>```

#### 1.4.2 Mapper接口定义

有了 `User` 实体后，直接创建一个继承了 `Mapper` 的接口即可：

```java
//io.mybatis.mapper.Mapper
public interface UserMapper extends Mapper<User, Long> {

}
```

这个接口只要被 MyBatis 扫描到即可直接使用。

> 下面是几种常见的扫描配置：
>
> 1. MyBatis 自带的配置文件方式 `mybatis-config.xml`：
> ```xml
>  <mappers>
>    <!-- 扫描指定的包 -->
>    <package name="com.example.mapper"/>
>  </mappers>
> ```
>
> 2. Spring 中的 `spring.xml` 配置：
> ```xml
> <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
>   <property name="basePackage" value="com.example.mapper"/>
>   <property name="markerInterface" value="io.mybatis.service.mapper.RoleMarker"/>
>   <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryRole"/>
> </bean>
> ```
>
> 3. Spring Boot 配置，启动类注解方式：
> ```java
> @MapperScan(basePackages = "com.example.mapper")
> @SpringBootApplication
> public class SpringBootDemoApplication {
>
>   public static void main(String[] args) {
>     SpringApplication.run(SpringBootDemoApplication.class, args);
>   }
>
> }
> ```
> Spring Boot 中，还可以直接给接口添加 `@org.apache.ibatis.annotations.Mapper` 注解，增加注解后可以省略 `@MapperScan` 配置。

#### 1.4.3 使用

定义好接口后，就可以获取 `UserMapper` 使用，下面是简单示例：

```java
User user=new User();
    user.setUserName("测试");
    userMapper.insert(user);
//保存后自增id回写，不为空
    Assert.assertNotNull(user.getId());
//根据id查询
    user=userMapper.selectByPrimaryKey(user.getId());
//删除
    Assert.assertEquals(1,userMapper.deleteByPrimaryKey(user.getId()));
```

看到这里，可以发现除了 MyBatis 自身的配置外，MyBatis Mapper 只需要配置实体类注解， 创建对应的 Mapper 接口就可以直接使用，没有任何繁琐的配置。

上面的示例只是简单的使用了 MyBatis Mapper，还有很多开箱即用的功能没有涉及， 建议在上述示例运行成功后，继续查看本项目其他模块的详细文档，熟悉各部分文档后， 在使用 MyBatis Mapper 时会更得心应手，随心所欲。

### 1.4.4 wrapper 用法

在 1.2.0 版本之后，针对 Example 封装了一个 ExampleWrapper，可以通过链式调用方便的使用 Example 方法。

```java
mapper.wrapper()
    .eq(User::getSex,"女")
    .or(c->c.gt(User::getId,40),c->c.lt(User::getId,10))
    .or()
    .startsWith(User::getUserName,"张").list();
```

对应的 SQL 如下：

```sql
SELECT id, name AS userName, sex
FROM user
WHERE (sex = ? AND ((id > ?) OR (id < ?)))
   OR (name LIKE ?)
```

详细的介绍可以查看 [1.2.0 更新日志](https://mapper.mybatis.io/releases/1.2.0.html)。

## 2. 示例项目

项目地址: https://github.com/mybatis-mapper/mybatis-mapper-example-springboot

项目目前包含 3 个分支，分别为：

- master 简单集成
- baseid 简单封装，所有表都使用名为 id，类型为 bigint 的自增主键
- shardingsphere 分库分表，支持分库分表的代码生成，每个表有不同的id

通过示例项目可以结合代码生成器自动生成大部分代码，可以用于测试和学习 mybatis-mapper 中的功能。
