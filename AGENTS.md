# Open-Slide - AI 代理幻灯片框架

> Forked from: https://github.com/1weiho/open-slide
> Fork URL: https://github.com/afanty2021/open-slide
> 更新时间: 2026-05-07

## 项目概览

**Open-Slide** 是专为 AI 编码代理（Claude Code、Codex、Cursor 等）设计的幻灯片运行时框架。用自然语言描述 PPT → AI 编写 React 代码 → open-slide 处理画布、缩放、导航和演示模式。

**核心技术**：
- **画布尺寸**：每张幻灯片固定 **1920 × 1080**
- **幻灯片格式**：任意 React 组件（无约束 DSL）
- **架构**：pnpm + Turbo Monorepo
- **语言**：TypeScript (94.9%), MDX (3.1%), CSS (1.8%)

---

## Monorepo 结构

| 路径 | 包名 | 职责 |
|------|------|------|
| `packages/core` | `@open-slide/core` | 运行时（查看器、演示模式、检查器）、Vite 插件、`open-slide` dev/build CLI |
| `packages/cli` | `@open-slide/cli` | `npx @open-slide/cli init` 脚手架 + 项目模板 |
| `apps/demo` | private | 本地测试应用，通过 `workspace:*` 消费 `@open-slide/core` |
| `apps/web` | private | 营销网站（Next.js） |

**共享配置**：`biome.json`、`turbo.json`、`pnpm-workspace.yaml`、`tsconfig`

---

## 核心特性

### 🤖 代理原生创作
- 兼容任何编码代理（Claude Code, Codex, Cursor）
- 内置代理技能：
  - **`/create-slide`** - 端到端创建幻灯片
  - **`/slide-authoring`** - 技术参考文档（画布、类型比例、调色板、布局规则）

### 🎯 浏览器内检查器
- 点击任意元素添加注释（如 *"把这个变红"*、*"缩小标题"*）
- 注释持久化为 `@slide-comment` 标记
- 运行 `/apply-comments` 让代理应用所有编辑

### 🖼️ 资源管理器 + svgl 集成
- 管理每套幻灯片的图片、视频、字体
- 集成 [svgl](https://svgl.app/) 目录搜索品牌 logo

### 🎬 专业演示模式
- 全屏播放 + 键盘导航
- 演讲者模式（当前/下一页预览、演讲者备注、计时器）

### 📦 导出 & 部署
- 一键导出为静态 HTML 或打印就绪的 PDF
- 静态构建，一键部署到 Vercel/Cloudflare Pages/Zeabur/Netlify

---

## 快速开始（用户）

```bash
# 初始化项目
npx @open-slide/cli init my-slide
cd my-slide

# 启动开发服务器
pnpm dev
```

---

## 开发工作流（框架开发）

```bash
# 安装依赖
pnpm install

# 开发模式（运行 demo 测试本地 core）
pnpm dev

# 构建所有包
pnpm build

# 类型检查
pnpm typecheck

# 代码检查（Biome）
pnpm check        # 检查格式 + lint + import 组织
pnpm check:fix    # 自动修复可修复的问题

# 测试
pnpm test
```

**按包运行脚本**：
```bash
pnpm core dev      # 只在 core 包运行 dev
pnpm cli build     # 只在 cli 包运行 build
```

---

## 开发循环

```mermaid
graph LR
    A[展示幻灯片] --> B[点击元素添加注释]
    B --> C[运行 /apply-comments]
    C --> D[代理应用编辑]
    D --> A
```

1. 展示幻灯片 → 点击元素添加注释
2. 运行 `/apply-comments` → 代理应用编辑
3. 重复迭代

---

## 硬性规则（Framework 开发）

### 提交前检查
- **Biome 必须通过**：运行 `pnpm check`（或 `pnpm check:fix`）
- CI 和用户审查都期望干净的代码树

### Changeset 管理
- **如果 `packages/core` 或 `packages/cli` 改变，添加 changeset**
  ```bash
  pnpm changeset
  ```
  - 选择正确的包
  - 选择 bump 类型：
    - `patch` - 修复/优化
    - `minor` - 新公共 API
    - `major` - 破坏性变更
- **Apps（`demo`、`web`）和根工具不需要 changeset**

### Changeset 描述规范
- **简短直接**：一行，现在时，从用户视角描述变化
- **匹配仓库中 `.changeset/*.md` 的语调**
- **禁止**：
  - ❌ 段落
  - ❌ 理由解释（"这个 PR 引入了……"）
  - ❌ "this PR" 引用

**示例**：
- ✅ Good: `Replace spinner with a hairline + sliding bar for slide and presenter loading states.`
- ❌ Bad: `This change introduces a new loading indicator because the previous spinner felt heavy and we wanted something more subtle for presentation contexts…`

### 版本管理
- **不要手动 bump 版本或编辑 `CHANGELOG.md**
- `changeset version` 负责版本发布

### 依赖管理
- **不要随意添加依赖**
- `core` 运行时会分发给用户，每个依赖都会增加安装大小

### UI 组件
- `packages/core/src/app/components/ui` 是 shadcn 生成的，biome 忽略
- 不要修改，除非重新生成

### 注释规范
- **默认不写注释**
- 仅在以下情况添加：
  - 隐藏约束
  - 微妙的不变量
  - 特定 bug 的变通方案
  - 可能让读者惊讶的行为
- **禁止**：
  - ❌ 解释代码做什么（命名良好的标识符已说明）
  - ❌ 引用任务/PR/调用者（"为 X 添加"、"被 Y 使用"）
  - ❌ 部分隔横幅（`// ── Section ──`）
  - ❌ 模块头部描述
  - ❌ 注释掉的代码
- **判断标准**：如果删除注释不会让读者困惑，就不要写

---

## 发布流程（参考）

```bash
pnpm release
```

- 构建 `core` + `cli`
- 运行 `changeset publish`
- 由维护者触发，不是由代理触发

---

## Git 远程配置

```bash
origin    git@github.com:afanty2021/open-slide.git (fetch/push)
upstream  git@github.com:1weiho/open-slide.git (fetch/push)
```

**同步上游**：
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

## 技术栈

- **包管理**: pnpm + workspace
- **构建**: Turbo + Vite
- **代码风格**: Biome（格式化 + lint + import 组织）
- **类型检查**: TypeScript (tsc)
- **测试**: Vitest
- **UI 组件**: shadcn/ui

---

## 关键文件

- `AGENTS.md` - 代理开发指南（框架仓库指南）
- `CLAUDE.md` - 项目文档（本文件）
- `apps/demo/.claude/skills/` - 幻灯片创作技能
  - `slide-authoring` - 技术参考
  - `create-slide` - 创建命令
- `apps/demo/slides/` - 示例幻灯片

---

*详细变更记录请查看上游仓库的 CHANGELOG.md*
