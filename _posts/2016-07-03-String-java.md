---
layout: post
title: Java-String之寻根问底 
categories: Java
description: Java-String之寻根问底。
keywords: Java
---

#### Java-String之寻根问底 


#  引言

在java编程中，几乎每天都会跟String打交道，因此，深入理解String及其用法十分有必要。下面分三方面来详细说明下String相关的特点及用法
•Immutable（不可变）特性
•连接符号+的本质
•相等判断两种方式（==/equals）说明

##  一、 Immutable特性

  Java设计人员为了方便大家对字符串的各种操作，抽象出String类，该类封装了对字符串的查找、拼接、替换、截取等一系列操作。查看java.lang.String的源码，首先就能看到如下描述：


The String class represents character strings. All string literals in Java programs, such as “abc”, are implemented as instances of this class. 
 Strings are constant; their values cannot be changed after they are created. String buffers support mutable strings. Because String objects are immutable they can be shared.

大意是：String类代表着字符序列。Java语言中所有的字符串字面量，如“abc”，都实现为String类的实例。 
String对象都是常量，其值在创建之后就不能改变。字符串缓冲区支持可变的字符序列。由于String对象的不可变性，他们可以被共享。

String的immutable体现在两个方面：

•String类被final关键字修饰，意味着该类不能被继承。由于String类不能有子类，保证了String类的静态和实例方法不可能被继承修改，保证了安全性。


•String类内部的私有成员变量，如value(char[])，offset(int)，count(int)都被final关键字修饰。当String对象创建后，offset跟count字段的值不可改变（因为是基本数据类型int），value变量不能再指向其它的字符数组（因为其是引用类型变量）


看到这里，可能有人会说：虽然value属性不能指向其它的字符数组，但其指向的字符数组内容还是可以改变，如果能够改变其内容，那不意味着String对象是可变的了。

上面的说法没错，但是String类本身没有提供修改字符数组的方法，除非你用非常规手段(例如反射)去改变私有属性的值（后面会附上代码实现）。虽然从代码上完全是可以改变创建后String对象的各属性的值（即使属性被private final修饰），但毕竟是采用反射这种非常规手段。按照正常使用方式，我们是不能改变String对象的值，所以还是认为String对象是不可变的。

注意：这里说String不变性，是指String对象在创建后其值不能改变。对于引用类型的变量，是可以指向不同的String对象来改变其所代表的值。如

```
String s = "abc"; 
s = "def";
```

引用类型变量s指向值为‘abc’的String对象，然后s又指向了值为‘def’的String对象。虽然s所代表的字符串确实改变了，但是对于String对象abc和def并没有改变，仅仅是s指向了不同的String对象而已。

关于String设计为immutable，至少有两方面的好处：

•一是安全。String类被final修饰，意味着不可能有子类继承String类而改变其原有行为。并且，生成的String对象是不变的，在多线程环境也是安全的。


•二是效率。String类被final修饰，隐含着该类的所有方法都是final的，编译器可以进行一些优化。另外，由于String对象是不变的，可以被多处共享且不需要进行多线程之间的同步，提高了效率。


由于String对象的不变性，在用+号进行字符串连接时，可能会造成效率低下，下面详细说明下连接符号+的本质是什么？底层是如何进行字符串拼接的？什么情况下用+号进行字符串连接效率较低？

##  二、 连接符号+本质

要了解+号的本质，先从java编译说起。众所周知，java代码在运行前都需要先编译成Class文件（关于Class文件的结构，由于篇幅有限，这里不作详细说明）。在Class文件中，有一部分是叫属性表集合，其中包括Code属性，简单说，Code属性包含的就是方法体里面的代码经过编译后对应的字节码指令。因此，我们可以直接查看Class文件中的字节码指令来了解+的本质。示例代码如下

```
public class StringTest {
    public static void main(String[] args) {
        String s = "Hello";
        s = s + " world!";
    }
}

```
由于我们不熟悉Class文件结构，而且字节码非常不容易看懂，在这里不直接查看编译生成的StringTest.class文件的内容，而是通过jad工具反编译字节码查看结果。在cmd下执行jad命令jad -o -a -sjava StringTest.class成功执行上述命令后，会发现StringTest.class文件所在目录下会多出源文件StringTest.java，内容如下：

