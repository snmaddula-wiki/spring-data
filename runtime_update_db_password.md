## Handle DataSource Password Changes during Runtime

To handle password changes during runtime in a Spring Boot application with a datasource, you'll need a way to update the datasource configuration dynamically.

### 1. Use a `DataSource` Bean
Instead of relying on the default auto-configuration, define your **`DataSource`** as a bean. This allows you to have more control over its lifecycle.

### 2. Create a Custom DataSource Configuration
You can create a configuration class that manages the datasource and listens for password changes.

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

@Configuration
public class DataSourceConfig {

  @Value("${spring.datasource.url}")
  private String url;

  @Value("${spring.datasource.username}")
  private String username;

  private String password;

  @Bean
  public DataSource dataSource() {
    return createDataSource();
  }

  private DataSource createDataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setUrl(url);
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    return dataSource;
  }

  public void updatePassword(String newPassword) {
    this.password = newPassword;
    ((DriverManagerDataSource) dataSource()).setPassword(newPassword);
  }
}
```

### 3. Create a Service to Manage Password Updates
You'll need a service that will handle password updates, including fetching the new password and notifying the **`DataSourceConfig`**:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.sterotype.Service;

@Service
public class PasswordService {

  @Autowired
  private PasswordFetcher passwordFetcher; // Assume that this is the service to fetch password from external source.

  @Autowired
  private DataSourceConfig dataSourceConfig;

  public void refreshPassword() {
    String newPassword = passwordFetcher.fetchPassword();
    dataSourceConfig.updatePassword(newPassword);
  }
}
```

### 4. Schedule Password Refresh (Optional)
If you expect the password to change at regular intervals, consider using a scheduled task to refresh it.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class PasswordScheduler {

  @Autowired
  private PasswordService passwordService;

  @Scheduled(fixedRate = 36_00_000) // Every hour
  public void schedulePasswordRefresh() {
    passwordService.refreshPassword();
  }
}
```

### 5. Ensure Proper Exception Handling
Make sure to handle exceptions n both the **`PasswordService`** and **`PasswordScheduler`** to avoid crashes due to unreachable URLs or parsing errors.

### 6. Testing the Setup
- **Unit Tests:** Write tests for the **`PasswordService`** and **`DataSourceConfig`** to ensure they behave as expected when the password changes.
- **Integration Tests:** Run integration tests to check if the application can still connect to the database with the new password.

### 7. Considerations
- **Connection Pooling:** If you're using a connection pool (like HikariCP), consider how the pool will manage connections when the password changes. You may need to close existing connections and create new ones.
- **Error Handlng:** Make sure to include robust error handling to manage scenarios where the password fetch fails.
- **Performance:** Be mindful of the performance impact of frequent password checks, especially if the external URL is slow or unreliable.

### Example of External Password Fetching Service
Here's a simple example of how your **`PasswordFetcher`** might look:

```java
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class PasswordFetcher {

  private final String passwordUrl = "http://example.com/api/password"; // Replace with actual url

  public String fetchPassword() {
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate.getForObject(passwordUrl, String.class);
  }
}
```

### Conclusion
With the above setup, your Spring Boot application can dynamically update the datasource password at runtime without requiring a restart. This solution is flexible and allows you to adapt to changing credentials while maintaining database connectivity.
