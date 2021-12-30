# 静态资源打包

## 打包服务启动

rollup如果需要读取配置文件的话，需要你在启动命令进行参数配置才可以使用

`rollup --config my.config.js`否则他不会像其他的打包工具一样，默认在根目录下识别出来

# JavaScript API

这个js api的作用就是，把rollup里面的打包方法，暴露了出来给开发者使用。而不是只能通过一个配置文件，这样子就过于死板了，不够灵活。

例如：

```ts
const rollup = require('rollup');

// see below for details on the options
const inputOptions = {...};
const outputOptions = {...};

async function build() {
  // create a bundle
  const bundle = await rollup.rollup(inputOptions);

  console.log(bundle.imports); // an array of external dependencies
  console.log(bundle.exports); // an array of names exported by the entry point
  console.log(bundle.modules); // an array of module objects

  // generate code and a sourcemap
  const { code, map } = await bundle.generate(outputOptions);

  // or write the bundle to disk
  await bundle.write(outputOptions);
}

build();
```

# 使用插件

rollup插件的作用就是为了扩展rollup的功能。

举一个例子：

```js
import { version } from '../package.json';
// 尝试从package.json文件中获取内容
export default function () {
  console.log('version ' + version);
}
```

```js
import json from 'rollup-plugin-json';
const { resolve } = require('path')

export default {
   input: resolve(__dirname, 'src/main.js'),

   output: {
      file: resolve(__dirname, 'dist/bundle.js'),
      format: 'iife',
      name: 'MyBundle'
   },

   plugins: [json()]
}
```

# import函数使用

import函数是异步导入，当你使用了这个东西，**你的异步模块就会在dist中被拆分出来**

当rollup里面使用了import，配置就必须要指定dir选项，告诉rollup输出目录是什么

# 插件开发

## 简单模板

```js
export default function myPlugins() {
   return {
      name: 'my-example', // this name will show up in warnings and errors
      resolveId ( source ) {
         console.log(11111)
         console.log(source)
         return null; // other ids should be handled as usually
      },
      load ( id ) {
         console.log(2222)
         console.log(id)
         return null; // other ids should be handled as usually
      }
   };
}

// config引入
export default {
	input: resolve(__dirname, 'src/main.js'),

	output: {
		file: resolve(__dirname, 'dist/bundle.js'),
		format: 'iife',
		name: 'MyBundle'
	},

	plugins: [myPlugins()]
}
```

## 属性

### name

给插件命名，方便在执行过程中给出错误提示和警告

### 构建hooks类型

有四种不同的钩子：

- async：异步钩子，它可以返回要给promise
- first：如果有几个插件去实现这个hooks，这些hooks就会串行执行知道一个返回了null或者undefined
- sequential：如果多个插件订阅了这个hooks，那么这些插件就会按照一定的顺序执行，如果有插件是异步的，那么就会等待那个异步插件resolve之后，才会继续执行
- parallel：如果多个插件订阅了这个hooks，那么这些插件就会按照一定的顺序执行，但是遇到了异步的插件，他是不会等待的。

### hooks订阅时机

#### buildEnd

参数访问不到什么资源信息

当打包结束之后触发，在写入文件之前

#### buildStart

参数：inputOptions

执行rollup.rollup的时候，我们可以使用这个hook访问我们的选项，去做一些默认的设置等等

#### closeWatcher

和监听模式相关

#### load

这个钩子比较复杂

参数：id

返回值：难说

这个插件可以让我们定义一些lodaer

```js
load ( id ) {
   const reg = /.vue$/

   if (reg.test(id)) {
      console.log('执行')

      // 你return什么，那个解析的文件就会用这个字符串当作到代码注入bundle
      return `render()`
   }

   return null;
},
```

#### moduleParsed

参数，模块信息

当每一个模块被完全解析的时候`this.getModuleInfo`对象就会被传入rollup中

