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

##### Using JDBC without Springboot

![JDBCWithoutSpringboot](https://github.com/DharaniDJ/spring-boot-daily-learnings/blob/assets/JDBCWithoutSpringboot.png)

I have created one public class database connection as the class name. I have created one method called `getConnection` and its returning me the connection object. The first step is to load the driver into the JVM. Now once the driver is loaded into the JVM, the second step is to establish a connection with DB.

I also created another class `UserDAO` in which I wrote `createUserTable`, `createUser`, `readUsers`. First thing you need is get a connection of a database. So it's creating a new database connection object and calling the getConnection method. Once you get the connection then you can run the query.

If any exception comes, you have to handle the exception and in finally block you have to close the statement and connection.
15:05
#### 2. Plain JDBC Handling
- To use JDBC, you need to load the database driver, establish a connection, create statements, execute queries, and handle results.
- Example:
  ```java
  Class.forName("com.mysql.jdbc.Driver");
  Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "password");
  Statement statement = connection.createStatement();
  ResultSet resultSet = statement.executeQuery("SELECT * FROM users");
  while (resultSet.next()) {
      // Process results
  }
  resultSet.close();
  statement.close();
  connection.close();
  ```

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
  spring.datasource.password=password
  ```

#### 5. Spring Boot JDBC Helper Methods
- **JdbcTemplate**: Provides methods for executing SQL queries, updates, and batch operations.
  - **Execute**: For executing DDL statements.
    ```java
    jdbcTemplate.execute("CREATE TABLE users (id INT, name VARCHAR(50))");
    ```
  - **Update**: For executing DML statements (INSERT, UPDATE, DELETE).
    ```java
    jdbcTemplate.update("INSERT INTO users (id, name) VALUES (?, ?)", 1, "John");
    ```
  - **Query**: For executing SELECT statements and mapping results.
    ```java
    List<User> users = jdbcTemplate.query("SELECT * FROM users", new RowMapper<User>() {
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            return user;
        }
    });
    ```

- **Exception Handling**: Spring translates SQL exceptions into more specific exceptions (e.g., `DataIntegrityViolationException`).

- **Connection Pooling**: Spring Boot uses HikariCP by default for connection pooling, which can be configured in `application.properties`.

These notes provide a comprehensive overview of JDBC and how Spring Boot simplifies its usage with `JdbcTemplate`.