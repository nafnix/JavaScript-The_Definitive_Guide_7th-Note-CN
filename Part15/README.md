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

如果JS程序在运行期间出现异常，且代码里没有`catch`语句处理它，开发者控制台会显示一条错误消息，但任何已经注册的事件处理程序照样会继续运行和响应事件。

如果想处理未捕获异常，可以把`Window`对象的`oneerror`属性设为一个错误处理函数。

若错误消息即将显示在开发者控制台时，`window.onerror`函数将会以三个参数被调用：

1. 描述错误的消息
2. 包含导致错误的JS代码的URL的字符串
3. 文档中发生错误的行号

如果`onerror`返回`true`，即表示已经处理错误，也就是浏览器不应该再显示自己的错误消息了。

若期约被拒绝且没有`.catch()`函数处理它，那这种情况类似未处理异常。可以定义`window.onunhandledrejection`函数或用`window.addEventListener()`为"unhandledrejection"事件注册一个处理程序来发现它。传给该处理程序的事件对象会有两个属性：

1. `promise`：被拒绝的`Promise`对象
2. `reason`：本来要传给`.catch()`函数的拒绝理由

若在该事件对象上调用`preventDefault()`，浏览器就会认为错误已经处理，而不会在开发者控制台中显示错误。

虽然捕获`onerror`和`onunhandledrejection`处理程序经常不是必需，但如果想知道用户浏览器发生了哪些意外错误，可以用它们把客户端错误上报给服务器。

### Web安全模型

#### JavaScript不能做什么

客户端JS不能向客户端计算机中写入或删除任何文件，也不能展示任意目录的内容。

客户端JS没有通用网络能力。无法随意访问任意服务器。

#### 同源策略

同源策略：对JS代码能够访问和操作什么Web内容的一整套限制。

通常在页中包含`<iframe>`元素时就会涉及同源策略，同源策略控制着一个窗格中的JS与另一个窗格中的JS的交互。例如，脚本只能读取与包含它的文档同源的Window和Document对象的属性。

文档的源就是文档URL的协议、主机和端口。

不同源：

- 从不同服务器加载的文档
- 从相同主机的不同端口加载的文档
- `http:`和`https:`加载的文档即使来自同一服务器

浏览器通常把每个`file:URL`看作一个独立的源。

即如果程序显示同一台服务器上的多个文档，则可能无法使用`file:URL`在本地测试它，必须在开发期间运行一个静态Web服务器。

如果一台主机(A)的页面里包含另一台主机(B)上的脚本，那么这个脚本的源是页面所在的主机，这个脚本对该页面具有完全访问权。如果页面里有`<iframe>`并且嵌入的还是A主机所在的页面，那么这个脚本还是对这个页面有完全访问权；但如果`<iframe>`里的文档来自其它的主机(C或甚至是A)，那么同源策略就会阻止该脚本访问这个嵌入的文档。

JS代码可以向托管其包含文档的服务器发送任意HTTP请求，但不能和其它服务器通信(除非那些服务器开启了后面要介绍的CORS)。

同源策略对使用多子域的大型网站造成了麻烦。例如`orders.example.com`的脚本可能要读取`example.com`上文档的属性。为支持这种多子域的网站，脚本可以通过`document.domain`设为一个域名后缀来修改自己的源。比如源为`https://orders.example.com`的脚本通过把`document.domain`设为`https://example.com`。

另一种缓解同源策略的技术是跨源资源共享(Cross-Origin Resource Sharing，CORS)，它允许服务器决定对哪些源提供服务。CORS扩展了HTTP，增加了一个新的`Origin:`请求头和一个新的`Access-Control-Allow-Origin`响应头。服务器可以通过这个头部明确列出对哪些源提供服务，或者使用通配符表示可以接收任何网站的请求。浏览器会根据这些CORS头部的有无决定是否放松同源限制。

#### 跨站点脚本

