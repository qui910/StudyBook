# 1 概述

PipedReader和PipedWriter。它们和PipedInputStream和PipedOutputStream]一样，都可以用于管道通信。也属于**基础流**。

PipedWriter 是字符管道输出流，它继承于Writer。

PipedReader 是字符管道输入流，它继承于Writer。

PipedWriter和PipedReader的作用是可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedReader配套使用。