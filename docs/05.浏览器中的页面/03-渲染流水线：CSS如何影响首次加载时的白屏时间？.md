# 渲染流水线：CSS 如何影响首次加载时的白屏时间？

CSS 是页面的核心资源,决定页面最终显示效果,影响网站的第一体验;

## CSSOM 的作用

1. 渲染引擎无法理解 CSS,需要将 CSS 转换为 CSSOM
2. 提供给 JavaScript 操作样式表的能力
3. 为合成布局树提供基础样式

## 渲染流水线视角下的 CSS

```css
/* theme.css */
div {
  color: coral;
  background-color: black;
}
```

```html
<!-- 外部CSS示例 -->
<html>
  <head>
    <link href="theme.css" rel="stylesheet" />
  </head>
  <body>
    <div>渲染流水线视角下的 CSS</div>
  </body>
</html>
```

![含有CSS的页面流水线](../images/05/dom_css.png)

1. 渲染进程或浏览器进程发起主页面请求,请求派发到网络进程中执行
2. 网络进程接收到请求,发起请求并将结果返回给渲染进程(网络进程下载到响应数据中间有一段空闲时间)
3. 渲染进程接收到网络进程发送的数据,开启一个预解析进程,预解析进程判断当前文档存在外部 JavaScript 文件或 CSS 文件,会提前下载这些文件
4. DOM 构建完成,结合 CSSOM 生成布局树(如果 DOM 构建完成,CSS 文件未下载,也将会存在空闲时间,因为下一步是构建布局树,需要 CSSOM)

```css
/* theme.css */
div {
  color: coral;
  background-color: black;
}
```

```html
<!-- 外部CSS和JavaScript示例 -->
<html>
  <head>
    <link href="theme.css" rel="stylesheet" />
  </head>
  <body>
    <div>渲染流水线视角下的 CSS</div>
    <script>
      console.log("渲染流水线视角下的 CSS")
    </script>
  </body>
</html>
```

![含有CSS和JS的页面流水线](../images/05/dom_css_js.png)

1. 渲染进程或浏览器进程发起主页面请求,请求派发到网络进程中执行
2. 网络进程接收到请求,发起请求并将结果返回给渲染进程(网络进程下载到响应数据中间有一段空闲时间)
3. 渲染进程接收到网络进程发送的数据,开启一个预解析进程,预解析进程判断当前文档存在外部 JavaScript 文件或 CSS 文件,会提前下载这些文件
4. 遇到 JS,HTML 解析器暂停,因为 JS 有操作 CSS 的能力,所以需要等待 CSSOM 构建完成(等待 CSS 下载和构建存在空闲时间)
5. CSS 文件下载文成,CSS 解析器解析 CSS 为 CSSOM,开始执行 JavaScript 代码,执行完毕,继续 DOM 解析
6. DOM 解析完成,开始构建布局树

```css
/* theme.css */
div {
  color: coral;
  background-color: black;
}
```

```js
//foo.js
console.log("外部CSS和外部JavaScript示例")
```

```html
<!-- 外部CSS和外部JavaScript示例 -->
<html>
  <head>
    <link href="theme.css" rel="stylesheet" />
  </head>
  <body>
    <div>渲染流水线视角下的 CSS</div>
    <script src="foo.js"></script>
  </body>
</html>
```

![含有外部CSS和JS的页面流水线](../images/05/dom_css_js2.png)

1. 渲染进程或浏览器进程发起主页面请求,请求派发到网络进程中执行
2. 网络进程接收到请求,发起请求并将结果返回给渲染进程(网络进程下载到响应数据中间有一段空闲时间)
3. 渲染进程接收到网络进程发送的数据,开启一个预解析进程,预解析进程判断当前文档存在外部 JavaScript 文件或 CSS 文件,会提前同时下载这些文件
4. CSS 文件下载文成,CSS 解析器构建 CSSOM,JS 文件下载完成(如果 JS 提前下载完成,需要等待 CSSOM 的构建),开始执行 JS
5. JS 执行完毕,继续构建 DOM,DOM 构建完毕,开始构建布局树

## 印象页面展示因素和优化策略

从发起 URL 请求到页面展示内容,视觉上经历三种阶段

1. 发起请求,到提交数据阶段,页面展示上一个页面的内容
2. 提交数据后,渲染进程创建一个空白页面(解析白屏时间),等待 CSS 文件和 JS 文件下载,生成 CSSOM,执行 JS,生成 DOM,然后合成布局树,直到准备首次渲染
3. 首次渲染完成,逐渐生成完整页面

第一阶段主要受到网络或服务器的影响

1. 压缩 HTML,CSS,JS 文件,去除注释
2. 使用 CDN,减少单个域名请求限制,或使用 HTTP2
3. 使用缓存,加速下次请求速度

第二阶段主要问题是白屏问题

白屏时间主要分为,解析 HTML,下载 CSS,下载 JS,生成 CSSOM,执行 JavaScript,生成 DOM,生成布局树,绘制页面等任务

解决白屏问题主要体现在下载 CSS 文件,下载 JS 文件,执行 JS

1. 内联 CSS 和 JS 文件,避免网络进程下载这些问题(小项目)
2. 压缩 CSS 和 JS 文件大小,移除注释
3. 异步加载 JavaScript(async,defer)
4. 将较大的 CSS 文件,通过媒介查询拆分为不同用途的 CSS 文件
