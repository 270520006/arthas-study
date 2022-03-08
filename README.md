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

### 使用arthas查看方法耗时

​	例如我这里有一段代码，然后我想查看对应方法的耗时，那么就可以用arthas。

* 要运行的代码

```java
package com.zsp.freemarker;
import freemarker.cache.StringTemplateLoader;
import freemarker.template.Configuration;
import freemarker.template.Template;
import java.util.Scanner;
/**
 * 普通的freemark模板导入案例
 */
public class TestScannerFreemarker {
	static volatile int times=0;
	public static void main(String[] args) throws Exception {
		String data = get1M();
		Configuration conf = new Configuration();
		conf.setTemplateUpdateDelay(10);
		StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();
		for (int i = 0; i <1000 ; i++) {
			stringTemplateLoader.putTemplate(""+i,data);
			times+=1;
			System.out.println(times);
		}
		conf.setTemplateLoader(stringTemplateLoader);
		while (true) {
			Scanner input = new Scanner(System.in);

			int j = input.nextInt();
			if (j==0){
				updateTemplate(conf);
				continue;
			}
			System.out.println(j);
			long start = System.currentTimeMillis();
			Template template = conf.getTemplate("" + j, "utf-8");
			System.out.println("获取" + j + "号模板使用时间：" + (System.currentTimeMillis() - start));
			System.out.println("模板名为：" + template.getSourceName() + "模板长度为：" + template.toString().length());
		}
	}
	public static void updateTemplate(Configuration conf){
		StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();
		String data = get1M();

		for (int i = 0; i <1000 ; i++) {
			stringTemplateLoader.putTemplate(""+i,data+"--");

			System.out.println(i);
		}
		conf.setTemplateLoader(stringTemplateLoader);
	}
	public static String get1M(){
		StringBuilder sb1=new StringBuilder();
		for(int i=0;i<1024*100;i++){
			int j = (int) (Math.random()*10);
			sb1.append(j+"");
		}
		return sb1.toString();
	}
}
```

* 先把代码打成jar包放到linux环境中，然后运行jar包（使用窗口1打开）

```bash
drwxr-xr-x 2 root root     6 3月   3 16:05 arthas-output
drwxr-xr-x 3 root root    17 3月   3 16:08 classes
-rw-r--r-- 1 root root 15256 3月   3 16:39 freemarker-demo-1.0-SNAPSHOT.jar
drwxr-xr-x 3 root root    25 3月   3 16:08 generated-sources
drwxr-xr-x 3 root root    30 3月   3 16:08 generated-test-sources
drwxr-xr-x 2 root root    35 3月   3 16:28 hzwlib
drwxr-xr-x 2 root root    28 3月   3 16:08 maven-archiver
drwxr-xr-x 3 root root    35 3月   3 16:08 maven-status
drwxr-xr-x 2 root root    90 3月   3 16:08 surefire-reports
drwxr-xr-x 2 root root    47 3月   3 16:08 test-classes
[root@localhost target]# java -jar freemarker-demo-1.0-SNAPSHOT.jar 
```

* 启动arthas，输入进程序号1，对其进行监测（使用窗口2打开）

```bash
[root@localhost arthas]# java -jar arthas-boot.jar 
[INFO] arthas-boot version: 3.5.5
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 8308 freemarker-demo-1.0-SNAPSHOT.jar
1
```

* 现在我要对updateTemplate方法进行耗时检测，输入命令后会进入等待（使用窗口2打开）

```bash
[arthas@9236]$ trace com.zsp.freemarker.TestScannerFreemarker  updateTemplate
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 124 ms, listenerId: 2

```

* 然后去使用代码的窗口触发，就会出现方法使用时间

