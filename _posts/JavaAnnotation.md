## 引子

写代码的每个同学估计都对注解（annotation）并不陌生，至少也用过@Override这样的注解。Java中的注解是个很神奇的东西，用了注解就可以少些很多代码，但是有没有想过这些注解呢如何实现的呢？这篇文章就带你走进Java注解的世界。本文的所有代码都在我的GitHub上的annokit里面，欢迎star和fork and pull request。

## Java注解介绍

开始之前我们先来说一些基本的概念：

### 注解的介绍

Java注解是附加在代码中的一些元信息，用于编译和运行时进行解析和使用，起到说明、配置的功能。注解不会影响代码的实际逻辑，仅仅起到辅助性的作用。包含在java.lang.annotation包中。注解的定义类似于接口的定义，使用@interface来定义，定义一个方法即为注解类型定义了一个元素，方法的声明不允许有参数或throw语句，返回值类型被限定为原始数据类型、字符串String、Class、enums、注解类型，或前面这些的数组，方法可以有默认值。注解并不直接影响代码的语义，但是他可以被看做是程序的工具或者类库。它会反过来对正在运行的程序语义有所影响。注解可以从源文件、class文件或者在运行时通过反射机制多种方式被读取。

### Java元注解

元注解是指注解的注解。包括 @Retention @Target @Document @Inherited四种。（java.lang.annotation中提供，为注释类型）。

注解	  | 说明
----------+----------------------  
@Target	    定义注解的作用目标

@Retention	 定义注解的保留策略。RetentionPolicy.SOURCE:注解仅存在于源码中，在class字节码文件中不包含；RetentionPolicy.CLASS:默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得;RetentionPolicy.RUNTIME:注解会在class字节码文件中存在，在运行时可以通过反射获取到。

@Document	说明该注解将被包含在javadoc中

@Inherited	说明子类可以继承父类中的该注解

### Target类型说明

Target类型					| 说明  
----------------------------+----------------------  
ElementType.TYPE			| 接口、类、枚举、注解  
ElementType.FIELD			| 字段、枚举的常量  
ElementType.METHOD			| 方法  
ElementType.PARAMETER		| 方法参数  
ElementType.CONSTRUCTOR		| 构造函数  
ElementType.LOCAL_VARIABLE	| 局部变量  
ElementType.ANNOTATION_TYPE	| 注解  
ElementType.PACKAGE			| 包  

## Java注解实现

运行时处理的注解（反射机制）

1. 定义注解

我们定义了一个使用反射实现的注解Reflect,注解有一个参数name,含有默认值。另外，保留策略要使用RUNTIME,将注解保留到运行时，这样才能在运行时使用反射来获取到注解。
```java
package com.github.hackersun.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Desc:
 * Author:sunguoli@meituan.com
 * Date:15/12/20
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Reflect {

    String name() default "sunguoli";

}
```
2. 实现注解处理器

注解处理器的代码：
```java
package com.github.hackersun.processor.reflect;

import com.github.hackersun.annotation.Reflect;
import java.lang.reflect.Method;

/**
 * Desc:
 * Author:sunguoli@meituan.com
 * Date:15/12/20
 */
public class ReflectProcessor {

    public void parseMethod(final Class<?> clazz) throws Exception {
        final Object obj = clazz.getConstructor(new Class[] {}).newInstance(new Object[] {});
        final Method[] methods = clazz.getDeclaredMethods();
        for (final Method method : methods) {
            final Reflect my = method.getAnnotation(Reflect.class);
            if (null != my) {
                method.invoke(obj, my.name());
            }
        }
    }
}
```
3. 测试注解

测试代码：
```java
package com.github.hackersun.sample;

import com.github.hackersun.annotation.Reflect;
import com.github.hackersun.processor.reflect.ReflectProcessor;

/**
 * Desc:
 * Author:sunguoli@meituan.com
 * Date:15/12/20
 */
public class ReflectTest {

    @Reflect
    public static void sayHello(final String name) {
        System.out.println("==>> Hi, " + name + " [sayHello]");
    }

    @Reflect(name = "AngelaBaby")
    public static void sayHelloToSomeone(final String name) {
        System.out.println("==>> Hi, " + name + " [sayHelloToSomeone]");
    }

    public static void main(final String[] args) throws Exception {
        final ReflectProcessor relectProcessor = new ReflectProcessor();
        relectProcessor.parseMethod(ReflectTest.class);
    }
}
```
Output:

```
==>> Hi, sunguoli [sayHello]  
==>> Hi, AngelaBaby [sayHelloToSomeone]
```

[自己动手实现Java注解（Java Annotation in Action）](http://blog.csdn.net/doc_sgl/article/details/50367083)