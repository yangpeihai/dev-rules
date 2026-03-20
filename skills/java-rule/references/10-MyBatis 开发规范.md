# MyBatis/MyBatis-Plus 开发规范

## (一) SQL 编写方式选择

1. **【推荐】** 根据场景选择合适的 SQL 编写方式：
   | 场景 | 推荐方式 |
   |------|----------|
   | 简单 CRUD | MyBatis-Plus BaseMapper |
   | 单表条件查询 | MyBatis-Plus LambdaQueryWrapper |
   | 多表关联查询 | Mapper XML |
   | 复杂动态 SQL | Mapper XML |
   | 批量操作 | Mapper XML 或 MyBatis-Plus 批量方法 |

2. **【强制】** Mapper 接口继承 BaseMapper，获得基础 CRUD 能力。
   ```java
   @Mapper
   public interface OrderMapper extends BaseMapper<Order> {
       // 自定义复杂查询方法
       OrderDetailVo selectDetailById(Long orderId);
   }
   ```

---

## (二) SQL 编写规范

1. **【强制】** 禁止使用 SELECT *，必须明确列出字段。
   - 正例：
     ```sql
     SELECT id, order_no, user_id, total_amount, status
     FROM t_order
     WHERE id = #{orderId}
     ```
   - 反例：
     ```sql
     SELECT * FROM t_order WHERE id = #{orderId}
     ```

2. **【强制】** SQL 语句必须利用索引，避免全表扫描。
   - 说明：在索引列上避免使用函数、计算、隐式类型转换

3. **【强制】** 禁止在索引列上使用函数或计算。
   - 反例：
     ```sql
     SELECT * FROM t_order WHERE DATE(create_time) = #{date}
     ```
   - 正例：
     ```sql
     SELECT * FROM t_order
     WHERE create_time >= #{startTime}
       AND create_time < #{endTime}
     ```

4. **【推荐】** 使用 UNION ALL 替代 UNION（不去重场景）。

5. **【推荐】** 批量操作使用 IN 语句时，控制数量在 1000 以内。

---

## (三) 动态 SQL 规范

1. **【强制】** 使用 `<if>`、`<choose>`、`<foreach>` 等标签编写动态 SQL。
   - 正例：
     ```xml
     <select id="selectOrderList" resultMap="BaseResultMap">
         SELECT id, order_no, user_id, total_amount, status
         FROM t_order
         WHERE deleted = 0
         <if test="userId != null">
             AND user_id = #{userId}
         </if>
         <if test="status != null">
             AND status = #{status}
         </if>
         <choose>
             <when test="startTime != null and endTime != null">
                 AND create_time BETWEEN #{startTime} AND #{endTime}
             </when>
             <when test="startTime != null">
                 AND create_time &gt;= #{startTime}
             </when>
         </choose>
     </select>
     ```

2. **【强制】** 使用 `<set>` 标签进行动态更新。
   ```xml
   <update id="updateOrderStatus">
       UPDATE t_order
       <set>
           <if test="status != null">
               status = #{status},
           </if>
           <if test="remark != null">
               remark = #{remark},
           </if>
           update_time = NOW()
       </set>
       WHERE id = #{orderId}
   </update>
   ```

3. **【推荐】** XML 文件命名与 Mapper 接口保持一致，放在 `resources/mapper/` 目录下。

---

## (四) ResultMap 映射规范

1. **【强制】** 定义 ResultMap 明确字段与属性映射关系。
   ```xml
   <resultMap id="BaseResultMap" type="com.sf.order.entity.Order">
       <id column="id" property="id"/>
       <result column="order_no" property="orderNo"/>
       <result column="user_id" property="userId"/>
       <result column="total_amount" property="totalAmount"/>
       <result column="status" property="status"/>
       <result column="create_time" property="createTime"/>
       <result column="update_time" property="updateTime"/>
       <result column="deleted" property="deleted"/>
   </resultMap>
   ```

2. **【推荐】** 使用 `extends` 属性复用 ResultMap。
   ```xml
   <resultMap id="OrderWithItemsMap" type="com.sf.order.vo.OrderDetailVo"
              extends="BaseResultMap">
       <collection property="items" ofType="com.sf.order.vo.OrderItemVo"
                   column="id" select="selectItemsByOrderId"/>
   </resultMap>
   ```

3. **【推荐】** 一对多关联查询使用 `<collection>` 标签。

4. **【推荐】** 一对一关联查询使用 `<association>` 标签。

---

## (五) 分页查询规范

