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

搞清楚字符，字符串，编码的知识~



# 探究字符串的长度

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

运行结果

```
[1]👦👩
[1]9
[1]8
[2]hello
[2]6
[2]5
[3]𡃁妹
[3]8
[3]7
[4]中国
[4]7
[4]6
```



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

运行结果：

```
1. string length =2
1. string bytes length =6
1. string char length =2
你
20320
22909
2

2. string length =4
2. string bytes length =8
2. string char length =4
?
128102
56422
128105
56425
2

3. string length =3
3. string bytes length =7
3. string char length =3
?
135361
56513
2

```

总结上面：

- 一个java的char放不下一个表情或者某些中文生僻字，需要两个char才行。
- 一个java的char是放得下大部分的中文的。
- C语言的char只有1个字节，java的有2个字节，存放中文的原理都是类似的。
- java的codePointCount可以获得正确的字符的数量（不是char的数量）



# 字符串的编码

```java
/**
 * 测试String bytes的转换
 */
public class StringBytes {

    /**
     * 按照8位二进制打印数值
     * @param value
     * @return
     */
    public static String format(int value) {
        String bin = Integer.toBinaryString(value);
        StringBuilder sbBin = new StringBuilder();
        if (value >= 0) {
            int temp = 8 - bin.length();
            for (int i = 0; i < temp; i++) {
                sbBin.append("0");
            }
            sbBin.append(bin);
        } else {
            sbBin.append(bin.substring(24));
        }
        return sbBin.toString();
    }

    /**
     * 打印字符串的编码
     * @param s
     * @throws UnsupportedEncodingException
     */
    public static void printBytesUtf8(String s) throws UnsupportedEncodingException {
        byte[] bytes = s.getBytes("utf-8");
        for (int i = 0; i < bytes.length; i++) {
            System.out.print(format(bytes[i]) + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) throws UnsupportedEncodingException {
        printBytesUtf8("123456");
        printBytesUtf8("社会主义核心价值观");
        printBytesUtf8("社");
        printBytesUtf8("沆瀣一气");
        printBytesUtf8("𡃁");
    }
}
```

输出结果：

```
00110001 00110010 00110011 00110100 00110101 00110110 

11100111 10100100 10111110 11100100 10111100 10011010 11100100 10111000 10111011 11100100 10111001 10001001 11100110 10100000 10111000 11100101 10111111 10000011 11100100 10111011 10110111 11100101 10000000 10111100 11101000 10100111 10000010 

11100111 10100100 10111110 

11100110 10110010 10000110 11100111 10000000 10100011 11100100 10111000 10000000
11100110 10110000 10010100 

11110000 10100001 10000011 10000001 
```

总结上面：

字符到字节的编码，一个字符编码成1,2,3,4个字节都是有可能的。

