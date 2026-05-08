## v0.0.1 (2026-05-07 10:30)

### perf: 优化 Docker 构建阶段依赖安装可观测性

- 在 `openscg_app/Dockerfile` 中增加 `PUPPETEER_SKIP_DOWNLOAD`，避免镜像构建时自动下载 Puppeteer 浏览器二进制
- 将构建阶段和运行阶段的 `npm install` 调整为 `--verbose`，便于在 Docker 构建日志中观察依赖下载进度

## v0.0.2 (2026-05-07 16:53)

### perf: 将 Docker 构建依赖安装切换为 pnpm

- 在 `openscg_app/Dockerfile` 中启用 `corepack`，将构建阶段和运行阶段的依赖安装从 `npm` 切换为 `pnpm`
- 新增 `openscg_app/pnpm-lock.yaml`，让镜像构建基于 lockfile 执行确定性安装
- 保留 `PUPPETEER_SKIP_DOWNLOAD` 与 `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD`，减少 Puppeteer 对 Docker 构建耗时的影响
- 移除 `src/app/layout.tsx` 中对 `next/font/google` 的 `Geist` 在线字体依赖，避免 Docker 构建阶段因外网字体请求失败而中断

## v0.0.3 (2026-05-08 14:20)

### fix: 迁移 pnpm 11 的 Docker 构建白名单配置

- 在 `openscg_app/package.json` 中新增 `packageManager` 固定 `pnpm@11.0.8`，并移除已废弃的 `pnpm.onlyBuiltDependencies`
- 新增 `openscg_app/pnpm-workspace.yaml`，通过 `allowBuilds` 显式放行 `esbuild`、`puppeteer`、`sharp`、`unrs-resolver`
- 更新 `openscg_app/Dockerfile`，在镜像安装依赖前复制 `pnpm-workspace.yaml`，并在构建命令中关闭运行前依赖自检，避免再次触发 ignored builds 校验
