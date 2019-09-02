# 1.下载与编译jdk

## 1.1下载jdk

openjdk8下载地址: [http://download.java.net/openjdk/jdk8u40/ri/openjdk-8u40-src-b25-10_feb_2015.zip ](http://download.java.net/openjdk/jdk8u40/ri/openjdk-8u40-src-b25-10_feb_2015.zip) 

Java类实现代码所在目录：`openjdk/jdk/src/share/classes`



## 1.2编译环境配置

下载jdk并解压压缩包之后，根目录中有名为`README-builds.html`的文件，里面详细介绍了编译源码所需的条件与相关事项。

以Ubuntu 16.04为例：

1. 准备bootstrap-jdk

   按照文档中的编译要求，如果我们编译的的是jdk8，那么bootstrap-jkd的版本应该时jdk7。也就是说，jdk的主版本号需要减一才能作为bootstrap-jdk。如下为官方文档中的说明：

   > **Building JDK 8 requires use of a version of JDK 7 that is at Update 7 or newer. JDK 8 developers should not use JDK 8 as the boot JDK, to ensure that JDK 8 dependencies are not introduced into the parts of the system that are built with JDK 7.** 

   所以，在正式编译jdk之前，需要下载一个jdk7，下载之后将其解压到任何位置，后面配置`configure`指定bootstrap-jdk的目录为该解压目录。

   

2. 打开终端，进入jdk解压之后的根目录，可以看到`configure`文件![dirlist](assets/dirlist.png)

   终端中执行`bash configure`，检查编译环境是否满足要求，执行过程中会在`openjdk`目录中生成一些文件。指定bootstrap-jdk的目录使用`--with-boot-jdk=`参数，命令为：

   `bash configure --with-boot-jdk=/home/xanarry/Desktop/java-se-7u75-ri`

   如果环境OK的话，终端最后会输出如下结果：

   ```
   configure: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/config.status
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/spec.gmk
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/hotspot-spec.gmk
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/bootcycle-spec.gmk
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/compare.sh
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/spec.sh
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/Makefile
   config.status: creating /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/config.h
   config.status: /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/config.h is unchanged
   
   ====================================================
   A new configuration has been successfully created in
   /home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release
   using default settings.
   
   Configuration summary:
   * Debug level:    release
   * JDK variant:    normal
   * JVM variants:   server
   * OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
   
   Tools summary:
   * Boot JDK:       java version "1.8.0_201" Java(TM) SE Runtime Environment (build 1.8.0_201-b09) Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)  (at /opt/jdk1.8.0_201)
   * C Compiler:     gcc-5 (Ubuntu 5.4.0-6ubuntu1~16.04.11) 5.4.0 version 20160609 (at /usr/bin/gcc-5)
   * C++ Compiler:   g++-5 (Ubuntu 5.4.0-6ubuntu1~16.04.11) 5.4.0 version 20160609 (at /usr/bin/g++-5)
   
   Build performance summary:
   * Cores to use:   7
   * Memory limit:   15798 MB
   * ccache status:  installed, but disabled (version older than 3.1.4)
   
   WARNING: The result of this configuration has overridden an older
   configuration. You *should* run 'make clean' to make sure you get a
   proper build. Failure to do so might result in strange build problems.
   ```

   如果环境检查没有成功会提示一个`error:1`，不过没关系，会提示缺少哪些依赖库，依次安装上就行，直到检查成功。

   

## 1.3编译代码

在openjdk根目录下执行：`make` 即可。

**注意：**如果机器的内核版本大于等于4.x，那么会报错如下：

   ```
Building OpenJDK for target 'default' in configuration 'linux-x86_64-normal-server-release'
   
## Starting langtools
   
Compiling 2 files for BUILD_TOOLS
Compiling 32 properties into resource bundles
Compiling 781 files for BUILD_BOOTSTRAP_LANGTOOLS
Creating langtools/dist/bootstrap/lib/javac.jar
Updating langtools/dist/lib/src.zip
Compiling 784 files for BUILD_FULL_JAVAC
Creating langtools/dist/lib/classes.jar
   
## Finished langtools (build time 00:00:38)
   
## Starting hotspot
   
make[2]: warning: -jN forced in submake: disabling jobserver mode.
INFO: ENABLE_FULL_DEBUG_SYMBOLS=1
INFO: ALT_OBJCOPY=/usr/bin/objcopy
INFO: /usr/bin/objcopy cmd found so will create .debuginfo files.
INFO: STRIP_POLICY=min_strip
INFO: ZIP_DEBUGINFO_FILES=1
INFO: ENABLE_FULL_DEBUG_SYMBOLS=1
INFO: ALT_OBJCOPY=/usr/bin/objcopy
INFO: /usr/bin/objcopy cmd found so will create .debuginfo files.
INFO: STRIP_POLICY=min_strip
INFO: ZIP_DEBUGINFO_FILES=1
INFO: ENABLE_FULL_DEBUG_SYMBOLS=1
INFO: ALT_OBJCOPY=/usr/bin/objcopy
INFO: /usr/bin/objcopy cmd found so will create .debuginfo files.
INFO: STRIP_POLICY=min_strip
INFO: ZIP_DEBUGINFO_FILES=1
*** This OS is not supported: Linux ThinkPad 4.15.0-47-generic #50~16.04.1-Ubuntu SMP Fri Mar 15 16:06:21 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
/home/xanarry/Desktop/openjdk/hotspot/make/linux/Makefile:238: recipe for target 'check_os_version' failed
make[5]: *** [check_os_version] Error 1
/home/xanarry/Desktop/openjdk/hotspot/make/linux/Makefile:259: recipe for target 'linux_amd64_compiler2/debug' failed
make[4]: *** [linux_amd64_compiler2/debug] Error 2
make[3]: *** [generic_build2] Error 2
Makefile:230: recipe for target 'generic_build2' failed
make[2]: *** [product] Error 2
Makefile:177: recipe for target 'product' failed
HotspotWrapper.gmk:44: recipe for target '/home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/hotspot/_hotspot.timestamp' failed
make[1]: *** [/home/xanarry/Desktop/openjdk/build/linux-x86_64-normal-server-release/hotspot/_hotspot.timestamp] Error 2
/home/xanarry/Desktop/openjdk//make/Main.gmk:108: recipe for target 'hotspot-only' failed
make: *** [hotspot-only] Erro
   ```

从错误中可以看出，在`openjdk/hotspot/make/linux/Makefile`第238行提示错误，是因为这个`Makefile`会检查内核版本是否为

`SUPPORTED_OS_VERSION = 2.4% 2.5% 2.6% 3%`

也就是要求内核版本小于4.x，所以在末尾加上“**4%**”即可

`SUPPORTED_OS_VERSION = 2.4% 2.5% 2.6% 3% 4%`

编译成功之后可以看到make最后的输出为：

   ```
## Finished jdk (build time 00:02:44)
   
----- Build times -------
Start 2019-04-07 12:42:33
End   2019-04-07 12:50:33
00:00:28 corba
00:04:02 hotspot
00:00:19 jaxp
00:00:27 jaxws
00:02:44 jdk
00:00:00 langtools
00:08:00 TOTAL
-------------------------
Finished building OpenJDK for target 'default'
   ```

## 1.4编译结果

编译的结果会保存到openjdk根目录中的`openjdk/build `目录下，以下是我本次的编译结果，diz文件包含了调试信息。

![Selection_002](assets/Selection_002.png)

java的版本输出：

![](assets/java-version-outpu.png)















# 2.java内存区域与内存溢出异常

java虚拟机运行时数据区

![](assets/java-runtime-data-area.jpg)

程序计数器：一块较小的内存，记录java程序字节码当前执行到的位置。程序计数器仅仅对java方法有效，当程序执行到native方法时，计数器将被置为空（undefined）。

虚拟机栈：虚拟机执行java方法所使用的栈，主要存储局部变量，操作数栈，方法入口信息，函数调用等信息。该区域会发生两种异常一种是线程请求的栈深度超过了所允许的深度（例如程序进入了死递归），抛出StackOverflow异常；另一种是OutOfMemoryError异常，即程序在扩展过程中无法请求到足够的内存。

本地方法栈：与虚拟机栈一样，保存本地方法在执行过程中的局部变量，操作数，维持函数调用信息等。



方法区：与java堆一样，是各个线程共享的内存区域，他用于存储已被虚拟机加载的类信息，常量，静态变量，字段描述，方发描述及时编译器编译后的代码等数据。该区域别名也叫非堆（Non-Heap），与java堆区别开来。许多开发者也称其为永久代（permanent generation）。该区域在内存无法分配时会抛出OutOfMemoryError异常。   同时，方法区还包括运行时常量区（Runtime Constant Pool）。

堆：java堆在虚拟机启动时候创建，是虚拟机所管理的最大的一块空间，也是垃圾回收器管理的主要区域。从垃圾回收的角度来看，可以被分为：新生代和老年代，更细致为**Eden空间（占80%）、From Survivor空间（占10%）、To Survivor空间（占10%）**， 此区域的唯一目的就是存放对象实例，几乎所有的对象实例都要在堆上分配内存。java堆可以处于物理上不连续的空间，只要逻辑上是连续的即可。



java内存模型：

![](assets/java-mem-structure.jpg)

**整个JVM内存大小=年轻代大小 + 年老代大小 + 持久代大小**





控制该区域容量的相关参数：

| 参数名 |                             作用                             |
| :----: | :----------------------------------------------------------: |
|  -Xss  |                    设置每个线程的堆栈大小                    |
|  -Xmx  |                    设置JVM最大可用内存为                     |
|  -Xms  | 设置堆的最小容量，虚拟机启动后就分配这个值。当最小堆占满后，会尝试进行GC， |
|  -Xmn  |                        设置新生代大小                        |



各个参数控制的内存区域：

![](assets/java-mem-and-parameter.jpg)



































# 3.垃圾收集器与内存分配策略





#  4.虚拟机性能监控与故障处理工具





# 5.调优方案与实战



# 6.类文件结构



# 7.虚拟机类加载机制




# 8.虚拟机字节码与执行引擎


# 9.类加载及执行子系统的案例与实战


# 10.编译期（早期）优化

# 11.运行时（晚期）优化

# 12.Java内存模型与线程

# 13.线程安全与锁优化