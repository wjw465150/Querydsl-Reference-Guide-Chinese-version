# Querydsl Reference Guide(中文版)

翻译: <span style="font-size:1.5rem; background:yellow;">**白石**</span>

开源地址: https://github.com/wjw465150/Querydsl-Reference-Guide-Chinese-version

# 前言

Querydsl是一个框架，它支持构造静态类型的类似sql的查询。不需要将查询写成内联字符串或外部化到XML文件中，而是可以通过像Querydsl这样的流畅API构造查询。

例如，与简单的字符串相比，使用流畅的API的好处是

- IDE中的代码完成
- 几乎不允许语法上无效的查询
- 可以安全地引用域类型和属性
- 更好地对域类型的更改进行重构

# 1. 介绍

## 1.1. 背景

Querydsl诞生于以类型安全的方式维护HQL查询的需求。HQL查询的增量构造需要字符串连接，导致代码难以阅读。通过纯字符串对域类型和属性的不安全引用是基于String的HQL构造的另一个问题。

随着领域模型的不断变化，类型安全在软件开发中带来了巨大的好处。域的变化直接反映在查询中，查询构造中的自动补全使查询构造更加快速和安全。

HQL for Hibernate是Querydsl的第一个目标语言，但现在它支持JPA、JDO、JDBC、Lucene、Hibernate Search、MongoDB、Collections和RDFBean。

如果您是Java数据库访问的新手，https://www.marcobehler.com/guides/a-guide-to-accessing-databases-in-java 包含了对各个部分、选项的良好概述，并向您展示了QueryDSL适合的具体位置。

## 1.2. 规范

*Type safety(类型安全)*  是Querydsl的核心原则。查询是根据生成的查询类型构造的，这些查询类型反映域类型的属性。另外，函数/方法调用也是以完全类型安全的方式构造的。

*Consistency(一致性)* 是另一个重要原则。 查询路径和操作在所有实现中都是相同的，而且 Query 接口有一个共同的基本接口。

要了解 Querydsl 查询和表达式类型的表达能力，请访问 javadocs 并探索 `com.querydsl.core.Query`、`com.querydsl.core.Fetchable` 和 `com.querydsl.core.types.Expression `.

# 2. 教程

我们为 Querydsl 的主要后端提供集成指南，而不是一般的入门指南。

## 2.1. Querying JPA(查询JPA)

Querydsl 定义了一种通用的静态类型语法，用于在持久域模型数据之上进行查询。 JDO 和 JPA 是 Querydsl 的主要集成技术。 本指南介绍了如何将 Querydsl 与 JPA 结合使用。

Querydsl for JPA 是 JPQL 和 Criteria 查询的替代方案。 它以完全类型安全的方式将 Criteria 查询的动态特性与 JPQL 的表达能力以及所有这些结合起来。

### 2.1.1. Maven集成

将以下依赖项添加到您的 Maven 项目中：

```xml
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-jpa</artifactId>
  <version>${querydsl.version}</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.6.1</version>
</dependency>
```

现在，配置 Maven APT 插件：

```xml
<project>
  <build>
  <plugins>
    ...
    <plugin>
      <groupId>com.mysema.maven</groupId>
      <artifactId>apt-maven-plugin</artifactId>
      <version>1.1.3</version>
      <executions>
        <execution>
          <goals>
            <goal>process</goal>
          </goals>
          <configuration>
            <outputDirectory>target/generated-sources/java</outputDirectory>
            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
          </configuration>
        </execution>
      </executions>
    </plugin>
    ...
  </plugins>
  </build>
</project>
```

`JPAAnnotationProcessor` 查找使用 `javax.persistence.Entity` 注释的域类型并为它们生成查询类型。

如果您在域类型中使用 Hibernate 注释，则应改用 APT 处理器 `com.querydsl.apt.hibernate.HibernateAnnotationProcessor`。

运行`maven clean install`，您将得到生成的查询类型到 `target/generated-sources/java`目录下。

如果您使用 Eclipse，请运行 `mvn eclipse:eclipse` 来更新您的 Eclipse 项目以包含 `target/generated-sources/java` 作为源文件夹。

现在您可以构建 JPA 查询实例和查询域模型的实例。

### 2.1.2. Ant集成

将 full-deps 包中的 jar 文件放在类路径上，并使用以下任务生成 Querydsl 代码：

```xml
    <!-- APT based code generation -->
    <javac srcdir="${src}" classpathref="cp">
      <compilerarg value="-proc:only"/>
      <compilerarg value="-processor"/>
      <compilerarg value="com.querydsl.apt.jpa.JPAAnnotationProcessor"/>
      <compilerarg value="-s"/>
      <compilerarg value="${generated}"/>
    </javac>

    <!-- compilation -->
    <javac classpathref="cp" destdir="${build}">
      <src path="${src}"/>
      <src path="${generated}"/>
    </javac>
```

将 *src* 替换为您的主源文件夹，将 *generated* 替换为用于生成源的文件夹，将 *build* 替换为您的目标文件夹。

### 2.1.3. 在 Roo 中使用 Querydsl JPA

如果您将 Querydsl JPA 与 Spring Roo 一起使用，您可以将`com.querydsl.apt.jpa.JPAAnnotationProcessor`替换为`com.querydsl.apt.roo.RooAnnotationProcessor`，将处理`@RooJpaEntity`和`@RooJpaActiveRecord`注释的类，而不是`@Entity`注释的类。

基于 APT 的代码生成不适用于 AspectJ IDT。

### 2.1.4. 从hbm.xml文件生成模型

如果您将 Hibernate 与基于 XML 的配置一起使用，您可以使用 XML 元数据来创建您的 Querydsl 模型。

`com.querydsl.jpa.codegen.HibernateDomainExporter` 提供了以下功能：

```java
HibernateDomainExporter exporter = new HibernateDomainExporter(
  "Q",                     // name prefix
  new File("target/gen3"), // target folder
  configuration);          // instance of org.hibernate.cfg.Configuration

exporter.export();
```

HibernateDomainExporter 需要在域类型可见的类路径中执行，因为属性类型是通过反射解析的。

所有 JPA 注释都被忽略，但 Querydsl 注释如 `@QueryInit` 和 `@QueryType` 被考虑在内。

### 2.1.5. 使用查询类型

要使用Querydsl创建查询，需要实例化变量和Query实现。我们将从变量开始。

让我们假设你的项目有以下域类型:

```java
@Entity
public class Customer {
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setFirstName(String fn) {
        firstName = fn;
    }

    public void setLastName(String ln) {
        lastName = ln;
    }
}
```

Querydsl 将在与 Customer 相同的包中生成一个简单名称为 QCustomer 的查询类型。 QCustomer 可以用作 Querydsl 查询中的静态类型变量，作为 Customer 类型的代表。

QCustomer 有一个可以作为静态字段访问的默认实例变量：

```java
QCustomer customer = QCustomer.customer;
```

或者，您可以像这样定义自己的 Customer 变量：

```java
QCustomer customer = new QCustomer("myCustomer");
```

### 2.1.6. 查询

Querydsl JPA 模块支持 JPA 和 Hibernate API。

要使用 JPA API，您可以使用 `JPAQuery` 实例进行查询，如下所示：

```java
// where entityManager is a JPA EntityManager
JPAQuery<?> query = new JPAQuery<Void>(entityManager);
```

如果您使用的是 Hibernate API，则可以像这样实例化一个 `HibernateQuery`：

```java
// where session is a Hibernate session
HibernateQuery<?> query = new HibernateQuery<Void>(session);
```

`JPAQuery` 和 `HibernateQuery` 都实现了 `JPQLQuery` 接口。

对于本章的示例，查询是通过`JPAQueryFactory`实例创建的。 `JPAQueryFactory` 应该是获取 `JPAQuery` 实例的首选选项。

可以使用 Hibernate API `HibernateQueryFactory`

要检索名字为 Bob 的客户，您将构造如下查询：

```java
QCustomer customer = QCustomer.customer;
Customer bob = queryFactory.selectFrom(customer)
  .where(customer.firstName.eq("Bob"))
  .fetchOne();
```

selectFrom 调用定义了查询源和投影， where 部分定义了过滤器， fetchOne 告诉 Querydsl 返回单个元素。 很简单，对吧？

要使用多个源创建查询，您可以使用如下查询：

```java
QCustomer customer = QCustomer.customer;
QCompany company = QCompany.company;
query.from(customer, company);
```

要使用多重过滤就像这样

```java
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob"), customer.lastName.eq("Wilson"));
```

或者像这样

```java
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob").and(customer.lastName.eq("Wilson")));
```

在原生 JPQL 形式中，查询将这样编写：

```sql
select customer from Customer as customer
where customer.firstName = "Bob" and customer.lastName = "Wilson"
```

如果要通过“或”组合过滤器，请使用以下模式

```java
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob").or(customer.lastName.eq("Wilson")));
```

### 2.1.7. 使用连接

Querydsl 支持 JPQL 中的以下联接变体：内联接、联接、左联接和右联接。 联接使用是类型安全的，并遵循以下模式：

```java
QCat cat = QCat.cat;
QCat mate = new QCat("mate");
QCat kitten = new QCat("kitten");
queryFactory.selectFrom(cat)
    .innerJoin(cat.mate, mate)
    .leftJoin(cat.kittens, kitten)
    .fetch();
```

查询的native JPQL 版本将是

```sql
select cat from Cat as cat
inner join cat.mate as mate
left outer join cat.kittens as kitten
```

另一个例子

```java
queryFactory.selectFrom(cat)
    .leftJoin(cat.kittens, kitten)
    .on(kitten.bodyWeight.gt(10.0))
    .fetch();
```

相应的JPQL 版本将是

```sql
select cat from Cat as cat
left join cat.kittens as kitten
on kitten.bodyWeight > 10.0
```

### 2.1.8. 一般用法

像这样使用 JPQLQuery 接口的级联方法

**select:**  设置查询的投影。 （如果通过查询工厂创建则不需要）

**from:**  在此处添加查询源。

**innerJoin, join, leftJoin, rightJoin, on:**  使用这些构造添加连接元素。 对于连接方法，第一个参数是连接源，第二个参数是目标（别名）。

**where:** 添加查询过滤器，以逗号分隔的可变参数形式或通过 and 运算符级联。

**groupBy:** 以可变参数形式添加 group by 参数。

**having:** 添加具有“group by”分组的过滤器作为谓词表达式的 varags 数组。

