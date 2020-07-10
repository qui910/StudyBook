# 1 概述

​		结合XML和Java后，就产生了Bind技术，将XML和Java Bean进行相互转化。

​		JAXB（Java API for XML Binding）定义了Java数据对象和xml结构之间的一种双向映射关系。这样就可以很方便地将一个Java对象存储为一个xml文档，也可以从一个xml文档实例化一个Java对象，他们之间的桥梁就是XML的Schema。原来JAXB是Java EE的一部分，在JDK1.6中，SUN将其放到了Java SE中，这也是SUN的一贯做法。JDK1.6中自带的这个JAXB版本是2.0，比起1.0(JSR 31)来，JAXB2(JSR 222)用JDK5的新特性Annotation来标识要作绑定的类和属性等，极大简化了开发的工作量。 

​		想要学习JAXB，可以参考 [官方文档](https://docs.oracle.com/javase/7/docs/technotes/guides/xml/jaxb/index.html) 或 [GITHUB](https://github.com/javaee/jaxb-v2 )

​		它的结构：首先要有xml的dtd以及binding schema（这个不是xml的schema，而是一个定义Java对象和xml结构之间映射关系xml文档），通过这两个文件JAXB就可以生成与xml文档结构一致的Java源文件，编译之后就可以很方便地通过具体的xml文档得到与xml结构一致的Java类（就是生成的那些类）unmarshalling，反过来marshalling也可以。

​		它的缺点也很明显，一旦xml的结构发生了改变，就要重新写bindng schema以及重新生成编译Java类。

​		Sun的动作总是一如既往地慢，在JAXB出台之前已经有了一些用于xml data binding的框架，我们再来看看同样也是做xml databinding但是并没有实现JAXB的框架：

​	    **(1)Castor**
​		Castor不仅仅支持对XML的绑定，它还支持对LDAP对象，用OQL将SQL查询映射为对象，以及对JDO的支持。与JAXB不同的是，它需要的仅仅是xml的Schema.通过xml的Schema来生成相应的Java源代码，编译之后就可以marshalling和unmarshalling了。

​		**(2)Zeus**
​		Zeus与Castor和JAXB相比，在class generation方面多做了些步骤，因此它可以支持多种的约束关系，包括对DTD，XML Schema以及TREX等等的支持。不过目前该项目好像已经不做了。

​		**(3)Quick**
 　	Quick也是一个非常灵活的框架，详细的情况可以google一下。