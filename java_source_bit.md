Title: Bits
Category: JAVA
Tags: JDK Source, Algorithm

## 二进制与位操作
位操作有&，|，~，^，<<，>>，>>>。位操作应用于二进制，把二进制的原理结合到实际的场景中，再通过美妙的位操作有时候可以产生很多巧妙优雅的方法。同时，数的二进制表示是运用位操作的基础，比如[补码][1]。然后，总结下几个遇到过的吧。
###JDK Integer Implements
java的Integer类有很多位操作的用法
```java
 public static int lowestOneBit(int i) {
        return i & -i;
    }
(n & -n) == n //2的幂次
```
这是灵活的运用了补码取反加1的原理。
```java
  public static int highestOneBit(int i) {
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
```
上面的过程是这样的，假设按8bit表示一个数i
```java
i             ==> 12345678
i | i >> 1 ==> 12345678 | 01234567
i | i >> 1 | i >> 2| i >> 3 ==> 12345678 | 01234567 | 00123456| 00012345
i | i >> 1 | i >> 2| i >> 3 | i >> 4 | i >> 5 | i >> 6 | i >> 7 ==>
12345678| 
01234567| 
00123456| 
00012345| 
00001234| 
00000123| 
00000012| 
00000001
```
最后i上的每一位k是原数与其左边所有位"|"的结果，也就是说最高位如果为1，那它后面的位都为1，i - (i >>> 1)，就是把除最高位1之后的其他位减掉。
```java
 for (;;) {
            q = (i * 52429) >>> (16+3);
            //2^19=524288
            //52429/524288=0.1000003814697
            //q=i*0.1
            r = i - ((q << 3) + (q << 1));  
            // r = i-(q*10) ...
        }
```
其中，[除法][2]有专门的算法研究。乘法可以用左移二进制的表示法来提高性能。
```java
public static int numberOfTrailingZeros(int i) {
    // HD, Figure 5-14
    int y;
    if (i == 0) return 32;
    int n = 31;
    y = i <<16; if (y != 0) { n = n -16; i = y; }
    y = i << 8; if (y != 0) { n = n - 8; i = y; }
    y = i << 4; if (y != 0) { n = n - 4; i = y; }
    y = i << 2; if (y != 0) { n = n - 2; i = y; }
    return n - ((i << 1) >>> 31);
}
```
末尾有多少个0？我本来内心有一个很普通的实现，但是一看到这个实现，就觉得好喵啊～二分的思想，我这么没用起来了。
```java
public static int bitCount(int i) {
        // HD, Figure 5-2
        i = i - ((i >>> 1) & 0x55555555);
        i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
        i = (i + (i >>> 4)) & 0x0f0f0f0f;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        return i & 0x3f;
    }
```
可能下面的代码更加直观一点
```java
int bitCount(int n) 
{ 
    n = (n & 0x55555555) + ((n>>>1) & 0x55555555); 
    n = (n & 0x33333333) + ((n>>>2) & 0x33333333); 
    n = (n & 0x0f0f0f0f) + ((n>>>4) & 0x0f0f0f0f); 
    n = (n & 0x00ff00ff) + ((n>>>8) & 0x00ff00ff); 
	n = (n & 0x0000ffff) + ((n>>>16)& 0x0000ffff);        
	return n ; 
}
```
也是通过二分的思想，将相邻1位，2位，4位，8位最后16位的1的个数加起来，最后的值就是答案。
可惜的是，JVM虚拟机在碰到上面的方法时，并不会编译执行这些看似巧妙的方法，而是将这些方法直接用[指令][4]实现——[HotSpot Intrinsics][3]
###二进制状态压缩
之前遇到一个面试题，是说有1000瓶药，其中有1瓶是有毒的，用10只老鼠喝一次药来判断哪一瓶药是有毒的。之前没有接触过状态压缩，老鼠的生和死用二进制来标识就是2^10=1024种状态，把10只老鼠看成一个10bit的数，我想这就是一种状态压缩吧。搜到一个高中生的[博客][5]，我还是好好的写点业务代码吧～洗洗睡。

[1]: http://zh.wikipedia.org/wiki/%E4%BA%8C%E8%A3%9C%E6%95%B8 
[2]:http://en.wikipedia.org/wiki/Division_algorithm#Newton.E2.80.93Raphson_division
[3]:http://www.java-gaming.org/index.php?topic=27010.0
[4]:http://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[5]:http://www.cppblog.com/zyn1996/archive/2011/12/04/161415.aspx