跨站点脚本(Cross-Site Scripting，XSS)，是种攻击方式，攻击者向目标网站注入HTML标签或脚本。

若页面的内容是动态生成，例如根据用户提交的数据生成内容，但没有提前对那些数据"消毒"，那就可能成为跨站点脚本的攻击目标。

```html
<script>
	let name = new URL(document.URL).searchParams.get("name");
    document.querySelector('h1').innerHTML = "Hello " + name;
</script>
```

页面期望调用：

```js
http://www.example.com/greet.html?name=David
// 对该URL，页面显示"Hello David"
```

但若用户输入查询参数：

```js
name=%3Cimg%20src=%22x.png%22%20onload=%22alert(%27hacked%27)%22/%3E
```

那就会被转为：

```js
hello <img src="x.png" onload="alert('hacked')" />
```

图片加载后，`onload`中的JS字符串就会执行。显然这是开发者不希望见到的。

称为跨站点脚本攻击，是因为会涉及到不止一个网站。网站b包含一个特殊链接指向A。若网站B能够说服用户点击该链接，用户就会导航到网站A，但网站A此时会运行来自网站B的代码。这个代码可能会破坏网站A的页面，或导致它功能失效。甚至可能读取网站A存储的`cookie`并将其发送回网站B。这种注入的代码甚至能够跟踪用户键盘输入，并将数据发送回网站B。

一般来说，防止XSS攻击的方法是从不可信数据中删除HTML标签，然后再用它去动态创建文档内容。

对前面展示的`greet.html`，可以把不可信输入中的特殊HTML字符改为等价的HTML实体来解决：

```js
name = name
	.replace(/&/g, "&amp;")
	.replace(/</g, "&lt;")
	.replace(/>/g, "&gt;")
	.replace(/"/g, "&quot;")
	.replace(/'/g, "&x27;")
	.replace(/\//g, "&#x2F;")
```

应对XSS攻击的另一个思路是让自己的Web应用始终在一个`<iframe>`中显示不可信内容，并将该`<iframe>`的`sandbox`属性设为禁用脚本和其它能力。

## 事件

##### 事件类型

是个字符串，表示发生了什么事件。例如`"mousemove"`表示用户移动了鼠标，`"keydown"`表示用户按下了键盘上的某个键。

因为时间类型是字符串，所以有时也成为事件名称。

##### 事件目标

是个对象，事件发生在该对象上或这事件与该对象有关。事件必须明确它的类型和目标。

##### 事件处理程序或事件监听器

事件处理程序或事件监听器是个函数，负责处理或响应事件。

应用通过浏览器注册自己的事件处理程序，指定事件类型和事件目标。当事件目标上发生指定类型的事件时候，浏览器会调用对应的处理程序。

当事件处理程序在某个对象上被调用时，说浏览器"触发""派发"或"分配"了该事件。

##### 事件对象

是与特定事件关联的对象，包含有关该时间的细节。

事件对象作为事件处理程序的参数传入。所有事件都有以下两个属性：

- `type`：事件类型
- `target`：事件目标

且每种事件类型都为相关的事件对象定义了一组属性。

##### 事件传播

是个过程，浏览器会决定在该过程中哪些对象触发事件处理程序。

对于`Window`对象上的`"load"`或`Worker`对象上的`"message"`等特定于一个对象的事件，不需要传播。

但对于发生在HTML文档里的某些事件，则会"冒泡"(bubble)到文档根元素。例如在一个超链接上移动鼠标，这个鼠标事件先会在定义该超链接的`<a>`元素上触发，然后在包含该元素的元素上触发，可能经过一个`<p>`元素，然后到达文档对象本身。

有些事件有与之关联的默认动作(default action)。比如单击一个超链接，默认动作是让浏览器跟随链接，加载一个新页面。事件处理程序可以通过调用事件对象的一个方法来阻止该默认动作。

### 事件类别

##### 设备相关输入事件

直接与特定输入设备(如鼠标键盘)相关。

