# 优化

## 重排,重绘与合成

重排: 更新元素的几何(位置)属性

```js
document.querySelect("div")[0].style.height = 200
```

![重排](./layout.png);

重排会触发`Style`流程及其之后的所有流程,重排需要更新完成的渲染流水线,开销较大;

## 重绘

重绘: 更新元素的绘制属性

```js
document.querySelect("div")[0].style.backgroundColor = "#00ab84"
```

![重绘](./paint.png)

重绘会触发主线程的`Style`和`Paint`流程,及其之后非主线程的所有流程,省去了`Layout`和`Layer`流程,执行效率比重排要高一些;

## 合成

合成: 修改元素的合成属性(不会触发布局,分层,和绘制流程)

```js
document.querySelect("div")[0].style.transform = "scale(1.2)"
```

![合成](./composition.png)

合成会触发主线程`Style`流程,以及非主线程所有流程,省去了`Layout`和`Layer`,`Paint`流程,大大提升了绘制效率;

## 减少重排重绘

1. 使用 class 操作样式，而不是频繁操作 style(DOM 样式集中修改)
2. 避免使用 table 布局
3. 批量 dom 操作,dom 离线操作(display:none; documentFragment, clone & replace)
4. Debounce window resize 事件
5. 对 dom 属性的读写要分离
6. will-change: transform 做优化
7. 减少不必要的 DOM 深度
8. 尽可能减少 CSS 规则的数量，并移除未使用的 CSS 规则
9. 如果您想进行复杂的渲染更改（例如动画）,可以使用 position-absolute 或 position-fixed 来实现此目的(脱离文档流,避免后续大量的重排)。
10. 避免使用不必要且复杂的 CSS 选择器（尤其是后代选择器）