```java
[arthas@21555]$ trace freemarker.template.Configuration  getTemplate
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 7) cost in 183 ms, listenerId: 2
`---ts=2022-03-04 17:55:10;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@42a57993
    `---[37.93243ms] freemarker.template.Configuration:getTemplate()
        `---[37.712134ms] freemarker.template.Configuration:getTemplate() #1664
            `---[37.53198ms] freemarker.template.Configuration:getTemplate()
                +---[0.062235ms] freemarker.template.Configuration:getLocale() #1780
                +---[37.34836ms] freemarker.cache.TemplateCache:getTemplate() #1786
                `---[0.011529ms] freemarker.cache.TemplateCache$MaybeMissingTemplate:getTemplate() #1787
```

* 如果想要找不同类下的运行情况，在方法处加上|表示和，-E为正则表达

```shell
[arthas@21555]$ trace -E freemarker.template.Configuration|freemarker.cache.TemplateCache  getTemplate
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 10) cost in 475 ms, listenerId: 6
`---ts=2022-03-04 18:00:32;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@42a57993
    `---[1.006896ms] freemarker.template.Configuration:getTemplate()
        `---[0.757065ms] freemarker.template.Configuration:getTemplate() #1664
            `---[0.718762ms] freemarker.template.Configuration:getTemplate()
                +---[0.036673ms] freemarker.template.Configuration:getLocale() #1780
                +---[0.625308ms] freemarker.cache.TemplateCache:getTemplate() #1786
                |   `---[0.595497ms] freemarker.cache.TemplateCache:getTemplate()
                |       +---[0.007078ms] freemarker.template.utility.NullArgumentException:check() #242
                |       +---[0.002521ms] freemarker.template.utility.NullArgumentException:check() #243
                |       +---[0.0023ms] freemarker.template.utility.NullArgumentException:check() #244
                |       +---[0.011573ms] freemarker.cache.TemplateNameFormat:normalizeAbsoluteName() #247
                |       +---[0.490901ms] freemarker.cache.TemplateCache:getTemplate() #261
                |       |   `---[0.471566ms] freemarker.cache.TemplateCache:getTemplate()
                |       |       +---[0.007297ms] freemarker.log.Logger:isDebugEnabled() #294
                |       |       +---[0.005379ms] freemarker.cache.TemplateCache$TemplateKey:<init>() #298
                |       |       +---[0.019306ms] freemarker.cache.CacheStorage:get() #302
```

* 如果想要找同个类下的其他方法，也同样

```shell
[arthas@21555]$ trace -E freemarker.cache.TemplateCache  getTemplate|loadTemplate
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 4) cost in 181 ms, listenerId: 9
`---ts=2022-03-04 18:05:37;thread_name=main;id=1;is_daemon=false;priority=5;TCCL=sun.misc.Launcher$AppClassLoader@42a57993
    `---[38.607984ms] freemarker.cache.TemplateCache:getTemplate()
        +---[0.131334ms] freemarker.template.utility.NullArgumentException:check() #242
        +---[0.005889ms] freemarker.template.utility.NullArgumentException:check() #243
        +---[0.002904ms] freemarker.template.utility.NullArgumentException:check() #244
        +---[0.016624ms] freemarker.cache.TemplateNameFormat:normalizeAbsoluteName() #247
        +---[38.325221ms] freemarker.cache.TemplateCache:getTemplate() #261
        |   `---[38.279232ms] freemarker.cache.TemplateCache:getTemplate()
        |       +---[0.008646ms] freemarker.log.Logger:isDebugEnabled() #294
        |       +---[0.005427ms] freemarker.cache.TemplateCache$TemplateKey:<init>() #298
        |       +---[0.010686ms] freemarker.cache.CacheStorage:get() #302
        |       +---[0.005082ms] freemarker.cache.TemplateCache$CachedTemplate:<init>() #387
        |       +---[0.096613ms] freemarker.cache.TemplateCache:lookupTemplate() #390
        |       +---[0.004032ms] freemarker.cache.TemplateLookupResult:isPositive() #392
        |       +---[0.003868ms] freemarker.cache.TemplateLookupResult:getTemplateSource() #400
        |       +---[0.004951ms] freemarker.cache.TemplateLoader:getLastModified() #408
        |       +---[0.003828ms] freemarker.cache.TemplateLookupResult:getTemplateSourceName() #411
        |       +---[37.931488ms] freemarker.cache.TemplateCache:loadTemplate() #409
        |       |   `---[37.887049ms] freemarker.cache.TemplateCache:loadTemplate()
        |       |       +---[0.007472ms] freemarker.cache.TemplateLoader:getReader() #493
        |       |       +---[37.692165ms] freemarker.template.Template:<init>() #495
        |       |       +---[0.027137ms] freemarker.template.Template:setLocale() #536
        |       |       +---[0.007237ms] freemarker.template.Template:setCustomLookupCondition() #537
        |       |       `---[0.005575ms] freemarker.template.Template:setEncoding() #538
        |       +---[0.060561ms] freemarker.cache.TemplateCache:storeCached() #415
        |       +---[0.004461ms] freemarker.cache.TemplateLookupResult:isPositive() #428
        |       +---[0.004216ms] freemarker.cache.TemplateLookupResult:getTemplateSource() #429
        |       `---[0.010549ms] freemarker.cache.TemplateLoader:closeTemplateSource() #429
        `---[0.008907ms] freemarker.cache.TemplateCache$MaybeMissingTemplate:<init>() #262

```

