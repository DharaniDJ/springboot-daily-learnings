### JPA (Part 1) | JDBC Template
We have reached to a stage where we have to insert data into the database, how we can do that. So that's where the JPA, JDBC comes into the picture okay. So before even jumping into the JPA, we have to first understand the complete sequence.

Let's say you are writing a springboot application. You have an application logic or you have written so many beans okay. Now what you do is if you have to use JPA, you have to interact with the APIs, which are provided by JPA. so always remember that JPA is mostly an interface, which means it does not provide you with the implementation. Implementation is provided by `Hibernate` or `openJPA`.

So your application talks to JPA, and JPA internally has to be implemented be some other component which you decide. Generally Hibernate is the most popular one. so hibernate internally utilize JDBC APIs(interface). And then there are specific DB drivers. so this DB drivers are nothing but an implementation of this JDBC APIs.

Lets say you are using a MySQL DB, then there would be a specific DB driver for it. Let's say it is `Connector/J` which is very specific for MySQL which knows how to connect to MySQL.

![JPAFlow](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JPAFlow.png)

Now there is something called `ORM Framework` (Object Relational Mapping), it is nothing but a bridge between Java object and your relational databases. So remember JPA, JDBC are just for maintaining relational databases.

#### 1. Introduction to JDBC
- JDBC (Java Database Connectivity) is an API interface that allows Java applications to interact with databases.
- It provides methods to connect to a database, execute queries, and process the results.
- JDBC is an interface, and the actual implementation is provided by database-specific drivers (e.g., MySQL, PostgreSQL).

- For example: 
  1. MySQL
      - Driver : Connector/J
      - Class : com.mysql.cj.jdbc.Driver
  2. PostgreSQL
      - Driver : PostgreSQL JDBC Driver
      - Class : org.postgresql.Driver
  3. H2(in-memory)
      - Driver : H2 Database Engine
      - Class : org.h2.Driver

#### 2. Plain JDBC Handling

![JDBCWithoutSpringboot](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JDBCWithoutSpringboot.png)

I have created one public class database connection as the class name. I have created one method called `getConnection` and its returning me the connection object. The first step is to load the driver into the JVM. Now once the driver is loaded into the JVM, the second step is to establish a connection with DB.

I also created another class `UserDAO` in which I wrote `createUserTable`, `createUser`, `readUsers`. First thing you need is get a connection of a database. So it's creating a new database connection object and calling the getConnection method. Once you get the connection then you can run the query.

If any exception comes, you have to handle the exception and in finally block you have to close the statement and connection.

If we see above example:
- Connection
- Statement
- PreparedStatement
- ResultSet etc.
All are interfaces which JDBC provide and each specific driver provide the implementation for it.

#### 3. Problems with Plain JDBC
- **Boilerplate Code**: Requires repetitive code for loading drivers, creating connections, and handling exceptions.
- **Exception Handling**: Throws generic `SQLException` which needs to be parsed for specific errors.
- **Resource Management**: Manual closing of connections, statements, and result sets to avoid memory leaks.
- **Connection Pooling**: No built-in support for connection pooling, leading to potential performance issues.

#### 4. Spring Boot JDBC
- Spring Boot simplifies JDBC usage with `JdbcTemplate`, which handles boilerplate code and resource management.
- **Dependencies**:
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
  </dependency>
  ```
- **Configuration**:
  ```properties
  spring.datasource.url=jdbc:h2:mem:testdb
  spring.datasource.driverClassName=org.h2.Driver
  spring.datasource.username=sa
  spring.datasource.password=
  spring.h2.console.enabled=true
  ```
[JDBCWithSpringBoot](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JDBCWithSpringBoot.png)
[JDBCWithSpringBoot1](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JDBCWithSpringBoot1.png)

- **Explanation**:
I have created a basic POJO of `User` which has `userId`, `userName`, `age`. Second, I have created `UserRepository` class annotated with `@Repository`. Unlike Plain JDBC, when an SQL exception comes, it gives a granular level of exception. In the `UserRepository` class, I've added a dependency of JdbcTemplate which helps use to remove all the boiler code. I also have the same methods I have written previously. One is to create table, one is insert user and another is read user. Here I haven't creating DB Connection. I've not written any finally block where I am closing the DB connection or closing the prepared statement. All I'm using is just the JDBC Template execute to execute the plain query.

  I've written `UserService` in which i've `userRepository` as dependency. This class is nothing but a service class calling the repository class and it is your business logic class.

#### 5. Spring Boot JDBC Helper Methods

[HikariCP](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/HikariCP.png)

[AppConfigDataSource](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/AppConfigDataSource.png)

[JDBCTemplateMethods](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JDBCTemplateMethods.png)