##### 设备无关输入事件

不与特定输入设备直接相关。

如`"click"`事件表示一个链接或按钮已被激活。一般来说该事件通过鼠标触发，但也可能是通过键盘或在触屏设备上通过轻击触发。而`"input"`事件是对`"keydown"`事件的设备无关的替代，既支持键盘输入，也支持剪切粘贴和表意文字的输入法。

##### 用户界面事件

UI事件是高级事件，通常在定义应用界面的HTML表单元素上触发。

这类事件包括：

- `focus`：当文本输入字段获得键盘焦点时
- `change`：当用户修改了表单元素显示的值时
- `submit`：当用户单击表单中的"提交"按钮时

##### 状态变化事件

这类事件表示某种生命期或状态相关的变化。

例如`Window`和`Document`对象在文档加载结束时触发`load`和`DOMContentLoaded`事件。

##### API特定事件

一些HTML及相关规范定义的Web API包含自己的事件类型。

### 注册事件处理程序

有两种注册事件处理程序的方式：

1. Web早期，设置作为事件目标的对象或文档元素的一个属性
2. 通用，将处理程序传给该对象或元素的`addEventListener()`方法

##### 设置事件处理程序属性：JavaScript

将事件目标的一个属性设为关联的事件处理程序函数。

事件处理程序属性的名字都由`on`和事件名称组成。

示例：

```js
// 设置Window对象的onload属性为一个函数
// 该函数是事件处理程序 会在文档加载完成时被调用
window.onload = function() {
    // 找一个<form>元素
    let form = document.querySelector("form#shipping");
    // 在该表单上注册一个事件处理程序 在表单被提交前会调用该函数 假设其他地方已经定义了isFormValid()
    form.onsubmit = function(event) {		// 用户提交表单时
        is(!isFormValid(this)) {			// 检查表单是否有效
            event.preventDefault();			// 无效则组织提交
        }
    };
};
```

缺点：这种方式假设事件目标对这种事件最多只有一个处理程序。

##### 设置事件处理程序属性

文档元素的事件处理程序属性也可以直接在HTML文件中作为对应HTML标签的属性来定义。

```html
<button onclick="console.log('Thank you');">Please Click</button>
```

在给HTML事件处理程序属性指定JS代码字符串时，浏览器会将该字符串转为函数。该函数类似如下：

```js
function(event) {					
    with(document) {				
        with(this.form || {}) {
            with(this) {
                /* 代码在这里 */
            }
        }
    }
}
```

`event`意味处理程序代码可以通过它引用当前的事件对象。

`with`意味处理程序可以直接引用目标对象、外层form(如果有)，乃至`Document`对象的属性，就像它们都是作用域里的遍历一样。

严格模式下是禁止使用`with`语句的，但HTML属性里的JS代码没有严格之说。

这样定义的事件处理程序将在一个可能存在意外变量的环境中执行，因此可能是一些bug的来源。

##### addEventListener()

任何能作为事件目标的对象，都定义了`addEventListener()`方法，可以用它来注册目标是调用对象的事件处理程序。

该方法接收三个参数：

1. 注册处理程序的事件类型，字符串，不包含作为HTML元素属性使用时的前缀"on"
2. 指定类型的事件发生时调用的函数
3. 可选布尔值或对象：
   1. `true`：函数被注册为捕获事件处理程序，从而在事件派发的另一阶段调用它

在一个`<button>`上为`click`事件注册两个事件处理程序：

```html
<button id="mybutton">
    Click me
</button>

<script>
	let b = document.querySelector("#mybutton");
    
    b.onclick = function() { console.log("Thanks for clicking me!"); };
    
    b.addEventListener("click", () => { console.log("Thanks again!"); });
</script>
```

当对象上发生事件时，所有未该事件注册的处理程序会按照注册它们的顺序被调用。

同一对象上以相同参数多次调用`addEventListener()`没有作用，同一处理程序只能注册一次。

