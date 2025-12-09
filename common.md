---
trigger: always_on

---

# Java Enterprise Development Standards v1.0

**Scope**: Java Web Application (Spring Boot / Jakarta EE)  
**Enforcement**: Strict for core conventions (1-9), recommended for advanced patterns (10-12)  
**AI Instruction**: All code generation MUST follow these standards. When uncertain, prefer clarity and consistency over brevity.

---

## 1. NAMING CONVENTIONS

### 1.1 Classes

- **Style**: PascalCase (noun)
- **Rule**: Descriptive nouns; avoid acronyms unless universally known (e.g., URL, HTTP)
- **Examples**:
    - ✅ `UserAccount`, `OrderProcessor`, `EmailValidator`
    - ❌ `UserAccMgr`, `Processor`, `Helper`, `Manager`, `Util`

### 1.2 Interfaces

- **Style**: PascalCase (adjective ending in "able" where possible)
- **Rule**: Describe behaviors, not implementations
- **Examples**:
    - ✅ `Serializable`, `Comparable`, `UserRepository`, `PaymentService`
    - ❌ `IUser`, `UserInt`, `AbstractUser`

### 1.3 Methods

- **Style**: camelCase (verb or verb phrase)
- **Rule**: Boolean methods start with `is`, `has`, `can`, `should`
- **Examples**:
    - ✅ `calculateTotal()`, `getUserById(Long id)`, `isActive()`, `hasPermission()`
    - ❌ `total()`, `user(Long id)`, `active()`, `check()`

### 1.4 Variables & Fields

- **Style**: camelCase (noun)
- **Rule**: Descriptive; avoid single letters except loop counters (`i`, `j`, `k`)
- **Examples**:
    - ✅ `userName`, `orderCount`, `isAuthenticated`
    - ❌ `un`, `cnt`, `flag`, `temp`

### 1.5 Constants

- **Style**: UPPER_SNAKE_CASE
- **Rule**: `static final` fields; grouped in dedicated classes or enums
- **Examples**:
    - ✅ `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS`, `API_BASE_URL`
    - ❌ `maxRetry`, `timeout`, `url`

### 1.6 Packages

- **Style**: lowercase, reverse domain notation
- **Rule**: Organize by feature or layer (avoid deep nesting)
- **Examples**:
    - ✅ `com.company.product.domain.user`, `com.company.product.api.controller`
    - ❌ `com.Company.Product`, `user`, `controllers`

### 1.7 Enum Types

- **Style**: PascalCase (type), UPPER_SNAKE_CASE (values)

- **Examples**:

  ```java
  public enum OrderStatus {
      PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
  }
  ```

---

## 2. CODE STRUCTURE

### 2.1 Package Organization (Layered Architecture)

```
com.example.campuserrand
├── commons               // Common module
│   ├── config           // Configuration classes
│   ├── constants        // Constant definitions
│   ├── dto              // Data Transfer Objects
│   ├── enums            // Enumeration types
│   ├── exception        // Exception definitions
│   ├── filter           // Filters
│   ├── handler          // Handlers (e.g., exception handlers)
│   ├── pojo             // Plain Old Java Objects
│   ├── security         // Security-related components
│   ├── utils            // Utility classes
│   └── vo               // View Objects
├── controller           // Controller layer
├── entity               // Entity layer (database mapping)
├── mapper               // Data access layer (MyBatis)
└── service              // Business logic layer

```

### 2.2 Class Structure Order

1. Static constants
2. Static fields
3. Instance fields (private, grouped by purpose)
4. Constructors
5. Static factory methods
6. Public methods
7. Protected methods
8. Private methods
9. Nested classes

### 2.3 File Naming

- **Rule**: One public class per file; file name matches class name
- **Test files**: `{ClassName}Test.java` in `src/test/java` (same package structure)

---

## 3. DEPENDENCY INJECTION (Spring Boot)

### 3.1 Constructor Injection (Preferred)

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

### 3.2 Field Injection (Avoid)

```java
// ❌ WRONG: Hard to test, hides dependencies
@Autowired
private UserRepository userRepository;
```

### 3.3 Bean Configuration

