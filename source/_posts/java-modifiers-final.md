---
title: Java-修饰符-final 
date: 2023-08-15 10:59:17.0
updated: 2023-09-17 00:28:30.125
url: /archives/java-modifiers-final
categories: 技术笔记
tags: 编程语言
---

# final 

初识 final 几年，现在再回头看，是否能够对 final 有新的理解？



## Java 语言规范中对 `final` 的定义

- 修饰类 （Class）：不能被任何类继承
- 修饰方法(Method)：不能被子类重写
- 修饰变量 (Fields)：完成初始化（静态和实例）后不能被更改

### final Class (§8.1.1.2)


 A class can be declared final if its definition is complete and no subclasses are desired or required.
 
**It is a compile-time error if the name of a final class appears in the extends clause (§8.1.4) of another class declaration; this implies that a final class cannot have any subclasses.**

**It is a compile-time error if a class is declared both final and abstract, because the implementation of such a class could never be completed (§8.1.1.1).**

Because a final class never has any subclasses, the methods of a final class are never overridden (§8.4.8.1).

> 简单来说：一个类不想要被任何类继承，就用 final 修饰此类。有其他类想继承被 final 修饰类的会在编译期报错的（编译上限制不能被继承），还有一种情况是抽象类想要修饰final 也是会被报错的（和 abstract 所违背，抽象类是让子类去做个继承实现，而 final 的目的是为了不让任何类去继承）。 

### final Method (§8.4.3.3)


A method can be declared final to prevent subclasses from overriding or `hiding` it.

**It is a compile-time error to attempt to override or hide a final method.**

A private method and all methods declared immediately within a final class behave as if they are final, since it is impossible to override them.
> 简单来说：防止子类对重写某个方法，用 final 修饰此方法。试图去重写被 final 修饰的父类方法会在编译期报错的。一个私有方法和final修饰类的所有方法表现会像是被final修饰一样的，因为它们不会被重写。


*At run time, a machine-code generator or optimizer can "inline" the body of a final method, replacing an invocation of the method with the code in its body.
The inlining process must preserve the semantics of the method invocation. 
In particular, if the target of an instance method invocation is null, then a NullPointerException must be thrown even if the method is inlined.
A Java compiler must ensure that the exception will be thrown at the correct point, so that the actual arguments to the method will be seen to have been evaluated in the correct order prior to the method invocation.*

> 在运行时，机器语言生成器或者是优化器能做“内联”，一个方法调用会把它调用的 final 修饰方法的方法体，调用给合成 final 方法的方法体。在内联的过程中必须得保证方法调用原本的语义，特别是在异常处理上，方法可能是内联的。Java 编译器必须保证异常抛在正确的点上。

```java
// Consider the example:  
final class Point {
    int x, y;
    void move(int dx, int dy) { x += dx; y += dy; }
}
class Test {
    public static void main(String[] args) {
        Point[] p = new Point[100];
        for (int i = 0; i < p.length; i++) {
            p[i] = new Point();
            p[i].move(i, p.length-1-i);
        }
    }
}
```


```Java
// Inlining the method move of class Point in method main 
// would transform the for loop to the form:

// 内联 Point 类的 move 会被转换成为这种：

    for (int i = 0; i < p.length; i++) {
        p[i] = new Point();
        Point pi = p[i];
        int j = p.length-1-i;
        pi.x += i;
        pi.y += j;
    }


// The loop might then be subject to further optimizations.
// Such inlining cannot be done at compile time
// unless it can be guaranteed that Test and Point will always be recompiled together,
// so that whenever Point - and specifically its move method - changes, 
// the code for Test.main will also be updated.

// 这个循环可能会被进一步优化。
// 这样的内联不能在编译时做完成
// 除非能够保证 Test 和 Point 总是能够在一起重新编译。
// 这样每次 Point 发生改变，尤其是改变它的move方法，Test.main 也得更新。
```




### final Fields

 A field can be declared final (§4.12.4). Both class and instance variables (static and non-static fields) may be declared final.
A blank final class variable must be definitely assigned by a static initializer of the class in which it is declared, or a compile-time error occurs (§8.7, §16.8).
A blank final instance variable must be definitely assigned at the end of every constructor of the class in which it is declared, or a compile-time error occurs (§8.8, §16.9).

> 简单来说： 一个字段可以被 final 修饰，静态和非静态字段都是可以被final 修饰的。一个未赋值的静态变量必须要在这个类的静态初始化时确定值，否则编译期报错。一个未赋值的实例变量必须要在构造实例结束前确定确定值，否认编译报错。

## JVM 角度的 final 

### final 修饰类的字节码