与之对应的是`removeEventListener()`方法，前两个参数相同，第三个参数也是可选，如果`addEventListener()`第三个参数为`true`，那么该方法也要是`true`。该方法是从同一对象上移除事件处理程序。。

可以给`mousemove`和`mouseup`事件注册临时事件处理程序，以便知道用户是否拖动鼠标，在`mouseup`事件发生时移除这两个处理程序：

```js
document.removeEventListener("mousemove", handleMouseMove);
document.removeEventListener("mouseup", handleMouseUp);
```

给第三个参数传递对象：

```js
document.addEventListener("click", handleClick, {
    capture: true,
    once: true,
    passive: true
});
```

- `capture`属性：
  - `true`：该函数会被注册为捕获处理程序
  - `false`或省略：处理程序不会注册到捕获阶段
- `once`属性：
  - `true`：事件监听器在被触发一次后自动移除
  - `false`或省略：处理程序永远不会被自动移除
- `passive`属性：
  - `true`：事件处理程序永远不会调用`preventDefault()`取消默认动作：
    - 如果`touchmove`事件可以阻止浏览器的默认滚动动作，那浏览器就不能实现平滑滚动
  - `false`：将`touchmove`和`mousewheel`事件注册为一个会调用`preventDefault()`的事件处理程序

`passive`提供了一种机制，注册一个可能存在破坏性操作的事件处理程序时，让浏览器知道可以在事件处理程序运行的同时安全地开始其默认行为(如滑动)。

Firefox和Chrome默认把`touchmove`和`mousewheel`事件设为"被动式"(`passvie: true`)。

可以把上述(选项)对象传给`removeEventListener()`，但只有`capture`是有效的，其它两个属性会被忽略。

### 调用事件处理程序

注册事件处理程序后，浏览器会在指定对象发生指定事件时自动调用它。

#### 事件处理程序的参数

事件处理程序被调用时会接收到一个`Event`对象作为唯一的参数。

该`Event`对象的属性提供了事件的详细信息。

##### type

发生事件的类型

##### target

发生事件的对象

##### currentTarget

注册当前事件处理程序的对象

##### timeStamp

事件发生时间的时间戳(毫秒)，不是绝对时间，可以用两个事件的时间戳相减来计算两个事件间隔的事件。

##### isTrusted

若事件由浏览器自身派发，则属性为`true`；若事件由JS代码派发，则属性为`false`。

#### 事件处理程序的上下文

在事件处理程序的函数体里，`this`关键字引用的是注册事件处理程序的对象。

用`addEventListener()`注册，处理程序在被调用时也会以目标作为其`this`值。不过不适用于箭头函数形式的处理程序：箭头函数的`this`值始终等于定义它的作用域里的`this`值。

#### 处理程序的返回值

现代JS里，事件处理程序不应该返回值。

老代码里返回的值通常用于告诉浏览器不要执行与事件相关的默认动作。如阻止单击表单`submit`的提交。

阻止浏览器执行默认动作的标准且推荐的方式，是调用`Event`对象的`preventDefault()`方法。

#### 调用顺序

事件目标可能会为一种事件注册多个处理程序。浏览器会按照注册处理程序的顺序调用它们。

即便混用`addEventListener()`和在对象属性`onclick`上注册的事件处理程序，结果依然如此。

### 事件传播

若事件目标为`Window`或其它独立属性，浏览器对该事件的响应只是简单地调用该对象上地事件处理程序。

事件目标为`Document`或其它文档元素时，注册在目标元素上的事件处理程序被调用后，多数事件会沿DOM树向上"冒泡"。注册到目标父元素的事件处理程序会被调用。一直上升到`Document`对象，后再到`Window`对象。

可以在它们的公共祖先元素上注册一个事件处理程序，在其中处理事件。

