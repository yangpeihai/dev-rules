# Spring Cloud 微服务规范

## (一) 服务注册与发现

1. **【强制】** 使用 Nacos 作为服务注册中心，配置服务名称和命名空间。
   ```yaml
   spring:
     application:
       name: order-service
     cloud:
       nacos:
         discovery:
           server-addr: ${NACOS_SERVER:localhost:8848}
           namespace: ${NACOS_NAMESPACE:prod}
           group: DEFAULT_GROUP
   ```

2. **【强制】** 服务名称统一使用小写字母 + 中划线格式。
   - 正例：`order-service` / `user-service`
   - 反例：`OrderService` / `order_service`

3. **【推荐】** 配置服务元数据（版本、区域等）。
   ```yaml
   metadata:
     version: 1.0.0
     region: cn-shenzhen
   ```

4. **【推荐】** 配置健康检查，确保注册的服务可用。

---

## (二) 服务间通信

1. **【强制】** 使用 OpenFeign 进行服务间同步调用。
   - 正例：
     ```java
     @FeignClient(
         name = "inventory-service",
         fallbackFactory = InventoryServiceFallbackFactory.class
     )
     public interface InventoryServiceClient {
         @GetMapping("/api/v1/inventory/{skuId}")
         Result<InventoryVo> getInventory(@PathVariable("skuId") Long skuId);
     }
     ```

2. **【强制】** Feign 客户端必须配置降级工厂（FallbackFactory）。
   - 说明：实现服务降级，避免级联故障

3. **【强制】** 配置 Feign 超时时间。
   ```java
   @Bean
   public Request.Options feignOptions() {
       return new Request.Options(5, TimeUnit.SECONDS, 10, TimeUnit.SECONDS, true);
   }
   ```

4. **【推荐】** 配置 Feign 重试机制。
   ```java
   @Bean
   public Retryer feignRetryer() {
       return new Retryer.Default(100, TimeUnit.SECONDS.toMillis(1), 3);
   }
   ```

5. **【推荐】** Feign 调用路径包含版本号。
   - 正例：`/api/v1/inventory/{skuId}`

---

## (三) 配置中心

1. **【强制】** 使用 Nacos 作为配置中心，支持配置热更新。
   ```yaml
   spring:
     cloud:
       nacos:
         config:
           server-addr: ${NACOS_SERVER:localhost:8848}
           namespace: ${NACOS_NAMESPACE:prod}
           group: DEFAULT_GROUP
           file-extension: yaml
           refresh-enabled: true
   ```

2. **【强制】** 敏感配置必须加密存储。
   - 使用 Nacos 加密配置或 Jasypt
   - 正例：
     ```yaml
     spring:
       datasource:
         password: ENC(加密后的密文)
     jasypt:
       encryptor:
         password: ${JASYPT_PASSWORD}
     ```

3. **【推荐】** 使用 `@RefreshScope` 注解支持配置热更新。

4. **【推荐】** 配置变更必须记录审计日志。

---

## (四) 网关规范

1. **【强制】** 使用 Spring Cloud Gateway 作为统一入口。

2. **【强制】** 配置路由规则和重试机制。
   ```yaml
   spring:
     cloud:
       gateway:
         routes:
           - id: order-service
             uri: lb://order-service
             predicates:
               - Path=/api/v1/orders/**
             filters:
               - name: Retry
                 args:
                   retries: 3
                   statuses: BAD_GATEWAY,SERVICE_UNAVAILABLE
   ```

3. **【强制】** 配置全局限流过滤器。
   ```yaml
   default-filters:
     - name: RequestRateLimiter
       args:
         redis-rate-limiter.replenishRate: 100
         redis-rate-limiter.burstCapacity: 200
   ```

4. **【推荐】** 配置熔断器（CircuitBreaker）。

5. **【推荐】** 配置跨域（CORS）策略。

---

## (五) 熔断与降级

1. **【强制】** 使用 Sentinel 配置限流和降级规则。
   - 限流规则：
     ```java
     FlowRule rule = new FlowRule();
     rule.setResource("createOrder");
     rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
     rule.setCount(100);  // QPS 限制
     ```
   - 降级规则：
     ```java
     DegradeRule rule = new DegradeRule();
     rule.setResource("createOrder");
     rule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO);
     rule.setCount(0.5);  // 异常比例超过 50%
     rule.setTimeWindow(30);  // 熔断 30 秒
     ```

2. **【推荐】** 使用 `@SentinelResource` 注解定义资源。
   ```java
   @SentinelResource(
       value = "createOrder",
       blockHandler = "createOrderBlockHandler",
       fallback = "createOrderFallback"
   )
   public String createOrder(OrderCreateDto dto) {
       // 业务逻辑
   }
   ```

3. **【推荐】** 降级策略：
   - 返回默认值
   - 返回缓存数据
   - 抛出友好的业务异常

---

## (六) 链路追踪

1. **【推荐】** 使用 Sleuth + Zipkin 进行链路追踪。
   ```yaml
   spring:
     sleuth:
       enabled: true
       sampler:
         probability: 0.1  # 生产环境采样率
       propagation:
         type: W3C,B3
     zipkin:
       base-url: http://zipkin-server:9411
   ```

2. **【推荐】** 日志输出 TraceId 和 SpanId。
   ```xml
   <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId:-},%X{spanId:-}] %-5level %logger{36} - %msg%n</pattern>
   ```

3. **【推荐】** 关键业务方法手动打点。
   ```java
   @Span(value = "processOrder")
   public void processOrder(Order order) {
       // 业务逻辑
   }
   ```

---

## (七) 微服务拆分原则

1. **【强制】** 遵循单一职责原则，每个服务只负责一个明确的业务能力。

2. **【强制】** 服务间数据独立，禁止跨库查询。
   - 说明：通过服务间调用或事件驱动获取数据

3. **【推荐】** 服务拆分维度：
   - 按业务领域（订单、用户、支付）
   - 按功能职责（计算、存储、通知）

4. **【推荐】** 通信方式选择：
   | 场景 | 通信方式 | 说明 |
   |------|----------|------|
   | 同步调用 | OpenFeign + HTTP | 实时性要求高 |
   | 异步消息 | Spring Cloud Stream + MQ | 削峰填谷、解耦 |
   | 事件驱动 | 领域事件 + 消息队列 | 最终一致性 |

---

## (八) 负载均衡

1. **【推荐】** 使用 Spring Cloud LoadBalancer 进行客户端负载均衡。

2. **【推荐】** 配置负载均衡策略。
   ```yaml
   order-service:
     ribbon:
       NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
   ```

3. **【推荐】** 配置服务实例健康检查。

---

## (九) 分布式事务

1. **【推荐】** 优先使用最终一致性方案（消息队列 + 本地事务表）。

2. **【推荐】** 使用 Seata 处理强一致性场景。
   ```java
   @GlobalTransactional
   public void createOrder(OrderCreateDto dto) {
       // 扣减库存
       inventoryService.deduct(dto.getSkuId(), dto.getQuantity());
       // 创建订单
       orderService.create(dto);
   }
   ```

3. **【强制】** 禁止在微服务间使用 @Transactional 传播事务。
