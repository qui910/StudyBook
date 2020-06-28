# 1 概述

​		Schema需要完成的工作与DTD类似，就是为了验证XML的有效性。但是Schema提供了比DTD更强大的功能和更细粒度的数据类型，另外Schema还可以自定义数据类型。此外，Schema是一个XML文件，而DTD不是。

​		Schema可以用关系型数据库的方式来理解，以下是对应关系：

| 关系型数据库 |     XML     |
| :----------: | :---------: |
|    数据库    | XML文档数据 |
|    表结构    |   Schema    |
|     SQL      |    XPath    |

​		Xml Schema是用一套预先规定的XML元素和属性创建的，这些元素和属性定义了XML文档的结构和内容模式。

​		Xml Schema规定XML文档实例的结构和每个元素/属性的数据类型。

XML示例：

```xml
<book>
    <name>书剑恩仇录</name>
    <author>金庸</author>
</book>
```

DTD示例：

```dtd
<!ELEMENT book(name,author)>
<!ELEMENT name(#PCDATA)>
<!ELEMENT author(#PCDATA)>
```

Schema示例：

```xml
<xs:element name="book" type="booktype" />
<xs:complexType name="booktype">
    <xs:sequence>
        <xs:element name="name" type="xs:string" />
        <xs:element name="author" type="xs:string" />
    </xs:sequence>
</xs:complexType>
```

​		Schema是验证xml的有效性，但是Schema本身也是个xml，它本身的有效性是通过DTD来验证的，所以DTD可以认为是本源的验证手段。而这个DTD就是在`xmlns:xs="http://www.w3.org/2001/XMLSchema"`这个URL中会指定，所以说这个URL不能修改。

## 1.1 DTD的局限性

* DTD不遵守XML语法（写XML文档实例时用一种语法，写DTD用另外一种语法）
* DTD数据类型有限（与数据库数据类型不一致）
* DTD不可扩展
* DTD不支持命名空间（命名冲突）



## 1.2 Schema的新特性

* Schema基于XML语法
* Schema可以用能处理XML文档的工具处理
* Schema大大扩充了数据 类型，可以自定义数据类型
* Schema支持元素的继承--Object-Oriented
* Schema支持属性组



# 2 Schema文档结构

```xml
<!-- demo1.xsd -->
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://mynamespace/myschema"
           >
    <!-- 放入实际内容 -->
    <xs:element name="book" type="booktype" />
    <xs:complexType name="booktype">
        <xs:sequence>
        	<xs:element name="name" type="xs:string" />
        	<xs:element name="author" type="xs:string" />
        </xs:sequence>
    </xs:complexType>
</xs:schema>
```

* 所有Schema文档使用schema作为根元素
* 用于构造schema的元素和数据类型来自http://www.w3.org/2001/XMLSchema命名空间，这个命名空间URL是固定写法。
* 本schema定义的元素和数据类型属于http://mynamespace/myschema命名空间，此命名空间属于自定义，可以自由修改。

待验证XML：

```xml
<?xml version="1.0"?>
<book xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:noNamespaceSchemaLocation="demo1.xsd"
      >
    <name>书剑恩仇录</name>
    <author>金庸</author>
</book>    
```



# 3 数据类型

##  3.1 简单数据类型

### 3.1.1 内置的数据类型（built-in data types）

#### 3.1.1.1 基本的数据类型

| 数据类型  |                             描述                             |
| :-------: | :----------------------------------------------------------: |
|  string   |                          表示字符串                          |
|  boolean  |                            布尔型                            |
|  decimal  |                      代表特定精度的数字                      |
|   float   |                     表示单精度32位浮点数                     |
|  double   |                     表示双精度64位浮点数                     |
| duration  |                         表示持续时间                         |
| datatime  |                        代表特定的时间                        |
|   time    |               代表特定的时间，但是是每天重复的               |
|   date    |                           代表日期                           |
| hexBinary |                        代表十六进制数                        |
|  anyURI   |                  代表一个URI，用来定位文件                   |
|   long    | 表示整型数，大小介于-9223372036854775808和9223372036854775807之间 |
|    int    |       表示整型数，大小介于-2147483648和2147483647之间        |
|   short   |            表示整型数，大小介于-32768和32767之间             |
|   bytes   |               表示整型，大小介于-128和127之间                |

#### 3.1.1.2 扩展的数据类型

| 数据类型 |          描述          |
| :------: | :--------------------: |
|    ID    |    用于唯一表示元素    |
|  IDREF   | 参考ID类型的元素或属性 |
|  ENTITY  |        实体类型        |
| NMTOKEN  |      NMTOKEN类型       |
| NMTOKENS |     NMTOKEN类型集      |
| NOTATION |    代表NOTATION类型    |

 

