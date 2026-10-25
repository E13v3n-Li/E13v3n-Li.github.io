---
title: XXE
date: 2026-10-25 13:48:36
tags:
  - web
  - xxe
categories:
  - web
  - xxe
---

# 0x01 ~ 前置知识

## a. 什么是 XML

`XML` （Extensible Markup Language），可扩展标记语言。

`XML` 用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型。

> `HTML` 用来显示网页；`XML` 用来保存、传输结构化的数据，标签可以自定义。



## b. XML 的文档结构

`XML` 文档结构包括 `XML` 声明、`DTD` 文档类型定义（可选）、文档元素

举个例子

```xml-dtd
<!-- XML声明 -->
<?xml version="1.0" ?>
<!-- DTD文档类型定义 -->
<!DOCTYPE note [
	<!ELEMENT note (to,from,heading,body)>
	<!ELEMENT to		(#PCDATA)>
	<!ELEMENT from		(#PCDATA)>
	<!ELEMENT heading	(#PCDATA)>
	<!ELEMENT body		(#PCDATA)>
]>
<!-- 文档元素 -->
<!-- 根元素为 <note> 必须要和 DTD 文档类型定义中的根元素一致>
<note>
    <to>George</to>
    <from>John</from>
    <heading>Reminder</heading>
    <body>Don't forget the meeting!</body>
</note>
```



## c. XML 基本语法规则

1、`XML` 声明：`<?xml version="1.0" ?>` 放在文档第一行

2、唯一根目录：整篇文档只有一个根元素（如 `b.` 中的`<note>`）

3、标签成对闭合：`<to>` 必须有 `</to>`，标签名大小写敏感

4、标签自定义：标签名可以根据需要自行命名

5、属性：标签内可以带属性，如`<to id="1">`



## d. DTD 概念

`DTD（document type definition）` 控制 `XML` 的文档的格式规范。

`DTD` 文档类型定义，是一类可定义合法的 `XML` 文档构建模块。（`DTD` 定义了一种对 `XML` 的格式描述，任何一个 `XML` 文件都可以选择引用）

`DTD` 就是长得这个样子。~~直接拿的上面的，为了省事。~~

```dtd
<?xml version="1.0" ?>
<!-- DTD文档类型定义 -->
<!DOCTYPE note [
	<!ELEMENT note (to,from,heading,body)>
	<!ELEMENT to		(#PCDATA)>
	<!ELEMENT from		(#PCDATA)>
	<!ELEMENT heading	(#PCDATA)>
	<!ELEMENT body		(#PCDATA)>
]>
```



> `DTD` 可以在 `XML` 文档内声明，也可以外部引用
>
> 1、内部声明：`<!DOCTYPE 根元素 [元素声明]>`
>
> 内部声明的完整例子就是 b. 中所写
>
> 2、外部声明（引用）：`<!DOCTYPE 根元素 SYSTEM "文件名"> ex:<!DOCTYPE test SYSTEM 'http://www.test.com/evil.dtd'>`
>
> 例子如下：
>
> ```xml
> <?xml version="1.0"?>
> <!DOCTYPE note SYSTEM "note.dtd">
> <note>
> <to>George</to>
> <from>John</from>
> <heading>Reminder</heading>
> <body>Don't forget the meeting!</body>
> </note> 
> ```
>
> 其中 `note.dtd` 的内容为
>
> ```dtd
> <!ELEMENT note (to,from,heading,body)>
> <!ELEMENT to (#PCDATA)>
> <!ELEMENT from (#PCDATA)>
> <!ELEMENT heading (#PCDATA)>
> <!ELEMENT body (#PCDATA)>
> ```



## e. DTD 实体

`XXE` 漏洞就是利用了 `<!ENTITY>` 实体声明，让解析器读取本地文件或访问内网地址。

`DTD` 实体可以看作一个变量，可以在 `XML` 中通过 `&[entity];` 进行引用，如下所示（这就是个 `XXE` 注入的简单利用）

```xml-dtd
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY myfile SYSTEM "file:///etc/passwd">	<!-- 实体定义 -->
]>
<msg>&myfile;</msg>

<!--  &myfile; 引用实体变量，触发本地文件读取。 --> 
```

> `SYSTEM` 关键字：表示引用外部资源（本地文件、HTTP网址）



### 实体的种类

实体可以存一段文本，也可以指向本地文件或网络 `URI` 等外部资源。

实体可以分为 **普通实体** 和 **参数实体**：

| 对比项   | 普通实体                           | 参数实体                        |
| :------- | :--------------------------------- | :------------------------------ |
| 声明语法 | `<!ENTITY 名称 "值">`              | `<!ENTITY % 名称 "值">`         |
| 引用语法 | `&名称;`                           | `%名称;`                        |
| 使用位置 | XML 文档内容、属性值等             | DTD 内部，用于复用 DTD 声明片段 |
| 用途     | 回显型 XXE（读取文件直接返回页面） | 盲 OOB-XXE（无回显外带数据）    |