### 监听方法执行情况

| 参数名称            | 参数说明                                          |
| ------------------- | ------------------------------------------------- |
| *class-pattern*     | 类名表达式匹配                                    |
| *method-pattern*    | 函数名表达式匹配                                  |
| *express*           | 观察表达式，默认值：`{params, target, returnObj}` |
| *condition-express* | 条件表达式                                        |
| [b]                 | 在**函数调用之前**观察                            |
| [e]                 | 在**函数异常之后**观察                            |
| [s]                 | 在**函数返回之后**观察                            |
| [f]                 | 在**函数结束之后**(正常返回和异常返回)观察        |
| [E]                 | 开启正则表达式匹配，默认为通配符匹配              |
| [x:]                | 指定输出结果的属性遍历深度，默认为 1              |

* 监听方法执行后和执行前

```shell
[arthas@23916]$ watch com.zsp.freemarker.TestScannerFreemarker  updateTemplate -f
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 44 ms, listenerId: 3
method=com.zsp.freemarker.TestScannerFreemarker.updateTemplate location=AtExit
ts=2022-03-04 18:19:31; [cost=647.018553ms] result=@ArrayList[
    @Object[][isEmpty=false;size=1],
    null,
    null,
]
[arthas@23916]$ watch com.zsp.freemarker.TestScannerFreemarker  updateTemplate -b
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 27 ms, listenerId: 4
method=com.zsp.freemarker.TestScannerFreemarker.updateTemplate location=AtEnter
ts=2022-03-04 18:20:33; [cost=0.03805ms] result=@ArrayList[
    @Object[][isEmpty=false;size=1],
    null,
    null,
]
```

* 监听属性深度为2

```shell
[arthas@24352]$ watch com.zsp.freemarker.TestScannerFreemarker  updateTemplate  -x 2
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 133 ms, listenerId: 1
method=com.zsp.freemarker.TestScannerFreemarker.updateTemplate location=AtExit
ts=2022-03-04 18:23:03; [cost=938.210116ms] result=@ArrayList[
    @Object[][
        @Configuration[freemarker.template.Configuration@7b3300e5],
    ],
    null,
    null,
]
```

* 监听属性深度为3

