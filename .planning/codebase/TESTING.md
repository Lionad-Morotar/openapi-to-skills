# 测试模式

**分析日期：** 2026-03-02

## 测试框架

**运行器：**
- Bun 的内置测试运行器（bun:test）
- 配置：无单独配置文件，使用默认值
- 版本：与 Bun 运行时捆绑

**断言库：**
- 通过 `expect` 使用 Bun 的内置断言
- Jest 兼容 API

**运行命令：**
```bash
bun test                 # 运行所有测试（单元 + 端到端）
bun test src/            # 仅运行单元测试
bun test e2e/            # 仅运行端到端快照测试
bun run snapshot:update  # 重新生成端到端快照
```

## 测试文件组织

**位置：**
- 单元测试与源代码同位置：`src/parser.ts` → `src/parser.test.ts`
- 端到端测试在专用目录：`e2e/snapshot.test.ts`

**命名：**
- 单元测试：`[name].test.ts`
- 端到端测试：`e2e/` 目录中的 `[name].test.ts`

**结构：**
```
/Users/lionad/Github/Source/openapi-to-skills/
├── src/
│   ├── parser.ts
│   ├── parser.test.ts       # 单元测试
│   ├── converter.ts
│   ├── converter.test.ts    # 单元测试
│   └── ...
└── e2e/
    └── snapshot.test.ts     # 端到端测试
```

## 测试结构

**套件组织：**
```typescript
import { describe, expect, test } from "bun:test";

describe("Parser.parse - meta", () => {
  const parser = new Parser();

  test("extracts basic meta info", () => {
    const spec = createMinimalSpec({
      info: {
        title: "My API",
        version: "2.0.0",
      },
    });

    const doc = parser.parse(spec);

    expect(doc.meta.title).toBe("My API");
    expect(doc.meta.version).toBe("2.0.0");
  });
});
```

**模式：**
- 使用 `describe` 块分组相关测试
- 使用描述性测试名称："extracts basic meta info"
- 每个套件或测试创建新的解析器实例
- 使用工厂函数创建测试夹具

**设置/清理：**
```typescript
import { beforeAll, afterAll } from "bun:test";

describe("TemplateRenderer template fallback", () => {
  const testDir = join(import.meta.dir, "../.test-templates");

  beforeAll(() => {
    mkdirSync(testDir, { recursive: true });
  });

  afterAll(() => {
    rmSync(testDir, { recursive: true, force: true });
  });
});
```

## 模拟

**框架：** Bun 内置的 `mock()` 函数

**模式：**
```typescript
import { mock } from "bun:test";

function createMockWriter() {
  const mkdirCalls: string[] = [];
  const writeFileCalls: Array<{ path: string; content: string }> = [];

  const writer: Writer = {
    mkdir: mock((path: string) => {
      mkdirCalls.push(path);
      return Promise.resolve();
    }),
    writeFile: mock((path: string, content: string) => {
      writeFileCalls.push({ path, content });
      return Promise.resolve();
    }),
  };

  return { writer, mkdirCalls, writeFileCalls };
}
```

**全局模拟：**
```typescript
const originalFetch = globalThis.fetch;
globalThis.fetch = mock(async () => {
  return new Response("...", { status: 200 });
}) as typeof fetch;

try {
  // 测试代码
} finally {
  globalThis.fetch = originalFetch;
}
```

**模拟内容：**
- 文件系统操作（Writer 接口）
- 网络请求（fetch）
- 外部依赖（Renderer 接口）

**不模拟：**
- 被测试的内部逻辑
- 纯函数
- 类型守卫

## 夹具和工厂

**测试数据：**
```typescript
function createMinimalSpec(overrides: Partial<OpenAPISpec> = {}): OpenAPISpec {
  return {
    openapi: "3.0.0",
    info: {
      title: "Test API",
      version: "1.0.0",
    },
    paths: {},
    ...overrides,
  };
}
```

**模式：**
- 支持覆盖的工厂函数
- 默认最小有效数据
- 使用展开运算符进行覆盖
- 使用适当的 TypeScript 类型确保类型安全

**位置：**
- 工厂在测试文件顶部定义
- 不跨文件共享（每个测试文件有自己的）

## 覆盖率

**要求：** 未强制执行（未找到覆盖率配置）

**查看覆盖率：**
```bash
bun test --coverage  # 如果 Bun 支持
```

## 测试类型

**单元测试：**
- 范围：隔离的单个函数和类
- 位置：`src/*.test.ts`
- 方法：模拟依赖项，测试纯逻辑

**示例：**
```typescript
describe("Parser.parse - meta", () => {
  const parser = new Parser();

  test("extracts basic meta info", () => {
    // 测试实现
  });
});
```

**集成测试：**
- 范围：组件交互
- 位置：`src/converter.test.ts`
- 方法：模拟 Writer 和 Renderer，测试 Parser + Converter 集成

**端到端测试：**
- 范围：完整的 CLI 执行
- 位置：`e2e/snapshot.test.ts`
- 方法：针对真实规范运行 CLI，将输出与快照比较

**快照测试：**
```typescript
describe("e2e snapshot tests", async () => {
  const specs = await getInputSpecs();

  for (const spec of specs) {
    describe(spec, async () => {
      test("has same files", () => {
        const actualFiles = [...actual.keys()].sort();
        expect(actualFiles).toEqual(expectedFiles);
      });

      test(`file: ${path}`, () => {
        expect(actualContent).toBe(expectedContent);
      });
    });
  }
});
```

## 常见模式

**异步测试：**
```typescript
test("creates skill directory", async () => {
  await convertOpenAPIToSkill(spec, options);
  expect(mkdirCalls).toContain("/out/test-api");
});
```

**错误测试：**
```typescript
test("throws when custom template dir not found", () => {
  expect(() => {
    new TemplateRenderer("/nonexistent/path");
  }).toThrow("Custom templates directory not found");
});
```

**参数化测试：**
```typescript
for (const spec of specs) {
  describe(spec, async () => {
    // 每个规范文件的测试
  });
}
```

**动态测试生成：**
```typescript
const files = await readDirRecursive(expectedDir);
for (const [path] of files) {
  test(`file: ${path}`, () => {
    const expectedContent = expected.get(path);
    const actualContent = actual.get(path);
    expect(actualContent).toBe(expectedContent);
  });
}
```

## 测试工具

**辅助函数：**
- `createMinimalSpec()` - 创建最小有效 OpenAPI 规范
- `createMockWriter()` - 创建带跟踪的模拟 Writer
- `createMockRenderer()` - 创建模拟 Renderer
- `runCLI()` - 在子进程中执行 CLI

**文件系统测试：**
- 使用 `mkdtemp` 创建临时目录
- 在 `afterAll` 中清理
- 使用 `Bun.Glob` 扫描文件（仅测试文件）

---

*测试分析：2026-03-02*
