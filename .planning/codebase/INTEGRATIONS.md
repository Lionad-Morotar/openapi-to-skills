# 外部集成

**分析日期：** 2026-03-02

## API 和外部服务

**HTTP 客户端：**
- 原生 `fetch()` API - 用于从 URL 加载 OpenAPI 规范
  - 位置：`src/spec-loader.ts`（第 40-49 行）
  - 支持：HTTP 和 HTTPS 协议
  - 错误处理：HTTP 状态码验证

**无外部 API：**
- 没有第三方 API 集成（Stripe、Supabase、AWS 等）
- 独立的 CLI 工具

## 数据存储

**数据库：**
- 无 - 无状态的 CLI 工具

**文件存储：**
- 仅本地文件系统
  - 读取：OpenAPI 规范文件（JSON/YAML）
  - 写入：生成的 Markdown 输出
  - Node.js API：`node:fs`、`node:fs/promises`

**缓存：**
- 无

## 认证和身份

**认证提供方：**
- 无 - 无认证系统

**输入安全：**
- URL 验证，仅支持 HTTP/HTTPS 协议
- 通过 Node.js 路径工具解析文件路径
- 不处理凭证

## 监控和可观测性

**错误跟踪：**
- 无 - 仅基于控制台的错误报告

**日志：**
- consola 3.4.2 - 结构化控制台日志
  - 级别：错误、警告、信息、成功、开始
  - 支持安静模式（`--quiet` 标志）
  - 位置：`src/cli.ts`

## CI/CD 和部署

**托管：**
- npm 注册表（公共包）
- GitHub Releases（源代码）

**CI 流水线：**
- GitHub Actions
  - 工作流：`.github/workflows/ci.yml`
  - 任务：代码检查、测试（含覆盖率）、构建
  - 平台：Ubuntu Latest
  - 运行时：Bun（最新版）
  - 覆盖率：Codecov 集成

**发布：**
- 触发：Git 标签（`v*`）
- 平台：带来源证明的 npm
- Node.js：版本 20
- 认证：`NPM_TOKEN` 密钥

## 环境配置

**必需的环境变量：**
- 运行时操作不需要

**CI/CD 密钥：**
- `CODECOV_TOKEN` - 覆盖率报告（仅 CI）
- `NPM_TOKEN` - 包发布（仅发布工作流）

**密钥位置：**
- GitHub Secrets（仓库设置）
- 不在代码库中

## Webhook 和回调

**入站：**
- 无 - CLI 工具，无服务器组件

**出站：**
- HTTP fetch 用于从 URL 加载 OpenAPI 规范
  - 无认证头
  - 无重试逻辑
  - 仅基本错误处理

## 包分发

**注册表：**
- npm：https://www.npmjs.com/package/openapi-to-skills

**安装：**
```bash
npx openapi-to-skills <spec> -o <output>
bunx openapi-to-skills <spec> -o <output>
npm install -g openapi-to-skills
```

**二进制文件：**
- 入口：`dist/src/cli.js`
- 命令：`openapi-to-skills`

---

*集成审计：2026-03-02*
