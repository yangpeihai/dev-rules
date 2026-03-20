# Proto 定义规范

## .proto 文件结构

```protobuf
syntax = "proto3";

package user;
option go_package = "./user";

// 导入其他 proto 文件
import "google/protobuf/empty.proto";

// 消息定义
message GetUserReq {
    int64 id = 1;
}

message GetUserResp {
    int64 id = 1;
    string username = 2;
    string email = 3;
}

// 服务定义
service UserService {
    rpc GetUser(GetUserReq) returns(GetUserResp);
    rpc ListUsers(google.protobuf.Empty) returns(ListUsersResp);
}
```

## 消息命名规范

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| 请求消息 | `{操作}Req` | `GetUserReq`, `CreateOrderReq` |
| 响应消息 | `{操作}Resp` | `GetUserResp`, `CreateOrderResp` |
| 数据消息 | `{实体}Info` 或 `{Entity}` | `UserInfo`, `Order` |

## 字段编号规则

```protobuf
// 预留字段编号
message Message {
    // 1-15: 单字节编码，常用字段
    string id = 1;
    string name = 2;

    // 16-2047: 双字节编码，不常用字段
    repeated string tags = 16;

    // 预留：19000-19999（protobuf 内部使用）
    // 避免使用这些编号
}
```

## 服务定义规范

```protobuf
// 单服务定义
service UserService {
    // CRUD 操作
    rpc CreateUser(CreateUserReq) returns(CreateUserResp);
    rpc GetUser(GetUserReq) returns(GetUserResp);
    rpc UpdateUser(UpdateUserReq) returns(UpdateUserResp);
    rpc DeleteUser(DeleteUserReq) returns(google.protobuf.Empty);

    // 批量操作
    rpc BatchGetUsers(BatchGetUsersReq) returns(BatchGetUsersResp);
    rpc ListUsers(ListUsersReq) returns(ListUsersResp);
}

// 多服务定义（使用 -m 参数）
service UserService {
    rpc GetUser(GetUserReq) returns(GetUserResp);
}

service OrderService {
    rpc CreateOrder(CreateOrderReq) returns(CreateOrderResp);
}
```

## Go Package 配置

```protobuf
// 推荐：明确指定 go_package
syntax = "proto3";

package user.v1;                    // proto 包名
option go_package = "github.com/project/proto/user/v1;userv1";  // Go 包路径

// 多版本支持
// user/v1/user.proto -> option go_package = "github.com/project/proto/user/v1;userv1"
// user/v2/user.proto -> option go_package = "github.com/project/proto/user/v2;userv2"
```

## 常见类型映射

| Protobuf 类型 | Go 类型 | 说明 |
|---------------|---------|------|
| `double` | `float64` | 双精度浮点 |
| `float` | `float32` | 单精度浮点 |
| `int32` | `int32` | 32位整数 |
| `int64` | `int64` | 64位整数 |
| `uint32` | `uint32` | 无符号32位整数 |
| `uint64` | `uint64` | 无符号64位整数 |
| `bool` | `bool` | 布尔值 |
| `string` | `string` | 字符串 |
| `bytes` | `[]byte` | 字节数组 |
| `repeated T` | `[]T` | 切片 |
