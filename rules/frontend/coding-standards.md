# 前端编码标准

> 配合 `frontend-design` skill 使用

## 基础规范

| 项 | 规则 |
|----|------|
| 缩进 | 2 空格 |
| 变量/函数命名 | camelCase |
| 组件/类名 | PascalCase |
| 常量 | UPPER_SNAKE_CASE |

## 工具链

- 格式化：Prettier
- Lint：ESLint

## 具体规则

- TypeScript 必须使用严格模式 (`strict: true`)
- 组件必须使用函数组件和 Hooks
- 避免在组件内定义大的内联函数，提取到外部或使用 useCallback
- 所有自定义 Hook 必须以 `use` 开头
- 禁止使用 `any` 类型，必要时使用 `unknown` 并进行类型收窄
