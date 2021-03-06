---
title: Java类文件结构解析（一）
date: 2017-10-25 17:01:51
categories: Java
tags: 虚拟机
thumbnail: /images/javalogo.png
---

![](/images/javalogo.png)
作为一个Java开发，每天编写的Java代码都是成百上千行，略去枯燥的业务代码不提，追寻代码从编译成class文件开始，虚拟机如何加载、解析这些代码的过程，一定要比单纯写业务代码有意思的多。

# 一、class文件的结构
我们日常开发的过程大致是这样的，我要提供个某某功能的服务，所以，我们开始巴拉巴拉、、

``` java
public class AaBaServiceImpl implements AaBaService {
   //巴拉巴拉一堆代码：服务依赖、变量定义、方法定义、常量、日志 等等等
}

```
<!-- more -->
是不是机械的很，熟练的不行啊，快飞起来了。瞬间觉得自己狂拽掉咋天。BUT FUCK为啥我非要写成这样？我可是程序员啊 ，创造力无限的说，咋就不能一朝顿悟，一掌辟出个黯然销魂掌？答曰：那必须不能啊，因为神雕侠侣特么它不是你写的啊。类比可知：Java特么也不是你发明的，人家的规范早就定义好了，在这个基础规范里面，文件结构那只能这么搞，为了一个崇高的“梦想”：师夷长技以制夷。万一哪天牛逼了，一不小心独创了个语言呢？人家杨过那也是厚积薄发，内力深厚，阅经沧桑，不然还销魂啥啊，没等销魂，就被蛤蟆功撑死了。所以，先积一下。来看看这个[虚拟机规范](http://docs.oracle.com/javase/specs/jvms/se7/html/index.html) ，章节翻到第四节：Chapter 4. The class File Format，类文件格式：

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}

```

这里u4、u2表示的4个字节，2个字节的意思。但是这么看起来还是很抽象，不明觉厉。那先画个图给他串一下，顺便，把这几个英文的意思也标记一下：

![](/images/javabytecode.png)
对照着图，分别简单聊一下这些class文件的结构内容：

- magic：魔数。值是十六进制：CAFEBABE。作用是标记文件的身份，也就是标记该文件是否可以被虚拟机接受。

- minor_version: 次版本号。
- major_version：主版本号。

这两哥们决定了类文件版本格式。简单的说，就是决定类文件可以被哪个版本的虚拟机执行。假设minor_version的值是M，major_version的值是m，那么这个类的版本号就是M.m。

- constant_pool_count：常量池统计数。表示常量池的大小。

- constant_pool：常量池。大小是constant_pool_count-1。常量池的主要内容是字面量和符号引用。

- access_flags：访问标志。表示类或者接口的访问信息。比如是Class还是Interface，定义域是不是public，Class是不是抽象的等。

- this_class: 类索引。标记这个类的全量名，指向常量值的字面量。
- super_class: 父类索引。标记类父类的全量名。
- interfaces_count：接口计数器。表示接口的数量。
- interfaces：具体的接口信息。
- fields_count：字段信息计数器，用来描述接口或者类中的声明的变量。
- fields: 字段的具体信息。
- methods_count：方法表计数器。
- methods：方法表具体信息。
- attributes_count：属性表计数器。
- attributes：属性表具体信息。

# 二、具体代码实例的对比
拿到类文件的结构以后，我们写一个简单的java代码做个对比，源文件代码如下：
``` java
public class JavaCodeTest {

    private String test = "java code test";

    public String getTest(){
        return "ceshi";
    }
}
```
有了源文件以后，需要把源文件编译成Java字节码，以供我们学习比对，首先执行javac JavaCodeTest.java，编译成class文件，之后需要查看编译后的class文件以及其二进制的内容，这里简单介绍一下具体方法：

### 1.命令行方式查看class文件的内容：
``` linux
javap -verbose JavaCodeTest

```

执行后的结果会带常量池的具体信息，如果不想查看常量池，执行javap -c filenName即可：

``` java
Compiled from "JavaCodeTest.java"
public class JavaCodeTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#17         // java/lang/Object."<init>":()V
   #2 = String             #18            // java code test
   #3 = Fieldref           #5.#19         // JavaCodeTest.test:Ljava/lang/String;
   #4 = String             #20            // ceshi
   #5 = Class              #21            // JavaCodeTest
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               test
   #8 = Utf8               Ljava/lang/String;
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               getTest
  #14 = Utf8               ()Ljava/lang/String;
  #15 = Utf8               SourceFile
  #16 = Utf8               JavaCodeTest.java
  #17 = NameAndType        #9:#10         // "<init>":()V
  #18 = Utf8               java code test
  #19 = NameAndType        #7:#8          // test:Ljava/lang/String;
  #20 = Utf8               ceshi
  #21 = Utf8               JavaCodeTest
  #22 = Utf8               java/lang/Object
{
  public JavaCodeTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String java code test
         7: putfield      #3                  // Field test:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 4: 0
        line 5: 4

  public java.lang.String getTest();
    descriptor: ()Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: ldc           #4                  // String ceshi
         2: areturn
      LineNumberTable:
        line 8: 0
}
SourceFile: "JavaCodeTest.java"