**orderBy:** 将结果的排序添加为顺序表达式的可变参数数组。 在数字、字符串和其他可比较的表达式上使用 asc() 和 desc() 来访问 OrderSpecifier 实例。

**limit, offset, restrict:** 设置结果的分页。 最大结果的限制，跳过行的偏移量和在一次调用中定义两者的限制。

### 2.1.9. 排序

声明排序的语法是

```java
QCustomer customer = QCustomer.customer;
queryFactory.selectFrom(customer)
    .orderBy(customer.lastName.asc(), customer.firstName.desc())
    .fetch();
```

相当于下面的原生 JPQL

```sql
select customer from Customer as customer
order by customer.lastName asc, customer.firstName desc
```

### 2.1.10. 分组

可以按以下形式进行分组

```java
queryFactory.select(customer.lastName).from(customer)
    .groupBy(customer.lastName)
    .fetch();
```

相当于下面的原生 JPQL

```sql
select customer.lastName
from Customer as customer
group by customer.lastName
```

### 2.1.11. 删除子句

Querydsl JPA 中的删除子句遵循简单的 delete-where-execute 形式。 这里有些例子：

```java
QCustomer customer = QCustomer.customer;
// delete all customers
queryFactory.delete(customer).execute();
// delete all customers with a level less than 3
queryFactory.delete(customer).where(customer.level.lt(3)).execute();
```

where 调用是可选的，execute 调用执行删除并返回已删除实体的数量。

>  ==**💡提示:**== JPA中的DML子句没有考虑JPA级别的级联规则，也不提供细粒度的二级缓存交互。

### 2.1.12. 更新子句

Querydsl JPA 中的更新子句遵循简单的 update-set/where-execute 形式。 这里有些例子：

```java
QCustomer customer = QCustomer.customer;
// rename customers named Bob to Bobby
queryFactory.update(customer).where(customer.name.eq("Bob"))
    .set(customer.name, "Bobby")
    .execute();
```

set调用以sql -Update样式定义属性更新，而execute调用执行Update并返回更新的实体数量。

>  ==**💡提示:**== JPA中的DML子句没有考虑JPA级别的级联规则，也不提供细粒度的二级缓存交互。

### 2.1.13. 子查询

要创建子查询，您可以使用`JPAExpressions` 的静态工厂方法并通过 `from` 、`where`  等定义查询参数。

```java
QDepartment department = QDepartment.department;
QDepartment d = new QDepartment("d");
queryFactory.selectFrom(department)
    .where(department.size.eq(
        JPAExpressions.select(d.size.max()).from(d)))
     .fetch();
```

另一个例子

```java
QEmployee employee = QEmployee.employee;
QEmployee e = new QEmployee("e");
queryFactory.selectFrom(employee)
    .where(employee.weeklyhours.gt(
        JPAExpressions.select(e.weeklyhours.avg())
            .from(employee.department.employees, e)
            .where(e.manager.eq(employee.manager))))
    .fetch();
```

### 2.1.14. 暴露原始查询

如果你需要在执行查询之前调整原始查询，你可以像这样暴露它:

```java
Query jpaQuery = queryFactory.selectFrom(employee).createQuery();
// ...
List results = jpaQuery.getResultList();
```

### 2.1.15. 在JPA查询中使用Native SQL

Querydsl通过`JPASQLQuery`类在JPA中支持Native SQL。

To use it, you must generate Querydsl query types for your SQL schema. This can be done for example with the following Maven configuration:

```xml
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-maven-plugin</artifactId>
        <version>${querydsl.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>export</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <jdbcDriver>org.apache.derby.jdbc.EmbeddedDriver</jdbcDriver>
          <jdbcUrl>jdbc:derby:target/demoDB;create=true</jdbcUrl>
          <packageName>com.mycompany.mydomain</packageName>
          <targetFolder>${project.basedir}/target/generated-sources/java</targetFolder>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.apache.derby</groupId>
            <artifactId>derby</artifactId>
            <version>${derby.version}</version>
          </dependency>
        </dependencies>
      </plugin>
      ...
    </plugins>
  </build>
</project>
```

当查询类型成功生成到您选择的位置时，您可以在查询中使用它们。

单列查询:

```java
// serialization templates
SQLTemplates templates = new DerbyTemplates();
// query types (S* for SQL, Q* for domain types)
SAnimal cat = new SAnimal("cat");
SAnimal mate = new SAnimal("mate");
QCat catEntity = QCat.cat;

JPASQLQuery<?> query = new JPASQLQuery<Void>(entityManager, templates);
List<String> names = query.select(cat.name).from(cat).fetch();
```

如果您在查询中混合使用实体（例如 QCat）和表（例如 SAnimal）引用，您需要确保它们使用相同的变量名称。 SAnimal.animal 具有变量名称“animal”，因此使用了一个新实例 (new SAnimal("cat"))。

另一种模式可能是

```java
QCat catEntity = QCat.cat;
SAnimal cat = new SAnimal(catEntity.getMetadata().getName());
```

查询多列：

```java
query = new JPASQLQuery<Void>(entityManager, templates);
List<Tuple> rows = query.select(cat.id, cat.name).from(cat).fetch();
```

查询所有列：

```java
List<Tuple> rows = query.select(cat.all()).from(cat).fetch();
 
```

在 SQL 中查询，但投影为实体：

```java
query = new JPASQLQuery<Void>(entityManager, templates);
List<Cat> cats = query.select(catEntity).from(cat).orderBy(cat.name.asc()).fetch();
```

使用连接查询：

```java
query = new JPASQLQuery<Void>(entityManager, templates);
cats = query.select(catEntity).from(cat)
    .innerJoin(mate).on(cat.mateId.eq(mate.id))
    .where(cat.dtype.eq("Cat"), mate.dtype.eq("Cat"))
    .fetch();
```

查询并投影到 DTO：

```java
query = new JPASQLQuery<Void>(entityManager, templates);
List<CatDTO> catDTOs = query.select(Projections.constructor(CatDTO.class, cat.id, cat.name))
    .from(cat)
    .orderBy(cat.name.asc())
    .fetch();
```

如果您使用的是 Hibernate API 而不是 JPA API，那么请使用`HibernateSQLQuery`。

## 2.2. Querying JDO

Querydsl defines a general statically typed syntax for querying on top of persisted domain model data. JDO and JPA are the primary integration technologies for Querydsl. This guide describes how to use Querydsl in combination with JDO.

### 2.2.1. Maven integration

Add the following dependencies to your Maven project:

```
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-jdo</artifactId>
  <version>${querydsl.version}</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.6.1</version>
</dependency>
```

And now, configure the Maven APT plugin which generates the query types used by Querydsl:

```
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
        <version>1.1.3</version>
        <executions>
          <execution>
            <goals>
              <goal>process</goal>
            </goals>
            <configuration>
              <outputDirectory>target/generated-sources/java</outputDirectory>
              <processor>com.querydsl.apt.jdo.JDOAnnotationProcessor</processor>
            </configuration>
          </execution>
        </executions>
      </plugin>
    ...
    </plugins>
  </build>
</project>
```

The JDOAnnotationProcessor finds domain types annotated with the `javax.jdo.annotations.PersistenceCapable` annotation and generates query types for them.

Run clean install and you will get your query types generated into target/generated-sources/java.

If you use Eclipse, run mvn eclipse:eclipse to update your Eclipse project to include target/generated-sources/java as a source folder.

Now you are able to construct JDO query instances and instances of the query domain model.

### 2.2.2. Ant integration

Place the jar files from the full-deps bundle on your classpath and use the following tasks for Querydsl code generation:

```
    <!-- APT based code generation -->
    <javac srcdir="${src}" classpathref="cp">
      <compilerarg value="-proc:only"/>
      <compilerarg value="-processor"/>
      <compilerarg value="com.querydsl.apt.jdo.JDOAnnotationProcessor"/>
      <compilerarg value="-s"/>
      <compilerarg value="${generated}"/>
    </javac>

    <!-- compilation -->
    <javac classpathref="cp" destdir="${build}">
      <src path="${src}"/>
      <src path="${generated}"/>
    </javac>
```

Replace *src* with your main source folder, *generated* with your folder for generated sources and *build* with your target folder.

### 2.2.3. Using query types

To create queries with Querydsl you need to instantiate variables and Query implementations. We will start with the variables.

Let's assume that your project has the following domain type:

```
@PersistenceCapable
public class Customer {
  private String firstName;
  private String lastName;

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public void setFirstName(String fn) {
    firstName = fn;
  }

  public void setLastName(String ln) {
    lastName = ln;
  }
}
```

Querydsl will generate a query type with the simple name QCustomer into the same package as Customer. QCustomer can be used as a statically typed variable in Querydsl as a representative for the Customer type.

QCustomer has a default instance variable which can be accessed as a static field:

```
QCustomer customer = QCustomer.customer;
```

Alternatively you can define your own Customer variables like this:

```
QCustomer customer = new QCustomer("myCustomer");
```

QCustomer reflects all the properties of the original type Customer as public fields. The firstName field can be accessed like this

```
customer.firstName;
```

### 2.2.4. Querying with JDO

For the JDO-module `JDOQuery` is the main Query implementation. It is instantiated like this:

```
PersistenceManager pm = ...;
JDOQuery<?> query = new JDOQuery<Void>(pm);
```

For the examples of this chapter the queries are created via a `JDOQueryFactory` instance. `JDOQueryFactory` should be the preferred option to obtain `JDOQuery` instances.

To retrieve the customer with the first name Bob you would construct a query like this:

```
QCustomer customer = QCustomer.customer;
Customer bob = queryFactory.selectFrom(customer)
                   .where(customer.firstName.eq("Bob"))
                   .fetchOne();
```

The selectFrom call defines the query source and projection, the where part defines the filter and fetchOne tells Querydsl to return a single element. Easy, right?

Alternatively you can express it also like this

```
QCustomer customer = QCustomer.customer;
Customer bob = queryFactory.select(customer).from(customer)
                   .where(customer.firstName.eq("Bob"))
                   .fetchOne();
```

To create a query with multiple sources you just use the JDOQuery class like this:

```
QCustomer customer = QCustomer.customer;
QCompany company = QCompany.company;
query.from(customer, company);
```

And to use multiple filters use it like this

```
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob"), customer.lastName.eq("Wilson"));
```

Or like this

```
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob").and(customer.lastName.eq("Wilson")));
```

If you want to combine the filters via "or" then use the following pattern

```
queryFactory.selectFrom(customer)
    .where(customer.firstName.eq("Bob").or(customer.lastName.eq("Wilson")));
```

