## 3. Spring Data JPA 使用

> 怎么用

### 3.0 前置知识

#### EntityManager

> Spring Data JPA 基于Hibernate的封装，核心是EntityManger对数据库机型操作

#### Entity

> Entity是Spring Data的核心，它定义了应用实体与存储介质的映射关系

#### Repository

>  Spring Data库的核心接口是`Repository`。该接口作为一个标记接口，利用Java语言本身特性来发现Repository接口。



![image-20220228235420438](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220228235420438.png)

![JPARepository](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/JPARepository.png)

**`CrudRepository `接口**

```java
public interface CrudRepository<T, ID extends Serializable>

 extends Repository<T, ID> {        

 <S extends T> S save(S entity);    (1)

 T findOne(ID primaryKey);          (2)

 Iterable<T> findAll();             (3)

 Long count();                      (4)

 void delete(T entity);             (5)

 boolean exists(ID primaryKey);     (6)

 // … more functionality omitted.

}
```

**`CrudRepository `接口**

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	Iterable<T> findAll(Sort sort);

	Page<T> findAll(Pageable pageable);
}
```

`JpaSpecificationExecutor`**接口**

```java

/**
 * Interface to allow execution of {@link Specification}s based on the JPA criteria API.
 *
 */
public interface JpaSpecificationExecutor<T> {

	/**
	 * Returns a single entity matching the given {@link Specification} or {@link Optional#empty()} if none found.
	 *
	 * @param spec can be {@literal null}.
	 * @return never {@literal null}.
	 * @throws org.springframework.dao.IncorrectResultSizeDataAccessException if more than one entity found.
	 */
	Optional<T> findOne(@Nullable Specification<T> spec);

	/**
	 * Returns all entities matching the given {@link Specification}.
	 *
	 * @param spec can be {@literal null}.
	 * @return never {@literal null}.
	 */
	List<T> findAll(@Nullable Specification<T> spec);

	/**
	 * Returns a {@link Page} of entities matching the given {@link Specification}.
	 *
	 * @param spec can be {@literal null}.
	 * @param pageable must not be {@literal null}.
	 * @return never {@literal null}.
	 */
	Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);

	/**
	 * Returns all entities matching the given {@link Specification} and {@link Sort}.
	 *
	 * @param spec can be {@literal null}.
	 * @param sort must not be {@literal null}.
	 * @return never {@literal null}.
	 */
	List<T> findAll(@Nullable Specification<T> spec, Sort sort);

	/**
	 * Returns the number of instances that the given {@link Specification} will return.
	 *
	 * @param spec the {@link Specification} to count instances for. Can be {@literal null}.
	 * @return the number of instances.
	 */
	long count(@Nullable Specification<T> spec);
}

```

### 3.1 spring-boot集成

#### 3.1.1 创建工程

- 工程名：`awesome-jpa`

- 构建工具：`gradle`

- spring-boot版本：`2.5.11-SNAPSHOT`
- 数据库： `Mysql5.7`

![image-20220511164207873](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511164207873.png)

#### 3.1.2 引入依赖

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

#### 3.1.3 初始化数据库

```sql
CREATE DATABASE `awesome-jpa` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

#### 3.1.4 服务配置

```yaml
server:
  port: 8080
spring:
  application:
    name: awesome-jpa
  cache:
    type: redis
    redis:
      time-to-live: 3600000
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.zaxxer.hikari.HikariDataSource
    url: jdbc:mysql://localhost:3306/awesome-jpa?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&useSSL=false
    username: root
    password: root
  jpa:
    database: MySQL
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    hibernate:
      ddl-auto: update
    generate-ddl: true
```

### 3.2 基本用法

#### 3.2.1 实体映射

##### 1) 常用JPA配置

```
spring.jpa.hibernate.ddl-auto=none|create|create-drop|upadte|validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
datasource ...
```

- `create`：每次运行程序时，都会重新创建表，故而数据会丢失
- `create-drop`：每次运行程序时会先创建表结构，然后待程序结束时清空表
- `upadte`：每次运行程序，没有表时会创建表，如果对象发生改变会更新表结构，原有数据不会清空，只会更新
- `validate`：运行程序会校验数据与数据库的字段类型是否相同，字段不同会报错
- `none`: 禁用DDL处理

