# 配置文件类型

有三种配置文件类型：

- js
- yml
- json

# ws eslint插件无效问题

有一些高版本的eslint在ws下可能运行不了，那你降低一下版本吧

## 层级bug

不知道为什么理论上eslint的作用规则是逐级合并，但是似乎ws并不是以这个规范来的，他似乎都是以根目录下的配置文件为规范，eslint命令行校验没错，但是ws工具提示是错误的

# 作用范围

假设我们有这个目录

```js
your-project
├── .eslintrc.json
├── lib
│ └── source.js
└─┬ tests
  ├── .eslintrc.json
  └── test.js
```

如果配置文件与被linted的文件在哦同一个目录中，则该配置优先，然后eslint搜索目录结构，**合并**他沿途找到的所有配置文件，直到有一个配置文件配置了`root:true`

如果`package.json`根目录下有一个eslintConfig字段的文件，他所描述的配置将应用到他下面所有的子目录，但是目录中描述的配置tests/会在有冲突的规范地方覆盖他

如果同一个目录下有配置文件和package.json文件，配置文件优先package.json会被忽略

默认情况下：**eslint将在所有的父文件夹中查找配置文件，直到根目录**，如果你希望项目所有的地方都遵守某个规范约定，这个将会十分有用。如果说你希望eslint限制为特定的某个目录下的就设置root选项

# 扩展配置文件

extends属性

一定使用了扩展，一个配置文件就能够继承另外配置文件的所有特性，存在三种配置：

- 基本配置：扩展的配置
- 派生配置：扩展基本配置的配置
- 生成实际配置：将派生配置合并到基本配置中的结果



extends属性的属性值可以是：

- 指定配置文件字符串（路径，配置文件名称或eslint：all）
- 一个字符串数组，其中每个附加配置都扩展了前面的配置



## 重写覆盖规则

rule属性可以执行一下任何操作来扩展或者覆盖规则：

- 改变一个继承的规则无需改变他的options：
  - 基本配置假设是这个`eqe:['error', 'sllow-null']`
  - 你这样鞋`eqe:"warn`，这种就是覆盖部分
  - 实际的配置变成了`eqe:['warn', 'sllow-null']`
- 直接重写基本配置
  - 假设基本配置如下：`"quotes": ["error", "single", "avoid-escape"]`
  - 你的配置：`"quotes": ["error", "single"]`
  - 实际配置就是这个：`"quotes": ["error", "single"]`
- 重写一些是对象的rules
  - 假设基本配置：`"max-lines": ["error", { "max": 200, "skipBlankLines": true, "skipComments": true }]`
  - 你的配置：`"max-lines": ["error", { "max": 100 }]`
  - 实际配置：`"max-lines": ["error", { "max": 100 }]`

## 使用共享配置文件

extends配置是忽略那些eslint包的 `eslint-config-`前缀的，加入你的eslint包叫做`eslint-config-standard`，那么你使用他的时候直接就可以写：

```js
{
	extends: standard
}
```

## 使用eslint推荐选项

```js
"extends": "eslint:recommended"
```

当你使用了他之后，在官方上一些打勾的特性就会被使用上，自行查阅官网。

## 使用插件的配置

同理，插件也是忽略`eslint-plugin-`前缀的。

使用插件的方法：

```js
{
    "plugins": [
        "react"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "rules": {
       "react/no-set-state": "off"
    }
}
```

plugins：数组，直接写插件名称省略前缀

extends：名字组成，plugin: + 插件包名称 + ‘/’ + 配置名称

例如： `"plugin:react/recommended"`

你也可以直接extends别人的配置文件

```js
{
    "extends": [
        "./node_modules/coding-standard/eslintDefaults.js",
        "./node_modules/coding-standard/.eslintrc-es6",
        "./node_modules/coding-standard/.eslintrc-jsx"
    ],
    "rules": {
        "eqeqeq": "warn"
    }
}
```

## eslint:all

extends的值可以是这个，这个选项可以开启当前eslint版本所有的核心rule，这个rule可能在不同的eslint大小版本中改变，所有不太推荐你使用这个选项

## 基于全局的配置

有时一个更加精细的配置是必要的，举个例子，假如你的配置在同一个目录的文件配置必须不同。因此，你可以在overrodes仅适用于匹配特定模式的文件提供配置，举个例子：假设有如下文件

