# Architecture Standards

> Applicable mode: Development mode + Review mode
> Rule prefix: R-ARCH

## R-ARCH-01 ORM must use MyBatis + MyBatisPlus

Persistence layer framework must uniformly use MyBatis with MyBatisPlus.

Project dependency requirement:
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>
```

## R-ARCH-02 Service must inherit IService<T>

Business Service interfaces must inherit `com.baomidou.mybatisplus.extension.service.IService<T>`.

**Correct example**:
```java
public interface UserService extends IService<User> {
    // custom business methods
}
```

## R-ARCH-03 Impl must inherit ServiceImpl<Mapper, T>

Business Service implementation classes must inherit `com.baomidou.mybatisplus.extension.service.impl.ServiceImpl<M, T>`.

**Correct example**:
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // implement custom methods
}
```

## R-ARCH-04 Mapper must inherit BaseMapper<T>

Data access layer Mapper must inherit `com.baomidou.mybatisplus.core.mapper.BaseMapper<T>`.

**Correct example**:
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // custom SQL methods
}
```

## R-ARCH-05 Composite Services do not need to inherit MP

Composite Services (for orchestrating multiple base Services in complex business) are high-custom logic and do not need to inherit the MyBatisPlus Service system.

**Correct example**:
```java
@Service
public class OrderCompositeService {
    @Autowired
    private OrderService orderService;
    @Autowired
    private InventoryService inventoryService;
    
    // orchestrate business logic, do not inherit ServiceImpl
}
```

## R-ARCH-06 Circular dependencies must be decoupled

Circular dependencies between Services and between Controllers are prohibited.

**Detection methods**:
- Development mode: Check injection relationships when generating code
- Review mode: Scan the class dependency graph of `@Autowired` / `@Resource` fields

**Common decoupling methods**:
- Sink common logic down to Mapper or utility classes
- Use event mechanism (ApplicationEvent)
- Use Controller layer orchestration instead of Service layer calls
