---
title: CSS
categories:
  - Front-End
  - CSS
---

## CSS

### 1. Normal Flow

HTML 元素的默认排列方式

- `display: inline-block | block | inline | none; // 隐藏`
- NOT using `float`, `position: absolute`, `display: flex / grid`...

#### 1.1 排列规则 - Block elements

- 独立区域: Occupy one or more rows, 有 height, length, align...
- **纵向排列, 占满整行**: 网页布局和网页结构的搭建
- `div`, `p` (段落), `h1, h6` (header 标题从 1-6 从大到小)
- 列表 - `li` (行); `ul` (无序, bullet point); `ol` (有序, 一二三)
- 自定义列表 dl - dt / dd

#### 1.2 排列规则 - Inline elements

- 无独立区域: (一行内), 有多少占多少. 没有 height, length, align...
- **横向排列, 不换行**: 常用于控制页面中文本的样式
- `a` (links, interactive), `span` (content), `strong` (加粗)

#### 1.3 Inline-block

对于 inline, 可以设置 height, length, align 等属性

- `img`, `input`, `td` (表格标签 - 一行里面的一个 column, 被包裹到 tr 里面)

### 2. 盒子模型 Box - Block elements

- 所有元素的表示单位为 Box
- `content`
  - 内容 - Text & image
- `padding`
  - 内边距 - 空白, 不能 < 0
- `margin`
  - 外边距 - 空白
- `border`
  - 边框 - 粗细, 样式, 颜色

#### 2.1 Box-sizing

`box-sizing: content-box | border-box | inherit`

- `content-box`: = W3C 标准盒子 (default)
  - 总长度 = margin + padding + boarder + width (or height)
  - `content-box.width` = content width
- `border-box`: = IE 怪异盒子
  - 总长度 = margin + height (or width)
  - width/height 包含 padding + border
  - `border-box.width`= width (content + padding + border)
- `inherit`: same as parent's value

### 3. Overflow

当内容**超出**元素盒子(容器)大小时的显示

```css
overflow: visible; /* 默认值，超出也会显示 */
overflow: hidden; /* 超出部分会被裁剪掉 */
overflow: scroll; /* 总是显示滚动条（不管内容有没有超出） */
overflow: auto; /* 有超出时才显示滚动条 */
```

### 4. Float

Place an element to the left or right of its container

- `float` & `box`: box 会纵向堆叠, 搭配 float 就可以解决布局问题
- if child is `float`
  - Collapse:
    - child are invisible to parent
    - `.parent`: height = 0, 像是“消失了”
    - 看起来像是子元素“跑到外面去了”
  - Parent“重新感知” float child:
    - `.parent {overflow: hidden | auto}`
    - `overflow: hidden | auto`
- 尽量**向左或向右**移动, 直到它的外边缘碰到包含框或另一个浮动框的边框为止
- 脱离标准流 (Normal Flow): 不会占据原本的位置
- 不占位置: 位置不会被 child 识别 - 可能页面布局错乱

### 5. Position

if child is absolute, parent needs relative

- 起点位置只需要在更改位置时考虑
- `relative` 起点位置是**自己的左上角**
  - 移动后不会影响其他标准流 (别人不知道)
  - 兄弟元素还是按照原本空间排列
- `absolute` 起点位置是**parent 左上角**
  - 完全拖标: "盖"在其他元素上
  - 类似`float`: 不留位置, 不会影响其他元素
  - 需要`parent.position = relative`
- `fixed` 固定在屏幕位置, 完全拖标
  - 不随滚动
- `static` 默认
- `z-index` 只有`position != static`才会生效
  - 设定`position = static`
- `sticky` “粘”在屏幕上
  - 需要设置`top/left/right/bottom`
  - 不能`parent.overflow: hidden` / 没有滚动容器

### 6. Flex 伸缩布局

1D 布局: 横/竖排地排列元素

- **快速实现居中对齐、间距自动分配、响应式布局**
- 伸缩要 flex, 主轴靠 justify, 交叉轴靠 align
  - 主轴: 横竖
  - 交叉轴: 垂直 水平

#### 6.1 核心属性 - 加在 parent 属性中

- `display: flex` flex 容器
- `flex-direction: row | column` 主轴方向
- `flex-wrap: norap | rap` 是否换行
- `justify-content: flex-start | center | space-between` 主轴对齐方式
- `align-items: stretch | center` 交叉轴对齐方式
- `align-content: center | space-around` 多行整体交叉轴对齐

