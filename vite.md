# 自己快速搭建

根目录下创建一个index.html，安装vite模块，输入vite指令就马上可以使用了

![image-20211208161538821](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211208161538821.png)

# TypeScript

vite原生就支持ts，不需要任何的配置，就可以运行ts文件

# 基础package.json

```js
{
  "name": "vite-my-build",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build --mode=staging",
    "preview": "vite preview"
  },
  "dependencies": {
    "@types/node": "^16.11.12",
    "vue": "^3.2.23"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^1.10.2",
    "@vitejs/plugin-vue-jsx": "^1.3.1",
    "less": "^4.1.2",
    "typescript": "^4.4.4",
    "vite": "^2.7.0",
    "vue-tsc": "^0.28.10"
  }
}
```

## 配置

默认配置

```ts
{
  "compilerOptions": {
    "target": "esnext",
    "useDefineForClassFields": true,
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"]
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}
```

# 热更新

原生支持热更新，也不需要任何的配置

# Vue配置

要想启动vue，因为需要解析.vue文件也需要像webpack一样安装插件，loader一样的东西。在vite里面也有他的插件，安装即可`@vitejs/plugin-vue`

## typescript支持

使用ts的时候，需要安装上这两个东西

```ts
vue-tsc": "^0.28.10",
"typescript": "^4.4.4"
```

不然你在vue中出现了类型错误，就不会被检查出来

```ts
// 这个类型显然是错误的，但是如果不装上面的东西，是不会被查出来的
const count = ref<number>("23213213")
```

## jsx和tsx支持

需要使用插件才能够使用`@vitejs/plugin-vue-jsx"`

可以使用函数风格的

```tsx
export default function Picker(): any {
    return (
        <div>picker</div>
    )
}
```

也可以使用defineComponent，写成类风格的

```tsx
import { onMounted, ref, defineComponent } from 'vue'
import Picker from "./Picker";

interface Person {
    name: number
}

const App = defineComponent({
    setup() {
        const count = ref<number>(2222)
        // mounted
        onMounted(() => {
            console.log('Component is mounted!')
        })

        return {
            count
        }
    },

    render() {
        return (
            <div>
                Hello Vue 3 {this.count}
                <Picker></Picker>
            </div>
        )
    }
})

export default App
```

# css支持

安装了less或者scss，就直接可以使用

## css module

任何以*.module.css为后缀的css文件都被认为是一个css modules文件，导出文件会返回一个模块对象

```js
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

这个不能够直接使用，需要进行配置



# 静态资源处理

导入一个静态资源会返回解析后的url

```js
// 导入一个静态资源会返回解析后的 URL：
import assetAsURL from './asset.png'
// 打印结果/src/asserts/1.png
```

添加一些特殊的查询参数可以更改资源被引入的方式

```js
// 显式加载资源为一个 URL
import assetAsURL from './asset.js?url'

// 没有了url就变成了一个拿到一个对象
```

```
// 以字符串形式加载资源
import assetAsString from './shader.glsl?raw'

// 加载为 Web Worker
import Worker from './worker.js?worker'

// 在构建时 Web Worker 内联为 base64 字符串
import InlineWorker from './worker.js?worker&inline'
```

## public目录

如果你有这些资源：

- 不会被源码引用
- 必须保持原有文件名
- 压根不引入，只想的到他的url

那么你就可以将该资源放在指定的public目录中，位于项目的根目录，开发的时候能直接通过/根路径访问到，并且打包会被完整复制到目标目录的根目录下

目录默认是<root>/public

注意：

- 引入 `public` 中的资源永远应该使用根绝对路径 —— 举个例子，`public/icon.png` 应该在源码中被引用为 `/icon.png`。而不是写http://localhost:3000/image，这样写是错误的
- pulbic中资源不应该被js文件，import

```js
<img style="height: 300px;width:300px" src="/image.png" alt=""/>
// 正确

<img style="height: 300px;width:300px" src="http://localhost:3000/image" alt=""/>
// 错误，引入不到的
```

## new URL(url, import.meta.url)

[import.meta.url](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) 是一个 ESM 的原生功能，会暴露当前模块的 URL。将它与原生的 [URL 构造器](https://developer.mozilla.org/en-US/docs/Web/API/URL) 组合使用，在一个 JavaScript 模块中，通过相对路径我们就能得到一个被完整解析的静态资源 URL：

```js
const imgUrl = new URL('./img.png', import.meta.url).href

document.getElementById('hero-img').src = imgUrl

