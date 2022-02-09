# Arthas

* 先下载一个arthas

```shell
curl -O https://arthas.aliyun.com/arthas-boot.jar
```

* 然后确保自己电脑上或者容器上有运行jvm，最后启动arthas

```
java -jar arthas-boot.jar
```

​	如果下载速度比较慢，可以使用aliyun的镜像：`java -jar arthas-boot.jar --repo-mirror aliyun --use-http`

* 启动arthas，选择对应要监视

```shell
[root@localhost arthas]# java -jar arthas-boot.jar  # 启动arthas
[INFO] arthas-boot version: 3.5.5
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 9466 zspdisk-0.0.1-SNAPSHOT.jar	#选择jvm的标号，下面填了1
1
[INFO] arthas home: /root/.arthas/lib/3.5.5/arthas
[INFO] Try to attach process 9466
[INFO] Attach process 9466 success.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.                           
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'                          
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.                          
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |                         
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'                          

wiki       https://arthas.aliyun.com/doc                                        
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html                  
version    3.5.5                                                                
main_class                                                                      
pid        9466                                                                 
time       2022-02-07 16:58:38            
```

## 查看dashboard

* 输入[dashboard](https://arthas.aliyun.com/doc/dashboard.html)，按`回车/enter`，会展示当前进程的信息，按`ctrl+c`可以中断执行。

```jvm
[arthas@9466]$ dashboard
ID    NAME                             GROUP            PRIORITY   STATE      %CPU       DELTA_TIME  TIME       INTERRUPTE DAEMON     
-1    C2 CompilerThread0               -                -1         -          0.0        0.000       0:20.575   false      true       
41    DestroyJavaVM                    main             5          RUNNABLE   0.0        0.000       0:7.507    false      false      
-1    C1 CompilerThread1               -                -1         -          0.0        0.000       0:6.676    false      true       
-1    VM Periodic Task Thread          -                -1         -          0.0        0.000       0:3.056    false      true       
25    https-jsse-nio-443-exec-10       main             5          WAITING    0.0        0.000       0:1.775    false      true       
55    https-jsse-nio-443-exec-11       main             5          WAITING    0.0        0.000       0:1.371    false      true       
24    https-jsse-nio-443-exec-9        main             5          WAITING    0.0        0.000       0:1.093    false      true       
22    https-jsse-nio-443-exec-7        main             5          WAITING    0.0        0.000       0:1.084    false      true       
16    https-jsse-nio-443-exec-1        main             5          WAITING    0.0        0.000       0:0.982    false      true       
21    https-jsse-nio-443-exec-6        main             5          WAITING    0.0        0.000       0:0.893    false      true       
57    https-jsse-nio-443-exec-13       main             5          WAITING    0.0        0.000       0:0.891    false      true       
20    https-jsse-nio-443-exec-5        main             5          WAITING    0.0        0.000       0:0.846    false      true
```

​	可以展示当前线程和容器的状态，jvm内存情况，运行时间，环境配置参数等。

## 查看具体线程

* 使用thread可以查看所有执行的线程

```bash
[arthas@28822]$ thread
Threads Total: 21, NEW: 0, RUNNABLE: 7, BLOCKED: 0, WAITING: 4, TIMED_WAITING: 3, TERMINATED: 0, Internal threads: 7                  
ID    NAME                             GROUP            PRIORITY   STATE      %CPU       DELTA_TIME  TIME       INTERRUPTE DAEMON     
-1    C2 CompilerThread0               -                -1         -          2.94       0.005       0:0.762    false      true       
-1    C1 CompilerThread1               -                -1         -          2.03       0.004       0:0.905    false      true       
21    arthas-command-execute           system           5          RUNNABLE   0.14       0.000       0:0.569    false      true       
-1    VM Periodic Task Thread          -                -1         -          0.1        0.000       0:1.119    false      true       
2     Reference Handler                system           10         WAITING    0.0        0.000       0:0.003    false      true       
3     Finalizer                        system           8          WAITING    0.0        0.000       0:0.003    false      true       
4     Signal Dispatcher                system           9          RUNNABLE   0.0        0.000       0:0.000    false      true       
8     Attach Listener                  system           9          RUNNABLE   0.0        0.000       0:0.021    false      true       
10    arthas-timer                     system           9          WAITING    0.0        0.000       0:0.000    false      true       

```

* 获取主线程运行的栈

```bash
thread 1 | grep 'main'
    at demo.MathGame.main(MathGame.java:17)
```

* 使用jad可以反编译该方法

```bash
[arthas@28822]$ jad TestArthas
ClassLoader:
+-sun.misc.Launcher$AppClassLoader@73d16e93
  +-sun.misc.Launcher$ExtClassLoader@18c012ba
Location:
/home/arthas/       
       /*
        * Decompiled with CFR.
        */
       import java.util.Date;
       
       public class TestArthas {
           public static void main(String[] stringArray) throws InterruptedException {
               while (true) {
/*12*/             Thread.sleep(1000L);
/*13*/             System.out.println(new Date());
               }
           }
       }
Affect(row-cnt:1) cost in 109 ms.
```

* 通过watch命令来查看函数的返回值：（由于main方法没有返回值，所以查看不到任何结果）

```bash
[arthas@28822]$ watch TestArthas main
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 144 ms, listenerId: 1
```

### 基础命令

```bash
help——查看命令帮助信息

cat——打印文件内容，和linux里的cat命令类似

echo–打印参数，和linux里的echo命令类似

grep——匹配查找，和linux里的grep命令类似

base64——base64编码转换，和linux里的base64命令类似

tee——复制标准输入到标准输出和指定的文件，和linux里的tee命令类似

pwd——返回当前的工作目录，和linux命令类似

cls——清空当前屏幕区域

session——查看当前会话的信息

reset——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类

version——输出当前目标 Java 进程所加载的 Arthas 版本号

history——打印命令历史

quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响

stop——关闭 Arthas 服务端，所有 Arthas 客户端全部退出

keymap——Arthas快捷键列表及自定义快捷键
```

### jvm相关

```bash
dashboard——当前系统的实时数据面板

thread——查看当前 JVM 的线程堆栈信息

jvm——查看当前 JVM 的信息

sysprop——查看和修改JVM的系统属性

sysenv——查看JVM的环境变量

vmoption——查看和修改JVM里诊断相关的option

perfcounter——查看当前 JVM 的Perf Counter信息

logger——查看和修改logger

getstatic——查看类的静态属性

ognl——执行ognl表达式

mbean——查看 Mbean 的信息

heapdump——dump java heap, 类似jmap命令的heap dump功能

vmtool——从jvm里查询对象，执行forceGc
```

### class/classloader相关

```bash
sc——查看JVM已加载的类信息

sm——查看已加载类的方法信息

jad——反编译指定已加载类的源码

mc——内存编译器，内存编译.java文件为.class文件

retransform——加载外部的.class文件，retransform到JVM里

redefine——加载外部的.class文件，redefine到JVM里

dump——dump 已加载类的 byte code 到特定目录

classloader——查看classloader的继承树，urls，类加载信息，使用classloader去getResource

monitor/watch/trace相关
```