```
public class StringTest
{

    public StringTest()
    {
    //    0    0:aload_0         
    //    1    1:invokespecial   #8   <Method void Object()>
    //    2    4:return          
    }

    public static void main(String args[])
    {
        String s = "Hello";
    //    0    0:ldc1            #16  <String "Hello">
    //    1    2:astore_1        
        s = (new StringBuilder(String.valueOf(s))).append(" world!").toString();
    //    2    3:new             #18  <Class StringBuilder>
    //    3    6:dup             
    //    4    7:aload_1         
    //    5    8:invokestatic    #20  <Method String String.valueOf(Object)>
    //    6   11:invokespecial   #26  <Method void StringBuilder(String)>
    //    7   14:ldc1            #29  <String " world!">
    //    8   16:invokevirtual   #31  <Method StringBuilder StringBuilder.append(String)>
    //    9   19:invokevirtual   #35  <Method String StringBuilder.toString()>
    //   10   22:astore_1        
    //   11   23:return          
    }
}



```

上述反编译出的源代码包含了注释行，代表与源代码相对应的字节码指令。很显然，源代码中并没有字符串连接符+，也就是说，+号在经过编译后，已经被替换成StringBuilder的append方法调用（实现上在jdk1.5版本之前，+号在编译器编译后是替换为StringBuffer的append方法调用）。所谓的+号连接字符串，本质上是通过new StringBuilder对象后调用其append方法进行字符串拼接。

java通过在编译阶段重载字符串操作符+，在方便对字符串的操作同时也带来了一定的副作用，比如由于程序员不清楚+号的本质而编写出效率低下的代码，请看如下代码：

   

```
 public String concat(){
        String result = "";
        for (int i = 0; i < 1000; i++) {
            result += i;
        }
        return result;
    }


```

在for循环体内，出现+号的地方，编译后都会被替换为如下调用： 

```
result = (new StringBuilder(String.valueOf(result))).append(i).toString(); 
```

显然，每次循环都需要在构造StringBuilder对象时对result中的字符数组进行拷贝，而在调用toString方法时，又要拷贝StringBuilder中的字符数组来构建String对象。相当于每次for循环，要进行两次对象创建及两次字符数组拷贝，因而程序效率低下。更高效的代码如下：

  

```
  public String concat(){
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            result.append(i);
        }
        return result.toString();
    }

```

至此，我相信大家已经知道了+号的本质，以及如何避免低效的使用+号。下面我们就来深入了解下String对象相等判断的两种方式（经常出现在java面试题中）。

##  三、 相等判断两种方式（==/equals）说明
•==：当两个操作数是基本数据类型时，比较值是否相等；当两个操作数是引用类型时，比较是否指向同一个对象。
•equals方法用来比较两个对象的内容是否相等。

由于String是引用类型，当用==判断时，比较的是两个String变量是否指向同一个String对象；当用equals方法判断时，才会比较两个String对象的内容是否相等。在实际项目中进行字符串比较时，基本是比较两个String对象的内容是否相等，因此建议大家全部使用equals方法进行比较。

用==进行字符串比较，经常出现在面试题中，而不是项目代码中，对于实际工作考核的意义不大，只是作为大家对String了解程度的一个考核。下面列举一些面试题如下：

题1

```
        String a = "a1";   
        String b = "a" + 1;   
        System.out.println(a == b);
 
```

       
答案：true 
说明：当两个字面量进行连接时，实际上在java编译器的编译期，已经进行了字面量的拼接。也就是说编译生成的Class文件中并不存在String b = “a” + 1对应的字节码指令，已经被优化为String b = “a1”对应的字节码指令。我想这步优化大家应该很能理解，在编译期间能够确定结果并进行计算，就能有效减少Class文件中的字节码指令，即减少了程序运行时需要执行的指令，提高了程序效率（大家可以用上述jad命令反编译Class文件进行验证）。同理，对于基本数据类型字面量的算术操作，也在编译期间进行了计算，例如int day = 24 * 60 * 60，编译后被替换成代码int day = 0x15180。由于在编译期间已经进行了拼接，这样局部变量a和b都指向了常量池中的’a1’对象，因此a == b输出为true。

题2

