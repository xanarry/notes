java访问控制符的意义与控制范围

|               | 类内部 | 子类 | 本包 | 外部包 |
| :-----------: | :----: | :--: | :--: | :----: |
|  **public**   |   Y    |  Y   |  Y   |   Y    |
|  **default**  |   Y    |  Y   |  Y   |        |
| **protected** |   Y    |  Y   |      |        |
|  **private**  |   Y    |      |      |        |



# Java JNI的使用

一个简单例子`hello world`

先写java的代码

```java
public class JavaJNI{
    static{
        System.loadLibrary("JavaJNI"); //加载dll
    }
	
    //函数声明，native的函数都没有函数体，只有函数签名
    public static native void sayHello(); 

    public static void main(String[]args) {
        new JavaJNI().sayHello();//调用native函数
    }

}
```

