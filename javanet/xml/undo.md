# JAXB

​		



# JAXM（Java API for XML Messaging）

------

JAXM是为SOAP通信提供访问方法和传输机制的API.目前它支持SOAP1.1规范以及同步和异步通信。JAXM定义了大量服务，JAXM的实现产品将会提供这些服务，使得开发者不用面对复杂的通信系统。JAXM体系结构中包括两个重要的组件：JAXM Client和Provider.Client通常是作为J2EE web或EJB容器的一部分，以提供你所写的程序访问JAXM服务的能力。而Provider可以以不同的方式实现，主要负责发送和接收SOAP消息。这样你就可以直接地使用JAXM的API直接发送和接收SOAP消息。

# JAX-RPC（Java API for XML-RPC）

------

JAX-RPC是通过xml进行远程过程调用的Java API.它是基于SOAP技术的，使用SOAP作为底层的协议。这样对于开发者来说，只有方法，参数，返回值是可见的，而底层的soap通信都被隐藏起来了，开发人员不需要与之直接打交道。
 　JAXM和JAX-RPC在Web Services方面有很重要的作用。
 　JDK1.4自带的是JAXP的参考实现：Crimson的DOM， SAX解析器，Xalan的XSLT处理器。
 　如果你想用其他的实现替代它们，那就必须了解JAXP框架查找实现的具体步骤：
 　1、首先，算法会通过诸如javax.xml.transform.TranformerFactory这样的系统属性来定位具体实现的类。你可以在命令行中直接指定：
 　java -Djavax.xml.transform.TransformerFactory=com.foo.ConcreteTransformer YourApp
 　ConcreteTransformer是实现了TransformerFactory的子类，如果你用的是ant，也可以在build file中指定。
 　同样地有，javax.xml.parsers.document.uilderFactory和javax.xml.parsers.SAXBuilderFactory属性。
 　2、接着，如果系统属性中没有指定，JAXP将会在JRE的目录中查找lib/jaxp.properties属性文件，它像一般的properties文件一样是由name=value组成的，假设有如下的一行：
 　javax.xml.transform.TransformerFactory=com.foo.ConcreteTransformer
 　那么JAXP就会使用相应的TransformerFactory实现。
 　在Java程序中，你可以通过如下的代码获得JRE所在的目录：
 　String javaHomeDir = System.getProperty（"java.home"）；
 　不过要注意，如果是在一些IDE中使用，IDE会改变这个java.home的值，比如JBuilder.
 　3、如果jaxp.properties不存在或者没有相应的值，那么JAXP将会使用JAR文件的服务提供体制来定位正确的子类。简单地说，你可以在jar文件的META-INF/services目录下新建一个名为javax.xml.transform.TransformerFactory的文件，这个文件中只有一行：com.foo.ConcreteTransformer就可以了。
 　4、最后，如果上面3步都没有找到任何具体的实现，JAXP就会使用缺省的实现：Crimson和Xalan。

# JAX-WS（Java API for XML-based Web services）

------