### 2.2.5. General usage

Use the the cascading methods of the JDOQuery class like this

*select:* Set the projection of the query. (Not necessary if created via query factory)

*from:* Add query sources here, the first argument becomes the main source and the others are treated as variables.

*where:* Add query filters, either in varargs form separated via commas or cascaded via the and-operator.

*groupBy:* Add group by arguments in varargs form.

*having:* Add having filters of the "group by" grouping as an varargs array of Predicate expressions.

*orderBy:* Add ordering of the result as an varargs array of order expressions. Use asc() and desc() on numeric, string and other comparable expression to access the OrderSpecifier instances.

*limit, offset, restrict:* Set the paging of the result. Limit for max results, offset for skipping rows and restrict for defining both in one call.

### 2.2.6. Ordering

The syntax for declaring ordering is

```
QCustomer customer = QCustomer.customer;
queryFactory.selectFrom(customer)
    .orderBy(customer.lastName.asc(), customer.firstName.desc())
    .fetch();
```

### 2.2.7. Grouping

Grouping can be done in the following form

```
queryFactory.select(customer.lastName).from(customer)
    .groupBy(customer.lastName)
    .fetch();
```

### 2.2.8. Delete clauses

Delete clauses in Querydsl JDO follow a simple delete-where-execute form. Here are some examples:

```
QCustomer customer = QCustomer.customer;
// delete all customers
queryFactory.delete(customer).execute();
// delete all customers with a level less than 3
queryFactory.delete(customer).where(customer.level.lt(3)).execute();
```

The second parameter of the JDODeleteClause constructor is the entity to be deleted. The where call is optional and the execute call performs the deletion and returns the amount of deleted entities.

### 2.2.9. Subqueries

To create a subquery you can use one of the factory methods of `JDOExpressions` and add the query parameters via from, where etc.

```
QDepartment department = QDepartment.department;
QDepartment d = new QDepartment("d");
queryFactory.selectFrom(department)
    .where(department.size.eq(JDOExpressions.select(d.size.max()).from(d))
    .fetch();
```

represents the following native JDO query

```
SELECT this FROM com.querydsl.jdo.models.company.Department
WHERE this.size ==
(SELECT max(d.size) FROM com.querydsl.jdo.models.company.Department d)
    
```

Another example

```
QEmployee employee = QEmployee.employee;
QEmployee e = new QEmployee("e");
queryFactory.selectFrom(employee)
    .where(employee.weeklyhours.gt(
        JDOExpressions.select(e.weeklyhours.avg())
                      .from(employee.department.employees, e)
                      .where(e.manager.eq(employee.manager)))
    .fetch();
```

which represents the following native JDO query

```
SELECT this FROM com.querydsl.jdo.models.company.Employee
WHERE this.weeklyhours >
(SELECT avg(e.weeklyhours) FROM this.department.employees e WHERE e.manager == this.manager)
    
```

### 2.2.10. Using Native SQL

Querydsl supports Native SQL in JDO via the `JDOSQLQuery` class.

To use it, you must generate Querydsl query types for your SQL schema. This can be done for example with the following Maven configuration:

```
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-maven-plugin</artifactId>
        <version>${querydsl.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>export</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <jdbcDriver>org.apache.derby.jdbc.EmbeddedDriver</jdbcDriver>
          <jdbcUrl>jdbc:derby:target/demoDB;create=true</jdbcUrl>
          <packageName>com.mycompany.mydomain</packageName>
          <targetFolder>${project.basedir}/target/generated-sources/java</targetFolder>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.apache.derby</groupId>
            <artifactId>derby</artifactId>
            <version>${derby.version}</version>
          </dependency>
        </dependencies>
      </plugin>
      ...
    </plugins>
  </build>
</project>
```

When the query types have successfully been generated into the location of your choice, you can use them in your queries.

Single column query:

```
// serialization templates
SQLTemplates templates = new DerbyTemplates();
// query types (S* for SQL, Q* for domain types)
SAnimal cat = new SAnimal("cat");
SAnimal mate = new SAnimal("mate");

JDOSQLQuery<?> query = new JDOSQLQuery<Void>(pm, templates);
List<String> names = query.select(cat.name).from(cat).fetch();
```

Query multiple columns:

```
query = new JDOSQLQuery<Void>(pm, templates);
List<Tuple> rows = query.select(cat.id, cat.name).from(cat).fetch();
```

Query all columns:

```
List<Tuple> rows = query.select(cat.all()).from(cat).fetch();
 
```

Query with joins:

```
query = new JDOSQLQuery<Void>(pm, templates);
cats = query.select(catEntity).from(cat)
    .innerJoin(mate).on(cat.mateId.eq(mate.id))
    .where(cat.dtype.eq("Cat"), mate.dtype.eq("Cat"))
    .fetch();
```

Query and project into DTO:

```
query = new JDOSQLQuery<Void>(pm, templates);
List<CatDTO> catDTOs = query.select(Projections.constructor(CatDTO.class, cat.id, cat.name))
    .from(cat)
    .orderBy(cat.name.asc())
    .fetch();
```

## 2.3. Querying SQL

本章介绍 SQL 模块的查询类型生成和查询功能。

### 2.3.1. Maven集成

将以下依赖项添加到您的 Maven 项目中：

```xml
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-sql</artifactId>
  <version>${querydsl.version}</version>
</dependency>

<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-sql-codegen</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.6.1</version>
</dependency>
```

如果通过Maven生成代码，可以跳过querydsl-sql-codegen依赖项。

### 2.3.2. 通过Maven生成代码

这个功能应该主要通过Maven插件来使用。下面是一个例子:

```xml
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-maven-plugin</artifactId>
        <version>${querydsl.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>export</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <jdbcDriver>org.apache.derby.jdbc.EmbeddedDriver</jdbcDriver>
          <jdbcUrl>jdbc:derby:target/demoDB;create=true</jdbcUrl>
          <packageName>com.myproject.domain</packageName>
          <targetFolder>${project.basedir}/target/generated-sources/java</targetFolder>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.apache.derby</groupId>
            <artifactId>derby</artifactId>
            <version>${derby.version}</version>
          </dependency>
        </dependencies>
      </plugin>
      ...
    </plugins>
  </build>
</project>
```

使用目标*test-export*将目标文件夹视为测试源代码文件夹，以便与测试代码一起使用。



**Table 2.1. Parameters**

| Name                     | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| jdbcDriver               | class name of the JDBC driver                                |
| jdbcUrl                  | JDBC url                                                     |
| jdbcUser                 | JDBC user                                                    |
| jdbcPassword             | JDBC password                                                |
| namePrefix               | name prefix for generated query classes (default: Q)         |
| nameSuffix               | name suffix for generated query classes (default: )          |
| beanPrefix               | name prefix for generated bean classes                       |
| beanSuffix               | name suffix for generated bean classes                       |
| packageName              | package name where source files should be generated          |
| beanPackageName          | package name where bean files should be generated, (default: packageName) |
| beanInterfaces           | array of interface classnames to add to the bean classes (default: empty) |
| beanAddToString          | set to true to create a default toString() implementation (default: false) |
| beanAddFullConstructor   | set to true to create a full constructor in addition to public empty (default: false) |
| beanPrintSupertype       | set to true to print the supertype as well (default: false)  |
| schemaPattern            | a schema name pattern in LIKE pattern form; must match the schema name as it is stored in the database, multiple can be separated by comma (default: null) |
| tableNamePattern         | a table name pattern in LIKE pattern form; must match the table name as it is stored in the database, multiple can be separated by comma (default: null) |
| targetFolder             | target folder where sources should be generated              |
| beansTargetFolder        | target folder where bean sources should be generated, defaults to the same value as targetFolder |
| namingStrategyClass      | class name of the NamingStrategy class (default: DefaultNamingStrategy) |
| beanSerializerClass      | class name of the BeanSerializer class (default: BeanSerializer) |
| serializerClass          | class name of the Serializer class (default: MetaDataSerializer) |
| exportBeans              | set to true to generate beans as well, see section 2.14.13 (default: false) |
| innerClassesForKeys      | set to true to generate inner classes for keys (default: false) |
| validationAnnotations    | set to true to enable serialization of validation annotations (default: false) |
| columnAnnotations        | export column annotations (default: false)                   |
| createScalaSources       | whether to export Scala sources instead of Java sources, (default: false) |
| schemaToPackage          | append schema name to package (default: false)               |
| lowerCase                | lower case transformation of names (default: false)          |
| exportTables             | export tables (default: true)                                |
| exportViews              | export views (default: true)                                 |
| exportPrimaryKeys        | export primary keys (default: true)                          |
| tableTypesToExport       | Comma-separated list of table types to export (allowable values will depend on JDBC driver). Allows for arbitrary set of types to be exported, e.g.: "TABLE, MATERIALIZED VIEW". The exportTables and exportViews parameters will be ignored if this parameter is set. (default: none) |
| exportForeignKeys        | export foreign keys (default: true)                          |
| exportDirectForeignKeys  | export direct foreign keys (default: true)                   |
| exportInverseForeignKeys | export inverse foreign keys (default: true)                  |
| customTypes              | Custom user types (default: none)                            |
| typeMappings             | Mappings of table.column to Java type (default: none)        |
| numericMappings          | Mappings of size/digits to Java type (default: none)         |
| imports                  | Array of java imports added to generated query classes: *com.bar* for package (without .* notation), *com.bar.Foo* for class (default: empty) |



Custom types can be used to register additional Type implementations:

```
<customTypes>
  <customType>com.querydsl.sql.types.InputStreamType</customType>
</customTypes>
```

Type mappings can be used to register table.column specific java types:

```
<typeMappings>
  <typeMapping>
    <table>IMAGE</table>
    <column>CONTENTS</column>
    <type>java.io.InputStream</type>
  </typeMapping>
</typeMappings>
```

The defaults for the numeric mappings are



**Table 2.2. Numeric mappings**

| Total digits | Decimal digits | Type       |
| ------------ | -------------- | ---------- |
| > 18         | 0              | BigInteger |
| > 9          | 0              | Long       |
| > 4          | 0              | Integer    |
| > 2          | 0              | Short      |
| > 0          | 0              | Byte       |
| > 0          | > 0            | BigDecimal |



They can be customized for specific total/decimal digits combinations like this:

```
<numericMappings>
  <numericMapping>
    <total>1</total>
    <decimal>0</decimal>
    <javaType>java.lang.Byte</javaType>
  </numericMapping>
</numericMappings>
```

Imports can be used to add cross-schema foreign keys support.

