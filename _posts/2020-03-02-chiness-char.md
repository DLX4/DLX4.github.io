---
title: C语言和JAVA是怎么保存中文字符的
url: 431.html
id: 431
categories:
  - 编程
date: 2020-03-02 22:12:13
tags:
  - 中文字符
---

给定下面几个题目，聪明的你们能否很快答得上来呢？ 1、C语言能够保存中文字符吗？是怎么保存的呢？ 2、java能够保存中文字符吗？是怎么保存的呢？ 对于程序员来说最囧的事情莫过于此，每天都在使用字符串，但是却成了最熟悉的陌生人。我们都知道前半问是肯定的，在写好hello world之后加中文肯定木有问题。 赶紧跑个hello world压压惊：

```java
#include <stdio.h>
#include <string.h>

int main(void) { 
    char str1[] = "👦👩";
    printf("[1]%s\n", str1);
    printf("[1]%d\n", sizeof(str1));
    printf("[1]%d\n", strlen(str1));

    char str2[] = "hello";
    printf("[2]%s\n", str2);
    printf("[2]%d\n", sizeof(str2));
    printf("[2]%d\n", strlen(str2));

    char str3[] = "𡃁妹";
    printf("[3]%s\n", str3);
    printf("[3]%d\n", sizeof(str3));
    printf("[3]%d\n", strlen(str3));

    char str4[] = "中国";
    printf("[4]%s\n", str4);
    printf("[4]%d\n", sizeof(str4));
    printf("[4]%d\n", strlen(str4));

    return 0;
}
```


以上程序将输出： \[1\]👦👩 \[1\]9 \[1\]8 \[2\]hello \[2\]6 \[2\]5 \[3\]𡃁妹 \[3\]8 \[3\]7 \[4\]中国 \[4\]7 \[4\]6

```java
public static void main(String[] args) {
//        String s = "helloworld中国";
//        System.out.println(s.charAt(10));

        // 中文常见字
        String s = "你好";
        System.out.println("1. string length =" + s.length());
        System.out.println("1. string bytes length =" + s.getBytes().length);
        System.out.println("1. string char length =" + s.toCharArray().length);
        System.out.println(s.charAt(0));
        System.out.println(s.codePointAt(0));
        System.out.println(s.codePointAt(1));
        System.out.println(s.codePointCount(0, s.length()));
        System.out.println();
        // emojis
        s = "👦👩";
        System.out.println("2. string length =" + s.length());
        System.out.println("2. string bytes length =" + s.getBytes().length);
        System.out.println("2. string char length =" + s.toCharArray().length);
        System.out.println(s.charAt(0));
        System.out.println(s.codePointAt(0));
        System.out.println(s.codePointAt(1));
        System.out.println(s.codePointAt(2));
        System.out.println(s.codePointAt(3));
        System.out.println(s.codePointCount(0, s.length()));
        System.out.println();
        // 中文生僻字
        s = "𡃁妹";
        System.out.println("3. string length =" + s.length());
        System.out.println("3. string bytes length =" + s.getBytes().length);
        System.out.println("3. string char length =" + s.toCharArray().length);
        System.out.println(s.charAt(0));
        System.out.println(s.codePointAt(0));
        System.out.println(s.codePointAt(1));
        System.out.println(s.codePointCount(0, s.length()));
        System.out.println();
    }
```

以上代码将输出： 

   1\. string length =2 1. string bytes length =6 1. string char length =2 你 20320 22909 2

2.  string length =4
3.  string bytes length =8
4.  string char length =4 ? 128102 56422 128105 56425 2
    
5.  string length =3
    
6.  string bytes length =7
7.  string char length =3 ? 135361 56513 2

聪明的你们肯定一眼看穿了：C语言把字符串按照uft-8或者gbk（取决于运行的平台）编码后存在内存中，众所周知utf-8是可变长的，因此我们看到有的字符用3个字节存，有的字符用4个字节存，有的字符用1个字节存。那么问题来了给你一个字符串指针，你怎么判断里面有没有中文字符呢？一般的做法是先把utf-8字节反编码到unicode，然后判断再去查unicode码表就行了（参考：https://blog.csdn.net/tge7618291/article/details/7599902）。 java也是一样的道理，众所周知Java的字符在内部以UTF-16编码，自然也是可变长的，从上面的代码可以看出有的字符用了java的1个char（注意是16字节），有的字符用了java的2个char。 当我们每天打磨自己的技术使其日益精湛时，知识的细节和知识的完备性非常重要，具体的我们可能需要去看更多前面没看过的源码，去动手实验各种道听途说不确定存疑的技术点。以上这个问题成了我的尴尬的盲区，算是一个警醒吧。