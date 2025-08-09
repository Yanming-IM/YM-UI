# YM-UI Copilot 指导说明

## 架构概览

YM-UI 是一个基于 **Vue 3.6 Vapor 模式** 的现代组件库，采用 TypeScript + LESS，结合了 Vapor 模式的性能优势与传统组件库的成熟架构模式。核心理念：

- **Vue 3.6 Vapor 模式**: 使用 `<template vapor>` 实现直接 DOM 操作，绕过虚拟 DOM diff
- **TypeScript 严格模式**: 完整的类型系统，确保代码质量和开发体验
- **LESS + CSS 变量**: 灵活的样式系统，支持主题切换
- **组件化架构**: 采用成熟的目录结构和开发工作流

## 核心开发模式

### 组件结构
每个组件都遵循以下结构：
```
src/components/[name]/
├── index.ts          # 导出文件
├── [name].vue        # 主组件 (Vue SFC)
├── [name].less       # 组件样式
├── types.ts          # TypeScript 接口
├── README.md         # 组件文档
└── __tests__/        # 测试文件
```

### 组件创建
使用 `pnpm run create:component <name>` 命令生成新组件脚手架。脚本会生成：
- Vue SFC 组件使用 `<template vapor>` 模式
- `types.ts` 中的 TypeScript 接口
- 使用 CSS 变量的 LESS 样式表
- 基于 Vitest + @vue/test-utils 的测试文件

### Vue Vapor 组件模式
组件使用 Vue SFC + Vapor 模式：
```vue
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md'
});

const emit = defineEmits<{
  (e: 'click', event: MouseEvent): void;
  (e: 'focus'): void;
  (e: 'blur'): void;
}>();

const classes = computed(() => [
  'ym-button',
  `ym-button--${props.variant}`,
  `ym-button--${props.size}`,
  {
    'ym-button--disabled': props.disabled,
    'ym-button--loading': props.loading
  }
]);

function handleClick(event: MouseEvent) {
  if (!props.disabled && !props.loading) {
    emit('click', event);
  }
}
</script>

<template vapor>
  <button 
    :class="classes"
    :disabled="disabled || loading"
    @click="handleClick"
    @focus="$emit('focus')"
    @blur="$emit('blur')"
  >
    <span v-if="loading" class="vc-button__loading"></span>
    <slot />
  </button>
</template>

<style scoped lang="less">
@import '../../../styles/variables.less';

.ym-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: var(--ym-border-radius-md);
  font-family: var(--ym-font-family);
  font-weight: var(--ym-font-weight-medium);
  cursor: pointer;
  transition: all var(--ym-transition-duration) var(--ym-transition-timing);

  &--primary {
    background-color: var(--ym-color-primary);
    color: var(--ym-color-primary-contrast);
  }

  &--secondary {
    background-color: var(--ym-color-secondary);
    color: var(--ym-color-secondary-contrast);
  }

  &--danger {
    background-color: var(--ym-color-danger);
    color: var(--ym-color-danger-contrast);
  }

  &--sm {
    padding: var(--ym-space-xs) var(--ym-space-sm);
    font-size: var(--ym-font-size-sm);
  }

  &--md {
    padding: var(--ym-space-sm) var(--ym-space-md);
    font-size: var(--ym-font-size-md);
  }

  &--lg {
    padding: var(--ym-space-md) var(--ym-space-lg);
    font-size: var(--ym-font-size-lg);
  }

  &--disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  &--loading {
    cursor: wait;
  }

  &__loading {
    width: 1em;
    height: 1em;
    border: 2px solid transparent;
    border-top: 2px solid currentColor;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-right: var(--ym-space-xs);
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
```

## 开发工作流程

### 核心命令
- `pnpm dev` - 启动 Vite 开发服务器（启用 Vapor 模式）
- `pnpm dev:play` - 启动 Playground 开发环境
- `pnpm create:component <name>` - 生成新组件脚手架
- `pnpm build:lib` - 构建组件库（用于发布）
- `pnpm build:demo` - 构建演示页面
- `pnpm build:playground` - 构建 Playground
- `pnpm build:all` - 构建所有内容（组件库 + 演示页面 + Playground）
- `pnpm preview:lib` - 构建并预览组件库
- `pnpm preview:demo` - 构建并预览演示页面
- `pnpm preview:playground` - 构建并预览 Playground
- `pnpm test` - 运行 Vitest 测试（监听模式）
- `pnpm test:coverage` - 生成测试覆盖率报告

### 构建输出
- `dist/` - 组件库构建输出（ES/UMD/CJS + 类型声明）
- `dist-demo/` - 演示页面构建输出
- `dist-playground/` - Playground 构建输出
- 支持多种模块格式：ES Module、UMD、CommonJS

### 测试策略
- 使用 Vitest + @vue/test-utils 进行组件测试
- 支持 Vapor 模式的自定义测试预处理
- 所有组件必须包含属性、事件和边界情况的测试覆盖
- 测试文件位于组件旁的 `__tests__/` 目录中

