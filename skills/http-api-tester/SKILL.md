---
name: http-api-tester
description: Use when testing HTTP API endpoints, verifying API changes, generating API test cases or execution plans, running HTTP requests for pass/fail evidence, or analyzing why an HTTP API test failed.
---

# HTTP API 测试助手

帮助 AI 或开发者在限定范围内完成 HTTP API 的测试设计、请求执行、结果记录与失败归因，并输出可复测、可追踪、可交接的结构化结果。

## Overview

这个 skill 的重点不是“尽量多发请求”，而是“在正确范围内，把 HTTP API 测试做成一个闭环”。

它默认服务于两类场景：
- 单轮快速验证：围绕单个 HTTP 接口设计测试点、执行请求、记录结果
- 回归与交接：围绕接口组或改动点形成可继续复测、修复、追踪的结构化结果

## AI 速用入口

如果只看这一节，也应按下面执行：

1. 先确认测试范围，不擅自扩大
2. 再识别输入来源并归一化接口信息
3. 再判断是否满足最小执行条件
4. 再选择最相关的测试点类别
5. 执行请求并记录证据
6. 输出通过、失败、阻塞与后续建议

### 最短输出模板

```markdown
## Test Result
- [API-P001][passed] POST /api/v1/orders
  测试点：
  断言：
  证据：

- [API-F001][failed] POST /api/v1/orders
  测试点：
  预期：
  实际：
  归因：
  下一步：

## Summary
- 通过：
- 失败：
- 阻塞：
```

### 速停条件

出现以下情况时，不要继续盲目发请求：
- 接口地址、鉴权、请求体等关键输入缺失
- 当前任务只要求设计测试点，不要求实际执行
- 结果已经明确需要先补环境或测试数据
- 失败根因明显属于环境阻塞，继续重试价值很低

## AI 触发条件

当任务满足以下任一条件时，应优先使用本规范：
- 测试某个 HTTP API 接口
- 验证某次接口改动是否回归
- 根据接口文档、OpenAPI、curl、代码生成测试方案
- 执行 HTTP 请求并判断接口是否符合预期
- 输出接口测试结果并分析失败原因

## AI 决策顺序

按以下顺序处理，避免一上来就机械发请求：

1. 先确认测试范围：单接口、接口组、变更点回归，还是只做方案设计
2. 再确认输入来源：文档、OpenAPI、curl、代码、日志、环境信息
3. 再把输入归一化为统一测试对象
4. 再判断是否具备执行条件
5. 再选择最相关的测试点类别
6. 再执行并记录证据
7. 最后输出结果、归因和下一步建议

## 核心边界

### 必须遵守

- 只做 HTTP API 测试，不擅自扩展成全系统排查
- 只在用户允许的范围内测试
- 发起请求前先确认最小必要输入
- 每个失败项都要给出证据和归因方向
- 无法安全执行时，必须明确标记阻塞原因

### 不应做的事

- 不把接口测试扩展成 UI 自动化、压测、MQ、RPC 或渗透测试
- 不自行编造 Header、Token、请求参数、请求体或预期结果
- 不因为接口返回了状态码就宣称测试完成
- 不把环境问题直接断言为服务端业务问题
- 不在缺少前置条件时无休止重试

## 输入归一化

支持以下输入来源，按可用性收敛：

1. 用户本轮明确给出的接口信息
2. API 文档或 OpenAPI / Swagger
3. curl 或 HTTP 请求示例
4. 路由、handler、controller 等代码
5. 环境信息、鉴权说明、测试数据说明
6. 历史响应样例、日志、错误截图

无论来源如何，都先归一化为统一测试对象：

| 字段 | 说明 |
|------|------|
| `method` | HTTP 方法 |
| `url` / `path` | 完整地址或路径 |
| `query` | 查询参数 |
| `headers` | 请求头 |
| `auth` | 鉴权方式与所需凭证 |
| `request_body` | 请求体 |
| `content_type` | 内容类型 |
| `preconditions` | 前置条件 |
| `expected_status` | 预期状态码 |
| `expected_response_fields` | 预期响应字段或结构 |
| `data_dependencies` | 测试数据依赖 |
| `environment_dependencies` | 环境依赖 |