##### 2) 常用注解

- @Entity

表示这是一个实体类，默认情况下，类名就是表名

- @Table

与`@Entity`并列使用，自定义数据库表名，该注解完全可以忽略掉不用

- @Id 

表明该属性字段是一个主键，该属性必须具备，不可缺少。

- @GeneratedValue

和 @Id 主键注解一起使用

1. @GeneratedValue(strategy= GenerationType.IDENTITY) 该注解由数据库自动生成，主键自增型，在 mysql 数据库中使用最频繁，oracle 不支持。
2. @GeneratedValue(strategy= GenerationType.AUTO) 主键由程序控制，默认的主键生成策略，oracle 默认是序列化的方式，mysql 默认是主键自增的方式。
3. @GeneratedValue(strategy= GenerationType.SEQUENCE) 根据底层数据库的序列来生成主键，条件是数据库支持序列，Oracle支持，Mysql不支持。
4. @GeneratedValue(strategy= GenerationType.TABLE) 使用一个特定的数据库表格来保存主键，较少使用。

- @Column

用来描述实体属性对应数据表字段，其中`name、nullable、unique、length`属性用的较多，分别用来描述字段名、字段是否可以null、字段是否唯一、字段的长度。

- @Transient 

该注解标注的字段不会被应射到数据库当中。

#### 3.2.2 通用增删查改

##### JpaRepository  

##### crudRepository  

##### PagingAndSortingRepository  

#### 3.2.3 方法名查询

##### 1) 根据指定字段匹配

findByPostsaleNo

##### 2) 根据指定字段模糊匹配

findByPostsaleNoLike

##### 3) 关联关系

And Or 

##### 4) limit

Top first

##### 5) 分页&排序

Pageable  Sort 

#### 3.2.4 JPQL查询

```
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```

#### 3.2.5 原生SQL



#### 3.2.6 Example

Example翻译过来叫做"按例查询"。是一种用户界面友好的查询技术。 它允许动态创建查询，并且不需要编写包含字段名称的查询。 而且按示例查询不需要使用特定的数据库的查询语言来编写查询语句。

**优势：**

- 可以使用动态或者静态的限制去查询
- 在重构你的实体的时候，不用担心影响到已有的查询
- 可以独立地工作在数据查询API之外

**劣势：**

- 不支持组合查询，比如：firstname = ?0 or (firstname = ?1 and lastname = ?2).
- 只支持字符串的starts/contains/ends/regex匹配，对于非字符串的属性，只支持精确匹配。换句话说，并不支持大于、小于、between等匹配。

简单来说，Example查询具有更好的可读性。但是笔者使用了一下，觉得功能太弱，不如Specification查询。就是**使用很简单，但功能很弱**。

```java
public interface QueryByExampleExecutor<T> {
    <S extends T> Optional<S> findOne(Example<S> example);
    <S extends T> Iterable<S> findAll(Example<S> example);
    <S extends T> Iterable<S> findAll(Example<S> example, Sort sort);
    <S extends T> Page<S> findAll(Example<S> example, Pageable pageable);
    <S extends T> long count(Example<S> example);
    <S extends T> boolean exists(Example<S> example);
}
```

另外对于字符串还支持其他匹配（精准匹配之外）

```java
User exampleUser = User.builder()
        .name("B")
        .username("Example_User")
        .address(exampleAddress)
        .build();
    ExampleMatcher matcher = ExampleMatcher.matching()
        .withMatcher("name", m -> m.startsWith())
        .withMatcher("address.detail", m -> m.endsWith())
        .withMatcher("username", m -> m.ignoreCase());

    Optional<User> userOptional = userRepository.findOne(Example.of(exampleUser, matcher));
```

#### 3.2.7 Specification

JPA 2 规范引进了criteria查询API。Spring Data JPA对此提供了支持。如果你想使用这个功能，只需要继承`JpaSpecificationExecutor`接口。这个接口已经实现了基本的查询方法（findOne，findAll，count等）。