### 3.1.2 用户自定义数据类型（通过simpleType定义）

(后续讲解)

### 3.1.3  数据类型的特性

|      特性      |                描述                |
| :------------: | :--------------------------------: |
|  enumeration   | 指定的数据集种选项，限定用忽的选择 |
| fractionDigits |   限定最大的小数位，用于控制精度   |
|     length     |           指定数据的长度           |
|  maxExclusive  |      指定数据的最大值（小于）      |
|  maxInclusive  |    指定数据的最大值（小于等于）    |
|   maxLength    |          指定长度的最大值          |
|  minExclusive  |      指定数据的最小值（大于）      |
|  minInclusive  |    指定数据的最小值（大于等于）    |
|   minLength    |          指定长度的最小值          |
|    Pattern     |         指定数据的显示规范         |



## 3.2. 复杂类型（通过complexType定义）

(后续讲解)

# 4 元素类型



## 4.1 schema

作用：包含已经定义的schema

用法：`<xs:schema>`

属性：

* xmlns
* targetNamespace

```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://mynamespace/myschema"
           >
    <!-- 放入实际内容 -->
</xs:schema>
```

## 4.2 element

作用：声明一个元素

| 属性              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| id                | 可选。规定该元素的唯一的 ID。                                |
| name              | 可选。规定元素的名称。如果父元素是 schema 元素，则此属性是必需的。 |
| ref               | 可选。对另一个元素的引用。ref 属性可包含一个命名空间前缀。如果父元素是 schema 元素，则不是使用该属性。 |
| type              | 可选。规定内建数据类型的名称，或者规定 simpleType 或 complexType 元素的名称。 |
| substitutionGroup | 可选。规定可用来替代该元素的元素的名称。 该元素必须具有相同的类型或从指定元素类型派生的类型。 如果父元素不是 schema 元素，则不可以使用该属性。 |
| default           | 可选。为元素规定默认值（仅当元素内容是简单类型或 textOnly 时使用）。 |
| fixed             | 可选。为元素规定固定值（仅当元素内容是简单类型或 textOnly 时使用）。 |
| form              | 可选。该元素的形式。 默认值是包含该属性的 schema 元素的 elementFormDefault 属性的值。 该值必须是下列字符串之一： "qualified" 或 "unqualified"。如果父元素是 schema 元素，则不能使用该属性。如果该值是 "unqualified"，则无须通过命名空间前缀限定该元素。如果该值是 "qualified"，则必须通过命名空间前缀限定该元素。 |
| maxOccurs         | 可选。规定 element 元素在父元素中可出现的最大次数。该值可以是大于或等于零的整数。若不想对最大次数设置任何限制，请使用字符串 "unbounded"。 默认值为 1。如果父元素是 schema 元素，则不能使用该属性。 |
| minOccurs         | 可选。规定 element 元素在父元素中可出现的最小次数。该值可以是大于或等于零的整数。默认值为 1。如果父元素是 schema 元素，则不能使用该属性。 |
| nillable          |                                                              |
| abstract          | 可选。指示元素是否可以在实例文档中使用。如果该值为 true，则元素不能出现在实例文档中。 相反，substitutionGroup 属性包含该元素的限定名 (QName) 的其他元素必须出现在该元素的位置。多个元素可以在其 substitutionGroup 属性中引用该元素。默认值是 false。 |
| block             | 可选。派生的类型。 block 属性防止具有指定派生类型的元素被用于替代该元素。该值可以包含 #all 或者一个列表，该列表是 extension、restriction 或 substitution 的子集：extension - 防止通过扩展派生的元素被用来替代该元素。restriction - 防止通过限制派生的元素被用来替代该元素。substitution - 防止通过替换派生的元素被用来替代该元素。#all - 防止所有派生的元素被用来替代该元素。 |
| final             | 可选。设置 element 元素上 final 属性的默认值。如果父元素不是 schema 元素，则不能使用该属性。该值可以包含 #all 或者一个列表，该列表是 extension 或 restriction 的子集：extension - 防止通过扩展派生的元素被用来替代该元素restriction - 防止通过限制派生的元素被用来替代该元素#all - 防止所有派生的元素被用来替代该元素 |
| *any attributes*  | 可选。规定带有 non-schema 命名空间的任何其他属性。           |

示例：

```xml
<!--demo.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:element name="cat" type="xs:string" />
	<xs:element name="dog" type="xs:string"/>
	<xs:element name="pets">
		<xs:complexType>
			<xs:sequence minOccurs="0" maxOccurs="unbounded">
				<xs:element ref="cat"/>
				<xs:element ref="dog"/>
			</xs:sequence>
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<pets xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/demo.xsd">
	<cat>hello</cat>
	<dog>world</dog>
</pets>
```



