### JPA | JDBC Template Notes

#### 1. Introduction to JDBC
- JDBC (Java Database Connectivity) is an API that allows Java applications to interact with databases.
- It provides methods to connect to a database, execute queries, and process the results.
- JDBC is an interface, and the actual implementation is provided by database-specific drivers (e.g., MySQL, PostgreSQL).

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