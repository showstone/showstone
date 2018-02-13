---
layout: post
title: "java8里lambda里的 this 为什么会指向 lamdba 所在的外部类"
description: ""
category: JAVA
tags: [java8,lambda,this]
---

这几天复习了 lambda ，发现有个细节，十分难以理解，那就是 lambda 里的 this指针。

Lambda 里的this指针指向其所属的内部类， 是怎么实现的呢？

写了一个例子，作为测试:

```java
import java.util.function.Supplier;

public class LambdaTest {
    public static void main(String[] args) {
        new LambdaTest().test();
    }

    public void test() {
        String para = "abc";
        String para2 = "abc";
        System.out.println(this);
        Supplier<String> supplier = () -> {
            return para + para2;
        };
        System.out.println(supplier.get());
    }
}
```

对应的jvm 机器码是:
```
  private static java.lang.String lambda$test$0(java.lang.String, java.lang.String);
    descriptor: (Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: new           #12                 // class java/lang/StringBuilder
         3: dup
         4: invokespecial #13                 // Method java/lang/StringBuilder."<init>":()V
         7: aload_0
         8: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        11: aload_1
        12: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        15: invokevirtual #15                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        18: areturn
      LineNumberTable:
        line 16: 0
```

通过方法签名可以知道，如果一个类没有带 this,被编译成了一个静态内部类方法。


带 this：
```java
import java.util.function.Supplier;

public class LambdaTest {
    public static void main(String[] args) {
        new LambdaTest().test();
    }

    public void test() {
        String para = "abc";
        String para2 = "abc";
        System.out.println(this);
        Supplier<String> supplier = () -> {
            System.out.println(this);
            return para + para2;
        };
        System.out.println(supplier.get());
    }
}
```

对应的 jvm 机器码:
```
  private java.lang.String lambda$test$0(java.lang.String, java.lang.String);
    descriptor: (Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    flags: ACC_PRIVATE, ACC_SYNTHETIC
    Code:
      stack=2, locals=3, args_size=3
         0: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_0
         4: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
         7: new           #12                 // class java/lang/StringBuilder
        10: dup
        11: invokespecial #13                 // Method java/lang/StringBuilder."<init>":()V
        14: aload_1
        15: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        18: aload_2
        19: invokevirtual #14                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        22: invokevirtual #15                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        25: areturn
      LineNumberTable:
        line 15: 0
        line 16: 7
```

lambda 被编译成了一种内部类！这就能说通了。

结论:
> lambda一般情况下会被编译成静态匿名方法，引用的外部变量以参数的方式传递。
> 如果 lambda 里使用了this 指标，则被编译为匿名内部方法，以让 this 指针指向lambda 外部类。



