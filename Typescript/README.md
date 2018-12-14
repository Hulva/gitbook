# Typescript

## 编译上下文

编译上下文是一个比较花哨的术语，它用来给文件分组，告诉TypeScript哪些文件是有效的，哪些是无效的。除了有效文件所携带的信息外，编译上下文还包含了有哪些编译选项正在使用。

### tsconfig.json

#### 编译选项

##### **compilerOptions**

```json
{
  "compilerOptions": {
    /* 基本选项 */
    "target": "es5",                       // 指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'
    "module": "commonjs",                  // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    "lib": [],                             // 指定要包含在编译中的库文件
    "allowJs": true,                       // 允许编译 javascript 文件
    "checkJs": true,                       // 报告 javascript 文件中的错误
    "jsx": "preserve",                     // 指定 jsx 代码的生成: 'preserve', 'react-native', or 'react'
    "declaration": true,                   // 生成相应的 '.d.ts' 文件
    "sourceMap": true,                     // 生成相应的 '.map' 文件
    "outFile": "./",                       // 将输出文件合并为一个文件
    "outDir": "./",                        // 指定输出目录
    "rootDir": "./",                       // 用来控制输出目录结构 --outDir.
    "removeComments": true,                // 删除编译后的所有的注释
    "noEmit": true,                        // 不生成输出文件
    "importHelpers": true,                 // 从 tslib 导入辅助工具函数
    "isolatedModules": true,               // 将每个文件做为单独的模块 （与 'ts.transpileModule' 类似）.

    /* 严格的类型检查选项 */
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 在表达式和声明上有隐含的 any类型时报错
    "strictNullChecks": true,              // 启用严格的 null 检查
    "noImplicitThis": true,                // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */
    "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误
    "noFallthroughCasesInSwitch": true,    // 报告switch语句的fallthrough错误。（即，不允许switch的case语句贯穿）

    /* 模块解析选项 */
    "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    "typeRoots": [],                       // 包含类型声明的文件列表
    "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,  // 允许从没有设置默认导出的模块中默认导入。

    /* Source Map Options */
    "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性

    /* 其他选项 */
    "experimentalDecorators": true,        // 启用装饰器
    "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
  }
}
```

## 声明空间

类型声明空间 和 变量声明空间


## 模块

### 全局模块

```const foo = 123;```

危险 推荐使用文件模块

### 文件模块

也被称为外部模块。如果你在TypeScript文件的根级别位置含有 import 或 export ，它会在这个文件中创建一个本地的作用域。

```export const foo = 123;```

这样在全局命名空间里，我们不再有 foo ， 这可以通过创建一个新文件 bar.ts 来证明：

```const bar = foo; // ERROR: "cannot find name 'foo'" ```

如果您想要在 bar.ts 里使用来自 foo.bar 的内容，你必须显示导入它：

```
imort { foo } from './foo';
const bar = foo; // fine
```

在 bar.ts 文件里使用 import ， 不但允许你使用从其他文件中导入的内容，而且它会将此文件 bar.ts 标记为一个模块，文件内定义的声明也不会污染全局命名空间。

#### 文件模块详情

文件模块拥有强大的能力和可用性。

> NOTE: commonjs / amd / es modules / others

* AMD：不要使用它，仅能在浏览器工作
* SystemJS：这是一个好的实验，已经被ES模块替代
* ES 模块：它并没有准备好

使用 module：commonjs 选项来替代这些模式

### ES 模块语法

使用 exeport 关键字导出一个变量(或类型)

```
export const someVar = 123;
export type someType = {
    foo: string;
};
```