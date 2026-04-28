# ORM and Pagination Standards

> Applicable mode: Development mode + Review mode
> Rule prefix: R-ORM

## R-ORM-01 Code generation order

Code files must be generated in the following order:

1. **Entity** — defines data model
2. **Mapper** — defines data access interface
3. **Controller** — defines external API
4. **Service / ServiceImpl** — defines business logic

**Reverse order generation is prohibited**: Do not generate Service before the entity class is determined.

## R-ORM-02 Pagination parameters passed via header

Pagination parameters are passed via HTTP Header, not Query Parameter or Request Body.

Header names:
| Header | Description | Default |
|--------|-------------|---------|
| `X-Page-Num` | Current page number | 1 |
| `X-Page-Size` | Items per page | 10 |

**Controller example**:
```java
@GetMapping("/users")
public Page<UserVo> list(
    @RequestHeader(value = "X-Page-Num", defaultValue = "1") Integer pageNum,
    @RequestHeader(value = "X-Page-Size", defaultValue = "10") Integer pageSize
) {
    return userService.page(new Page<>(pageNum, pageSize));
}
```

## R-ORM-03 Deep pagination optimization must be considered

Deep pagination (flipping to later pages with large data volumes) has performance issues; optimization solutions must be considered.

**Optimization methods**:

1. **Cursor pagination (recommended)**
   - Use the sorting field value of the last item on the previous page as the starting point for the next page
   - Avoid excessively large `OFFSET`

2. **Limit maximum page number**
   ```java
   private static final int MAX_PAGE_NUM = 1000;
   ```

3. **Provide export alternative**
   - For large data volume scenarios, provide async export instead of pagination query

## R-ORM-04 Pagination parameters obtained via injection

Pagination parameters should be obtained through Spring's injection mechanism (`@RequestHeader`), prohibited from hardcoding inside methods or reading from other non-standard locations.

**Correct example**:
```java
public Page<User> queryUserPage(@RequestHeader("X-Page-Num") Integer pageNum,
                                 @RequestHeader("X-Page-Size") Integer pageSize) {
    return page(new Page<>(pageNum, pageSize));
}
```

**Incorrect example**:
```java
public Page<User> queryUserPage() {
    int pageNum = 1; // hardcoded
    int pageSize = 10; // hardcoded
    return page(new Page<>(pageNum, pageSize));
}
```

## Appendix: MyBatisPlus Common Standards

### Entity Annotations
```java
@Data
@TableName("user")
public class User {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;
    
    @TableField("user_name")
    private String userName;
    
    @TableLogic
    @TableField("deleted")
    private Integer deleted;
}
```

### Mapper XML Location
```yaml
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
```
