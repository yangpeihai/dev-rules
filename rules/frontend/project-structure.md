# 前端项目结构（React 风格）

```
project/
├── src/
│   ├── assets/          # 静态资源 (图片, 字体, 样式)
│   ├── components/      # 公共 UI 组件
│   ├── pages/           # 页面级组件 (路由对应页面)
│   ├── hooks/           # 自定义 React Hooks
│   ├── services/        # API 请求与网络服务封装
│   ├── store/           # 状态管理 (Redux/Zustand)
│   ├── utils/           # 通用工具函数
│   ├── types/           # TypeScript 类型定义接口
│   ├── App.tsx          # 根组件，路由配置或全局 Provider 包装
│   └── main.tsx         # 入口文件，挂载 React 根节点
├── public/              # 不需要构建的公共资源
├── package.json         # 项目元数据与依赖
├── tsconfig.json        # TypeScript 配置
├── vite.config.ts       # Vite 配置
├── .eslintrc.json       # ESLint 规则配置
├── .prettierrc          # Prettier 代码格式化配置
├── .gitignore           # Git 忽略文件列表
└── README.md            # 项目说明
```