```shell
[arthas@24352]$ watch com.zsp.freemarker.TestScannerFreemarker  updateTemplate  -x 3
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 42 ms, listenerId: 2
method=com.zsp.freemarker.TestScannerFreemarker.updateTemplate location=AtExit
ts=2022-03-04 18:24:23; [cost=705.97691ms] result=@ArrayList[
    @Object[][
        @Configuration[
            CACHE_LOG=@JULLogger[freemarker.log._JULLoggerFactory$JULLogger@7382f612],
            VERSION_PROPERTIES_PATH=@String[freemarker/version.properties],
            DEFAULT_ENCODING_KEY_SNAKE_CASE=@String[default_encoding],
            DEFAULT_ENCODING_KEY_CAMEL_CASE=@String[defaultEncoding],
            DEFAULT_ENCODING_KEY=@String[default_encoding],
            LOCALIZED_LOOKUP_KEY_SNAKE_CASE=@String[localized_lookup],
            LOCALIZED_LOOKUP_KEY_CAMEL_CASE=@String[localizedLookup],
            LOCALIZED_LOOKUP_KEY=@String[localized_lookup],
            STRICT_SYNTAX_KEY_SNAKE_CASE=@String[strict_syntax],
            STRICT_SYNTAX_KEY_CAMEL_CASE=@String[strictSyntax],
            STRICT_SYNTAX_KEY=@String[strict_syntax],
            WHITESPACE_STRIPPING_KEY_SNAKE_CASE=@String[whitespace_stripping],
            WHITESPACE_STRIPPING_KEY_CAMEL_CASE=@String[whitespaceStripping],
            WHITESPACE_STRIPPING_KEY=@String[whitespace_stripping],
            CACHE_STORAGE_KEY_SNAKE_CASE=@String[cache_storage],
            CACHE_STORAGE_KEY_CAMEL_CASE=@String[cacheStorage],
            CACHE_STORAGE_KEY=@String[cache_storage],
            TEMPLATE_UPDATE_DELAY_KEY_SNAKE_CASE=@String[template_update_delay],
            TEMPLATE_UPDATE_DELAY_KEY_CAMEL_CASE=@String[templateUpdateDelay],
            TEMPLATE_UPDATE_DELAY_KEY=@String[template_update_delay],
            AUTO_IMPORT_KEY_SNAKE_CASE=@String[auto_import],
            AUTO_IMPORT_KEY_CAMEL_CASE=@String[autoImport],
            AUTO_IMPORT_KEY=@String[auto_import],
            AUTO_INCLUDE_KEY_SNAKE_CASE=@String[auto_include],
            AUTO_INCLUDE_KEY_CAMEL_CASE=@String[autoInclude],
            AUTO_INCLUDE_KEY=@String[auto_include],
            TAG_SYNTAX_KEY_SNAKE_CASE=@String[tag_syntax],
            TAG_SYNTAX_KEY_CAMEL_CASE=@String[tagSyntax],
            TAG_SYNTAX_KEY=@String[tag_syntax],
            NAMING_CONVENTION_KEY_SNAKE_CASE=@String[naming_convention],
            NAMING_CONVENTION_KEY_CAMEL_CASE=@String[namingConvention],
            NAMING_CONVENTION_KEY=@String[naming_convention],
            TEMPLATE_LOADER_KEY_SNAKE_CASE=@String[template_loader],
            TEMPLATE_LOADER_KEY_CAMEL_CASE=@String[templateLoader],
            TEMPLATE_LOADER_KEY=@String[template_loader],
            TEMPLATE_LOOKUP_STRATEGY_KEY_SNAKE_CASE=@String[template_lookup_strategy],
            TEMPLATE_LOOKUP_STRATEGY_KEY_CAMEL_CASE=@String[templateLookupStrategy],
            TEMPLATE_LOOKUP_STRATEGY_KEY=@String[template_lookup_strategy],
            TEMPLATE_NAME_FORMAT_KEY_SNAKE_CASE=@String[template_name_format],
            TEMPLATE_NAME_FORMAT_KEY_CAMEL_CASE=@String[templateNameFormat],
            TEMPLATE_NAME_FORMAT_KEY=@String[template_name_format],
            INCOMPATIBLE_IMPROVEMENTS_KEY_SNAKE_CASE=@String[incompatible_improvements],
            INCOMPATIBLE_IMPROVEMENTS_KEY_CAMEL_CASE=@String[incompatibleImprovements],
            INCOMPATIBLE_IMPROVEMENTS_KEY=@String[incompatible_improvements],
            INCOMPATIBLE_IMPROVEMENTS=@String[incompatible_improvements],
            INCOMPATIBLE_ENHANCEMENTS=@String[incompatible_enhancements],
            SETTING_NAMES_SNAKE_CASE=@String[][isEmpty=false;size=14],
            SETTING_NAMES_CAMEL_CASE=@String[][isEmpty=false;size=14],
            AUTO_DETECT_TAG_SYNTAX=@Integer[0],
            ANGLE_BRACKET_TAG_SYNTAX=@Integer[1],
            SQUARE_BRACKET_TAG_SYNTAX=@Integer[2],
            AUTO_DETECT_NAMING_CONVENTION=@Integer[10],
            LEGACY_NAMING_CONVENTION=@Integer[11],
            CAMEL_CASE_NAMING_CONVENTION=@Integer[12],
            VERSION_2_3_0=@Version[2.3.0],
            VERSION_2_3_19=@Version[2.3.19],
            VERSION_2_3_20=@Version[2.3.20],
            VERSION_2_3_21=@Version[2.3.21],
            VERSION_2_3_22=@Version[2.3.22],
            VERSION_2_3_23=@Version[2.3.23],
            DEFAULT_INCOMPATIBLE_IMPROVEMENTS=@Version[2.3.0],
            DEFAULT_INCOMPATIBLE_ENHANCEMENTS=@String[2.3.0],
            PARSED_DEFAULT_INCOMPATIBLE_ENHANCEMENTS=@Integer[2003000],
            DEFAULT=@String[default],
            VERSION=@Version[2.3.23],
            FM_24_DETECTION_CLASS_NAME=@String[freemarker.core._2_4_OrLaterMarker],
            FM_24_DETECTED=@Boolean[false],
            defaultConfigLock=@Object[java.lang.Object@1055e4af],
            defaultConfig=null,
            strictSyntax=@Boolean[true],
            localizedLookup=@Boolean[true],
            whitespaceStripping=@Boolean[true],
            incompatibleImprovements=@Version[2.3.0],
            tagSyntax=@Integer[1],
            namingConvention=@Integer[10],
            cache=@TemplateCache[freemarker.cache.TemplateCache@3caeaf62],
            templateLoaderExplicitlySet=@Boolean[true],
            templateLookupStrategyExplicitlySet=@Boolean[false],
            templateNameFormatExplicitlySet=@Boolean[false],
            cacheStorageExplicitlySet=@Boolean[false],
            objectWrapperExplicitlySet=@Boolean[false],
            templateExceptionHandlerExplicitlySet=@Boolean[false],
            logTemplateExceptionsExplicitlySet=@Boolean[false],
            sharedVariables=@HashMap[isEmpty=false;size=5],
            rewrappableSharedVariables=null,
            defaultEncoding=@String[UTF-8],
            localeToCharsetMap=@ConcurrentHashMap[isEmpty=true;size=0],
            autoImports=@ArrayList[isEmpty=true;size=0],
            autoIncludes=@ArrayList[isEmpty=true;size=0],
            autoImportNsToTmpMap=@HashMap[isEmpty=true;size=0],
            class$freemarker$template$Configuration=@Class[class freemarker.template.Configuration],
            class$java$lang$String=null,
            class$freemarker$cache$CacheStorage=null,
            class$freemarker$cache$TemplateLoader=null,
            class$freemarker$cache$TemplateLookupStrategy=null,
            C_TRUE_FALSE=@String[true,false],
            DEFAULT=@String[default],
            DEFAULT_2_3_0=@String[default_2_3_0],
            JVM_DEFAULT=@String[JVM default],
            LOCALE_KEY_SNAKE_CASE=@String[locale],
            LOCALE_KEY_CAMEL_CASE=@String[locale],
            LOCALE_KEY=@String[locale],
            NUMBER_FORMAT_KEY_SNAKE_CASE=@String[number_format],
            NUMBER_FORMAT_KEY_CAMEL_CASE=@String[numberFormat],
            NUMBER_FORMAT_KEY=@String[number_format],
            TIME_FORMAT_KEY_SNAKE_CASE=@String[time_format],
            TIME_FORMAT_KEY_CAMEL_CASE=@String[timeFormat],
            TIME_FORMAT_KEY=@String[time_format],
            DATE_FORMAT_KEY_SNAKE_CASE=@String[date_format],
            DATE_FORMAT_KEY_CAMEL_CASE=@String[dateFormat],
            DATE_FORMAT_KEY=@String[date_format],
            DATETIME_FORMAT_KEY_SNAKE_CASE=@String[datetime_format],
            DATETIME_FORMAT_KEY_CAMEL_CASE=@String[datetimeFormat],
            DATETIME_FORMAT_KEY=@String[datetime_format],
            TIME_ZONE_KEY_SNAKE_CASE=@String[time_zone],
            TIME_ZONE_KEY_CAMEL_CASE=@String[timeZone],
            TIME_ZONE_KEY=@String[time_zone],
            SQL_DATE_AND_TIME_TIME_ZONE_KEY_SNAKE_CASE=@String[sql_date_and_time_time_zone],
            SQL_DATE_AND_TIME_TIME_ZONE_KEY_CAMEL_CASE=@String[sqlDateAndTimeTimeZone],
            SQL_DATE_AND_TIME_TIME_ZONE_KEY=@String[sql_date_and_time_time_zone],
            CLASSIC_COMPATIBLE_KEY_SNAKE_CASE=@String[classic_compatible],
            CLASSIC_COMPATIBLE_KEY_CAMEL_CASE=@String[classicCompatible],
            CLASSIC_COMPATIBLE_KEY=@String[classic_compatible],
            TEMPLATE_EXCEPTION_HANDLER_KEY_SNAKE_CASE=@String[template_exception_handler],
            TEMPLATE_EXCEPTION_HANDLER_KEY_CAMEL_CASE=@String[templateExceptionHandler],
            TEMPLATE_EXCEPTION_HANDLER_KEY=@String[template_exception_handler],
            ARITHMETIC_ENGINE_KEY_SNAKE_CASE=@String[arithmetic_engine],
            ARITHMETIC_ENGINE_KEY_CAMEL_CASE=@String[arithmeticEngine],
            ARITHMETIC_ENGINE_KEY=@String[arithmetic_engine],
            OBJECT_WRAPPER_KEY_SNAKE_CASE=@String[object_wrapper],
            OBJECT_WRAPPER_KEY_CAMEL_CASE=@String[objectWrapper],
            OBJECT_WRAPPER_KEY=@String[object_wrapper],
            BOOLEAN_FORMAT_KEY_SNAKE_CASE=@String[boolean_format],
            BOOLEAN_FORMAT_KEY_CAMEL_CASE=@String[booleanFormat],
            BOOLEAN_FORMAT_KEY=@String[boolean_format],
            OUTPUT_ENCODING_KEY_SNAKE_CASE=@String[output_encoding],
            OUTPUT_ENCODING_KEY_CAMEL_CASE=@String[outputEncoding],
            OUTPUT_ENCODING_KEY=@String[output_encoding],
            URL_ESCAPING_CHARSET_KEY_SNAKE_CASE=@String[url_escaping_charset],
            URL_ESCAPING_CHARSET_KEY_CAMEL_CASE=@String[urlEscapingCharset],
            URL_ESCAPING_CHARSET_KEY=@String[url_escaping_charset],
            STRICT_BEAN_MODELS_KEY_SNAKE_CASE=@String[strict_bean_models],
            STRICT_BEAN_MODELS_KEY_CAMEL_CASE=@String[strictBeanModels],
            STRICT_BEAN_MODELS_KEY=@String[strict_bean_models],
            AUTO_FLUSH_KEY_SNAKE_CASE=@String[auto_flush],
            AUTO_FLUSH_KEY_CAMEL_CASE=@String[autoFlush],
            AUTO_FLUSH_KEY=@String[auto_flush],
            NEW_BUILTIN_CLASS_RESOLVER_KEY_SNAKE_CASE=@String[new_builtin_class_resolver],
            NEW_BUILTIN_CLASS_RESOLVER_KEY_CAMEL_CASE=@String[newBuiltinClassResolver],
            NEW_BUILTIN_CLASS_RESOLVER_KEY=@String[new_builtin_class_resolver],
            SHOW_ERROR_TIPS_KEY_SNAKE_CASE=@String[show_error_tips],
            SHOW_ERROR_TIPS_KEY_CAMEL_CASE=@String[showErrorTips],
            SHOW_ERROR_TIPS_KEY=@String[show_error_tips],
            API_BUILTIN_ENABLED_KEY_SNAKE_CASE=@String[api_builtin_enabled],
            API_BUILTIN_ENABLED_KEY_CAMEL_CASE=@String[apiBuiltinEnabled],
            API_BUILTIN_ENABLED_KEY=@String[api_builtin_enabled],
            LOG_TEMPLATE_EXCEPTIONS_KEY_SNAKE_CASE=@String[log_template_exceptions],
            LOG_TEMPLATE_EXCEPTIONS_KEY_CAMEL_CASE=@String[logTemplateExceptions],
            LOG_TEMPLATE_EXCEPTIONS_KEY=@String[log_template_exceptions],
            STRICT_BEAN_MODELS=@String[strict_bean_models],
            SETTING_NAMES_SNAKE_CASE=@String[][isEmpty=false;size=20],
            SETTING_NAMES_CAMEL_CASE=@String[][isEmpty=false;size=20],
            parent=null,
            properties=@Properties[isEmpty=false;size=16],
            customAttributes=@HashMap[isEmpty=true;size=0],
            locale=@Locale[zh_CN],
            numberFormat=@String[number],
            timeFormat=@String[],
            dateFormat=@String[],
            dateTimeFormat=@String[],
            timeZone=@ZoneInfo[sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null]],
            sqlDataAndTimeTimeZone=null,
            sqlDataAndTimeTimeZoneSet=@Boolean[false],
            booleanFormat=@String[true,false],
            trueStringValue=null,
            falseStringValue=null,
            classicCompatible=@Integer[0],
            templateExceptionHandler=@TemplateExceptionHandler$3[freemarker.template.TemplateExceptionHandler$3@e6ea0c6],
            arithmeticEngine=@BigDecimalEngine[freemarker.core.ArithmeticEngine$BigDecimalEngine@6a38e57f],
            objectWrapper=@DefaultObjectWrapper[freemarker.template.DefaultObjectWrapper@1433867275(2.3.0, useAdaptersForContainers=false, forceLegacyNonListCollections=true, exposureLevel=1, exposeFields=false, sharedClassIntrospCache=none, ...)],
            outputEncoding=null,
            outputEncodingSet=@Boolean[false],
            urlEscapingCharset=null,
            urlEscapingCharsetSet=@Boolean[false],
            autoFlush=@Boolean[true],
            newBuiltinClassResolver=@TemplateClassResolver$1[freemarker.core.TemplateClassResolver$1@1c6b6478],
            showErrorTips=@Boolean[true],
            apiBuiltinEnabled=@Boolean[false],
            logTemplateExceptions=@Boolean[true],
            ALLOWED_CLASSES=@String[allowed_classes],
            TRUSTED_TEMPLATES=@String[trusted_templates],
            class$freemarker$template$TemplateExceptionHandler=null,
            class$freemarker$core$ArithmeticEngine=null,
            class$freemarker$template$ObjectWrapper=null,
            class$freemarker$core$TemplateClassResolver=null,
            class$freemarker$ext$beans$BeansWrapper=null,
        ],
    ],
    null,
    null,
]
```

