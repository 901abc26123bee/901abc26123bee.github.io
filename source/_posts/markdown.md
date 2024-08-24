---
title: markdown
date: 2020-10-09 20:27:12
categories: Blog
tags:
- Markdown
- Blog
---
# Markdown標題
Using [Typora Editor](https://typora.io/)

# 大標題：

```
#＋空格＋標題＋enter	(beware of that '#' must type in English)
```

## 二級標題

```
##＋空格＋二級標題＋enter
```

### 三級標題

```
###＋空格＋二級標題＋enter
```
<!--more-->

## 字體

**Hello World!**

```
**Hello World!**  #粗體字	
#(beware of that '*' must type in English)
```

*Hello World!*

```
*Hello World!* 	#斜體字
```

***Hello World!***

````
***Hello World!*** 	#粗體＋斜體
````

~~Hello World!~~

```
~~Hello World!~~	#刪除線
```

## 引用

> 

```
>＋空格		#引用
```

## 分割線

---

***

```
---+Enter		#分割線
#or
***+Enter		#分割線
```

## 連結、圖片
image from online ---
![圖片名稱](online path)

image from Destop ---
直接把圖檔拉到Typora中要放的位置，會顯示：
![image-name](image path from desktop)
建議修改成相對路徑
```

├── file for image
│   └── imagename.png
└── Markdown file

---> ![image-name](./image/imagename.png)
```
```
"." 代表當前所在目錄，相對路徑
".." 代表上一層目錄，相對路徑。
"../../" 代表的是上一層目錄的上一層目錄，相對路徑
"/" 代表根目錄,絕對路徑

在使用相對路徑時，我們用符號“.”來表示當前目錄；用符號“..”來表示當前目錄的父目錄。
```

[文章名稱](文章路徑)
```
[文章名稱]((文章路徑)
```



## #列表

1. a
2. b
3. c

```
number+.+空格
enter+enter = finish列表	#only work when there is no word urder

ex:
	1. apple
	2. orange
		 enter+enter --> has word urder this line --> won't work	 

		 I won't let you stop the list
	3. enter+enter --> no word urder --> finish the list
next line start here

```

- a
- b
- c

```
-+空格
enter+enter = finish列表	#only work when there is no word urder
```

## 表格

方法ㄧ：

1|2|3
--|--|--|
a|b|c|

1|2|3|

--|--|--|

a|b|c|

檢視 --> 原始碼模式 --> 刪除行內空格



方法二：

段落 ---> 插入表格

## code

```java

```

```
···（最左上）＋程式語言+Enter
```

## 文字調亮

Markdown不支持，採用HTML

<span style="color:red">some **This is Red Bold.** text</span>

<span style="color:orange">some *This is Blue italic.* text</span>

<mark>Marked text</mark>

<span style="background-color: #FFFF00">Marked text</span>

HTML Color Names https://www.w3schools.com/colors/colors_names.asp