### 实体的两种声明

1、内部实体声明：

```xml-dtd
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY myfile "这是一段文本">	<!-- 实体定义 -->
]>
<msg>&myfile;</msg>
```

2、外部实体声明：

```xml-dtd
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY myfile SYSTEM "http://xxx.com/">	<!-- 实体定义 -->
    <!ENTITY myfile SYSTEM "file:///etc/passwd">
]>
<msg>&myfile;</msg>
```



# 0x02 ~ XML 注入

简单学习一下 `XML` 注入，举个栗子：

`注入前 XML 代码`：

```xml
<?xml version="1.0">
<USER ROLE="admin">用户输入位置</USER>
```

用户注入恶意代码，如：`User1</USER><USER role="admin">User2`，原 `XML` 代码变为：

```xml
<?xml version="1.0">
<USER role="admin">User1</USER>
<USER role="admin">User2</USER>
```

> 上述例子可以看出 `XML` 其实就是 `XML` 代码直接拼接造成的。



# 0x03 ~ XXE 注入

## a. 原理

`XXE (XML External Entity Injection)` ，即 `XML` 外部实体注入。

当 `XML` 解析器启用了 `DTD` 处理且允许引用外部实体时，攻击者可通过构造恶意 `DOCTYPE` 和 `ENTITY` 声明，引用外部资源，可能导致：

- 读取服务器本地文件（受进程权限限制）；
- SSRF：让服务器向内网发起请求，探测内网端口或访问内网服务；
- DoS：如实体膨胀导致资源耗尽；
- 在特定解析器/环境下，可能进一步导致命令执行，但这不是 XXE 的通用后果。



## b. 触发条件

- 请求报文必须是 `XML` (`Content-Type: application/xml` / `text/xml`)

- 解析器未禁用外部实体/`DTD`（`libxml2` 默认会解析外部实体，Java 部分解析器也默认开启）



# 0x04 ~ 如何检测 XXE

- 通过 HTTP 头的 `Content-Type` 查看是否为 `application/xml`、`text/xml`、`application/soap+xml` 等 XML 类型；也可关注 `wsdl`、`soap`、`xml`、`svg` 等关键字及常见 XML 使用场景。
- 使用 Burp Suite 插件 **Generic XXE Detector** 辅助自动检测 XXE 漏洞。
- 将 GET 请求转换为 POST 请求，在 POST 请求体中重新发送 XML 格式数据，观察服务端是否按 XML 解析。
- 删除 URL 中的查询参数和路径参数，尝试触发服务端接收其他格式输入的模式。
- 添加 `Content-Type: application/xml` 请求头，并在请求体中放入不符合规范的无效 XML，观察是否返回 XML 解析错误信息，以判断后端是否使用 XML 解析器。
- 对无回显的盲 XXE，需要借助公网带外主机、DNSLog 或 Burp Collaborator 等带外通道确认请求是否被解析，并可结合参数实体和外部 DTD 外带数据。



# 0x05 ~ XXE 的利用

--------------------------------------------------------  先欠着 ovo  明天补上 真的 --------------------------------------------------------



# 0x06 ~ XXE 的防御

## 方法一

使用开发语言提供的禁用外部实体的方法

**Java**:

```java
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);
```

**Python**:

```python
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

**PHP**

```php
libxml_disable_entity_loader(true);
```



## 方法二

过滤用户提交的 `XML` 数据

过滤关键词：`<!DOCTYPE>` 和 `<!ENTITY>`，或者 **SYSTEM** 和 **PUBLIC**



# 0x07 ~ 参考文献

https://yinwc.github.io/2018/11/19/XXE%E6%BC%8F%E6%B4%9E%E5%AD%A6%E4%B9%A0/#/DTD%E7%9A%84%E6%A6%82%E5%BF%B5

http://fangqi.xyz/post.html?slug=xml%E6%B3%A8%E5%85%A5-%E4%B8%8E-xxe-xml%E5%A4%96%E9%83%A8%E5%AE%9E%E4%BD%93%E6%B3%A8%E5%85%A5#heading-7

http://3lizabeth.fun/posts/study/xxe_basic/#xxe---%e5%a4%96%e9%83%a8%e5%ae%9e%e4%bd%93%e6%b3%a8%e5%85%a5xml-external-entity-injection

https://teamssix.com/191209-192903#toc-heading-1



# 结语

```text
时间过得好快啊，转眼间又要迎来 2027 年！2027 年会是怎么样，2029 年会是怎么样，2031 年、2036 年又会怎么样，我看不要想这么多了，专注于当下吧。好好感受每天的幸福时刻，吃美味的事物、看美丽的风景、睡美美的觉、做感兴趣的事情，也是很不错的嘛。
借用教员的诗，来与诸位共勉。“一万年太久，只争朝夕！”
																				   	- 2026.10.25 21:26 大连
```



