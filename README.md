# SpringBoot学习笔记（三）：Jpa基本使用

[Spring Data JPA 参考指南](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)

## 一、Spring Boot Jpa 介绍

### Jpa 是什么？

- Jpa (Java Persistence API) 是 Sun 官方提出的 Java 持久化规范。它为 Java 开发人员提供了一种对象/关联映射工具来管理 Java 应用中的关系数据。

- Spring Data JPA为Java Persistence API（JPA）提供了存储库支持。它简化了需要访问JPA数据源的应用程序的开发。

## 二、pom.xml引入Maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 基本查询

- Spring Data 默认实现
- 根据查询的方法来自动解析成 SQL。

### 预先生成方法

Spring Boot Jpa 默认预先生成了一些基本的CURD的方法，例如：增、删、改等等

1 继承 JpaRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

2 使用默认方法

```java
@Test
public void testBaseQuery() throws Exception {
	User user=new User();
	userRepository.findAll();
	userRepository.findOne(1l);
	userRepository.save(user);
	userRepository.delete(user);
	userRepository.count();
	userRepository.exists(1l);
	// ...
}
```

### 自定义简单查询

自定义的简单查询就是根据方法名来自动生成 SQL，主要的语法是`findXXBy`,`readAXXBy`,`queryXXBy`,`countXXBy`, `getXXBy`后面跟属性名称：

```java
User findByUserName(String userName);
```

也使用一些加一些关键字`And `、 `Or`

```java
User findByUserNameOrEmail(String username, String email);
```

修改、删除、统计也是类似语法

```java
Long deleteById(Long id);
Long countByUserName(String userName)
```

基本上 SQL 体系中的关键词都可以使用，例如：` LIKE `、 `IgnoreCase`、 `OrderBy`。

```java
List<User> findByEmailLike(String email);
User findByUserNameIgnoreCase(String userName);
List<User> findByUserNameOrderByEmailDesc(String email);
```

**具体的关键字，使用方法和生产成SQL如下表所示**

| Keyword           | Sample                                  | JPQL snippet                                                 |
| :---------------- | :-------------------------------------- | :----------------------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                                           |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1                              |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %)   |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                                      |
| TRUE              | findByActiveTrue()                      | … where x.active = true                                      |
| FALSE             | findByActiveFalse()                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)                        |

## 复杂查询

### 分页查询

分页查询在实际使用中非常普遍了，Spring Boot Jpa 已经帮我们实现了分页的功能，在查询的方法中，需要传入参数`Pageable` ,当查询中有多个参数的时候`Pageable`建议做为最后一个参数传入.

```java
Page<User> findALL(Pageable pageable);
Page<User> findByUserName(String userName,Pageable pageable);
```

`Pageable` 是 Spring 封装的分页实现类，使用的时候需要传入页数、每页条数和排序规则

```java
@Test
public void testPageQuery() throws Exception {
	int page=1,size=10;
	Sort sort = new Sort(Direction.DESC, "id");
    Pageable pageable = new PageRequest(page, size, sort);
    userRepository.findALL(pageable);
    userRepository.findByUserName("testName", pageable);
}
```

- 注意下面的说明

```java
//@Deprecated表示不推荐使用
@Deprecated
Pageable pageable = new PageRequest(page, size, sort);
//最好使用下面的方式
Pageable pageable = PageRequest.of(page, size, sort);
```

**限制查询**

有时候我们只需要查询前N个元素，或者支取前一个实体。

```java
User findFirstByOrderByLastnameAsc();
User findTopByOrderByAgeDesc();
Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
List<User> findFirst10ByLastname(String lastname, Sort sort);
List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

### 自定义SQL查询

使用自定义的 SQL 来查询，Spring Data 也是完美支持的；

- 在 SQL 的查询方法上面使用`@Query`注解。
- 如涉及到删除和修改在需要加上`@Modifying`。
- 根据需要添加 `@Transactional`对事物的支持，查询超时的设置等。

```java
//?1表示用方法中的第一个参数。
@Modifying
@Query("update User u set u.userName = ?1 where u.id = ?2")
int modifyByIdAndUserId(String  userName, Long id);
	
@Transactional
@Modifying
@Query("delete from User where id = ?1")
void deleteByUserId(Long id);
  
@Transactional(timeout = 10)
@Query("select u from User u where u.emailAddress = ?1")//注意此时select u而不是select *
User findByEmailAddress(String emailAddress);
```

### 多表查询

多表查询 Spring Boot Jpa 中有两种实现方式

- 利用 Hibernate 的级联查询来实现
- 创建一个结果集的接口来接收连表查询后的结果，这里主要第二种方式。

首先需要定义一个结果集的接口类。

```java
public interface HotelSummary {

	City getCity();

	String getName();

	Double getAverageRating();

	default Integer getAverageRatingRounded() {
		return getAverageRating() == null ? null : (int) Math.round(getAverageRating());
	}

}
```

查询的方法返回类型设置为新创建的接口

```java
@Query("select h.city as city, h.name as name, avg(r.rating) as averageRating "
		- "from Hotel h left outer join h.reviews r where h.city = ?1 group by h")
Page<HotelSummary> findByCity(City city, Pageable pageable);

@Query("select h.name as name, avg(r.rating) as averageRating "
		- "from Hotel h left outer join h.reviews r  group by h")
Page<HotelSummary> findByCity(Pageable pageable);
```

使用

```java
Page<HotelSummary> hotels = this.hotelRepository.findByCity(new PageRequest(0, 10, Direction.ASC, "name"));
for(HotelSummary summay:hotels){
		System.out.println("Name" +summay.getName());
	}
```

> 在运行中 Spring 会给接口（HotelSummary）自动生产一个代理类来接收返回的结果，代码汇总使用 `getXX`的形式来获取

