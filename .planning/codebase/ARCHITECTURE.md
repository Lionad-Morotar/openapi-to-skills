# 架构

**分析日期：** 2026-03-02

## 模式概述

**整体：** 基于中间表示（IR）的流水线式编译器架构

**关键特征：**
- **流水线模式**：数据流经不同的转换阶段（解析器 → 渲染器 → 写入器）
- **中间表示**：OpenAPI 规范在渲染前被转换为规范化的 IR（`SkillDocument`）
- **模板驱动**：输出生成使用支持覆盖的 Eta 模板
- **依赖注入**：核心组件接受接口（`Renderer`、`Writer`）以提高可测试性
- **纯函数**：解析器在不产生副作用的情况下将输入转换为输出

## 分层

**CLI 层：**
- 用途：命令行界面和参数解析
- 位置：`src/cli.ts`
- 包含：citty 命令定义、参数验证、用户反馈
- 依赖：`spec-loader.ts`、`converter.ts`、`renderer.ts`（用于 `toFileName`）
- 使用者：最终用户（二进制入口点）

**编排层：**
- 用途：协调转换流水线
- 位置：`src/converter.ts`
- 包含：`convertOpenAPIToSkill()` 函数、输出目录结构管理
- 依赖：`Parser`、`Renderer`、`Writer` 接口
- 使用者：CLI、程序化 API 消费者

**转换层：**
- 用途：OpenAPI 规范 → IR → Markdown
- 位置：`src/parser.ts`、`src/renderer.ts`
- 包含：
  - `Parser`：OpenAPI 规范 → `SkillDocument` IR
  - `TemplateRenderer`：通过 Eta 模板将 IR → Markdown
- 依赖：`types.ts`、`eta`（仅渲染器）
- 使用者：转换器

**I/O 层：**
- 用途：外部数据加载和文件系统操作
- 位置：`src/spec-loader.ts`、`src/writer.ts`
- 包含：
  - `loadSpecFromInput()`：支持格式检测的文件系统或 HTTP 加载
  - `FileSystemWriter`：异步文件系统操作
- 依赖：`node:fs/promises`、`node:path`、`yaml`
- 使用者：CLI（加载器）、转换器（写入器）

**类型层：**
- 用途：TypeScript 接口和类型守卫
- 位置：`src/types.ts`
- 包含：所有 IR 类型、OpenAPI 类型重导出、解析器选项、接口
- 依赖：`openapi-types`
- 使用者：所有其他层

## 数据流

**OpenAPI 到 Skill 转换：**

1. **输入加载** (`spec-loader.ts`)
   - CLI 接收文件路径或 URL
   - `loadSpecFromInput()` 检测源类型（HTTP 与文件）
   - 根据扩展名/内容解析为 YAML 或 JSON
   - 返回：`OpenAPISpec` 对象

2. **解析** (`parser.ts`)
   - `Parser.parse()` 接收规范和选项
   - 提取元数据（标题、版本、服务器、联系人、许可证）
   - 按标签、路径或自动策略分组操作
   - 将模式解析为前缀分组
   - 提取认证方案
   - 返回：`SkillDocument` IR

3. **渲染** (`renderer.ts`)
   - `TemplateRenderer` 接收 IR 组件
   - 为每个组件选择模板（自定义 → 默认回退）
   - Eta 模板将数据转换为 Markdown
   - 模板：`skill.md.eta`、`resource.md.eta`、`operation.md.eta`、`schema.md.eta`、`schema-index.md.eta`、`authentication.md.eta`
   - 返回：Markdown 字符串

4. **写入** (`converter.ts` 编排，`writer.ts` 执行)
   - 创建目录结构：`{output}/{skillName}/references/{resources,operations,schemas}`
   - 写入 `SKILL.md`（根索引）
   - 将资源文件写入 `resources/`
   - 将操作文件写入 `operations/`
   - 将模式文件写入 `schemas/{prefix}/`
   - 如果存在方案，则写入认证文档