Schemas, tables and columns can also be renamed using the plugin. Here are some examples:

Renaming a schema:

```
<renameMappings>
  <renameMapping>
    <fromSchema>PROD</fromSchema>
    <toSchema>TEST</toSchema>
  </renameMapping>
</renameMappings>
```

Renaming a table:

```
<renameMappings>
  <renameMapping>
    <fromSchema>PROD</fromSchema>
    <fromTable>CUSTOMER</fromTable>
    <toTable>CSTMR</toTable>
  </renameMapping>
</renameMappings>
```

Renaming a column:

```
<renameMappings>
  <renameMapping>
    <fromSchema>PROD</fromSchema>
    <fromTable>CUSTOMER</fromTable>
    <fromColumn>ID</fromColumn>
    <toColumn>IDX</toTable>
  </renameMapping>
</renameMappings>
```

Note: fromSchema can be omitted when renaming tables and columns.

Compared to APT based code generation certain functionality is not available such as QueryDelegate annotation handling.

### 2.3.3. Code generation via ANT

The ANT task `com.querydsl.sql.codegen.ant.AntMetaDataExporter` of the querydsl-sql module provides the same functionality as an ANT task. The configuration parameters of the task are the same as for the Maven plugin, except for the composite types.

The composite types are used without the wrapper element like in this example.

```
<project name="testproject" default="codegen" basedir=".">

  <taskdef name="codegen" classname="com.querydsl.sql.codegen.ant.AntMetaDataExporter"/>

  <target name="codegen">
    <codegen
      jdbcDriver="org.h2.Driver"
      jdbcUser="sa"
      jdbcUrl="jdbc:h2:/dbs/db1"
      packageName="test"
      targetFolder="target/generated-sources/java">
      <renameMapping fromSchema="PUBLIC" toSchema="PUB"/>
    </codegen>
  </target>
</project>
```

### 2.3.4. Creating the query types

To get started export your schema into Querydsl query types like this:

```
java.sql.Connection conn = ...;
MetaDataExporter exporter = new MetaDataExporter();
exporter.setPackageName("com.myproject.mydomain");
exporter.setTargetFolder(new File("target/generated-sources/java"));
exporter.export(conn.getMetaData());
```

This declares that the database schema is to be mirrored into the com.myproject.domain package in the target/generated-sources/java folder.

The generated types have the table name transformed to mixed case as the class name and a similar mixed case transformation applied to the columns which are available as property paths in the query type.

In addition to this primary key and foreign key constraints are provided as fields which can be used for compact join declarations.

### 2.3.5. Configuration

The configuration is done via the com.querydsl.sql.Configuration class which takes the Querydsl SQL dialect as an argument. For H2 you would create it like this

```
SQLTemplates templates = new H2Templates();
Configuration configuration = new Configuration(templates);
```

Querydsl uses SQL dialects to customize the SQL serialization needed for different relational databases. The available dialects are:

- CUBRIDTemplates (tested with CUBRID 8.4)
- DB2Templates (tested with DB2 10.1.2)
- DerbyTemplates (tested with Derby 10.8.2.2)
- FirebirdTemplates (tested with Firebird 2.5)
- HSQLDBTemplates (tested with HSQLDB 2.2.4)
- H2Templates (tested with H2 1.3.164)
- MySQLTemplates (tested with MySQL 5.5)
- OracleTemplates (test with Oracle 10 and 11)
- PostgreSQLTemplates (tested with PostgreSQL 9.1)
- SQLiteTemplates (tested with xerial JDBC 3.7.2)
- SQLServerTemplates (tested with SQL Server)
- SQLServer2005Templates (for SQL Server 2005)
- SQLServer2008Templates (for SQL Server 2008)
- SQLServer2012Templates (for SQL Server 2012 and later)
- TeradataTemplates (tested with Teradata 14)

For customized SQLTemplates instances you can use the builder pattern like this

```
  H2Templates.builder()
     .printSchema() // to include the schema in the output
     .quote()       // to quote names
     .newLineToSingleSpace() // to replace new lines with single space in the output
     .escape(ch)    // to set the escape char
     .build();      // to get the customized SQLTemplates instance
```

The methods of the Configuration class can be used to enable direct serialization of literals via setUseLiterals(true), override schema and tables and register custom types. For full details look at the javadocs of Configuration.

### 2.3.6. Querying

For the following examples we will be using the `SQLQueryFactory` class for query creation. Using it results in more concise code compared to constructor based query creation.

```
SQLQueryFactory queryFactory = new SQLQueryFactory(configuration, dataSource);
```

Querying with Querydsl SQL is as simple as this:

```
QCustomer customer = new QCustomer("c");

List<String> lastNames = queryFactory.select(customer.lastName).from(customer)
    .where(customer.firstName.eq("Bob"))
    .fetch();
```

which is transformed into the following sql query, assuming that the related table name is *customer* and the columns *first_name* and *last_name*:

```
SELECT c.last_name
FROM customer c
WHERE c.first_name = 'Bob'
```

### 2.3.7. General usage

Use the the cascading methods of the SQLQuery class like this

*select:* Set the projection of the query. (Not necessary if created via query factory)

*from:* Add the query sources here.

*innerJoin, join, leftJoin, rightJoin, fullJoin, on:* Add join elements using these constructs. For the join methods the first argument is the join source and the second the target (alias).

*where:* Add query filters, either in varargs form separated via commas or cascaded via the and-operator.

*groupBy:* Add group by arguments in varargs form.

*having:* Add having filter of the "group by" grouping as an varags array of Predicate expressions.

*orderBy:* Add ordering of the result as an varargs array of order expressions. Use asc() and desc() on numeric, string and other comparable expression to access the OrderSpecifier instances.

*limit, offset, restrict:* Set the paging of the result. Limit for max results, offset for skipping rows and restrict for defining both in one call.

### 2.3.8. Joins

Joins are constructed using the following syntax:

```
QCustomer customer = QCustomer.customer;
QCompany company = QCompany.company;
queryFactory.select(customer.firstName, customer.lastName, company.name)
    .from(customer)
    .innerJoin(customer.company, company)
    .fetch();
```

and for a left join:

```
queryFactory.select(customer.firstName, customer.lastName, company.name)
    .from(customer)
    .leftJoin(customer.company, company)
    .fetch();
```

Alternatively the join condition can also be written out:

```
queryFactory.select(customer.firstName, customer.lastName, company.name)
    .from(customer)
    .leftJoin(company).on(customer.company.eq(company))
    .fetch();
```

### 2.3.9. Ordering

The syntax for declaring ordering is

```
queryFactory.select(customer.firstName, customer.lastName)
    .from(customer)
    .orderBy(customer.lastName.asc(), customer.firstName.asc())
    .fetch();
```

which is equivalent to the following native SQL

```
SELECT c.first_name, c.last_name
FROM customer c
ORDER BY c.last_name ASC, c.first_name ASC
```

### 2.3.10. Grouping

Grouping can be done in the following form

```
queryFactory.select(customer.lastName)
    .from(customer)
    .groupBy(customer.lastName)
    .fetch();
```

which is equivalent to the following native SQL

```
SELECT c.last_name
FROM customer c
GROUP BY c.last_name
```

### 2.3.11. Using Subqueries

To create a subquery you can use one of the factory methods of `SQLExpressions` and add the query parameters via from, where etc.

```
QCustomer customer = QCustomer.customer;
QCustomer customer2 = new QCustomer("customer2");
queryFactory.select(customer.all())
    .from(customer)
    .where(customer.status.eq(
        SQLExpressions.select(customer2.status.max()).from(customer2)))
    .fetch()
```

Another example

```
QStatus status = QStatus.status;
queryFactory.select(customer.all())
    .from(customer)
    .where(customer.status.in(
        SQLExpressions.select(status.id).from(status).where(status.level.lt(3))))
    .fetch();
```

### 2.3.12. Selecting literals

To select literals you need to create constant instances for them like this:

```
queryFactory.select(Expressions.constant(1),
                    Expressions.constant("abc"));
```

The class `com.querydsl.core.types.dsl.Expressions` offers also other useful static methods for projections, operation and template creation.

### 2.3.13. Query extension support

Custom query extensions to support engine specific syntax can be created by subclassing AbstractSQLQuery and adding flagging methods like in the given MySQLQuery example:

```
public class MySQLQuery<T> extends AbstractSQLQuery<T, MySQLQuery<T>> {

    public MySQLQuery(Connection conn) {
        this(conn, new MySQLTemplates(), new DefaultQueryMetadata());
    }

    public MySQLQuery(Connection conn, SQLTemplates templates) {
        this(conn, templates, new DefaultQueryMetadata());
    }

    protected MySQLQuery(Connection conn, SQLTemplates templates, QueryMetadata metadata) {
        super(conn, new Configuration(templates), metadata);
    }

    public MySQLQuery bigResult() {
        return addFlag(Position.AFTER_SELECT, "SQL_BIG_RESULT ");
    }

    public MySQLQuery bufferResult() {
        return addFlag(Position.AFTER_SELECT, "SQL_BUFFER_RESULT ");
    }


    // ...
}
```

The flags are custom SQL snippets that can be inserted at specific points in the serialization. The supported positions are the enums of the `com.querydsl.core.QueryFlag.Position` enum class.

### 2.3.14. Window functions

Window functions are supported in Querydsl via the methods in the `SQLExpressions` class.

Usage example:

```
queryFactory.select(SQLExpressions.rowNumber()
        .over()
        .partitionBy(employee.name)
        .orderBy(employee.id))
     .from(employee)
```

### 2.3.15. Common table expressions

Common table expressions are supported in Querydsl SQL via two syntax variants

```
QEmployee employee = QEmployee.employee;
queryFactory.with(employee, SQLExpressions.select(employee.all)
                                          .from(employee)
                                          .where(employee.name.startsWith("A")))
            .from(...)
```

And using a column listing

```
QEmployee employee = QEmployee.employee;
queryFactory.with(employee, employee.id, employee.name)
            .as(SQLExpressions.select(employee.id, employee.name)
                              .from(employee)
                              .where(employee.name.startsWith("A")))
            .from(...)
```

If the columns of the common table expression are a subset of an existing table or view it is advisable to use a generated path type for it, e.g. QEmployee in this case, but if the columns don't fit any existing table PathBuilder can be used instead.

Below is an example for such a case

