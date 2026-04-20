# AGENTS.md

> 适用范围：本仓库（Vue 3 + Vite + TypeScript 管理后台模板）中的所有代码改动与协作任务。

## 1. 项目速览（先建立全局认知）

- 技术栈：Vue 3.5、Vite 7、TypeScript（strict）、Vue Router、Pinia、Element Plus、UnoCSS、Vitest、ESLint。
- 包管理：`pnpm`。
- 别名约定：
  - `@/*` → `src/*`
  - `@@/*` → `src/common/*`
- 开发端口：`3333`（见 `vite.config.ts`）。
- 环境变量核心项：
  - `VITE_APP_TITLE`
  - `VITE_BASE_URL`
  - `VITE_PUBLIC_PATH`
  - `VITE_ROUTER_HISTORY`

## 2. 目录与职责（按“业务就近”放置代码）

```txt
src/
  main.ts                  # 应用入口：createApp -> installPlugins -> pinia/router -> mount
  App.vue                  # 根组件：主题、灰度/色弱、通知初始化
  router/                  # 常驻路由 + 动态路由 + 导航守卫 + 白名单
  pinia/stores/            # app/user/permission/settings/tags-view
  http/axios.ts            # request 封装与统一拦截
  layouts/                 # 三种布局模式 + 导航相关组件
  pages/                   # 页面模块（dashboard/login/demo/error/...）
  common/
    apis/                  # 通用接口（users/tables）
    components/            # 通用组件（Notify/SearchMenu/ThemeSwitch/...）
    composables/           # 通用 hooks
    constants/             # 常量与枚举
    utils/                 # 工具函数（local-storage/permission/validate/...）
tests/                     # Vitest 用例
types/                     # 全局类型与自动生成类型声明
```

放置规则：

- 通用能力放 `src/common/*`。
- 页面私有逻辑放 `src/pages/<module>/*`（如 login 的 apis/components/composables/images）。
- 不要把页面私有 API 放到 `common/apis`。

## 3. 关键运行链路（改动前必须理解）

1. 启动链路：`src/main.ts` 挂载插件、Pinia、Router；`router.isReady()` 后再 `mount`。
2. 路由权限链路：
   - 路由定义：`constantRoutes` + `dynamicRoutes`（`src/router/index.ts`）
   - 守卫：`src/router/guard.ts`
   - 权限过滤：`src/pinia/stores/permission.ts`
   - 用户角色：`src/pinia/stores/user.ts`
3. 请求链路：`src/http/axios.ts`
   - 约定 `ApiResponseData<T>`（`types/api.d.ts`）
   - `code === 0` 视为成功
   - `401` 触发登出逻辑
4. 布局/设置链路：
   - `src/layouts/config.ts` 定义布局配置
   - `src/pinia/stores/settings.ts` 根据配置动态建 store，并持久化到 localStorage

## 4. 代码风格与约束（遵循项目现状，不自创体系）

- Vue 组件默认使用 `<script setup lang="ts">`。
- Store 使用 Setup Store 风格（本仓库统一如此）。
- TypeScript：
  - 开启 strict，避免 `any`
  - 对象优先 `interface`，联合/映射可用 `type`
- 样式：
  - 默认 `scss`，组件内优先 `scoped`
  - 已接入 UnoCSS，可复用 `uno.config.ts` 的快捷类
- 命名遵循：
  - 组件：PascalCase（`index.vue` 例外）
  - 页面/TS 文件：kebab-case
  - 组合式函数：`useXxx`

## 5. 任务执行准则（本仓库专用）

1. **先定位，再动手**：先确认改动位于 page-private 还是 common 层。
2. **外科手术式修改**：只改与需求直接相关的文件，不顺手重构。
3. **保持链路一致性**：涉及路由、权限、缓存、主题、布局时，要同步检查上下游。
4. **避免重复造轮子**：优先复用已有 composables、utils、components、stores。
5. **补齐类型**：新增接口时同步更新对应 `type.ts` / `types/*.d.ts`。

## 6. 常见改动操作指引

- 新增页面：
  - 页面文件放 `src/pages/<module>/index.vue`
  - 在 `src/router/index.ts` 配置路由
  - 若需缓存，`route.meta.keepAlive = true` 且组件 `name` 与路由 `name` 保持一致
- 新增接口：
  - 在对应模块 `apis/index.ts` + `apis/type.ts` 定义
  - 统一走 `request`，不要直接创建新的 axios 实例
- 新增全局设置项：
  - 先加到 `src/layouts/config.ts` 的 `LayoutsConfig` 与默认值
  - `settings` store 会自动纳入并持久化
- 新增权限点：
  - 路由级：`meta.roles`
  - 按钮级：`v-permission` 或 `checkPermission`

## 7. 开发与验证命令

- 安装依赖：`pnpm install`
- 本地开发：`pnpm dev`
- 代码检查（自动修复）：`pnpm lint`
- 构建：`pnpm build`
- 测试（单次运行）：`pnpm test --run`

说明：

- `pnpm test` 会进入监听模式；CI/自动化请使用 `pnpm test --run`。
- `pre-commit`（`.husky/pre-commit`）会执行 `vue-tsc` 与 `lint-staged`。

## 8. Agent 协作文件

- 现有规则文件位于 `.claude/rules/*.mdc`（包含 project/vue/ts/git 约束）。
- 若你所在分支仍使用 `.agents/rules`，按同一规则语义执行即可。

## 9. 最低成功标准

在交付前应满足：

1. 改动与需求一一对应，可解释每一处 diff 的必要性。
2. 类型、构建、测试命令可通过（按任务范围执行最小充分验证）。
3. 不引入与本任务无关的结构性重构或行为变化。
