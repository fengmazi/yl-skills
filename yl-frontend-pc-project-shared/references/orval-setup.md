# Swagger → TypeScript 类型自动生成（orval 方案）

> 从后端 Swagger 2.0 / OpenAPI 3 文档自动生成 TS 类型定义，替代手动维护 `docs/` 目录的旧协作方式。

---

## 方案选型

| 方案 | 状态 | 原因 |
|------|------|------|
| openapi-typescript | ❌ | 只生成 types.d.ts（纯类型），无请求函数；Swagger 2.0 需先手动转 OpenAPI 3，且 redoc lint 对中文 DTO 名报错 |
| orval 7 | ❌ | 不支持 Node 22，有 ajv 编译错误 |
| orval 8 | ✅ | 内置 Swagger 2.0 → OpenAPI 3.1 自动升级，支持自定义 axios mutator，可按 tag 拆分输出 |

要求：**Node.js >= 22.18**（orval 8 硬性要求）。

---

## 安装与脚本

### 添加依赖

```bash
pnpm add -D orval@latest
```

### package.json 脚本

```json
{
  "scripts": {
    "gen:api": "pnpm run gen:api:test",
    "gen:api:test": "node scripts/fetch-swagger.mjs <测试环境URL> swagger/input.json && orval --config ./orval.config.ts",
    "gen:api:local": "node scripts/fetch-swagger.mjs <本地环境URL> swagger/input.json && orval --config ./orval.config.ts",
    "gen:api:check": "orval --config ./orval.config.ts && pnpm run typecheck"
  }
}
```

### scripts/fetch-swagger.mjs

```js
import { writeFileSync, mkdirSync } from 'node:fs';
import { dirname } from 'node:path';

const url = process.argv[2];
const outputPath = process.argv[3];

const res = await fetch(url);
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const text = await res.text();
mkdirSync(dirname(outputPath), { recursive: true });
writeFileSync(outputPath, text, 'utf-8');
console.log(`Saved: ${outputPath} (${text.length} bytes)`);
```

### orval.config.ts

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  api: {
    input: {
      target: './swagger/input.json',
      // 跳过 @scalar/openapi-parser 校验：中文 schema 名 / originalRef / type:"ref"
      // 等非标字段会导致校验失败，但 orval 内部能正确处理
      unsafeDisableValidation: true,
      override: {
        transformer: './scripts/transformer.mjs',
      },
    },
    output: {
      target: './src/api/generated/api.ts',
      client: 'axios',
      mode: 'tags-split',
      clean: true,
      override: {
        mutator: {
          path: './src/api/client.ts',
          name: 'customInstance',
        },
      },
    },
  },
});
```

### .gitignore

```
src/api/generated/
swagger/
```

---

## Transformer：预处理 Swagger 2.0 的非标字段

orval 内部升级 Swagger 2.0 → OpenAPI 3.1 时，两个坑需要 transformer 预处理：

### 坑 1：`/` 在 schema 名中（JSON Pointer 路径分隔符冲突）

后端 DTO 名如 `内部用车业务附件/审批流水附件`，orval 内部 ref 解析时 `/` 被当作 JSON Pointer 路径分隔符导致找不到 schema。

**修法**：把 `/` 替换为 `__slash__`，同步更新所有 `$ref`。

### 坑 2：`List`/`Map` 裸 ref（Spring `List<String>` 残留）

body 参数中出现 `"$ref": "#/definitions/List"`，但 definitions 里没有 `List`。

**修法**：直接 inline 为 `{ type: 'array', items: { type: 'string' } }`。

### scripts/transformer.mjs 完整代码

```js
import { defineTransformer } from 'orval';

