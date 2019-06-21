[TOC]



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



# java.lang.AbstractStringBuilder类

StringBuilder内部结构与String的一致，同为维护了一个名为value的字符数组，与String的不同的地方在于：

1. StringBuilder只有在调用toString函数时候才会将内部的value数组构造为String对象返回
2. StringBuilder提供了大量可以对value数组修改的方法，比如插入、替换、反转、追加等操作，以避免在String对象上操作构造出目标字符串从而提升效率
3. StringBuilder内部的相当一部分函数使用了函数System.arraycopy(src, srcBegin, dst, dstBegin, len)，该函数为C++native实现，具有较高的执行效率



## 容量扩充

StringBuilder中value数组的默认长度为16，扩充一次默认为当前长度的2倍加2。

初始化函数：

```java
public StringBuilder() {
    super(16);
}
```

容量扩充函数：

```java
void expandCapacity(int minimumCapacity) {
    int newCapacity = value.length * 2 + 2;
    if (newCapacity - minimumCapacity < 0)
        newCapacity = minimumCapacity;
    if (newCapacity < 0) {
        if (minimumCapacity < 0) // overflow
            throw new OutOfMemoryError();
        newCapacity = Integer.MAX_VALUE;
    }
    value = Arrays.copyOf(value, newCapacity);
}
```



## 插入操作

StringBuilder在value数组中插入元素的原理：

![](imgs/StringBuilder_insert.png)

```java
public AbstractStringBuilder insert(int index, char[] str, int offset, int len) {
        if ((index < 0) || (index > length()))
            throw new StringIndexOutOfBoundsException(index);
        if ((offset < 0) || (len < 0) || (offset > str.length - len))
            throw new StringIndexOutOfBoundsException(
                "offset " + offset + ", len " + len + ", str.length "
                + str.length);
        ensureCapacityInternal(count + len);
    
    	//这两行操作如上图
        System.arraycopy(value, index, value, index + len, count - index);
        System.arraycopy(str, offset, value, index, len);
    
        count += len;
        return this;
}
```



## replace操作

StringBuilder对value做replace操作的原理：

![](imgs/stringbuilder_replace.png)

```java
public AbstractStringBuilder replace(int start, int end, String str) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (start > count)
            throw new StringIndexOutOfBoundsException("start > length()");
        if (start > end)
            throw new StringIndexOutOfBoundsException("start > end");

        if (end > count)
            end = count;
        int len = str.length();
        int newCount = count + len - (end - start);
        ensureCapacityInternal(newCount);
    
		//这两行操作如上图
        System.arraycopy(value, end, value, start + len, count - end);
        str.getChars(value, start);
    
        count = newCount;
        return this;
}
```



# java.lang.StringBuilder类

StringBuilder继承于AbstractStringBuilder，该类中的所有方法的实现都是直接调用其父类方法。StringBuilder中的方法在被调用时，会调用父类的方法去完成任务。

## 非线程安全

StringBuilder并**不保证其类中的方法线程安全**，所以相比StringBuffer，在单线程的环境下推荐使用StringBuilder以获取最佳的性能。



# java.lang.StringBuffer类

StringBuffer与StringBuilder一样，同是继承于AbstractStringBuilder，该类中的所有方法的实现都是直接调用其父类方法，且每个方法都加上了**synchronized**关键词以保证线程安全。

```java
@Override
public synchronized String substring(int start, int end) {
    return super.substring(start, end);
}
```

## toStringCache

```private transient char[] toStringCache;```

toStringCache缓存最后一次toString的结果，在toString之后的操纵并不修改其内容，每执行一次toString更新一次其值。

### transient关键词

> Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。
>
> transient是Java语言的关键字，用来表示一个域不是该对象串行化的一部分。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，然而非transient型的变量是被包括进去的。



## 线程安全


StringBuffer的所有方法都有**synchronized**关键词，因此该类保证其类中的方法线程安全。



## StringBuilder，StringBuffer，AbstractStringBuilder的关系

![](imgs/StringBuilder.png)

# Java集合框架

## List

### List类图

![](imgs/list_oop.png)

### ArrayList
正如其名字，ArrayList内部维护一个Object类型的数组，与一个size变量。



#### 插入
1. add(E element)

   检查容量，在必要的时候扩充容量，然后将新的元素插入到内部数组末尾。