import.meta.url //http://localhost:3000/src/App.tsx?t=1638964490085
```

# 构建生产版本

当需要构建生产环境的时候，只需要运行vite build，默认情况下，它使用<root>/index.html作为构建入口

## 浏览器兼容性

他打包出的html会使用这个，模板如下“

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
  <script type="module" crossorigin src="/assets/index.db6f2447.js"></script>
  <link rel="modulepreload" href="/assets/vendor.458152d2.js">
  <link rel="stylesheet" href="/assets/index.f3e7f869.css">
</head>
<body>
<div id="app"></div>

</body>
</html>
```

那么就需要你的浏览器能够支持 `type="module"`，否则就翻车了。

vite使用browserslist作为查询标准

**传统浏览器可以通过插件 [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) 来支持**

## 公共基础路径

如果你需要在嵌套的公共路径部署项目，只需要指定base路径，这个也可以通过命令行参数执行例如 `vite build --base=/my/public/path/`

例如：

```
"build": "vite build --base=/admin"
```

![image-20211209104523834](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211209104523834.png)

打包出来的资源路径会自动加上admin，你就可以部署在二级目录下了

或者配置文件里面写也可以的

```js
export default {
   base: '/nl'
}
```

### 自定义构建

结合Rollup，其实这个Rollup和其他的打包工具是一个什么关系，这个我不知道？？？？

## 文件变化时重新构建

你可以通过参数，也可以的通过配置

注意：**这里构建是指产生构建，不是热更新的开发环境重新构建**

`vite build --watch`

```js
// vite.config.js
module.exports = defineConfig({
  build: {
    watch: {
      // https://rollupjs.org/guide/en/#watch-options
      // 看着也是rollup的用法
    }
  }
})
```

## 多页面应用模式

```js
build: {
   rollupOptions: {
      input: {
         main: resolve(__dirname, 'index.html'),
         nested: resolve(__dirname, 'nested/index.html')
      }
   }
}
```

打包之后

![image-20211209110647511](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211209110647511.png)

```html
<a href="/nested/index.html">跳转</a>
```

## 库模式

当要把库就行发布过构建时需要使用build.lib配置项，确保那些你不希望打包进库的依赖进行外部化处理。

### build.lib

- entry，打包入口，string
- name： string，暴露的全局变量
- formats：es，cjs，umd，iife，[]，数组。数组需要包含umd或者iife，默认[es, umd]
- filename：string是打包的packjson name的配置项



# 静态站点部署

## preview作用

preview可以以dist文件构建一个本地服务，方便我们在本地进行测试。

就像我们之前使用vue cli的时候，build之后，生成一个dist，但是你直接打开那些资源路径是有问题的无法进行本地测试，preview，就是帮助你搭建一个本地服务，让你可以进行测试

