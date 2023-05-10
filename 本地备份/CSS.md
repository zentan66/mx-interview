- 样式覆盖如何处理
- width:100%和width:**auto**有什么区别？
- `space-between` 和 `space-around` 区别，是否能调整他们间隙的值？
- `align-content` 和 `justify-content` 区别？
- less 和 sass 什么区别？



#### `#` 常见的水平垂直居中实现方案

1. absolute+margin 定宽高情况
2. absolute+transform 定宽高情况
3. flex
4. grid



#### `#` flex：1是哪些属性的缩写，对应的属性代表什么含义

flex:1在浏览器查看中分别是flex-grow（对应元素的增长系数）、flex-shrink（指定了元素的收缩规则，只有在所有元素的默认宽度之和大于容器宽度时才会触发）、flex-basis（指定对应元素在主轴上的大小）







## less

### 函数写法

```less
.pxToVW(@px, @attr: width) {
	@vw: (@px / 375) * 100;
  @{attr}: ~"@{vw}vw";
}
.box {
  .pxToVW(75);
}

```

