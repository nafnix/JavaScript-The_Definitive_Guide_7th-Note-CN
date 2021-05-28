# 浏览器中的JavaScript

客户端JavaScript：在浏览器中的运行的JavaScript代码。

服务器端代码：运行在服务器上的程序。

## Web编程基础

### HTML<script>标签中的JavaScript

```html
<script>
function displayTime(){
    let clock = document.querySelector("#clock");		// 取得带有id="clock"属性的元素
    let now = new Date();								// 取得当前时间
    clock.textContent = now.toLocaleTimeString();		// 时钟里显示时间
}

displayTime()						// 立即显示时间
setInterval(displayTime, 1000);		// 每秒更新一次
</script>
```

也可以使用`src`指定JS代码文件的URL：

```html
<script src="scripts/digital_clock.js"></script>
```

使用`src`的优点：

- 简化HTML文件。将实现内容与行为分离。
- 多个页面共享同一份JS代码时，只需要维护一份代码。
- 若一个JS文件被多个页面共享，则它只会被使用它的第一个页面下载一次，后续页面可以直接从浏览器缓存中获取该文件。
- 因为`src`以任意URL作为值，所以一个Web服务器的JS程序或网页可以利用其他服务器暴露的代码。

#### 模块

第十章中已经说过。

如果用模块写了个JS程序，那就要用一个带有`type="module"`属性的`<script>`标签加载该程序的顶级模块。浏览器会加载指定的模块，并加载该模块导入的所有模块，以及递归地加载这些模块导入的模块。

#### 指定脚本类型

Web早期人们认为浏览器以后会可能实现JS以外的语言，所以程序员要给`<script>`标签添加`<language="javascript">`或`<type="application/javascript">`。

这是没必要的。`language`属性被废弃了，`type`属性只有两个使用场景：

1. 用于指定脚本是模块
2. 在网页中嵌入但是不显示

#### 脚本运行实际：async和defer

浏览器引入JS之初，还没有任何API可以遍历和操作已经渲染好的文档的结构或内容。

为了确保不漏掉脚本可能输出的HTML内容，同时避免同步或阻塞式脚本执行模式并非唯一选项。`<script>`也支持`async`和`defer`属性。这两个是布尔属性：

```html
<script defer src="deferred.js"></script>
<script async src="async.js"></script>
```

这两个属性都告知浏览器，当前链接的脚本没有用`document.write()`生成HTML输出。所以浏览器可以在下载脚本的同时继续解析和渲染文档。

`defer`：让浏览器把脚本的执行推迟到文档完全加载和解析之后。会按照它们出现在文档的顺序运行。

`async`：让浏览器尽早运行脚本，但在脚本下载期间不会阻塞文档解析。运行顺序无法预测。

若同时有`defer`和`async`，则`async`起作用。

带`type="module"`的脚本默认在文档加载完成后执行。可以用`async`覆盖，这样会导致代码在模块及其所有依赖加载完毕后就立即执行。

如果不用`async`和`defer`，也可以把`<script>`放在HTML文件末尾，这样脚本在运行时就知道自己前面的文档内容已经解析完毕。

#### 按需加载脚本

有时候文档刚加载完还不需要某些JS代码，可以通过动态向文档添加`<script>`的方式按需加载脚本：

```js
// 异步加载和执行指定URL的脚本
// 返回期约 脚本加载完毕后解决
function importScript(url) {
    return new Promise((resolve, reject) => {
        let s = document.createElement("script");			// 创建一个<script>元素
        s.onload = () => { resolve(); };					// 加载后解决期约
        s.onerror = (e) => { reject(e); };					// 失败时拒绝期约
        s.src = url;										// 设置脚本的URL
        document.head.append(s);							// 把<script>添加到文档
    })
}
```

### 文档对象模型

客户端JS编程里最重要的一个对象就是Document对象，代表浏览器窗口或标签页中显示的HTML文档。

用于操作HTML文档的API称为文档对象模型(Doucment Object Model，DOM)。

每个HTML标签类型都有个与之对应的JS类，例如`<body>`为`HTMLBodyElement`，`<table>`为`HTMLTableElement`。类的属性也通常对应着标签的属性，某些类定义了额外的方法。

### 浏览器中的全局对象

每个浏览器窗口或标签页都有一个全局对象。一个窗口里运行的所有JS代码都共享同一个全局对象。

全局对象上定义了JS标准库和各种WebAPI的主入口

### 脚本共享一个命名空间

非模板脚本里，一个脚本里定义了一个函数，该函数在其它脚本里是可见的。这对小程序而言或许方便，但大程序中避免命名冲突是件麻烦事，特别是在有些脚本还是第三方库的情况下。