### 📄 Playground 开发标准
Playground 作为组件开发和演示的主要环境，遵循以下标准：

- **开发环境引入**：使用 `import { createApp } from 'vue'` 和组件库源码
- **Vue 应用结构**：标准的 Vue 3 应用结构
- **路由系统**：使用 Vue Router 管理组件示例页面
- **主题切换**：集成主题切换功能，支持 light/dark 模式
- **组件演示**：每个组件都有对应的演示页面
- **实时编辑**：支持组件 props 的实时调整和预览

Playground 应用模板：
```vue
<!-- playground/src/App.vue -->
<script setup lang="ts">
import { ref } from 'vue';
import { RouterView } from 'vue-router';
import ThemeSwitcher from './components/ThemeSwitcher.vue';

const currentTheme = ref<'light' | 'dark'>('light');

function toggleTheme() {
  currentTheme.value = currentTheme.value === 'light' ? 'dark' : 'light';
  document.documentElement.setAttribute('data-theme', currentTheme.value);
}
</script>

<template vapor>
  <div id="app" class="playground">
    <header class="playground__header">
      <h1>YM-UI Component Library</h1>
      <ThemeSwitcher @change="toggleTheme" />
    </header>
    
    <main class="playground__main">
      <RouterView />
    </main>
  </div>
</template>

<style lang="less">
@import '../styles/variables.less';

.playground {
  min-height: 100vh;
  background-color: var(--ym-color-background);
  color: var(--ym-color-text);

  &__header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: var(--ym-space-md) var(--ym-space-lg);
    border-bottom: 1px solid var(--ym-color-border);
  }

  &__main {
    padding: var(--ym-space-lg);
  }
}
</style>
```

## 关键约定

### 🎨 LESS/CSS 编码规范
- **禁用 !important**: 严禁使用 `!important`，这是UI基础库，不允许强制性覆盖样式
- **🚫 禁止权重选择器**: 严禁使用重复类名增加权重（如`.ym-button.ym-button`），必须按照更规范的样式定义顺序解决优先级问题
- **🚫 样式变量强制在LESS中定义**: 严禁在JavaScript中定义、设置或操作CSS变量，所有样式变量必须在LESS文件中声明
- **纯CSS样式控制**: JavaScript只负责业务逻辑，严禁直接操作DOM样式属性（element.style.xxx）
- **LESS 分层嵌套**: 必须使用正确的LESS嵌套语法，避免长选择器重复
  ```less
  .ym-component {
    &__container {
      > .element {
        &:first-child {
          // 样式规则
        }
      }
    }
    
    &--variant {
      // 变体样式
    }
    
    &--state {
      // 状态样式
    }
  }
  ```
- **样式定义顺序规范**: 按照CSS层叠规则和源码顺序解决优先级问题，通过合理的类名设计和嵌套结构确保样式生效
- **变量使用**: 所有颜色、尺寸必须使用 CSS 变量，禁止硬编码值

### Vapor 模式约定
- **模板声明**: 所有组件必须使用 `<template vapor>`
- **环境变量**: 确保 `VUE_VAPOR=1` 环境变量在开发和构建时启用
- **性能优化**: 避免在模板中使用复杂的对象字面量，优先使用 computed 属性
- **测试兼容**: 使用自定义预处理插件确保测试环境的兼容性

### 样式系统
- 使用 `src/styles/variables.less` 中定义的 CSS 变量
- 组件样式从 `src/styles/mixins.less` 导入以保持一致的模式
- 通过文档根元素的 `data-theme` 属性切换主题
- 移动端优先的响应式设计方法

### 🌙 暗黑模式与主题系统强制要求
- **所有示例页面必须兼容暗黑模式**：每个演示都必须在 light/dark 模式下正常显示
- **强制主题切换器**：Playground 和演示页面都必须包含主题切换功能
- **CSS 变量依赖**：禁止硬编码任何颜色值，必须使用 CSS 变量以确保主题切换生效
- **主题测试**：创建组件时必须在两种主题下都进行测试验证
- **响应式主题**：支持系统主题自动切换（prefers-color-scheme）

### 组件属性和事件
- 使用 `defineProps<T>()` 和 `defineEmits<T>()` 进行类型定义
- 使用 `withDefaults()` 为 props 提供默认值
- 事件命名遵循 kebab-case 约定
- 布尔属性使用 `boolean` 类型而不是字符串

### 性能监控
在关键组件中添加性能标记：
```typescript
// 在 setup 中
if (import.meta.env.DEV) {
  performance.mark(`${componentName}-setup-start`);
  onMounted(() => {
    performance.mark(`${componentName}-setup-end`);
    performance.measure(
      `${componentName}-setup`,
      `${componentName}-setup-start`,
      `${componentName}-setup-end`
    );
  });
}
```

