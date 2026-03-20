# Spring MVC 规范

## (一) 请求参数绑定

1. **【强制】** 正确使用参数绑定注解：
   | 注解 | 使用场景 | 示例 |
   |------|----------|------|
   | `@RequestParam` | URL 参数、表单参数 | `?page=1&size=10` |
   | `@PathVariable` | RESTful 路径参数 | `/orders/{id}` |
   | `@RequestBody` | JSON 请求体 | POST/PUT 请求 |
   | `@RequestHeader` | 请求头参数 | Token、版本号 |
   | `@ModelAttribute` | 表单对象绑定 | 复杂表单 |

2. **【强制】** 路径变量必须与 URL 中的占位符名称一致。
   - 正例：`@GetMapping("/orders/{orderId}")` + `public Result getOrder(@PathVariable Long orderId)`

3. **【推荐】** 请求参数设置默认值或标记必填。
   - 正例：`@RequestParam(defaultValue = "1") int page`
   - 正例：`@RequestParam(required = false) String status`

---

## (二) 数据校验

1. **【强制】** 使用 JSR 380 Bean Validation 进行参数校验。
   - 常用校验注解：
     | 注解 | 说明 |
     |------|------|
     | `@NotNull` | 值不能为 null |
     | `@NotEmpty` | 字符串/集合不能为空 |
     | `@NotBlank` | 字符串不能为空白 |
     | `@Size` | 字符串长度或集合大小范围 |
     | `@Min`/`@Max` | 数值最小/最大值 |
     | `@Pattern` | 正则表达式匹配 |
     | `@Email` | 邮箱格式 |
     | `@Positive` | 正数 |

2. **【强制】** DTO 中校验注解必须带 message 属性，说明校验失败原因。
   - 正例：
     ```java
     @NotNull(message = "用户 ID 不能为空")
     @Positive(message = "用户 ID 必须为正数")
     private Long userId;
     ```

3. **【推荐】** 使用分组实现差异化校验。
   - 正例：
     ```java
     public interface ValidationGroups {
         interface Create {}
         interface Update {}
     }

     @Null(groups = ValidationGroups.Create.class)
     @NotNull(groups = ValidationGroups.Update.class)
     private Long id;
     ```

4. **【推荐】** Controller 中使用 `@Validated` 或 `@Valid` 触发校验。

---

## (三) 拦截器规范

1. **【强制】** 拦截器实现 `HandlerInterceptor` 接口。

2. **【强制】** 在 `afterCompletion` 中清理 ThreadLocal，防止内存泄漏。
   - 正例：
     ```java
     @Override
     public void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler, Exception ex) {
         UserContext.clear();
     }
     ```

3. **【推荐】** 拦截器注册时明确指定路径和排除路径。
   - 正例：
     ```java
     registry.addInterceptor(authInterceptor)
         .addPathPatterns("/api/**")
         .excludePathPatterns("/api/auth/**", "/api/public/**");
     ```

4. **【推荐】** 按职责分离拦截器：认证、限流、日志分别实现。

---

## (四) 统一响应格式

1. **【强制】** 所有接口返回统一 Result 对象。
   ```java
   @Data
   public class Result<T> {
       private String code;      // 状态码
       private String message;   // 描述信息
       private T data;           // 数据内容
       private Long timestamp;   // 时间戳
   }
   ```

2. **【强制】** 状态码规范：
   - `SUCCESS`：成功
   - `PARAM_ERROR`：参数错误
   - `BUSINESS_ERROR`：业务异常
   - `SYSTEM_ERROR`：系统错误

3. **【推荐】** 分页结果统一封装。
   ```java
   @Data
   public class PageResult<T> {
       private List<T> records;
       private long total;
       private long current;
       private long size;

       public static <T> PageResult<T> of(List<T> records, long total, long current, long size) {
           // ...
       }
   }
   ```

---

## (五) 全局异常处理

1. **【强制】** 使用 `@RestControllerAdvice` 定义全局异常处理器。

2. **【强制】** 按异常类型分别处理，返回统一格式。
   - 正例：
     ```java
     @RestControllerAdvice
     public class GlobalExceptionHandler {

         @ExceptionHandler(BusinessException.class)
         public Result<Void> handleBusinessException(BusinessException e) {
             log.warn("业务异常：code={}, message={}", e.getErrorCode(), e.getErrorMessage());
             return Result.fail(e.getErrorCode(), e.getErrorMessage());
         }

         @ExceptionHandler(SystemException.class)
         public Result<Void> handleSystemException(SystemException e) {
             log.error("系统异常：{}", e.getErrorMessage(), e);
             return Result.fail("SYSTEM_ERROR", "系统繁忙，请稍后重试");
         }

         @ExceptionHandler(MethodArgumentNotValidException.class)
         public Result<Void> handleValidationException(MethodArgumentNotValidException e) {
             String message = e.getBindingResult().getFieldErrors().stream()
                 .map(error -> error.getField() + ":" + error.getDefaultMessage())
                 .collect(Collectors.joining(", "));
             return Result.fail("PARAM_ERROR", message);
         }

         @ExceptionHandler(Exception.class)
         public Result<Void> handleException(Exception e) {
             log.error("未知异常", e);
             return Result.fail("SYSTEM_ERROR", "系统繁忙，请稍后重试");
         }
     }
     ```

3. **【推荐】** 业务异常携带错误码，便于前端展示。

4. **【推荐】** 未知异常记录完整堆栈，便于排查。

---

## (六) 文件上传下载

1. **【强制】** 文件上传必须校验：
   - 文件不能为空
   - 文件大小限制（如 10MB）
   - 文件类型校验（MIME 类型或扩展名）

2. **【强制】** 文件下载必须设置正确的 Content-Type 和 Content-Disposition。
   - 正例：
     ```java
     response.setContentType("application/pdf");
     response.setHeader(HttpHeaders.CONTENT_DISPOSITION,
         "attachment; filename=\"" + filename + "\"");
     ```

3. **【推荐】** 使用临时文件处理大文件上传。

4. **【推荐】** 文件存储使用对象存储（OSS/S3），不保存在应用服务器。

---

## (七) RESTful 风格

1. **【推荐】** 遵循 RESTful 设计原则：
   - GET：查询资源
   - POST：创建资源
   - PUT：更新资源（全量）
   - PATCH：更新资源（部分）
   - DELETE：删除资源

2. **【推荐】** 使用复数名词表示资源集合。
   - 正例：`/api/v1/orders`
   - 反例：`/api/v1/getOrder`

3. **【推荐】** 使用 HTTP 状态码表达结果：
   - 200：成功
   - 201：创建成功
   - 204：删除成功（无返回内容）
   - 400：参数错误
   - 401：未认证
   - 403：无权限
   - 404：资源不存在
   - 500：服务器错误
