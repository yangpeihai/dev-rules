# API 定义规范

## .api 文件结构

```go
syntax = "v1"

// 服务信息
info(
    title: "用户服务"
    desc: "用户相关接口"
    author: "author"
    version: "v1.0"
)

// 导入其他 API 文件
import "base.api"

// 类型定义
type (
    // 请求类型
    LoginReq {
        Username string `json:"username" validate:"required"`
        Password string `json:"password" validate:"required,min=6"`
    }

    // 响应类型
    LoginResp {
        AccessToken string `json:"accessToken"`
        ExpireTime  int64  `json:"expireTime"`
    }
)

// 服务定义
@server(
    group: user
    prefix: /user
    middleware: AuthMiddleware
)
service user-api {
    @doc "用户登录"
    @handler login
    post /login (LoginReq) returns (LoginResp)

    @doc "获取用户信息"
    @handler getUserInfo
    get /info (GetUserInfoReq) returns (GetUserInfoResp)
}
```

## 类型定义规范

### 基本类型

| API 类型 | Go 类型 | 说明 |
|----------|---------|------|
| `string` | `string` | 字符串 |
| `int` | `int` | 整数 |
| `int64` | `int64` | 长整数 |
| `float` | `float64` | 浮点数 |
| `bool` | `bool` | 布尔值 |
| `[]T` | `[]T` | 切片/数组 |

### 字段标签

```go
type UserReq struct {
    // JSON 标签
    Username string `json:"username"`

    // 可选字段
    Email string `json:"email,optional"`

    // 带默认值
    Page int `json:"page,default=1"`

    // 路径参数
    Id int64 `path:"id"`

    // 查询参数
    Keyword string `form:"keyword"`

    // 参数校验
    Password string `json:"password" validate:"required,min=6,max=32"`

    // 正则校验
    Phone string `json:"phone" validate:"regexp=^1[3-9]\\d{9}$"`
}
```

### 复合类型

```go
// 嵌套结构
type Address struct {
    Province string `json:"province"`
    City     string `json:"city"`
}

type CreateUserReq {
    Username string   `json:"username"`
    Address  Address  `json:"address"`
}

// 切片类型
type BatchDeleteReq {
    Ids []int64 `json:"ids"`
}
```

## 路由定义规范

### HTTP 方法

```go
service user-api {
    // GET 请求
    @doc "获取用户列表"
    @handler list
    get /list (ListReq) returns (ListResp)

    // POST 请求
    @doc "创建用户"
    @handler create
    post /create (CreateReq) returns (CreateResp)

    // PUT 请求
    @doc "更新用户"
    @handler update
    put /update (UpdateReq) returns (UpdateResp)

    // DELETE 请求
    @doc "删除用户"
    @handler delete
    delete /delete/:id (DeleteReq) returns (DeleteResp)
}
```

### 路径参数

```go
// 单个路径参数
type GetUserReq {
    Id int64 `path:"id"`
}

@handler getUser
get /user/:id (GetUserReq) returns (GetUserResp)

// 多个路径参数
type GetResourceReq {
    ResourceId int64 `path:"resourceId"`
    SubId      int64 `path:"subId"`
}

@handler getResource
get /resource/:resourceId/sub/:subId (GetResourceReq) returns (GetResourceResp)
```

### 查询参数

```go
type ListReq {
    Page    int64  `form:"page,default=1"`
    Limit   int64  `form:"limit,default=20"`
    Keyword string `form:"keyword,optional"`
}

@handler list
get /list (ListReq) returns (ListResp)
```

## 服务分组规范

```go
// 用户服务组
@server(
    group: user
    prefix: /api/v1/user
    middleware: AuthMiddleware, LogMiddleware
)
service user-api {
    @handler login
    post /login (LoginReq) returns (LoginResp)

    @handler getUserInfo
    get /info (GetUserInfoReq) returns (GetUserInfoResp)
}

// 订单服务组
@server(
    group: order
    prefix: /api/v1/order
    middleware: AuthMiddleware
)
service user-api {
    @handler createOrder
    post /create (CreateOrderReq) returns (CreateOrderResp)

    @handler listOrders
    get /list (ListOrdersReq) returns (ListOrdersResp)
}
```

## JWT 鉴权

```go
// 定义 JWT 鉴权
@server(
    group: user
    prefix: /user
    jwt: Auth
)
service user-api {
    // 以下接口都需要 JWT 鉴权
    @handler getUserInfo
    get /info (GetUserInfoReq) returns (GetUserInfoResp)

    @handler updateProfile
    put /profile (UpdateProfileReq) returns (UpdateProfileResp)
}
```

## API 导入

```go
// base.api - 基础类型定义
syntax = "v1"

type (
    // 通用响应
    Response {
        Code int64  `json:"code"`
        Msg  string `json:"msg"`
        Data any    `json:"data,optional"`
    }

    // 分页请求
    PageReq {
        Page  int64 `form:"page,default=1"`
        Limit int64 `form:"limit,default=20"`
    }

    // 分页响应
    PageResp {
        Total int64 `json:"total"`
        List  any   `json:"list"`
    }
)

// user.api - 导入基础类型
syntax = "v1"

import "base.api"

type (
    GetUserReq {
        PageReq // 嵌入分页请求
    }
)
```

## API 注解说明

| 注解 | 位置 | 说明 | 示例 |
|------|------|------|------|
| `@server` | service 前面 | 服务配置 | `@server(group: user, prefix: /user)` |
| `@handler` | 路由前面 | Handler 名称 | `@handler login` |
| `@doc` | 路由前面 | 接口描述 | `@doc "用户登录"` |
| `@jwt` | server 或 service | JWT 鉴权 | `@jwt Auth` |
| `@middleware` | server | 中间件 | `@middleware AuthMiddleware` |
| `@group` | server | 分组名称 | `@group user` |
| `@prefix` | server | 路由前缀 | `@prefix /api/v1` |
