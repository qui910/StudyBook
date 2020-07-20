# 1 概述

PrintStream 是打印输出流，它继承于FilterOutputStream，用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。

与其他输出流不同，PrintStream 永远不会抛出 IOException；它产生的IOException会被自身的函数所捕获并设置错误标记， 用户可以通过 checkError() 返回错误标记，从而查看PrintStream内部是否产生了IOException。

另外，**PrintStream 提供了自动flush 和 字符集设置功能**。所谓自动flush，就是往PrintStream写入的数据会立刻调用flush()函数。