## 4.3 attribute

作用：声明一个属性

属性：name/type/ref/use

示例：

```xml
<!-- demo2.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:attribute name="interest" type="xs:integer" />
	<xs:element name="person">
		<xs:complexType>
			<xs:sequence>
				<xs:element name="hello" type="xs:string" />
				<xs:element name="world" type="xs:string" />	
			</xs:sequence>
			<xs:attribute ref="interest" />
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<person xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/demo2.xsd"
interest="12">
	<hello>测试Hello</hello>
	<world>测试World</world>
</person>
```

## 4.4 group

作用：把一组元素声明组合在一起，以便他们能够一起被复合类型应用

属性：name/ref

示例：

```xml
<!-- demo2.xsd-->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:group name="myGroup">
		<xs:sequence>
			<xs:element name="name" type="xs:string" />
			<xs:element name="birthday" type="xs:date" />
			<xs:element name="age" type="xs:integer" />
		</xs:sequence>
	</xs:group>
	<xs:element name="person">
		<xs:complexType>
			<xs:group ref="myGroup" />
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<person xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/demo1.xsd">
	<name>小米</name>
	<birthday>2020-01-01</birthday>
	<age>20</age>
</person>
```



## 4.5 attributeGroup

作用：把一组属性声明组合在一起，以便可以被复合类型应用

属性：name/ref

示例：

```xml
<!-- demo3.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:attributeGroup name="myAttributeGroup">
		<xs:attribute name="hello" type="xs:string" use="required" />
		<xs:attribute name="world" type="xs:string" use="optional" />
	</xs:attributeGroup>
	<xs:element name="myElement">
		<xs:complexType>
			<xs:attributeGroup ref="myAttributeGroup" />
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<myElement hello="测试Hello" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/demo3.xsd" world="测试world" />
```



## 4.6 simpleType

作用：定义一个简单类型，它决定了元素和属性的约束和相关信息

属性：name

内容：应用已经存在的简单类型，三种方式

* restrict  限定一个范围
* list  从列表中选择
* union 包含一个值的结合

### 4.6.1 restriction

定义一个约束条件

示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:simpleType name="myType">
		<xs:restriction base="xs:integer">
			<xs:minInclusive value="0" />
			<xs:maxInclusive value="100" />
		</xs:restriction>
	</xs:simpleType>
	<xs:element name="hello" type="myType">
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/demo4.xsd">
89
</hello>
```

这里xml的hello元素的值必须在`0~100`的范围。

### 4.6.2 list

从一个特定数据类型的集合中选择定义一个简单类型元素

示例：

```xml
<!-- demo5.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:simpleType name="myType">
		<xs:list itemType="xs:integer">
		</xs:list>
	</xs:simpleType>
	<xs:element name="hello" type="myType"/>
</xs:schema>
```

XML示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo5.xsd">
1 2 3
</hello>
```

这里xml的`hello`的值，只要是list限定的integer类型，就可以为多个，中间空格分隔。

### 4.6.3 union

从一个特定简单数据类型的集合中选择定义一个简单类型元素

```xml
<!-- demo6.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:attribute name="allFrameSize">
		<xs:simpleType>
			<xs:union memberTypes="roadbikeSize mountainbikeSize" />
		</xs:simpleType>
	</xs:attribute>
	
	<xs:simpleType name="roadbikeSize">
		<xs:restriction base="xs:positiveInteger">
			<xs:enumeration value="46" />
			<xs:enumeration value="55" />
			<xs:enumeration value="60" />
		</xs:restriction>
	</xs:simpleType>
	
	<xs:simpleType name="mountainbikeSize">
		<xs:restriction base="xs:string">
			<xs:enumeration value="small" />
			<xs:enumeration value="medium" />
			<xs:enumeration value="large" />
		</xs:restriction>
	</xs:simpleType>
	
	<xs:element name="hello">
		<xs:complexType>
			<xs:sequence>
				<xs:element name="welcome" type="xs:string" />
			</xs:sequence>
			<xs:attribute ref="allFrameSize" use="required" />
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hello allFrameSize="small" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo6.xsd">
	<welcome/>
</hello>
```

这里xml的`allFrameSize`属性可以是 `46`,`55`,`60`,`small`,`medium`,`large`中的一个。



## 4.7 simpleContent

作用：应用于complexType，对它的内容进行约束和扩展。

`<xs:simpleContent>` 元素下不包括子元素

`<xs:extension`表示这个元素的类型

这里complexType，simpleContent，simpleType和联合起来理解：

