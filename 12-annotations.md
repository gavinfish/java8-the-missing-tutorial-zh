# 注解的改进

---

Java8在注解机制方面进行了一系列的改进，包括：

1. 可重复的注解
2. 类型注解

> 这节的代码在[ch12](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch12)包中

##可重复的注解

在Java8之前，是不能在同一个地点连续用同一个注解两次的，比如你不能将`@Foo`注解在一个方法上使用两次。如果你用过JPA，那么你已经用过一个叫做`@JoinColumns`的注解，它允许像下面这样将多个`@JoinColumn`注解包装起来。

```java
@ManyToOne
@JoinColumns({
    @JoinColumn(name="ADDR_ID", referencedColumnName="ID"),
    @JoinColumn(name="ADDR_ZIP", referencedColumnName="ZIP")
})
public Address getAddress() { return address; }
```

在Java8中，你可以像下面的代码一样写，因为Java8让你可以在同一个地点将一个注解重复多遍。

```java
@ManyToOne
@JoinColumn(name="ADDR_ID", referencedColumnName="ID"),
@JoinColumn(name="ADDR_ZIP", referencedColumnName="ZIP")
public Address getAddress() { return address; }
```

***有了Java8，你可以在你代码的任何地点（类、方法、成员变量声明）多次使用拥有不同参数的同一个注解。***

## 编写自己的可重复注解

为了编写你自己的可重复注解，你需要进行如下的步骤：

**步骤1：**创建如下所示拥有`@Repeatable`元注解的一个注解。`@Repeatable`元注解要求你为存储注解设定一个强制的值，这个值是用来指定创建的注解的容器类型。我们将在步骤2中创建存储注解。

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(CreateVms.class)
public @interface CreateVm {
    String name();
}
```

**步骤2：**创建一个包含着上面注解的存储注解。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface CreateVms {
    CreateVm[] value();
}
```

现在你可以像下面这样在任何方法上多次使用我们的注解。

```java
@CreateVm(name = "vm1")
@CreateVm(name = "vm2")
public void manage() {
    System.out.println("Managing ....");
}
```

如果你需要找到一个方法上所有的可重复注解，那么你可以使用`getAnnotationByType`方法，它现在可以在`java.lang.Class`和`java.lang.reflect.Method`中找到。为了打印出所有虚拟机的名字，你可以编写如下的代码。

```java
CreateVm[] createVms = VmManager.class.getMethod("manage").getAnnotationsByType(CreateVm.class);
Stream.of(createVms).map(CreateVm::name).forEach(System.out::println);
```

## 类型注解

你现在可以将注解应用在额外的两个地点——`TYPE_PARAMETER`和`TYPE_USE`中。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE_PARAMETER})
public @interface MyAnnotation {
}
```

你可以这样使用它。

```java
class MyAnnotationUsage<@MyAnnotation T> {
}
```

你也可以像下面这样将注解作用在类型使用上。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE_USE})
public @interface MyAnnotation {
}
```

然后你就可以像下面那样使用它们。

```java
public static void doSth() {
    List<@MyAnnotation String> names = Arrays.asList("shekhar");
}
```