```java
public interface Specification<T> extends Serializable {

	long serialVersionUID = 1L;

	static <T> Specification<T> not(@Nullable Specification<T> spec) {

		return spec == null //
				? (root, query, builder) -> null //
				: (root, query, builder) -> builder.not(spec.toPredicate(root, query, builder));
	}

	static <T> Specification<T> where(@Nullable Specification<T> spec) {
		return spec == null ? (root, query, builder) -> null : spec;
	}

	default Specification<T> and(@Nullable Specification<T> other) {
		return SpecificationComposition.composed(this, other, CriteriaBuilder::and);
	}

	default Specification<T> or(@Nullable Specification<T> other) {
		return SpecificationComposition.composed(this, other, CriteriaBuilder::or);
	}

	@Nullable
	Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
}
```

root、query、criteriaBuild三个参数的类型都是在javax.persistence.criteria包下，且都是在JPA 2.0规范添加的类和接口。这里分别介绍一下三个重要的参数。

**root**

`Root<X>`接口，主要用于处理实体和字段、实体与实体之间的关系。除了上述例子中的取字段的操作以外，还可以做`join`操作。具体可以参考[Hibernate implementation’s documentation for an example](http://docs.jboss.org/hibernate/jpamodelgen/1.0/reference/en-US/html_single/#whatisit)。

**query**

`CriteriaQuery<T>`接口，主要用于对查询结果的处理，包括groupBy、orderBy、having、distinct等操作。

**criteriaBuilder**

`CriteriaBuilder`接口，主要用于各种条件查询、模拟sql函数等。

![image-20220510173536344](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220510173536344.png)

Specification查询主要用于复杂查询。通过三个参数的组合，可以实现基本上绝大部分sql能实现的复杂查询，且基于Java代码，具有很好的可读性。

### 3.3 进阶用法

#### 3.3.1 JPA 审计

在数据创建或者修改时，就能够记录下是谁、什么时间创建/修改了这个数据。Spring Data JPA就为我们提供了这个功能，叫Auditing，中文翻译是“审计、查账”的意思。你只需要一些简单的配置，就能轻松地实现这个功能。Auditing的原理是基于**Aspect**的“面向切面编程”。

##### 1）配置

```
@Configuration
@EnableJpaAuditing
public class JPAConfiguration {
    @Bean
    public AuditorAware<User> getCurrentUser() {
        User currentUser = User.builder().name("Bob").age(20).build();
        return () -> Optional.of(currentUser);
    }
}

// 官方文档以SpringSecurity为示例
class SpringSecurityAuditorAware implements AuditorAware<User> {

  public Optional<User> getCurrentAuditor() {

    return Optional.ofNullable(SecurityContextHolder.getContext())
			  .map(SecurityContext::getAuthentication)
			  .filter(Authentication::isAuthenticated)
			  .map(Authentication::getPrincipal)
			  .map(User.class::cast);
  }
}
```

##### 2）使用

```
@EntityListeners(AuditingEntityListener.class)
```

- `@CreatedBy`：谁创建的
- `@LastModifiedBy`：谁最后修改
- `@CreatedDate`：创建时间
- `@LastModifiedDate`：最后修改时间

```
@Builder
@EqualsAndHashCode
@Data
@Entity
@AllArgsConstructor
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class Product implements Serializable {

    private static final long serialVersionUID = 6141353065320670470L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String title;

    private String brand;

    private String description;

    private BigDecimal price;

    private int stock;

    @CreatedBy
    @OneToOne(cascade = CascadeType.ALL)
    private User createdUser;

    @LastModifiedBy
    @OneToOne(cascade = CascadeType.ALL)
    private User modifiedUser;

    @CreatedDate
    private LocalDateTime createdTime;

    @LastModifiedDate
    private LocalDateTime modifiedTime;
}
```



#### 3.3.2 事务

默认情况下，Spring Data JPA提供的CRUD方法都添加了事务，这里的事务使用的是Spring的事务管理机制。对于读操作来说，事务的`readOnly`属性是设置的`true`（默认值是false），而其他操作都是设置的一个空的`@Transactional`注解，所以使用的都是Spring事务的默认配置。

![image-20220510173811454](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220510173811454.png)

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	/**
	 * Alias for {@link #transactionManager}.
	 * @see #transactionManager
	 */
	@AliasFor("transactionManager")
	String value() default "";

	/**
	 * A <em>qualifier</em> value for the specified transaction.
	 * <p>May be used to determine the target transaction manager, matching the
	 * qualifier value (or the bean name) of a specific
	 * {@link org.springframework.transaction.TransactionManager TransactionManager}
	 * bean definition.
	 * @since 4.2
	 * @see #value
	 * @see org.springframework.transaction.PlatformTransactionManager
	 * @see org.springframework.transaction.ReactiveTransactionManager
	 */
	@AliasFor("value")
	String transactionManager() default "";

	/**
	 * Defines zero (0) or more transaction labels.
	 * <p>Labels may be used to describe a transaction, and they can be evaluated
	 * by individual transaction managers. Labels may serve a solely descriptive
	 * purpose or map to pre-defined transaction manager-specific options.
	 * <p>See the documentation of the actual transaction manager implementation
	 * for details on how it evaluates transaction labels.
	 * @since 5.3
	 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute#getLabels()
	 */
	String[] label() default {};

	/**
	 * The transaction propagation type.
	 * <p>Defaults to {@link Propagation#REQUIRED}.
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#getPropagationBehavior()
	 */
	Propagation propagation() default Propagation.REQUIRED;

	/**
	 * The transaction isolation level.
	 * <p>Defaults to {@link Isolation#DEFAULT}.
	 * <p>Exclusively designed for use with {@link Propagation#REQUIRED} or
	 * {@link Propagation#REQUIRES_NEW} since it only applies to newly started
	 * transactions. Consider switching the "validateExistingTransactions" flag to
	 * "true" on your transaction manager if you'd like isolation level declarations
	 * to get rejected when participating in an existing transaction with a different
	 * isolation level.
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#getIsolationLevel()
	 * @see org.springframework.transaction.support.AbstractPlatformTransactionManager#setValidateExistingTransaction
	 */
	Isolation isolation() default Isolation.DEFAULT;

	/**
	 * The timeout for this transaction (in seconds).
	 * <p>Defaults to the default timeout of the underlying transaction system.
	 * <p>Exclusively designed for use with {@link Propagation#REQUIRED} or
	 * {@link Propagation#REQUIRES_NEW} since it only applies to newly started
	 * transactions.
	 * @return the timeout in seconds
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#getTimeout()
	 */
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	/**
	 * The timeout for this transaction (in seconds).
	 * <p>Defaults to the default timeout of the underlying transaction system.
	 * <p>Exclusively designed for use with {@link Propagation#REQUIRED} or
	 * {@link Propagation#REQUIRES_NEW} since it only applies to newly started
	 * transactions.
	 * @return the timeout in seconds as a String value, e.g. a placeholder
	 * @since 5.3
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#getTimeout()
	 */
	String timeoutString() default "";

	/**
	 * A boolean flag that can be set to {@code true} if the transaction is
	 * effectively read-only, allowing for corresponding optimizations at runtime.
	 * <p>Defaults to {@code false}.
	 * <p>This just serves as a hint for the actual transaction subsystem;
	 * it will <i>not necessarily</i> cause failure of write access attempts.
	 * A transaction manager which cannot interpret the read-only hint will
	 * <i>not</i> throw an exception when asked for a read-only transaction
	 * but rather silently ignore the hint.
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#isReadOnly()
	 * @see org.springframework.transaction.support.TransactionSynchronizationManager#isCurrentTransactionReadOnly()
	 */
	boolean readOnly() default false;

	/**
	 * Defines zero (0) or more exception {@linkplain Class classes}, which must be
	 * subclasses of {@link Throwable}, indicating which exception types must cause
	 * a transaction rollback.
	 * <p>By default, a transaction will be rolled back on {@link RuntimeException}
	 * and {@link Error} but not on checked exceptions (business exceptions). See
	 * {@link org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn(Throwable)}
	 * for a detailed explanation.
	 * <p>This is the preferred way to construct a rollback rule (in contrast to
	 * {@link #rollbackForClassName}), matching the exception type, its subclasses,
	 * and its nested classes. See the {@linkplain Transactional class-level javadocs}
	 * for further details on rollback rule semantics and warnings regarding possible
	 * unintentional matches.
	 * @see #rollbackForClassName
	 * @see org.springframework.transaction.interceptor.RollbackRuleAttribute#RollbackRuleAttribute(Class)
	 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn(Throwable)
	 */
	Class<? extends Throwable>[] rollbackFor() default {};

	/**
	 * Defines zero (0) or more exception name patterns (for exceptions which must be a
	 * subclass of {@link Throwable}), indicating which exception types must cause
	 * a transaction rollback.
	 * <p>See the {@linkplain Transactional class-level javadocs} for further details
	 * on rollback rule semantics, patterns, and warnings regarding possible
	 * unintentional matches.
	 * @see #rollbackFor
	 * @see org.springframework.transaction.interceptor.RollbackRuleAttribute#RollbackRuleAttribute(String)
	 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn(Throwable)
	 */
	String[] rollbackForClassName() default {};

	/**
	 * Defines zero (0) or more exception {@link Class Classes}, which must be
	 * subclasses of {@link Throwable}, indicating which exception types must
	 * <b>not</b> cause a transaction rollback.
	 * <p>This is the preferred way to construct a rollback rule (in contrast to
	 * {@link #noRollbackForClassName}), matching the exception type, its subclasses,
	 * and its nested classes. See the {@linkplain Transactional class-level javadocs}
	 * for further details on rollback rule semantics and warnings regarding possible
	 * unintentional matches.
	 * @see #noRollbackForClassName
	 * @see org.springframework.transaction.interceptor.NoRollbackRuleAttribute#NoRollbackRuleAttribute(Class)
	 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn(Throwable)
	 */
	Class<? extends Throwable>[] noRollbackFor() default {};

	/**
	 * Defines zero (0) or more exception name patterns (for exceptions which must be a
	 * subclass of {@link Throwable}) indicating which exception types must <b>not</b>
	 * cause a transaction rollback.
	 * <p>See the {@linkplain Transactional class-level javadocs} for further details
	 * on rollback rule semantics, patterns, and warnings regarding possible
	 * unintentional matches.
	 * @see #noRollbackFor
	 * @see org.springframework.transaction.interceptor.NoRollbackRuleAttribute#NoRollbackRuleAttribute(String)
	 * @see org.springframework.transaction.interceptor.DefaultTransactionAttribute#rollbackOn(Throwable)
	 */
	String[] noRollbackForClassName() default {};

}
```



#### 3.3.3 Lock

```java
interface UserRepository extends Repository<User, Long> {