```
QEmployee employee = QEmployee.employee;
QDepartment department = QDepartment.department;
PathBuilder<Tuple> emp = new PathBuilder<Tuple>(Tuple.class, "emp");
queryFactory.with(emp, SQLExpressions.select(employee.id, employee.name, employee.departmentId,
                                          department.name.as("departmentName"))
                                      .from(employee)
                                      .innerJoin(department).on(employee.departmentId.eq(department.id))))
            .from(...)
```

### 2.3.16. Other SQL expressions

Other SQL expressions are also available from the `SQLExpressions` class as static methods.

### 2.3.17. Using Data manipulation commands

#### 2.3.17.1. Insert

With columns

```
QSurvey survey = QSurvey.survey;

queryFactory.insert(survey)
    .columns(survey.id, survey.name)
    .values(3, "Hello").execute();
```

Without columns

```
queryFactory.insert(survey)
    .values(4, "Hello").execute();
```

With subquery

```
queryFactory.insert(survey)
    .columns(survey.id, survey.name)
    .select(SQLExpressions.select(survey2.id.add(1), survey2.name).from(survey2))
    .execute();
```

With subquery, without columns

```
queryFactory.insert(survey)
    .select(SQLExpressions.select(survey2.id.add(10), survey2.name).from(survey2))
    .execute();
```

As an alternative to the columns/values usage, Querydsl provides also a set method which can be used like this

```
QSurvey survey = QSurvey.survey;

queryFactory.insert(survey)
    .set(survey.id, 3)
    .set(survey.name, "Hello").execute();
```

which is equivalent to the first example. Usage of the set method always expands internally to columns and values.

Beware that

```
columns(...).select(...)
```

maps the result set of the given query to be inserted whereas

To get the created keys out instead of modified rows count use one of the executeWithKey/s method.

```
set(...)
```

maps single columns and nulls are used for empty subquery results.

To populate a clause instance based on the contents of a bean you can use

```
queryFactory.insert(survey)
    .populate(surveyBean).execute();
```

This will exclude null bindings, if you need also null bindings use

```
queryFactory.insert(survey)
    .populate(surveyBean, DefaultMapper.WITH_NULL_BINDINGS).execute();
```

#### 2.3.17.2. Update

With where

```
QSurvey survey = QSurvey.survey;

queryFactory.update(survey)
    .where(survey.name.eq("XXX"))
    .set(survey.name, "S")
    .execute();
```

Without where

```
queryFactory.update(survey)
    .set(survey.name, "S")
    .execute();
```

Using bean population

```
queryFactory.update(survey)
    .populate(surveyBean)
    .execute();
```

#### 2.3.17.3. Delete

With where

```
QSurvey survey = QSurvey.survey;

queryFactory.delete(survey)
    .where(survey.name.eq("XXX"))
    .execute();
```

Without where

```
queryFactory.delete(survey)
    .execute()
```

### 2.3.18. Batch support in DML clauses

Querydsl SQL supports usage of JDBC batch updates through the DML APIs. If you have consecutive DML calls with a similar structure, you can bundle the the calls via addBatch() usage into one DMLClause. See the examples how it works for UPDATE, DELETE and INSERT.

Update:

```
QSurvey survey = QSurvey.survey;

queryFactory.insert(survey).values(2, "A").execute();
queryFactory.insert(survey).values(3, "B").execute();

SQLUpdateClause update = queryFactory.update(survey);
update.set(survey.name, "AA").where(survey.name.eq("A")).addBatch();
update.set(survey.name, "BB").where(survey.name.eq("B")).addBatch();
```

Delete:

```
queryFactory.insert(survey).values(2, "A").execute();
queryFactory.insert(survey).values(3, "B").execute();

SQLDeleteClause delete = queryFactory.delete(survey);
delete.where(survey.name.eq("A")).addBatch();
delete.where(survey.name.eq("B")).addBatch();
assertEquals(2, delete.execute());
```

Insert:

```
SQLInsertClause insert = queryFactory.insert(survey);
insert.set(survey.id, 5).set(survey.name, "5").addBatch();
insert.set(survey.id, 6).set(survey.name, "6").addBatch();
assertEquals(2, insert.execute());
```

### 2.3.19. Bean class generation

To create JavaBean DTO types for the tables of your schema use the MetaDataExporter like this:

```
java.sql.Connection conn = ...;
MetaDataExporter exporter = new MetaDataExporter();
exporter.setPackageName("com.myproject.mydomain");
exporter.setTargetFolder(new File("src/main/java"));
exporter.setBeanSerializer(new BeanSerializer());
exporter.export(conn.getMetaData());
```

Now you can use the bean types as arguments to the populate method in DML clauses and you can project directly to bean types in queries. Here is a simple example in JUnit form:

```
QEmployee e = new QEmployee("e");

// Insert
Employee employee = new Employee();
employee.setFirstname("John");
Integer id = queryFactory.insert(e).populate(employee).executeWithKey(e.id);
employee.setId(id);

// Update
employee.setLastname("Smith");
assertEquals(1l, queryFactory.update(e).populate(employee).where(e.id.eq(employee.getId())).execute());

// Query
Employee smith = queryFactory.selectFrom(e).where(e.lastname.eq("Smith")).fetchOne();
assertEquals("John", smith.getFirstname());

// Delete
assertEquals(1l, queryFactory.delete(e).where(e.id.eq(employee.getId())).execute());
```

### 2.3.20. Extracting the SQL query and bindings

The SQL query and bindings can be extracted via the getSQL method:

```
SQLBindings bindings = query.getSQL();
System.out.println(bindings.getSQL());
```

If you need also all literals in the SQL string you can enable literal serialization on the query or configuration level via setUseLiterals(true).

### 2.3.21. Custom types

Querydsl SQL provides the possibility to declare custom type mappings for ResultSet/Statement interaction. The custom type mappings can be declared in com.querydsl.sql.Configuration instances, which are supplied as constructor arguments to the actual queries:

```
Configuration configuration = new Configuration(new H2Templates());
// overrides the mapping for Types.DATE
configuration.register(new UtilDateType());
```

And for a table column

```
Configuration configuration = new Configuration(new H2Templates());
// declares a mapping for the gender column in the person table
configuration.register("person", "gender",  new EnumByNameType<Gender>(Gender.class));
```

To customize a numeric mapping you can use the registerNumeric method like this

```
configuration.registerNumeric(5,2,Float.class);
```

This will map the Float type to the NUMERIC(5,2) type.

### 2.3.22. Listening to queries and clauses

SQLListener is a listener interface that can be used to listen to queries and DML clause. SQLListener instances can be registered either on the configuration and on the query/clause level via the addListener method.

Use cases for listeners are data synchronization, logging, caching and validation.

### 2.3.23. Spring integration

Querydsl SQL integrates with Spring through the querydsl-sql-spring module:

```
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-sql-spring</artifactId>
  <version>${querydsl.version}</version>
</dependency>
```

It provides Spring exception translation and a Spring connection provider for usage of Querydsl SQL with Spring transaction managers. Below is a configuration example:

```
package com.querydsl.example.config;

import com.querydsl.sql.H2Templates;
import com.querydsl.sql.SQLQueryFactory;
import com.querydsl.sql.SQLTemplates;
import com.querydsl.sql.spring.SpringConnectionProvider;
import com.querydsl.sql.spring.SpringExceptionTranslator;
import com.querydsl.sql.types.DateTimeType;
import com.querydsl.sql.types.LocalDateType;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

import javax.inject.Inject;
import javax.inject.Provider;
import javax.sql.DataSource;
import java.sql.Connection;

@Configuration
public class JdbcConfiguration {

    @Bean
    public DataSource dataSource() {
        // implementation omitted
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean
    public com.querydsl.sql.Configuration querydslConfiguration() {
        SQLTemplates templates = H2Templates.builder().build(); //change to your Templates
        com.querydsl.sql.Configuration configuration = new com.querydsl.sql.Configuration(templates);
        configuration.setExceptionTranslator(new SpringExceptionTranslator());
        return configuration;
    }

    @Bean
    public SQLQueryFactory queryFactory() {
        Provider<Connection> provider = new SpringConnectionProvider(dataSource());
        return new SQLQueryFactory(querydslConfiguration(), provider);
    }

}
```



## 2.8. Querying Collections

The querydsl-collections module can be used with generated query types and without. The first section describes the usage without generated query types:

### 2.8.1. Usage without generated query types

To use querydsl-collections without generated query types you need to use the Querydsl alias feature. Here are some examples.

To get started, add the following static imports:

```
// needed for access of the Querydsl Collections API
import static com.querydsl.collections.CollQueryFactory.*;
// needed, if you use the $-invocations
import static com.querydsl.core.alias.Alias.*;
```

And now create an alias instance for the Cat class. Alias instances can only be created for non-final classes with an empty constructor. Make sure your class has one.

The alias instance of type Cat and its getter invocations are transformed into paths by wrapping them into dollar method invocations. The call `c.getKittens()` for example is internally transformed into the property path `c.kittens` inside the dollar method.

```
Cat c = alias(Cat.class, "cat");
for (String name : select($(c.getName())).from($(c),cats)
  .where($(c.getKittens()).size().gt(0))
  .fetch()) {
    System.out.println(name);
}
```

The following example is a variation of the previous, where the access to the list size happens inside the dollar-method invocation.

```
Cat c = alias(Cat.class, "cat");
for (String name : select($(c.getName())).from($(c),cats)
  .where($(c.getKittens().size()).gt(0))
  .fetch()) {
    System.out.println(name);
}
```

All non-primitive and non-final typed properties of aliases are aliases themselves. So you may cascade method calls until you hit a primitive or non-final type (e.g. java.lang.String) in the dollar-method scope.

e.g.

```
$(c.getMate().getName())
```

is transformed into *c.mate.name* internally, but

```
$(c.getMate().getName().toLowerCase())
```

is not transformed properly, since the toLowerCase() invocation is not tracked.

Note also that you may only invoke getters, size(), contains(Object) and get(int) on alias types. All other invocations throw exceptions.

### 2.8.2. Usage with generated query types

The example above can be expressed like this with generated expression types

```
QCat cat = new QCat("cat");
for (String name : select(cat.name).from(cat,cats)
  .where(cat.kittens.size().gt(0))
  .fetch()) {
    System.out.println(name);
}
```

When you use generated query types, you instantiate expressions instead of alias instances and use the property paths directly without any dollar-method wrapping.

### 2.8.3. Maven integration

Add the following dependencies to your Maven project:

```
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-collections</artifactId>
  <version>${querydsl.version}</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.6.1</version>
</dependency>
```

