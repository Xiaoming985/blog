---
title: Monorepo + PNPM + Vite + Vue3 + Typescript 搭建组件库
date: 2023-07-27 15:49:08
tags:
 - 组件库
 - Monorepo
 - Vite + Vue3 + Typescript
---

许久不见，甚是想念～

好久没有更新博客了，今天给大家分享一下搭建组件库的方法。良心干货，赶紧码住收藏吧～

## 安装PNPM
```bash
$ npm install -g pnpm
```

## 初始化项目
```bash
$ mkdir yumyum-ui && cd yumyum-ui
$ pnpm init
```

## 新建.npmrc文件
```
shamefully-hoist=true # 作用依赖包都扁平化的安装在node_modules下面
# strict-peer-dependencies=false
# shell-emulator=true
```

## 新建pnpm-workspace.yaml文件
```yaml
packages:
  - packages/** # 存放所有组件
  - docs # 文档
  - examples
```

## 新建packages、docs、examples目录
在项目根目录创建pakages、docs、examples目录
- packages：组件目录
- docs：文档
- examples：demo案例

## 手动搭建一个examples项目(组件demo)
```bash
$ cd examples
$ pnpm init
$ pnpm install vue@next typescript sass -D -w
$ pnpm install vite @vitejs/plugin-vue vue-tsc -D
# 在examples/package.json中添加启动脚本     "script": {"dev": "vite"}
# 开发环境中的依赖一般全部安装在整个项目根目录下，方便每个包都可以引用，所以在安装的时候需要加个 -w
# 当然，也可以使用pnpm create vite构建
# 另外，在项目的根目录，即跟examples目录的平级的package.json中的添加启动脚本 "script": {"dev": "pnpm -C examples dev"}，这样就可以在根目录下启动examples工程了
```

```json
// examples/package.json
{
  "name": "@yumyum-ui/examples",
  "version": "0.0.1",
  "description": "",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview"
  },
  "keywords": [],
  "author": "xiaoming985",
  "license": "MIT"
}
```

```json
// package.json
{
  "name": "yumyum-ui",
  "version": "0.0.3",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "pnpm -C examples dev"
  }
  // ...
}
```

## tsconfig.json
在项目的根目录下创建tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": true
  },
  "include": [ // 指定哪些文件需要被编译
    "**/*.ts", 
    "**/*.d.ts", 
    "**/*.tsx", 
    "**/*.vue"
  ],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```json
{
  "compilerOptions": {
    "composite": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true
  },
  "include": ["**/vite.config.ts"]
}
```

## packages/utils
```
pnpm init
# 执行完成后，修改该目录下的package.json        =>       "name": "@yumyum-ui/utils"
```

## packages/components
```
pnpm init
# 执行完成后，修改该目录下的package.json        =>       "name": "@yumyum-ui/components"

pnpm install @yumyum-ui/utils 
# 在components中引入utils，执行完成后会看到"dependencies": {"@yumyum-ui/utils": "workspace:^1.0.0"}
# 自动构建了软链接指向了utils目录，注意组织名（@yumyum-ui）不重复，否则可能会从npm库中下了一个同名的包
```

### 使用vite打包components
```ts
// components/vite.config.ts
import { defineConfig } from "vite"
import vue from "@vitejs/plugin-vue"
import dts from "vite-plugin-dts"
import { resolve } from "path"

export default defineConfig(
  {
    build: {
      target: 'modules',
      //打包文件目录
      outDir: "es",
      //压缩
      minify: false,
      //css分离
      //cssCodeSplit: true,
      rollupOptions: {
        //忽略打包vue文件
        external: ['vue'],
        input: ['index.ts'],
        output: [
          {
            format: 'es',
            //不用打包成.es.js,这里我们想把它打包成.js
            entryFileNames: '[name].js',
            //让打包目录和我们目录对应
            preserveModules: true,
            //配置打包根目录
            dir: resolve(__dirname, '../../build/es'), // 'es'
            preserveModulesRoot: 'src'
          },
          {
            format: 'cjs',
            entryFileNames: '[name].js',
            //让打包目录和我们目录对应
            preserveModules: true,
            //配置打包根目录
            dir: resolve(__dirname, '../../build/lib'), // 'lib'
            preserveModulesRoot: 'src'
          }
        ]
      },
      lib: {
        entry: './index.ts',
        formats: ['es', 'cjs'],
        name: 'yumyum-ui'
      }
    },
    plugins: [
      vue(),
      dts({
        // 指定使用的 tsconfig.json，如果不配置也可以在 components 下新建 tsconfig.json
        tsConfigFilePath: '../../tsconfig.json',
        // 因为这个插件默认打包到es下，我们想让lib目录下也生成声明文件需要再配置一个
        outputDir: [ // outputDir: 'lib'
          resolve(__dirname, '../../build/es/components'),
          resolve(__dirname, '../../build/lib/components')
        ]
      })
    ],
    resolve: {
      alias: {
        '@': resolve(__dirname, 'components')
      }
    }
  }
)
```

```json
// packages/components/package.json说明
{
  // ...
  "main": "lib/index.js",
  "module": "es/index.js", // # module：组件库默认入口文件是传统的 CommonJS 模块，但是如果环境支持 ESModule 的话构建工具会优先使用 module 入口
  "files": [ // files：files 用于指定需要发布到 npm 上的目录，因为不可能将 components 下的所有目录都发布上去
    "es",
    "lib"
  ],
  // ...
}
```

