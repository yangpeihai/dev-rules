# Java 后端项目结构 (标准 Maven 5 层模块结构)

## 1. 命名约定

**【强制】** 包名根目录 `com.sf.aiops` 为固定值，各部分含义：
- `sf`：公司缩写（深信服）
- `aiops`：业务线缩写（智能运维）

完整包名格式：`com.sf.aiops.{系统}`，其中 `{系统}` 需替换为具体业务系统缩写（如 `sds`、`rms`、`cds`）。

模块命名格式：`sf-aiops-{系统}-{层级}`，如 `sf-aiops-sds-web`。

---

## 2. 标准 Maven 多模块结构

**【强制】** 采用标准 Maven 5 层模块工程结构：

```
project-root/                        # 项目根目录
├── pom.xml                          # 父 POM，统一管理依赖版本
├── README.md                        # 项目说明文档
│
├── sf-aiops-sds-web/                # Controller 层，提供 REST API
│   ├── pom.xml
│   └── src/main/java/com/sf/aiops/sds/
│       ├── controller/              # REST 接口
│       ├── third/                   # 第三方调用接口（供外部系统调用）
│       ├── job/                     # 定时任务（xxl-job）
│       ├── config/                  # 配置类
│       ├── interceptor/             # 拦截器
│       ├── filter/                  # 过滤器
│       └── handler/                 # 全局异常处理器
│
├── sf-aiops-sds-service/            # Service 层，业务逻辑实现
│   ├── pom.xml
│   └── src/main/java/com/sf/aiops/sds/
│       ├── service/                 # 业务接口
│       ├── service/impl/            # 业务实现
│       ├── domain/                  # 领域模型（BO，Service 层内部使用）
│       ├── proxy/                   # 内部服务调用代理（Feign）
│       │   └── third/               # 第三方系统调用代理（HttpUtil）
│       ├── converter/               # 对象转换器（PO↔VO↔DTO）
│       └── event/                   # 领域事件（可选）
│
├── sf-aiops-sds-mapper/             # Mapper 层，数据库操作
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/sf/aiops/sds/
│       │   └── mapper/              # MyBatis Mapper 接口
│       └── resources/
│           └── mapper/              # Mapper XML 文件
│
├── sf-aiops-sds-po/                 # 实体类（PO，Persistent Object）
│   ├── pom.xml
│   └── src/main/java/com/sf/aiops/sds/
│       └── po/                      # 数据库实体（PO）
│
└── sf-aiops-sds-vo/                 # 视图对象（VO，View Object）
    ├── pom.xml
    └── src/main/java/com/sf/aiops/sds/
        ├── vo/                      # 视图对象
        ├── dto/                     # 数据传输对象
        ├── query/                   # 查询对象
        ├── enums/                   # 枚举定义
        └── constant/                # 常量定义
```

> **说明**：以上目录结构以服务器交付系统 `sds` 为例。实际项目中，将 `sds` 替换为具体业务系统名称。

---

## 3. 模块职责定义

| 层级 | 模块 | 职责 | 禁止行为 |
|------|------|------|----------|
| Controller | web | 接收请求、参数校验、调用 Service、返回响应 | 禁止编写业务逻辑；禁止直接依赖 Mapper；禁止返回 PO 类型 |
| Service | service | 业务逻辑编排、事务管理、对象转换（PO↔VO↔DTO） | 禁止直接操作 SQL；禁止返回 PO 类型；禁止包含 HTTP 处理；禁止将 BO 暴露给外部 |
| Mapper | mapper | 数据访问、SQL 执行 | 禁止包含业务逻辑；禁止直接调用 Service |
| PO | po | 数据库实体定义 | 禁止包含业务方法；禁止包含 Swagger/API 注解 |
| VO | vo | 视图对象、DTO、查询对象定义 | 禁止包含业务逻辑；禁止与数据库表结构耦合 |

---

