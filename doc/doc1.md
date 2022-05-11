## 1. Spring Data JPA介绍
### 1.1 Spring Data

Spring Data 的使命是为数据访问提供熟悉且一致的、基于 Spring 的编程模型，同时仍保留底层数据存储的特殊特征。

它使得使用数据访问技术、关系和非关系数据库、map-reduce 框架和基于云的数据服务变得容易。这是一个总括项目，其中包含许多特定于给定数据库的子项目。这些项目是通过与这些令人兴奋的技术背后的许多公司和开发人员合作开发的。

**Spring子项目使用占比**

<img src="https://haopeng.oss-cn-beijing.aliyuncs.com/blogblogimage-20220509194649583.png" style="zoom: 80%;" />

### 1.2 什么是Spring Data JPA

![image-20220509162437172](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220509162437172.png)

#### **JPA**

JPA是JDK5.0新增的协议，通过持久层注解（@Entity）来描述对象和数据库的表映射关系。将Java项目运行期的实体对象，通过一种Session持久化到数据库中去。

**JPA主要涵盖以下三方面的功能:**

- 1)ORM映射元数据：JPA支持XML和注解两种元数据形式来描述定义实体与表之前的关系
- 2)通用API：通过操作对象实体来操作CRUD，框架屏蔽了jdbc的复杂性
- 3)JPQL 通过面向对象，而非面向数据库的查询语言来查询数据，避免程序与sql的耦合

#### **Spring Data JPA**

为 Java Persistence API (JPA) 提供存储库支持。它简化了需要访问 JPA 数据源的应用程序的开发。

 Jpa 就是定义了一系列标准，让实体类和数据库中的表建立一个对应的关系，当我们在使用 java 操作实体类的时候能达到操作数据库中表的效果(不用写sql ,就可以达到效果），jpa 的实现思想即是 ORM （Object Relation Mapping），对象关系映射，用于在关系型数据库和业务实体对象之间作一个映射。

jpa 并不是一个框架，是一类框架的总称，持久层框架 Hibernate 是 jpa 的一个具体实现，本文要谈的 spring data jpa 又是在 Hibernate 的基础之上的封装实现。

**总结：**提供通用接口，和模板类。屏蔽存储介质差异。

### 1.3 特性

- Sophisticated support to build repositories based on Spring and JPA
- Support for [Querydsl](http://www.querydsl.com/) predicates and thus type-safe JPA queries
- Transparent auditing of domain class
- Pagination support, dynamic query execution, ability to integrate custom data access code
- Validation of `@Query` annotated queries at bootstrap time
- Support for XML based entity mapping
- JavaConfig based repository configuration by introducing `@EnableJpaRepositories`.