function fixSlashInSchemaNames(spec) {
  const nameMap = {};

  for (const name of Object.keys(spec.definitions || {})) {
    if (name.includes('/')) {
      nameMap[name] = name.split('/').join('__slash__');
    }
  }

  if (Object.keys(nameMap).length === 0) return spec;

  const newDefs = {};
  for (const [name, schema] of Object.entries(spec.definitions || {})) {
    newDefs[nameMap[name] || name] = schema;
  }
  spec.definitions = newDefs;

  function fixRefs(obj) {
    if (!obj || typeof obj !== 'object') return;
    if (Array.isArray(obj)) {
      for (const item of obj) fixRefs(item);
      return;
    }
    for (const key of Object.keys(obj)) {
      if (key === '$ref' && typeof obj[key] === 'string') {
        const match = obj[key].match(/^#\/definitions\/(.+)$/);
        if (match && nameMap[match[1]]) {
          obj[key] = `#/definitions/${nameMap[match[1]]}`;
        }
        if (match) {
          const refName = match[1];
          if (refName === 'List' || refName.startsWith('List«')) {
            delete obj.$ref;
            obj.type = 'array';
            obj.items = { type: 'string' };
          } else if (refName === 'Map' || refName.startsWith('Map«')) {
            delete obj.$ref;
            obj.type = 'object';
            obj.additionalProperties = { type: 'string' };
          }
        }
        continue;
      }
      fixRefs(obj[key]);
    }
  }
  fixRefs(spec);

  // formData 参数中 type:"ref" → type:"file"
  function fixFormDataRefs(obj) {
    if (!obj || typeof obj !== 'object') return;
    if (Array.isArray(obj)) {
      for (const item of obj) fixFormDataRefs(item);
      return;
    }
    if (obj.parameters && Array.isArray(obj.parameters)) {
      for (const param of obj.parameters) {
        if (param.in === 'formData' && (param.type === 'ref' || param.type === 'file')) {
          param.type = 'file';
        }
      }
    }
    for (const val of Object.values(obj)) {
      fixFormDataRefs(val);
    }
  }
  fixFormDataRefs(spec);

  return spec;
}

export default defineTransformer(fixSlashInSchemaNames);
```

---

## 自定义 Mutator（对接项目 axios 拦截器）

```ts
// src/api/client.ts
import http from '@/plugins/axios';
import type { AxiosRequestConfig, AxiosResponse } from 'axios';

/**
 * orval 自定义 mutator
 * http 拦截器返回 { code, message, data } 信封，这里解包取出业务 data
 */
export const customInstance = <T>(config: AxiosRequestConfig): Promise<AxiosResponse<T>> => {
  return http(config).then((envelope: any) => {
    return {
      data: envelope.data as T,
      status: 200,
      statusText: 'OK',
      headers: {},
      config: config as any,
    } as AxiosResponse<T>;
  });
};

export default customInstance;
```

---

## 多模块场景：Select a definition

部分后端项目的 Swagger 按模块拆分为多个 definition（如 Swagger UI 页面 top-right 下拉 "Select a definition" → `系统管理` / `预算管理`）。

每个 definition 对应一个独立的 JSON 端点：

```
https://xxx.com/project/v3/api-docs/系统管理
https://xxx.com/project/v3/api-docs/预算管理
```

### orval 多模块配置

```ts
import { defineConfig } from 'orval';

export default defineConfig({
  system: {
    input: {
      target: 'https://xxx.com/project/v3/api-docs/系统管理',
      // OpenAPI v3 规范文件通常不需要 unsafeDisableValidation
    },
    output: {
      target: './src/api/generated/system.ts',
      client: 'axios',
      clean: true,
      override: {
        mutator: {
          path: './src/api/client.ts',
          name: 'customInstance',
        },
      },
    },
  },
  budget: {
    input: {
      target: 'https://xxx.com/project/v3/api-docs/预算管理',
    },
    output: {
      target: './src/api/generated/budget.ts',
      client: 'axios',
      clean: true,
      override: {
        mutator: {
          path: './src/api/client.ts',
          name: 'customInstance',
        },
      },
    },
  },
});
```

**注意**：orval 的 `target` 数组是回退逻辑（试第一个，不行试第二个），不是合并。多模块必须分别定义为多组配置。

---

## 业务代码使用方式

项目继续用已有的 `http.post()` 封装，类型从 `api.schemas.ts` 导入：

```ts
import http from '@/plugins/axios';
import type { AcceptanceOrderDTO } from '@/api/generated/api.schemas';

const res = await http.post<AcceptanceOrderDTO>('/acceptanceOrder/query', payload);
```

不做 orval 生成的请求函数（`getXxx().xxx()`），只在类型层面接入生成能力。

---

## CI 集成

```json
{
  "scripts": {
    "ci:check": "pnpm gen:api:test && pnpm typecheck"
  }
}
```

在 CI 阶段拉最新 swagger → 生成 → typecheck，后端接口变更时 CI 报类型错误，提前发现不兼容。