If you are not using JPA or JDO you can generate expression types for your domain types by annotating them with the `com.querydsl.core.annotations.QueryEntity` annotation and adding the following plugin configuration into your Maven configuration (pom.xml):

```
<project>
  <build>
  <plugins>
    ...
    <plugin>
      <groupId>com.mysema.maven</groupId>
      <artifactId>apt-maven-plugin</artifactId>
      <version>1.1.3</version>
      <executions>
        <execution>
          <goals>
            <goal>process</goal>
          </goals>
          <configuration>
            <outputDirectory>target/generated-sources/java</outputDirectory>
            <processor>com.querydsl.apt.QuerydslAnnotationProcessor</processor>
          </configuration>
        </execution>
      </executions>
    </plugin>
    ...
  </plugins>
  </build>
</project>
```

### 2.8.4. Ant integration

Place the jar files from the full-deps bundle on your classpath and use the following tasks for Querydsl code generation:

```
    <!-- APT based code generation -->
    <javac srcdir="${src}" classpathref="cp">
      <compilerarg value="-proc:only"/>
      <compilerarg value="-processor"/>
      <compilerarg value="com.querydsl.apt.QuerydslAnnotationProcessor"/>
      <compilerarg value="-s"/>
      <compilerarg value="${generated}"/>
    </javac>

    <!-- compilation -->
    <javac classpathref="cp" destdir="${build}">
      <src path="${src}"/>
      <src path="${generated}"/>
    </javac>
```

Replace *src* with your main source folder, *generated* with your folder for generated sources and *build* with your target folder.

### 2.8.5. Hamcrest matchers

Querydsl Collections provides Hamcrest matchers. With these imports

```
import static org.hamcrest.core.IsEqual.equalTo;
import static com.querydsl.collections.PathMatcher.hasValue;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertThat;
```

they can be used like this:

```
Car car = new Car();
car.setHorsePower(123);

assertThat(car, hasValue($.horsePower));
assertThat(car, hasValue($.horsePower, equalTo(123)));
```

The Hamcrest matchers have been contributed by [Jeroen van Schagen](https://github.com/jeroenvs) .

### 2.8.6. Usage with the Eclipse Compiler for Java

If Querydsl Collections is used with a JRE where the system compiler is not available, CollQuery instances can also be configured to use the Eclipse Compiler for Java (ECJ) instead:

```
DefaultEvaluatorFactory evaluatorFactory = new DefaultEvaluatorFactory(
    CollQueryTemplates.DEFAULT,
    new ECJEvaluatorFactory(getClass().getClassLoader()));
QueryEngine queryEngine = new DefaultQueryEngine(evaluatorFactory);
CollQuery query = new CollQuery(queryEngine);
```

# 3. 通用用法

通用用法部分涵盖了参考文档的教程部分中没有涉及的方面。它遵循一个面向用例的结构。

## 3.1. 创建查询

Querydsl 中的查询构造涉及使用表达式参数调用查询方法。 由于查询方法大多是特定于模块的，并且已经在教程部分介绍过，因此本部分将重点介绍表达式。

表达式通常是通过访问域模块生成的表达式类型的字段和调用方法来构造的。对于代码生成不适用的情况，可以使用通用的方法来构造表达式。

### 3.1.1. 复杂的谓词(predicates)

要构造复杂的布尔表达式，请使用`com.querydsl.core.BooleanBuilder` 类。 它实现了 Predicate 并且可以以级联形式使用：

```java
public List<Customer> getCustomer(String... names) {
    QCustomer customer = QCustomer.customer;
    JPAQuery<Customer> query = queryFactory.selectFrom(customer);
    BooleanBuilder builder = new BooleanBuilder();
    for (String name : names) {
        builder.or(customer.name.eq(name));
    }
    query.where(builder); // customer.name eq name1 OR customer.name eq name2 OR ...
    return query.fetch();
}
```

`BooleanBuilder` 是可变的，最初表示为 null，并且在每次 `and` 或 `or` 调用之后表示操作的结果。

### 3.1.2. 动态表达式(expressions)

`com.querydsl.core.types.dsl.Expressions` 类是用于动态表达式构建的静态工厂类。 工厂方法由返回的类型命名，并且大多是自文档的。

一般来说，`Expressions` 类应该只在不能使用流畅的 DSL 形式的情况下使用，例如动态路径、自定义语法或自定义操作。

下面的表达式

```java
QPerson person = QPerson.person;
person.firstName.startsWith("P");
```

如果 Q 类型不可用，可以像这样构造

```java
Path<Person> person = Expressions.path(Person.class, "person");
Path<String> personFirstName = Expressions.path(String.class, person, "firstName");
Constant<String> constant = Expressions.constant("P");
Expressions.predicate(Ops.STARTS_WITH, personFirstName, constant);
```

Path 实例表示变量和属性，Constants 是常量，Operations 是操作，TemplateExpression 实例可用于将表达式表示为 String 模板。

### 3.1.3. 动态路径

除了基于`Expressions` 的表达式创建，Querydsl 还为动态路径创建提供了更流畅的 API。

对于动态路径生成，可以使用 `com.querydsl.core.types.dsl.PathBuilder` 类。 它扩展了`EntityPathBase`，可以用作路径生成的类生成和别名使用的替代方法。

与 Expressions API 相比，PathBuilder 不提供对未知操作或自定义语法的直接支持，但语法更接近于普通的 DSL。

字符串属性：

```java
PathBuilder<User> entityPath = new PathBuilder<User>(User.class, "entity");
// fully generic access
entityPath.get("userName");
// .. or with supplied type
entityPath.get("userName", String.class);
// .. and correct signature
entityPath.getString("userName").lower();
```

带有组件类型的列表属性:

```java
entityPath.getList("list", String.class).get(0);
```

使用组件表达式类型:

```java
entityPath.getList("list", String.class, StringPath.class).get(0).lower();
```

带有键和值类型的Map属性:

```java
entityPath.getMap("map", String.class, String.class).get("key");
```

使用组件表达式类型:

```java
entityPath.getMap("map", String.class, String.class, StringPath.class).get("key").lower();
```

对于PathBuilder验证，可以使用PathBuilderValidator。它可以被注入到构造函数中，并将被传递到新的PathBuilder中

```java
PathBuilder<Customer> customer = new PathBuilder<Customer>(Customer.class, "customer", validator);
```

PathBuilderValidator.FIELDS 将验证字段是否存在，PathBuilderValidator.PROPERTIES 验证 Bean 属性，JPAPathBuilderValidator 使用 JPA 元模型进行验证。

### 3.1.4. CASE 表达式

要构造 case-when-then-else 表达式，请使用`CaseBuilder`类，如下所示：

```java
QCustomer customer = QCustomer.customer;
Expression<String> cases = new CaseBuilder()
    .when(customer.annualSpending.gt(10000)).then("Premier")
    .when(customer.annualSpending.gt(5000)).then("Gold")
    .when(customer.annualSpending.gt(2000)).then("Silver")
    .otherwise("Bronze");
// 现在可以在投影或条件中使用 case 表达式
```

对于具有相等操作的 case 表达式，请改用以下更简单的形式：

```java
QCustomer customer = QCustomer.customer;
Expression<String> cases = customer.annualSpending
    .when(10000).then("Premier")
    .when(5000).then("Gold")
    .when(2000).then("Silver")
    .otherwise("Bronze");
// 现在可以在投影或条件中使用 case 表达式
```

JDOQL 尚不支持 Case 表达式。

### 3.1.5. Casting 表达式

为了避免表达式类型中的泛型签名，类型层次结构被扁平化。 结果是所有生成的查询类型都是 `com.querydsl.core.types.dsl.EntityPathBase` 或 `com.querydsl.core.types.dsl.BeanPath` 的直接子类，不能直接转换为它们的逻辑超类型。

超类型引用可以通过`_super`'字段访问，而不是直接的Java强制转换。`_super-field`在所有生成的查询类型中都是可用的，只有一个超类型:

```java
// from Account
QAccount extends EntityPathBase<Account> {
    // ...
}

// from BankAccount extends Account
QBankAccount extends EntityPathBase<BankAccount> {

    public final QAccount _super = new QAccount(this);

    // ...
}
```

要将超类型转换为子类型，可以使用EntityPathBase类的as-method:

```java
QAccount account = new QAccount("account");
QBankAccount bankAccount = account.as(QBankAccount.class);
```

### 3.1.6. 选择字面量(Select literals)

可以通过常量表达式引用字面量来选择它们。这里有一个简单的例子

```java
query.select(Expressions.constant(1),
             Expressions.constant("abc"));
```

常量表达式常用于子查询中。

## 3.2. 结果处理

Querydsl 提供了两种自定义结果的方式，FactoryExpressions 用于基于行的转换，ResultTransformer 用于聚合。

`com.querydsl.core.types.FactoryExpression` 接口用于 Bean 创建、构造函数调用和更复杂对象的创建。 Querydsl 的 FactoryExpression 实现的功能可以通过`com.querydsl.core.types.Projections` 类访问。

对于`com.querydsl.core.ResultTransformer` 接口，`GroupBy` 是主要的实现。

### 3.2.1. 返回多列

从 Querydsl 3.0 开始，多列结果的默认类型是 `com.querydsl.core.Tuple`。 Tuple 提供了一个类似类型安全 Map 的接口来访问来自 Tuple 行对象的列数据。

```java
List<Tuple> result = query.select(employee.firstName, employee.lastName)
                          .from(employee).fetch();
for (Tuple row : result) {
     System.out.println("firstName " + row.get(employee.firstName));
     System.out.println("lastName " + row.get(employee.lastName));
}}
```

这个例子也可以像这样通过 QTuple 表达式类编写

```java
List<Tuple> result = query.select(new QTuple(employee.firstName, employee.lastName))
                          .from(employee).fetch();
for (Tuple row : result) {
     System.out.println("firstName " + row.get(employee.firstName));
     System.out.println("lastName " + row.get(employee.lastName));
}}
```

### 3.2.2. Bean 操作

在需要根据查询结果填充 Bean 的情况下，可以像这样使用 Bean 投影

```java
List<UserDTO> dtos = query.select(
    Projections.bean(UserDTO.class, user.firstName, user.lastName)).fetch();
```

当应该直接使用字段而不是 setter 时，可以使用以下变体:

```java
List<UserDTO> dtos = query.select(
    Projections.fields(UserDTO.class, user.firstName, user.lastName)).fetch();
```

### 3.2.3.构造函数用法

可以像这样使用基于构造函数的行转换

```java
List<UserDTO> dtos = query.select(
    Projections.constructor(UserDTO.class, user.firstName, user.lastName)).fetch();
```

作为泛型构造函数表达式的替代用法，构造函数也可以使用`QueryProjection`注解:

```java
class CustomerDTO {

  @QueryProjection
  public CustomerDTO(long id, String name) {
     ...
  }

}
```

然后你可以像这样在查询中使用它

```java
QCustomer customer = QCustomer.customer;
JPQLQuery query = new HibernateQuery(session);
List<CustomerDTO> dtos = query.select(new QCustomerDTO(customer.id, customer.name))
                              .from(customer).fetch();
```

虽然这个示例是特定于Hibernate的，但该特性在所有模块中都可用。

如果QueryProjection注释的类型不是一个带注释的实体类型,您可以使用构造函数投影,但如果注释的类型是实体类型，则需要通过调用 查询类型的静态`create`方法：

```java
@Entity
class Customer {

  @QueryProjection
  public Customer(long id, String name) {
     ...
  }

}

QCustomer customer = QCustomer.customer;
JPQLQuery query = new HibernateQuery(session);
List<Customer> dtos = query.select(QCustomer.create(customer.id, customer.name))
                           .from(customer).fetch();
```

或者，如果代码生成是可选的，你可以像这样创建一个构造函数投影:

```java
List<Customer> dtos = query
    .select(Projections.constructor(Customer.class, customer.id, customer.name))
    .from(customer).fetch();
```

### 3.2.4. 结果聚合

`com.querydsl.core.group.GroupBy` 类提供聚合功能，可用于在内存中聚合查询结果。 下面是一些使用示例。

聚合父子关系

```java
import static com.querydsl.core.group.GroupBy.*;

Map<Integer, List<Comment>> results = query.from(post, comment)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(list(comment)));
```

这将返回一个帖子 ID 映射到相关评论。

多个结果列

```java
Map<Integer, Group> results = query.from(post, comment)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(post.name, set(comment.id)));
```

这将返回一个帖子 ID 映射到可以访问帖子名称和评论 ID 的 Group 实例。

Group 是相当于 Tuple 接口的 GroupBy。

更多例子可以在这里找到[这里](https://github.com/querydsl/querydsl/blob/master/querydsl-collections/src/test/java/com/querydsl/collections/GroupByTest.java) .

## 3.3. 代码生成

Querydsl中使用了Java 6 APT注解处理功能，用于JPA、JDO和Mongodb模块中的代码生成。本节描述用于代码生成的各种配置选项和APT使用的一种替代方法。

### 3.3.1. 路径初始化

默认情况下，Querydsl 仅初始化前两层的引用属性。 在需要更长的初始化路径的情况下，必须通过`com.querydsl.core.annotations.QueryInit` 注释在域类型中注释这些路径。 QueryInit 用于需要深度初始化的属性。 下面的例子演示了用法。

```java
@Entity
class Event {
    @QueryInit("customer.address")
    Account account;
}

@Entity
class Account {
    Customer customer;
}

@Entity
class Customer {
    String name;
    Address address;
    // ...
}
```

当事件路径被初始化为根`路径/变量`时，此示例强制执行 account.customer 路径的初始化。 路径初始化格式也支持通配符，例如 `"customer.*"` 或只是 `"*"`.

自动路径初始化取代了手动路径初始化，手动路径初始化要求实体字段是非final的。声明式格式的好处是可以应用于Query类型的所有顶级实例，并允许使用final实体字段。

自动路径初始化是首选的初始化策略，但是手动初始化可以通过Config注释激活，下面将对此进行描述。

### 3.3.2. 定制化

Querydsl的序列化可以通过包和类型上的Config注解进行定制。它们自定义带注解的包或类型的序列化。

序列化选项是



**表 3.1. 配置选项**

|          名称          |                         描述                          |
| --------------------- | ---------------------------------------------------- |
| entityAccessors       | 实体路径的访问器方法而不是公共最终字段（默认值：false） |
| listAccessors         | listProperty(int index) 样式方法 (默认值: false)      |
| mapAccessors          | mapProperty(Key key) 样式存取方法 (默认值: false)     |
| createDefaultVariable | 生成默认变量 (默认值: true)                           |
| defaultVariableName   | 默认变量的名称                                        |



下面是一些例子。

实体类型序列化定制:

```java
@Config(entityAccessors=true)
@Entity
public class User {
    //...
}
```

包内容定制:

```java
@Config(listAccessors=true)
package com.querydsl.core.domain.rel;

import com.querydsl.core.annotations.Config;
```

如果您想全局定制序列化器配置，可以通过以下APT选项来实现



**表 3.2. APT 选项**

|              名称               |                        描述                         |
| ------------------------------ | -------------------------------------------------- |
| querydsl.entityAccessors       | 启用引用字段访问器                                   |
| querydsl.listAccessors         | 为直接索引列表访问启用访问器                         |
| querydsl.mapAccessors          | 为直接基于键的映射访问启用访问器                     |
| querydsl.prefix                | 覆盖查询类型的前缀(默认:Q)                           |
| querydsl.suffix                | 为查询类型设置后缀                                   |
| querydsl.packageSuffix         | 设置查询类型包的后缀                                 |
| querydsl.createDefaultVariable | 设置是否创建默认变量                                 |
| querydsl.unknownAsEmbeddable   | 设置未知的非注解类应被视为可嵌入的（默认值：false）   |
| querydsl.includedPackages      | 要包含在代码生成中的包的逗号分隔列表（默认值：all）   |
| querydsl.includedClasses       | 要包含在代码生成中的类名的逗号分隔列表（默认值：all）  |
| querydsl.excludedPackages      | 要从代码生成中排除的包的逗号分隔列表（默认值：none)   |
| querydsl.excludedClasses       | 要从代码生成中排除的类名的逗号分隔列表（默认值：none) |
| querydsl.useFields             | 设置字段是否用作元数据源（默认值：true）              |
| querydsl.useGetters            | 设置是否将访问器用作元数据源（默认值：true）          |



