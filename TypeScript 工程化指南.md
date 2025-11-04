# TypeScript 工程化完整指南

## 📖 前言

TypeScript 作为 JavaScript 的超集，不仅提供了静态类型检查，更重要的是它为现代前端开发带来了更好的工程化体验。本文将全面介绍 TypeScript 工程化生态中的核心工具和最佳实践，帮助开发者构建高效、可维护的 TypeScript 项目。

## 🎯 什么是 TypeScript 工程化

TypeScript 工程化是指在 TypeScript 项目开发过程中，通过一系列工具和流程来提高开发效率、代码质量和项目可维护性的实践。它包括：

- **构建工具链**：编译、打包、优化
- **开发工具**：代码检查、格式化、调试
- **项目管理**：依赖管理、版本控制、发布流程
- **质量保证**：测试、文档、持续集成

## 🛠️ 核心工具分类

### 1. 构建工具类

#### Vite - 下一代前端构建工具
```bash
# 安装
npm create vite@latest my-ts-app -- --template vanilla-ts

# 特点
- 极速的开发服务器（基于 ESM）
- 原生 TypeScript 支持
- 丰富的插件生态
- 优化的生产构建（基于 Rollup）
```

**适用场景**：现代前端应用开发，特别是 Vue、React 项目

#### Rollup - 模块打包器
```bash
# 安装
npm install --save-dev rollup @rollup/plugin-typescript

# 配置示例
// rollup.config.js
export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/bundle.js',
    format: 'esm'
  },
  plugins: [typescript()]
}
```

**适用场景**：库开发、Tree-shaking 优化需求

#### esbuild - 极速构建工具
```bash
# 安装
npm install --save-dev esbuild

# 使用
esbuild src/index.ts --bundle --outfile=dist/bundle.js
```

**适用场景**：需要极速构建的项目、CI/CD 环境

### 2. 包管理和 Monorepo

#### pnpm - 高效包管理器
```bash
# 安装
npm install -g pnpm

# 特点
- 节省磁盘空间（硬链接机制）
- 更快的安装速度
- 严格的依赖管理
- 原生 Monorepo 支持
```

**Monorepo 配置示例**：
```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

#### Nx - 智能构建系统
```bash
# 创建 Nx 工作空间
npx create-nx-workspace@latest myworkspace --preset=ts

# 特点
- 智能缓存和增量构建
- 依赖图分析
- 代码生成器
- 插件生态系统
```

#### Lerna - 传统 Monorepo 管理
```bash
# 安装和初始化
npm install -g lerna
lerna init

# 发布流程
lerna version
lerna publish
```

### 3. 开发工具链

#### TypeScript ESLint - 代码质量检查
```bash
# 安装
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin

# 配置 .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended'
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn'
  }
}
```

#### Prettier - 代码格式化
```bash
# 安装
npm install --save-dev prettier

# 配置 .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

#### Husky - Git Hooks 管理
```bash
# 安装和配置
npm install --save-dev husky lint-staged
npx husky install

# 添加 pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"
```

**lint-staged 配置**：
```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

### 4. 脚手架和模板工具

#### 现代脚手架对比

| 工具 | 适用框架 | TypeScript 支持 | 特点 |
|------|----------|----------------|------|
| Vite | Vue/React/Vanilla | ✅ 原生支持 | 极速开发体验 |
| Create React App | React | ✅ 内置模板 | 零配置启动 |
| Vue CLI | Vue | ✅ 插件支持 | 图形化界面 |
| Angular CLI | Angular | ✅ 默认使用 | 企业级特性 |

#### 自定义脚手架开发
```typescript
// 使用 Yeoman 创建自定义生成器
import Generator from 'yeoman-generator';

export default class extends Generator {
  async prompting() {
    this.answers = await this.prompt([
      {
        type: 'input',
        name: 'name',
        message: 'Project name?'
      }
    ]);
  }

  writing() {
    this.fs.copyTpl(
      this.templatePath('package.json'),
      this.destinationPath('package.json'),
      { name: this.answers.name }
    );
  }
}
```

### 5. 构建配置工具

#### tsup - 零配置库打包
```bash
# 安装
npm install --save-dev tsup

# 使用
tsup src/index.ts --format cjs,esm --dts
```

**配置文件示例**：
```typescript
// tsup.config.ts
import { defineConfig } from 'tsup'

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'],
  dts: true,
  splitting: false,
  sourcemap: true,
  clean: true,
})
```

#### unbuild - 统一构建系统
```bash
# 安装
npm install --save-dev unbuild

# 配置 build.config.ts
export default {
  entries: ['src/index'],
  declaration: true,
  rollup: {
    emitCJS: true
  }
}
```

### 6. 测试工具

#### Vitest - 现代测试框架
```bash
# 安装
npm install --save-dev vitest @vitest/ui

# 配置 vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts'
  }
})
```

**测试示例**：
```typescript
// src/utils.test.ts
import { describe, it, expect } from 'vitest'
import { add } from './utils'