1. **【强制】** 使用 MyBatis-Plus 分页或 PageHelper 分页，禁止内存分页。
   - 正例：
     ```java
     Page<Order> page = new Page<>(queryDto.getPage(), queryDto.getSize());
     LambdaQueryWrapper<Order> wrapper = new LambdaQueryWrapper<>();
     // 构建查询条件
     Page<Order> orderPage = orderMapper.selectPage(page, wrapper);
     ```

2. **【推荐】** 大表分页使用延迟关联优化。
   ```sql
   SELECT a.* FROM t_order a
   INNER JOIN (
       SELECT id FROM t_order
       WHERE condition
       ORDER BY create_time DESC
       LIMIT 100000, 10
   ) b ON a.id = b.id
   ```

3. **【推荐】** 限制最大分页大小（如 1000），避免深度分页。

---

## (六) MyBatis-Plus 使用规范

1. **【强制】** Entity 定义规范：
   ```java
   @Data
   @TableName("t_order")
   public class Order {
       @TableId(type = IdType.AUTO)
       private Long id;

       private String orderNo;
       private Long userId;

       @TableField(fill = FieldFill.INSERT)
       private LocalDateTime createTime;

       @TableField(fill = FieldFill.INSERT_UPDATE)
       private LocalDateTime updateTime;

       @TableLogic
       @TableField(fill = FieldFill.INSERT)
       private Integer deleted;
   }
   ```

2. **【推荐】** Service 继承 IService，获得通用 CRUD 方法。
   ```java
   public interface OrderService extends IService<Order> {
       // 扩展业务方法
   }
   ```

3. **【推荐】** 使用 LambdaQueryWrapper 避免字段名写错。
   ```java
   List<Order> orders = lambdaQuery()
       .eq(Order::getUserId, 1001L)
       .ge(Order::getCreateTime, LocalDateTime.now().minusDays(7))
       .orderByDesc(Order::getCreateTime)
       .list();
   ```

4. **【推荐】** 逻辑删除使用 `@TableLogic` 注解。

5. **【推荐】** 自动填充使用 `MetaObjectHandler`。

---

## (七) 批量操作规范

1. **【强制】** 批量插入使用 XML 批量语法或 MyBatis-Plus 的 `saveBatch`。
   ```xml
   <insert id="batchInsert">
       INSERT INTO t_order (order_no, user_id, total_amount)
       VALUES
       <foreach collection="orders" item="order" separator=",">
           (#{order.orderNo}, #{order.userId}, #{order.totalAmount})
       </foreach>
   </insert>
   ```

2. **【推荐】** 批量操作分批次提交，每批 500-1000 条。

3. **【推荐】** 批量更新使用 CASE WHEN 语法。

---

## (八) 数据库连接池配置

1. **【强制】** 使用 HikariCP 连接池。
   ```yaml
   spring:
     datasource:
       type: com.zaxxer.hikari.HikariDataSource
       hikari:
         pool-name: OrderHikariPool
         minimum-idle: 10
         maximum-pool-size: 20
         idle-timeout: 600000
         max-lifetime: 1800000
         connection-timeout: 30000
   ```

2. **【推荐】** 连接池参数建议：
   | 参数 | 建议值 | 说明 |
   |------|--------|------|
   | maximum-pool-size | 10-20 | 根据数据库连接数限制设置 |
   | minimum-idle | 与 max 相同或略低 | 保持连接池大小稳定 |
   | connection-timeout | 30000 | 获取连接的最大等待时间 |
   | idle-timeout | 600000 | 空闲连接回收时间 |
   | max-lifetime | 1800000 | 连接最大存活时间 |

3. **【推荐】** 配置连接测试查询。
   ```yaml
   connection-test-query: SELECT 1
   ```

---

## (九) 代码生成器

1. **【推荐】** 使用 MyBatis-Plus 代码生成器快速生成基础代码。
   ```java
   FastAutoGenerator.create("jdbc:mysql://localhost:3306/order_db", "root", "password")
       .globalConfig(builder -> builder
           .author("zhangsan")
           .outputDir(System.getProperty("user.dir") + "/src/main/java")
           .dateType(DateType.TIME_PACK)
       )
       .packageConfig(builder -> builder
           .parent("com.sf.order")
           .entity("entity")
           .mapper("mapper")
           .service("service")
           .serviceImpl("service.impl")
       )
       .strategyConfig(builder -> builder
           .addInclude("t_order", "t_order_item")
           .addTablePrefix("t_")
           .entityBuilder()
               .enableLombok()
               .enableTableFieldAnnotation()
       )
       .execute();
   ```

2. **【推荐】** 生成代码后进行人工审查，补充业务逻辑。