## 4. 模块依赖关系

**【强制】** 模块依赖必须遵循：

```
web (Controller)
  ├── 依赖 service
  │     ├── 依赖 mapper
  │     │     └── 依赖 po
  │     ├── 依赖 vo
  │     └── 依赖 po
  └── 依赖 vo
```

**pom.xml 依赖配置示例**：

```xml
<!-- sf-aiops-sds-web/pom.xml -->
<dependencies>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-service</artifactId>
    </dependency>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-vo</artifactId>
    </dependency>
</dependencies>

<!-- sf-aiops-sds-service/pom.xml -->
<dependencies>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-mapper</artifactId>
    </dependency>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-vo</artifactId>
    </dependency>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-po</artifactId>
    </dependency>
</dependencies>

<!-- sf-aiops-sds-mapper/pom.xml -->
<dependencies>
    <dependency>
        <groupId>com.sf.aiops</groupId>
        <artifactId>sf-aiops-sds-po</artifactId>
    </dependency>
</dependencies>
```

---

## 5. 跨层调用禁止

**【强制】** 调用必须遵循 `Controller → Service → Mapper` 的层级关系，禁止跨层调用。

**正确示例**：

```java
// Controller（web 模块）- 只依赖 Service
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;

    @GetMapping("/orders/{id}")
    public Result<OrderVo> getOrder(@PathVariable Long id) {
        return Result.success(orderService.getOrder(id));
    }
}

// Service（service 模块）- 依赖 Mapper，返回 VO
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired
    private OrderMapper orderMapper;

    @Override
    public OrderVo getOrder(Long id) {
        OrderPO order = orderMapper.selectById(id);
        return OrderConverter.INSTANCE.toVo(order);
    }
}
```

**错误示例**：

```java
// 错误：Controller 直接调用 Mapper（跨层调用）
@RestController
public class OrderController {
    @Autowired
    private OrderMapper orderMapper;  // 禁止！
}

// 错误：Service 直接返回 PO 给 Controller
@Service
public class OrderService {
    public OrderPO getOrder(Long id) {  // 应返回 VO
        return orderMapper.selectById(id);
    }
}
```

---

## 6. 领域模型对应关系

| 模型类型 | 全称 | 所属模块 | 使用场景 |
|---------|------|---------|---------|
| PO/DO | Persistent Object | po 模块 | 与数据库表结构一一对应，Mapper 层使用 |
| BO | Business Object | service 模块（domain 包） | 业务对象，Service 层内部使用，**不对外暴露** |
| DTO | Data Transfer Object | vo 模块 | 数据传输对象，Service 层向外部传输 |
| VO | View Object | vo 模块 | 视图对象，Controller 层返回给前端 |
| Query | 数据查询对象 | vo 模块 | 接收查询请求，超过 2 个参数禁止使用 Map |

**DTO 与 VO 的区分**：

| 对比项 | DTO | VO |
|-------|-----|-----|
| **主要用途** | 跨层/跨服务数据传输 | 前端视图展示 |
| **使用场景** | Service 接收请求参数、RPC 接口参数/返回值 | Controller 返回给前端 |
| **命名示例** | `OrderCreateDto`、`UserUpdateDto` | `OrderVo`、`UserVo` |

---

## 7. 对象转换器（Converter）

**Converter** 位于 service 模块的 `converter/` 包下，负责各层领域模型之间的转换。

**MapStruct Converter 示例**：

```java
@Mapper
public interface OrderConverter {
    OrderConverter INSTANCE = Mappers.getMapper(OrderConverter.class);

    OrderVo toVo(OrderPO po);
    OrderPO toPo(OrderVo vo);
    OrderDto toDto(OrderPO po);
}
```

**使用规范**：
- Converter 类以 `Converter` 结尾命名
- 使用 MapStruct 时，通过 `INSTANCE` 单例调用
- 禁止在 Converter 中包含业务逻辑