#### 6.2 核心属性 - 加在 child 属性中

- `flex` 宽度比例
- `flex-grow` 空间可扩展比例
- `flex-shrink` 收缩比例
- `flex-basis` 初始 width / height
- `align-self` 自己在交叉轴的对齐方式

### 7. BFC - Block formatting context

- 渲染规则
  - 垂直放置内部盒子
  - 相同 BFC 中, margin 会发生重叠, 与方向无关
  - 每个元素的 left margin 和它的 left border 接触 (左到右), including floating element
  - BFC 区域不和 float element 重叠
  - BFC 高度包含 float element
  - BFC 是隔离容器, 里外互不影响

#### 7.1 Trigger condition

- `Root: HTML`
- `Float: float value = left | right`
- `overflow = auto | scroll | hidden | != visible`
- `display = inline-block | table-caption | table | inline-table | flex | inline-flex | grid | inline-grid`
- `position = absolute | fixed`

#### 7.2 Application 应用场景

防止 margin 重叠 (塌陷)

````jsx
<>
	<p />
	<div overflow=hidden> // 生成一个BFC - 两个p不属于同一个BFC
		<p />
	</div>
</>
	```
清除内部浮动
```jsx
<div overflow=hidden> // 生成BFC, 内部元素高度影响当前div, 包括浮动元素
	<div float=left/>
	<div float=left/>
</div>
````

自适应多栏布局

```jsx
<div>
	<div float=left/>
	<div overflow=hidden/> // 生成BFC, 不会和浮动盒子重叠
</div>
```

### 8. 响应式设计 Responsive Web design

dynamic: `Content is like water`

- 适配 PC + 平板 + 手机...

#### 8.1 实现方式

`viewport`: 应对不同屏幕尺寸

- `<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">`
  - `width=device-width`: 自适应屏幕尺寸宽度
  - `initial-scale`: 缩放初始化
  - `maximum-scale`: 缩放比例最大值
  - `user-scalable`: 用户可以的缩放操作

#### 8.2 方式 - 媒体查询 `@media`

- 类似`if`
- 不同分辨率配备不同的布局

```jsx
@media screen and (max-width: 1920px) {...}

@media screen and (min-width: 375px) and (max-width: 600px) {
	body {
		font-size: 18px;
	}
}
```

#### 8.3 方式 - 百分比 `%`

`height`, `width`属性依赖于 parent, 其他盒子不完全依赖于 parent

- 不建议实现响应式: 如果每个属性都用百分比, 会造成布局复杂度
- child `top/left` / `bottom/right/`: 相对于非 static 定位的 parent 高度/宽度
- child `padding / margin`: 相对于 parent 的 width, 无关 height
- `border-radius`: 相对于自身的宽度

#### 8.4 方式 - vw/vh

x = x% of viewport width/height

- 和%类似, 但是相应更快
- `vw`: viewport width
- `vh`: viewport height

### 9. Import & Usage CSS

- 行内式 inline style
  - 最高优先级
  - `<h1 style="..."/>`
- 内部样式表 internal stylesheet
  - 写在 head
  - `h1 {color: ...}`
- 外部样式表 external stylesheet
  - 写在 head
  - `<link rel="stylesheet" href="./dir"/>`

#### 9.1 选择器 selectors

head 里面的 style.

- 优先级: **id -> class -> tag**
- `!important` 强制优先
- 标签选择器
  - `body{color:"..."}`
- 类选择器 - 基于 class, class 可以重复
  - `.className{}`
- id 选择器 - id 不能重复
  - `#idName{}`
- 通配符选择器
  - `*.name{}`
- 并集选择器
  - `div, #idName, .className {}`
- 后代选择器 - 设置 father 里面的 child, 不论 level
  - `father child`
- 子代选择器 - 设置 father 里面的 child, 一层 level
  - `father > child`
- 伪类选择器 - 一般和 a 标签使用, link, visited, hover, active...
  - `a:link {}`
- neighbor selector (同级相邻)
  - `A + 被选择的`
- 后面所有的同级兄弟元素
  - `A ~ 被选择的`

### 10. 水平居中的方法

#### 10.1 定位 + `margin:auto`

```css
.father {
  position: relative;
  width: 200px;
  height: 50px;
}

.son {
  "定位&top&left&right&bottom的原因,son和father宽高相同"position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  "设置宽高,但是虚拟占位已经撑满了father"width: 100px;
  height: 40px;
  "实现居中"margin: auto;
}
```

