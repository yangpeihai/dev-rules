# 数据库操作规范

## AI 使用提示

- 当任务涉及事务、缓存、Model 扩展、批量写入或一致性问题时，先看本文件。
- 默认把事务边界放在 Logic 层；若事务出现在 Handler / Server，优先检查是否分层错误。
- 不要因为 goctl 或缓存特性而自动引入复杂度，先判断是否真的需要。

## 核心原则

- 数据访问通过 Model 层封装，Logic 负责业务编排与事务边界。
- 事务使用项目统一封装，不在不同模块各搞一套风格。
- 缓存策略以一致性、可维护性为先，不要为了框架特性盲目引入缓存。

## Model 生成

### 从 MySQL DDL 生成

```bash
# 从 SQL 文件生成 Model
goctl model mysql ddl --src ./deploy/sql/user.sql --dir ./model --cache

# 从数据库连接生成
goctl model mysql datasource \
    --url "user:password@tcp(127.0.0.1:3306/dbname" \
    --table "user" \
    --dir ./model \
    --cache
```

## 数据库事务处理

```go
func (l *CreateOrderLogic) CreateOrder(req *types.CreateOrderReq) error {
    // 使用项目统一的事务封装；go-zero 常见写法是 TransactCtx
    err := l.svcCtx.OrderModel.TransactCtx(l.ctx, func(ctx context.Context, session sqlx.Session) error {
        // 1. 创建订单
        order := &model.Order{
            UserId:  req.UserId,
            Product: req.Product,
            Amount:  req.Amount,
        }
        if err := l.svcCtx.OrderModel.Insert(ctx, session, order); err != nil {
            return err
        }

        // 2. 扣减库存
        if err := l.svcCtx.ProductModel.DecreaseStock(ctx, session, req.ProductId, req.Quantity); err != nil {
            return err
        }

        // 3. 扣减余额
        if err := l.svcCtx.UserModel.DecreaseBalance(ctx, session, req.UserId, req.Amount); err != nil {
            return err
        }

        return nil
    })

    return err
}
```

## 默认规则

- Model 优先复用 goctl 生成骨架，再通过扩展文件承载手写逻辑。
- 事务默认放在 Logic 层开启，Model 接收事务上下文或 session。
- 是否启用缓存取决于读写模式、一致性要求和运维成本。

## 缓存使用

默认优先复用 goctl 生成的缓存模型；如项目已有自定义缓存层，保持缓存键、失效策略和数据一致性约定即可，不必为了“纯 goctl”强行重构。

```go
// 使用 goctl --cache 生成的 Model
type UserModel struct {
    *cache.Cache            // 嵌入缓存
}

func (m *UserModel) FindOne(ctx context.Context, id int64) (*User, error) {
    // 1. 先查缓存
    var user User
    key := fmt.Sprintf("user:%d", id)

    err := m.Cache.Get(ctx, key, &user)
    if err == nil {
        return &user, nil
    }

    // 2. 缓存未命中，查数据库
    result, err := m.FindOneFromDB(ctx, id)
    if err != nil {
        return nil, err
    }

    // 3. 写入缓存
    m.Cache.Set(ctx, key, result)

    return result, nil
}
```

## 推荐实践

- 为事务入口补充失败分支和回滚场景测试。
- 对热点查询明确缓存键规范、TTL、主动失效策略。
- 对写路径评估唯一索引、幂等约束和事务边界，而不是只靠代码判断。

## 禁止事项

- ❌ 在 Handler / Server 层直接开启数据库事务
- ❌ 在事务中混入长耗时 HTTP / RPC 调用
- ❌ 不经评估就给所有查询套缓存
- ❌ 手写逻辑直接改动生成代码，导致后续再生成困难

## 审查清单

- [ ] 数据访问位于 Model 层，事务边界位于 Logic 层
- [ ] 事务封装符合项目统一约定
- [ ] 缓存策略与一致性要求匹配
- [ ] 生成代码与手写扩展边界清晰