|     类型      | 是否包含子元素 | 是否包含属性 |
| :-----------: | :------------: | :----------: |
|  simpleType   |       否       |      否      |
| simpleContent |       否       |      是      |
|  complexType  |       是       |      是      |

示例：

```xml
<!-- demo7.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:element name="showSize">
		<xs:complexType>
			<xs:simpleContent>
				<xs:extension base="xs:decimal">
					<xs:attribute name="sizing" use="required">
						<xs:simpleType>
							<xs:restriction base="xs:string">
								<xs:enumeration value="us" />
								<xs:enumeration value="europe" />
								<xs:enumeration value="uk" />
							</xs:restriction>
						</xs:simpleType>
					</xs:attribute>
				</xs:extension>
			</xs:simpleContent>
		</xs:complexType>
	</xs:element>
</xs:schema>
```

此例中，simpleContent，用于complexType元素上，用于限定该complexType的内容类型，表示该complexType没有子元素，同时该complexType需要有属性，否则它就称为了simpleType。



## 4.8 complexType

作用：定义一个复合类型，它决定了一组元素和属性值的约束和相关信息。

属性：name

### 4.8.1 complexType与simpleType区别

* simpleType类型的元素中不能包含元素或属性，如下xml是没有子元素或属性的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hello allFrameSize="small" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo6.xsd">
	<welcome/>
</hello>
```

* 当需要声明一个元素的子元素和`/` 或 属性时，用complexType
* 当需要基于内置的基本数据类型定义一个新的数据类型时，用simpleType



### 4.8.2 示例

```xml
<!-- demo7.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:element name="showSize">
		<xs:complexType>
			<xs:simpleContent>
				<xs:extension base="xs:decimal">
					<xs:attribute name="sizing" use="required">
						<xs:simpleType>
							<xs:restriction base="xs:string">
								<xs:enumeration value="us" />
								<xs:enumeration value="europe" />
								<xs:enumeration value="uk" />
							</xs:restriction>
						</xs:simpleType>
					</xs:attribute>
				</xs:extension>
			</xs:simpleContent>
		</xs:complexType>
	</xs:element>
</xs:schema>
```

XML样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<showSize sizing="uk" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo7.xsd">
1.2
</showSize>
```

这里的含义就是定义一个`showSize`元素，它是基于原始类型`decimal`的扩展，并且有一个属性值`sizing`，且属性值的选项是`us`,`europe`,`uk`。



## 4.9 choice

作用：允许唯一的一个元素从一个组中被选择

属性：minOccurs/maxOccurs

示例：

```xml
<!-- demo8.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:complexType name="myType">
		<xs:choice minOccurs="1" maxOccurs="3">
			<xs:element name="hello" type="xs:string" />
			<xs:element name="world" type="xs:string" />
		</xs:choice>
	</xs:complexType>
	<xs:element name="helloworld" type="myType" />
</xs:schema>
```

XML示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<helloworld xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo8.xsd">
	<world />
	<world />
	<hello />
</helloworld>
```

在schema中`minOccurs="1" maxOccurs="3"` 表明被选择的元素（`world`,`hello`）选择时整体最少选择一个，最多只能出现3次（具体每次出现hello还是world元素，都可以）。

## 4.10 sequence

作用：给一组元素一个特定的序列

属性：minOccurs/maxOccurs

示例：

```xml
<!-- emo9.xsd -->
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" attributeFormDefault="unqualified">
	<xs:complexType name="myType">
		<xs:sequence minOccurs="1" maxOccurs="1">
			<xs:element name="hello" type="xs:string" />
			<xs:element name="world" type="xs:string" />
		</xs:sequence>
	</xs:complexType>
	<xs:element name="helloworld" type="myType" />
</xs:schema>

```

XML示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<helloworld xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="file:///C:/Users/Administrator/Desktop/xmltest/demo9.xsd">
	<hello/>
	<world/>
    <hello/>
	<world/>
    <hello/>
	<world/>
</helloworld>
```

`sequence`中的`minOccurs="1" maxOccurs="3"` 表明整体元素最少出现一次，最多出现3次。

# 5 总结

​		通过DOCTYPE可以明确指定文档的根元素，因为DOCTYPE后面跟的元素就是文档的根元素。通过Schema是没法明确指定目标XML文档的根元素，XmlSpy是通过推断哪个元素包含了其他元素来选择包含其他元素最多的那个元素作为文档的根，但是用户可以明确指定文档的根元素而不必按照XmlSpy的生成来做。

* Schema是另一种文档类型定义，它遵循XML的语言规范。
* Schema是可扩展的，支持命名空间
* Schema支持更多的数据类型与元素类型
* Schema用element声明元素，用attribute声明元素的属性
* Schema用simpleType定义简单类型，用complexType定义复杂类型