  @Lock(LockModeType.READ)
  List<User> findByLastname(String lastname);
}
```
锁的类型

```java
public enum LockModeType
{
    READ, // 与下面的OPTIMISTIC同义

    WRITE, // 与下面的OPTIMISTIC_FORCE_INCREMENT同义

    OPTIMISTIC, //读操作时不会更新Version字段的值，只有在写操作的时候会

    OPTIMISTIC_FORCE_INCREMENT, //在读和写操作时都会更新Version字段的值

    PESSIMISTIC_READ,

    PESSIMISTIC_WRITE,

    PESSIMISTIC_FORCE_INCREMENT,

    NONE // 无锁
}
```

推荐阅读：https://www.byteslounge.com/tutorials/locking-in-jpa-lockmodetype

#### 3.3.4 实体关联

1. 一对一的关系，jpa 使用的注解是 @OneToOne
2. 一对多的关系，jpa 使用的注解是 @OneToMany
3. 多对一的关系，jpa 使用的注解是 @ManyToOne
4. 多对多的关系，jpa 使用的注解是 @ManyToMany

#### 3.3.5 查询结果

##### 1）Optional

#####  2）流查询结果

```java
Stream<User> findAllByCustomQueryAndStream();

// 一个数据流可能包裹底层数据存储特定资源，因此在使用后必须关闭。 你也可以使用close()方法或者JAVA 7 try-with-resources区块手动关闭数据流。
 try(Stream<User stream = repository.findAllByCustomQueryAndStream()){

 stream.forEach(...);

 }
```

##### 3）异步查询结果

```java
@Async
Future<User> findByFirstname(String firstname);  (1)  

@Async
CompletableFuture<User> findOneByFirstname(String firstname);  (2)

@Async
ListenableFuture<User> findOneByLastname(String lastname);     (3)

(1) 使用 java.util.concurrent.Future 作为返回类型

(2) 使用 Java 8 java.util.concurrent.CompletableFuture 作为返回类型

(3) 使用 org.springframework.util.concurrent.ListenableFuture 作为返回类型
```