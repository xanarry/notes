# java.lang.Integer类

`Integer`在表示32位整数，其范围为`[Integer.MIN_VALUE, Integer.MAX_VALUE]`，下图的源码表示了常量的具体值。

```java
	/**
	 * A constant holding the minimum value an {@code int} can
	 * have, -2<sup>31</sup>.
     */
    @Native public static final int   MIN_VALUE = 0x80000000;

    /**
     * A constant holding the maximum value an {@code int} can
     * have, 2<sup>31</sup>-1.
     */
    @Native public static final int   MAX_VALUE = 0x7fffffff;
```





## Integer的缓存机制

要理解`Integer`的缓存机制，我们首先需要了解一下`Java`的自动装箱（autoboxing）机制。Java 编译器把原始类型自动转换为封装类的过程称为自动装箱（autoboxing），这相当于调用 valueOf 方法

```java
Integer a = 10;
//自动装箱会把上面的代码解释为如下：
Integer a = Integer.valueOf(10);
```

`valueOf()`执行时会先判断用户定义的变量值是否在缓存中，如果在缓存中，直接返回缓存中的应用，否则创建一个新的整数对象。

`valueOf()`的代码如下：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```



如下是`Integer`缓存机制的代码，从代码中可以看出，默认情况下，缓存整数的范围在[-128, 127]之间，但是可以通过虚拟机参数`-XX:AutoBoxCacheMax`去调整默认的范围。`IntegerCache`在首次被使用的时候会创建该范围中的所有整数并有序放入`cache`数组中，有序保存便于快速索引目标对象。

```java
/**
 * Cache to support the object identity semantics of autoboxing for values between
 * -128 and 127 (inclusive) as required by JLS.
 *
 * The cache is initialized on first usage.  The size of the cache
 * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
 * During VM initialization, java.lang.Integer.IntegerCache.high property
 * may be set and saved in the private system properties in the
 * sun.misc.VM class.
*/
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];//缓存数组，保存指定范围以为的整数
    
    //static块以保证代码仅仅在首次使用的时候执行一次
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
        
        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
        
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
    
    private IntegerCache() {}
}
```

如果需要该缓存机制在代码中为我们所用，那么需要使用如下的写法，不能使用`int`去定义变量：

```java
Integer a = 100;
//or
Integer a = Integer.valueOf(100)
```



因为整数的缓存机制，所以对于如下这样的代码，我们是可以预知其行为的

```java
public class Main {
    public static void main(String[] argv) {
        Integer a = 100;
        Integer b = 100;
        Integer c = 23;

        System.out.println(a == b); //输出true, 100在缓存值中，直接给a,b同一个引用
        System.out.println(c == b); //b,c持有不同的值，通用直接分别给缓存中的两个不同的引用，所以指针不同

        Integer e = 2000003;
        Integer f = 2000003;
        //输出false, 虽然定义的同一个值，但是2000003并不在缓存中，
        //所以Integer会调用了new Integer(2000003),创建一个新的对象
        System.out.println(e == f);
        //输出true,equals函数做字面上的笔记，虽然地址不同，但是都是同一个数值
        System.out.println(e.equals(f));
    }
}
```







## Integer的public static String toString(int i, int radix)

这个函数将整数转为对应进制的字符串，能在看Java源码的人肯定都能轻易实现一个等价的函数的。那么Java自己是怎么实现的这一功能呢，我们看看源码。

```java
/**
 * All possible chars for representing a number as a String
 */
final static char[] digits = {
    '0' , '1' , '2' , '3' , '4' , '5' ,
    '6' , '7' , '8' , '9' , 'a' , 'b' ,
    'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
    'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
    'o' , 'p' , 'q' , 'r' , 's' , 't' ,
    'u' , 'v' , 'w' , 'x' , 'y' , 'z'
};

