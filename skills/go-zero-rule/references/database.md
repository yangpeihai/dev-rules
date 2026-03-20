# 数据库操作规范

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
    // 使用事务
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

## 缓存使用

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
