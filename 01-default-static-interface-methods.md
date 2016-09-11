# 接口的默认方法和静态方法

---

我们都知道应该面向接口编程。接口给定用户应该使用的协议，而不用依赖该接口的具体实现细节。

因此，为了做到**[松耦合](https://en.wikipedia.org/wiki/Loose_coupling)**，设计出干净的接口成为API设计的要素之一。SOLID五大原则之一的**[接口隔离原则](https://en.wikipedia.org/wiki/Interface_segregation_principle)**要求我们设计有具体目的的小接口，而不是一个通用却臃肿的接口。对你的类库和应用来说，接口设计是能否得到干净而高效的API的关键。

> 这一节的代码在[ch01](https://github.com/shekhargulati/java8-the-missing-tutorial/tree/master/code/src/main/java/com/shekhargulati/java8_tutorial/ch0)包中

如果你曾经设计过API，随着时间的推移，你可能觉得有必要在API中添加新的方法。一旦API已经发布，想要在不破坏已有实现的前提下对一个接口添加方法会变得非常困难。为了说清这一点，假设你正在构建一个简单的能够支持`加`、`减`、`乘`、`除`操作的计算器API。我们可以写一个`Calculator`的接口，如下所示。**为了简单起见，我们将数值类型设为`int`。**

```java
public interface Calculator {

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);
}
```

回到这个`Caculator`接口，你创建了一个`BasicCaculator`的实现类，如下所示。

```java
public class BasicCalculator implements Calculator {

    @Override
    public int add(int first, int second) {
        return first + second;
    }

    @Override
    public int subtract(int first, int second) {
        return first - second;
    }

    @Override
    public int divide(int number, int divisor) {
        if (divisor == 0) {
            throw new IllegalArgumentException("divisor can't be zero.");
        }
        return number / divisor;
    }

    @Override
    public int multiply(int first, int second) {
        return first * second;
    }
}
```

## 静态工厂方法

假设这个计算器API非常有用也容易上手。用户只需要自己创建一个`BasicCalculator`的实例，就能够使用该API。你可以看到如下的代码。

```java
Calculator calculator = new BasicCalculator();
int sum = calculator.add(1, 2);

BasicCalculator cal = new BasicCalculator();
int difference = cal.subtract(3, 2);
```

然而有些用户没有面向接口`Caculator`进行编程，相反，他们面向该接口的 实现类`BasicCalculator`进行编程。你的API没有强制用户面向接口编程，因为`BasicCalculator`是public的。如果你将`BasicCalculator`声明为protected，那么你需要创建一个能够提供`Calculator`实现类的静态工厂。我们通过优化代码来解决这个问题。

首先，我们将`BasicCalculator`声明为protected以便用户不能够直接使用该类。

```java
class BasicCalculator implements Calculator {
  // rest remains same
}
```

接着，我们编写一个能够给我提供`Calculator`实例的工厂类，如下所示。

```java
public abstract class CalculatorFactory {

    public static Calculator getInstance() {
        return new BasicCalculator();
    }
}
```

现在，用户会被迫面向`Calculator`接口进行编程，而且他们不能够知道该接口具体的实现细节。

虽然我们实现了我们的目的，但我们添加的新类`CaculatorFactory`也增加了API的复杂度。现在API的用户在使用API之前需要多了解一个类。这是在该问题在Java8之前唯一的解决方案。

**Java8允许在接口中定义静态方法**。这允许API设计者在接口中定义像`getInstance`一样的静态工具方法，这样就能够使得API简洁而精练。在接口中的静态方法能够用来代替辅助类（像`CalculatorFactory`），通常我们创建这些辅助类来定义一些与特定类型相关的辅助方法。举例来说，`Collections`类就是一个定义了众多辅助方法来与集合和其相关接口进行协作的辅助类。在`Collections`中定义的方法能够轻易的添加到`Collection`接口或者它的子接口中。

以上的代码在Java8中可以通过添加在`Calculator`接口中添加一个`getInstance`方法来改进。

```java
public interface Calculator {

    static Calculator getInstance() {
        return new BasicCalculator();
    }

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);

}
```
## 与时俱进地优化API

有些用户可能决定通过添加像`remainder`这样的方法，或者为`Calculator`接口给出他们自己的实现来扩展`Calculator`API。通过与用户的沟通后，你发现大多数用户想要在`Calculator`接口中添加一个`remainder`方法。这看起来是一个非常简单的API变动，所以你在API中多添加了一个方法。

```java
public interface Calculator {

    static Calculator getInstance() {
        return new BasicCalculator();
    }

    int add(int first, int second);

    int subtract(int first, int second);

    int divide(int number, int divisor);

    int multiply(int first, int second);

    int remainder(int number, int divisor); // new method added to API
}
```

在接口中添加方法破坏了API的源码兼容性。这意味着实现了`Calculator`接口的用户需要添加`remainder`方法的实现，否则他们的代码将不能通过编译。这对于API开发者来说是一个大问题，这使得API很难进行改进。在Java8之前，接口中是不能有方法的具体实现的。这往往在API需要拓展的时候成为一个问题，比如在接口定义中添加一两个方法。

为了使API随着时间的推移能够不断改进，Java8允许用户在接口中给方法提供默认实现。这些方法被称为**default**方法，或者**defender**方法。实现了该接口的类不需要给这些方法提供实现。如果一个实现类为这些方法提供了实现，那么新给出的实现将会被使用，否则接口中的默认实现将被使用。`List`接口定义了一些`default`方法，像`replaceAll`、`sort`和`splitIterator`。

```java
default void replaceAll(UnaryOperator<E> operator) {
    Objects.requireNonNull(operator);
    final ListIterator<E> li = this.listIterator();
    while (li.hasNext()) {
        li.set(operator.apply(li.next()));
    }
}
```

如下面的代码所示，我们可以通过添加一个`default`方法来解决上述的API问题。`default`方法通常使用已经存在的方法进行定义，如`remainder`方法使用了`subtract`、`multiply`和`divide`方法。

```java
default int remainder(int number, int divisor) {
    return subtract(number, multiply(divisor, divide(number, divisor)));
}
```

## 多重继承

Java中一个类只能继承一个类，但可以实现多个接口。现在在接口中包含方法的实现是可行的，Java也就有了类似多重继承的特性。Java在类型层面已经存在多重继承特性，现在在行为层面也有了多重继承特性。有三条规则来帮助决定哪一个方法会被选中。

**规则1：在类中定义的方法胜过在接口中定义的方法**

```java
interface A {
    default void doSth(){
        System.out.println("inside A");
    }
}
class App implements A{

    @Override
    public void doSth() {
        System.out.println("inside App");
    }

    public static void main(String[] args) {
        new App().doSth();
    }
}
```
这将打印出`inside App`，因为在实现类中已经覆盖了接口中定义的方法。

**规则2：明确的接口胜过上层接口**

```java
interface A {
    default void doSth() {
        System.out.println("inside A");
    }
}
interface B {}
interface C extends A {
    default void doSth() {
        System.out.println("inside C");
    }
}
class App implements C, B, A {

    public static void main(String[] args) {
        new App().doSth();
    }
}
```
这将打印出`inside C`。

**规则3：执行在类中明确指明的实现方式**

```java
interface A {
    default void doSth() {
        System.out.println("inside A");
    }
}
interface B {
    default void doSth() {
        System.out.println("inside B");
    }
}
class App implements B, A {

    @Override
    public void doSth() {
        B.super.doSth();
    }

    public static void main(String[] args) {
      new App().doSth();
    }
}
```

这将会打印出`inside B`。

