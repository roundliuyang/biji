# JDK8 jvm参数配置及说明





## **堆栈内存参数设置**

```bash
-Xss或-XX:ThreadStackSize=n
	每个线程堆栈最大值
	指令1：-Xss256k
	指令2：-XX:ThreadStackSize=256k
	注意：
	默认堆栈大小为1M，应该128K就够用，大的堆栈建议256K，栈设置太大，会导致线程创建减少。栈设置小，会导致	   深入不够，深度的递归会导致栈溢出。
```

```ba
-Xms或-XX:InitialHeapSize=n
	设置堆的初始值
	指令1：-Xms2g
	指令2：-XX:InitialHeapSize=2048m
```













## **垃圾回收器设置**

