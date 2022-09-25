---
title: vue 2.x 接入 typescript
toc: true
cover: /cover/vue-typescript.jpg
categories:
  - Web 技术
tags:
  - vue
  - typescript
abbrlink: '5141e527'
date: 2022-01-08 11:50:19
---

对于新项目，通过 [vue-cli](https://cli.vuejs.org/zh/guide/creating-a-project.html#vue-create) 手动选择需要的特性就可以很方便的接入 typescript，但对于一开始没有引入 typescript 的 vue 项目，再进行接入就较为繁琐了，最近公司就有一个项目是这种情况，折腾了比较久，在这里整理一下完整的引入方式。

<!--more-->

## 基本框架

为了更好的演示效果，使用 vue-cli 工具创建了一个默认的模板项目，即 preset 选择 default：

<img src="创建默认模板.png" alt="创建默认模板" style="zoom: 100%;margin: auto;display: block"/>

创建完成后项目技术栈为经典的 vue + js，项目目录结构如下：

<img src="目录结构.png" alt="目录结构" style="zoom: 100%;margin: auto;display: block"/>

该模板项目已经同步到 GitHub 上：[vue2.x-ts-template](https://github.com/cherrow/vue2.x-ts-template)，接下来开始进行 typescript 的引入。

## 安装依赖

项目根目录下执行：

```bash
npm install --save-dev typescript ts-loader@8.2.0 fork-ts-checker-webpack-plugin
```

* [typescript](https://github.com/Microsoft/TypeScript): 无需太多解释，提供 ts 支持
* [ts-loader](https://github.com/TypeStrong/ts-loader): 解析 ts 文件，⚠️**注意**：webpack 4.x 对应的 `ts-loader` 版本必须是 8.x
* [fork-ts-checker-webpack-plugin](https://github.com/TypeStrong/fork-ts-checker-webpack-plugin): 更全面的 ts 语法检测，提供更完善的报错信息，以及对 `.vue` 单文件的支持

 ## 配置 webpack

项目根目录下创建 `vue.config.js` 文件，配置代码如下：

```javascript vue.config.js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin')

module.exports = {
  configureWebpack: {
    // 入口文件根据项目实际情况填写，如不写，默认为 main.js
    entry: './src/main.ts',
    resolve: { extensions: ['.ts', '.tsx'] },
    module: {
      rules: [
        {
          // 解析 ts 与 tsx 文件
          test: /\.tsx?$/,
          loader: 'ts-loader',
          exclude: /node_modules/,
          options: {
            transpileOnly: true,
            appendTsSuffixTo: [/\.vue$/],
            happyPackMode: false,
          },
        },
      ],
    },
    plugins: [
      // 用于提供完善的报错信息
      new ForkTsCheckerWebpackPlugin({
        async: false,
        typescript: {
          // 提供对 .vue 单文件的检测支持
          extensions: {
            vue: true,
          },
        },
      }),
    ],
  },
}

```

## 配置 tsconfig.json

项目根目录下创建 `tsconfig.json` 文件，配置代码如下：

```json tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "noEmitOnError": true,
    "target": "esnext",
    "module": "esnext",
    "strict": true,
    "allowJs": true,
    "noEmit": false,
    "noImplicitThis": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["**/*.vue", "**/*.ts"],
  "exclude": ["node_modules", "dist"]
}

```

至此，你已经可以进行 typescript 的编码了，语法有误时，编译报错如下：

<img src="编译报错.png" alt="编译报错" style="zoom: 100%;margin: auto;display: block"/>

接下来的步骤视项目具体情况而定。

## 添加自定义变量声明

开发过程中通常会有自定义变量挂载到 vue 原型上的情况，以便于在其他地方直接使用 `this.` 进行调用，如

```javascript main.ts
import Vue from 'vue'
import * as api from "@/service/api";

// 将 $api 对象挂载到 vue 原型
Vue.prototype.$api = api

```

但在其他的 ts 文件中进行引用时，会发现 `$api` 对象无法识别导致报错：

```typescript home.vue
this.$api.getAppName()
// error: TS2339: Property '$api' does not exist on type 'CombinedVueInstance   >>'.
```

这是因为 typescript 对类型有严格的检测机制，未声明的属性或变量被引用了就会报错，解决方法即手动添加一个类型定义文件，在 src 目录下创建 `global.d.ts` 文件，定义 vue 上的自定义变量：

```typescript global.d.ts
import Vue from 'vue'

declare module 'vue/types/vue' {
  interface Vue {
    $api: any
  }
}
```

## 添加 window 变量声明

与上一步骤类似，对于 window 上的自定义变量，如果没有定义就引用，同样会导致报错，解决方法为在 src 目录下创建 `window.d.ts` 文件，定义 window 上的自定义变量：

```typescript window.d.ts
interface Window {
  customProperty: any
}
```

然后就可以引用自定义属性了:  `window.customProperty`

## 写在最后

完成以上所有步骤后，[demo](https://github.com/cherrow/vue2.x-ts-template) 项目的目录结构如下：

<img src="配置后目录结构.png" alt="配置后目录结构" style="zoom: 100%;margin: auto;display: block"/>

关于 vue 中如何使用 typescript，可以参考官方文档[基本用法](https://cn.vuejs.org/v2/guide/typescript.html#%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95)，接下来就愉快的使用 ts 吧～
