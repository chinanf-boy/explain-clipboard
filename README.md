# clipboard js

[![explain](./minilogo.svg)](https://github.com/chinanf-boy/Source-Explain)

[一键复制文本-->网址](https://clipboardjs.com/)

版本 `「 1.7.1 」`

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

- 

---

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