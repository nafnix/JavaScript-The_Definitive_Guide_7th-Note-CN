# 模块

实践中，模块化的主要作用体现在封装和隐藏私有实现细节，以及保证全局命名空间清洁上。

## 基于类、对象和闭包的模块

可以使用立即调用的函数表达式来实现某种模块化，把实现细节和辅助函数隐藏在包装函数中，只把模块的公共API作为函数的值返回。

例如：

```js
const stats = (function(){
    const sum = (x, y) => x + y;
    const square = x => x * x;
    
    function mean(data) {
        return data.reduce(sum)/data.length;
    }
    
    function stddev(data) {
        let m = mean(data);
        return Math.sqrt(data.map(x => x - m).map(square).reduce(sum)/(data.length-1));
    }
    
    return {mean, stddev};
}());

stats.mean([1,3,5,7,9]);		// 5
stats.stddev([1,3,5,7,9]);		// Math.sqrt(10)
```

### 基于闭包的自动模块化

可以想象有个工具，它能解析代码文件，把每个文件的内容包装在一个立即调用的函数表达式中，还可以跟踪每个函数的返回值，并将所有内容拼接为一个大文件。

## Node中的模块

Node模块使用`require()`函数导入其他模块，通过设置`Exports`对象的属性或完全替换`module.exports`对象来导出公共API。

### Node的导出

Node定义了一个全局`exports`对象，这个对象始终有定义。若要写一个导出多个值的Node模块，可以直接把这些值设置为`exports`对象的属性：

```js
const sum = (x, y) => x + y;
const square = x => x * x;

exports.mean = data => data.reduce(sum)/data.length
exports.stddev = function(d) {
    let m = exports.mean(d);
    return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length-1));
};
```

更多时候只想导出一个函数或类，这时只要把想导出的值直接赋给`module.exports`即可：

```js
module.exports = class BitSet extends AbstracWritableSet {
    // 省略实现
};
```

对于`mean`和`stddev`实际上也可以：

```js
const sum = (x, y) => x + y;
const square = x => x * x;

mean = data => data.reduce(sum)/data.length
stddev = function(d) {
    let m = exports.mean(d);
    return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length-1));
};

module.exports = { mean, stddev };
```

### Node的导入

Node模块通过调用`require()`函数导入其他模块。该函数的参数是要导入模块的名字，返回值是该模块导出的值。

导入Node内置的系统模块或通过包管理器安装在系统上的模块，可以使用模块的非限定名：

```js
const fs = require("fs");		// 内置的文件系统模块
const http = require("http");	// 内置的HTTP模块

// Express HTTP服务器框架是第三方模块 不属于Node 但已经安装在本地
const express = require("express");
```

如果想导入你自己代码的模块，则模块名应指向包含模块代码的模块文件的路径，通常以相对路径导入：

```js
const stats = require('./stats.js');
const BitSet = require('./utils/bitset.js');
```

虽然省略`.js`后缀之后Node还是可以找到这些文件，但一般还是包括。

如果模块只导出一个类或函数，那么只用`require()`取得返回值就行。如果模块导出一个带多个属性的对象，那么就有两个选择：

1. 导入整个对象
2. 通过解构赋值只导入打算使用的特定属性

```js
// 导入整个stats对象 包含所有函数
const stats = require('./stats.js')

// 虽然导入了用不到的函数 但这些函数都隐藏在"stats"命名空间后
let average = stats.mean(data);

// 当然 也可以使用常见的解构赋值直接向本地命名空间中导入想用的函数
const { stddev } = require('./stats.js');

// 这样简洁明了 只是stddev()函数没有'stats'前缀作为命名空间 因此少了上下文信息
let sd = stddev(data);
```

### 在Web上使用Node风格的模块

通过Exports对象和`require()`函数定义和使用的模块是内置于Node中的。但如果使用webpack等打包工具来处理代码，也可以对浏览器中运行的代码使用这种风格的模块。很多在浏览器上运行的代码都是这么做的。

但既然JS有自己的标准模块语法，开发者即便使用打包工具，通常也会在自己的代码里使用基于`import`和`export`语句的官方JS模块。

## ES6中的模块

ES6为JS添加了`import`和`export`关键字。

ES6模块化和Node的模块化在概念上是相同的。