1. 第一阶段：目标处理程序被调用前，称"捕获"阶段。若`addEventListener()`的第三个参数为`true`或`{capture:true}`，即表明该事件处理程序会注册为捕获事件处理程序，将在事件传播的第一阶段被调用。
2. 第二阶段：调用目标对象本身的事件处理程序
3. 第三阶段：事件冒泡

### 事件取消

浏览器对很多用户事件都会做出响应。如用户在链接上单击鼠标，浏览器就会跟随该链接。如用户在触摸屏上滑动手指，浏览器就会滚动。

事件对象方法：

- `preventDefault()`：阻止浏览器执行其默认动作。
- `stopPropagation()`：取消事件传播。如果同一对象上定义了其它处理程序，那么它们会被调用。但其父级元素上的事件处理程序不会捕获到该事件。

`stoppropagation()`可以在捕获阶段、在事件目标本身，以及在冒泡阶段起作用。

`stopImmediatePropagation()`与`stopPropagation()`类似，不过它会阻止在同一个对象上注册的后续事件处理程序的执行。

### 派发自定义事件

可以用客户端JS事件API定义和派发自己的事件。

例如在程序需要周期性执行耗时计算或发送网络请求，在执行此操作期间，不能执行其他操作。想在此时显示一个转轮图标。这时只需派发一个事件。再在不忙时派发另一个事件即可。

若JS对象有`addEventListener()`方法，那就是个"事件目标"。也意味该对象有个`dispatchEvent()`方法。

可以用`CustomEvent()`构造函数创建自定义事件对象，再将它传给`dispatchEvent()`。它的参数：

1. 字符串，事件类型
2. 对象，事件对象属性，可以将该对象的`detail`属性设为字符串、对象或其他值，表示事件的上下文。若想在一个文档元素上派发自己的事件，并希望它沿文档树向上冒泡，就要在第二个参数里添加`bubbles:true`。

```js
// 派发自定义事件 告知UI自己在忙
document.dispatchEvent(new CustomEvent("busy", { detail: true }));

// 执行网络操作
fetch(url)
	.then(handleNetworkResponse)
	.catch(handleNetworkError)
	.finally(() => {
    // 无论网络请求成功还是失败 都再派发一个事件 通知UI自己不再忙了
    document.dispatchEvent(new CustomEvent("busy", { detail: false}));
});

// 在代码其他地方为"busy"事件注册一个处理程序 并通过它显示或隐藏转轮图标 告知用户忙于闲
document.addEventListener("busy", (e) => {
    if(e.detail) {
        showSpinner();
    } else {
        hideSpinner();
    }
});
```

## 操作DOM

#### 通过CSS选择符选择元素

`querySelector()`：

- 参数：CSS选择符字符串
- 返回：文档中找到的第一个匹配的元素，没找到就返回`null`

```js
// 查找文档里所有HTML标签包含属性id="spinner"的元素
let spinner = document.querySelector("#spinner");
```

`querySelectorAll()`类似，但返回类似数组的`NodeList`对象，其中包含所有匹配的元素，该对象有个`length`属性，可以像数组一样通过索引访问，也是可迭代对象。

```js
// 查找所有<h1>、<h2>、<h3>标签的Element对象
let titles = document.querySelectorAll("h1, h2, h3");
```

在元素上调用时，这两个方法只返回该元素后代中的元素。

`closest()`：

- 参数：CSS选择符
- 返回：
  - 若选择符匹配那个调用它的元素，则返回该元素
  - 否则返回与选择符匹配的最近祖先元素
  - 如果没有匹配则返回`null`

让事件处理程序查找最近的超链接：

```js
// 查找有href属性的最近的外围<a>标签
let hyperlink = event.target.closest("a[href]");
```

另一个例子：

```js
// 若e被包含在一个HTML列表元素里就返回true
function insideList(e) {
    return e.closest("ul,ol,dl") !== null;
}
```

`matches()`可以检查元素是否与选择符匹配，如果匹配就返回`true`，否则`false`：

```js
// 若e是个HTML标题元素则返回true
function inHeading(e) {
    return e.matches("h1,h2,h3,h4,h5,h6");
}
```

