# 编码规范

**分析日期：** 2026-03-02

## 命名模式

**文件：**
- 源文件使用短横线命名：`spec-loader.ts`、`parser.test.ts`
- 测试文件与源代码同位置：`parser.ts` → `parser.test.ts`
- 模板文件使用描述性名称加后缀：`skill.md.eta`、`schema-index.md.eta`

**函数：**
- 所有函数使用 camelCase：`createParser()`、`toFileName()`、`loadSpecFromInput()`
- 构造函数使用工厂模式：`createParser()`、`createRenderer()`、`createWriter()`
- 私有方法使用常规 camelCase（无前缀下划线）

**变量：**
- 变量和参数使用 camelCase
- 描述性名称优于缩写：`operation`、`spec`、`options`
- 布尔标志使用描述性名称：`excludeDeprecated`、`isPathExcluded`

**类型：**
- 接口和类型使用 PascalCase：`SkillDocument`、`ParserOptions`
- 后缀模式：
  - `*Document` 用于 IR 类型：`OperationDocument`、`ResourceDocument`
  - `*Object` 用于 OpenAPI 类型：`OperationObject`、`SchemaObject`
  - `*Options` 用于配置：`ConvertOptions`、`ParserOptions`

**类：**
- 类名使用 PascalCase：`Parser`、`TemplateRenderer`、`FileSystemWriter`

## 代码风格

**格式化：**
- 工具：Biome（v2.3.12）
- 缩进：制表符（非空格）
- 引号：字符串使用双引号
- 行宽：默认（暗示 80 字符）
- 尾随逗号：不强制

**关键 Biome 设置：**
```json
{
  "formatter": {
    "indentStyle": "tab"
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double"
    }
  }
}
```

**代码检查：**
- Biome 代码检查器启用推荐规则
- 通过辅助操作启用导入组织
- VCS 集成支持 git 忽略文件

## 导入组织

**顺序：**
1. 外部依赖（例如 `citty`、`consola`、`eta`）
2. Node.js 内置模块（例如 `node:fs`、`node:path`）
3. 内部类型（例如 `./types.js`）
4. 内部模块（例如 `./parser.js`）

**路径别名：**
- 不使用路径别名；所有导入使用带 `.js` 扩展名的相对路径
- 即使对于 TypeScript 文件也始终包含 `.js` 扩展名（ESM 要求）

**示例：**
```typescript
import { defineCommand } from "citty";
import { existsSync } from "node:fs";
import type { OpenAPISpec } from "./types.js";
import { createParser } from "./parser.js";
```

## 错误处理

**模式：**
- 使用 `consola` 进行 CLI 日志记录和错误报告
- CLI 错误时进程以代码 1 退出：`process.exit(1)`
- 异步错误冒泡到 CLI 处理程序
- 验证错误提供描述性消息

**示例：**
```typescript
if (!spec.openapi) {
  consola.error('Invalid OpenAPI spec: missing "openapi" field');
  process.exit(1);
}
```

**类型守卫：**
- 使用类型守卫进行运行时检查：`isReferenceObject()`
- 带类型谓词的过滤器函数：`.filter((p): p is ParameterObject => !isReferenceObject(p))`

## 日志

**框架：** `consola` v3.4.2

**模式：**
- `consola.start()` 用于操作开始
- `consola.info()` 用于信息性消息
- `consola.success()` 用于成功完成
- `consola.error()` 用于错误
- `consola.box()` 用于最终摘要

**日志级别：**
- 安静模式将级别设置为 1（仅错误）：`consola.level = 1`

## 注释

**何时注释：**
- 公共函数和复杂逻辑的 JSDoc
- 长文件的分隔符
- 非明显行为的实现说明

**JSDoc 模式：**
```typescript
/**
 * Convert an OpenAPI spec to Agent Skills format
 */
export async function convertOpenAPIToSkill(
  spec: OpenAPISpec,
  options: ConvertOptions,
): Promise<void> {
```

**分隔符：**
```typescript
// =============================================================================
// Parser Class
// =============================================================================
```

## 函数设计

**大小：**
- 函数通常小而集中（少于 50 行）
- 像 `Parser` 这样的大类将逻辑拆分为私有方法

**参数：**
- 对多个参数使用选项对象
- 适当时在函数签名中解构

**示例：**
```typescript
export interface ParserOptions {
  skillName?: string;
  filter?: ParserFilter;
  groupBy?: GroupByStrategy;
}

parse(spec: OpenAPISpec, options: ParserOptions = {}): SkillDocument
```

**返回值：**
- 返回类型化对象，避免 `any`
- 使用 TypeScript 的严格模式

## 模块设计

**导出：**
- 首选命名导出而非默认导出
- 工厂函数与类一起导出
- 仅类型导出使用 `type` 关键字

**示例：**
```typescript
export class Parser { ... }
export function createParser(): Parser { ... }
export type { OpenAPISpec } from "./types.js";
```

**桶文件：**
- 不使用桶文件；直接从源文件导入
- 类型集中在 `types.ts`

## TypeScript 配置

**严格设置：**
- `strict: true` - 启用所有严格模式检查
- `noUncheckedIndexedAccess: true` - 防止未定义访问
- `noImplicitOverride: true` - 需要 override 关键字
- `noFallthroughCasesInSwitch: true` - 穷举 switch case

**模块系统：**
- `module: "NodeNext"` - ES 模块与 Node.js 解析
- `moduleResolution: "NodeNext"`
- `target: "ES2022"`

---

*规范分析：2026-03-02*
