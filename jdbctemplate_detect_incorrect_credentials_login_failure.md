## JDBCTemplate : Detect Incorrect Credentials / Login Failure

Detecting an incorrect password while using **`JdbcTemplate`** in a Spring Boot application typically involves handling exceptions that occur when a database connection fails due to authentication issues. Here's how you can effectively handle and detect an incorrect password when using **`JdbcTemplate`**.

### 1. Setup JdbcTemplate
Ensure that you have a **`JdbcTemplate`** configured properly in your Spring Boot application. You can define a **`JdbcTemplate`** bean in your configured class:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import javax.sql.DataSource;

@Configuration
public class DatabaseConfig {

  @Bean
  public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setUrl("url");
    dataSource.setUsername("username");
    dataSource.setPassword("password");
    return dataSource;
  }

  @Bean
  public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
  }
}
```

### 2. Executing a Query with Exception Handling
When you execute a query using **`JdbcTemplate`**, you can catch exceptions to determine if the password is incorrect. The **`org.springframework.jdbc.BadSqlGrammerException`** or **`org.springframework.dao.InvalidDataAccessResourceUsageException`** can indicate an authentication error, depending on your database.

Here's an example of how to handle this:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

@Service
public class UserService {

}
```
