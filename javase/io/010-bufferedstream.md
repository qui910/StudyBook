# 1 概述

BufferedOutputStream 是缓冲输出流。它继承于FilterOutputStream，作用是为另一个输出流提供“缓冲功能”。

BufferedInputStream 是缓冲输入流。它继承于FilterInputStream，作用是为另一个输入流添加一些功能，例如，提供“缓冲功能”以及支持“mark()标记”和“reset()重置方法”。

BufferedInputStream 本质上是通过一个内部缓冲区数组实现的。例如，在新建某输入流对应的BufferedInputStream后，当我们通过read()读取输入流的数据时，BufferedInputStream会将该输入流的数据分批的填入到缓冲区中。每当缓冲区中的数据被读完之后，输入流会再次填充数据缓冲区；如此反复，直到我们读完输入流数据位置。