#### 其他选择元素的方法

一些老式的元素选择方法。

```js
// 通过id查找
let sect1 = document.getElementById("sect1");

// 查找具有name="color"属性的所有元素 类似document.querySelector('*[name="color"]')
let colors = document.getElementsByName("color");

// 查找文档中所有的<h1>元素
let headings = document.getElementsByTagName("h1");

// getElementsByTagName()在Element对象上也有定义 取得sect1的后代里的所有<h2>元素
let subheads = sect1.getElementsByTagName("h2");

// 查找所有类名里包含"tooltip"的元素
let tooltips = document.getElementByClassName("tooltip");

// 查找sect1的后代里所有类名包含"sidebar"的元素
let sidebars = sect1.getElementsByClassName("sidebar");
```

它们也返回`NodeList`，除了`getElementById()`返回`Element`对象。但它们的`NodeList`的`length`属性和其中的元素会随着文档内容或结构的变化而变化。

#### 预选择的元素

历史原因，`Document`类定义了一些快捷属性，可以通过它们直接访问某种节点。

`images`、`forms`和`links`属性可以直接访问文档里的`<img>`、`<form>`、`<a>`元素。

这些属性引用的时`HTMLCollection`对象，与`NodeList`对象非常类似，但还可以用元素ID或名字来索引其中的元素。

```js
document.forms.address;
```

还有`document.all`，该属性引用的对象类似`HTMLCollection`。

### 文档结构与遍历

`parentNode`：该属性引用元素的父节点，即另一个`Element`对象或`Document`对象。

`chlidren`：该属性是`NodeList`，含元素的所有子元素，不含`Element`节点，如`Text`节点。

`childElementCount`：该属性是元素所有子元素的个数。

`firstElementChild`、`lastElementChild`：分别引用元素的第一个子元素和最后一个子元素。如果没有子元素，它们的值为`null`。

`previousElementSibling`、`nextElementSibling`：元素左侧紧邻同辈元素和右侧紧邻同辈元素，若没有则为`null`。

```js
// 引用Document第一个子元素的的第二个元素
document.children[0].children[1]
document.firstElementChild.firstElementChild.nextElementSibling
```

标准HTML中，这两个表达式引用的都是文档的`<body>`标签。

下面两个函数对文档执行深度优先的遍历，并对文档的每个元素都调用一次指定的函数：

```js
// 递归遍历Document或Element e
// 在e和每个后代元素上调用函数f
function traverse(e, f){
    f(e);								// 在e上调用f()
    for(let child of e.children){		// 迭代所有孩子
        traverse(child, f);				// 每个孩子递归
    }
}

function traverse2(e, f){
    f(e);								// e上调用f()
    let child = e.firstElementChild;	// 链表式迭代孩子
    while(child !== null) {
        traverse2(child, f);			// 并在这里递归
        child = child.nextElementSibling;
    }
}
```

#### 作为节点树的文档

所有`Node`对象都定义了以下属性：

`parentNode`：当前节点的父节点，没有父节点或`Document`对象则为`null`。

`childNodes`：只读的`NodeList`对象，包含节点的所有子节点(不只包含`Element`子节点)。

`firstChild`、`lastChild`：当前节点的第一个子节点和最后一个子节点，如果没有子节点则为`null`。

`previousSibling`、`nextSibling`：当前节点的前一个同辈节点和后一个同辈节点。这两个属性通过双向链接连接节点。

`nodeType`：表示当前节点类型的数值：

- `Document`：9
- `Element`：1
- `Text`：3
- `Comment`：8

`nodeValue`：`Text`或`Comment`节点的文本内容

`nodeName`：`Element`节点的HTML标签名，会转为全部大写

引用`Document`第一个子节点的第二个子节点：

```js
document.childNodes[0].childNodes[1]
document.firstChild.firstChild.nextSibling
```

对应HTML：

