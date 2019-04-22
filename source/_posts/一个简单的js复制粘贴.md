---
title: 一个简单的js复制粘贴
date: 2019-03-15 09:41:12
tags:
---

## 原生js实现点击按钮复制文本

最近遇到一个需求，需要点击按钮，将相应内容复制到剪切板


原理分析
-----------

浏览器提供了copy命令，可以复制选中的内容

```````js
document.execCommand('copy');

```````

如果是输入框，可以通过 `select()` 方法，选中输入框的文本，然后调用  `copy` 命令，将文本复制到剪切板
但是 select() 方法只对 `<input>` 和 `<textarea>` 有效。

最后我的解决方案是，在页面动态中添加一个 `<textarea>` ，然后把它隐藏掉（这里的隐藏指的是位置，图层，透明度的隐藏，`非display: none;` 否则使用select()选择不了标签）


```````
点击按钮时，动态创建一个 <textarea>，将需要复制的内容复制给 <textarea>，在使用select()进行内容选择

```````

代码实现
-----------

HTML部分

``````html
<div class="wrapper">
    <button onclick="copyText('你想要复制的内容')">点击复制</button>
</div>

``````

JS部分

``````js
<script>
    function copyText(str) {
        var input = document.createElement('textarea');
        input.style.cssText = 'position: absolute; top: 0; left: 0; opacity: 0; z-index: -10;';
        input.value = str;
        document.body.appendChild(input);
        input.select();
        document.execCommand('copy');
        alert('复制成功');
        document.body.removeChild(input);
    }
</script>

``````