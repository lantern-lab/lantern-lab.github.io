
# Java反射

参考文章：

[JAVA反序列化 - 反射机制](https://xz.aliyun.com/t/7029?u_atoken=b4026ca7ac4886a04af3de4a4571024f&amp;u_asession=01aoSN4PMUUPqyTdMMk1Ef5vZNjTnru6QmmPA4RX5NLmMKfZYSnl1Cbx7o0G-Rj69PJB-YY_UqRErInTL5mMzm-GyPlBJUEqctiaTooWaXr7I&amp;u_asig=05oI5YNvJnnW4ZbDwZ0jTEAhHEvIxj9gbBrxIpeiXg5xCtxncx5qFaMXntgOnS7d1l7VGQuoM5HoVu0W2GAzl_O2K7YRS3n97skPU9-oXW66Mqubl6cGDOBwX4yhwDVq4pUs1U4-SwkHFuYFf4nLXW3wlzUqT9HJifiHAxYDNjzobBzhvSc0Kr8URjOX9Xe4tkHSLrYJwk8-bo4x4HDY6cVzvv2QhtnwjqKFq4SjV1HUVqvx1u0HrR_v_qdKanBIunIp98F86zJTfJkIUQnSMrXMQZqSccbYAETgQHDh3s22Z6gx6UxFgdF3ARCQ86jS_u_XR5hatHQVh06VuUZ-D1wA&amp;u_aref=yc88oUkFt0p4squ9WnOlseT1TjY%3D "JAVA反序列化 - 反射机制")

[Java反射（超详细！）](https://blog.csdn.net/qq_44715943/article/details/120587716)

[Java反射（简单详细且易懂，快速入门）](https://blog.csdn.net/qq_64997449/article/details/133838157)

## 0 准备一个User类

```java
package org.example.reflect;

public class User {
    public String name;
    private int age;

    public User(){}

    private User(String name){
        this.name = name;
    }

    public User(String name, int age){
        this.name = name;
        this.age = age;
    }

    private void testPrivate(String name, int age){
        System.out.println("这是一个用来测试的私有方法。User [name=\"+ name +\", age=\"+age+\"]");
    }

    public void testPublic(String name, int age) {
        System.out.println("这是一个用来测试的公有方法。User [name=\"+ name +\", age=\"+age+\"]");
    }

    public String toString(){
        return "User [name="+ name +", age="+age+"]";
    }
}
```

## 1 反射相关类

|类|含义|
| -------------------------------| -------------------------------------------------------------------|
|java.lang.Class|代表整个字节码；代表一个类型，代表整个类|
|java.lang.reflect.Constructor|代表字节码中的构造方法字节码；代表类中的构造方法|
|java.lang.reflect.Field|代表字节码中的属性字节码；代表类中的成员变量（静态变量+实例变量）|
|java.lang.reflect.Method|代表字节码中的方法字节码；代表类中的方法|

通常是先获取Class类，再通过Class类的方法获取Method类、Constructor类或Field类

## 2 获取Class类

* Class.forName("类的静态路径")
* 类名.Class
* 对象名.getClass()

```java
package org.example.reflect;

public class ReflectTest01 {
    public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {

        Class<?> c1 = Class.forName("org.example.reflect.User");
        //如果无参构造函数是private,则无法使用newInstance方法
        //Object o1 = c1.newInstance();
        System.out.println(c1.getName());

        Class<?> c2 = User.class;
        //Object o2 = c2.newInstance();
        System.out.println(c2.getName());

        User user = new User("zhangsan",18);
        Class<? extends User> c3 = user.getClass();
        Object o3 = c3.newInstance();
        System.out.println(o3);
    }
}
```

获取到Class类后，可以通过如下方法获取对应的构造函数、成员方法或者成员变量才进行其它操作

### 通过Class获取Constructor

|方法|含义|
| --------------------------------------------------------------------------| ----------------------------------------------|
|public Constructor<T> getConstructor(Class<?>... parameterTypes)|获取公开的构造方法|
|public Constructor<?>[] getConstructors()|获取所有的公开的构造方法|
|public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)|获取指定包括私有,不包括继承的Constructor对象|
|public Constructor<?>[] getDeclaredConstructors()|获取所有的构造方法,包括私有|

### 通过Class获取Field

|方法|含义|
| --------------------------------------------| ----------------------------------------------------|
|public Field getField(String name)|获取指定公共属性的Field对象|
|public Field[] getFields()|获取所有公开的成员变量,包括继承变量|
|public Field getDeclaredField(String name)|获取指定包括私有,不包括继承的Field对象|
|public Field[] getDeclaredFields()|获取本类定义的成员变量,包括私有,但不包括继承的变量|

### 通过Class获取Method

|方法|含义|
| --------------------------------------------------------------------------| ------------------------------------------------|
|public Method getMethod(String name, Class<?>... parameterTypes)|获取指定方法的Method对象|
|public Method[] getMethods()|获取所有可见的方法,包括继承的方法|
|public Method getDeclaredMethod(String name, Class<?>... parameterTypes)|获取指定包括私有,不包括继承的Method对象|
|public Method[] getDeclaredMethods()|获取本类定义的的方法,包括私有,不包括继承的方法|

## 3 获取Constructor

```java
package org.example.reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class ReflectTest02 {
    public static void main(String[] args) {
        Class<?> userClass = User.class;

        //测试getConstructors()函数
        Constructor<?>[] constructors = userClass.getConstructors();
        System.out.println("\n通过getConstructors()获取所有公开构造方法");
        for (Constructor<?> con : constructors) {
            System.out.println("构造方法名称为：" + con.getName());
            Class<?>[] paramList = con.getParameterTypes();
            System.out.println("参数数量为：" + paramList.length);
            System.out.println("形参类型为：");
            for (Class<?> p : paramList) {
                System.out.println(p);
            }
        }
        System.out.println();//换行

        //测试getDeclaredConstructors()函数
        Constructor<?>[] DeclaredConstructors = userClass.getDeclaredConstructors();
        System.out.println("\n通过getDeclaredConstructors()获取所有构造方法(包括私有,包括继承)");
        for (Constructor<?> con : DeclaredConstructors) {
            System.out.println("构造方法名称为：" + con.getName());
            Class<?>[] paramList = con.getParameterTypes();
            System.out.println("参数数量为：" + paramList.length);
            System.out.println("形参类型为：");
            for (Class<?> p : paramList) {
                System.out.println(p);
            }
        }

        //测试getConstructor()函数
        try {
            Constructor<?> constructor = userClass.getConstructor(String.class, int.class);
            Object o = constructor.newInstance("Alice",18);
            System.out.println("\n构造的对象为：" + o);
        } catch (NoSuchMethodException | InvocationTargetException | InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 4 获取Field

‍

```java
package org.example.reflect;

import java.lang.reflect.Field;

public class ReflectTest03 {
    public static void main(String[] args) throws InstantiationException, IllegalAccessException {
        Class<User> userClass = User.class;
        Object o = userClass.newInstance();

        //测试getFields()函数
        Field []studentFields = userClass.getFields();
        System.out.println("\n通过getFields获取Student类所有公开属性");
        for (Field field:studentFields) {
            System.out.println("属性的类型为："+field.getType()+"属性的名称为："+field.getName());
        }

        //测试getDeclaredFields()函数
        Field []studentDeclaredFields = userClass.getDeclaredFields();
        System.out.println("\n通过getDeclaredFields获取Student类所有属性(包括私有,不包括继承)");
        for (Field field :studentDeclaredFields) {
            System.out.println("属性的类型为："+field.getType()+"\t属性的名称为："+field.getName());
        }

        //测试getField(String)函数
        try {
            Field field = userClass.getField("name");
            System.out.println("\n通过getField(\"name\")获取公开属性name");
            System.out.println("属性的类型为："+field.getType()+"\t属性的名称为："+field.getName());
            field.set(o,"Bob");
            System.out.println(o);
        } catch (NoSuchFieldException e) {
            throw new RuntimeException(e);
        }

        //测试getDeclaredField(String)函数
        try {
            Field declaredFiled = userClass.getDeclaredField("age");
            System.out.println("\n通过getDeclaredField(\"age\")获取私有属性age");
            System.out.println("属性的类型为："+declaredFiled.getType()+"\t属性的名称为："+declaredFiled.getName());
            declaredFiled.setAccessible(true);//为了可以访问私有成员变量，我们需要强制设置访问权限。
            declaredFiled.set(o, 18);
            System.out.println(declaredFiled.get(o));
        } catch (NoSuchFieldException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 5 获取Method

```java
package org.example.reflect;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ReflectTest04 {

    public static void main(String[] args) throws NoSuchMethodException, InstantiationException, IllegalAccessException {
        Class<User> userClass = User.class;
        Object o = userClass.newInstance();
      
        //测试getMethods()函数
        Method[] methods = userClass.getMethods();
        System.out.println("\n通过getMethods()获取所有公开方法");
        for (Method method : methods) {
            System.out.println(method.getName());
            Class<?>[] paramList = method.getParameterTypes();
            System.out.println("参数数量为：" + paramList.length);
            for (Class<?> p : paramList) {
                System.out.println("参数类型为:" + p.getTypeName());
            }
        }

        //测试getDeclaredMethods()函数
        Method[] declaredMethods = userClass.getDeclaredMethods();
        System.out.println("\n通过getdeclaredMethods()获取所有方法(包括私有,不包括继承)");
        for (Method method : declaredMethods) {
            System.out.println(method.getName());
            Class<?>[] paramList = method.getParameterTypes();
            System.out.println("参数数量为：" + paramList.length);
            for (Class<?> p : paramList) {
                System.out.println("参数类型为:" + p.getTypeName());
            }
        }

        //测试getMethod(参数)函数
        Method method = userClass.getMethod("testPublic", String.class, int.class);
        System.out.println("\n通过getMethod获取testPublic");
        System.out.println(method.getName());
        try {
            method.invoke(o, "Alice", 18);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        }
        Class<?>[] paramList = method.getParameterTypes();
        System.out.println("参数数量为：" + paramList.length);
        for (Class<?> p : paramList) {
            System.out.println("参数类型为:" + p.getTypeName());
        }

        //测试getDeclaredMethod(参数)函数
        Method declaredMethod = userClass.getDeclaredMethod("testPrivate", String.class, int.class);
        System.out.println("\n通过getDeclaredMethod获取testPrivate方法");
        System.out.println(declaredMethod.getName());
        try {
            declaredMethod.setAccessible(true);
            declaredMethod.invoke(o, "Bob", 20);
        } catch (InvocationTargetException e) {
            throw new RuntimeException(e);
        }
        Class<?>[] pls = declaredMethod.getParameterTypes();
        System.out.println("参数数量为：" + pls.length);
        for (Class<?> p : pls) {
            System.out.println("参数类型为:" + p.getTypeName());
        }
    }
}

```

## 6 例子

### 例一

​`Runtime.getRuntime().exec("calc.exe");`​

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

​`public Object invoke(Object obj, Object... args)`​

它的第一个参数是执行method的对象：

* 如果这个方法是一个普通方法，那么第一个参数是类对象
* 如果这个方法是一个静态方法，那么第一个参数是类

### 例二

```java
List<String> paramList = new ArrayList<>();
paramList.add("calc.exe");
ProcessBuilder pb = new ProcessBuilder(paramList);
pb.start();
```

ProcessBuilder有两个构造函数

* ​`public ProcessBuilder(List<String> command)`​
* ​`public ProcessBuilder(String... command)`​

根据这两个构造函数，有两个反序列化的构造

```java
Class.forName("java.lang.ProcessBuilder")
                .getMethod("start")
                .invoke(
                        Class.forName("java.lang.ProcessBuilder")
                                .getConstructor(List.class)
                                .newInstance(Arrays.asList("calc.exe"))
                );
```

```java
((ProcessBuilder)Class.forName("java.lang.ProcessBuilder")
                .getConstructor(String[].class)
                .newInstance(new String[][]{{"calc.exe"}}))
                .start();
```

newInstance函数接受参数是一个`Object..`​也就是Object数组，它会给String[][]去掉一层。剩下的String[]匹配ProcessBuilder变量格式。