- Use `@Configuration` classes for complex bean creation
- Prefer `@Component`, `@Service`, `@Repository`, `@Controller` for auto-detection
- Avoid `@Autowired` on constructors (implicit since Spring 4.3)

---

## 4. DESIGN PATTERNS & BEST PRACTICES

### 4.1 Layering Rules

- **Controllers**: Handle HTTP concerns only (validation, response mapping)
- **Services**: Contain business logic; no direct DB or HTTP access
- **Repositories**: Data access only; return domain objects, not DTOs
- **DTOs**: Transfer data between layers; never expose entities directly

### 4.2 Exception Handling

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}
```

### 4.3 Common Patterns

- **Repository Pattern**: Use Spring Data JPA interfaces
- **Service Layer**: Separate business logic from controllers
- **DTO Pattern**: Convert entities to DTOs in service layer
- **Builder Pattern**: For objects with 4+ parameters (use Lombok `@Builder`)
- **Factory Pattern**: Use `@Bean` methods in `@Configuration` classes

---

## 5. DATA ACCESS (JPA / Spring Data)

### 5.1 Entity Design

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### 5.2 Repository Methods

- Use Spring Data JPA query methods (e.g., `findByEmail`, `existsByUsername`)
- Use `@Query` for complex queries
- Avoid N+1 queries: use `@EntityGraph` or JOIN FETCH

### 5.3 Transaction Management

```java
@Transactional(readOnly = true)
public class UserService {
    @Transactional
    public User createUser(CreateUserRequest request) {
        // Write operation
    }
}
```

---

## 6. REST API CONVENTIONS

### 6.1 URL Structure

- **Style**: kebab-case, plural nouns
- **Examples**:
    - ✅ `GET /api/v1/users`, `POST /api/v1/orders`, `GET /api/v1/users/{id}/orders`
    - ❌ `GET /api/getUsers`, `POST /api/createOrder`, `GET /api/user`

### 6.2 HTTP Methods

- **GET**: Retrieve (idempotent, no side effects)
- **POST**: Create new resource
- **PUT**: Full update (replace entire resource)
- **PATCH**: Partial update
- **DELETE**: Remove resource

### 6.3 Response Status Codes

- **200 OK**: Successful GET/PUT/PATCH
- **201 Created**: Successful POST (include `Location` header)
- **204 No Content**: Successful DELETE
- **400 Bad Request**: Validation error
- **401 Unauthorized**: Missing/invalid authentication
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource doesn't exist
- **500 Internal Server Error**: Unhandled exceptions

---

## 7. VALIDATION

### 7.1 Request Validation

```java
@PostMapping("/users")
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest request) {
    // Spring validates automatically; throws MethodArgumentNotValidException if invalid
}

public class CreateUserRequest {
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}
```

### 7.2 Validation Groups (Optional)

```java
public interface OnCreate {}
public interface OnUpdate {}

public class UserRequest {
    @Null(groups = OnCreate.class)
    @NotNull(groups = OnUpdate.class)
    private Long id;
}
```

---

## 8. TESTING STANDARDS

### 8.1 Test Structure

```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;

    @MockBean
    private UserRepository userRepository;

    @Test
    @DisplayName("Should create user when valid request provided")
    void shouldCreateUserWhenValidRequest() {
        // Given
        CreateUserRequest request = new CreateUserRequest("test@example.com", "password123");
        User expectedUser = new User(1L, "test@example.com");
        when(userRepository.save(any())).thenReturn(expectedUser);

        // When
        User result = userService.createUser(request);

        // Then
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        verify(userRepository, times(1)).save(any());
    }
}
```

### 8.2 Naming Convention

- **Test classes**: `{ClassName}Test`
- **Test methods**: `should{ExpectedBehavior}When{Condition}` or `{methodName}_{condition}_{expectedResult}`
- **Examples**: `shouldThrowExceptionWhenEmailInvalid`, `createUser_existingEmail_throwsException`

### 8.3 Coverage Requirements

- **Unit tests**: 80%+ for service layer
- **Integration tests**: Critical user flows (login, checkout, etc.)
- **Controller tests**: Use `@WebMvcTest` for isolated endpoint testing

---

## 9. LOGGING

### 9.1 Logger Declaration

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class UserService {
    public User createUser(CreateUserRequest request) {
        log.info("Creating user with email: {}", request.getEmail());
        // ...
        log.debug("User created successfully with ID: {}", user.getId());
    }
}
```