```js
{
  ast: Node {
    type: 'Program',
    start: 0,
    end: 17,
    body: [ [Node] ],
    sourceType: 'module'
  },
  code: 'console.log(1111)',
  dynamicallyImportedIds: [Getter],
  dynamicImporters: [Getter],
  hasModuleSideEffects: true,
  id: 'D:\\study\\rollup\\src\\main.js',
  implicitlyLoadedAfterOneOf: [Getter],
  implicitlyLoadedBefore: [Getter],
  importedIds: [Getter],
  importers: [Getter],
  isEntry: true,
  isExternal: false,
  meta: {},
  syntheticNamedExports: false
}
```

注意：在这个钩子中，你还没能够访问访问import信息，

#### options

这个第一个触发的钩子

#### resolveDynamicImport

参数：specifier，你传入import的字符串参数，importer：导入他的模块

触发的条件当且仅当你在模块里面使用了Import函数

用来定义自定义的解决方案为这个`import()`的模块，返回false这个import就会保持原来的不会被传递到别的resolvers中，因为会被标记为外部模块。你也可以return一个对象，导出一个不同的id同事标记为一个外部模块

#### resolveId

参数：source，你import的那个路径

返回值：他的返回值可以影响load阶段获取到的id。**返回false意思就是，rollup就会把这个引入的模块当作第三方模块不会打包进入bundle里面**

![image-20211209202925009](C:\Users\freddieyu\AppData\Roaming\Typora\typora-user-images\image-20211209202925009.png)

这个钩子可以知道那些被解析了的模块的路径，**同时也包含入口文件**，对于入口文件来说他的importer是undefined

如果你返回一个对象，你就可以改变模块的id，同时也可以把他当作第三方的模块

```js
resolveId(source) {
  if (source === 'my-dependency') {
    return {id: 'my-dependency-develop', external: true};
  }
  return null;
}
```

定义一个自定义的解析器，这个解析器对于定位那些第三方的依赖很有用

```js
// 入口文件proxy例子
async resolveId(source,importer) {
  if (!importer) {
    // We need to skip this plugin to avoid an infinite loop
    const resolution = await this.resolve(source, undefined, { skipSelf: true });
    // If it cannot be resolved, return `null` so that Rollup displays an error
    if (!resolution) return null;
    return `${resolution.id}?entry-proxy`;
  }
  return null;
},
load(id) {
  if (id.endsWith('?entry-proxy')) {
    const importee = id.slice(0, -'?entry-proxy'.length);
    // Note that this will throw if there is no default export
    return `export {default} from '${importee}';`;
  }
  return null;
}
```

#### transform

参数：code，id

这个能够用来转换一些独立的模块，为了避免一些额外的解析开销。这个钩子已经使用了this.parse去产生了ast，这个钩子可以选择性返回`{ code, ast, map }`对象

### 输出的钩子

#### aumentChunkHash

参数：chunkInfo，chunk的信息

经过了前面的打包阶段，将会生成一个chunk，这个hook用来给chunk增加hash，返回false值则不修改，返回字符串，则会用这个字符串修改这个模块名称

```
augmentChunkHash(chunkInfo) {
  if(chunkInfo.name === 'foo') {
    return Date.now().toString();
  }
}
```

这个钩子似乎调用不了，不知道为什么。

#### closeBundle

它用来中止一些别的服务可能正在运行的，你也可以使用`bundle.close()`来中止

#### footer

#### generateBundle

参数：outputOptions，bundle提供被打包的资源的详细信息（指的是输出文件，并不是被打包的资源）

在`bundle.generate()`结束的时候被调用，在`bundle.write`方法之前。去修改这个文件在他被写入之后，你就要使用writeBundle钩子

## 插件上下文

有很多的实用的一些函数给我们通过this调用，这些函数都是为插件所服务的，在插件中通过this，就可以使用这些函数

### emitFile