使用 javap 查看被 final 修饰类的字节码中的 `acc_flags` （用来表示访问全权限）。
![image-1692081074704](https://source.youchuanlong.com/blog/image-1692081074704.png)

![image-1692080839437](https://source.youchuanlong.com/blog/image-1692080839437.png)

>  ACC_FINAL: 不允许有子类


### final 修饰方法
方法也是有 `acc_flags` 和类的一样： ACC_FINAL

![image-1692081629277](https://source.youchuanlong.com/blog/image-1692081629277.png)

> ACC_FINAL: 不允许被覆盖。

### final 修饰的变量 

变量也有 `acc_flags` ,同上：ACC_FINAL。

![image-1692082038697](https://source.youchuanlong.com/blog/image-1692082038697.png)


> ACC_FLAGS: 在创建对象后不会被改变


是变量就会和存储有关，看看有什么不同吗。

#### 静态 final 变量

在一般情况下，我们要定义常量，一般都推荐用 final 修饰静态变量，保证常量不可变。

final 修饰是静态变量是放在字节码的常量池，A 类 调用定义在 B 类中定义的常量，该常量直接放在 A 类的常量池。（Classloader 没加载 B.class 也是可以的，意味着不会触发 B 被加载，可以把 B 的 class 文件给删掉来验证。）

```java
// A:  FinalExample class 
public  class FinalExample {
	public static void main(String[] args) {
    	// 调用另一个类的常量
		System.out.println(FinalConst.CONST);
	}
}
```


```java

// B: FinalConst class 
public class FinalConst {
	public static final int CONST = 20230815;
}

```

FinalExample 的常量池和 main 方法中的调用：

```
// #### 常量池：
Constant pool:
   #1 = Methodref          #7.#21         // java/lang/Object."<init>":()V
   #2 = Fieldref           #22.#23        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Class              #24            // com/example/spec/FinalConst
   #4 = Integer            20230815     // 可以看到在定义的常量
   # ...
{
  // ... 
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #4                  // int 20230815 ，ldc: 将常量池的操作数 push 到操作栈中
         5: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
         8: return
      LineNumberTable:
        line 12: 0
        line 13: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
}
```


##### 加 final 和 不加 final 的静态变量有什么不同？

- 我验证的不加 final 也会存在常量池中 ？？？
- ChatGTP:
```txt
# Question: 

加 final 和 不加final 修饰的静态变量，除了保证不可变外有什么区别 ，都会存常量池吗?

# Answer:

不是所有的静态变量都会被存储在常量池中，final 修饰的静态变量和非 final 修饰的静态变量在这方面可能会有一些区别。让我解释一下：

final 修饰的静态变量： 使用 final 修饰的静态变量在编译期间就会被确定其值，并且在类加载时进行初始化。这种情况下，它们的值会被存储在常量池中，以便在编译时被优化和直接使用。因此，final 修饰的静态变量通常会存储在常量池中。

非 final 修饰的静态变量： 非 final 修饰的静态变量的值在编译期间无法确定，它们会在类加载时被初始化。这些变量的值不会被存储在常量池中，而是存储在类的静态变量区域中。

需要注意的是，即使变量的值存储在常量池中，其内存地址并不一定相同。在某些情况下，不同的类加载器可能会导致同一个常量池中的值在内存中存在多个副本。这取决于类加载器的实现和加载过程。

综上所述，final 修饰的静态变量通常会存储在常量池中，但非 final 修饰的静态变量的值会存储在静态变量区域中，而不是常量池。
```


#### 实例 final 变量


```java
public  class FinalExample {

	// 没有 final 修饰的实例
	private int y = 1999;

	public static void main(String[] args) {
		// final 修饰的实例变量
		int val = new FinalVal().x;
		int val2 = new FinalExample().y;
	}

}
```

```java
public class FinalVal {
	public final int x = 20230815;
}
```

```
Constant pool:
   #1 = Methodref          #9.#27         // java/lang/Object."<init>":()V
   #2 = Fieldref           #7.#28         // com/example/spec/FinalExample.y:I
   #3 = Class              #29            // com/example/spec/FinalVal
   #4 = Methodref          #3.#27         //  com/example/spec/FinalVal."<init>":()V
   #5 = Methodref          #30.#31        // java/util/Objects.requireNonNull:(Ljava/lang/Object;)Ljava/lang/Object;
   #6 = Integer            20230815  // 可以看到常量池中有final 声明的实例常量的值
  #...
{

	// 无参实例方法
  public com.example.spec.FinalExample();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: sipush        1999  // 可以看到实例变量其实是在构造中赋值的
         8: putfield      #2                  // Field y:I
        11: return
      LineNumberTable:
        line 7: 0
        line 12: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      12     0  this   Lcom/example/spec/FinalExample;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #3                  // class com/example/spec/FinalVal
         3: dup
         4: invokespecial #4                  // Method com/example/spec/FinalVal."<init>":()V
         7: invokestatic  #5                  // Method java/util/Objects.requireNonNull:(Ljava/lang/Object;)Ljava/lang/Object;
        10: pop
        11: ldc           #6                  // int 20230815
        13: istore_1
        14: new           #7                  // class com/example/spec/FinalExample
        17: dup
        18: invokespecial #8                  // Method "<init>":()V
        21: getfield      #2                  // Field y:I,getfield
        24: istore_2
        25: return
      LineNumberTable:
        line 16: 0
        line 17: 14
        line 18: 25
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      26     0  args   [Ljava/lang/String;
           14      12     1   val   I
           25       1     2  val2   I
}
SourceFile: "FinalExample.java"
```


#### 局部 final 变量
平时会用 final 在修饰局部变量，在它本身的语义上保证变量不可变，另外来提高代码的阅读性。

```Java
// 表明这个变量值不要改动。（个人习惯）
final boolean isInterruptIfRunning = false;
future.cancel(isInterruptIfRunning);
```
发现 final 在一般的方法中修饰变量，除了保持final的特性外，没什么不同，都是存储在局部方法的局部变量中。但是 main 方法中的修饰的变量除了局部变量表外，还会存在常量池中。


```java
package com.example.spec;

import java.io.PrintStream;

/**
 * @author YouChuanlong
 * Created at 2023/8/15 14:18
 */
public  class FinalExample {

	private final static PrintStream OUT = System.out;

	public static void main(String[] args) {
		// main 方法中声明的变量也会存在常量池中
		final int a = 2000217;
		final int b = 2000218;
		int c = 2000219;
		FinalVal valObj = new FinalVal();  // 验证对象
		hello();
	}

	public static void hello() {
		// 静态方法中修饰变量
		final int helloA = 4321;
		int helloB = 8642; // A B 指令是否相同
		OUT.println(helloA);
	}

	public void hey() {
		// 实例方法中修饰变量
		final int heyVal = 9765;
		OUT.println(heyVal);
	}

}
```


```
 # 常量池
Constant pool:
   #1 = Methodref          #12.#40        // java/lang/Object."<init>":()V
   #2 = Integer            2000217	      // main 方法中定义	
   #3 = Integer            2000218		  // main 方法中定义
   #4 = Integer            2000219		  // main 方法中定义
   #5 = Class              #41            // com/example/spec/FinalVal
   #6 = Methodref          #5.#40         // com/example/spec/FinalVal."<init>":()V
   #7 = Methodref          #11.#42        // com/example/spec/FinalExample.hello:()V
   #8 = Fieldref           #11.#43        // com/example/spec/FinalExample.OUT:Ljava/io/PrintStream;
   // ... 
{

  // ...
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=5, args_size=1
         0: ldc           #2                  // int 2000217
         2: istore_1
         3: ldc           #3                  // int 2000218
         5: istore_2
         6: ldc           #4                  // int 2000219
         8: istore_3
         9: new           #5                  // class com/example/spec/FinalVal
        12: dup
        13: invokespecial #6                  // Method com/example/spec/FinalVal."<init>":()V
        16: astore        4
        18: invokestatic  #7                  // Method hello:()V
        21: return
      LineNumberTable:
       	// ... 
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      22     0  args   [Ljava/lang/String;
            3      19     1     a   I
            6      16     2     b   I
            9      13     3     c   I
           18       4     4 valObj   Lcom/example/spec/FinalVal;

  public static void hello();
    descriptor: ()V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=0
         0: sipush        4321
         3: istore_0
         4: sipush        8642
         7: istore_1
         8: getstatic     #8                  // Field OUT:Ljava/io/PrintStream;
        11: sipush        4321
        14: invokevirtual #9                  // Method java/io/PrintStream.println:(I)V
        17: return
      LineNumberTable:
       // ...
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            4      14     0 helloA   I
            8      10     1 helloB   I

  public void hey();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=1
         0: sipush        9765
         3: istore_1
         4: getstatic     #8                  // Field OUT:Ljava/io/PrintStream;
         7: sipush        9765
        10: invokevirtual #9                  // Method java/io/PrintStream.println:(I)V
        13: return
      LineNumberTable:
       // ...
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      14     0  this   Lcom/example/spec/FinalExample;
            4      10     1 heyVal   I

	// ...
SourceFile: "FinalExample.java"
```

## 学习参考
- [The Java® Language Specification Java SE 8 Edition](https://docs.oracle.com/javase/specs/jls/se8/html/index.html)
- [The Java® Virtual Machine Specification
Java SE 8 Edition](https://docs.oracle.com/javase/specs/jvms/se8/html/)