**状态管理：**
- 转换之间没有持久状态
- 所有状态通过函数参数传递
- IR（`SkillDocument`）创建后不可变

## 关键抽象

**中间表示（IR）：**
- 用途：将 OpenAPI 解析与输出生成解耦
- 核心：`SkillDocument` 接口（`src/types.ts` 第 20-25 行）
- 包含：元数据、资源（分组操作）、模式分组、认证方案
- 好处：支持多种输出格式、更容易测试、潜在的转换缓存

**渲染器接口：**
- 用途：抽象模板引擎以提高可测试性并支持自定义实现
- 位置：`src/types.ts` 第 174-181 行
- 方法：`renderSkill()`、`renderResource()`、`renderOperation()`、`renderSchema()`、`renderSchemaIndex()`、`renderAuthentication()`
- 默认实现：使用 Eta 的 `TemplateRenderer`

**写入器接口：**
- 用途：抽象文件系统以提高可测试性
- 位置：`src/types.ts` 第 187-190 行
- 方法：`writeFile()`、`mkdir()`
- 默认实现：`FileSystemWriter`

**解析器策略模式：**
- 用途：可配置的操作分组
- 位置：`src/parser.ts` 第 155-175 行
- 策略：
  - `"tags"`：按 OpenAPI 标签分组（回退到"default"）
  - `"path"`：按第一个路径段分组
  - `"auto"`：如果有标签则使用标签，否则使用路径（默认）

## 入口点

**CLI 入口点：**
- 位置：`src/cli.ts`
- 二进制文件：`dist/src/cli.js`（在 `package.json` bin 中定义）
- 触发：通过 `openapi-to-skills` 命令直接执行
- 职责：
  - 参数解析和验证
  - 规范加载（通过 `loadSpecFromInput`）
  - 基本规范验证（openapi 字段、info.title、paths）
  - 输出目录冲突检测
  - 通过 consola 进行进度日志记录
  - 委托给 `convertOpenAPIToSkill()`

**程序化 API 入口点：**
- 位置：`index.ts`
- 导出：`convertOpenAPIToSkill`、`createParser`、`createRenderer`、`createWriter`、`Parser`、`TemplateRenderer`、`FileSystemWriter`、所有类型
- 触发：从消费代码 `import`
- 职责：隐藏内部结构的干净公共接口

**规范加载器入口点：**
- 位置：`src/spec-loader.ts`
- 函数：`loadSpecFromInput(input: string): Promise<OpenAPISpec>`
- 触发：由 CLI 调用
- 职责：
  - HTTP 与文件检测
  - 格式检测（YAML 与 JSON）
  - 使用适当的解析器进行解析

## 错误处理

**策略：** 在 CLI 层快速失败并提供描述性消息，在库级别传播错误

**模式：**
- CLI 在转换前验证规范结构（`cli.ts` 第 89-101 行）
- 解析器在无效引用时抛出（通过类型守卫优雅降级）
- 如果模板目录缺失，渲染器抛出
- 文件操作使用 async/await 和标准 Node.js 错误行为
- HTTP 错误包含状态码和状态文本

## 横切关注点

**日志：**
- 框架：consola
- 用途：进度指示、成功消息、结构化输出摘要
- 级别：错误（1）、信息（默认）、详细（通过 `--quiet` 标志）

**验证：**
- 方法：边界处的运行时检查（CLI 参数、规范结构）
- 类型安全：严格模式的完整 TypeScript 覆盖
- 细化：OpenAPI 联合类型的类型守卫（`isReferenceObject`）

**文件命名：**
- 工具：`src/renderer.ts` 第 126-134 行的 `toFileName()`
- 用途：将资源/标签名称清理为文件系统安全名称
- 算法：NFC 规范化、非字母数字字符转为连字符、合并多个连字符、修剪边缘

---

*架构分析：2026-03-02*
