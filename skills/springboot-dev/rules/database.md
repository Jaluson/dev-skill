# Database Standards

> Applicable mode: Development mode + Review mode
> Rule prefix: R-DB

## R-DB-01 Table primary key must use Snowflake algorithm

Primary key type must use `BIGINT`, generated via Snowflake algorithm.

MyBatisPlus configuration:
```java
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

Entity primary key annotation:
```java
@TableId(type = IdType.ASSIGN_ID)
private Long id;
```

## R-DB-02 Must have audit fields

Every table must contain the following audit fields:

| Field Name | Type | Description |
|------------|------|-------------|
| `id` | BIGINT | Primary key, Snowflake algorithm |
| `create_by` | VARCHAR(64) | Creator |
| `create_time` | DATETIME | Create time |
| `update_by` | VARCHAR(64) | Updater |
| `update_time` | DATETIME | Update time |
| `deleted` | TINYINT(1) | Logical deletion flag, 0-not deleted 1-deleted |

MyBatisPlus auto-fill configuration:
```java
@Component
public class MetaObjectHandler implements com.baomidou.mybatisplus.core.handlers.MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "deleted", Integer.class, 0);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

## R-DB-03 Fields should preferably use NOT NULL DEFAULT

Table field design principles:
- Prefer `NOT NULL`
- Provide reasonable `DEFAULT` values for `NOT NULL` fields
- Avoid large numbers of nullable fields that degrade data quality

**Correct example**:
```sql
`status` TINYINT NOT NULL DEFAULT 0 COMMENT 'Status: 0-disabled 1-enabled',
`remark` VARCHAR(500) NOT NULL DEFAULT '' COMMENT 'Remark',
```

## R-DB-04 Index design must consider business needs

Index design principles:
- Primary key automatically creates clustered index
- Foreign key association fields create indexes
- High-frequency query condition fields create indexes
- Sort fields consider indexes
- Avoid excessive indexes (affects write performance)
- Composite indexes follow leftmost prefix principle

## R-DB-05 Use EXISTS to check data existence

When determining whether data exists, use `EXISTS` instead of `COUNT(*)`.

**Incorrect example**:
```java
// Mapper
@Select("SELECT COUNT(*) FROM user WHERE status = 1")
int countActiveUsers();

// Service
if (userMapper.countActiveUsers() > 0) { ... }
```

**Correct example**:
```java
// Mapper
@Select("SELECT 1 FROM user WHERE status = 1 LIMIT 1")
Integer existsActiveUser();

// Service
if (userMapper.existsActiveUser() != null) { ... }
```

Or use MyBatisPlus's `exists` method.

## R-DB-06 Queries must only fetch necessary fields

`SELECT *` is prohibited in business queries. Required fields must be explicitly specified.

**Incorrect example**:
```xml
<select id="selectById" resultType="User">
    SELECT * FROM user WHERE id = #{id}
</select>
```

**Correct example**:
```xml
<select id="selectById" resultType="User">
    SELECT id, username, email, status, create_time
    FROM user WHERE id = #{id}
</select>
```

MyBatisPlus's `select` method queries all fields by default; to limit, use `select(String... columns)`.

## R-DB-07 Subqueries are prohibited; consider JOIN optimization

Complex queries should prefer JOIN over subqueries.

**Incorrect example**:
```sql
SELECT * FROM order
WHERE user_id IN (SELECT id FROM user WHERE status = 1)
```

**Correct example**:
```sql
SELECT o.* FROM order o
INNER JOIN user u ON o.user_id = u.id
WHERE u.status = 1
```

For subqueries that must be used, add a comment explaining the reason.

## R-DB-08 Data modification logic must be wrapped in transactions

Operations involving data modification (INSERT / UPDATE / DELETE) must use transaction control; transaction scope should be as small as possible.

**Correct example**:
```java
@Transactional(rollbackFor = Exception.class)
public void createOrder(OrderRequest request) {
    // 1. Save order
    // 2. Deduct inventory
    // 3. Record log (if non-core operation, consider async)
}
```

**Notes**:
- Do not add `@Transactional` to query operations
- Transactional methods should be `public`
- Calling transactional methods within the same class does not take effect (must go through proxy)
- Avoid remote calls and complex computations within transaction scope