如果关键字段不全，应先补足理解，无法补足时标记为 `blocked`，而不是靠猜补齐。

## 测试点分类模型

HTTP API 测试点默认按以下类别组织，避免退化成只测 `200 OK`：

| 类别 | 重点关注 |
|------|------|
| `contract` | 方法、路径、Header、Content-Type、鉴权方式、状态码、响应结构 |
| `positive` | 合法输入下的成功主路径 |
| `validation` | 必填缺失、类型错误、格式非法、枚举越界、长度越界 |
| `auth` | 未登录、Token 无效、Token 过期、权限不足、租户隔离 |
| `state` | 资源不存在、重复创建、重复提交、状态冲突、幂等性 |
| `boundary` | 最大最小值、分页边界、空集合、超长字符串、特殊字符 |
| `error_handling` | 上游失败、超时、内部错误映射、错误信息脱敏 |
| `regression` | 围绕改动点补最小必要回归 |

### 裁剪原则

- 单轮快速验证时，优先 `contract`、`positive`、`validation`
- 涉及鉴权时，补 `auth`
- 涉及创建、更新、删除等状态变化时，补 `state`
- 涉及改动回归时，补 `regression`
- 只有在用户明确要求或接口特征明显需要时，才扩大到更多类别

## 执行前检查

发请求前至少检查以下条件：

- 目标环境是否明确
- 接口地址是否明确
- 请求方法是否明确
- 最小请求参数或请求体是否明确
- 必要 Header 和鉴权信息是否明确
- 至少一个核心断言目标是否明确

如果条件不足，仍然可以做：
- 设计测试点
- 输出待执行用例
- 标记阻塞项
- 明确说明缺的是什么

## 执行状态机

推荐统一使用以下状态：

| 状态 | 含义 |
|------|------|
| `pending` | 待处理 |
| `ready` | 信息足够，可以执行 |
| `blocked` | 缺少关键信息或环境条件，无法安全执行 |
| `passed` | 已执行且断言通过 |
| `failed` | 已执行且断言失败 |
| `needs_verification` | 已有部分证据，但仍需更多运行验证 |

### 状态判定要求

- 只有发起请求并完成最小断言，才可标记为 `passed`
- 已发起请求但断言不成立，标记为 `failed`
- 输入不足、环境不可达、缺少鉴权或缺少前置数据时，标记为 `blocked`
- 已观察到现象但证据不足以锁定结论时，标记为 `needs_verification`

## 测试执行闭环

对每个测试用例都按以下顺序处理：

1. 确认测试目标与类别
2. 组装最小可执行请求
3. 明确预期断言
4. 执行 HTTP 请求
5. 记录实际状态码、响应体、关键 Header、错误信息
6. 判定状态
7. 回填失败归因或阻塞原因
8. 给出下一步建议

不要跳过第 5 步和第 7 步。

## 失败归因模型

接口失败后，不要只说“接口失败”，而要判断失败更可能属于哪一层：

| 类型 | 说明 |
|------|------|
| `test_case_issue` | 用例本身设计不合理或预期错误 |
| `request_issue` | 请求构造错误，如 Header、参数、Body 不对 |
| `auth_or_env_issue` | 鉴权缺失、环境不可达、网关阻塞、配置问题 |
| `test_data_issue` | 缺少前置数据、数据状态不满足 |
| `contract_mismatch` | 接口实现与文档或约定不一致 |
| `server_logic_issue` | 服务端业务逻辑异常或校验缺失 |
| `downstream_issue` | 下游依赖异常、超时、错误透传 |
| `insufficient_evidence` | 现象存在，但证据不足以下结论 |

### 失败项最少应包含

- `expected`
- `actual`
- `evidence`
- `analysis`
- `next_action`

## 最小验证原则

每个已执行用例至少给出一项验证依据：