```js
{
  "rules": {
    "quotes": ["error", "double"]
  },

  "overrides": [
    {
      "files": ["bin/*.js", "lib/*.js"],
      "excludedFiles": "*.test.js",
      "rules": {
        "quotes": ["error", "single"]
      }
    }
  ]
}
```

这个的含义就是说，files字段所在的文件，都会使用`overrides`下这条rules进行重写

**这种全局模式的优先级是最高的**

# add share setting

插件使用了settings选项去表明一些信息，这些信息可以被所有他的规则共享，这个东西用在插件的时候特别的好用

```js
{
    "settings": {
        "sharedData": "Hello"
    }
}
```

# 语言选项

## 特殊的环境

一个环境提供了一个预定义的变量，环境设置对了，你在使用一些全局变量二点时候，eslint就不会给你抛出异常

## 配置方法

```js
{
    "env": {
        "browser": true,
        "node": true
    }
}
```

或者在package.json中

```js
{
    "name": "mypackage",
    "version": "0.0.1",
    "eslintConfig": {
        "env": {
            "browser": true,
            "node": true
        }
    }
}
```

## 使用插件

```js
{
    "plugins": ["example"],
    "env": {
        "example/custom": true
    }
}
```

## 特殊的全局变量

ESLint 的一些核心规则依赖于对代码在运行时可用的全局变量的了解。由于这些在不同环境之间可能会有很大差异，并且在运行时会进行修改，因此 ESLint 不会假设您的执行环境中存在哪些全局变量。如果您想使用需要了解哪些全局变量可用的规则，您可以在配置文件中或通过在源代码中使用配置注释来定义全局变量。

使用方法:

```js
{
    "globals": {
        "var1": "writable",
        "var2": "readonly"
    }
}
```

## 指定解析器选项

eslint允许你指定要支持的js语言选项。默认情况下eslint使用es5语法，**意味着你不能使用let const**

如何让eslint支持es6语法。

`"parserOptions": { "ecmaVersion": 6 } }`

ecmaVersion：指定语言版本

sourceType：script或者module。您的代码是否在 ECMAScript 中

