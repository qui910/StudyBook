# 1 概述

​	StAX（Streaming API for XML）：StAX解析器是最后出来的解析器，被认为比前两种都好。它和SAX非常像，也是event-based API，不同点是：

* StAX是Pull类型，而SAX是Push类型。
* StAX相对SAX来说，更易于使用，编程上更方便一点。
* SAX只能对XML内容进行读，不能写；而StAX既可进行读，也可以进行写。

   （StAX有两种API,一种是cursor-based,一种是iterator-based）

  	优点：处理速度快，节省内存，可进行读写。缺点：不能再次读取已经读过的内容。 适用于只扫描一次XML内容，就能提取想要的数据的场合。