```
### 2.命令行查看十六进制的class文件内容
``` java
  1）首先以二进制方式编辑这个文件：vi -b datafile
  2）使用xxd转换为16进制：:%!xxd
  3）转换16进制回来vi：:%!xxd -r
```

执行完命令1跟2后，命令行显示内容如下：

``` java
00000000: cafe babe 0000 0034 0017 0a00 0600 1108  .......4........
00000010: 0012 0900 0500 1308 0014 0700 1507 0016  ................
00000020: 0100 0474 6573 7401 0012 4c6a 6176 612f  ...test...Ljava/
00000030: 6c61 6e67 2f53 7472 696e 673b 0100 063c  lang/String;...<
00000040: 696e 6974 3e01 0003 2829 5601 0004 436f  init>...()V...Co
00000050: 6465 0100 0f4c 696e 654e 756d 6265 7254  de...LineNumberT
00000060: 6162 6c65 0100 0767 6574 5465 7374 0100  able...getTest..
00000070: 1428 294c 6a61 7661 2f6c 616e 672f 5374  .()Ljava/lang/St
00000080: 7269 6e67 3b01 000a 536f 7572 6365 4669  ring;...SourceFi
00000090: 6c65 0100 114a 6176 6143 6f64 6554 6573  le...JavaCodeTes
000000a0: 742e 6a61 7661 0c00 0900 0a01 000e 6a61  t.java........ja
000000b0: 7661 2063 6f64 6520 7465 7374 0c00 0700  va code test....
000000c0: 0801 0005 6365 7368 6901 000c 4a61 7661  ....ceshi...Java
000000d0: 436f 6465 5465 7374 0100 106a 6176 612f  CodeTest...java/
000000e0: 6c61 6e67 2f4f 626a 6563 7400 2100 0500  lang/Object.!...
000000f0: 0600 0000 0100 0200 0700 0800 0000 0200  ................
00000100: 0100 0900 0a00 0100 0b00 0000 2700 0200  ............'...
00000110: 0100 0000 0b2a b700 012a 1202 b500 03b1  .....*...*......
00000120: 0000 0001 000c 0000 000a 0002 0000 0004  ................
00000130: 0004 0005 0001 000d 000e 0001 000b 0000  ................
00000140: 001b 0001 0001 0000 0003 1204 b000 0000  ................
00000150: 0100 0c00 0000 0600 0100 0000 0800 0100  ................
```

这个内容看起来不纯粹，进一步把class文件用sublime编辑工具打开，打开后的效果如下：

``` java
cafe babe 0000 0034 0017 0a00 0600 1108
0012 0900 0500 1308 0014 0700 1507 0016
0100 0474 6573 7401 0012 4c6a 6176 612f
6c61 6e67 2f53 7472 696e 673b 0100 063c
696e 6974 3e01 0003 2829 5601 0004 436f
6465 0100 0f4c 696e 654e 756d 6265 7254
6162 6c65 0100 0767 6574 5465 7374 0100
1428 294c 6a61 7661 2f6c 616e 672f 5374
7269 6e67 3b01 000a 536f 7572 6365 4669
6c65 0100 114a 6176 6143 6f64 6554 6573
742e 6a61 7661 0c00 0900 0a01 000e 6a61
7661 2063 6f64 6520 7465 7374 0c00 0700
0801 0005 6365 7368 6901 000c 4a61 7661
436f 6465 5465 7374 0100 106a 6176 612f
6c61 6e67 2f4f 626a 6563 7400 2100 0500
0600 0000 0100 0200 0700 0800 0000 0200
0100 0900 0a00 0100 0b00 0000 2700 0200
0100 0000 0b2a b700 012a 1202 b500 03b1
0000 0001 000c 0000 000a 0002 0000 0004
0004 0005 0001 000d 000e 0001 000b 0000
001b 0001 0001 0000 0003 1204 b000 0000
0100 0c00 0000 0600 0100 0000 0800 0100
0f00 0000 0200 10

```
这样看起来清爽多了，没有无关的东西了，只剩下纯粹的16进制码了。

这里多说一句，Java中byte用二进制表示要占用8位，而16进制的每个字符需要用4位二进制位来表示，所以我们就可以把每个byte转换成两个相应的16进制字符，即把byte的高4位和低4位分别转换成相应的16进制字符，并组合起来得到byte转换到16进制字符串的结果。即byte用十六进制表示只占2位。同理，相反的转换也是将两个16进制字符转换成一个byte，原理同上。

接下来会讲一下对照着这些十六进制码如何查看class字节码的具体内容~~~
