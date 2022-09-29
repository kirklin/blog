---
title: 不推荐使用@Autowired进行Field注入的原因
description: 
date: 2022-09-07
image: banner.jpg
categories: [ "后端","Spring" ]

---

今天写Spring项目的时候发现，在@Autowired上出现了一个警告

> Field injection is not recommended （不建议使用字段注入）
>  Inspection info:
> Reports injected or autowired fields in Spring components.
> The quick-fix suggests the recommended constructor-based dependency injection in beans and assertions for mandatory fields.
> Example:
> class MyComponent {
>   @Inject MyCollaborator collaborator; // injected field
>
>   public void myBusinessMethod() {
>     collaborator.doSomething(); // throws NullPointerException
>   }
> }
>
> After applying the quick-fix:
> class MyComponent {
>
>   private final MyCollaborator collaborator;
>
>   @Inject
>   public MyComponent(MyCollaborator collaborator) {
>     Assert.notNull(collaborator, "MyCollaborator must not be null!");
>     this.collaborator = collaborator;
>   }
>
>   public void myBusinessMethod() {
>     collaborator.doSomething(); // now this call is safe
>   }
>
> }

既然提示警告了，说明这种写法肯定存在利弊关系，立马上Google找原因，最后总结如下：

## @AutoWired三种注入的方式

一共有三种：分别是（ Field-based Injection（属性注入），Setter-based Injection（set方法注入），Constructor-base Injection（构造器注入））

### Constructor-base Injection（构造器注入）

```java
public class DI {
    private DependencyA dependencyA;
    private DependencyB dependencyB;
    private DependencyC dependencyC;

    @Autowired
    public DI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC) {
        this.dependencyA = dependencyA;
        this.dependencyB = dependencyB;
        this.dependencyC = dependencyC;
    }
}
```

### Setter-based Injection（set方法注入）

```java
public class DI {
    private DependencyA dependencyA;
    private DependencyB dependencyB;
    private DependencyC dependencyC;

    @Autowired
    public void setDependencyA(DependencyA dependencyA) {
        this.dependencyA = dependencyA;
    }

    @Autowired
    public void setDependencyB(DependencyB dependencyB) {
        this.dependencyB = dependencyB;
    }

    @Autowired
    public void setDependencyC(DependencyC dependencyC) {
        this.dependencyC = dependencyC;
    }
}
```

### Field-based Injection（属性注入）

```java
public class DI {
    @Autowired
    private DependencyA dependencyA;

    @Autowired
    private DependencyB dependencyB;

    @Autowired
    private DependencyC dependencyC;
}
```



## Filed Injection这么香，为什么要用别的？

虽然属性注入写的代码，不会对代码进行过多的改动，不需要额外加上构造器或者set方法，但是这也会带来很多问题。

### 不能使用final修饰符

在使用属性注入之后，属性就不能加final修饰符了。但是一个好的编程习惯是我们应该**尽量将变量声明为final**的，除非变量必须是可变的。
而使用构造器注入，我们可以在属性上添加final修饰符。

在阿里巴巴开发手册上面对final的用法推荐

> 【推荐】final可以声明类、成员变量、方法、以及本地变量，下列情况使用final关键字:
>
> 1. 不允许被继承的类，如: String类。
> 2. 不允许修改引用的域对象，如: PoJo类的域变量。
> 3. 不允许被重写的方法，如:Poo类的setter方法。
> 4. 不允许运行过程中重新赋值的局部变量。
> 5. 避免上下文重复使用一个变量，使用final描述可以强制重新定义一个变量，方便更好地进行重构。
>

### 不利于遵守单一职责原则（SRP：Single responsibility principle）

虽然使用属性注入很方便，这也造成了**引入依赖过多时不够明显**，因为添加一个属性不需要太多额外的代码。
假设我需要添加六个、十个甚至十几个依赖项，使用属性注入，我们很少关心是否违反了SRP原则。
反之用构造器注入代码很快就会显得十分庞大，有太多的依赖通常意味着这个类有太多的职责。 这时候应该考虑一下此组件是不是**违反了单一职责原则**

### 为什么要遵守 SRP 原则

> When a class has more than one responsibility, there are also more triggers and reasons to change that class. A responsibility is the same as “a reason for change” in this context.