2. add(int index, E element)

   检查容量，在必要的时候扩充容量，**将从index开始到尾部的所有元素通过System.arraycopy()后移一个位置**，然后将新的元素插入到内部数组末尾。

3. addAll(Collection<? extends E> c)

   首先得c中的内部数组引用到变量a，将ArrayList的容量进行检查并在必要的时候扩充容量，使用**System.arraycopy**将数组a的追加到ArrayList中的数组。代码如下：

   ```java
   System.arraycopy(a, 0, elementData, size, numNew);
   //(插入数组，插入数组的起始访问坐标，ArrayList的内部数组，插入为尾部参数size,插入数量a的长度)
   ```

4. addAll(int index, Collection<? extends E> c)

   检查index是否在ArrayList的最大下标范围之中；取得c中的内部数组引用到变量a；检查ArrayList的容量必要的情况下对其进行扩充；**将从index开始到尾部的所有元素通过System.arraycopy()后移c.size()个位置**； 使用**System.arraycopy**将c复制到ArrayList从index处开始的位置。 代码大致如下：

   ```java
   //后移元素
   System.arraycopy(elementData, index, elementData, index + numNew,numMoved);
   //插入元素
   System.arraycopy(a, 0, elementData, index, numNew);
   ```

   

**插入元素时的容量检查与扩充**
> The array buffer into which the elements of the ArrayList are stored.
> The capacity of the ArrayList is the length of this array buffer. Any
> empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
> will be expanded to DEFAULT_CAPACITY when the first element is added.

不带容量参数定义ArrayList时，elementData指向DEFAULTCAPACITY_EMPTY_ELEMENTDATA，当第一个元素插入时，将其容量扩充为DEFAULT_CAPACITY，10个容量。



对程序员开放的调整容量的函数`ensureCapacity`，扩充的最小容量小于ArrayList默认最小容量10的时候，扩充操纵将被忽略。

> Arraylist在扩充容量的时候，新的容量首先被扩充为就容量的1.5倍，如果1.5倍值不足minCapacity，那么新的容量被调整为minCapacity，容量的的最大值可以达到Integer.MAX_VALUE

```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY; //10
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```



私有函数`ensureExplicitCapacity`的定义

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {//空数组
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
	}
	ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
	modCount++;
	// overflow-conscious code
    if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}
```



扩充的具体操作

```java
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //默认扩大为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩大1.5倍之后还是达不到目标值minCapacity，那么直接使用minCapacity的值，
        //如果newCapacity溢出之后为负数，那么了newCapacity将被minCapacity替换，
        //minCapacity会保证是一个正数
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        
        //如果新的容量值超出了MAX_ARRAY_SIZE，那么对minCapacity再一次检查，如果超过了MAX_ARRAY_SIZE
        //且没有溢出，直接将容量调整为正数最大值，如果没有超过MAX_ARRAY_SIZE，那么使用这个值作为容量
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        
        // minCapacity is usually close to size, so this is a win:
        //开辟新的容量为newCapacity的数组，将旧数组的元素复制进入，然后将其返回给elementData
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



#### 删除

1. remove(int index)

   ```java
   System.arraycopy(elementData, index+1, elementData, index, numMoved);
   elementData[--size] = null; // clear to let GC do its work
   ```

   将index+1处开始到末尾的元素向前移动一位；将size值减一并置最后一个元素为null方便垃圾回收。

2. remove(Object o)

   删除机制与remove(int index)一致，先查找目标对象的下标，然后执行删除操作。

3. removeAll(Collection\<?> c) 与retainAll(Collection<?> c)

   removeAll与retailAll是一对相反的操作，所以其内部实现实际上是调用的同一个函数，核心代码如下：

   ```java
   int r = 0, w = 0;
   for (; r < size; r++)
       //不同函数收集不同的目标元素，remove->complement为false, retail->complement为true
   	if (c.contains(elementData[r]) == complement)
   		elementData[w++] = elementData[r];//收集目标元素防止在数组靠前位置
   
   if (w != size) {
   	// clear to let GC do its work
       for (int i = w; i < size; i++)
   		elementData[i] = null;//将末尾置为null
   	size = w;//更新数组容量
   }
   ```

   

4. removeIf(Predicate<? super E> filter)

   removeIf先使用BitMap记录符合filter规则的元素，也就是需要被删除的元素；然后执行的操纵与removeAll类似。



#### 迭代器