`this.emitFile(emittedFile: EmittedChunk | EmittedAsset) => string`

这个函数可以输出一些文件的，

它可以输出两种类型的文件：

- 输出一个chunk
- 输出一个asset

```js
// EmittedChunk
{
  type: 'chunk',
  id: string,
  name?: string,
  fileName?: string,
  implicitlyLoadedAfterOneOf?: string[],
  importer?: string,
  preserveSignature?: 'strict' | 'allow-extension' | 'exports-only' | false,
}

// EmittedAsset
{
  type: 'asset',
  name?: string,
  fileName?: string,
  source?: string | Uint8Array
}
```

两种情况下都需要提供一个name或者是filename。如果两个都不提供，那么将会有一个默认的名字。如果有了filename，他将会直接用来作为输出文件的名字，不会做任何的改动。当然如果有了名字冲突的话就会抛出一个异常。如果有了name，那么这个名字将可以根据output.chunkFileNames或output.assetFileNames配置项进行模式的替换，这样就不会有名字冲突。

你可以通过`import.meta.ROLLUP_FILE_URL_referenceId`引用一个输出文件的，这个输出文件要求return通过load或者transform插件，url

type是chunk，然后将会输出一个新的chunk带有给定的一个module id作为入口文件。

importer选项作用：

默认情况下，rollup假定输出的chunk会被作为其他入口单独执行，甚至可能在其他代码之前执行。这意味着如果这个输出的chunk和一个已有文件共享有着相同的依赖，rollup将会为这个共享的快创建一个额外的chunk（意思就是webpack减少包的体积一个意思）。提供一个存放模块id的非空数组`implicitlyLoadedAfterOneOf`，这个将会改变rollup上面这个提取公公块的行为·.这些id将会被解析就像解析id一样，如果impoter属性被提供，rollup将会遵循这属性。rollup将会现在假设输出的chunk仅仅会在至少一个入口文件（就是配置了id）被加载之后才会执行，所以你引入的chunks，我也可以从implicitlyLoadedAfterOneOf的entry导出给你去使用（这个是一个优化方案）

```js
// rollup.config.js
function generateHtml() {
  let ref1, ref2, ref3;
  return {
    buildStart() {
      ref1 = this.emitFile({
        type: 'chunk',
        id: 'src/entry1'
      });
      ref2 = this.emitFile({
        type: 'chunk',
        id: 'src/entry2',
        implicitlyLoadedAfterOneOf: ['src/entry1']
      });
      ref3 = this.emitFile({
        type: 'chunk',
        id: 'src/entry3',
        implicitlyLoadedAfterOneOf: ['src/entry2']
      });
    },
    generateBundle() {
      this.emitFile({type: 'asset', fileName: 'index.html', source: `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="UTF-8">
        <title>Title</title>
       </head>
      <body>
        <script src="${this.getFileName(ref1)}" type="module"></script>
        <script src="${this.getFileName(ref2)}" type="module"></script>
        <script src="${this.getFileName(ref3)}" type="module"></script>
      </body>
      </html>
      `})
    }
  };
}

export default {
  input: [],
  preserveEntrySignatures: false,
  plugins: [
    generateHtml(),
  ],
  output: {
    format: 'es',
    dir: 'dist'
  }
};
```

如果没有动态的import，这个将会产生三个chunks，第一个chunk包含src/entry1所有的依赖，第二个chunk包含仅仅src/entry2的依赖，他不会包含第一个chunk的依赖，但是他会importing那些依赖从第一个chunk中。对于第三个chunk也是一样的。

看看效果

