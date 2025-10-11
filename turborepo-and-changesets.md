# Turborepo + Changesets 完整实现流程（详细说明版）

## 1. 项目初始化

### 1.1 创建项目目录
```bash
# 创建项目根目录
mkdir my-turborepo
cd my-turborepo

# 初始化 package.json
npm init -y
```
**详细说明**：
- 创建一个新的项目根目录，这是整个 monorepo 的容器
- `npm init -y` 使用默认配置快速生成 package.json，后续会修改这个文件

### 1.2 安装核心依赖
```bash
# 安装 Turborepo - 负责构建任务调度和缓存
npm install -D turbo

# 安装 Changesets - 负责版本管理和发布
npm install -D @changesets/cli

# 安装 pnpm (推荐用于 monorepo)
npm install -g pnpm
```
**详细说明**：
- `turbo` 是 Turborepo 的核心包，提供任务编排和缓存功能
- `@changesets/cli` 是 Changesets 的命令行工具，用于管理版本和发布
- `pnpm` 是推荐的包管理器，对 monorepo 有更好的支持

### 1.3 配置 package.json 脚本
```json
{
  "name": "my-turborepo",
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev --parallel",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "clean": "turbo run clean",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "changeset publish"
  },
  "devDependencies": {
    "@changesets/cli": "^2.27.1",
    "turbo": "^1.10.12"
  },
  "packageManager": "pnpm@8.0.0"
}
```
**详细说明**：
- `private: true` 防止意外发布根目录
- `turbo run` 命令会按照 pipeline 配置执行任务
- `--parallel` 标志让开发命令并行运行
- Changesets 相关脚本用于版本管理和发布流程

## 2. 项目结构配置

### 2.1 创建 pnpm-workspace.yaml
```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```
**详细说明**：
- 这个文件告诉 pnpm 哪些目录包含包
- `packages/*` 包含可发布的库包
- `apps/*` 包含应用程序（如 Next.js、React 应用）
- pnpm 会将这些目录中的包链接在一起

### 2.2 创建目录结构
```bash
# 创建包目录结构
mkdir -p packages/ui packages/utils apps/web

# 为每个包创建基础结构
mkdir -p packages/ui/src packages/ui/dist
mkdir -p packages/utils/src packages/utils/dist  
mkdir -p apps/web/src apps/web/public
```
**详细说明**：
- `packages/` 目录存放可重用的库包
- `apps/` 目录存放具体的应用程序
- 每个包都有 `src/`（源代码）和 `dist/`（构建输出）目录

## 3. Turborepo 配置

### 3.1 创建 turbo.json
```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "build/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```
**详细说明**：
- `dependsOn: ["^build"]` 表示先构建依赖项（^ 符号）
- `outputs` 定义哪些文件应该被缓存
- `dev` 任务设置 `cache: false` 因为开发模式不需要缓存
- `persistent: true` 表示这是长期运行的任务
- 任务依赖关系确保正确的构建顺序

## 4. Changesets 配置

### 4.1 初始化 Changesets
```bash
# 初始化 Changesets 配置
pnpm changeset init
```
**详细说明**：
- 这个命令会创建 `.changeset` 目录和配置文件
- 生成默认的 Changesets 工作流程配置

### 4.2 配置 .changeset/config.json
```json
{
  "$schema": "https://unpkg.com/@changesets/config@2.3.1/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": true,
  "fixed": [],
  "linked": [],
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```
**详细说明**：
- `changelog`: 使用 Changesets 内置的 changelog 生成器
- `commit: true`: 自动提交版本变更
- `access: "restricted"`: 包默认为私有，可改为 "public"
- `baseBranch: "main"`: 指定主分支名称
- `updateInternalDependencies: "patch"`: 内部依赖更新策略

## 5. 创建示例包

### 5.1 工具包配置 (packages/utils)
```json
{
  "name": "@myrepo/utils",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "clean": "rm -rf dist"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```
**详细说明**：
- `name`: 使用 scope 名称 (@myrepo) 避免命名冲突
- `main`: 指定包的入口文件
- `types`: TypeScript 类型定义文件
- 构建脚本使用 TypeScript 编译器

### 5.2 创建工具包源代码
```bash
# 创建 packages/utils/src/index.ts
echo 'export const add = (a: number, b: number): number => a + b;
export const multiply = (a: number, b: number): number => a * b;
export const formatMessage = (msg: string): string => `[UTILS] ${msg}`;' > packages/utils/src/index.ts
```

### 5.3 UI 包配置 (packages/ui)
```json
{
  "name": "@myrepo/ui",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc && vite build",
    "dev": "vite build --watch",
    "clean": "rm -rf dist"
  },
  "dependencies": {
    "@myrepo/utils": "workspace:*"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^4.4.0"
  }
}
```
**详细说明**：
- `dependencies` 中使用 `workspace:*` 引用本地工作区包
- 构建过程结合了 TypeScript 编译和 Vite 打包
- 依赖关系确保构建顺序正确

