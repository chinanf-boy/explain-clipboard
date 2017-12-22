# ClipboardAction 

> 居然找到这里，我行动大魔王把 .btn 抓到这里

> [抢救你的 .btn](#点击事件) `<#######----or----#######>` [返回 Clipboard 类 解释家](./README.md)

---

抢救开始 `explain`

- [配置开始](#配置开始)

- [初始化复制目标](#初始化复制目标)

- [其他](#其他)

    - tiny-emitter

    - execCommand

    - [select库 - 获取元素文本](#选定文本)

- [彩蛋](#类的属性)
    
---

点击事件之后

## 配置开始

在 ` Clipboard.js `中

``` js
        this.clipboardAction = new ClipboardAction({
            action    : this.action(trigger),
            target    : this.target(trigger),
            text      : this.text(trigger),
            container : this.container,
            trigger   : trigger,
            emitter   : this
        });
    }

```

### ClipboardAction.js 应用配置

``` js
    constructor(options) {
        this.resolveOptions(options); //<--
        this.initSelection();
    }
    resolveOptions(options = {}) {
    this.action    = options.action; 
    this.container = options.container;
    this.emitter   = options.emitter;
    this.target    = options.target;
    this.text      = options.text;
    this.trigger   = options.trigger;

    this.selectedText = '';
}
```

这个时候要注意啦

`this.action` 的 获得

``` js
    constructor(options) {
        // this.action == undefined  --- 0
        this.resolveOptions(options);
        // this.action == copy 因为 set   ---- 3
        this.initSelection(); 
    }
    resolveOptions(options = {}) {
        //-- set触发 this.action
    this.action    = options.action;  // --- 下面的代码块
        // --- 2  
    //...
    }
```

``` js
    set action(action = 'copy') { // <--- copy 初始化
        this._action = action;
        //   --- 1
        if (this._action !== 'copy' && this._action !== 'cut') {
            throw new Error('Invalid "action" value, use either "copy" or "cut"');
        }
    }

```   

可以看到, `this.action` -声明的顺序

>再找遍全代码都没有明确声明 

>`this.action 默认 == 'copy'`

>居然用 `set`

[很有想法-讨论](#类的属性) or

next

---

### 初始化，需要复制的文本

``` js
    constructor(options) {
        this.resolveOptions(options); 
        this.initSelection(); //<--
    }
    /**
     * Decides which selection strategy is going to be applied based
     * on the existence of `text` and `target` properties.
     */
    initSelection() {
        if (this.text) { // 有没有 指定 text
            // 指定了  Text
            this.selectFake(); 
        }
        else if (this.target) { // 有没有指定 目标元素
            // 指定了  目标元素
            this.selectTarget();
        }
    }
```

next

---

## 初始化复制目标

还记得，我们这次的例子吗 `new Clipboard('.btn');`

在本次例子中 target 是存在的 , 当然你也可以看看 selectFake 做了什么

``` html
<!-- Target -->
<input id="foo" value="https://github.com/zenorocha/clipboard.js.git">

<!-- Trigger -->
<button class="btn" data-clipboard-target="#foo">
    <img src="assets/clippy.svg" alt="Copy to clipboard">
</button>
```

- [text存在 -> this.selectFake ](#文本存在) <-- 随便逛逛

- [target存在 -> this.selectTarget ](#目标存在) <--本次例子来这里

### 文本存在

``` js
   /**
     * Creates a fake textarea element, sets its value from `text` property,
     * and makes a selection on it.
     */
    selectFake() {
        const isRTL = document.documentElement.getAttribute('dir') == 'rtl';

        this.removeFake(); // <-- 进入先

        ...
    }
    /**
     * Only removes the fake element after another click event, that way
     * a user can hit `Ctrl+C` to copy because selection still exists.
     */
    removeFake() { // 看来是移除 fake 事件和 元素
        if (this.fakeHandler) {
            this.container.removeEventListener('click', this.fakeHandlerCallback);
            this.fakeHandler = null;
            this.fakeHandlerCallback = null;
        }

        if (this.fakeElem) {
            this.container.removeChild(this.fakeElem);
            this.fakeElem = null;
        }
    }
```

next

``` js
   selectFake() {
        const isRTL = document.documentElement.getAttribute('dir') == 'rtl'; // ？

        this.removeFake();
        
        // 接下来
        this.fakeHandlerCallback = () => this.removeFake();

        // this.container == document.body
        this.fakeHandler = this.container.addEventListener('click', this.fakeHandlerCallback) || true;

        // 创建一个 textarea 空元素
        this.fakeElem = document.createElement('textarea');
        // Prevent zooming on iOS 
        this.fakeElem.style.fontSize = '12pt';
        // Reset box model
        this.fakeElem.style.border = '0';
        this.fakeElem.style.padding = '0';
        this.fakeElem.style.margin = '0';
        // Move element out of screen horizontally
        this.fakeElem.style.position = 'absolute';
        this.fakeElem.style[ isRTL ? 'right' : 'left' ] = '-9999px';
        // Move element to the same position vertically
        let yPosition = window.pageYOffset || document.documentElement.scrollTop;
        this.fakeElem.style.top = `${yPosition}px`;

        this.fakeElem.setAttribute('readonly', '');
        this.fakeElem.value = this.text;

        //______********_______*********

        // this.fakeElem
        // <textarea style="font-size:12pt;border:0;padding:0;margin:0;position:'absolute';left:'-9999px';top:***">
        // this.text is nothing
        //</textarea>

    //top: *** 表示滚动网页离最上面的距离
    // 所以其实，本次例子，没有进到 selectFake,因为有 target

        this.container.appendChild(this.fakeElem);

        this.selectedText = select(this.fakeElem); //？
        this.copyText();
    }
```

``` js
    /**
     * Only removes the fake element after another click event, that way
     * a user can hit `Ctrl+C` to copy because selection still exists.
     */
    removeFake() {
        if (this.fakeHandler) {
            // 全局 事件 remove
            this.container.removeEventListener('click', this.fakeHandlerCallback);
            this.fakeHandler = null;
            this.fakeHandlerCallback = null;
        }

        if (this.fakeElem) { // 创建的 元素 
        //remove
            this.container.removeChild(this.fakeElem);
            this.fakeElem = null;
        }
    }
```

---

### 目标存在

``` js
    /**
     * Selects the content from element passed on `target` property.
     */
    // 真的短，额不是我是说，真简洁
    selectTarget() {
        this.selectedText = select(this.target); //
        this.copyText();
    }
```

看来我们需要，[弄清楚 select库 的能力了--点击](#选定文本)

不过初步看应该是对元素 `element`的 `value` 进行选定

next

---

复制文本

`this.copyText`

``` js
    /**
     * Executes the copy operation based on the current selection.
     */
    copyText() {
        let succeeded;

        try {
            // this.action 默认 copy 
            succeeded = document.execCommand(this.action);

            // 详细 execCommand 在 其他便签
        }
        catch (err) {
            succeeded = false;
        }
        // succeeded == false
        this.handleResult(succeeded);
    }
        /**
     * Fires an event based on the copy operation result.
     * @param {Boolean} succeeded
     */
    handleResult(succeeded) {
        // this.emitter 
        // Emitter.emit 类  
        // 触发一个命名的事件
        // succeeded == false
        
        this.emitter.emit(succeeded ? 'success' : 'error', {
            action: this.action,
            text: this.selectedText,
            trigger: this.trigger,
            clearSelection: this.clearSelection.bind(this)
        });
    }
```

[传递 `删除函数 destroy`](./README.md#L321)

or [源码--](./clipboard.js-master/src/clipboard.js#111)

``` js
    /**
     * Destroy lifecycle.
     */
    destroy() {
        this.removeFake();
    }
```

## 其他

- [tiny-emitter 小于1](#https://github.com/scottcorgan/tiny-emitter)

- [document.execCommand- 命令](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand)

### 选定文本

select 库

``` js
import select from 'select';
```
[https://github.com/zenorocha/select](#https://github.com/zenorocha/select)

## 类的属性

使用 `set` 初始化 变量

示例

[http://jsbin.com/jogewib/edit?js,console](http://jsbin.com/jogewib/edit?js,console)

除了 `action` , 还有 `target`

``` js
    /**
     * Sets the `target` property using an element
     * that will be have its content copied.
     * @param {Element} target
     */
    set target(target) {
        if (target !== undefined) {
            if (target && typeof target === 'object' && target.nodeType === 1) {
                if (this.action === 'copy' && target.hasAttribute('disabled')) {
                    throw new Error('Invalid "target" attribute. Please use "readonly" instead of "disabled" attribute');
                }

                if (this.action === 'cut' && (target.hasAttribute('readonly') || target.hasAttribute('disabled'))) {
                    throw new Error('Invalid "target" attribute. You can\'t cut text from elements with "readonly" or "disabled" attributes');
                }

                this._target = target;
            }
            else {
                throw new Error('Invalid "target" value, use a valid Element');
            }
        }
    }

        /**
     * Gets the `target` property.
     * @return {String|HTMLElement}
     */
    get target() {
        return this._target;
    }
```

对于 Clipboard 作者来说

 `set`的作用似乎是`初始化变量`，以及 对 `变量的错误处理`