## 关键集成点

### Vapor 编译系统
- Vite 插件配置支持 Vapor 模式编译
- 环境变量 `VUE_VAPOR=1` 控制编译行为
- 测试环境使用自定义插件转换 `<template vapor>` 为标准模板

### 主题管理系统
- 基于 CSS 变量的主题系统
- 支持 'auto'、'light'、'dark' 模式
- 响应式主题切换，支持系统主题跟随

### 构建系统 (Vite + Rollup)
- TypeScript 编译，生成完整的类型声明
- LESS 预处理，自动注入 CSS 变量
- 支持热模块替换（HMR）进行组件开发
- 多格式输出：ES Module、CommonJS、UMD

### 测试系统 (Vitest)
- 基于 Vitest + @vue/test-utils
- 自定义 Vapor 模式预处理插件
- 覆盖率报告和性能测试

## 常见陷阱

1. **忘记 Vapor 标记**: 组件模板必须使用 `<template vapor>`，否则回退到标准编译
2. **环境变量缺失**: 确保开发和构建环境都设置了 `VUE_VAPOR=1`
3. **测试兼容性**: Vapor 模式在测试环境需要特殊处理，依赖自定义预处理插件
4. **CSS 变量硬编码**: 主题值来自 CSS 变量，不要在 JavaScript 中硬编码颜色
5. **复杂模板表达式**: 避免在 Vapor 模板中使用复杂对象，使用 computed 代替
6. **🚫 直接 DOM 操作**: 严禁直接操作 DOM 样式，使用 Vue 的响应式系统
7. **🚫 JavaScript样式操作**: 严禁在JavaScript中定义CSS变量或直接操作DOM样式，必须使用LESS/CSS
8. **组件导出遗漏**: 新建组件后记得在 `src/index.ts` 中导出
9. **类型定义缺失**: 确保所有 props 和 emits 都有完整的 TypeScript 类型定义
10. **主题切换测试遗漏**: 创建组件后必须在 light/dark 模式下都进行测试

## 常见任务的文件位置

- 添加新组件：`pnpm run create:component <name>`
- 修改主题变量：`src/styles/variables.less`
- 更新全局样式：`src/styles/base.less`
- 添加构建配置：`vite.config.ts` / `vite.config.playground.ts`
- 配置测试环境：`vitest.config.ts`
- Playground 路由配置：`playground/src/router/index.ts`
- 组件演示页面：`playground/src/views/[ComponentName].vue`

## 项目结构

```
project-root/
├── src/                          # 组件库源码
│   ├── components/               # 组件目录
│   │   └── [name]/              # 单个组件目录
│   │       ├── index.ts         # 导出
│   │       ├── [name].vue       # 组件实现
│   │       ├── [name].less      # 组件样式
│   │       ├── types.ts         # 类型定义
│   │       ├── README.md        # 组件文档
│   │       └── __tests__/       # 测试文件
│   ├── styles/                  # 全局样式
│   │   ├── variables.less       # CSS 变量定义
│   │   ├── mixins.less         # LESS 混入
│   │   └── base.less           # 基础样式
│   ├── utils/                   # 工具函数
│   └── index.ts                 # 主入口文件
├── playground/                   # Playground 应用
│   ├── src/
│   │   ├── components/          # Playground 组件
│   │   ├── views/              # 组件演示页面
│   │   ├── router/             # 路由配置
│   │   ├── styles/             # Playground 样式
│   │   ├── App.vue             # 主应用组件
│   │   └── main.ts             # 应用入口
│   ├── index.html              # HTML 入口
│   └── vite.config.ts          # Playground 构建配置
├── tests/                       # 全局测试
├── docs/                        # 文档
├── scripts/                     # 构建和开发脚本
├── dist/                        # 组件库构建输出
├── dist-playground/             # Playground 构建输出
├── vite.config.ts              # 主构建配置
├── vitest.config.ts            # 测试配置
├── package.json
└── README.md
```

## 开发最佳实践

### 组件设计原则
1. **单一职责**: 每个组件只负责一个特定功能
2. **可组合性**: 组件应该易于组合和扩展
3. **一致性**: 遵循统一的命名约定和API设计
4. **可访问性**: 提供必要的 ARIA 属性和键盘支持
5. **性能优先**: 利用 Vapor 模式的性能优势

### 代码质量保证
1. **类型安全**: 所有组件都必须有完整的 TypeScript 类型定义
2. **测试覆盖**: 核心功能必须有对应的单元测试
3. **文档完整**: 每个组件都需要有清晰的使用文档和示例
4. **代码审查**: 所有代码变更都需要经过 PR 审查
5. **性能监控**: 关键组件需要包含性能监控代码

这份指导说明结合了 Vue 3.6 Vapor 模式的最新特性和成熟的组件库架构模式，为新项目提供了完整的开发指南。请根据具体项目需求调整配置和约定。
