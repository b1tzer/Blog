# [简介](#Overview)
本文章为快速入门教程，介绍一些基本用法，可以立马使用起来。如果需要详细的介绍可以访问[官网](https://www.markdownguide.org/)

## [基础语法](#BasicSyntax)
以下是[John Gruber](https://zh.wikipedia.org/wiki/%E7%B4%84%E7%BF%B0%C2%B7%E6%A0%BC%E9%AD%AF%E4%BC%AF)(Markdown发布格式的发明者)在原始设计文件中的元素。所有Markdown应用程序都支持这些元素。

| Element | Description |
| :-----: | ------ | 
| 一级标题</br>二级标题</br>二级标题 | # text</br>## text</br>### text</br> |
| 加粗 | \*\*text\*\* |
| 斜体 | \*text\* | 
| 引用块 | > blockquote | 
| 有序列表 | 1. list1</br>2. list2 | 
| 无序列表 | - text1 </br>- text2 | 
| 代码 | \`code\`| 
| 水平分割线 | \-\-\- | 
| 链接 |[text]\(https://www.example.xxx\) |
| 图片 | ![alt text]\(image.jpg) |


## [扩展语法](#ExtendedSyntax)

这些元素是额外添加的功能，***并非所有Markdown应用都支持***。


### [表格](#Table)
形如
:  
\| Syntax | Description |  
\| ----------- | ----------- |  
\| row1 | Title |  
\| row2 | Text |
示例：
| Syntax | Description |  
| ----------- | ----------- |  
| row1 | Title |  
| row2 | Text |

### [代码块](#FencedCodeBlock)
形如：
\```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}  
示例：
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

### [脚注](#Footnote)
形如：
Here's a sentence with a footnote.\[^1]

\[^1]: 注脚在文章底部.

示例：
Here's a sentence with a footnote.[^1]

[^1]: 注脚在文章底部.

### [标题ID](#HeadingID)

形如：
\### My Great Heading {#heading-id}

示例：
### My Great Heading {#heading-id}

### [定义](#DefinitionList)
形如：
term
\: definition

示例：
term
: definition

### [删除线](#Strikethrough)
形如：
\~~The world is flat.~~  

示例：
~~The world is flat.~~

### [任务清单](#TaskList)

形如：
\- [x] Write the press release
\- [ ] Update the website

示例：
- [x] Write the press release
- [ ] Update the website


## [下载](#Download)
你可以[下载](https://www.markdownguide.org/assets/markdown-cheat-sheet.md)这份快速手册到本地，方便您随时查阅。