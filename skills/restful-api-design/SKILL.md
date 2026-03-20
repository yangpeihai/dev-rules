---
name: "restful-api-design"
description: "RESTful API设计规范指南。Invoke when user needs help designing REST APIs, including URL structure, HTTP methods, request/response formats, status codes, pagination, authentication, error handling, and API documentation."
---

# RESTful API 设计规范

## 概述

本规范定义了企业级RESTful API的设计标准，涵盖URL设计、HTTP方法、请求响应格式、状态码、分页、认证授权、错误处理和文档编写等内容。

## URL 设计规范

### 基本格式

```
https://api.example.com/{version}/{resource}/{resource-id}/{sub-resource}
```

**正确示例**：
```
https://api.example.com/v1/users
https://api.example.com/v1/users/123/orders
```

**错误示例**：
```
https://api.example.com/getUsers          # 错误：包含动词
https://api.example.com/v1/user           # 错误：资源名应为复数
https://api.example.com/V1/Users          # 错误：大小写混用
```

### 资源命名规范

| 规范项 | 要求 | 正确示例 | 错误示例 |
|--------|------|----------|----------|
| 形式 | 名词复数 | `/users` | `/user` |
| 大小写 | 全小写 | `/user-profiles` | `/UserProfiles` |
| 分隔符 | 连字符（-） | `/user-groups` | `/user_groups` |
| 层级关系 | 使用路径表达 | `/users/123/orders` | `/users/123?type=orders` |

### 版本控制

版本号必须放在URL路径中：`/api/v1/users`

版本号格式：`v{主版本号}`，如 `v1`、`v2`。

## HTTP 方法语义

| 方法 | 语义 | 幂等性 | 用途 | 响应体 |
|------|------|--------|------|--------|
| GET | 获取资源 | 是 | 查询资源或资源列表 | 返回资源数据 |
| POST | 创建资源 | 否 | 创建新资源 | 返回创建的资源 |
| PUT | 全量更新 | 是 | 完整替换资源 | 返回更新后的资源 |
| PATCH | 部分更新 | 否 | 部分修改资源 | 返回更新后的资源 |
| DELETE | 删除资源 | 是 | 删除资源 | 通常无响应体 |

### 使用示例

```http
GET    /users           # 查询用户列表
POST   /users           # 创建用户
GET    /users/123       # 查询特定用户
PUT    /users/123       # 全量更新用户
PATCH  /users/123       # 部分更新用户
DELETE /users/123       # 删除用户
```

## 统一响应结构

所有API响应必须使用统一的JSON结构：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": { ... },
  "timestamp": "2024-01-15T09:30:00Z",
  "requestId": "req_abc123xyz",
  "traceId": "trace_def456uvw"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| code | Integer | 是 | 业务状态码，200表示成功 |
| message | String | 是 | 状态描述信息 |
| data | Object/Array | 否 | 响应数据，可为null |
| timestamp | String | 是 | ISO 8601格式时间戳 |
| requestId | String | 推荐 | 请求唯一标识 |
| traceId | String | 推荐 | 分布式追踪ID |

## 状态码规范

### 2xx 成功状态码

| 状态码 | 场景 | 说明 |
|--------|------|------|
| 200 | 通用成功 | GET、PUT、PATCH成功 |
| 201 | 创建成功 | POST创建资源成功 |
| 204 | 无内容 | DELETE成功，或无返回数据 |

### 4xx 客户端错误

| 状态码 | 场景 | 说明 |
|--------|------|------|
| 400 | 请求参数错误 | 参数缺失、格式错误、验证失败 |
| 401 | 未认证 | 缺少认证信息或认证失败 |
| 403 | 禁止访问 | 已认证但无权限 |
| 404 | 资源不存在 | 请求的资源不存在 |
| 409 | 资源冲突 | 资源已存在或状态冲突 |
| 422 | 语义错误 | 请求格式正确但语义错误 |
| 429 | 请求过多 | 触发限流 |

### 5xx 服务端错误

| 状态码 | 场景 | 说明 |
|--------|------|------|
| 500 | 内部错误 | 服务端未预期的错误 |
| 502 | 网关错误 | 上游服务无响应 |
| 503 | 服务不可用 | 服务暂时不可用 |
| 504 | 网关超时 | 上游服务响应超时 |

## 分页规范