#### 10.2 定位 + `margin:负值`

- 不需要 parent 高度, 需要自身元素宽高

```css
.father {
  position: relative;
  width: 200px;
}

.son {
  position: absolute;
  top: 50%;
  left: 50%;
  "margin值=宽高/2"margin-left: -50px;
  margin-top: -50px;
  width: 100px;
  height: 100px;
}
```

#### 10.3 定位 + `transform`

- 不需要 parent 高度, 不需要自身元素宽高

```css
.father {
  position: relative;
  width: 200px;
}

.son {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100px;
  height: 100px;
}
```

#### 10.4 `table`

- 利用`display` & `vertical-align` & `text-align`

```css
.father {
  display: table-cell;
  width: 200px;
  vertical-align: middle;
  text-align: center;
}

.son {
  display: inline-block;
  width: 100px;
  height: 100px;
}
```

#### 10.5 `flex` | `grid`

```css
.father{
	display:grid; "or flex"
	width:200px;
	justify-content:center; "元素相对于容器垂直居中"
	align-items:center; "元素相对于容器水平居中"
}

.son{
	width:100px;
	height:100px;
}
```

### 11. 隐藏页面元素

主要是前三个方式

- `display: none`
  - **彻底**消失
  - 导致重排 & 重绘
  - 不可见, 不占空间, 不响应
- `visibility: hidden`
  - DOM 仍存在
  - 不重排, 但触发重绘
  - 不可见, 占空间, 不响应
- `opacity: 0`
  - 不重排, 一般触发重绘
  - 不可见, 占空间, 响应
- `height, width, margin, border, padding: 0`
  - 隐藏 child: `overflow: hidden`
  - 不可见, 不占空间, 不响应
- `position: absolute`
  - 不可见, 不影响布局
- `clip-path`
  - 裁剪的形式 `clip-path: polygon(0px, 0px, 0px, 0px, 0px, 0px);`
  - 不可见, 占空间, 不响应

### 12. 文本溢出的省略样式

#### 12.1 单行

- `white-space: nowrap`文本不换行 - 基础
- `text-overflow: clip | ellipsis` 溢出部分裁掉 | 省略标记
- `overflow: hidden` 隐藏内部溢出元素

#### 12.2 多行 - 基于高度截断

兼容性好, 响应式截断

```css
.p {
  position: relative;
  overflow: hidden;
  height: 40px;
  line-height: 20px;
}
.p::after {
  content: "...";
  position: absolute;
}
```

#### 12.3 多行 - 基于行数截断

... Google it :)

### 13. Reflow 回流 & Repaint 重绘

- 回流: 布局引擎计算盒子在页面上的大小 & 位置
- 重绘: 计算好盒子的属性后, 根据盒子特性进行绘制

1. 解析 HTML, 生成 DOM tree
2. 解析 CSS, 生成 CSSOM tree
3. 结合 DOM & CSSOM, 生成 Render Tree
4. Reflow: 根据 Render Tree, 进行 Layout, 得到节点的几何信息(位置, 大小)
5. Repaint: 根据几何信息, 得到节点的绝对像素
6. Display: 发送像素给 GPU, 显示

#### 13.1 Trigger Reflow

- 添加/删除**可见的**DOM 元素
- 元素位置变化
- 元素尺寸变化
- 内容变化
- 第一次渲染
- 浏览器窗口尺寸变化

#### 13.2 Trigger Repaint

Repaint 一定会 Trigger Reflow

- 颜色修改
- 文本方向修改
- 阴影修改

#### 13.3 减少 Reflow

- 尽可能脱离文档流: `position: flexd | absolute`, 减少对其他元素影响
- 避免使用`table`: 每个元素的改动都会导致整个 table 重算
- 离线操作: 设置`display:none`, 操作完成后改回

  ```js
  const box = document.getElementById("box");

  // 离线处理
  box.style.display = "none"; // 先脱离渲染树

  box.style.width = "100px";
  box.style.height = "100px";
  box.style.padding = "10px";

  box.style.display = "block"; // 加回去，一次性 Reflow
  ```

- 离线操作: 克隆

  ```js
  const parent = document.getElementById("parent");
  const child = document.getElementById("child");

  // 克隆一份，修改后再替换
  const clone = child.cloneNode(true);
  clone.style.width = "200px";
  clone.style.height = "200px";

  parent.replaceChild(clone, child);
  ```
