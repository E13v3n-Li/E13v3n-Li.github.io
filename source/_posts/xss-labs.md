---
title: xss-labs
date: 2026-09-03 18:30:59
tags: 
  - web
  - xss
  - xss-labs
categories: 
  - xss	
  - xss-labs
---

# 一、什么是XSS

XSS，全称为跨站脚本攻击（Cross-Site Scripting），因为要和CSS区分开来，故称之为XSS。

XSS是指攻击者将恶意构造的脚本代码（比如恶意的JavaScript代码）注入到网页中，由于Web应用程序对用户输入**缺乏有效的过滤或转义**，导致这些数据被浏览器当作**可执行代码**解析并运行，从而窃取用户敏感信息、劫持会话或篡改页面内容。



# 二、反射型XSS

**反射型 XSS** （**非持久型跨站脚本攻击**）是指攻击者将恶意脚本作为参数附加在 URL 地址、表单提交或 HTTP 请求头中，**提交给服务端**。服务端接收到该请求后，**未对恶意参数进行任何过滤或转义**，直接将包含该恶意脚本的数据拼接进 HTML 响应页面中，并“反射”回给当前浏览器。浏览器在解析该响应页面时，由于无法区分这是“数据”还是“代码”，从而执行了攻击者注入的 JavaScript 脚本。

以xss-labs中的level1为例，先看level1.php中的代码

```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["name"];
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
<center><img src=level1.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
```

可以看到，`$str = $_GET["name"];`，`$str`接受来自GET传参的`name`值。当我们使用GET去传参`name`时，如果使`name=<script>alert(1)</script>`时，便会触发反射型xss，网页出现弹窗`1`。（具体的页面请看下面的level1的贴图）



# 三、存储型XSS



# 四、常见的标签利用及绕过

## htmlspecialchars

`htmlspecialchars` 是 PHP 里用来**把特殊字符转成 HTML 实体**的函数，主要目的是防止输出到 HTML 页面时被浏览器当成标签或脚本执行，从而避免 **XSS 跨站脚本攻击**。

主要转换以下几个字符：

```markdown
		字符	转换后
		&	&amp;
		"	&quot;
		'	&#039;
		<	&lt;
		>	&gt;
```

- **PHP 8.1 之前**：
  `htmlspecialchars($str)` 默认 flags 是 `ENT_COMPAT`，只转义双引号 `"`，**不转义单引号 `'`**。

  php

  ```
  echo htmlspecialchars("'");
  // 输出：'
  ```

- **PHP 8.1 及以后**：
  默认 flags 变成了 `ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401`，这时**默认会转义单引号**。

  php

  ```
  echo htmlspecialchars("'");
  // 输出：&#039;
  ```

- **显式写 `ENT_QUOTES` 时**：
  不管哪个版本，单引号和双引号都会转义。

  php

  ```
  echo htmlspecialchars("'", ENT_QUOTES);
  // 输出：&#039;
  ```



# xss-labs

## level1

![image-20260903185623506](../images/image-20260903185623506.png)

当我们将url中的`name`改为`<script>alert(1)</script>`时，便触发了弹窗。由于xss-labs的前端代码会检测是否有弹窗，若有代码将会弹出`完成的不错！`字样，然后重定向到下一关。如下图

![image-20260903190009601](../images/image-20260903190009601.png)



## level2

首先来看源码

```php+html
1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level3.php?writing=wait"; 
     9	}
    10	</script>
    11	<title>欢迎来到level2</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level2</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    19	<form action=level2.php method=GET>
    20	<input name=keyword  value="'.$str.'">
    21	<input type=submit name=submit value="搜索"/>
    22	</form>
    23	</center>';
    24	?>
    25	<center><img src=level2.png></center>
    26	<?php 
    27	echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
    28	?>
    29	</body>
    30	</html>

```

我们可以看到虽然`htmlspecialchars($str)`过滤了双引号`"`、尖括号`<`等字符（上面第四部分的`htmlspecialchars`有介绍），但是下面还存在一个`<input>`标签，其中`value="'.$str.'"`这个`$str`是未经过滤的。

那么可以考虑从这里来入手，来达到一个弹窗的效果

通过`keyword`去传入参数`" onclick="alert(1)`，便可实现弹窗

这里的payload是通过前一个双引号`"`闭合前面的`value="`，那`onclick="alert(1)`则是为了闭合`input`标签内后面的`">`，达到一个xss注入攻击的效果

那么`keyword`传入`payload`之后，第20行php代码就变成了

`<input name=keyword  value="" onclick="alert(1)">`



那么我们看一下实际操作：

![image-20260912110058271](../images/image-20260912110058271.png)

我们点击搜索后，再次点击对话框即可完成

![image-20260912110154998](./../images/image-20260912110154998.png)



## level3

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level4.php?keyword=try harder!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level3</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level3</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
    19	<form action=level3.php method=GET>
    20	<input name=keyword  value='".htmlspecialchars($str)."'>	
    21	<input type=submit name=submit value=搜索 />
    22	</form>
    23	</center>";
    24	?>
    25	<center><img src=level3.png></center>
    26	<?php 
    27	echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
    28	?>
    29	</body>
    30	</html>
```

分析源码，可以看到18和20行处，都用了`htmlspecialchars`这个函数。由于我们这里的php版本是低于8.1的，所以这里是没有过滤`'`的。那么`payload`可以是`' onclick='alert(1)'`，如此便可实现xss注入。

![image-20260912112301219](./../images/image-20260912112301219.png)

![image-20260912112327205](./../images/image-20260912112327205.png)

但是如果我们要去考虑PHP8.1及之后的版本呢？还有没有其它手法？或者是寄希望于`htmlspecialchars`的`flags`参数的不恰当书写？

## level4

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level5.php?keyword=find a way out!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level4</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level4</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	$str2=str_replace(">","",$str);
    19	$str3=str_replace("<","",$str2);
    20	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    21	<form action=level4.php method=GET>
    22	<input name=keyword  value="'.$str3.'">
    23	<input type=submit name=submit value=搜索 />
    24	</form>
    25	</center>';
    26	?>
    27	<center><img src=level4.png></center>
    28	<?php 
    29	echo "<h3 align=center>payload的长度:".strlen($str3)."</h3>";
    30	?>
    31	</body>
    32	</html>
```

看到源码中17、18、19行是对尖括号`<`和`>`进行了一个过滤，没有进行其它过滤，22行代码和之前没啥区别

尝试`payload=" onclick="alert(1)`，应该就可以顺利通过了

![image-20260912112851367](./../images/image-20260912112851367.png)

## level5

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level6.php?keyword=break it out!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level5</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level5</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = strtolower($_GET["keyword"]);
    18	$str2=str_replace("<script","<scr_ipt",$str);
    19	$str3=str_replace("on","o_n",$str2);
    20	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    21	<form action=level5.php method=GET>
    22	<input name=keyword  value="'.$str3.'">
    23	<input type=submit name=submit value=搜索 />
    24	</form>
    25	</center>';
    26	?>
    27	<center><img src=level5.png></center>
    28	<?php 
    29	echo "<h3 align=center>payload的长度:".strlen($str3)."</h3>";
    30	?>
    31	</body>
    32	</html>
```

看到源码第17、18、19行，过滤了`<script`、`on`等字符





