---
title: CSS
categories:
  - Front-End
  - CSS
---

## CSS

### CSS 基础

#### 引入方式 to include CSS

- 行内式 inline style - 最高优先级
  - `<h1 style="..."/>`
- 内部样式表 internal stylesheet
  - 写在 head
  - `h1 {color: ...}`
- 外部样式表 external stylesheet
  - 写在 head
  - `<link rel="stylesheet" href="./dir"/>`

#### 选择器 selectors

都放到 head 里面的 style. 优先级: id -> class -> tag. 加`!important` after property 强制优先

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
- 其他
  - ![[Screenshot 2024-09-18 at 8.41.06 PM.png|600]]

#### 文本属性 text properties

- color - RGB, 十六进制
- line-height - px, em, %
- text-align - left, right, center
- text-indent - em
- text-decoration - text-decoration: ....

#### 背景样式 background-xxx

- color
- image
- repeat - 平铺方式
- size
- attachment - 附加方式 scroll / fixed
- position - 根据 x/y 设置平铺位置/方式

### CSS 进阶

#### Box

- border - 边线
  - 边框 width, style, color..
  - 盒子的大小
- margin - 边外距
  - 上右下左, 自动
    - 自动: 居中
  - 盒子往*外*的距离
- padding - 边内距
  - 和 margin 相似
  - 盒子往*内*的距离
- content-box: 默认属性 + width 只代表 content
- border-box: 默认属性 + width 代表 content + padding + border 的长度

#### Float

脱离标准流, 不占位置, 只能左右浮动. 浮起来的流也会“排队”

- left
- right
- **清楚浮动 - overflow, 使页面不要错乱**

#### Position

**子级是 absolute 的话, 父级要用 relative**. 起点位置只需要在更改位置时考虑

- relative 起点位置是**自己的左上角**, 移动后不会影响其他标准流 (别人不知道)
- absolute 起点位置是**parent 左上角**, 完全拖标, 会影响其他元素
- fixed 固定在屏幕位置, 完全拖标. scroll 不会弄走
- static 默认

#### Flex 伸缩布局

- flex:number 控制比例
- flex-direction 横向/纵向
- flex-wrap 是否控制单行
- align-items 内部元素位置
- align-content