public static String toString(int i, int radix) {
    //radix表示进制
    //Character.MIN_RADIX=2 Character.MAX_RADIX=36
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;

    /* Use the faster version */
    if (radix == 10) {
        return toString(i); //对于10进制的转换，java使用位运算
    }

    //32位整数加一个符号，所以需要33个char，极端情况表示-0x7fffffff
    char buf[] = new char[33];
    boolean negative = (i < 0);
    int charPos = 32;

    if (!negative) {
        i = -i; //变成负数再处理,取负数才能正确处理Integer.MAX_VALUE
    }

    /*不断求余取整，知道目标值小于基数可以直接当做转换结果*/
    while (i <= -radix) {
        /*从字符集中选取对应字符反向放入缓存区，例如十进制12,二进制1100
          依次:
            -12 < -2: buff[32]=0 
            -6  < -2: buff[31]=0
            -3  < -2: buff[30]=0
            -1 <> -2: exit loop, pos is 29
        */
        buf[charPos--] = digits[-(i % radix)];
        i = i / radix;
    }
    // buff[29]=1
    buf[charPos] = digits[-i];

    //如果是负数，加上负号
    if (negative) {
        buf[--charPos] = '-';
    }

    return new String(buf, charPos, (33 - charPos));
}
```

从源码中可以看出，Java的实现的思路就是对目标整数不断求余取整，找出每一位数对于的字符填充到`buf`数组中，与大多数人的实现无异，唯一的区别在于Java将数字变成了负数处理，这么做的原因在于:

**如果求余取整过程用正整数处理，当遇到一个`Integer.MIN_VALUE`时，其绝对值超出了最大正整数能表示的范围，因此在计算过程中如果遇到最小负数`Integer.MIN_VALUE`就会出错；反之用负数来处理的话，最大正整数取负值，在最小负数的有效范围中，能避免整数溢出的问题**

当进制是10的时候，因为10进制的使用频率巨高，所以Java单独为其设计的一个函数，使用了位运算去实现，使得其速度更快。





## Integer的public static int parseInt(String s, int radix)

`parseInt`将不同进制的字符串数字转换为Integer类型。

```java
public static int parseInt(String s, int radix) throws NumberFormatException
{
    /*
     * WARNING: This method may be invoked early during VM initialization
     * before IntegerCache is initialized. Care must be taken to not use
     * the valueOf method.
     */

    if (s == null) {
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) { //MIN_RADIX=2
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }

    if (radix > Character.MAX_RADIX) { //MAX_RADIX=36
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    int result = 0; //保存解析过程中的中间结果
    boolean negative = false; //正负标志
    int i = 0, len = s.length(); //变量字符串
    int limit = -Integer.MAX_VALUE; //默认设置正数的上限，用负数表示
    int multmin; //设置缩小10倍的大小限制
    int digit; //表示从每个字符串中取出的字符

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```



`parseInt`函数是很有技巧的实现方法，非常优雅有效的处理了字符数值可能超出整数范围而发生异常情况。`parseInt`的逻辑如下：

1. [9,21]行检查传入的参数字符串与进制数是否在有效范围中

2. [23,28]行初始化相关变量，默认为正整数设置

   ```java
   int result = 0; //保存解析过程中的中间结果，因为处理过程中都是负数，所以对result的操纵只有乘法与减法
   boolean negative = false; //正负标志
   int i = 0, len = s.length(); //变量字符串
   int limit = -Integer.MAX_VALUE; //默认设置正数的上限，用负数表示
   int multmin; //设置缩小10倍的大小限制
   int digit; //表示从每个字符串中取出的字符
   ```

3. [31,42]行对输入是负数的情况重新设置相关标志与限制值

4. [44,59]行解析字符串

   a. 从字符串中取出一个字符数字

   b. 将result乘以进制数，为了方便解释，假定进制是10，也就是得出除了个位数的整数部分

   c. 检查除了个位部分的结果是否超过了最大限制的除了个位的部分，如果除了个位都已经超出了限制，那么最后结果必然超过限制，抛出异常。

   d. 加上个位数，检查加上各位数之后的结果是否超过最大限制，如果超过，抛出异常。

   **这一步的技巧在于`if (result < limit + digit)`，因为`result`与`limit`都是负值，所以使用的是`<`符号，这个不等式实际上为`result - digit < limit`移项的结果，`result - digit` 可能超出整数范围，而移项之后，不等式等价，也解决了溢出了问题**

   e. 回到a步，抽取下一个数字计算。





## 一个快速求解整数位数的方法

```java
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
// Requires positive x
static int stringSize(int x) {
	for (int i=0; ; i++) //一次去比较不同位数对应的最大值，找到其索引作为返回值
		if (x <= sizeTable[i])
			return i+1;
}
```





## java中的位移

java中有三种移位运算符，**位移都在在补码上进行**

| 位移符号 | 功能       | 解释                      |
| -------- | ---------- | ------------------------- |
| `<<`     | 左移运算符 | num << 1，相当于num乘以2  |
| `>>`     | 右移运算符 | num >> 1，相当于num除以2  |
| `>>>`    | 无符号右移 | 忽略符号位，空位都以0补齐 |

注意：**没有无符号左移，因为符号位在左边，左移时直接补0而无需考虑符号问题**

计算机中二进制最高位为0表示正，1表示负，且无论正负，计算机中均以补码保存整数的二进制。因此也就有：

- 正数的补码为其二进制本身

- 负数的补码为其绝对值的二进制码取反加一

  

**7的二进制补码为`0000 0111**`

> 如果对7左移1位，得二进制`0000 1110`，为14，相当于乘2
>
> 如果对7右移1位，得二进制`0000 0011`，为3，相当于除2
>
> 如果对7无符号右移1位，得二进制`0000 0011`，为3，相当于除2



**-7的二进制补码为`1111 1001`， 由7的二进制码`0000 0111`    取反`1111 1000`    加一`1111 1001`得到。**

> 如果对-7左移1位，得二进制`1111 0010`，取反加一得`0000 1110`为14，符号为负，所以得-14，相当于乘2
>
> 如果对-7右移1位，得二进制`1111 1100`，取反加一得`0000 0100`为3， 符号为负，所以得-3，相当于除2
>
> 如果对-7无符号右移1位，得二进制`0111 1100`，最高位变成了0，被当成整数对待，值为124





# Byte，Short，Integer与Long

| 类型    | 带宽 | 数值范围                                    |
| ------- | ---- | ------------------------------------------- |
| Byte    | 8位  | [-128, 127]                                 |
| Short   | 16位 | [-32768, 32767]                             |
| Integer | 32位 | [-2147483648, 2147483647]                   |
| Long    | 64位 | [-9223372036854775808, 9223372036854775807] |
- Byte，Short，Integer与Long四种类型都用缓存，默认范围均为[-128, 127]，也就是说实际上`Byte`是全缓存。
- 在字符串数值解析函数中，因为`Short`与`Byte`的范围在`Integer`范围以内，所以直接调用了`Integer`的`parseInt`函数，然后批判范围是否越界。
- Long类的函数实现与Integer的基本一样，仅仅是在范围上的区别。












# java.lang.String类

`String`的内部维护着一个名为`value`的字符数组，String不允许直接修改字符串本身。

1. replace函数通过正则表达式实现，所以其效率是有折扣的

2. split函数在以简单单字符作为delimiter时，通过一段一段的截取做分割，当delimiter是其他的情况则使用正则表达式实现

```java
//判断条件，满足则使用截取方式，否则使用正则表达式
if (((regex.value.length == 1 && ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) 
	|| (regex.length() == 2 && regex.charAt(0) == '\\' && (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 && ((ch-'a')|('z'-ch)) < 0 && ((ch-'A')|('Z'-ch)) < 0)) && (ch < Character.MIN_HIGH_SURROGATE 
	|| ch > Character.MAX_LOW_SURROGATE))
```

3. Arrays.copyOf()直接调用了System.arraycopy(), 该方法是native实现

    ```java
    //Arrays.copyOf的具体实现
    public static short[] copyOf(short[] original, int newLength) {
        short[] copy = new short[newLength];
        				   
        System.arraycopy(original, //源数组
                         0,        //源数组复制起点
                         copy,     //目标数组
                         0,        //目标数组写入起点
                         Math.min(original.length, newLength) //复制长度
                        );
        return copy;
    }
    ```

    

## `String`的加法

String str = "to" + "gether";的等价模式

   ```
   new StringBuilder().append("to").append("gether").toString()
   ```

我们应该首选StringBuilder提升速度

   ```java
   public static void main(String[] args) {
       String cip = "cip";
       String ciop = "ciop";
       String plus = cip + ciop;
       String build = new StringBuilder(cip).append(ciop).toString();
   }
   ```

为什么应该考虑StringBuilder，考虑这样一种情况：我们需要不断对一个字符串做追加操作，以下两种实现方式。

a)加法方式：

   ```java
   for (int i = 0; i < loopCount; i++) {
   	result += str;
   }
   ```

实际上上面的代码等价于

   ```java
   for (int i = 0; i < loopCount; i++) {
   	result = new StringBuilder(result).append(str).toString();
   }
   ```

第二行在每次循环过程中都会重复构造StringBuilder对象，构造过程中会复制result中的字符到builder内部的字符数组，当result越来越长之后，这个过程会越来越慢。

b)StringBuilder方式

   ```java
   StringBuilder stringBuilder = new StringBuilder();
   for (int i = 0; i < loopCount; i++) {
   	stringBuilder.append(str);
   }
   ```

这种形式只构造一次Builder，然后每次追加到builder内部，最后得到目标字符串，所以效率更高。



## `concat`函数

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```



## `String.join()`

`String.join()`函数调用`StringJoiner`类实现字符串的连接。`StringJoiner`类中有如下三个属性，

```
    private final String prefix;   //用在拼接字符串前面，默认为空
    private final String delimiter;//连接符号
    private final String suffix;   //用在拼接字符串后面，默认为空
```

`StringJoiner`内部为一个`StringBuilder`，`String.join()`循环调用`StringJoiner`中的`add`函数：

首次调用的时候，`StringJoiner`添加`prefix`到`StringBuilder`,然后添加列表中的字符串。

第二次以及之后的调用，`StringJoiner`添加`delimiter`到`StringBuilder`，然后添加列表中后续的单词。

最后`String.join()`在return时调用`StringJoiner`的`toString`方法，改方法在之前的结果后面追加上后缀`suffix`然后返回结果

其实就是构造这样一个结果：

```
prefix+string1+delimier+string2+delimiter+string3+...+delimiter+stringN+suffix
```



# java.lang.StringBuilder类





# java.lang.StringBuffer类