- 实际请求信息
- 实际状态码
- 响应体关键字段
- 错误提示或超时信息
- 与预期的差异点

如果当前只能做到部分验证，应明确写出：
- 还缺什么证据
- 为什么拿不到
- 下一步该怎么补

## 推荐用例字段

为了兼容单轮输出和后续复测，推荐每个用例尽量包含以下字段：

| 字段 | 说明 |
|------|------|
| `case_id` | 用例编号，如 `API-P001` |
| `category` | 测试类别 |
| `request` | 请求标识，如 `POST /api/v1/orders` |
| `summary` | 一句话说明测试点 |
| `expected` | 预期行为 |
| `actual` | 实际结果 |
| `status` | `passed` / `failed` / `blocked` / `needs_verification` |
| `evidence` | 核心证据 |
| `analysis` | 失败归因或阻塞分析 |
| `next_action` | 建议下一步 |

## 输出落盘规则

默认应将测试结果同时保存为文档，而不是只停留在对话输出中。

### 默认目录

- 默认输出目录：`docs/api-tests/`
- 如果目录不存在，应先创建目录，再写入结果文件
- 除非用户明确指定其他位置，否则不要把测试结果直接散落到 `docs/` 根目录

### 默认文件格式

- 默认同时输出两份文件：
- 一份 Markdown：给人快速阅读和 review
- 一份 JSON：给后续 AI、脚本或复测流程继续消费

### 文件命名规则

默认文件名应包含以下信息：

1. 日期
2. 时间
3. 用户本轮任务主题
4. 从当前分支名提取的简短特性名

推荐格式：

- Markdown：`YYYY-MM-DD-HHMMSS-<task-topic>-<branch-feature>.md`
- JSON：`YYYY-MM-DD-HHMMSS-<task-topic>-<branch-feature>.json`

示例：

- `2026-04-02-153000-create-order-api-http-api-tester.md`
- `2026-04-02-153000-create-order-api-http-api-tester.json`

### 命名生成原则

- `task-topic` 优先使用用户本轮任务主题，转成简短、可读、短横线连接的英文或拼音标识
- `branch-feature` 作为补充信息，从当前分支名中提取更短的特性名
- 如果分支名本身已经较短且清晰，可直接使用
- 如果分支名包含噪音前缀、用户名、流水号等信息，应只保留最能表达特性的那一段
- 如果当前无法可靠提取分支特性名，可退化为完整分支名的短横线形式，但应优先保持简短

### 落盘要求

- 对话中给出结果摘要时，也应同步说明写入了哪些文件
- 如果只完成了部分测试，也应落盘当前结果，而不是等全部完成后再写
- Markdown 与 JSON 的核心结论必须保持一致
- 文件中必须包含测试范围、执行时间、环境信息、结果汇总、失败归因和下一步建议

## 双模式输出

### 快速执行模式

适用场景：
- 单接口快速验证
- 对话内临时复测
- 小范围变更确认

输出要求：
- 先列结果，再给总结
- 每项至少写测试点、状态、关键断言、证据
- 失败项必须写归因
- 阻塞项必须写缺口
- 默认同步生成 Markdown 文档，并在可行时同步生成对应 JSON 文件

### 结构化回填模式

适用场景：
- 接口组回归
- 需要交给人或下游 AI 继续处理
- 需要沉淀测试记录

输出要求：
- 保留稳定字段
- 区分通过、失败、阻塞、需验证
- 失败项可直接供修复或复测继续消费
- 默认落盘为 JSON，并同时补一份 Markdown 摘要

## 推荐输出模板

### Markdown 快速模板

