# 1 概述

InputStreamReader和OutputStreamWriter 是字节流通向字符流的桥梁：它使用指定的 charset 读写字节并将其解码为字符。

InputStreamReader 的作用是将“字节输入流”转换成“字符输入流”。它继承于Reader。

OutputStreamWriter 的作用是将“字节输出流”转换成“字符输出流”。它继承于Writer。