使用 Maven APT 插件，它的工作方式如下：

```xml
<project>
  <build>
  <plugins>
    ...
    <plugin>
      <groupId>com.mysema.maven</groupId>
      <artifactId>apt-maven-plugin</artifactId>
      <version>1.1.3</version>
      <executions>
        <execution>
          <goals>
            <goal>process</goal>
          </goals>
          <configuration>
            <outputDirectory>target/generated-sources/java</outputDirectory>
            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            <options>
              <querydsl.entityAccessors>true</querydsl.entityAccessors>
              <querydsl.useFields>false</querydsl.useFields>
            </options>
          </configuration>
        </execution>
      </executions>
    </plugin>
    ...
  </plugins>
  </build>
</project>
```

或者，`maven-compiler-plugin` 可以配置为将 APT 直接挂接到编译中：

```xml
      <project>
        <build>
        <plugins>
          ...
          <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
              <generatedSourcesDirectory>target/generated-sources/java</generatedSourcesDirectory>
              <compilerArgs>
                <arg>-Aquerydsl.entityAccessors=true</arg>
                <arg>-Aquerydsl.useFields=false</arg>
              </compilerArgs>
            </configuration>
            <dependencies>
              <dependency>
                <groupId>com.querydsl</groupId>
                <artifactId>querydsl-apt</artifactId>
                <version>${querydsl.version}</version>
                <classifier>jpa</classifier>
              </dependency>
              <dependency>
                <groupId>org.hibernate.javax.persistence</groupId>
                <artifactId>hibernate-jpa-2.1-api</artifactId>
                <version>1.0.0.Final</version>
              </dependency>
            </dependencies>
          </plugin>
          ...
        </plugins>
        </build>
      </project>
```

请注意，在定义对`com.querydsl:querydsl-apt` 的依赖时，您需要使用适当的分类器。 这些额外的工件定义了要在`META-INF/services/javax.annotation.processing.Processor`中使用的注释处理器。

可用的分类器包括：

- `general`
- `hibernate`
- `jdo`
- `jpa`

通过此配置，可以在编译域对象期间生成和编译查询对象的源。这还将自动将生成的源目录添加到Maven项目的根目录中。

这种方法的一大优点是它还可以使用 `groovy-eclipse` 编译器处理带注解的 Groovy 类：

```xml
      <project>
        <build>
        <plugins>
          ...
          <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
              <compilerId>groovy-eclipse-compiler</compilerId>
              <generatedSourcesDirectory>target/generated-sources/java</generatedSourcesDirectory>
              <compilerArgs>
                <arg>-Aquerydsl.entityAccessors=true</arg>
                <arg>-Aquerydsl.useFields=false</arg>
              </compilerArgs>
            </configuration>
            <dependencies>
              <dependency>
                <groupId>org.codehaus.groovy</groupId>
                <artifactId>groovy-eclipse-compiler</artifactId>
                <version>2.9.1-01</version>
              </dependency>
              <dependency>
                <groupId>org.codehaus.groovy</groupId>
                <artifactId>groovy-eclipse-batch</artifactId>
                <version>2.3.7-01</version>
              </dependency>
              <dependency>
                <groupId>com.querydsl</groupId>
                <artifactId>querydsl-apt</artifactId>
                <version>${querydsl.version}</version>
                <classifier>jpa</classifier>
              </dependency>
              <dependency>
                <groupId>org.hibernate.javax.persistence</groupId>
                <artifactId>hibernate-jpa-2.1-api</artifactId>
                <version>1.0.0.Final</version>
              </dependency>
            </dependencies>
          </plugin>
          ...
        </plugins>
        </build>
      </project>
```

### 3.3.3. 自定义类型映射

可以在属性上使用自定义类型映射来覆盖派生的Path类型。这在某些情况下很有用，例如比较和String操作应该在特定的String路径上被阻塞，或者需要添加对自定义类型的Date / Time支持。 支持Joda时间API和JDK的日期/时间类型(java.util.Date , Calendar和子类型)是内置的，但可能需要使用此特性支持其他api。

下面的例子演示了用法:

```java
@Entity
public class MyEntity {
    @QueryType(PropertyType.SIMPLE)
    public String stringAsSimple;

    @QueryType(PropertyType.COMPARABLE)
    public String stringAsComparable;

    @QueryType(PropertyType.NONE)
    public String stringNotInQuerydsl;
}
```

 `PropertyType.NONE` 可用于跳过查询类型生成中的属性。 这种情况与`@Transient` 或`@QueryTransient` 注解属性不同，属性不持久化。 `PropertyType.NONE` 只是从 Querydsl 查询类型中省略了属性。

### 3.3.4. 代理方法