- ```
  ecmaFeatures
  ```

   \- 一个对象，指示您要使用哪些附加语言功能：

  - `globalReturn`- 允许`return`全局范围内的语句
  - `impliedStrict`- 启用全局[严格模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)（如果`ecmaVersion`是 5 或更大）
  - `jsx`- 启用[JSX](https://facebook.github.io/jsx/)



# 命令行

使用以下格式来运行eslint

`eslint xxxx`运行完之后eslint就会检查你这个文件是否符合配置的规范要求

## 常用的命令行参数

--no-eslintrc，禁用配置文件和package.json的配置

```js
eslint --no-eslintrc file.js
```

-c,-config。为eslint指定另外的配置文件

```js
eslint -c ~/my-eslint.json file.js
```

--env，用来指定环境和配置文件中的是一个功能

--ext，允许指定eslint在指定目录下查找js文件时要使用的扩展文件名。

```js
# Use both .js and .js2
eslint . --ext .js --ext .js2
```

--global，指定全局变量。**当你开启了rule：no-undefine的时候**，对你某人环境没有的全局变量eslint是会给出错误的，这个时候就可以使用global进行指定

```js
eslint --global require,exports:true file.js
```

## 特殊规则和插件

--rulesdir，这个选项允许你指定另一个加载规则文件目录

--plugin，这个选项指定一个要加载的插件

## fix问题

--fix只是eslint试图修复尽可能多的问题。

--fiex-dry-run，这个选项和fix有相同的效果，唯一不同的是，**修复不会报错到文件中。这个是只是在控制台修复问题**

## 处理警告

--quiet，这个选项允许你禁止报告警告，开启这选项，eslint只会报告错误

--max-warnings，这个选项允许你指定一个警告的阈值，当项目有太多的警告，这个阈值就可以用来强制eslint以错误状态退出。

## output

-o或者--output-file，将报告写到一个文件中

# rules

## error等级

- off或者0，关闭这个规则
- warn或者1，开启警告
- error或者2，抛出异常

## 常见有用的一些规则

- getter-return 强制 getter 函数中出现 `return` 语句
- no-cond-assign 禁止条件表达式中出现赋值操作符
- no-console 禁用console `"no-console": 2`
- no-debugger，禁止使用debugger
- no-func-assign，禁止对函数重新赋值
- default-case，禁止switch预计有default分支
- eqeqeq，强制使用===
- camelcase 强制使用驼峰命名
- indent，强制使用一直缩进，`"indent": ["error", 2]`，两缩进
- jsx-quotes，强制jsx属性中一致地使用双引号或者单引号
- no-whitespace-before-property：禁止属性前有空白
- constructor-super，要求构造函数中有super调用
- no-var，要求使用let，const而不是var





# formatters

**这个选项的作用就是，控制eslint对文件进行lint之后，输出的报告格式**，感觉用不上

eslint带有几个内置的格式化程序来控制linting的结果外观，并且也支持第三方格式化程序

你命令行使用--format，-f来指定格式化程序，例如--format codeframe

内置的一些选项有：

- checkstyle
- codeframe
- compact
- .....

举个例子：





# 创建自定义规则

每一个规则都会创建对应名字的js文件，例如`no-extra-semi`，那么你创建的js文件就叫做`no-extra-semi.js`

一个规则的基本格式：

```js
module.exports = {
    meta: {
        type: "suggestion",

        docs: {
            description: "disallow unnecessary semicolons",
            category: "Possible Errors",
            recommended: true,
            url: "https://eslint.org/docs/rules/no-extra-semi"
        },
        fixable: "code",
        schema: [] // no options
    },
    create: function(context) {
        return {
            // callback functions
        };
    }
};
```

一个规则源文件输出一个对象，包含以下属性：

- meta包含规则的元数据
  - type指示规则类型，值为”problem“，”suggestion“，”layout“
  - problem，指这个规则的要不会导致错误或者产生令人歧义的行文，需要开发者优先解决
  - suggestion，说明这个东西有更好的方式解决
  - layout，该规则关心空格，分号等。这些影响程序外观的而不是影响执行的
  - docs，和文档描述有关
  - fixable，没有fixable属性，eslint也不会去修复这个错误，即使说它实现了fix功能，如果规则不可修复，就忽略fixable属性
  - 
- create，返回一个对象，其中包含了eslint在遍历js代码的ast时用来访问节点的方法
  - 如果key是一个节点类型或者选择器，在向下遍历树时，eslint调用vistor函数
  - 如果key是一个节点类型或者选择器，并带有:exit，再向上遍历树时，eslint调用vistor函数
  - 如果key是一个事件名字，eslint为代码路径分析调用handler函数
  - 一个规则可以使用当前节点和它周围的树，报告或者修复问题

举个例子：

```js
function checkLastSegment (node) {
    // report problem for function if last code path segment is reachable
}

module.exports = {
    meta: { ... },
    create: function(context) {
        // declare the state of the rule
        return {
            ReturnStatement: function(node) {
                // at a ReturnStatement node while going down
            },
            // at a function expression node while going up:
            "FunctionExpression:exit": checkLastSegment,
            "ArrowFunctionExpression:exit": checkLastSegment,
            onCodePathStart: function (codePath, node) {
                // at the start of analyzing a code path
            },
            onCodePathEnd: function(codePath, node) {
                // at the end of analyzing a code path
            }
        };
    }
};
```

## create函数key

这些key是一些语法规则，具体有很多，可以查询https://github.com/estree/estree/blob/master/es2015.md文档你就指导有什么key了

## selector

一个selector是一个字符串能够用来匹配ast节点的，这个有利于你描述一个特殊的规则在你的代码中。

这个ast的selector规则十分类似于css的规则。

最简单的selector就是一个node类型，一个node类型将会匹配所有的nodes符合要求的举个例子。

```js
var foo = 1;
bar.baz();
```

选择器Identifier就会匹配所有的Identifier节点节点，在这个例子中，这个选择器就会匹配所有的foo，bar和baz。这个选择器是遇到一个执行，一次不是说node参数就是给你一个数组

```js
create: function(context) {
  return {
    Identifier(node) {
      if (node.name === 'a') {
        context.report({
          node,
          messageId: 'invalidName'
        })
      }
      console.log(node)
    }
  };
}
```

初次之外还有其他规则：

- forStatement，



## context对象

contxt对象包含了eslint解析过程很多的上下文信息

- parserOptions，解析器选项
- id，规则id
- options，此规则的已配置选项数组
- settings，配置中的共享设置
- parsePath，配置中parser名称
- parserServices，
- getAncestors，返回当前遍历节点的祖先数组，从ast的根开始，不包含当前遍历节点
- getDeclaredVariables(node)，返回由给定节点声明的变量列表，用于跟踪对变量引用
  - 如果节点时VariableDeclaration，则返回声明中声明的所有变量
  - 如果节点是一个VariableDeclaration，则返回declarator中声明的所有变量
  - 如果节点是一个FunctionDeclaration` 或 `FunctionExpression，除了返回函数参数变量外，还返回函数名变量
  - 如果是一个ArrowFunctionExpression，返回参数变量
  - 如果节点是ClassDeclaration` 或 `ClassExpression，返回类名变量
  - 如果是CatchClause，则返回异常变量
  - 如果时ImportDeclaration。则返回其所有说明符变量
- getFilename()，返回与源文件关联的文件名
- getScope()，返回当前遍历节点的scope，此信息用于跟踪对变量引用
- getSourceCode()，返回soucecode对象，你可以使用该对象处理传递给eslint源代码
- markVariableAsUsed(name)，在当前作用域内用给定的名称标记一个变量
- report(descriptor)，报告问题代码

## getScope返回的作用域类型

| AST Node Type             | Scope Type |
| :------------------------ | :--------- |
| `Program`                 | `global`   |
| `FunctionDeclaration`     | `function` |
| `FunctionExpression`      | `function` |
| `ArrowFunctionExpression` | `function` |
| `ClassDeclaration`        | `class`    |
| `ClassExpression`         | `class`    |
| `BlockStatement` ※1       | `block`    |
| `SwitchStatement` ※1      | `switch`   |
| `ForStatement` ※2         | `for`      |
| `ForInStatement` ※2       | `for`      |
| `ForOfStatement` ※2       | `for`      |
| `WithStatement`           | `with`     |
| `CatchClause`             | `catch`    |

## report

这个用来当eslint错误的时候，给开发者的提醒，该方法接受一个参数对象，包含以下属性：

- message，有问题的消息
- node（可选），与问题有关的ast节点，如果存在且没有指定loc，那么该节点的开始位置用来作为问题的位置
- loc，用来指定问题位置的一个对象。如果同时指定的了loc和node，那么位置从loc获取而不是node
  - start，开始位置
    - line，问题发生的行号，
    - column，问题发生的列号
  - end，结束位置
    - line，问题发生的列好
    - column，问题发生的列号
- data，message的占位符
- fix，一个用来解决问题的修复函数

node和loc至少一个是必须的

```js
context.report({
    node: node,
    message: "Unexpected identifier"
});
```

消息中提供给占位符，就是你的消息带有变量的

```js
context.report({
    node: node,
    message: "Unexpected identifier: {{ identifier }}",
    data: {
        identifier: node.name
    }
});
```

## fix函数

如果你想让eslint尝试修复你所报告的问题，你可以使用report的fix函数

```js
context.report({
    node: node,
    message: "Missing semicolon",
    fix: function(fixer) {
        return fixer.insertTextAfter(node, ";");
    }
});
```

fixer对象的一些方法：

- insertTextAfter(nodeOrToken, text)，在给定的节点或记号之后插入文本
- insertTextAfterRange(range, text)，在给定的范围之后插入文本
- insertTextBefore(nodeOrToken, text)，之前，同上
- insertTextBeforeRange(range, text)，之前，同上
- remove(nodeOrToken)，删除给定节点或记号
- removeRange(range)，删除给定范围内文本
- replaceText(nodeOrToken, text)，替换
- replaceTextRange(range, text)。替换

## context.options

这个options就是我们的配置文件，这个就可以让你实现当你存在某个配置的时候存取修复

```js
module.exports = {
    create: function(context) {
        var isDouble = (context.options[0] === "double");

        // ...
    }
};
```

## context.getSourceCode

sourceCode是获取被检查源码的更多信息的主要对象

```js
var sourceCode = context.getSourceCode();
```

sourceCode实例还有很多的方法：

- getText(node)，返回给定节点的源码
- getAllComments()，返回一个包含源中所有的注释数组
- getCommentsBefore(nodeOrToken)，返回节点之前的注释数组
- getCommentsAfter(nodeOrToken)，返回节点之后的注释数组
- getCommentsInside(node)，返回节点内的注释数组
- isSpaceBetweenTokens(first, second)，如果两个记号之前有空白，返回true
- getFirstToken(node, skipOptions)，返回给定节点第一个token
- getFirstTokens(node, countOptions)，返回代表节点的第一个count数量token
- getLastToken(node, skipOptions)，最后，同上
- getLastTokens(node, countOptions)，最后，同上
- ......还有很多类似的方法
- getTokens(node)，返回给定节点所有的token
- getTokensBetween(nodeOrToken1, nodeOrToken2)，返回两个节点间的token
- getTokenByRangeStart(index, rangeOptions），返回园中范围从给定索引开始的token
- getNodeByRangeIndex(index)，返回ast中包含给定的源的索引最深节点
- commentsExistBetween(nodeOrToken1, nodeOrToken2)，如果两个节点中间存在注释返回true

# 自定义解析器

开发方式，拿出一个文件，导出一个parseForESLint方法：

```js
var espree = require("espree");
// awesome-custom-parser.js
exports.parseForESLint = function(code, options) {
    return {};
};
```

这个方法将会在解析代码的时候被调用。这个方法的返回值应该返回一个ast，这个ast是有要求的，必须包含一些属性ast还有一些可选的属性services`, `scopeManager`, and `visitorKeys

- ast，应该包含一个ast
- services，应该包含任何依赖于解析器的服务（类如节点检查其）。这个services值可以哦那个与规则为context.parserServices
- scopeManager，可以是Scopemanger对象，自定义解析器可以对增加语法使用自定义范围解析
- visitorKeys，可以是自定义个ast遍历的对象，对象的键是ast节点

```js
{
    "parser": "./path/to/awesome-custom-parser.js"
}

var espree = require("espree");
// awesome-custom-parser.js
exports.parseForESLint = function(code, options) {
    return {
        ast: espree.parse(code, options),
        services: {
            foo: function() {
                console.log("foo");
            }
        },
        scopeManager: null,
        visitorKeys: null
    };
};
```



## ast规范

### 所有节点

自定义解析的器创建的ast是基于estree的。

- 所有节点都必须具有range属性，number[]类型，它是一个由两个数字组成的数组，这两个数字都是从0开始的索引，他是源代码字符串数组的位置。第一个是节点的开始位置，第二个是节点的结束位置
- loc，

### program节点

这个programe节点必须有tokens和comments属性

token对象：

```ts
interface Token {
    type: string;
    loc: SourceLocation;
    range: [number, number]; // See "All nodes:" section for details of `range` property.
    value: string;
}
```

token，是一个token类型数组，将会影响节点的行为。任何的空格可以存在与两个token中，所以rules检查token$rangge来检查令牌之前的空格，这必须按排序

comments，是注释标记数组



# 插件开发

每个插件是一个命名格式为`eslint-plugin-<plugin-name>`的npm包。

可以用一个yeomna generator。他是一个cli可以帮助你快速完成插件框架的配置

插件可以暴露额外的规则以供使用，插件必须输出一个rules对象，包含规则id和对应的键值对。例如：

```js
module.exports = {
    rules: {
        "dollar-sign": {
            create: function (context) {
                // rule implementation ...
            }
        }
    }
};
```

这个规则id，因该就是用于

```js
{
    "plugins": ["example"],
    "env": {
    	// 用在这里的
        "example/custom": true
    }
}
```

插件可以暴露额外的环境以在eslint中使用，插件必须输出一个 `environments` 对象。`environments` 对象的 key 是不同环境提供的名字，值是不同环境的设置。例如：

```js
module.exports = {
    environments: {
        jquery: {
            globals: {
                $: false
            }
        }
    }
};
```

你也可以的告诉插件如何处理js以外的文件

```js
module.exports = {
    processors: {
        ".ext": {
            preprocess: function(text, filename) {

                return [string];  // return an array of strings to lint
            },

            postprocess: function(messages, filename) {
                return messages[0];
            },

            supportsAutofix: true // (optional, defaults to false)
        }
    }
};
```

## config in plugins

你可以在插件中congfigs键下指定打包的配置。当你想提供不仅仅是代码风格修改，还希望提供一些自定义规则的时候，他就比较有用，同时每个插件都可以指定多个配置，用户想使用你的风格的使用它只需要指定一下就好了

```js
module.exports = {
    configs: {
        myConfig: {
            plugins: ["myPlugin"],
            env: ["browser"],
            rules: {
                semi: "error",
                "myPlugin/my-rule": "error",
                "eslint-plugin-myPlugin/another-rule": "error"
            }
        },
        myOtherConfig: {
            plugins: ["myPlugin"],
            env: ["node"],
            rules: {
                "myPlugin/my-rule": "off",
                "eslint-plugin-myPlugin/another-rule": "off"
                "eslint-plugin-myPlugin/yet-another-rule": "error"
            }
        }
    }
};
// 使用
{
    "extends": ["plugin:myPlugin/myConfig"]
}
```