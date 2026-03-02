# 技术栈

**分析日期：** 2026-03-02

## 编程语言

**主要语言：**
- TypeScript 5.x - 所有源代码 (`src/**/*.ts`)
  - 目标版本：ES2022
  - 模块系统：NodeNext（ES 模块）
  - 启用严格模式及额外的安全检查

**次要语言：**
- YAML - OpenAPI 规范示例 (`examples/input/`)
- Markdown - 模板文件 (`templates/*.md.eta`)
- Eta 模板语法 - 模板文件

## 运行时环境

**执行环境：**
- Node.js 18+（根据 `engines` 字段的最低要求）
- Bun（开发和测试）

**包管理器：**
- Bun（开发首选）
  - 锁定文件：`bun.lock`
- pnpm（支持替代方案）
  - 锁定文件：`pnpm-lock.yaml`

## 框架

**核心框架：**
- Eta 4.5.0 - Markdown 生成的模板引擎
- citty 0.2.0 - 命令行界面（CLI）框架

**测试框架：**
- Bun 测试运行器 - 单元测试和端到端测试
  - 内置断言库
  - 内置模拟功能（`mock()`）
  - 端到端测试的快照测试

**构建/开发工具：**
- TypeScript 编译器（tsc）- 编译输出到 `dist/`
- Biome 2.3.12 - 代码检查和格式化

## 关键依赖

**核心依赖：**
- `eta@4.5.0` - Markdown 输出的模板渲染引擎
- `citty@0.2.0` - CLI 参数解析和命令结构
- `consola@3.4.2` - 日志工具（在 CLI 中使用）
- `yaml@2.8.0` - OpenAPI 规范的 YAML 解析

**基础设施：**
- `openapi-types@12.1.3` - OpenAPI 规范的 TypeScript 类型

**开发依赖：**
- `@biomejs/biome@2.3.12` - 快速的代码检查器和格式化工具
- `@types/node@20.x` - Node.js 类型定义
- `typescript@5.x` - TypeScript 编译器

## 配置

**TypeScript：**
- 配置：`tsconfig.json`
- 目标版本：ES2022
- 模块系统：NodeNext
- 严格模式及额外检查：
  - `noUncheckedIndexedAccess: true`
  - `noImplicitOverride: true`
  - `noFallthroughCasesInSwitch: true`

**构建：**
- 输出目录：`dist/`
- 包含源码映射和声明文件
- 模板复制到 `dist/templates/`

**代码检查/格式化：**
- 配置：`biome.json`
- 缩进：制表符
- 引号：双引号
- 导入组织：启用
- VCS 集成：支持 Git

**环境变量：**
- 运行时不需要环境变量
- CLI 仅使用参数和标志
- 不使用 `.env` 文件

## 平台要求

**开发环境：**
- Bun 1.0+（推荐）或 Node.js 18+
- Git

**生产环境：**
- Node.js 18+ 或 Bun
- 发布到 npm 注册表
- 支持 CommonJS 和 ESM 消费者

**构建输出：**
- `dist/` 目录中的编译 JavaScript
- 类型声明文件（`.d.ts`）
- 源码映射
- 捆绑的模板

---

*技术栈分析：2026-03-02*
