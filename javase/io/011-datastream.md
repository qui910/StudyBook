# 1 概述

**DataInputStream** 是数据输入流。它继承于FilterInputStream，是**用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。**应用程序可以使用DataOutputStream(数据输出流)写入由DataInputStream(数据输入流)读取的数据。

**DataOutputStream** 是数据输出流。它继承于FilterOutputStream，用来装饰其它输出流，将DataOutputStream和DataInputStream输入流配合使用，“允许应用程序以与机器无关方式从底层输入流中读写基本 Java 数据类型”。