迭代器是ArrayList中的一个内部类。对内部数组的访问与删除做了封装。

我们看调用iterater函数时发生了什么：

```java
public Iterator<E> iterator() {
	return listIterator();//返回ArrayList内部类的一个实例
}
```



内部类`Iterator`的实现代码

```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        //cursor表当前元素的下一个元素的下标，如果小于size说明还有元素可以访问
        return cursor != size; 
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor; //即将被访问的元素
        if (i >= size) //检查是否下标越界
            throw new NoSuchElementException();
        //取得ArrayList内部数组的引用
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)//检查是否下标越界
            throw new ConcurrentModificationException();
        cursor = i + 1;//光标后移一位
        return (E) elementData[lastRet = i]; //设置当前访问元素的下标并返回目标元素
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            //下面代码的等价形式
            //System.arraycopy(elementData, index + 1, elementData, index, size - index - 1);
            ArrayList.this.remove(lastRet);
            //因为当前元素被删除，后面的所有元素前移一位，
            //所以cursor置为lastRet就标记了下一个即将访问的元素的下标
            cursor = lastRet;
            //因为lastRet处的元素已经被删除，所以将该值设为-1
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```



Clear

```java
public void clear() {
    modCount++;
    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;
    size = 0;
}
```



sort调用Arrays.sort()，其内部为快速排序实现。

contains()线性查找，时间复杂度O(n)。

clone()函数实现

```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```



### Vector

Vector是一个可变长数组，可以通过数值下标访问元素，vector对象创建之后在插入或者删除的过程中可以收缩也可以扩张。vector会维持一个capacity变量capacityIncrement变量，capacity表当前可用容量，capacityIncrement表当容量不足以存储数据时一次扩张的量。

vector的默认容量是10，capacityIncrement的默认值是0。在创建vector对象时，可以同时指定这个两个参数，也可以仅指定capacity，或者不用任何参数使用默认配置。

Vector与ArrayList都继承了基类AbstractList，都实现了List接口，所以其内部实现逻辑上与ArrayList一致，但是与ArrayList有以下两个区别：



1.容量扩张量不同：

ArrayList在容量扩张的时候需要指定一个最下容量，扩充之后的容量一定大于等于这个minCapacity，ArrayList的扩张量一次为原始容量的1.5倍。

Vector没有minCapacity的约束，空间塞满了就会扩张，一次扩张为原始容量的2倍。



看看Vector的扩张源码：

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    
    //一次把容量设置为旧容量的2倍
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    
    //这里为什么不用if (newCapacity < minCapacity)是因为：首先在进入该本函数前确保了
    //minCapacity > elementData.length, newCapacity可能会溢出, newCapacity溢出之后
    //相当于是3倍的elementData.length，由于2倍的时候有已经溢出为负值，三倍情况会变为正值
    //所以不会进入该判断语句  
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //newCapacity可能已经溢出，那么和MAX_ARRAY_SIZE对比一次，如果满足则设置最大容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

2.ArrayList非线程安全，Vector线程安全

Vector的函数实现使用了synchronized关键词，确保的线程安全，相应损失是性能下降。



### LinkedList

LinkedList同时实现了List接口与queue接口，非线程安全。内部结构由双向链表实现，同时为每个结点配备了相对应的数值索引，需要注意的是该索引并没有保存到结点中，而是在**每次使用的时候通过遍历链表去查询相应索引所在的元素或者相应元素对应的索引**。内部为如图这样的结构：

![](imgs/double-linkedlist.png)

LinkedList中Node结点的定义，Node是LinkedList的一个内部类。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 插入

该类内部实现了头部插入、中间插入、尾部插入三种插入方式以满足各式需求。Java的实现方式与我们自己的实现方式并无二异，操作正确对next和prev指针的指向即可，插入后将size加1。

#### 删除

删除与插入一样，没有需要特别注意的地方。删除后将size减1。

#### 索引相关函数

1. 得到指定位置的元素

   值得学习的地方：**在链表的前一半从前向后查找；在链表的后一半从后向前查找**

```java
/**
 * Returns the (non-null) Node at the specified element index.
 */
Node<E> node(int index) {
    // assert isElementIndex(index);
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```



2. 得到目标元素所在的位置

   遍历链表，在变量的过程计数，遇到目标结点则停止。

```java
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```



#### 迭代器

