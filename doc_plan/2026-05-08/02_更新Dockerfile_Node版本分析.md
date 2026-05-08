## 📊 分析报告

### 1. 问题描述
当前项目中，由于 `pnpm` 升级到了版本 11，该版本强制要求 Node.js 版本 >= 22。而项目中现有的 `openscg_app/Dockerfile` 依然采用 `node:20-alpine` 作为基础镜像，导致在执行 `corepack enable` 和 `pnpm install` 时会触发引擎版本不兼容的致命错误，阻断容器构建流程。

### 2. 原因分析
- **pnpm 11 引擎要求变更**：社区生态持续推进，工具链对 Node 运行时的最低要求提升至了最新的 LTS 版本线。
- **构建环境滞后**：Dockerfile 锁定的基础环境未能跟随外围依赖版本同步演进。

### 3. 方案设计

#### 推荐方案：升级 Dockerfile 基础镜像至 Node 22
将 `openscg_app/Dockerfile` 中 `builder` 与 `runner` 两个阶段的基础镜像统一升级至 `node:22-alpine`。

**优点**：
- **治本之策**：直接满足 `pnpm 11` 的引擎硬性限制，彻底修复构建阻断问题。
- **性能与安全双收**：享受 Node 22 V8 引擎带来的执行效率提升和更完善的安全防护机制。
- **框架契合度高**：Next.js 15 全面支持 Node 22，两者结合稳定可靠。
- **环境一致性**：同时升级构建和运行环境，避免 ABI 版本差异导致的隐性 Bug。

**影响范围**：
- **涉及文件**：`openscg_app/Dockerfile`
- **潜在风险**：影响极小。仅在项目中使用了极个别尚未适配 Node 22 的陈旧 Native Addons（原生扩展包）时可能出现编译异常，但基于目前项目（Next.js 15）技术栈评估，此类概率微乎其微。

### 4. 实施计划
1. 修改 `openscg_app/Dockerfile` 第 2 行：`FROM node:20-alpine AS builder` 变更为 `FROM node:22-alpine AS builder`。
2. 修改 `openscg_app/Dockerfile` 第 21 行：`FROM node:20-alpine AS runner` 变更为 `FROM node:22-alpine AS runner`。

### 5. 风险评估
| 风险项 | 影响 | 缓解措施 |
|--------|------|---------|
| Native Addon 构建失败 | 镜像打包过程中断 | 重新评估并升级出问题的特定依赖包（概率极低）。 |

---

## 执行进度审查

- [x] 方案分析与比对完成
- [x] 征求用户确认执行方案 (当前状态：**已授权**)
- [x] 代码修改与提交
- [x] 验证修改生效