`vite preview` 命令会在本地启动一个静态 Web 服务器，将 `dist` 文件夹运行在 [http://localhost:5000](http://localhost:5000/)。这样在本地环境下查看该构建产物是否正常可用就方便了。

你可以通过 `--port` 参数来配置服务的运行端口。

```js
"preview": "vite preview --port 8080"
```

## 接入自动部署

# 环境变量和模式

在vite中暴露了一些环境变量在**`import.meta.env`** 对象上。我们可以在我们的页面中访问到这些环境变量，注意：**不是在node中的环境变量，node中访问这个对象是没有东西的，是在web中**

控制台打印：

![image-20211209144001653](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211209144001653.png)

## .env文件

在根目录下创建一个.env文件，我们在这个env里面可以增加环境变量

```
.env                # 所有情况下都会加载
.env.local          # 所有情况下都会加载，但会被 git 忽略
// 这个可以根据你的mode不同读取不同的env文件
.env.[mode]         # 只在指定模式下加载
.env.[mode].local   # 只在指定模式下加载，但会被 git 忽略
```

```
DB_PASSWORD=foobar
VITE_SOME_KEY=123
```

但是注意，**为了防止泄露**，只有加上了`VITE_` 为前缀的变量，才会暴露给客户端访问。

有个问题DB_PASSWORD=foobar，这个东西怎么访问呢？？？？

## 环境变量ts智能提示

正常情况下，直接使用`import.meta.env`，ts时会报错误的，为了让编译器能够识别，我们在src目录下加上一个`env.d.ts`就可以了

```ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
  // 更多环境变量...
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## 模式

我们可以通过在buid命令中增加参数

```
vite build --mode staging
```

这个参数作用就是，让我们的应用在不同的mode模式下启动，那么我们可以创建对应的.env文件，那么我们就可以根据不同的环境变量来输出不同的内容，例如上面的staging，我们可以创建一个.env.staging

```
VITE_TITLE=我让你
```

这样我们就可以动态改变我们的html标题了

# 构建优化

下面的功能会自动作为构建的一部分，不需要明确指出来，除非你想禁用他

## css代码分割

vite会自动将一个异步的chunk模块使用到的css抽取出来成为一个单独的文件。这个css文件将在该异步chunk加载完成时自动通过一个link标签载入，该异步chunk会保证只在css加载完毕后再执行。

你可以通过`build.cssCodeSplit:false`禁用css代码分割

亲测：假如某个组件通过`import()`函数引入，css代码确实会被分割出来

![image-20211223154146080](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211223154146080.png)

## 预加载执指令生成

vite回味入口的chunk和他们在打包的出的html直接引入自动生成<link rel="modulepreload">指令

## 异步chunk优化

在实际项目中，rollup通常会生成公用的chunk，就是被两个或以上的chunk共享的chunk。这会出现一个问题![image-20211223154645707](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211223154645707.png)

由于A动态导入了，c。当浏览器解析A的时候，才知道它需要使用c，这个时候才会去加载c模块，这就会导致多一次的网络请求，这是不好的

```js
Entry ---> A ---> C
```

vite将使用一个预计加载步骤自动重写代码，来分割动态导入调用，以实现A被请求时，C也将被同时请求到

```js
Entry ---> (A + C)
```

假如c又引入了别的，这个时候可能会有更加深入的网络请求，vite会一直追踪，一直合并

# 使用插件

## 添加一个插件

先安装在devDependencies选项下，然后添加到配置文件，plugins数组中

```js
$ npm i -D @vitejs/plugin-legacy
```

```js
// vite.config.js
import legacy from '@vitejs/plugin-legacy'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    legacy({
      targets: ['defaults', 'not IE 11']
    })
  ]
})
```

## 查找插件

官方插件：https://vitejs.cn/plugins/

社区插件：https://github.com/vitejs/awesome-vite#plugins

# 插件api

vite底层是基于rollup的，所以他的插件方案有两种，一个是兼容rollup插件和vite的专属插件。

## 兼容rollup插件

相当数量的rollup插件将直接作为vite插件工作，但并不是所有都可以，因为有一些插件的钩子在非构建式的开发服务器上下文没有意义，一般来说rollup插件符合一下标准就能够项vite插件一样工作：

- 没有moduleParsed钩子
- 他在打包狗子和输出钩子之间没有很强的耦合

如果一个rollup插件只在构建阶段有意义，则在`build.rollupOptions.plugins` 下指定即可，你也可以用vite的专属语法。

## 约定

rollup插件应该有一个带rollup-plugin-前缀

对于vite的专属插件：

- vite插件应该有一个带vite-plugin-前缀，语义清晰的名称

如果你的插件适用于特定框架：

- vite-plugin-vue-前缀

vite对于虚拟模块的规范实在路径前在上`virtual：`如果可能的话，插件名应该作为命名空间使用，以避免与生态系统中其他插件发生冲突。例如，`vite-plugin-posts` 可以让用户引入 `virtual:posts` 或 `virtual:posts/helpers` 虚拟模块，以获得构建时信息。在内部，使用虚拟模块的插件在解析模块id时应以\0为前缀

## 简单示例

引入一个虚拟文件：**理论上我们的插件是服务于我们的构建流程的，而不是给我们的引用的，所以我们引用的是一个虚拟的文件和真实文件的写法肯定是不同的**

```js
export default function myPlugin() {
  const virtualModuleId = '@my-virtual-module'
  const resolvedVirtualModuleId = '\0' + virtualModuleId

  return {
    name: 'my-plugin', // 必须的，将会在 warning 和 error 中显示
    resolveId(id) {
      if (id === virtualModuleId) {
        return resolvedVirtualModuleId
      }
    },
    load(id) {
      if (id === resolvedVirtualModuleId) {
        return `export const msg = "from virtual module"`
      }
    }
  }
}
```

```js
import { msg } from '@my-virtual-module'

console.log(msg)
```

转换自定义文件类型

```js
const fileRegex = /\.(my-file-ext)$/

export default function myPlugin() {
  return {
    name: 'transform-file',

    transform(src, id) {
      if (fileRegex.test(id)) {
        return {
          code: compileFileToJS(src),
          map: null // 如果可行将提供 source map
        }
      }
    }
  }
}
```

插件的写法和用法十分类似于rollup

## 通用钩子

就是rollup的钩子

## vite独有钩子

### config

在解析vite配置前调用，钩子接受原始用户配置和一个描述配置环境的变量，包含正在使用的mode和command。它返回一个将被深度合并到现有配置的部分配置对象，或者直接改变配置

```js
// 返回部分配置（推荐）
const partialConfigPlugin = () => ({
  name: 'return-partial',
  config: () => ({
    alias: {
      foo: 'bar'
    }
  })
})

