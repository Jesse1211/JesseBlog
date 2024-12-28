---
title: HTML
categories:
  - Front-End
  - HTML
---

## HTML

#### 网页结构

```html
<!doctype html>
	<html lang="en">
	<head>
		<meta charset="UTF-8" />
		// 适口信息 - 对于不同platform
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />

		<title>页面标签</title>

		// 引入外部标识 + CSS样式 + 脚本
		<link/>
		<script/>
	</head>

	<body>
		具体页面结构
	</body>
</html>
```

#### 标签类型

##### Block

每个 Block 会独自占据 one or more rows, 可以设置 height, length, align 等属性
常用于网页布局和网页结构的搭建

- div
- p - 段落
- h1-h6 - header 标题从 1-6 从大到小
- 列表 (行 - li)
  - 无序 - ul - 都是 bullet point
  - 有序 - ol - 带一二三
- 自定义列表 dl - dt / dd

##### Inline

行内元素, 不占有独立的区域 (都是一行内), 有多少占多少. 不可以设置 height, length, align 等属性
常用于控制页面中文本的样式

- a - links, interactive
- span - content
- strong - 加粗

##### Inline-block

对于 inline, 可以设置 height, length, align 等属性

- img
- input
- td 表格标签 - 一行里面的一个 column, 被包裹到 tr 里面

##### Display: block 和 inline 的转换

```cs
display: inline-block;
display: block;
display: inline;
display: none; // 隐藏
```

#### 表单与表格

##### Form, input, select

- form
  - 点击会出发 API 访问, 一般都是 POST
- input
  - type = text, password,... 选项方式
- select - options
  - 下拉框

##### Table - tr -td

```html
<table>
	<tr>
		<td>1</td>
		<td>1</td>
		<td>1</td>
	</tr>
</table>
```

#### 语意标签

- ![[Screenshot 2024-09-18 at 5.57.12 PM.png|300]]

#### 多媒体 - 通过 src 实现

- audio 音频
- video 视频

#### 本地存储

- localStorage 可以按照时间保存
  - `localStorage.setItem("name", "value");`
  - `localStorage.name = "value";`
  - `localStorage.getItem("name");`
- sessionStorage 关闭浏览器就会丢失 (做法和 localStorage 一样)

#### Status Code

- 1XX 表示消息
- 2XX 表示成功
- 3XX 表示重定向
- 4XX 表示客户端错误
- 5XX 表示服务端错误