---

## 8. Proxy 包职责

**proxy 包** 位于 service 模块，承担 Manager 层的核心职责：

```
proxy/
├── XxxProxy.java          # 内部服务调用（Feign）
└── third/
    └── XxxProxy.java      # 第三方系统调用（Hutool HttpUtil）
```

**使用规范**：
- Proxy 类只负责远程调用封装，不包含业务逻辑
- 必须处理超时、异常、重试等边界情况
- 返回结果应转换为 DTO 或 VO

---

## 9. 缓存与多 DAO 组合处理

**缓存处理**：直接在 Service 实现类中使用 Spring Cache 注解。

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;

    @Cacheable(value = "user", key = "#id")
    @Override
    public UserVo getUser(Long id) {
        return UserConverter.INSTANCE.toVo(userMapper.selectById(id));
    }

    @CacheEvict(value = "user", key = "#id")
    @Override
    public void updateUser(Long id, UserVo userVo) {
        userMapper.updateById(UserConverter.INSTANCE.toPo(userVo));
    }
}
```

**多 DAO 组合处理**：

```java
@Service
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final OrderMapper orderMapper;
    private final OrderItemMapper orderItemMapper;

    // 读操作无需事务控制
    @Override
    public OrderDetailVo getOrderDetail(Long orderId) {
        OrderPO order = orderMapper.selectById(orderId);
        List<OrderItemPO> items = orderItemMapper.selectByOrderId(orderId);
        return OrderDetailConverter.INSTANCE.toVo(order, items);
    }

    // 写操作必须添加事务控制
    @Transactional(rollbackFor = Exception.class)
    @Override
    public Long createOrder(OrderDto orderDto) {
        OrderPO order = OrderConverter.INSTANCE.toPo(orderDto);
        orderMapper.insert(order);
        // ...
        return order.getId();
    }
}
```

---

## 10. 定时任务与第三方接口

**job 包** 位于 web 模块：

```java
@Component
@RequiredArgsConstructor
public class SyncDataJob {

    private final SyncService syncService;

    @XxlJob("syncDataJob")
    public void execute() {
        syncService.syncData();  // 业务逻辑由 Service 层实现
    }
}
```

**third 包** 位于 web 模块，供外部系统调用：

```java
@RestController
@RequestMapping("/third/order")
@RequiredArgsConstructor
public class OrderThirdController {

    private final OrderService orderService;

    @PostMapping("/create")
    public Result<OrderVo> createOrder(@RequestBody OrderDto dto) {
        return Result.success(orderService.createOrder(dto));
    }
}
```

---

## 11. 包命名规范速查

```
com.sf.aiops.sds.controller      # Controller 层（web 模块）
com.sf.aiops.sds.third           # 第三方调用接口（web 模块）
com.sf.aiops.sds.job             # 定时任务（web 模块）
com.sf.aiops.sds.config          # 配置类（web 模块）
com.sf.aiops.sds.service         # Service 接口（service 模块）
com.sf.aiops.sds.service.impl    # Service 实现（service 模块）
com.sf.aiops.sds.domain          # 领域模型 BO（service 模块）
com.sf.aiops.sds.proxy           # 内部服务调用代理（service 模块）
com.sf.aiops.sds.proxy.third     # 第三方系统调用代理（service 模块）
com.sf.aiops.sds.converter       # 对象转换器（service 模块）
com.sf.aiops.sds.mapper          # Mapper 接口（mapper 模块）
com.sf.aiops.sds.po              # 实体类（po 模块）
com.sf.aiops.sds.vo              # 视图对象（vo 模块）
com.sf.aiops.sds.dto             # 数据传输对象（vo 模块）
com.sf.aiops.sds.query           # 查询对象（vo 模块）
com.sf.aiops.sds.enums           # 枚举（vo 模块）
com.sf.aiops.sds.constant        # 常量（vo 模块）
```