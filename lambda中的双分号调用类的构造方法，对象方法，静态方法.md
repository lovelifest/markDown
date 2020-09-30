### lambda中的::

Java 8 中我们可以通过 `::` 关键字来访问类的构造方法，对象方法，静态方法。

现有一个类 Something

```java
class Something {
    // constructor methods
    Something() {}

    Something(String something) {
    System.out.println(something);
    }

    // static methods
    static String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }

    // object methods
    String endWith(String s) {
        return String.valueOf(s.charAt(s.length()-1));
    }

    void endWith() {}
}
```

如何用 '::' 来访问类Something中的方法呢？先定义一个接口，因为必须要用 **functional interface** 来接收，否则编译错误（The target type of this expression must be a functional interface）

```
@FunctionalInterface
interface IConvert<F, T> {
    T convert(F form);
}
```

（**@FunctionalInterface** 注解要求接口有且只有一个抽象方法，JDK中有许多类用到该注解，比如 Runnable，它只有一个 Run 方法。）

**观察接口 IConvert，传参为类型 F，返回类型 T。所以，我们可以这样访问类Something的方法：**

访问静态方法

```java
// static methods
IConvert<String, String> convert = Something::startsWith;
String converted = convert.convert("123");
```

访问对象方法

```java
// object methods
Something something = new Something();
IConvert<String, String> converter = something::endWith;
String converted = converter.convert("Java");
```


访问构造方法

```java
// constructor methods
IConvert<String, Something> convert = Something::new;
Something something = convert.convert("constructors");
```

总结：**我们可以把类Something中的方法static String startsWith(String s){...}、String endWith(String s){...}、Something(String something){...}看作是接口IConvert的实现**，因为它们都符合接口定义的那个“模版”，有传参类型F以及返回值类型T。比如构造方法Something(String something)，它传参为String类型，返回值类型为Something。注解@FunctionalInterface保证了接口有且仅有一个抽象方法，所以JDK能准确地匹配到相应方法。

