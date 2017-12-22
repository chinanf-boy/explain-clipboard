# clipboard js

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

[一键复制文本-->网址](https://clipboardjs.com/)

版本 `「 1.7.1 」`

[<div style="text-align:right">chrome -clipboard 插件</div>](https://chrome.google.com/webstore/detail/codecopy/fkbfebkcoelajmhanocgppanfoojcdmg)

[<div style="text-align:right">firefox -clipboard 插件</div>](https://addons.mozilla.org/en-US/firefox/addon/codecopy/)

---

## 安装

通过 npm 获得

```
npm install clipboard --save
```


``` js
<script src="dist/clipboard.min.js"></script>
```

``` js
new Clipboard('.btn');
```


<a href="https://clipboardjs.com/#example-target"><img width="473" alt="example-2" src="https://cloud.githubusercontent.com/assets/398893/9983467/a4946aaa-5fb1-11e5-9780-f09fcd7ca6c8.png"></a>


``` html
<!-- Target -->
<input id="foo" value="https://github.com/zenorocha/clipboard.js.git">

<!-- Trigger -->
<button class="btn" data-clipboard-target="#foo">
    <img src="assets/clippy.svg" alt="Copy to clipboard">
</button>
```

---

## `explain` 开始

本目录

- [绑架 .btn listenClick 监听器](#点击监听器)

>被绑架的 .btn放一边, 看配置

- [resolveOptions 配置选项](#配置选项)

- [监听器定义-抢救.btn行动](./README.ClipboardAction.md)

- [有始有终-destroy](#删除函数)

- [有关Clipboard测试和自动部署CI](#检测构建)

> 彩蛋

- [isSupportsed-是否支持函数](#isSupportsed)

- [Emitter - 事件注册](#Emitter)

- [listen - 监听](#listener)

---

## 点击监听器
我们从

`new Clipboard('.btn');` 开始

`.btn` 代表 `class="btn"` 的 `button` 元素

让我们看看它做了什么

[clipboard.js-master/src/clipboard.js](clipboard.js-master/src/clipboard.js#L14)

``` js
class Clipboard extends Emitter {
    /**
     * @param {String|HTMLElement|HTMLCollection|NodeList} trigger
     * @param {Object} options
     */
    constructor(trigger, options) { // trigger == .btn 
        super();

        this.resolveOptions(options);
        this.listenClick(trigger); // 来到这里
    }
```

[clipboard.js-master/src/clipboard.js](clipboard.js-master/src/clipboard.js#L33)

``` js
    /**
     * 增加 一个 点击事件监听器 到 过滤的目标
     * Adds a click event listener to the passed trigger.
     * @param {String|HTMLElement|HTMLCollection|NodeList} trigger
     */
    listenClick(trigger) {
        this.listener = listen(trigger, 'click', (e) => this.onClick(e));
    }
```

三个点

- `this.listener` 存储

- `listen `函数

``` js
import listen from 'good-listener';
```

- `this.onClick(e)` 事件绑定 e == 元素

``` js
    onClick(e) {

        // 绑定好
        const trigger = e.delegateTarget || e.currentTarget;

        if (this.clipboardAction) {
            this.clipboardAction = null;
        } // 如果存在，清空

        // 重建
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

> ClipboardAction

``` js
import ClipboardAction from './clipboard-action';
```

那么，我们 在 初始化

`new Clipboard('.btn');` 开始 `.btn` 流动就结束了，

它被 Clipboard 绑定在了 click 的事件监听器上，并把相关逻辑，给了

`import ClipboardAction from './clipboard-action';` 

---

[<div style="text-align:right">⬆️目录，目录是谁，我怎么知道</div>](#目录)

next

## 配置选项

我们先把 被绑架的 `.btn` 放一边，看看 `this.resolveOptions(options);`

``` js
class Clipboard extends Emitter {
    /**
     * @param {String|HTMLElement|HTMLCollection|NodeList} trigger
     * @param {Object} options
     */
    constructor(trigger, options) { // trigger == .btn 
        super();

        this.resolveOptions(options); //<--- 这个
        this.listenClick(trigger);
    }
```

``` js
    /**
     * Defines if attributes would be resolved using internal setter functions
     * or custom functions that were passed in the constructor.
     * @param {Object} options
     */

    // options -就是--> {}，在本次说明中。
    resolveOptions(options = {}) {
        this.action    = (typeof options.action    === 'function') ? options.action    : this.defaultAction;
        this.target    = (typeof options.target    === 'function') ? options.target    : this.defaultTarget;
        this.text      = (typeof options.text      === 'function') ? options.text      : this.defaultText;
        this.container = (typeof options.container === 'object')   ? options.container : document.body;
    }
```

所以，上面代码的 `options` 都是 `default`

``` js
this.action    = this.defaultAction
this.target    = this.defaultTarget
this.text      = this.defaultText
this.container = document.body
```

- `this.defaultAction`

``` js
    /**
     * Default `action` lookup function.
     * @param {Element} trigger
     */
    defaultAction(trigger) {
        return getAttributeValue('action', trigger);
    }
```

- `this.defaultTarget`

``` js
    /**
     * Default `target` lookup function.
     * @param {Element} trigger
     */
    defaultTarget(trigger) {
        const selector = getAttributeValue('target', trigger);

        if (selector) {
            return document.querySelector(selector);
        }
    }
```
- `this.defaultText`
``` js
    /**
     * Default `text` lookup function.
     * @param {Element} trigger
     */
    defaultText(trigger) {
        return getAttributeValue('text', trigger);
    }
```

看来都指向一个函数 `getAttributeValue`

[getAttributeValue --- 点击](clipboard.js-master/src/clipboard.js#L125)

``` js
/**
 * Helper function to retrieve attribute value.
 * 帮助 函数去 识别 活动的值
 * @param {String} suffix
 * @param {Element} element
 */
function getAttributeValue(suffix, element) {
    const attribute = `data-clipboard-${suffix}`;

    // 前缀都是 `data-clipboard-` 的元素属性


    if (!element.hasAttribute(attribute)) {
        //没有那个值，返回 null
        return;
    }
        // 返回值
        return element.getAttribute(attribute);
}
```

到这里，我们重新表达一下, 配置

- `action`

``` js
this.action  => this.defaultAction => getAttributeValue('action', trigger)
// 变
this.action(trigger) = getAttributeValue('action', trigger)
```

- `target`

``` js
this.target(trigger) = 

const selector = getAttributeValue('target', trigger);

return document.querySelector(selector);

```

- `text`

``` js
this.text  => this.defaultText => getAttributeValue('text', trigger)
// 变
this.text(trigger)  = getAttributeValue('text', trigger)

```

`this.container = document.body`

---

next

我们把配置也加上去了，那么现在我们可以去找 `.btn` 了，

`waitting` == 

还有个 isSupported `是否支持 clipboard的功能测试噢`,

当然，你着急可以先一步噢[-->抢救.btn点击事件行动](./README.ClipboardAction.md)

[<div style="text-align:right">⬆️目录，目录是谁，我怎么知道</div>](#目录)

---

next 

## 删除函数

``` js
    /**
     * Destroy lifecycle.
     */
    destroy() {
        this.listener.destroy(); // 监听 去掉

        if (this.clipboardAction) {
            this.clipboardAction.destroy(); // action 元素和事件 去掉
            this.clipboardAction = null;
        }
    }
```

---

next

那么我们看彩蛋 

## isSupportsed

``` js
    /**
     * Returns the support of the given action, or all actions if no action is
     * given.
     * @param {String} [action]
     */

    // 静态函数，可以 Clipboard.isSupported 被人使用
    static isSupported(action = ['copy', 'cut']) {
        // 默认测两个 copy cut
        const actions = (typeof action === 'string') ? [action] : action;

        let support = !!document.queryCommandSupported; // 从英文上看，查询命令支持？

        actions.forEach((action) => {
            support = support && !!document.queryCommandSupported(action);
        });

        // support = support && !!document.queryCommandSupported(action);
        // 这行 support = support 很关键     
        // 一旦 support == false
        // 那么这个值就一直是 false
        return support;
    }
```

那么，Done or

[-->抢救.btn点击事件行动](./README.ClipboardAction.md)

---

# 其他引用库

---

## Emitter

``` js
import Emitter from 'tiny-emitter';
```

`Emitter` 用在了 `Clipboard` 的 继承上


``` js
class Clipboard extends Emitter {

```

---

## listener

``` js
import listen from 'good-listener';

```

`listener` 用在了 Clipboard `this.listener` 上

``` js
    listenClick(trigger) {
        this.listener = listen(trigger, 'click', (e) => this.onClick(e));
    }

```

[<div style="text-align:right">⬆️目录，目录是谁，我怎么知道</div>](#目录)

---