```
        String hw = "Hello world!";
        String h = "Hello";
        h = h + " world!";
        System.out.println(h == hw);

```


答案：false 
说明：通过之前关于字符串连接符+分析，我们知道h = h + " world!"经过编译后会被替换成h = (new StringBuilder(String.valueOf(h))).append(" world!").toString()。查看下StringBuilder的toString方法，可以看到该方法实际就是return new String(value, 0, count)，也就是h将指向java堆上的对象，而hw是指向常量池中的对象。虽然h和hw的内容相同，但由于指向不同的String对象，所以输出为false。

题3

```
    public static final String h2 = "Hello";

    public static final String h4 = getH();

    private static String getH() {  
        return "Hello";  
    }

    public static void main(String[] args) {
        String hw = "Hello world!";
        final String h1 = "Hello";
        final String h3 = getH();

        String hw1 = h1 + " world!";
        String hw2 = h2 + " world!";
        String hw3 = h3 + " world!";
        String hw4 = h4 + " world!";

        System.out.println(hw == hw1);
        System.out.println(hw == hw2);
        System.out.println(hw == hw3);
        System.out.println(hw == hw4);
    }
```


答案：true,true,false,false 
说明：局部变量h1被final修饰，意味着h1是常量，同时h1被直接赋值为字符串字面量”Hello”，这样java编译器在编译期就能确定h1的值，从而将h1出现的地方直接替换成字面量”Hello”(类似c/c++用define定义的常量)，再联系之前关于字面量会在编译期直接拼接说明，因此代码String hw1 = h1 + " world!"编译后优化为String hw1 = "Hello world!"，hw、hw1都指向了常量池中的String对象，输出为true。同理h2是静态常量，且是直接字面量赋值方式，h2出现的地方也会在编译后直接被字面量”Hello”替换，最终，hw2也是指向常量池中的String对象，输出为true。

局部变量h3也被final修饰，为常量，但是其是通过方法调用进行赋值的，编译期无法确定其具体值（此时代码都没执行，是无法通过静态分析得到方法的返回值的，即使方法体中只是简单的返回字符串常量，如上述例子），再联系之前关于+的本质分析，因此String hw3 = h3 + " world!"编译后为String hw3 = (new StringBuilder(String.valueOf(h3))).append(" world!").toString()，hw3将指向java堆上的String对象，hw == hw3输出为false。同理，hw4也指向java堆上的String对象，hw == hw4输出为false。

补充知识点

关于String类型变量的赋值，有两种方式：
•其一、直接字面量赋值，即String str = “abc”;
•其二、new方式赋值，即String str = new String(“abc”);

方式一中，变量str直接指向字符串常量池1中字面量为”abc”的String对象，即指向常量池中的String对象。 
 方式二中，变量str通过new构造函数String(String original)赋值，即指向java堆中的String对象。该构造函数接收String类型参数，而实参”abc”指向常量池中的String对象。

上面两种给String类型变量赋值的方式，除了它们指向不同的String对象外，其它并没有什么区别。从程序效率的角度看，推荐使用方式一给String类型变量赋值，因为方式二多了一次java堆的String对象分配。

前面说过，字符串字面量直接被看作String类的一个实例，实际是其在编译期就存放在Class文件的常量池中，当Class文件被jvm加载时，其就进入到方法区的运行时常量池中。如果想在运行期间将新的常量加入常量池中，可调用String的intern()方法。 
 当调用 intern方法时，如果常量池已经包含一个等于此String对象的字符串（用equals(Object)方法确定），则返回常池中的字符串。否则，将此String 对象添加到池中，并返回此String对象的引用。

附反射修改String对象代码：

   


```
 public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        String name = "angel";
        String name1 = "angel";

        Field strField = String.class.getDeclaredField("value");
        strField.setAccessible(true);
        char[] data = (char[])strField.get(name);
        data[4] = 'r';
        System.out.println(name);
        System.out.println(name1);
        System.out.println(name == name1);

        strField = String.class.getDeclaredField("count");
        strField.setAccessible(true);
        strField.setInt(name, 10);
        int i = (Integer)strField.get(name);
        System.out.println(i);
        System.out.println(name.length());
    }
   




```

1.字符串常量池是由String类管理，属于方法区的运行时常量池，也就是上文中所说的常量池 ↩



