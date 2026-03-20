# 配置管理规范

## 配置文件命名

- **环境区分**: `{service}-{env}.yaml`
- **环境列表**: `dev`, `test`, `uat`, `prod`

```
etc/
├── user-api-dev.yaml
├── user-api-test.yaml
├── user-api-uat.yaml
└── user-api-prod.yaml
```

## API 服务配置结构

```go
// internal/config/config.go
package config

import "github.com/zeromicro/go-zero/rest"

type Config struct {
    rest.RestConf
    Mysql struct {
        DataSource string
    }
    Redis struct {
        Host     string
        Type     string
        Pass     string
    }
    Auth struct {
        AccessSecret string
        AccessExpire int64
    }
    UserRpc struct {
        Etcd struct {
            Hosts []string
            Key  string
        }
    }
}
```

## RPC 服务配置结构

```go
// internal/config/config.go
package config

import "github.com/zeromicro/go-zero/zrpc"

type Config struct {
    zrpc.RpcServerConf
    Mysql struct {
        DataSource string
    }
    Redis redis.RedisConf
}
```

## 配置文件示例

```yaml
# etc/user-api.yaml
Name: user-api
Host: 0.0.0.0
Port: 8888

Mysql:
  DataSource: user:password@tcp(127.0.0.1:3306/dbname?charset=utf8mb4&parseTime=true

Redis:
  Host: 127.0.0.1:6379
  Type: node
  Pass: ""

Auth:
  AccessSecret: your-secret-key
  AccessExpire: 86400

UserRpc:
  Etcd:
    Hosts:
      - 127.0.0.1:2379
    Key: user.rpc

Log:
  ServiceName: user-api
  Mode: console
  Path: /var/log/user-api
  Level: info
  Compress: false
  KeepDays: 7
  StackCooldownMillis: 100
```