```js
// entry1
import Vue from "./index";
console.log(1111)

console.log(Vue)

// entry2
import Vue from "./index";
console.log(2)

console.log(Vue)

//注意 1 2入口同时都以来了一个Vue模块，假如没有配置entry2implicitlyLoadedAfterOneOf: ['src/entry1']选项
//那么输出文件就是直接引入Vue，这样子js的文件就会变大
// entry2输出样子
import { V as Vue } from './index-565006b4.js';

console.log(2);

console.log(Vue);

// 如果配置了选项，输出就是这样的，通过从entry1获取得到
// entry2输出
import { V as Vue } from './entry1-79a24580.js';

console.log(2);

console.log(Vue);

// entry1输出
const Vue = 123;

console.log(1111);

console.log(Vue);
// 他会导出vue给模块2使用
export { Vue as V };
```

这个好处就是可以减少包的体积。

注意，机关任何的module id都能够使用在implicitlyLoadedAfterOneOf，rollup将会抛出一个异常，如果一个id不是单独关联一个chunk，



如果你的type是asset，然后

### error

参数扽沟通与this.warn，除了这个钩子将会中止掉你的打包进程

### getCombinedSourcemap

获得一个合并的资源图，这个插件仅仅在tranform钩子使用

### getFilename

获得通过emitFile输出文件的名字

### getModuleInfo

返回一些额外的module信息

### parse

翻译字符串代码为ast

```js
export default function myPlugins() {
   return {
      transform(code) {
         console.log(this.parse(code))
      }
   };
}
```

### resolve

参数：source，importer，options

使用相同的rollup插件解析imports的module ids。决定是否一个import应该被external。如果一个null被返回了，这个import将会不会被解析通过rollup或者任何插件但是不会被开发者明确的标记为external。如果一个明确的external模块被返回，你需要在makeAbsoluteExternalsRelative在这里配置一下

## File URLS

### asserts

要从js代码中引用文件url，请使用`import.meta.ROLLUP_FILE_URL_referenceId` 代替。这将会输出一些代码，这个代码取决于你输出的格式（commonjs这些），同时产生一个url指向输出的文件在目标环境中

注意，所有的格式除了commonjs和umd，都会被假设在浏览器环境中运行。所以document，url对象都是可用的。

举一个例子：这个例子是检测导入了svg的文件，然后把这个svg作为资源文件输出，然后返回他的url，作为img的src属性使用

```js
export default function myPlugins() {
   let ref1, ref2, ref3;

   return {
      name: 'my-example', // this name will show up in warnings and errors
      resolveId(source, importer) {
         if (source.endsWith('.png')) {
            console.log(path.resolve(path.dirname(importer), source))
            return path.resolve(path.dirname(importer), source)
         }
      },
      load(id) {
         if (id.endsWith('.png')) {
            console.log('load钩子·')
            console.log(id)
            console.log(path.basename(id))
            const referenId = this.emitFile({
               type: 'asset',
               name: path.basename(id),
               source: fs.readFileSync(id)
            })

            console.log(referenId)

            return `export default import.meta.ROLLUP_FILE_URL_${referenId};`;
         }
      }

   };
}
```

使用：

```js
import logo from '../images/logo.svg';
const image = document.createElement('img');
image.src = logo;
document.body.appendChild(image);
```

这个代码的作用就是，从你的入口文件中读取你使用的png文件，然后通过`emitFile`输出到dist中。

`import.meta.ROLLUP_FILE_URL_referenceId`这个东西的作用，我觉得就是让rollup正确识别到你的引用路径，看下输出的js是怎么样的。

```js
(function () {
   'use strict';

    // import.meta.ROLLUP_FILE_URL_referenceId，就是为了生成这个东西
   var logo = new URL('assets/1-9f1f0fce.png', document.currentScript && document.currentScript.src || document.baseURI).href;

   const image = document.createElement('img');
   image.src = logo;
   document.body.appendChild(image);

})();
```

### chunk

类似assets文件，要想正确的引用chunk，同样也需要`import.meta.ROLLUP_FILE_URL_referenceId`这个东西。

举一个例子，这个例子的作用就是检测前缀是register-paint-worklet:，然后产生一些代码，同时分割chunk去产生css

