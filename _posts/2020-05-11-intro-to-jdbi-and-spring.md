---
title: "Intro to Using Jdbi3 in Spring Boot"
categories: 
    -   Software
tags:
    -   spring boot
    -   jdbi

excerpt_separator: <!--more-->
---
![jdbi]({{site.url}}{{site.baseurl}}/assets/blog/2020-03-04-intro-to-jdbi-and-spring/jdbi-logo.svg)

If you are a java developer and have ever done any application development, chances are you have encountered the need to add a persistence layer to your app. <!--more--> For many years, this meant the inclusion of heavy ORM such as Hibernate/JPA or some similar library and all of the quirks and challenges that went along with it. A year ago, a colleague of mine directed me to `Jdbi` as a new (for me) approach to working with data, and since then I have not turned back.

Jdbi sits comfortably between ORM libraries and low level `JDBC` Drivers as an intuitive, concise and lightweight library to interact with your persistence layer. Jdbi does not provide any entity management, intermediary services, or magic to manage your data. Additionally it does not provide automatic query composition (like from [QueryDSL](https://www.querydsl.com) or `spring-data-jpa`), DDL generation similar to Hibernate, or even container managed transactions which are hallmark of java application servers. 

At this point, you may be asking yourself, "Why would I sacrifice all of these fancy and useful features"? While there are many answers to this question, there are a few things which really drew me to Jdbi which are highlighted below:

- Jdbi has both a declarative and a fluent API. Both are incredibly easy to use, allow you to write clean and concise code, and leave little to the imagination about what is actually happening under the hood 
- Closures! One of my biggest complaints with using Hibernate is that connection boundaries are often hidden by the application server. Jdbi completely solves this with the ability to use Closures to explicitly define a connection lifecycle
- Easy automatic mapping of rows and columns into Beans where convenient, but the power to easily define custom behaviour when needed.
- Use native Joins, selects and complex queries and return whatever you want! Since your data model is not represented as code, you are not beholden to it when returning query results.  New views of data can easily be constructed and mapped into custom beans with little to no effort
- And finally since there is no middle layer managing your data, it is much more performant then Many ORMs!


# Building a Simple Application Using Jdbi And Spring Boot

The following small tutorial will outline some easy steps to get you started with using Jdbi within your own spring project. Both tools are incredibly feature rich, and this tutorial will not dive into the more powerful features of each, but just provide an overview of setup. Stay tuned for future posts for more information! 

## Add Jdbi Dependencies to your Project

Before you begin, you will need to add several Jdbi dependencies to your `pom.xml`. For the sake of this tutorial we will use my favorite database: [Postgres](https://www.postgresql.org/). If you need help creating a database for this example, you can refer to the [official documentation](https://www.postgresql.org/docs/9.1/tutorial-createdb.html) for Postgres. 

```xml
<dependencies>
    <!-- Additional Spring Dependency -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jdbc</artifactId>
    </dependency>
    <!-- Jdbi Dependencies -->
    <dependency>
        <groupId>org.jdbi</groupId>
        <artifactId>jdbi3-core</artifactId>
        <version>3.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.jdbi</groupId>
        <artifactId>jdbi3-sqlobject</artifactId>
        <version>3.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.jdbi</groupId>
        <artifactId>jdbi3-postgres</artifactId>
        <version>3.6.0</version>
    </dependency>
</dependencies>
```

Adding the `spring-boot-starter-data-jdbc` library is a lightweight way to get all of the handy spring-boot auto configuration for a `DataSource` without adding unnecessary dependencies to your project. Since `Jdbi` is just a wrapper around `JDBC` we are not adding anything which will not be used

## Setting up a DataSource

Before we start using our application, we will need to tell Spring where the database is which we would like it to use. The simplest way to achieve this is to add a few entries to our `src/main/resources/application.yml`. Generally, I will hard-code the values I need to develop locally in the `application.yml` then either provide a `prod` profile, or take advantage of springs [externalized configuration model](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config).

```yaml
spring:
  datasource:
    # I previously created a "Role" for my postgres database for this tutorial 
    username: jdbi-example-spring-boot
    # When developing locally , I tend not to use passwords for ease of use
    password: ""
    url: jdbc:postgresql://localhost/jdbi-example-spring-boot
    driver-class-name: org.postgresql.Driver
```



## Configure Jdbi Bean

We are going to want to make Jdbi available to the application aTEstingTs a bean which can be autowired into whatever services need it.To do this, we can create a new configuration class and to register the necessary bean.

```java
@Configuration
public class JdbiConfiguration {

    @Bean
    public Jdbi jdbi(DataSource datasource){
        return Jdbi.create(dataSource)
            .installPlugin(new PostgresPlugin())
            .installPlugin(new SqlObjectPlugin());
    }
}
```

Thanks to `spring-boot-starter-data-jdbc` there is no need to configure a DataSource ourselves, instead, we allow spring to inject a previously defined `DataSource` bean directly into the  `jdbi()` Bean definition method. With  Finally, we create the `Jdbi` bean and make sure to initialize the correct plugins, allowing `Jdbi` to understand the specific data types and operations (ie Postgres `jsonb` data) used by Postgres.

## Create a POJO

We will want to create a simple POJO to represent our data. Its important to note, that this does not need to correspond at all with our data model, but can contain anything that is useful for our needs without changing the data model. Jdbi will not auto generate any data model as in Hibernate. Please note, the `@Data` annotation is from [lombok](https://projectlombok.org/), a handy library for ANY java developer!

```java
@Data
public class User {
    private Long id;
    private String firstName;
    private String lastName;
    private String phoneNumber 
}
```

## Define a Declarative DAO

I am a big fan of Jdbi's declarative approach to defining persistence interactions. This approach uses interfaces with a combination of annotations which Jdbi can then use to generate an implementation of a DAO for you. By using the declarative approach you have a clear picture of exactly what is happening when jdbi executes your code.


```java
public interface UserDao {

    @Transaction
    @SqlUpdate("CREATE TABLE IF NOT EXISTS users(id BIGINT NOT NULL PRIMARY KEY, first_name VARCHAR(48), last_name VARCHAR(48), phone_number VARCHAR(48))")
    void createUserTable();

    @Transaction
    @SqlUpdate("INSERT INTO users(id,first_name,last_name,phone_number) VALUES(:id,:firstName,:lastName,:phoneNumber)")
    void createUser(@BindBean User user);

    @SqlQuery("SELECT * FROM users")
    @RegisterBeanMapper(User.class)
    List<User> getUsers();

    @SqlQuery("SELECT * FROM users WHERE id = :id")
    @RegisterBeanMapper(User.class)
    User getUser(@Bind("id") Long id);

}
```

There is a lot happening here, however from the annotations it is easy to understand exactly what each method is trying to achieve. The `@Transaction` annotation tells Jdbi to wrap a specific method call in a transaction. There are additional parameters that you can pass to this annotation in order to modify the transaction lifecycle, however for our purposes the default value is fine. The two annotation `@SqlQUery` and `@SqlUpdate` are very similar in appearance, both taking an `SQL` string as a parameter, however they dictate very different behaviours. `@SqlUpdate` is used for defining actions which change data in some way. This may be through a `SET`, `INSERT`, `DELETE`, `ALTER`  etc,  operation. The `@SqlQuery` annotation on the other hand, cannot modify the data in any way, but may only be used for retrieving data.

With Jdbi you can provide a parameterized `SQL` string and  bind method arguments when the `SQL` is executed. There are two methods demonstrated above (however Jdbi provides many more binding approaches) `@Bind` and `@BindBean`. The `@Bind` annotation maps a method argument to a specific parameter in the `SQL`, whereas the `@BindBean` annotation will use the `getters` of a bean to bind all of its properties to matching parameters in the `SQL`.

Finally, Jdbi provides simple approaches for mapping rows (even joined rows), into a desired bean or primitive type. The above example uses the `@RegisterBeanMapper(User.class)` annotation to tell Jdbi to convert the returned row into a `User` object, using the setter's that are present. Its important to note, that this does not Proxy the `User` class like Hibernate does, and the object you get back is a true POJO. If you require more control over how a bean is mapped Jdbi offers many additional strategies for doing row, column and collection level mapping.

## Setup A Controller 

```java
@RestController
public class UserController {

    private Jdbi jdbi;

    public UserController(Jdbi jdbi){
        this.jdbi = jdbi;
        jdbi.useExtension(UserDao.class,UserDao::createUserTable);
    }

    @PostMapping("/users")
    public User createUser(@RequestBody User user){
        user.setId(System.currentTimeMillis());
        jdbi.useExtension(UserDao.class, dao -> dao.createUser(user));
        return user;
    }

    @GetMapping("/users")
    public List<User> getUsers(){
        return jdbi.withExtension(UserDao.class, UserDao::getUsers);
    }

    @GetMapping("/users/{id}")
    public User getUsers(@PathVariable  Long id){
        return jdbi.withExtension(UserDao.class, dao -> dao.getUser(id));
    }

}
```

In the controller we are interacting with `Jdbi` using closures. In my opinion, this is a great approach for interacting with a database: it explicitly defines connection boundaries, it separates persistence logic from the service layer and it guarantees there are no side effects outside of the closure. Its clear, concise and clean code that will make your life easier.

There are two specific methods which we are using here: `withExtension` and `useExtension` which takes as a parameter the `Dao` interface to use as an "extension", and then it accepts closure passing in the `dao` instance it created from our interface. `withExtension` provides a way to return a value, whereas `useExtension` allows us to simply run something against the database. `Jdbi` of course provides additional ways that retrieve and update data, for a full breakdown please refer to the official documentation.

## Create your Spring Boot Application

```java
@SpringBootApplication
public class JdbiExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(JdbiExampleApplication.class, args);
    }
}
```

## Testing your API

Now that you have your Spring application setup, you are ready to test it out! You can issue a few simple curl requests to make sure that your REST-API is ready.

#### Create a New User
```bash
curl -XPOST -H 'Content-Type: application/json' 'http://localhost:8080/users' -d '{"firstName":"Joe","lastName":"Shmo","phoneNumber":"714-832-2211"}'
{
    "id":1589208108922,
    "firstName":"Maria",
    "lastName":"Magee",
    "phoneNumber":"676332415"
}
```

#### Retrieve All Users
```bash
curl 'http://localhost:8080/users'
[
    {
        "id":1589208108922,
        "firstName":"Maria",
        "lastName":"Magee",
        "phoneNumber":"676332415"
    }
]
```
#### Retrieve a Single User
```bash
curl 'http://localhost:8080/users/1589208108922'
{
    "id":1589208108922,
    "firstName":"Maria",
    "lastName":"Magee",
    "phoneNumber":"676332415"
}
```

# Closing

At this point, you should understand how to setup a simple spring-boot application with `Jdbi` powering the persistence layer! If you are interested in learning more about the amazing features that `Jdbi` offers, you can check out the [official documentation](https://jdbi.org/). If you would like to follow along, you can find all of the source code of this tutorial on [GitHub](https://github.com/patmagee/jdbi-tutorial-examples/tree/master/jdbi-example-sprint-boot)