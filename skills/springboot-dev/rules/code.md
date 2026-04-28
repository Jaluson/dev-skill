# Coding Standards

> Applicable mode: Development mode + Review mode
> Rule prefix: R-CODE

## R-CODE-01 Must use try-with-resources

All resources implementing `AutoCloseable` (such as Connection, InputStream, SqlSession) must use try-with-resources or equivalent safe close mechanism.

**Correct example**:
```java
try (InputStream is = Files.newInputStream(path)) {
    // use is
}
```

**Incorrect example**:
```java
InputStream is = Files.newInputStream(path);
// use is
is.close(); // may not close if exception occurs
```

## R-CODE-02 Magic values are prohibited; use enums for all

Unnamed constant values (magic values) are prohibited in code. All business constants must be encapsulated using enums.

**Incorrect example**:
```java
if ("ACTIVE".equals(status)) { ... }
if (code == 1) { ... }
```

**Correct example**:
```java
if (UserStatusEnum.ACTIVE.getCode().equals(status)) { ... }
```

Review checkpoint: Scan all string literals and numeric literals to determine if they are business constants.

## R-CODE-03 Input and output parameters must be encapsulated with classes

Interface input and output parameters are prohibited from using weak types such as `Map`, `JSONObject`, `Object`.

**Incorrect example**:
```java
public Map<String, Object> query(@RequestBody Map<String, Object> params)
```

**Correct example**:
```java
public UserPageResponse query(@RequestBody @Valid UserPageRequest request)
```

Data transfer between internal methods should also prefer concrete classes over Map.

## R-CODE-04 Status codes must use HTTP STATUS

Interface response status codes must use standard HTTP status codes:
- `200` Success
- `400` Bad request parameters
- `401` Unauthorized
- `403` Forbidden
- `404` Resource not found
- `500` Internal server error

Custom status codes (e.g., `code: 10001` to indicate success) are prohibited.

Business error information goes in the response body; status codes still use `200` or `400`.

## R-CODE-05 Minimum change principle

Implement features with the least amount of code and impact:
- Do not modify files unrelated to the requirement
- Do not refactor code not involved
- Do not add unrequested features

Review checkpoint (incremental review): Check if the diff scope includes unrelated files.

## R-CODE-06 Comments must follow Javadoc standard

All public classes, interfaces, and methods must have Javadoc comments.

Requirements:
- Concise and direct, do not describe the obvious
- Complex logic blocks must describe steps
- Key steps must explain design intent

**Correct example**:
```java
/**
 * Query order list by user ID.
 * Steps: 1. Validate user status 2. Query orders 3. Desensitize
 *
 * @param userId User ID, must not be null
 * @return Order list, sorted by create time descending
 */
public List<Order> queryByUserId(Long userId) { ... }
```

## R-CODE-07 Logging must use log4j2; sensitive data must be desensitized

Logging framework must uniformly use log4j2.

Requirements:
- Key steps (entry, exit, exception, status change) must print logs
- Sensitive information (phone, ID card, bank card, password) must be desensitized
- `System.out.println` is prohibited

**Desensitization example**:
```java
// Phone: 138****8888
// ID card: 110101********1234
```

## R-CODE-08 Use early return to avoid deep nesting

Reduce code nesting levels by returning early.

**Incorrect example**:
```java
if (user != null) {
    if (user.isActive()) {
        if (order != null) {
            // business logic
        }
    }
}
```

**Correct example**:
```java
if (user == null) {
    return;
}
if (!user.isActive()) {
    return;
}
if (order == null) {
    return;
}
// business logic
```

## R-CODE-09 Do not call beans at the same layer

Service layer must not inject and call other Services. Business orchestration should be done uniformly at the Controller layer.

**Incorrect example**:
```java
@Service
public class OrderService {
    @Autowired
    private UserService userService; // forbidden: same-layer call
}
```

**Correct example**:
```java
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;
    @Autowired
    private UserService userService;
    
    // orchestrate in Controller
}
```

Reusable logic should be extracted into composite Services (see R-CODE-10).

## R-CODE-10 Composite Services allow orchestration of multiple Services

For complex business logic spanning multiple Services, composite Services are allowed (e.g., `OrderFacadeService`, `OrderCompositeService`).

Requirements:
- Composite Services inject multiple base Services for orchestration
- Composite Services do not inherit MyBatisPlus Service interfaces (R-ARCH-05)
- Controllers can directly call composite Services

## R-CODE-11 Parameter validation must uniformly use @Valid / @Validated

Input validation must uniformly use Bean Validation:
- Single field validation: `@Valid`
- Group validation: `@Validated(Group.class)`

**Correct example**:
```java
public Response create(@RequestBody @Valid UserCreateRequest request) { ... }

public Response update(@RequestBody @Validated(UpdateGroup.class) UserUpdateRequest request) { ... }
```

## R-CODE-12 Use LocalDateTime instead of Date

All date-time types must use `java.time.LocalDateTime`; `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp` are prohibited (except for database mapping).

**Correct example**:
```java
private LocalDateTime createTime;
```

## R-CODE-13 Time format must be yyyy-MM-dd HH:mm:ss

Date-time formatting must uniformly use `yyyy-MM-dd HH:mm:ss`.

**Correct example**:
```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime createTime;
```

## R-CODE-14 Class conversion must use rich domain model

Object conversion (DTO ↔ Entity ↔ VO) should preferably be encapsulated as methods inside the class itself, avoiding independent Converter/Transformer class bloat.

**Correct example**:
```java
public class User {
    public static User from(UserCreateRequest request) { ... }
    public UserVo toVo() { ... }
}
```

**Avoid**:
```java
public class UserConverter {
    public static User convert(UserCreateRequest request) { ... }
}
```