### 5.4 创建 UI 包源代码
```bash
# 创建 packages/ui/src/index.ts
echo 'import { formatMessage } from "@myrepo/utils";

export const Button = () => {
  console.log(formatMessage("Button component loaded"));
  return "Button Component";
};

export const Card = () => {
  return "Card Component";
};' > packages/ui/src/index.ts
```

## 6. TypeScript 配置

### 6.1 根目录 tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@myrepo/utils": ["./packages/utils/src"],
      "@myrepo/ui": ["./packages/ui/src"]
    }
  }
}
```
**详细说明**：
- 配置适用于整个 monorepo 的 TypeScript 设置
- `paths` 配置允许在包之间直接导入源代码
- `baseUrl: "."` 让路径映射相对于根目录
- 严格的类型检查设置确保代码质量

## 7. 完整的工作流程演示

### 7.1 开发新功能
```bash
# 1. 切换到新分支
git checkout -b feat/new-component

# 2. 在 packages/utils 中添加新功能
# 编辑 packages/utils/src/index.ts，添加：
# export const subtract = (a: number, b: number): number => a - b;

# 3. 构建测试
pnpm build
```
**详细说明**：
- Turborepo 会检测到 utils 包的变化并重新构建
- 由于 UI 包依赖 utils，UI 包也会被重新构建
- 缓存机制确保未变化的包不会重复构建

### 7.2 创建变更集（关键步骤）
```bash
# 运行 changeset 命令
pnpm changeset
```
**详细说明**：
这个命令会进入交互式界面：

```
🦋  What kind of change is this for @myrepo/utils? (current version is 1.0.0)
❯ patch (1.0.0 -> 1.0.1)
  minor (1.0.0 -> 1.1.0) 
  major (1.0.0 -> 2.0.0)

🦋  What kind of change is this for @myrepo/ui? (current version is 1.0.0)
❯ patch (1.0.0 -> 1.0.1)
  minor (1.0.0 -> 1.1.0)
  major (1.0.0 -> 2.0.0)

🦋  Please enter a summary for this change (this will be in the changelogs)
  (submit empty line to open external editor)
❯ Add subtract function to utils package
```

选择后会在 `.changeset/` 目录生成一个 MD 文件：
```markdown
---
"@myrepo/utils": patch
"@myrepo/ui": patch
---

Add subtract function to utils package
```

### 7.3 提交变更集
```bash
# 添加变更集文件
git add .changeset/

# 提交变更
git commit -m "feat: add subtract function to utils"
```

### 7.4 版本发布准备
```bash
# 应用变更集，更新版本号和生成 changelog
pnpm version-packages
```
**详细说明**：
这个命令会：
1. 读取 `.changeset/` 中的变更集文件
2. 更新 `packages/utils/package.json` 和 `packages/ui/package.json` 的版本号
3. 生成或更新 `CHANGELOG.md` 文件
4. 自动提交这些变更（因为配置了 `"commit": true`）

### 7.5 发布包
```bash
# 发布到 npm registry
pnpm release
```
**详细说明**：
- 这个命令会将所有版本号更新的包发布到 npm
- 需要提前配置 `npm login` 和 NPM_TOKEN
- 发布前会自动构建所有包

## 8. CI/CD 集成

### 8.1 GitHub Actions 配置
创建 `.github/workflows/release.yml`：
```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'pnpm'

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Install Dependencies
        run: pnpm install

      - name: Build Packages
        run: pnpm build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```
**详细说明**：
- 在推送到 main 分支时自动运行
- `fetch-depth: 0` 确保获取完整的 git 历史
- Changesets Action 会自动处理版本管理和发布
- 需要设置 `NPM_TOKEN` 和 `GITHUB_TOKEN` 密钥

## 9. 验证工作流程

### 9.1 测试完整流程
```bash
# 1. 确保所有包都能构建
pnpm build

# 2. 运行测试（如果有）
pnpm test

# 3. 创建测试变更集
pnpm changeset
# 选择 patch 更新，输入测试信息

# 4. 应用变更集（不实际发布）
pnpm version-packages

# 5. 检查生成的 changelog
cat packages/utils/CHANGELOG.md
```

### 9.2 检查最终项目结构
```
my-turborepo/
├── .changeset/
│   ├── config.json
│   └── *.md (变更集文件)
├── .github/
│   └── workflows/
│       └── release.yml
├── packages/
│   ├── ui/
│   │   ├── src/
│   │   ├── dist/
│   │   ├── package.json
│   │   └── CHANGELOG.md
│   └── utils/
│       ├── src/
│       ├── dist/
│       ├── package.json
│       └── CHANGELOG.md
├── apps/
│   └── web/
├── package.json
├── turbo.json
├── pnpm-workspace.yaml
└── tsconfig.json
```

这个详细的流程说明涵盖了从零开始搭建 Turborepo + Changesets 项目的每一个步骤，确保你能够理解每个配置的作用和整个工作流程的运作方式。