```html
<html>
    <head>
        <title>Test</title>
    </head>
    <body>
        Hello World!
    </body>
</html>
```

第一个子节点的第二个子节点为`<body>`元素，`nodeType`为`1`，`nodeName`为`"BODY"`。

返回元素或文档中所有文本的函数：

```js
// 返回元素e的纯文本内容 递归包含子元素
// 该方法类似元素的textContent属性
function textContent(e) {
    let s = "";								// 累积文本
    for(let child = e.firstChild; child !== null; child = child.nextSibling) {
        let type = child.nodeType;
        if (type === 3){					// 若是Text节点
            s += child.nodeValue;			// 将文本内容追加到字符串
        } else if (type === 1) { 			// 若是Element节点
            s += textContent(child);		// 则递归
        }
    }
}
```

### 属性

`Element`类定义了`getAttribute()`、`setAttribute()`、`hasAttribute()`、`removeAttribute()`方法，用于查询、设置、检查、删除元素的属性。

#### 作为元素属性的HTML属性

表示HTML文档中元素的`Element`类通常会定义读/写属性。

`HTMLElement`为通用HTML属性和事件处理程序属性定义了属性。特定的`Element`子类型则定义了特定于相应元素的属性。

```js
// 修改img元素的src
let image = document.querySelector("#main_image");
let url = image.src;								// src属性为图片的URL
image.id === "main_image"							// true
```

有些元素的HTML属性名会映射到不同的JS属性。如`<input>`在HTML的`value`属性是由JS的`defaultValue`属性镜像的。JS的`value`属性包含用户当前在`<input>`元素里输入的值。但修改`value`属性，不会影响JS的`defaultValue`属性，也不会影响HTML的`value`属性。

HTML属性不区分大小写，但JS属性名区分大小写。将HTML属性转为JS属性，只要全部小写就行。如果HTML属性包含多个单词，那就从第二个单词开始，每个单词的首字母都大写。但事件处理程序属性除外。

有些HTML属性名和JS的保留字冲突，这些属性的对应规则通常是多出前缀"html"：`<lable>`在HTML里的`for`变成JS里的`htmlFor`属性。对于`class`因为转为JS也和`HTMLclass`冲突，所以实际是`className`。

JS中表示HTML属性的属性通常都是字符串值。但HTML属性是布尔值或数字值时，相应的JS属性则是布尔值或数值，不是字符串。事件处理程序属性的值则始终为函数或`null`。

该方式只能获取和设置HTML里的对应的属性值，不能删除，删除用`removeAttribute()`。

#### class属性

HTML中的`class`属性值是空格分隔的CSS类名列表。

`className`属性可用于设置或返回HTML中`class`属性的字符串值。

`Element`对象定义了`classList`属性，支持将`class`属性作为一个列表来操作，其值为一个可迭代的类数组对象。定义了`add()`、`remove()`、`contains()`、`toggle()`方法。

```js
// 让用户知道此刻在忙 显示个转轮图标 删除hidden类 添加animated类
let spinner = document.querySelector("#spinner");
spinner.classList.remove("hidden");
spinner.classList.add("animated");
```

#### dataset属性

有时在HTML元素上附加一些信息可以帮助JS代码选择操作这些元素。

DOM里，`Element`对象的`dataset`属性引用的对象包含与HTML中的`data-`属性对应的属性，但不带该前缀。

即`dataset.x`保存HTML中`data-x`属性的值。连字符分隔的属性映射为驼峰式属性名：HTML的`data-section-number`转为JS的`dataset.sectionNumber`。

HTML：

```html
<h2 id="title" data-section-number="16.1">
    Attribute
</h2>
```

可以用以下JS访问其中的节号：

```js
let number = document.querySelector("#title").dataset.sectionNumber;
```

### 元素内容

#### 作为HTML的内容

读取`Element`的`innerHTML`属性会返回该元素内容的标记字符串。元素上设置该属性会调用浏览器的解析器，并以新字符串解析后的表示替换元素当前的内容。