ES6模块与Node模块的区别在于导入和导出所用的语法，以及浏览器中定义模块的方式。

ES6模块与常规JS"脚本"的明显区别：

- 常规脚本中，顶级声明的变量、函数和类会进入被所有脚本共享的全局上下文。
- 但在模块里，每个文件都有自己的私有上下文，可以使用`import`和`export`语句，当然这正是模块应有之义。

ES6模块中的代码自动应用严格模式。且ES6模块比严格模式更加严格：

- 严格模式下，在作为函数调用的函数中`this`是`undefined`。
- 而在模块里，即便在顶级代码中`this`也是`undefined`。

#### Web和Node中的ES6模块

写作本书时，ES6模块已经得到了除IE之外所有浏览器的原生支持。以原生方式使用时，ES6模块通过特殊的`<script type="module">`标签添加到HTML页面中。

Node13开始支持ES6模块。

### ES6的导出

声明前加上`export`关键字：

```js
export const PI = Math.PI;

export { Circle, degreesToRadians, PI };
```

有时只导出一个值的情况是很常见的，此时通常可以使用`export default`而非`export`：

```js
export default class BitSet {
    // 省略实现
}
```

默认导出在导入时简单一些。

模块中同时出现一些常规导出和一个默认导出是合法的，只是不常见。如果模块里有默认导出，那就只能有一个。

### ES6的导入

导入其他模块导出的值要使用`import`关键字。

```js
import BitSet from './bistset.js';
```

指定模块默认导出的值会变成当前模块中指定标识符的值。

获得导入值的标识符是个常量，导入只能出现在模块顶层，不能出现在函数、类、循环或条件中。

导入与函数声明类似，会被"提升"到顶部，因此所有导入的值在模块代码运行时都是可用的。

导入的模块以常量字符串字面量的形式在单引号或双引号中给出。

该字符串必须是以路径(相对开头`./`或`../`，绝对开头`/`)或带有协议及主机名的完整URL。

从一个模块导入多个值：

```js
import { mean, stddev } from "./stats.js";
```

导入所有值：

```js
import * as stats from "./stats.js";
```

被导入模块的每个非默认导出都会变成这个`stats`对象的一个属性。

通过一条`import`语句同时导入默认值和命名值：

```js
import Histogram, { mean, stddev } from "./histogram-stats.js";
```

`import`还可以导入没有任何导出的模块。要在程序中包含没有导出的模块，只要在`import`关键字后面直接写出模块标识符即可：

```js
import './analytics.js';
```

这样的模块会在被首次导入时运行一次，之后就算再导入也什么都不做。

如果模块中只定义了一些函数，那么它至少要导出其中一个函数才有用。如果模块中运行一些代码，那么即便没有导入也很有用，因为有时候只需要确保某些东西先运行。

### 导入和导出时重命名

可以在导入时使用`as`对导入值重命名：

```js
import { render as renderImage } from "./imageutils.js";
import { render as renderUI } from "./ui.js";
```

导入时重命名的机制为同时定义了默认导出和命名导出的模块提供了另一种导入方式：

```js
import { default as Histogram, mean, stddev } from "./histogram-stats.js";
```

导出时也可以重命名，但只限于使用`export`语句的花括号形式：

```js
export {
    layout as calculateLayout,
    render as renderLayout
};
```

`export`要求`as`前是一个标识符，而非表达式，所以不能：

```js
export { Math.sin as sin, Math.cos as cos };
```

### 再导出

可能会把`mean`和`stddev`函数放在`stats`目录下，通过独立文件实现导入再导出：

```js
import { mean } from "./stats/mean.js";
import { stddev } from "./stats/stddev.js";
export { mean, stddev };
```

ES6将其这种导入和导出合二为一：

```js
export { mean } from "./stats/mean.js";
export { stddev } from "./stats/stddev.js";
```

如果不需要选择性再导出，而是导出另一个模块的所有命名值，可以使用通配符：

```js
export * from "./stats/mean.js";
export * from "./stats/stddev.js";
```

再导出语法允许`as`重命名：

```js
export { mean, mean as average } from "./stats/mean.js";
export { stddev } from "./stats/stddev.js";
```

为没有命名的默认导出定义名字：

```js
export { default as mean } from "./stats/mean.js";
export { default as stddev } from "./stats/stddev.js";
```