Web服务已经出现很久了。首先是 SOAP，但 SOAP 仅描述消息的情况，然后是 WSDL，WSDL 并不会告诉您如何使用 Java™ 编写 Web 服务。在这种情况下，JAX-RPC 1.0 应运而生。经过数月使用之后，编写此规范的 Java Community Process (JCP) 人员认识到需要对其进行一些调整，调整的结果就是 JAX-RPC 1.1。该规范使用大约一年之后，JCP 人员希望构建一个更好的版本：JAX-RPC 2.0。其主要目标是与行业方向保持一致，但行业中不仅只使用 RPC Web 服务，还使用面向消息的 Web 服务。因此从名称中去掉了“RPC”，取而代之的是“WS”（当然表示的是 Web 服务）。因此 JAX-RPC 1.1 的后续版本是 JAX-WS 2.0——Java API for XML-based Web services。
 　JAX-WS 仍然支持 SOAP 1.1 over HTTP 1.1，因此互操作性将不会受到影响，仍然可以在网上传递相同的消息。
 　JAX-WS 仍然支持 WSDL 1.1，因此您所学到的有关该规范的知识仍然有用。WSDL 2.0 规范已经接近完成，但在 JAX-WS 2.0 相关工作结束时其工作仍在进行中。
 　区别何在？
 SOAP 1.2
 JAX-RPC 和 JAX-WS 都支持 SOAP 1.1。JAX-WS 还支持 SOAP 1.2。
 XML/HTTP
 WSDL 1.1 规范在 HTTP 绑定中定义，这意味着利用此规范可以在不使用 SOAP 的情况下通过 HTTP 发送 XML 消息。JAX-RPC 忽略了 HTTP 绑定。而 JAX-WS 添加了对其的支持。
 WS-I Basic Profile
 JAX-RPC 支持 WS-I Basic Profile (BP) V1.0。JAX-WS 支持 BP 1.1。（WS-I 即 Web 服务互操作性组织。）
 新 Java 功能
 JAX-RPC 映射到 Java 1.4。JAX-WS 映射到 Java 5.0。JAX-WS 依赖于 Java 5.0 中的很多新功能。
 Java EE 5 是 J2EE 1.4 的后续版本，添加了对 JAX-WS 的支持，但仍然支持 JAX-RPC，这可能会对 Web 服务新手造成混淆。
 数据映射模型
 JAX-RPC 具有自己的映射模型，此模型大约涵盖了所有模式类型中的 90%。它没有涵盖的部分映射到了 javax.xml.soap.SOAPElement。
 JAX-WS 的数据映射模型是 JAXB。JAXB 可保证所有 XML 模式的映射。
 接口映射模型
 JAX-WS 的基本接口映射模型与 JAX-RPC 的区别并不大，不过二者之间存在以下差异：
 JAX-WS 的模型使用新的 Java 5.0 功能。
 JAX-WS 的模型引入了异步功能。
 动态编程模型
 JAX-WS 的动态客户机模型与 JAX-RPC 的对应模型差别很大。很多更改都是为了认可行业需求：
 引入了面向消息的功能。
 引入了动态异步功能。
 JAX-WS 还添加了动态服务器模型，而 JAX-RPC 则没有此模型。
 消息传输优化机制（Message Transmission Optimization Mechanism，MTOM）
 JAX-WS 通过 JAXB 添加了对新附件规范 MTOM 的支持。Microsoft 从来没有给 SOAP 添加附件规范；但似乎大家都支持 MTOM，因此应该能够实现附件互操作性。
 处理程序模型
 从 JAX-RPC 到 JAX-WS 的过程中，处理程序模型发生了很大的变化。
 JAX-RPC 处理程序依赖于 SAAJ 1.2。JAX-WS 处理程序依赖于新的 SAAJ 1.3 规范。







## SOAP消息（JAXM）

在javax.xml.soap包下。用于JAX-WS中的消息。

 

 

## WebService （JAX-RPC \ JAX-WS）

 

用于Web Service的API：Jax-rpc Jax-ws。

**Jax-rpc** ：https://java.net/projects/jax-rpc/

JAX-RPC(基于[可扩展标记语言](http://baike.baidu.com/view/159832.htm)XML的[远程过程调用](http://baike.baidu.com/view/431455.htm)的Java[应用程序接口](http://baike.baidu.com/view/592964.htm))是Java Web服务开发包(WSDP)的应用程序接口(API)，WSDP能使Java开发者在Web服务或其他的Web应用程序中包括远程过程调用(RPC)。JAX-RPC致力于要使应用程序或Web服务调用其他应用程序或Web服务变得更加容易。

JAX-RPC为基于SOAP([简单对象访问协议](http://baike.baidu.com/view/1695890.htm))的应用程序的开发提供了一个编程模型。JAX-RPC编程模型通过抽象SOAP协议层的运行机制与提供Java和Web服务描述语言(WSDL)间的映射服务来简化开发。

 

 

**Jax-ws** ：https://jax-ws.java.net/

JAX-WS规范是一组XML web services的[JAVA API](http://baike.baidu.com/view/3911786.htm)，JAX-WS允许开发者可以选择RPC-oriented或者message-oriented 来实现自己的web services。

在 JAX-WS中，一个[远程](http://baike.baidu.com/view/856599.htm)调用可以转换为一个基于XML的协议例如SOAP，在使用JAX-WS过程中，开发者不需要编写任何生成和处理[SOAP](http://baike.baidu.com/view/60663.htm)消息的[代码](http://baike.baidu.com/view/41.htm)。JAX-WS的运行时实现会将这些API的调用转换成为对应的SOAP消息。