### 9.2 Log Levels

- **ERROR**: Unrecoverable failures
- **WARN**: Recoverable issues (e.g., retry attempts)
- **INFO**: Key business events (user login, order placed)
- **DEBUG**: Detailed flow for troubleshooting
- **TRACE**: Very detailed (use sparingly)

### 9.3 Best Practices

- Use placeholders `{}` instead of string concatenation
- Never log sensitive data (passwords, tokens, credit cards)
- Include context (user ID, request ID) in logs

---

## 10. SECURITY (RECOMMENDED)

### 10.1 Authentication & Authorization

- Use Spring Security with JWT or OAuth 2.0
- Implement Role-Based Access Control (RBAC)

```java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    // Only admins can access
}
```

### 10.2 Input Sanitization

- Validate all user inputs with Bean Validation
- Use parameterized queries to prevent SQL injection (JPA does this automatically)
- Escape outputs in templates (Thymeleaf auto-escapes)

### 10.3 Secrets Management

- Store secrets in environment variables or secret managers (AWS Secrets Manager, HashiCorp Vault)
- Never commit `.env` files or `application-local.properties` to Git

---

## 11. PERFORMANCE OPTIMIZATION (RECOMMENDED)

### 11.1 Caching

```java
@Cacheable(value = "users", key = "#id")
public User getUserById(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new EntityNotFoundException("User not found"));
}

@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```

### 11.2 Async Processing

```java
@Async
public CompletableFuture<Void> sendEmailAsync(String email, String subject, String body) {
    emailService.send(email, subject, body);
    return CompletableFuture.completedFuture(null);
}
```

### 11.3 Pagination

```java
@GetMapping("/users")
public Page<UserResponse> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id,desc") String sort
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sort.split(",")));
    return userService.getUsers(pageable);
}
```

---

## 12. DOCUMENTATION (RECOMMENDED)

### 12.1 Javadoc

- Required for public APIs (methods, classes)
- Focus on "why" and "what", not "how"

```java
/**
 * Creates a new user account and sends a verification email.
 *
 * @param request the user registration details
 * @return the created user entity
 * @throws DuplicateEmailException if email already exists
 */
public User createUser(CreateUserRequest request) {
    // ...
}
```

### 12.2 API Documentation

- Use Springdoc OpenAPI (Swagger) for automatic API docs

```java
@Operation(summary = "Create a new user", description = "Registers a user and returns the created entity")
@ApiResponses({
    @ApiResponse(responseCode = "201", description = "User created successfully"),
    @ApiResponse(responseCode = "400", description = "Invalid request")
})
@PostMapping("/users")
public ResponseEntity<UserResponse> createUser(@RequestBody CreateUserRequest request) {
    // ...
}
```

---

## PROHIBITED PRACTICES

1. ❌ **No magic numbers/strings**: Use constants or enums
2. ❌ **No `System.out.println()`**: Use logger
3. ❌ **No empty catch blocks**: At minimum, log the exception
4. ❌ **No raw types**: Use generics (`List<String>` not `List`)
5. ❌ **No mutable static fields**: Causes concurrency issues
6. ❌ **No `@Autowired` on fields**: Use constructor injection
7. ❌ **No exposing entities in REST APIs**: Use DTOs
8. ❌ **No business logic in controllers**: Move to service layer

---

## ENFORCEMENT CHECKLIST

Before submitting code, verify:

- [ ] All classes/methods follow naming conventions (Section 1)
- [ ] Package structure matches layered architecture (Section 2.1)
- [ ] Constructor injection used for dependencies (Section 3.1)
- [ ] DTOs used for API requests/responses (Section 4.1)
- [ ] Bean Validation applied to request objects (Section 7.1)
- [ ] Unit tests cover service layer with 80%+ coverage (Section 8.3)
- [ ] Logging uses SLF4J with placeholders (Section 9.1)
- [ ] No prohibited practices violated (Section above)

---

**Version**: 1.0.0  
**Last Updated**: 2025-12-09  
**License**: MIT (derived from qoder-rules by lvzhaobo)