```js
document.body.innerHTML = "<h1>Oops</h1>";
```

通过`+=`操作符该`innerHTML`追加文本的效率不高。因为该操作会涉及序列化操作，也会涉及解析操作：将元素内容转为字符串，再将新字符串转回元素内容。

`Element`的`outerHTML`属性与`innerHTML`属性类似，只是返回的值包含元素自身。

`insertAdjacentHTML()`：用于插入与指定元素"相邻"(adjacent)的任意HTML标记字符串。插入标签为第二个参数，"相邻"的精确含义为第一个参数的值：

- `"before-begin"`
- `"afterbegin"`
- `"beforeend"`
- `afterend`

```html
(beforebegin)<div id="target">
    (afterbegin)This is the element content(beforeend)
</div>(afterend)
```

#### 作为纯文本的内容

获得元素的纯文本内容，或向文档中插入纯文本(不转义HTML中使用的尖括号和`&`字符)：

```js
let para = document.querySelector("p");		// 文档中第一个<p>
let text = para.textContent;				// 取得该段落的文本
para.textContent = "Hello World!";			// 修改该段落的文本
```

`textContent`由`Node`类定义。对于`Element`节点，会找到并返回元素所有后代中的文本。

`Element`类有个`innerText`属性与`textContent`类似。但`innerText`有些少见和复杂的行为，如试图阻止表格格式化。但该属性的定义不严谨，浏览器间的实现也存在兼容性问题，因此不该再使用。

`<script>`元素中的文本，行内`<script>`元素有个`text`属性，可以用于获取它的文本。浏览器永远不会显示`<script>`元素的内容，HTML解析器会忽略脚本中的尖括号和`&`字符。只要把该元素的`type`属性改为某个值(如`text/x-custom-data`)，明确其不是可执行的JS代码即可。这样，JS解释器就会忽略该脚本，但该元素还会出现在文档树里，其`text`属性可以返回在其中保存的数据。

### 创建、插入和删除节点

`Document`的`createElement()`可以创建新元素，并通过`append()`和`prepend()`添加文本或其他元素：

```js
let paragraph = document.createElement("p");		// 创建空p元素
let emphasis = document.createElement("em");		// 创建空em元素
emphasis.append("World");							// 向em元素添加文本
paragraph.append("Hello", emphasis, "!");			// 向p添加文本和em
paragraph.prepend("i");								// 在p开头添加文本
paragraph.innerHTML							// "iHello <em>World!</em>"
```

`append()`和`prepend()`接收任意多个参数，参数可以是`Node`对象或字符串。字符串自动转为`Text`节点(也可以用`document.createTextNode()`创建`Text`节点)。

在元素的子列表间插入`Element`或`Text`：

```js
// 找到class="greetings"的标题元素
let greetings = document.querySelector("h2.greetings");

// 在这个标题后插入新创建的paragraph和一条水平线
greetings.after(paragraph, document.createElement("hr"));
```

`after()`和`before()`类似`append()`和`prepend()`，参数相同。但它们同时存在于`Element`和`Text`节点上，可以使用它们相对于`Text`节点插入内容。

元素只能被插入到文档中的一个地方。若某个元素已经在文档中，再将其插到其他地方，那它就会移动到新位置。

如果想创建元素副本，可以用`cloneNode()`方法，传入`true`以复制其全部内容：

```js
// 创建paragraph的一个副本 再把它插入到greetings元素后面
greetings.after(paragraph.cloneNode(true));
```

`remove()`：将`Element`或`Text`从文档中删除，不接收参数。

`replaceWith()`：替换，参数与`before()`和`after()`一样。

```js
// 从文档中删除greetings元素 并代之以paragraph元素
// 若paragraph已在文档中 则将其从当前位置移走
greetings.replaceWith(paragraph);

// 删除paragraph元素
paragraph.remove();
```

## [下一节](./2.md)