要将静态方法声明为代理方法，请添加带有相应域类型作为值的`QueryDelegate`注解，并提供将相应的 Querydsl 查询类型作为第一个参数的方法签名。

这是单元测试中的一个简单示例：

```java
@QueryEntity
public static class User {

    String name;

    User manager;

}
@QueryDelegate(User.class)
public static BooleanPath isManagedBy(QUser user, User other) {
    return user.manager.eq(other);
}
```

以及 QUser 查询类型中生成的方法：

```java
public BooleanPath isManagedBy(QUser other) {
    return DelegateTest.isManagedBy(this, other);
}
```

代理方法也可用于扩展内置类型。 这里有些例子

```java
public class QueryExtensions {

    @QueryDelegate(Date.class)
    public static BooleanExpression inPeriod(DatePath<Date> date, Pair<Date,Date> period) {
        return date.goe(period.getFirst()).and(date.loe(period.getSecond()));
    }

    @QueryDelegate(Timestamp.class)
    public static BooleanExpression inDatePeriod(DateTimePath<Timestamp> timestamp, Pair<Date,Date> period) {
        Timestamp first = new Timestamp(DateUtils.truncate(period.getFirst(), Calendar.DAY_OF_MONTH).getTime());
        Calendar second = Calendar.getInstance();
        second.setTime(DateUtils.truncate(period.getSecond(), Calendar.DAY_OF_MONTH));
        second.add(1, Calendar.DAY_OF_MONTH);
        return timestamp.goe(first).and(timestamp.lt(new Timestamp(second.getTimeInMillis())));
    }

}
```

当为内置类型声明代理方法时，就会创建具有适当代理方法用法的子类:

```java
public class QDate extends DatePath<java.sql.Date> {

    public QDate(BeanPath<? extends java.sql.Date> entity) {
        super(entity.getType(), entity.getMetadata());
    }

    public QDate(PathMetadata<?> metadata) {
        super(java.sql.Date.class, metadata);
    }

    public BooleanExpression inPeriod(com.mysema.commons.lang.Pair<java.sql.Date, java.sql.Date> period) {
        return QueryExtensions.inPeriod(this, period);
    }

}

public class QTimestamp extends DateTimePath<java.sql.Timestamp> {

    public QTimestamp(BeanPath<? extends java.sql.Timestamp> entity) {
        super(entity.getType(), entity.getMetadata());
    }

    public QTimestamp(PathMetadata<?> metadata) {
        super(java.sql.Timestamp.class, metadata);
    }

    public BooleanExpression inDatePeriod(com.mysema.commons.lang.Pair<java.sql.Date, java.sql.Date> period) {
        return QueryExtensions.inDatePeriod(this, period);
    }

}
```

### 3.3.5. 非注解类型

可以通过创建 `@QueryEntities` 注解来为非注解类型创建 Querydsl 查询类型。 只需将 QueryEntities 注解放入您选择的包中，并在 value 属性中映射要映射的类。

要实际创建类型，请使用 `com.querydsl.apt.QuerydslAnnotationProcessor`。 在 Maven 中，您可以这样做：

```xml
<project>
  <build>
  <plugins>
    ...
    <plugin>
      <groupId>com.mysema.maven</groupId>
      <artifactId>apt-maven-plugin</artifactId>
      <version>1.1.3</version>
      <executions>
        <execution>
          <goals>
            <goal>process</goal>
          </goals>
          <configuration>
            <outputDirectory>target/generated-sources/java</outputDirectory>
            <processor>com.querydsl.apt.QuerydslAnnotationProcessor</processor>
          </configuration>
        </execution>
      </executions>
    </plugin>
    ...
  </plugins>
  </build>
</project>
```

### 3.3.6. 基于类路径(Classpath)的代码生成

对于带注解的 Java 源代码不可用的情况，例如使用不同的 JVM 语言（如 Scala 或 Groovy）或通过字节码操作添加注解，`GenericExporter`类可用于扫描带注解类的类路径并为他们生成查询类型 。

要使 GenericExporter 可用，请将 querydsl-codegen 模块的依赖项添加到您的项目中，或者更准确地说是`com.querydsl:querydsl-codegen:${querydsl.version}`。

以下是 JPA 的示例

```java
GenericExporter exporter = new GenericExporter();

exporter.setKeywords(Keywords.JPA);
exporter.setEntityAnnotation(Entity.class);
exporter.setEmbeddableAnnotation(Embeddable.class);
exporter.setEmbeddedAnnotation(Embedded.class);
exporter.setSupertypeAnnotation(MappedSuperclass.class);
exporter.setSkipAnnotation(Transient.class);
exporter.setTargetFolder(new File("target/generated-sources/java"));

exporter.export(DomainClass.class.getPackage());
export("org.xyz.abc")
```

这会将 指定包名和子包的包中的所有 JPA 注解类导出到 `target/generated-sources/java` 目录。

#### 3.3.6.1. 通过 Maven 使用

`querydsl-maven-plugin` 的目标 `generic-export`、`jpa-export` 和 `jdo-export` 可通过 Maven 用于 GenericExporter。

不同的目标映射到 Querydsl、JPA 和 JDO 注解。

配置元素是

**表 3.3. Maven 配置**

|   类型    |      元素       |                   描述                    |
| -------- | -------------- | ----------------------------------------- |
| File     | targetFolder   | 生成源代码的目标文件夹                     |
| boolean  | scala          | 是否应该生成 Scala 源代码（默认值：false） |
| String[] | packages       | 要为实体类自省的包                         |
| boolean  | handleFields   | 如果字段应被视为属性（默认值：true）        |
| boolean  | handleMethods  | 如果 getter 应该被视为属性（默认值：true） |
| String   | sourceEncoding | 生成的源文件的字符集编码                   |
| boolean  | testClasspath  | 是否应该使用测试类路径                     |



这是 JPA 注解类的示例

```xml
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-maven-plugin</artifactId>
        <version>${querydsl.version}</version>
        <executions>
          <execution>
            <phase>process-classes</phase>
            <goals>
              <goal>jpa-export</goal>
            </goals>
            <configuration>
              <targetFolder>target/generated-sources/java</targetFolder>
              <packages>
                <package>com.example.domain</package>
              </packages>
            </configuration>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
</project>
```

这会将 `com.example.domain` 包和子包的 JPA 注解类导出到 `target/generated-sources/java` 目录。

如果您需要在此之后直接编译生成的源代码，那么您可以使用 `compile` 目标。

```xml
<execution>
  <goals>
    <goal>compile</goal>
  </goals>
  <configuration>
    <sourceFolder>target/generated-sources/scala</targetFolder>
  </configuration>
</execution>
```

`compile` 目标具有以下配置元素



**表 3.4. Maven 配置**

|   类型   |       元素       |          描述           |
| ------- | --------------- | ----------------------- |
| File    | sourceFolder    | 生成的源代码的源文件目录 |
| String  | sourceEncoding  | 源文件的字符编码         |
| String  | source          | 编译器的 -source 选项    |
| String  | target          | 编译器的 -target 选项    |
| boolean | testClasspath   | 是否应该使用测试类路径   |
| Map     | compilerOptions | 编译器选项               |



除了 `sourceFolder` 之外的所有选项都是可选的。

#### 3.3.6.2. Scala 支持

如果您需要类的 Scala 输出，请使用以下配置的变体

```xml
<project>
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-maven-plugin</artifactId>
        <version>${querydsl.version}</version>
        <dependencies>
          <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-scala</artifactId>
            <version>${querydsl.version}</version>
          </dependency>
          <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
          </dependency>
        </dependencies>
        <executions>
          <execution>
            <goals>
              <goal>jpa-export</goal>
            </goals>
            <configuration>
              <targetFolder>target/generated-sources/scala</targetFolder>
              <scala>true</scala>
              <packages>
                <package>com.example.domain</package>
              </packages>
            </configuration>
          </execution>
        </executions>
      </plugin>
      ...
    </plugins>
  </build>
</project>
```

# 4. Troubleshooting

## 4.1. Insufficient type arguments

Querydsl needs properly encoded List Set, Collection and Map properties in all code generation scenarios.

When using improperly encoded fields or getters you might the following stacktrace:

```
java.lang.RuntimeException: Caught exception for field com.querydsl.jdo.testdomain.Store#products
  at com.querydsl.apt.Processor$2.visitType(Processor.java:117)
  at com.querydsl.apt.Processor$2.visitType(Processor.java:80)
  at com.sun.tools.javac.code.Symbol$ClassSymbol.accept(Symbol.java:827)
  at com.querydsl.apt.Processor.getClassModel(Processor.java:154)
  at com.querydsl.apt.Processor.process(Processor.java:191)
  ...
Caused by: java.lang.IllegalArgumentException: Insufficient type arguments for List
  at com.querydsl.apt.APTTypeModel.visitDeclared(APTTypeModel.java:112)
  at com.querydsl.apt.APTTypeModel.visitDeclared(APTTypeModel.java:40)
  at com.sun.tools.javac.code.Type$ClassType.accept(Type.java:696)
  at com.querydsl.apt.APTTypeModel.<init>(APTTypeModel.java:55)
  at com.querydsl.apt.APTTypeModel.get(APTTypeModel.java:48)
  at com.querydsl.apt.Processor$2.visitType(Processor.java:114)
  ... 35 more
```

Examples of problematic field declarations and their corrections:

```
    private Collection names; // WRONG

    private Collection<String> names; // RIGHT

    private Map employeesByName; // WRONG

    private Map<String,Employee> employeesByName; // RIGHT
```

## 4.2. Multithreaded initialization of Querydsl Q-types

When Querydsl Q-types are initialized from multiple threads, deadlocks can occur, if the Q-types have circular dependencies.

An easy to use solution is to initialize the classes in a single thread before they are used in different threads.

The com.querydsl.codegen.ClassPathUtils class can be used for that like this:

```
    ClassPathUtils.scanPackage(Thread.currentThread().getContextClassLoader(), packageToLoad);
```

Replace packageToLoad with the package of the classes you want to initialize.

## 4.3. JDK5 usage

When compiling your project with JDK 5, you might get the following compilation failure:

```
[INFO] ------------------------------------------------------------------------
[ERROR] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Compilation failure
...
class file has wrong version 50.0, should be 49.0
```

The class file version 50.0 is used by Java 6.0, and 49.0 is used by Java 5.0.

Querydsl is tested against JDK 6.0 only, as we use APT extensively, which is available only since JDK 6.0.

If you want to use it with JDK 5.0 you might want to try to compile Querydsl yourself.