如果想把另一个模块的命名符号再导出为当前模块的默认导出，可以在`import`语句后加个`export default`，或像下面按这样组合这两个语句：

```js
export { mean as default } from "./stats.js";
```

如果把另一个的默认导出再导出为当前模块的默认导出：

```js
export { default } from "./stats/mean.js";
```

把别的模块的默认导出再导出为当前模块的默认导出：

```js
export { default } from "./stats/mean.js";
```

### 在网页中使用JS模块

如果想在浏览器中用原生方式使用`import`指令，必须通过`<script type="module">`标签告诉浏览器该代码是个模块。

ES6模块的一个特性是每个模块的导入都是静态的。只要有一个起始模块，浏览器就可以加载它导入的所有模块，然后加载第一批模块导入的所有模块，以此类推，直至加载完所有程序代码。

`<script type="module">`标签用于标记一个模块化程序的起点。该起点模块导入的任何模块预期都不会出现在`<script>`标签中。这些依赖模块会像常规JS文件一样按需加载，且会像常规ES6模块一样在严格模式下执行。

支持`<script type="module">`的浏览器也必须支持`<script nomodule>`。支持模块的浏览器会忽略带有`nomodule`属性的脚本，不执行它们。

为了兼容IE11，可以使用Babel和webpack等工具把代码转换为非模块化的ES5代码，然后通过`<script nomodule>`来加载这些效率没那么高的转换代码。

常规脚本与模块脚本的另一个重要区别涉及跨源加载。`<script type="module">`增加了跨源加载的限制，只能从包含模块的HTML文档所在的域加载模块，除非服务器添加了适当的CORS头部允许跨源加载。这个新的安全限制带来一个副作用：不能在开发模式下使用`file:URL`来测试ES6模块。使用ES6模块时，需要启动一个静态Web服务器来测试。

有些程序员用扩展名`.mjs`来区分模块化JS文件和使用`.js`扩展名的常规、非模块化JS文件。

MIME类型很重要，如果用`.mjs`文件，就需要配置Web服务器以跟`.js`文件相同的MIME类型来发送他们。Node对ES6模块的支持依赖文件扩展名，要靠扩展名来区分加载的文件使用了哪种模块系统。

### 通过import()动态导入

2020年年初，所有支持ES6模块的浏览器都支持动态导入。

传给`import()`一个模块标识符，就会返回一个期约对象，表示加载和运行指定模块的异步过程。动态导入完成后，这个期约会"兑现"并产生一个对象，与使用静态导入语句`import * as`得到的对象类似。

关于期约会在第十三章学到。

如果时静态导入`./stats.js`模块，要这样写：

```js
import * as stats from "./stats.js";
```

如果要动态导入并使用，就要这样写：

```js
import("./stats.js").then(stats => { let average = stats.mean(data); })
```

或者在一个`async`函数里：

```js
async analyzeData(data) {
    let stats = await import("./stats.js");
    return {
        average: stats.mean(data);
        stddev: stats.stddev(data);
    };
}
```

对于`import()`的参数而言，只要求值是个字符串就行。`import()`是个操作符。`import()`需要将模块标识符作为相对于当前运行模块的URL来解析。

动态`import()`不仅在浏览器中有，webpack等打包工具也在积极地利用它。使用打包工具最简单的方式是告诉它程序的主入口，让它找到所有静态`import`指令并把所有代码汇总成一个大文件。然后通过动态`import()`调用，可以把这样一个大文件拆分成多个小文件，实现按需加载。

### import.meta.url

ES6模块中，`import.meta`这个特殊语法引用一个对象，这个对象包含当前执行模块的元数据。

其中该对象的`url`属性是加载模块时使用的URL。

`import.meta.url`的主要使用场景是引用与模块位于同一目录下的图片、数据文件或其他资源。

使用`URL()`构造函数可以方便地相对于`import.meta.url`这样的绝对URL来解析相对URL。

例如：

```js
// 模块 其中包含需要本地化地字符串 而相关地本地化文件保存在l10n/目录下 该目录也保存着模块本身
// 可以使用通过下面的函数创建的URL来加载其字符串
function localStringURL(locale) {
    return new URL(`l10n/${locale}.json`, import.meta.url);
}
```