迭代器为LinkedList的一个内部类，以便访问其中的元素，获取迭代器则取得该内部类的一个实例。

获取迭代器：

```java
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
```



迭代器的具体实现：

```java
private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;//即将被访问的元素
    private int nextIndex;//即将被访问元素的下标
    private int expectedModCount = modCount;

    //构造函数，设置起始迭代位置,如果index等于size,那么已经没有元素可以迭代
    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }
	
    //迭代位置的索引没有到尾部则可以继续向后迭代
    public boolean hasNext() {
        return nextIndex < size;
    }
	
    public E next() {
        checkForComodification();
        //如果已经到尾部，那么抛出异常
        if (!hasNext())
            throw new NoSuchElementException();
		//标记本次需要返回元素所在的结点
        lastReturned = next;
        next = next.next;//向后移动一次指针指向下次应该返回的元素
        nextIndex++;//索引加一，标记下一个元素的索引位置
        return lastReturned.item; //返回目标元素
    }

    public boolean hasPrevious() {
        return nextIndex > 0; //如果当前位置不是在第一个位置，那么有前驱结点
    }

    public E previous() {
        checkForComodification();
        if (!hasPrevious())//没有前驱结点可访问则抛出异常
            throw new NoSuchElementException();
		//如果已经到尾部之后，那么前一个元素一定是尾部元素，
        //否则返回即将访问元素的前一个元素，并重置lastReturned与next
        lastReturned = next = (next == null) ? last : next.prev;
        nextIndex--;//索引回退1
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;
    }

    public int previousIndex() {
        return nextIndex - 1;
    }

    //删除next()函数刚刚返回的结点，也就是正是被lastReturned变量引用的结点
    public void remove() {
        checkForComodification();
        if (lastReturned == null)//如果是null,抛出异常
            throw new IllegalStateException();
		
        //得到要删除元素的下一个元素，记为lastNext
        Node<E> lastNext = lastReturned.next;
        //删除目标元素
        unlink(lastReturned);
        
        if (next == lastReturned)
            //如果在删除之前执行了previous，就会导致这种情况，重置next，
            //因为nextIndex--已经在previous函数中执行，所以不用更新
            next = lastNext;
        else//next就在正确的位置，仅需要更新一下索引
            nextIndex--;
        lastReturned = null;//本用来被next函数访问的结点已经被删除，置为null
        expectedModCount++;
    }

    public void set(E e) {
        if (lastReturned == null)
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;//直接设置结点新值
    }

    public void add(E e) {
        checkForComodification();
        //next结点之前插入一个元素。
        lastReturned = null;
        if (next == null)
            linkLast(e);
        else
            linkBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```



#### LinkedList上的队列操作

A) 在链表头部的操作	

|             函数名             |                          功能                          |
| :----------------------------: | :----------------------------------------------------: |
|        public E peek()         |            返回链表头部的元素，可能返回null            |
|       public E element()       |         返回链表头部的元素，遇到null则抛出异常         |
|        public E poll()         |    返回链表头部的元素，可能返回null，**并删除头部**    |
|       public E remove()        | 返回链表头部的元素，遇到null则抛出异常，**并删除头部** |
| public boolean offerFirst(E e) |                   插入元素到链表头部                   |
|      public E peekFirst()      |            返回链表头部的元素，可能返回null            |
|      public E pollFirst()      |    返回链表头部的元素，可能返回null，**并删除头部**    |
|     public void push(E e)      |                   插入元素到链表头部                   |

B) 在链表尾部的操作

|            函数名             |                       功能                       |
| :---------------------------: | :----------------------------------------------: |
|   public boolean offer(E e)   |                在链表尾部插入元素                |
| public boolean offerLast(E e) |                在链表尾部插入元素                |
|      public E peekLast()      |         返回链表尾部的元素，可能返回null         |
|      public E pollLast()      | 返回链表头部的元素，可能返回null，**并删除头部** |
|        public E pop()         |                 删除链表头部元素                 |





## Set

### HashSet

#### 插入



#### 删除



#### 修改



#### 查看



#### 迭代器



### TreeSet



#### 插入



#### 删除



#### 修改



#### 查看



#### 迭代器



## Map

### HashMap

#### 插入



#### 删除



#### 修改



#### 查看



#### 迭代器



### TreeMap



#### 插入



#### 删除



#### 修改



#### 查看



#### 迭代器



## Queue



## Stack



## 迭代器