## 配置eslint
```
pnpm i eslint eslint-plugin-vue @typescript-eslint/parser @typescript-eslint/eslint-plugin -D -w
# vscode 需要安装eslint插件
```
- eslint: ESLint 的核心代码。
- eslint-plugin-vue：包含常用的 vue 规范。
- @typescript-eslint/parser：ESLint 的解析器，用于解析 typescript，从而检查和规范 Typescript 代码。
- @typescript-eslint/eslint-plugin：包含了各类定义好的检测 Typescript 代码的规范。
- eslint-plugin-import：意在提供对ES6+ import/export语法的支持，有助于防止你写错文件路径或者引用的变量名
- 根目录package.json添加script脚本："lint": "eslint . --ext .vue,.js,.ts,.jsx,.tsx --fix"
  
### 新建.eslintrc.js
```js
// 也可以使用pnpm eslint --init生成，生成的文件跟以下配置差不多，稍作修改即可
module.exports = {
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:vue/vue3-essential",
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended" // 安装prettier后，新增
  ],
  "overrides": [
  ],
  "parser": "vue-eslint-parser", // 新增/修改，原 => "parser": "@typescript-eslint/parser"
  "parserOptions": {
    "ecmaVersion": "latest",
    "parser": "@typescript-eslint/parser", // 新增
    "sourceType": "module"
  },
  "plugins": [
    "vue",
    "@typescript-eslint"
  ],
  "rules": { // 新增
    "@typescript-eslint/ban-types": [
      "error",
    {
        "extendDefaults": true,
        "types": {
          "{}": false
        }
      }
    ]
  }
}
```

### 新建.eslintignore
```
node_modules
dist
pnpm-lock.yaml
.eslintrc.js
```

## 配置prettier
```
pnpm i prettier eslint-plugin-prettier eslint-config-prettier -D -w
# vscode 需要安装prettier插件
```

### 新建.prettier.js
```js
module.exports = {
  printWidth: 100, //最大单行长度
  tabWidth: 2, //每个缩进的空格数
  useTabs: false, //使用制表符而不是空格缩进行
  semi: true, //在语句的末尾打印分号
  vueIndentScriptAndStyle: true, //是否缩进 Vue 文件中的代码<script>和<style>标签。
  singleQuote: true, //使用单引号而不是双引号
  quoteProps: 'as-needed', //引用对象中的属性时更改 "as-needed" "consistent" "preserve"
  bracketSpacing: true, //在对象文字中的括号之间打印空格
  trailingComma: 'none', //在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号）
  arrowParens: 'avoid', //箭头函数只有一个参数的时候是否使用括号 always：使用  avoid： 省略
  insertPragma: false, //是否在文件头部插入一个 @format标记表示文件已经被格式化了
  htmlWhitespaceSensitivity: 'css', //HTML 空白敏感性 css strict ignore
  endOfLine: 'auto', //换行符使用什么
  tslintIntegration: false //不让ts使用prettier校验
}
```

### 新建.prettierignore
```
node_modules
dist
pnpm-lock.yaml
.prettier.js
```

## VitePress 文档
- vitepress官网：https://vitepress.dev/
- Node.js >= 16
- 安装
```
pnpm install vitepress -D -w
```
- 初始化
```
npx vitepress init
```
```
  vitepress v1.0.0-alpha.74

┌   Welcome to VitePress! 
│
◇  Where should VitePress initialize the config?
│  ./docs
│
◇  Site title:
│  yumyum-ui
│
◇  Site description:
│  vue3组件库
│
◇  Theme:
│  Default Theme
│
◇  Use TypeScript for config and theme files?
│  Yes
│
◇  Add VitePress npm scripts to package.json?
│  Yes
│
└  Done! Now run npm run docs:dev and start writing.
```

## iconfont制作
- iconfont-阿里巴巴矢量图标库：https://www.iconfont.cn/

## packages/theme-chalk
- 新建packages/theme-chalk目录
- 在packages/theme-chalk下执行：pnpm init
- 修改该目录下的package.json         =>        "name": "@yumyum-ui/theme-chalk"

## 发布npm包
- package.json 中 files 属性，用于指定上传到npm的目录/文件
- .npmignore文件也可用于指定忽略上传的文件
- pnpm run build:components 执行成功后会在根目录下生成build目录
- cd build
- npm init 生成package.json文件，并修改（版本号等），另外还可手动添加README.md，有助于npm展示简介
- npm publish
```json
{
  "name": "yumyum-ui",
  "version": "0.0.1",
  "description": "A high quality UI Toolkit built on Vue.js3+.",
  "main": "lib/components/index.js",
  "module": "es/components/index.js",
  "exports": {
    ".": {
      "import": "./es/components/index.js",
      "require": "./lib/components/index.js"
    },
    "./*": "./*"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/xiaoming985/yumyum-ui"
  },
  "homepage": "https://xiaoming985.github.io/yumyum-ui",
  "bugs": {
    "url": "https://github.com/xiaoming985/yumyum-ui/issues"
  },
  "files": [
    "es",
    "lib"
  ],
  "keywords": [
    "yumyum-ui",
    "vue3组件库",
    "components",
    "vue ui library",
    "yumyumui",
    "YumyumUI",
    "vue"
  ],
  "typings": "lib/index.d.ts",
  "author": "xiaoming985",
  "license": "MIT"
}
```