因为每一个职责都是变化的因子，当需求变化时，该变化通常反映为类的职责的变化。
如果一个类承担了多于一个的职责，那么就意味着引起它的变化的原因会有多个，等同于把这些职责耦合在了一起。
一个职责的变化可能会抑制到该类完成其他职责的能力，这样的耦合会导致脆弱的设计。

**在设计类或接口时，如果能够遵守 SRP 原则，则会带来以下优点：**

1. **类的复杂性降低：**每个类或接口都只定义单一的职责，定义清晰明确
2. **可读性提高：**定义清晰明确，自然带来较高的代码可读性
3. **可维护性提高：**代码可读性提高，意味着更容易理解；单一职责使得类之间的耦合性较低，更改也会较为容易
4. **扩展性更好：**当需要扩展新的职责时，只需要定义新的接口和新的实现即可

### 隐藏依赖关系

在使用依赖注入时，受影响的类应该使用公共接口清楚地公开这些依赖项，方法是在构造函数中公开所需的依赖项，或者使用方法(setter)公开可选的依赖项。当使用基于字段的依赖注入时，实质上是将这些依赖对外隐藏了。

### 与依赖注入容器紧密耦合

使用基于字段的依赖注入的主要原因是为了避免getter和setter的样板代码或为类创建构造函数。最后，这意味着设置这些字段的唯一方法是通过Spring容器实例化类并使用反射注入它们，否则字段将保持null。

依赖注入设计模式将类依赖项的创建与类本身分离开来，并将此责任转移到类注入容器，从而允许程序设计解耦，并遵循单一职责和依赖项倒置原则(同样可靠)。因此，通过自动装配（Autowired）字段来实现的类的解耦，最终会因为再次与类注入容器(在本例中是Spring)耦合而丢失，从而使类在Spring容器之外变得无用。

这意味着，如果您想在应用程序容器之外使用您的类，例如用于单元测试，您将被迫使用Spring容器来实例化您的类，因为没有其他可能的方法(除了反射)来设置自动装配字段。



## 如何选择DI方式

既然不推荐我们使用属性注入，那么构造器注入和setter又该怎么选呢？这里引用[Spring 推荐的用法](https://docs.spring.io/spring-framework/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-dependency-resolution)

大致意思如下：

- **构造器注入**：**强依赖性**（即必须使用此依赖），**不变性**（各依赖不会经常变动）

- **Setter注入**：**可选**（没有此依赖也可以工作），**可变**（依赖会经常变动）

- **Field注入**：大多数情况下尽量**少使用**字段注入，一定要使用的话， **@Resource相对@Autowired**对IoC容器的**耦合更低**



------

> #### Constructor-based or setter-based DI?
>
> Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies. Note that use of the [@Required](https://docs.spring.io/spring-framework/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-required-annotation) annotation on a setter method can be used to make the property be a required dependency; however, constructor injection with programmatic validation of arguments is preferable.
>
> The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not `null`. Furthermore, constructor-injected components are always returned to the client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.
>
> Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later. Management through [JMX MBeans](https://docs.spring.io/spring-framework/docs/5.2.3.RELEASE/spring-framework-reference/integration.html#jmx) is therefore a compelling use case for setter injection.
>
> Use the DI style that makes the most sense for a particular class. Sometimes, when dealing with third-party classes for which you do not have the source, the choice is made for you. For example, if a third-party class does not expose any setter methods, then constructor injection may be the only available form of DI.



## 结论

应尽量避免属性注入。作为替代，应使用构造函数或方法来注入依赖项。两者都有其优点和缺点，使用取决于具体情况。由于这些方法可以混合使用，因此它不是非此即彼的选择，您可以将 setter 和构造函数注入组合到同一个类中。构造函数更适合于强制依赖项和依赖不会经常变动的情况。设置器更适合没有此依赖也可以工作，依赖会经常变动的情况



## 参考文献

[Field Dependency Injection Considered Harmful | Vojtech Ruzicka's Programming Blog](https://www.vojtechruzicka.com/field-dependency-injection-considered-harmful/)

[java - What exactly is Field Injection and how to avoid it? - Stack Overflow](https://stackoverflow.com/questions/39890849/what-exactly-is-field-injection-and-how-to-avoid-it)

[Core Technologies (spring.io)](https://docs.spring.io/spring-framework/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-dependency-resolution)