但在ES6中用`const`和`let`还有`class`的顶级声明不会在全局对象上创建属性。但还是会定义在一个共享的命名空间里。即如果一个脚本定了了类`C`，那么另一个脚本也可以用`new C()`创建该类实例，但不可通过`new window.C()`创建。

模板中，顶级声明被限制在模块内部，可以明确导出。

非模板脚本中，顶级声明被限制在包含文档内部，顶级声明由文档中所有的脚本共享。

以前的`var`和`function`通过全局对象的属性共享，现在的`const`和`let`还有`class`被共享且拥有相同的文档作用域，但它们不作为JS可以访问到的任何对象的属性存在。

### JS程序的执行

若网页中包含嵌入的窗格(`<iframe>`元素)，被嵌入文档与嵌入它的文档中的JS代码拥有不同的全局对象和Document对象，可以看作是两个不同的JS程序。但如果它们是从同一个服务器加载，那么它们之间的代码就能交互。

可以将JS程序的执行想象成发生在两个阶段：

1. 第一阶段：文档内容加载完成，`<script>`元素指定的代码运行。此时文档加载完毕且所有脚本运行。
2. 第二阶段：该阶段是异步的、事件驱动的。若脚本要在第二阶段执行，那至少要在第一阶段注册一个将被异步调用的事件处理程序或其它回调函数。

事件驱动阶段发生的第一批事件主要有`"DOMContentLoaded"`和`"load"`：

- `"DOMContent-Loaded"`在HTML文档被完全加载和解析后触发
- `"load"`事件在所有文档的外部资源都完全加载后触发。

#### 客户端JavaScript的线程模型

JS是单线程语言。

JavaScript程序员有责任确保JS脚本和事件处理程序不会长时间运行。

Web平台定义了一种受控的编程模型，即Web工作线程(Web worker)。工作线程是个后台线程，可以执行计算密集型任务而不冻结用户界面。

工作线程里运行的代码无权访问文档内容，不会和主线程或其他工作线程共享任何状态，只能通过异步消息事件与主线程或其它工作线程通信。所以这种并发对主线程没有影响，工作线程也不会改变JS程序的单线程执行模型。

#### 客户端JavaScript时间线

JS程序的执行阶段可以进一步分成下列步骤：

1. 浏览器创建Document对象并开始解析网页，一边解析HTML元素及其文本内容解析以便向文档添加`Element`对象和`Text`节点。此时`document.readyState`属性值为`loading`。
2. HTML解析器在碰到个没有`async`、`defer`或`type="module"`属性的`<script>`标签时，会将该标签添加到文档里，然后执行其中脚本。脚本同步执行，在脚本下载和运行期间，HTML解析器会暂停。它可以遍历已存在的文档树，但通常只会定义函数和注册事件处理程序。
3. 解析器在碰到带有`async`的`<script>`时，会开始下载该脚本的代码(如果该脚本是模块，会递归下载模块的所有依赖)并继续解析文档。`async`脚本不能使用`document.write()`。
4. 文档解析完成后，`document.readState`属性变成`"interactive"`。
5. 有`defer`的脚本会按照它们在文档中出现的顺序执行。异步脚本也可能在此时执行。延迟脚本可以访问完整的文档。不能使用`document.write()`。
6. 浏览器在Document对象上派发`DOMContentLoaded`事件。这标志着程序执行从同步脚本执行阶段过渡到异步的事件驱动阶段。但这时还有可能存在没执行的`async`脚本。
7. 文档完全解析，但浏览器可能在等待其他内容(如图片)加载。所有外部资源加载完成，且所有`async`脚本都加载并执行完成时，`document.readyState`属性变成`"complate"`，浏览器在`Window`对象上派发`"load"`事件。
8. 对用户输入事件、网络事件、定时器超时等的响应，浏览器开始异步调用事件处理程序。

### 程序输入与输出

输入来源：

- 文档内容本身，可通过DOM API访问
- 事件形式的用户输入，如点击HTML上的`<button>`
- 当前文档的URL可以在客户端JS中通过`document.URL`读到
- HTTP"Cookie"请求头的内容在客户端代码中可以通过`document.cookie`读到。Cookie常被服务器端代码用于维持用户会话，但需要时客户端代码也可以读取和写入Cookie
- 全局`navigator`属性暴露了关于浏览器、操作系统以及它们能力的信息：
  - `navigator.userAgent`：浏览器身份字符串
  - `navigator.language`：用户偏好语言
  - `navigator.hardwareConcurrency`：浏览器可用的逻辑CPU个数
  - 类似的，全局`screen`属性暴露用户显示器尺寸信息：
    - `screen.width`和`screen.height`：显示器宽高

### 程序错误