// 直接改变配置（应仅在合并不起作用时使用）
const mutateConfigPlugin = () => ({
  name: 'mutate-config',
  config(config, { command }) {
    if (command === 'build') {
      config.root = __dirname
    }
  }
})
```

### configResolved

在解析了vite配置之后调用，使用这个钩子读取和存储最终的解析配置，当配置需要根据命名行命名做一些不同的事情的时候他很有用

```js
const exmaplePlugin = () => {
  let config

  return {
    name: 'read-config',

    configResolved(resolvedConfig) {
      // 存储最终解析的配置
      config = resolvedConfig
    },

    // 在其他钩子中使用存储的配置
    transform(code, id) {
      if (config.command === 'serve') {
        // dev: 由开发服务器调用的插件
      } else {
        // build: 由 Rollup 调用的插件
      }
    }
  }
}
```



### configureServer

用于配置开发服务器的钩子。常见使用在内部connect应用程序中添加自定义中间件

```js
const myPlugin = () => ({
  name: 'configure-server',
  configureServer(server) {
    server.middlewares.use((req, res, next) => {
      // 自定义请求处理...
    })
  }
})
```

注入后置中间件

configureServer钩子将在内部中间件被安装钱调用，所以自定义中间件将会默认比内部的中间件早运行，。如果你想注入一个之后的钩子，你可以从configureServer返回一个函数

```js
const myPlugin = () => ({
  name: 'configure-server',
  configureServer(server) {
    // 返回一个在内部中间件安装后
    // 被调用的后置钩子
    return () => {
      server.middlewares.use((req, res, next) => {
        // 自定义请求处理...
      })
    }
  }
})
```

存储服务器的访问

某些情况下，其他插件钩子可能需要访问开发服务器实例，可以这样写

```js
const myPlugin = () => {
  let server
  return {
    name: 'configure-server',
    configureServer(_server) {
      server = _server
    },
    transform(code, id) {
      if (server) {
        // 使用 server...
      }
    }
  }
}
```

### transformIndexHtml

转换index.html的专用钩子。钩子接受当前的html字符串和转换上下文。下下文在开发期间暴露ViteDevServer实例，在构建期间暴露rollup输出包

这个钩子可以返回的一下其中之一：

- 经过转换的html字符串
- 注入到现有html标签描述符对象数组
- 一个包含html，tags对象

## handleHotUpdate

执行自定义hmr更新处理，钩子接受一个带有一下签名的上下文对象：

```ts
interface HmrContext {
  file: string
  timestamp: number
  modules: Array<ModuleNode>
  read: () => string | Promise<string>
  server: ViteDevServer
}
```

- modules是收更改文件影响的模块数组。他是一个数组，因为单个文件可能映射到多个服务模块
- read这是一个异步读函数，他返回文件的内容。

钩子可以选择：

- 过滤和缩小受影响的模块列表，是hmr更准确

- 返回一个空数组，并通过向客户端发送自定义事件来执行完成的自定义hmr处理

  ```js
  handleHotUpdate({ server }) {
    server.ws.send({
      type: 'custom',
      event: 'special-update',
      data: {}
    })
    return []
  }
  ```

  客户端代码应该使用hmr api注册响应的处理器

  ```js
  if (import.meta.hot) {
    import.meta.hot.on('special-update', (data) => {
      // 执行自定义更新
    })
  }
  ```

## 插件顺序

一个vite插件可以额外指定一个enforce属性来调用他的应用顺序enforce的值可以是pre或者post。解析后的插件将按照一下顺序排列。

- Alias
- 带有 `enforce: 'pre'` 的用户插件
- Vite 核心插件
- 没有 enforce 值的用户插件
- Vite 构建用的插件
- 带有 `enforce: 'post'` 的用户插件
- Vite 后置构建插件（最小化，manifest，报告）

# hmr api

# js api

vite的js api是完全类型化的



# 使用vite打包一个类库

配置

```js
// vite.config.js
const path = require('path')
const { defineConfig } = require('vite')

module.exports = defineConfig({
  build: {
    lib: {
      entry: path.resolve(__dirname, 'lib/main.js'),
      name: 'MyLib',
      fileName: (format) => `my-lib.${format}.js`
    },
    rollupOptions: {
      // 确保外部化处理那些你不想打包进库的依赖
      external: ['vue'],
      output: {
        // 在 UMD 构建模式下为这些外部化的依赖提供一个全局变量
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```