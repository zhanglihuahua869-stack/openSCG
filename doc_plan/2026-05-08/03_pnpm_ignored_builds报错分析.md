## 📊 分析报告

### 1. 问题描述
在使用 `docker-compose up -d --build` 构建镜像时，在执行到 `RUN pnpm install --frozen-lockfile --reporter=append-only` 步骤（第 5 步）时失败，导致构建中断。报错核心信息为：
```
[ERR_PNPM_IGNORED_BUILDS] Ignored build scripts: esbuild@0.21.5, puppeteer@24.43.0, sharp@0.34.5, unrs-resolver@1.11.1
Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts.
```

### 2. 原因分析
这是 `pnpm` 新版本的安全机制导致的。在较新的 pnpm 版本（特别是本项目中升级到的 v11 版本）中，默认会**阻止**所有依赖包在安装时执行其自定义的构建脚本（即 `postinstall` 等生命周期脚本），除非这些包在 `package.json` 的 `pnpm.onlyBuiltDependencies` 字段中被显式批准。

在我们的项目中，`esbuild`、`puppeteer` 和 `sharp` 这几个包在安装后都必须执行自身的脚本来下载相应的二进制文件或进行原生编译。因为它们被 pnpm 拦截了执行权限，所以导致安装过程直接抛出 `[ERR_PNPM_IGNORED_BUILDS]` 致命错误。

### 3. 方案设计

#### 推荐方案：配置 .npmrc 自动批准或在 package.json 中白名单放行
由于这是 Docker 构建环境，最稳妥且侵入性最小的方式是直接在项目中配置 `package.json` 的 `pnpm` 字段，将这些必须执行脚本的包加入白名单。

**具体改动**：
在 `openscg_app/package.json` 的根级添加如下配置：
```json
"pnpm": {
  "onlyBuiltDependencies": [
    "esbuild",
    "puppeteer",
    "sharp",
    "unrs-resolver"
  ]
}
```

**优点**：
- **符合 pnpm 最佳实践**：遵循了 pnpm 最新的安全白名单机制。
- **构建透明**：无需修改 Dockerfile 中的命令逻辑。

**影响范围**：
- **涉及文件**：`openscg_app/package.json`
- **潜在风险**：无明显风险，仅恢复了这些包原本应有的安装后执行权限。

### 4. 实施计划
1. 读取 `openscg_app/package.json`。
2. 找到 `package.json` 底部，在最后一个大括号前插入 `"pnpm": { "onlyBuiltDependencies": [...] }` 字段。

### 5. 风险评估
| 风险项 | 影响 | 缓解措施 |
|--------|------|---------|
| package.json 语法错误 | 导致后续 npm/pnpm 命令完全失效 | 使用 AST 解析或确保精确匹配插入，保持 JSON 格式完全合法。 |

---

## 执行进度审查

- [x] 方案分析与比对完成
- [x] 征求用户确认执行方案 (当前状态：**已授权**)
- [x] 代码修改与提交
- [x] 验证修改生效
