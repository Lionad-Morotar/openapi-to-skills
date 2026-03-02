# 代码库结构

**分析日期：** 2026-03-02

## 目录布局

```
[项目根目录]/
├── src/                    # 源代码（TypeScript）
├── templates/              # 用于 Markdown 生成的 Eta 模板
├── e2e/                    # 端到端快照测试
├── examples/               # 测试夹具和预期输出
│   ├── input/              # OpenAPI 规范示例
│   └── output/             # 预期的生成 skill 输出
├── scripts/                # 构建/维护脚本
├── dist/                   # 编译后的 JavaScript（构建输出）
├── .planning/              # 规划文档（此目录）
├── index.ts                # 主入口点（程序化 API）
├── package.json            # 包清单
├── tsconfig.json           # TypeScript 配置
├── biome.json              # 代码检查/格式化配置
└── bun.lock / pnpm-lock.yaml # 锁定文件
```

## 目录用途

**src/：**
- 用途：所有 TypeScript 源代码
- 包含：带有同位置测试的核心实现文件
- 关键文件：
  - `cli.ts`：CLI 入口点
  - `converter.ts`：编排逻辑
  - `parser.ts`：OpenAPI → IR 转换
  - `renderer.ts`：IR → Markdown 转换
  - `writer.ts`：文件系统抽象
  - `spec-loader.ts`：输入加载（文件/HTTP）
  - `types.ts`：所有 TypeScript 接口

**templates/：**
- 用途：用于 Markdown 生成的 Eta 模板文件
- 包含：6 个模板文件
- 关键文件：
  - `skill.md.eta`：主 skill 索引文档
  - `resource.md.eta`：资源索引页面
  - `operation.md.eta`：单个操作详情
  - `schema.md.eta`：模式文档
  - `schema-index.md.eta`：模式组索引
  - `authentication.md.eta`：认证文档
- 生成：否
- 提交：是（运行时必需）

**e2e/：**
- 用途：端到端快照测试
- 包含：`snapshot.test.ts` - 全面的 CLI 输出比较
- 模式：针对 examples/input 运行 CLI，与 examples/output 比较

**examples/：**
- 用途：测试夹具和预期输出
- 结构：
  - `input/`：用于测试的 OpenAPI 规范（YAML/JSON）
  - `output/`：预期的生成 skill 目录结构
- 用途：E2E 测试验证实际输出与这些快照匹配

**scripts/：**
- 用途：构建和维护工具
- 可能包含：快照更新脚本、发布自动化

**dist/：**
- 用途：编译后的 JavaScript 输出
- 生成：是（`tsc` 构建输出）
- 提交：是（用于 npm 包分发）
- 内容：`.js`、`.d.ts` 文件，镜像 src 结构，加上复制的模板

## 关键文件位置

**入口点：**
- `src/cli.ts`：CLI 二进制入口点
- `index.ts`：程序化 API 入口点
- `src/converter.ts`：主转换编排

**配置：**
- `package.json`：包元数据、依赖项、脚本
- `tsconfig.json`：TypeScript 编译器设置
- `biome.json`：代码检查和格式化规则

**核心逻辑：**
- `src/types.ts`：所有 TypeScript 接口和类型
- `src/parser.ts`：OpenAPI 解析和 IR 生成
- `src/renderer.ts`：基于模板的 Markdown 渲染
- `src/writer.ts`：文件系统操作

**测试：**
- `src/*.test.ts`：单元测试（与源代码同位置）
- `e2e/snapshot.test.ts`：端到端快照测试

## 命名约定

**文件：**
- 源代码：`kebab-case.ts`（例如 `spec-loader.ts`）
- 测试：与源代码同位置，命名为 `source-name.test.ts`（例如 `parser.test.ts`）
- 模板：`purpose.md.eta`（例如 `skill.md.eta`）

**目录：**
- 小写，短横线命名（例如 `openapi-to-skills`）

**类：**
- PascalCase（例如 `TemplateRenderer`、`FileSystemWriter`）

**函数：**
- camelCase（例如 `createParser`、`toFileName`）

**接口：**
- PascalCase 描述性名称（例如 `SkillDocument`、`ParserOptions`）

## 在哪里添加新代码

**新功能（例如新输出格式）：**
- 主代码：`src/`（新文件或扩展现有文件）
- 类型：`src/types.ts`（添加到 IR 或选项）
- 模板：`templates/`（新的 `.md.eta` 文件）
- 测试：同位置单元测试 + 端到端测试用例

**新组件/模块：**
- 实现：`src/{module-name}.ts`
- 测试：`src/{module-name}.test.ts`
- 导出：如果公共 API，添加到 `index.ts`

**工具函数：**
- 共享辅助函数：添加到相关现有文件（例如 `renderer.ts` 中的 `toFileName`）
- 横切：新文件 `src/utils.ts`（如果有多个工具函数）

**新模板：**
- 添加到 `templates/{name}.md.eta`
- 添加方法到 `src/types.ts` 中的 `Renderer` 接口
- 在 `src/renderer.ts` 中实现
- 从 `src/converter.ts` 调用

## 特殊目录

**dist/：**
- 用途：分发的编译输出
- 生成：是（通过 `npm run build`）
- 提交：是（用于从 git 安装 npm）
- 注意：构建期间模板复制到这里（`cp -r templates dist/`）

**node_modules/：**
- 用途：依赖项
- 包管理器：支持 Bun（bun.lock）和 pnpm（pnpm-lock.yaml）

**examples/output/：**
- 用途：E2E 测试的快照预期
- 生成：是（通过 `bun run snapshot:update`）
- 提交：是（快照测试）
- 结构镜像实际 CLI 输出结构

---

*结构分析：2026-03-02*
