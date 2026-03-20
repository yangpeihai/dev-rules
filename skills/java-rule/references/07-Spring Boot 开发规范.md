# Spring Boot 开发规范

## (一) 项目配置

1. **【强制】** 使用多环境配置文件管理，主配置仅包含公共配置。
   - 目录结构：
     ```
     src/main/resources/
     ├── application.yml           # 主配置，激活对应环境
     ├── application-dev.yml       # 开发环境
     ├── application-test.yml      # 测试环境
     ├── application-staging.yml   # 预发环境
     └── application-prod.yml      # 生产环境
     ```
   - application.yml 示例：
     ```yaml
     spring:
       profiles:
         active: ${SPRING_PROFILES_ACTIVE:dev}
       application:
         name: order-service
     ```

2. **【强制】** 禁止在配置文件中硬编码敏感信息（密码、密钥等），必须通过环境变量或配置中心注入。
   - 正例：`password: ${DB_PASSWORD}`
   - 反例：`password: root123`

3. **【推荐】** 使用 `@ConfigurationProperties` 进行类型安全的配置绑定。
   - 正例：
     ```java
     @Data
     @Configuration
     @ConfigurationProperties(prefix = "order")
     public class OrderProperties {
         private int maxRetryTimes = 3;
         private Duration timeout = Duration.ofSeconds(30);
     }
     ```

4. **【推荐】** 配置类命名以 Properties 或 Config 结尾，语义清晰。
   - 正例：`OrderProperties` / `RedisConfig`

---

## (二) 依赖管理

1. **【强制】** 优先使用官方 Starter，自定义 Starter 遵循命名规范。
   - 正例：`spring-boot-starter-web` / `sf-spring-boot-starter-trace`

2. **【强制】** 在父 POM 中使用 `dependencyManagement` 统一管理依赖版本。
   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-dependencies</artifactId>
               <version>3.2.0</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

3. **【推荐】** 避免依赖冲突，定期执行 `mvn dependency:tree` 检查。

---

## (三) Controller 规范

1. **【强制】** 使用 `@RestController` 替代`@Controller + @ResponseBody`。

2. **【强制】** 请求路径统一前缀，包含版本号。
   - 正例：`@RequestMapping("/api/v1/orders")`

3. **【强制】** 所有接口返回统一 Result 对象，包含 code、message、data。
   ```java
   @Data
   public class Result<T> {
       private String code;
       private String message;
       private T data;
       private Long timestamp;

       public static <T> Result<T> success(T data) { ... }
       public static <T> Result<T> fail(String code, String message) { ... }
   }
   ```

4. **【推荐】** 使用 `@Validated` 或 `@Valid` 进行参数校验。

5. **【推荐】** 使用 Swagger/OpenAPI 注解描述接口。
   - 正例：
     ```java
     @Tag(name = "订单管理", description = "订单相关接口")
     @Operation(summary = "查询订单详情")
     ```

---

## (四) Service 层规范

1. **【强制】** Service 必须定义接口，实现类以 Impl 结尾。
   - 正例：`OrderService` 接口 + `OrderServiceImpl` 实现类

2. **【强制】** 合理使用 `@Transactional`，注意传播行为和回滚策略。
   - 查询方法设置 `readOnly = true`
   - 必须指定 `rollbackFor = Exception.class`
   - 特殊场景使用 `REQUIRES_NEW` 传播行为

3. **【推荐】** Service 类命名以 Service 结尾，避免 Manager 等模糊命名。

4. **【推荐】** 事务方法内避免远程调用，确需调用时评估超时时间。

---

## (五) 异常处理

1. **【强制】** 使用 `@RestControllerAdvice` + `@ExceptionHandler` 统一处理异常。

2. **【强制】** 区分业务异常和系统异常。
   - `BusinessException`：用户输入错误、业务规则冲突，返回 400
   - `SystemException`：系统内部错误，返回 500

3. **【强制】** 禁止吞异常，捕获后必须记录日志并抛出或处理。
   - 正例：
     ```java
     try {
         // 业务逻辑
     } catch (Exception e) {
         log.error("处理失败，id={}", id, e);
         throw new SystemException("处理失败", e);
     }
     ```

4. **【推荐】** 自定义异常体系继承 BaseException。
   ```java
   public abstract class BaseException extends RuntimeException {
       private final String errorCode;
       private final String errorMessage;
       // ...
   }
   ```

---

## (六) 配置类规范

1. **【强制】** 使用 `@Configuration` 定义配置类，`@Bean` 定义 Bean。

2. **【推荐】** 使用条件注解控制配置加载。
   - `@ConditionalOnProperty`：属性存在时加载
   - `@ConditionalOnMissingBean`：容器中无此 Bean 时创建
   - `@ConditionalOnClass`：类路径存在某类时加载

3. **【推荐】** Bean 命名使用驼峰，语义清晰。
   - 正例：`@Bean("orderExecutor")`

---

## (七) 启动类规范

1. **【强制】** 明确指定扫描路径。
   - 正例：`@SpringBootApplication(scanBasePackages = "com.sf.order")`

2. **【推荐】** 使用 `@MapperScan` 指定 Mapper 扫描路径。

3. **【推荐】** 排除不必要的自动配置。
   - 正例：
     ```java
     @SpringBootApplication(
         exclude = {
             DataSourceAutoConfiguration.class,
             HibernateJpaAutoConfiguration.class
         }
     )
     ```

---

## (八) 日志规范

1. **【强制】** 统一使用 SLF4J API，配合 Logback 或 Log4j2 实现。
   - 正例：
     ```java
     @Slf4j
     @Service
     public class OrderService {
         public void process() {
             log.info("处理开始，id={}", id);
         }
     }
     ```

2. **【强制】** 使用 `{}` 占位符，避免字符串拼接。
   - 正例：`log.debug("id={}, name={}", id, name)`
   - 反例：`log.debug("id=" + id + ", name=" + name)`

3. **【强制】** 日志级别使用规范：
   | 级别 | 使用场景 |
   |------|----------|
   | ERROR | 系统错误、业务异常、需要立即处理的问题 |
   | WARN | 潜在问题、非预期情况但可继续执行 |
   | INFO | 业务关键流程记录、状态变更 |
   | DEBUG | 详细调试信息、入参出参 |
   | TRACE | 最详细的跟踪信息 |

4. **【推荐】** 生产环境默认 INFO 级别，排查问题时动态调整。

---

## (九) 自动装配

1. **【推荐】** 自定义 Starter 时提供自动配置类。
   ```java
   @Configuration
   @ConditionalOnClass(MyService.class)
   @EnableConfigurationProperties(MyProperties.class)
   public class MyAutoConfiguration {
       @Bean
       @ConditionalOnMissingBean
       public MyService myService(MyProperties properties) {
           return new MyService(properties);
       }
   }
   ```

2. **【强制】** 在 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`中注册自动配置类（Spring Boot 2.7+/3.x）。

---

## (十) Actuator 监控

1. **【强制】** 生产环境必须关闭敏感端点或增加安全认证。
   ```yaml
   management:
     endpoints:
       web:
         exposure:
           include: health,info,metrics
         endpoint:
           shutdown:
             access: none
     security:
       enabled: true
   ```

2. **【推荐】** 配置健康检查探针用于 K8s 环境。
   ```yaml
   management:
     health:
       livenessState:
         enabled: true
       readinessState:
         enabled: true
   ```

3. **【推荐】** 自定义健康指标继承 `AbstractHealthIndicator`。
