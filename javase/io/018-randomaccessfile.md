# 1 概述

RandomAccessFile 是随机访问文件(包括读/写)的类。它支持对文件随机访问的读取和写入，即我们可以从指定的位置读取/写入文件数据。

需要注意的是，RandomAccessFile 虽然属于java.io包，但它不是InputStream或者OutputStream的子类；它也不同于FileInputStream和FileOutputStream。 FileInputStream 只能对文件进行读操作，而FileOutputStream 只能对文件进行写操作；但是，RandomAccessFile 同时支持文件的读和写，并且它支持随机访问。

 