describe('add function', () => {
  it('should add two numbers correctly', () => {
    expect(add(2, 3)).toBe(5)
  })
})
```

#### Jest - 成熟测试解决方案
```bash
# 安装
npm install --save-dev jest @types/jest ts-jest

# 配置 jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts'
  ]
}
```

### 7. 代码分析工具

#### Biome - 一体化工具链
```bash
# 安装
npm install --save-dev @biomejs/biome

# 配置 biome.json
{
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space"
  }
}
```

#### SWC - 高性能编译器
```bash
# 安装
npm install --save-dev @swc/core @swc/cli

# 配置 .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "target": "es2020"
  }
}
```

### 8. 文档工具

#### TypeDoc - API 文档生成
```bash
# 安装
npm install --save-dev typedoc

# 生成文档
npx typedoc src/index.ts
```

**配置示例**：
```json
{
  "typedoc": {
    "entryPoints": ["src/index.ts"],
    "out": "docs",
    "theme": "default"
  }
}
```

#### Storybook - 组件文档
```bash
# 初始化
npx storybook@latest init

# 配置 TypeScript
# .storybook/main.ts
export default {
  typescript: {
    check: false,
    reactDocgen: 'react-docgen-typescript'
  }
}
```

## 🏗️ 最佳实践工程化配置

### 完整项目结构
```
my-ts-project/
├── src/
│   ├── components/
│   ├── utils/
│   ├── types/
│   └── index.ts
├── tests/
├── docs/
├── .github/
│   └── workflows/
├── .husky/
├── .vscode/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts
├── .eslintrc.js
├── .prettierrc
└── README.md
```

### TypeScript 配置最佳实践
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 包管理配置
```json
// package.json
{
  "name": "my-ts-project",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "lint": "eslint src --ext .ts,.tsx",
    "lint:fix": "eslint src --ext .ts,.tsx --fix",
    "format": "prettier --write src/**/*.{ts,tsx}",
    "type-check": "tsc --noEmit",
    "prepare": "husky install"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^4.0.0",
    "vitest": "^0.34.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "husky": "^8.0.0",
    "lint-staged": "^14.0.0"
  }
}
```

## 🚀 CI/CD 集成

### GitHub Actions 配置
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
          
      - name: Install pnpm
        run: npm install -g pnpm
        
      - name: Install dependencies
        run: pnpm install
        
      - name: Type check
        run: pnpm type-check
        
      - name: Lint
        run: pnpm lint
        
      - name: Test
        run: pnpm test --coverage
        
      - name: Build
        run: pnpm build
```

## 📊 工具选择指南

### 根据项目类型选择工具

#### 小型库项目
- **构建**: tsup
- **测试**: Vitest
- **发布**: np 或 semantic-release

#### 中型应用项目
- **构建**: Vite
- **包管理**: pnpm
- **测试**: Vitest + Testing Library
- **代码质量**: ESLint + Prettier

#### 大型企业项目
- **构建**: Nx + Webpack/Vite
- **包管理**: pnpm workspaces
- **测试**: Jest + Cypress
- **文档**: Storybook + TypeDoc

#### Monorepo 项目
- **管理**: Nx 或 Lerna
- **构建**: 统一构建配置
- **发布**: Changesets

### 性能对比

| 工具类别 | 推荐工具 | 性能特点 | 适用场景 |
|----------|----------|----------|----------|
| 构建工具 | Vite > esbuild > Webpack | 开发速度 | 现代应用 |
| 包管理器 | pnpm > Yarn > npm | 安装速度 | 所有项目 |
| 测试框架 | Vitest > Jest | 运行速度 | 现代项目 |
| 代码检查 | Biome > ESLint | 检查速度 | 追求极致性能 |

## 🔮 未来趋势

### 新兴工具
- **Bun**: 全新的 JavaScript 运行时和包管理器
- **Turbo**: 高性能构建系统
- **Rome/Biome**: 一体化工具链
- **Deno**: 安全的 TypeScript 运行时

### 发展方向
1. **更快的构建速度**: 基于 Rust/Go 的工具
2. **更好的开发体验**: 零配置、智能提示
3. **更强的类型安全**: 更严格的类型检查
4. **更完善的工具链**: 一体化解决方案

## 📝 总结

TypeScript 工程化是一个不断发展的领域，选择合适的工具组合对项目成功至关重要。建议：

1. **从简单开始**: 使用 Vite + TypeScript 快速启动
2. **逐步完善**: 根据需求添加 ESLint、Prettier、测试等
3. **保持更新**: 关注新工具和最佳实践
4. **团队协作**: 统一工具链和配置标准

记住，工程化的目标是提高效率和质量，而不是为了使用工具而使用工具。选择最适合你项目和团队的工具组合，才是最好的工程化实践。

---


*本文涵盖了 TypeScript 工程化的主要方面，随着生态的发展，建议定期更新和调整工具选择。*
