# 1 概述

​		JDOM是一种使用XML的独特Java工具包，用于快速开发XML应用程序。它的设计包含Java语言的语法及至语义。

​		JDOM是一个开源项目，它基于树形结构，利用纯Java的技术对XML文档实现解析，生成，序列化以及多种操作。[官网](www.jdom.org)

​		JDOM直接为Java编程服务。它利用更为强有力的JAVA语言的诸多特定（方法重载，集合概念等），把SAX和DOM的功能有效结合起来。

​		JDOM是用Java语言读，写，操作XML的新API函数。在直接，简单和高效的前提下，这些API函数被最大限度的优化。在使用设计上即可能的隐藏原来使用XML过程中的复杂性。利用JDOM处理XML文档将是一件轻松，简单的事。

​		JDOM主要用来弥补DOM即SAX在实际应用当中的不足之处。这些不足之处主要在于SAX没有文档修改，随机访问以及输出的功能，而对于DOM来说，Java程序员在使用时感觉不太方便。DOM的缺点主要是由于DOM是一个接口定义语言（IDL），它的任务是在不同语言实现中的一个最低的通用标准，并不是为Java特别设计的

​		在JDOM中，XML元素就是Element的实例，XML属性就是Attribute的实例，XML文档本身就是Document的实例。

# 2 JDOM基础

​		JDOM是Java平台专用的，是类驱动。因为JDOM对象就是像Document，Element和Attribute这些类的直接实例，因此创建一个新JDOM对象就如在Java语言中使用new操作符一样容易。它还意味着不需要进行工程化接口配置。

​		JDOM有以下几个包组成：

```java
// 包含了所有的xml文档要素和java类
org.jdom
// 包含了与dom适配的java类
org.jdom.adapters
// 包含了xml文档的过滤器类
org.jdom.filter
// 包含了读取xml文档的类
org.jdom.input
// 包含了写入xml文档的类
org.jdom.output
// 包含了将jdomxml文档建接口转换为其他xml文档接口
org.jdom.transform
// 包含了对xml文档xpath操作的类
org.jdom.xpath
```

## 2.1 org.jdom

​		org.jdom里的类主要是解析xml文档后所要用到的所有数据类型

```java
Attribute
CDATA
Coment
DocType
Document
Element
EntityRef
Namespace
ProscessingInstruction
Text
```

## 2.2 org.jdom.input

​		org.jdom.input输入类，主要用于文件的创建工作

```java
SAXBuilder
DOMBuilder
ResultSetBuilder
```

## 2.3 org.jdom.output

​		org.jdom.output输出类，用于文档转换输出

```
XMLOutputter
SAXOutputter
DomOutputter
JTreeOutputter
```



## 2.4 简单操作

### 2.4.1 Document

​		DOM的Document和JDOM的Document之间的相互转换使用方法

```java
DOMBuilder builder = new DOMBuilder();

org.jdom.Documet jdomDocument = builder.build(domDocument);

// work with the JDOM document
DOMOutputter converter = new DOMOutputter;

org.w3c.dom.Document domDocument = converter.output(jdomDocument);

// work with the DOM document...
```

### 2.4.2 XMLOutPutter

​		JDOM的输出非常灵活，支持多种IO格式以及风格的输出。

```java
Document doc = new Document(...);
XMLOutPutter outp - new XMLOutPutter();
// Raw output
outp.output(doc, fileOutputStream);
// Compressed output
outp.setTextTrim(true);
outp.output(doc,socket.getOutputStream());
// Pretty output
outp.setIndent(" ")
outp.setNewlines(true);
outp.output(doc,System.out);
```