```markdown
# API Test Report

## Metadata
- 任务主题：create-order-api
- 分支特性：http-api-tester
- 执行时间：2026-04-02 15:30:00
- 测试范围：POST /api/v1/orders
- 环境：

## Test Result
- [API-P001][passed] POST /api/v1/orders
  测试点：合法创建订单
  断言：状态码 201，返回 orderId
  证据：HTTP 201，响应 data.orderId 存在

- [API-F001][failed] POST /api/v1/orders
  测试点：缺少必填参数 userId
  预期：HTTP 400，返回参数错误
  实际：HTTP 500，message=internal error
  归因：更可能是服务端参数校验缺失
  下一步：检查请求绑定和参数校验逻辑

- [API-B001][blocked] POST /api/v1/orders
  测试点：已登录用户创建订单
  阻塞：缺少可用 Token
  下一步：补充测试账号或鉴权凭证

## Summary
- 通过：1
- 失败：1
- 阻塞：1

## Next Actions
- 
```

### JSON 结构化模板

```json
{
  "metadata": {
    "protocol": "http",
    "task_topic": "create-order-api",
    "branch_feature": "http-api-tester",
    "executed_at": "2026-04-02T15:30:00+08:00",
    "scope": "POST /api/v1/orders",
    "source": "openapi + user headers",
    "total_cases": 3,
    "passed": 1,
    "failed": 1,
    "blocked": 1
  },
  "cases": [
    {
      "case_id": "API-F001",
      "category": "validation",
      "request": "POST /api/v1/orders",
      "summary": "缺少 userId 时返回 500 而不是 400",
      "expected": "HTTP 400，返回参数错误信息",
      "actual": "HTTP 500，message=internal error",
      "status": "failed",
      "evidence": "状态码与响应体已记录",
      "analysis": "更可能是服务端校验缺失，而不是客户端请求构造错误",
      "next_action": "检查 handler/controller 入参校验与错误映射"
    }
  ]
}
```

## AI 默认执行步骤

1. 明确测试范围和输出目标
2. 归一化接口信息
3. 判断是否可执行
4. 选择最相关测试点
5. 执行请求并记录证据
6. 判定通过、失败、阻塞或需验证
7. 对失败项给出归因和下一步建议
8. 将结果同时整理为 Markdown 和 JSON，并默认写入 `docs/api-tests/`

## AI 自检清单

输出前至少快速检查：

- [ ] 是否只测试了用户允许的范围
- [ ] 是否没有编造请求参数、鉴权或预期结果
- [ ] 是否不是只看状态码，而忽略关键响应断言
- [ ] 是否每个失败项都有证据和归因
- [ ] 是否每个阻塞项都明确写出缺口
- [ ] 是否区分了 `failed`、`blocked`、`needs_verification`
- [ ] 是否给出了可执行的下一步，而不是空泛建议
- [ ] 是否已将结果写入默认文档目录
- [ ] 是否文件名包含日期时间、任务主题和分支特性名
- [ ] 是否同时生成了 Markdown 和 JSON 两份结果

## 停止条件

出现以下情况时，应停止继续扩大结论：

- 测试范围不清，继续执行会超出授权
- 关键输入缺失，无法安全构造请求
- 环境、网络、鉴权、测试数据明显阻塞
- 当前问题本质上需要联调、日志、数据库或下游权限才能判断
- 多次重复请求已无法带来新增信息

此时应明确说明缺口，而不是继续猜测或硬测。

## 何时联动其他文档

- **需要判断接口设计是否合理** → `restful-api-design`
- **需要审查 Go 代码本身是否存在实现问题** → `code-checker-go`
- **需要根据失败结果继续修 Go 代码** → `code-fixer-go`
- **需要补充通用安全边界** → `secure-development`

## 常见误区

| 误区 | 正确做法 |
|------|------|
| 只测成功路径 | 至少补参数校验和关键失败路径 |
| 返回 200 就算通过 | 继续校验核心字段和业务语义 |
| 缺 Token 也先随便测 | 明确标记 `blocked`，不要伪造鉴权 |
| 环境超时就断言代码有 bug | 先区分环境、下游、网关和服务端逻辑 |
| 用例失败只贴响应 | 补上预期、实际差异、归因和下一步 |

## 最终目标

这个 skill 的产出应当满足两件事：

1. 人可以快速看懂哪些 HTTP API 用例已经测过、结论是什么
2. AI 可以直接基于结果继续复测、排查、修复或交接
