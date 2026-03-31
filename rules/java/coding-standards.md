# Java 编码标准

> 配合 `java-rule` skill 使用

## 工具链

- 遵循阿里巴巴 Java 开发手册
- Lint：SonarQube / PMD / SpotBugs

## 具体规则

- 类名：UpperCamelCase
- 方法/变量：lowerCamelCase
- 使用 SLF4J 记录日志（禁止 System.out.println）
- 使用 ThreadPoolExecutor 创建线程池
- 避免 NPE（空指针异常）