### 分页参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| page | Integer | 1 | 当前页码，从1开始 |
| size | Integer | 20 | 每页条数，最大100 |

### 分页响应结构

```json
{
  "code": 200,
  "message": "查询成功",
  "data": {
    "list": [ ... ],
    "pagination": {
      "current": 1,
      "size": 20,
      "total": 100,
      "pages": 5
    }
  }
}
```

## 过滤、排序、搜索

### 过滤规范

```http
GET /users?status=active
GET /users?age_gte=18&age_lte=60
```

| 操作符 | 含义 | 示例 |
|--------|------|------|
| `_eq` | 等于 | `status_eq=active` |
| `_ne` | 不等于 | `status_ne=deleted` |
| `_gt` | 大于 | `age_gt=18` |
| `_gte` | 大于等于 | `age_gte=18` |
| `_lt` | 小于 | `age_lt=60` |
| `_lte` | 小于等于 | `age_lte=60` |
| `_in` | 包含在列表中 | `status_in=active,pending` |
| `_like` | 模糊匹配 | `name_like=zhang` |

### 排序规范

```http
GET /users?sort=createdAt           # 按createdAt升序
GET /users?sort=-createdAt           # 按createdAt降序
GET /users?sort=-priority,createdAt  # 多字段排序
```

### 搜索规范

```http
GET /users?q=zhangsan
GET /products?keyword=手机
GET /users?fields=id,username,email  # 字段选择
```

## 认证与授权

### 认证方式

优先使用Bearer Token方式：

```http
GET /users HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### JWT Token规范

JWT Token应包含以下声明：iss（签发者）、sub（用户唯一标识）、aud（接收方）、exp（过期时间）、iat（签发时间）、jti（Token唯一标识）、scope（权限范围）。

## 错误处理规范

### 错误响应结构

```json
{
  "code": 400001,
  "message": "请求参数错误",
  "data": {
    "errors": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "邮箱格式不正确"
      }
    ]
  },
  "timestamp": "2024-01-15T09:30:00Z",
  "requestId": "req_abc123",
  "documentation": "https://api.example.com/docs/errors/400001"
}
```

### 错误码设计

分层设计：`{HTTP状态码}{服务标识}{错误序号}`

| 错误码 | 说明 |
|--------|------|
| 400001 | 通用参数错误 |
| 400002 | JSON解析错误 |
| 401001 | Token过期 |
| 401002 | Token无效 |
| 403001 | 权限不足 |
| 404001 | 资源不存在 |
| 500001 | 内部服务器错误 |

**注意**：不要暴露敏感信息（堆栈跟踪、SQL语句等）。

## 限流与熔断策略

### 限流响应头

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
X-RateLimit-Window: 3600
```

### 限流策略

| 策略 | 场景 | 配置示例 |
|------|------|----------|
| 全局限流 | 保护整个系统 | 10000请求/分钟 |
| 用户限流 | 防止滥用 | 100请求/分钟/用户 |
| 接口限流 | 保护特定接口 | 10请求/秒/接口 |

## 幂等性设计

### 幂等键机制

POST请求使用`Idempotency-Key`头实现幂等：

```http
POST /orders HTTP/1.1
Content-Type: application/json
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "productId": "123",
  "quantity": 2,
  "amount": 199.99
}
```

幂等键规则：
1. 由客户端生成唯一标识（UUID推荐）
2. 服务端缓存幂等键及响应（建议24小时）
3. 相同幂等键的请求返回相同响应

## API文档规范

### 必填内容

| 内容 | 说明 |
|------|------|
| 接口描述 | 清晰描述接口功能 |
| 请求参数 | 参数名、类型、必填、默认值、说明 |
| 请求示例 | 完整的请求示例 |
| 响应结构 | 响应字段的详细说明 |
| 响应示例 | 成功和失败的响应示例 |
| 错误码 | 接口可能返回的错误码及说明 |
| 权限要求 | 需要的认证和权限 |

## 快速检查清单

- [ ] URL使用名词复数且全小写
- [ ] HTTP方法使用正确
- [ ] 响应格式符合统一结构
- [ ] HTTP状态码使用正确
- [ ] 分页参数和响应符合规范
- [ ] 错误码设计符合分层规范
- [ ] 接口文档已更新
- [ ] 幂等性已正确处理
- [ ] 限流和熔断已配置
