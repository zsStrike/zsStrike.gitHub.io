---
title: 《Effective Java》笔记
date: 2020-12-06 18:04:57
tags: ["Java"]
---

本文是《Effective Java》第三版的读书笔记。

<!-- More -->

## 第二章 创建和销毁对象

1. 考虑使用静态工厂方法代替构造函数：获取一个类的实例的传统方式是使用类提供的公开构造函数，另外一种方法是类提供公开静态工厂方法，用于返回实例。使用静态工厂方法优点：

   + 静态工厂方法有确切名称，便于阅读
   + 静态工厂方法不需要在每次调用时创建新对象
   + 可以通过静态工厂方法获取返回类型的任何子类的对象，提供灵活性
   + 返回对象的类可以随调用的不同而变化，作为输入参数的函数，声明的返回类型的任何子类型都是允许的
   + 当编写包含方法的类时，返回对象的类不需要存在，如JDBC

   静态工厂方法缺点：

   + 没有公共或受保护构造函数的类不能被子类化
   + 程序员很难找到它们，下面是一些静态工厂方法的常用名称：
     + from：一种类型转换方法，接收单个参数并且返回相应实例
     + of：一个聚合方法，接受多个参数返回一个实例
     + valueOf：替代from和of但是更加冗长的方法
     + instance或getInstance：返回一个实例，该实例由参数描述，但具有不同的值（可能会缓存）
     + create或newInstance：该方法保证每个调用都返回一个新实例
     + getType：类似于 getInstance，但如果工厂方法位于不同的类中，则使用此方法
     + newType：与 newInstance 类似，但是如果工厂方法在不同的类中使用
     + type：一个用来替代 getType 和 newType 的比较简单的方式

2. 当构造函数有多个参数的时候，考虑使用构造器：静态工厂和构造函数都有一个局限，就是不能对大量可选参数做很好的扩展。当我们的可选参数个数大于4个时，往往需要重载很多个构造函数，会降低代码的可维护性。另外一种选择是JavaBean模式，但是JavaBean可能在构建的过程中处于不一致状态。此时我们可以使用构造器来生成所需对象。

   ```java
   // Builder Pattern
   public class NutritionFacts {
       private final int servingSize;
       private final int servings;
       private final int calories;
       private final int fat;
       private final int sodium;
       private final int carbohydrate;
   
       public static class Builder {
           // Required parameters
           private final int servingSize;
           private final int servings;
           // Optional parameters - initialized to default values
           private int calories = 0;
           private int fat = 0;
           private int sodium = 0;
           private int carbohydrate = 0;
   
           public Builder(int servingSize, int servings) {
               this.servingSize = servingSize;
               this.servings = servings;
           }
   
           public Builder calories(int val) {
               calories = val;
               return this;
           }
   
           public Builder fat(int val) {
               fat = val;
               return this;
           }
   
           public Builder sodium(int val) {
               sodium = val;
               return this;
           }
   
           public Builder carbohydrate(int val) {
               carbohydrate = val;
               return this;
           }
   
           public NutritionFacts build() {
               return new NutritionFacts(this);
           }
       }
   
       private NutritionFacts(Builder builder) {
           servingSize = builder.servingSize;
           servings = builder.servings;
           calories = builder.calories;
           fat = builder.fat;
           sodium = builder.sodium;
           carbohydrate = builder.carbohydrate;
       }
   }
   ```

   这样我们在生成代码的时候就可以通过链式调用来生成我们的对象实例。构造器模式很灵活，一个构造器可以构造多个对象。但是构造器的缺点就是为了创建一个对象，必须首先创建它的构造器。

3. 使用私有构造函数或枚举类型实施单例模式：实现单例模式的第一种方法：

   ```java
   // Singleton with public final field
   public class Elvis {
       public static final Elvis INSTANCE = new Elvis();
       private Elvis() { ... }
       public void leaveTheBuilding() { ... }
   }
   ```

   上述代码可以防止用户来自己创建Elvis实例。但是拥有特殊权限的客户端可以借助AccessibleObject.setAccessible 方法利用反射调用私有构造函数，如果需要防范这种问题，需要修改构造器，使其在请求创建第二个实例的时候抛出异常即可。另外一种方法：

   ```java
   // Singleton with static factory
   public class Elvis {
       private static final Elvis INSTANCE = new Elvis();
       private Elvis() { ... }
       public static Elvis getInstance() { return INSTANCE; }
       public void leaveTheBuilding() { ... }
   }
   ```

   静态工厂方法的一个优点是，它可以在不更改 API 的情况下决定类是否是单例，如为每个线程返回一个单例；第二个优点是，如果应用程序需要的话，可以编写泛型的单例工厂。实现单例的第三种方法：

   ```java
   // Enum singleton - the preferred approach
   public enum Elvis {
       INSTANCE;
       public void leaveTheBuilding() { ... }
   }
   ```

   这种方法类似于 public 字段方法，但是它更简洁，默认提供了序列化机制，提供了对多个实例化的严格保证，即使面对复杂的序列化或反射攻击也是如此。

4. 用私有构造函数实现不可实例化：对于一个工具类库，如Arrays，实例化这些类是没有意义的。试图通过使类抽象来实施不可实例化是行不通的。因为可以对类进行子类化，并实例化子类。有一个简单的习惯用法来确保不可实例化。只有当类不包含显式构造函数时，才会生成默认构造函数，因此可以通过包含私有构造函数使类不可实例化：

   ```java
   // Noninstantiable utility class
   public class UtilityClass {
       // Suppress default constructor for noninstantiability
       private UtilityClass() {
           throw new AssertionError();
       } ... // Remainder omitted
   }
   ```

   因为显式构造函数是私有的，所以在类之外是不可访问的。AssertionError 不是严格要求的，但是它提供了保障，以防构造函数意外地被调用。

5. 依赖注入优于硬连接资源：尽量将类依赖的资源在创建新实例时将资源传递给构造函数，从而实现依赖注入。

   ```java
   // Dependency injection provides flexibility and testability
   public class SpellChecker {
       private final Lexicon dictionary;
       public SpellChecker(Lexicon dictionary) {
           this.dictionary = Objects.requireNonNull(dictionary);
       }
       public boolean isValid(String word) { ... }
       public List<String> suggestions(String typo) { ... }
   }
   ```

   另外，这种模式的一个有用变体是将资源工厂传递给构造函数。Java 8 中引入的 `Supplier<T>` 非常适合表示工厂。尽管依赖注入极大地提高了灵活性和可测试性，但它可能会使大型项目变得混乱，这些项目通常包含数千个依赖项。

6. 避免创建不必要的对象：作为一个不该做的极端例子，请考虑下面的语句：

   ```java
   String s = new String("bikini"); // DON'T DO THIS!
   ```

   该语句每次执行时都会创建一个新的 String 实例，而这些对象创建都不是必需的。String 构造函数的参数 `("bikini")` 本身就是一个 String 实例，在功能上与构造函数创建的所有对象相同。另外，有些对象的创建代价很高，如果你需要重复地使用这样一个「昂贵的对象」，那么最好将其缓存以供复用：

   ```java
   // Reusing expensive object for improved performance
   public class RomanNumerals {
       private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
       static boolean isRomanNumeral(String s) {
           return ROMAN.matcher(s).matches();
       }
   }
   ```

   另外，还需要注意基本类型优于包装类，需要提防意外的自动自动装箱。

7. 排除过时的对象引用：考虑一个栈的pop操作：

   ```java
   public Object pop() {
       if (size == 0)
       	throw new EmptyStackException();
       return elements[--size];
   }
   ```

   上述代码没有明显的问题，但是存在内存泄露的隐患：如果堆栈增长，然后收缩，那么从堆栈中弹出的对象将不会被垃圾收集，即使使用堆栈的程序不再引用它们。改进方式：

   ```java
   public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
   }
   ```

   一般来说，一个类管理它自己的内存时，程序员应该警惕内存泄漏。当释放一个元素时，该元素中包含的任何对象引用都应该被置为 null。

   另一个常见的内存泄漏源是缓存。一旦将对象引用放入缓存中，就很容易忘记它就在那里，并且在它变得无关紧要之后很久仍将它留在缓存中。如果你非常幸运地实现了一个缓存，只要缓存外有对其键的引用，那么就将缓存表示为 WeakHashMap，当条目过时后，条目将被自动删除。

   内存泄漏的第三个常见来源是侦听器和其他回调。 如果你实现了一个 API，其中客户端注册回调，但不显式取消它们，除非你采取一些行动，否则它们将累积。

8. 避免使用终结器和清除器：终结器是不可预测的，通常是危险的，也是不必要的。清除器的危险比终结器小，但仍然不可预测、缓慢，而且通常是不必要的。终结器和清除器的一个缺点是不能保证它们会被立即执行，另外一个缺点是它们可能会使的即将要被清理的对象死而复生。终结器和清除器可以充当一个安全网，以防资源的所有者忽略调用它的 close 方法。

9. 使用try-with-resources优于try-finally：从历史上看，try-finally 语句是确保正确关闭资源的最佳方法，即使在出现异常或返回时也是如此。但是当存在两个资源的时候，可能就需要嵌套的调用了，这会导致代码不易阅读。最好的方法就是使用try-with-resources：

   ```java
   // try-with-resources on multiple resources - short and sweet
   static void copy(String src, String dst) throws IOException {
       try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
           byte[] buf = new byte[BUFFER_SIZE];
           int n;
           while ((n = in.read(buf)) >= 0)
               out.write(buf, 0, n);
       }
   }
   ```




## 第三章 对象的通用方法

10. 覆盖 equals 方法时应该遵守的约定：当满足下面的条件的时候，不应该覆盖equals方法：

    + 类的每个实例本质上是唯一的
    + 该类不需要提供逻辑相等测试
    + 超类已经覆盖了equals，超类行为适合于这个类
    + 类是私有的或者包私有的，并且你确信它的 equals 方法永远不会被调用

    equals方法实现了等价关系：反身性，对称性，传递性，一致性，最后还需要满足非无效性：即`o.equals(null)`返回false。为了搞笑实现equals方法，需要：

    + 使用 == 运算符检查参数是否是对该对象的引用
    + 使用 instanceof 运算符检查参数是否具有正确的类型
    + 将参数转换为正确的类型
    + 对于类中的每个「重要」字段，检查参数的字段是否与该对象的相应字段匹配
    + 是否满足等价关系

11. 当覆盖 equals 方法的时候，总要覆盖 hashCode 方法：由于相等的对象必须具有相等的散列码，如果PhoneNumber没有实现hashCode方法的话：

    ```java
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "Jenny");
    // m.get(new PhoneNumber(707, 867,5309)) == null
    ```

    第三行的结果将是null，而不是`"Jenny"`。实现hashCode方法的一个简单方法步骤：

    + 声明一个名为 result 的 int 变量，并将其初始化为对象中第一个重要字段的散列码 c
    + 对象中剩余的重要字段 f，执行以下操作：
      + 为字段计算一个整数散列码 c：如果字段是基本数据类型，计算 `Type.hashCode(f)`，其中 type 是与 f 类型对应的包装类。如果字段是对象引用，并且该类的 equals 方法通过递归调用 equals 方法来比较字段，则递归调用字段上的 hashCode 方法。如果字段是一个数组，则将其每个重要元素都视为一个单独的字段。也就是说，通过递归地应用这些规则计算每个重要元素的散列码，并将每个步骤 2.b 的值组合起来。如果数组中没有重要元素，则使用常量，最好不是 0。如果所有元素都很重要，那么使用 `Arrays.hashCode`。
      + 将步骤 2.a 中计算的散列码 c 合并到 result 变量
    + 返回result

    一个简单的demo：

    ```java
    // Typical hashCode method
    @Override
    public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
    ```

12. 始终覆盖 toString 方法：虽然Object提供了默认的toString方法，但是它返回的字符串通常不是用户希望看到的。提供一个好的 toString 实现（能）使类更易于使用，使用该类的系统（也）更易于调试。当实际使用时，toString 方法应该返回对象中包含的所有有用信息。

13. 明智地覆盖 clone 方法：Cloneable 接口的目的是作为 mixin 接口，用于让类来宣称它们允许克隆。不幸的是，它没有达到这个目的。它的主要缺点是缺少 clone 方法，并且 Object 类的 clone 方法是受保护的。它决定了 Object 类受保护的 clone 实现的行为：如果一个类实现了 Cloneable 接口，Object 类的 clone 方法则返回该类实例的逐字段拷贝；否则它会抛出 CloneNotSupportedException。默认提供的clone方法执行的是浅拷贝，如果需要深拷贝，就需要自己覆盖clone方法，实现该功能。

14. 考虑实现 Comparable 接口：与本章讨论的其他方法不同，compareTo 方法不是在 Object 中声明的。相反，它是 Comparable 接口中的唯一方法。通过让类实现 Comparable，就可与依赖于此接口的所有通用算法和集合实现进行互操作。如果一个类有多个重要的字段，此时就需要用户来指定对应的比较顺序。在 Java 8 中，Comparator 接口配备了一组比较器构造方法，可以流畅地构造比较器。然后可以使用这些比较器来实现 Comparator 接口所要求的 compareTo 方法。

    ```java
    // Comparable with comparator construction methods
    private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);
    
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
    ```

    

## 第四章 类和接口

15. 尽量减少类和成员的可访问性：隐藏内部数据和其他实现细节用于实现信息封装，可以解耦组成系统的组件。通用方法是让每个类或者成员尽可能不可访问。对于顶级（非嵌套）类和接口，只有两个可能的访问级别：包私有和公共。如果一个方法覆盖了超类方法，那么它在子类中的访问级别就不能比超类更严格。公共类的实例字段很少采用 public 修饰，因为带有公共可变字段的类通常不是线程安全的。请注意，非零长度的数组总是可变的，因此对于类来说，拥有一个公共静态 final 数组字段或返回该字段的访问器是错误的。如果一个类具有这样的字段或访问器，客户端将能够修改数组的内容。对于 Java 9，作为模块系统的一部分，还引入了另外两个隐式访问级别。模块是包的分组单位，就像包是类的分组单位一样。模块可以通过模块声明中的导出声明显式地导出它的一些包。

16. 在公共类中，使用访问器方法，而不是公共字段：如果类可以在包之外访问，那么提供访问器方法来保持更改类内部表示的灵活性。但是，如果一个类是包级私有的或者是私有嵌套类，那么公开它的数据字段并没有什么本质上的错误。无论是在类定义还是在使用它的客户端代码中，这种方法产生的视觉混乱都比访问方法少。虽然公共类直接公开字段从来都不是一个好主意，但是如果字段是不可变的，那么危害就会小一些。

17. 减少可变性：不可变类就是一个实例不能被修改的类。要使类不可变，请遵循以下 5 条规则：

    1. 不要提供修改对象状态的方法
    2. 确保类不能被扩展
    3. 所有字段用 final 修饰
    4. 所有字段设为私有
    5. 确保对任何可变组件的独占访问

    不可变对象提供的好处：

    1. 不可变对象本质上是线程安全的
    2. 不可变对象可以很好的作为其他对象的构建模块
    3. 不可变对象自带提供故障原子性。他们的状态从未改变，所以不可能出现暂时的不一致。

    不可变类的主要缺点是每个不同的值都需要一个单独的对象。

18. 优先选择复合而不是继承：在包中使用继承是安全的，其中子类和超类实现由相同的程序员控制。在对专为扩展而设计和文档化的类时使用继承也是安全的。然而，对普通的具体类进行跨包边界的继承是危险的。与方法调用不同，继承破坏了封装。换句话说，子类的功能正确与否依赖于它的超类的实现细节。子类脆弱的一个原因是他们的超类可以在后续版本中获得新的方法。有一种方法可以避免上述所有问题。与其扩展现有类，不如为新类提供一个引用现有类实例的私有字段。这种设计称为复合，因为现有的类是新类的一个组件。只有在子类确实是超类的子类型的情况下，继承才合适。换句话说，只有当两个类之间存在「is-a」关系时，类 B 才应该扩展类 A。

19. 继承要设计良好并且具有文档，否则禁止使用：首先，类必须精确地在文档中记录覆盖任何方法的效果。换句话说，类必须在文档中记录它对可覆盖方法的自用性。对于每个公共或受保护的方法，文档必须指出方法调用的可覆盖方法、调用顺序以及每次调用的结果如何影响后续处理过程。但是，这是否违背了一个格言：好的 API 文档应该描述一个给定的方法做什么，而不是如何做？是的，它确实违背了！这是继承违反封装这一事实的不幸结果。要为一个类编制文档，使其能够安全地子类化，你必须描述实现细节，否则这些细节应该是未指定的。为了允许继承，类必须遵守更多的限制。构造函数不能直接或间接调用可重写的方法。 如果你违反了这个规则，程序就会失败。超类构造函数在子类构造函数之前运行，因此在子类构造函数运行之前将调用子类中的覆盖方法。如果重写方法依赖于子类构造函数执行的任何初始化，则该方法的行为将不像预期的那样。

20. 接口优于抽象类：Java 有两种机制来定义允许多种实现的类型：接口和抽象类。由于 Java 8 中引入了接口的默认方法，这两种机制都允许你为一些实例方法提供实现。一个主要区别是，一个类要实现抽象类定义的类型，该类必须是抽象类的子类。因为 Java 只允许单一继承，这种限制对抽象类而言严重制约了它们作为类型定义的使用。使用接口的优点：

    1. 可以很容易地对现有类进行改造，以实现新的接口
    2. 接口是定义 mixin（混合类型）的理想工具
    3. 接口允许构造非层次化类型框架

21. 为后代设计接口：在 Java 8 之前，在不破坏现有实现的情况下向接口添加方法是不可能的。如果在接口中添加新方法，通常导致现有的实现出现编译时错误，提示缺少该方法。在 Java 8 中，添加了默认的方法构造，目的是允许向现有接口添加方法。除非必要，否则应该避免使用默认方法向现有接口添加新方法，在这种情况下，你应该仔细考虑现有接口实现是否可能被默认方法破坏。尽管默认方法现在已经是 Java 平台的一部分，但是谨慎地设计接口仍然是非常重要的。虽然默认方法使向现有接口添加方法成为可能，但这样做存在很大风险。 如果一个接口包含一个小缺陷，它可能会永远影响它的使用者；如果接口有严重缺陷，它可能会毁掉包含它的 API。

22. 接口只用于定义类型：当一个类实现了一个接口时，这个接口作为一种类型，可以用来引用类的实例。不满足上述条件的一种接口是所谓的常量接口。如果你想导出常量，有几个合理的选择。如果这些常量与现有的类或接口紧密绑定，则应该将它们添加到类或接口。例如，所有数值包装类，比如 Integer 和 Double，都导出 MIN_VALUE 和 MAX_VALUE 常量。如果将这些常量看作枚举类型的成员，那么应该使用 enum 类型导出它们。否则，你应该使用不可实例化的工具类导出常量。

23. 类层次结构优于带标签的类：有时候，你可能会遇到这样一个类，它的实例有两种或两种以上的样式，并且包含一个标签字段来表示实例的样式。这样的标签类有许多缺点。它们充斥着样板代码，包括 enum 声明、标签字段和 switch 语句。标签类冗长、容易出错和低效。面向对象的语言提供了一个更好的选择来定义能够表示多种类型对象的单一数据类型：子类型。标签类只是类层次结构的简易模仿。

24. 静态成员类优于非静态成员类：有四种嵌套类：静态成员类、非静态成员类、匿名类和局部类。除了第一种，所有的类都被称为内部类。静态成员类是最简单的嵌套类。最好把它看做是一个普通的类，只是碰巧在另一个类中声明而已，并且可以访问外部类的所有成员，甚至那些声明为 private 的成员。静态成员类的一个常见用法是作为公有的辅助类。从语法上讲，静态成员类和非静态成员类之间的唯一区别是静态成员类在其声明中具有修饰符 static。如果声明的成员类不需要访问外部的实例，那么应始终在声明中添加 static 修饰符，使其成为静态的而不是非静态的成员类。匿名类的适用性有很多限制。你不能实例化它们，除非在声明它们的时候。在 lambda 表达式被添加到 Java 之前，匿名类是动态创建小型函数对象和进程对象的首选方法，但 lambda 表达式现在是首选方法。局部类是四种嵌套类中最不常用的。局部类几乎可以在任何能够声明局部变量的地方使用，并且遵守相同的作用域规则。局部类具有与其他嵌套类相同的属性。

25. 源文件仅限有单个顶层类：虽然 Java 编译器允许你在单个源文件中定义多个顶层类，但这样做没有任何好处，而且存在重大风险。这种风险源于这样一个事实：在源文件中定义多个顶层类使得为一个类提供多个定义成为可能。



## 第五章 泛型

26. 不要使用原始类型：声明中具有一个或多个类型参数的类或接口就是泛型类或泛型接口，每个泛型都定义了一个原始类型，它是没有任何相关类型参数的泛型的名称。例如，`List<E>` 对应的原始类型是 List。原始类型的行为就好像所有泛型信息都从类型声明中删除了一样。它们的存在主要是为了与之前的泛型代码兼容。当从集合中检索元素时，编译器会为你执行不可见的强制类型转换，并确保它们不会失败。使用原始类型（没有类型参数的泛型）是合法的，但是你永远不应该这样做。如果使用原始类型，就会失去泛型的安全性和表现力。考虑如下程序：

    ```java
    // Fails at runtime - unsafeAdd method uses a raw type (List)!
    
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0); // Has compiler-generated cast
    }
    
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
    ```

    该程序可以编译，但因为它使用原始类型 List，所以你会得到一个警告：

    ```
    Test.java:10: warning: [unchecked] unchecked call to add(E) as a
    member of the raw type List
    list.add(o);
    ^
    ```

    实际上，如果你运行程序，当程序试图将调用 `strings.get(0)` 的结果强制转换为字符串时，你会得到一个 ClassCastException。这是一个由编译器生成的强制类型转换，它通常都能成功，但在本例中，我们忽略了编译器的警告，并为此付出了代价。

    如果将 unsafeAdd 声明中的原始类型 List 替换为参数化类型 `List`，并尝试重新编译程序，你会发现它不再编译，而是发出错误消息：

    ```
    Test.java:5: error: incompatible types: List<String> cannot be
    converted to List<Object>
    unsafeAdd(strings, Integer.valueOf(42));
    ^
    ```

    对于元素类型未知且无关紧要的集合，你可能会尝试使用原始类型。这种方法是可行的，但是它使用的是原始类型，这是很危险的。安全的替代方法是使用无界通配符类型。如果你想使用泛型，但不知道或不关心实际的类型参数是什么，那么可以使用问号代替。

    ```
    // Uses unbounded wildcard type - typesafe and flexible
    static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
    ```

    对于不应该使用原始类型的规则，有一些小的例外。必须在类字面量中使用原始类型。换句话说，`List.class`，`String[].class` 和 `int.class` 都是合法的，但是 `List.class` 和 `List.class` 不是。第二个例外是 instanceof 运算符。由于泛型信息在运行时被删除，因此在不是无界通配符类型之外的参数化类型上使用 instanceof 操作符是非法的。使用无界通配符类型代替原始类型不会以任何方式影响 instanceof 运算符的行为。在这种情况下，尖括号和问号只是多余的。下面的例子是使用通用类型 instanceof 运算符的首选方法：

    ```java
    // Legitimate use of raw type - instanceof operator
    if (o instanceof Set) { // Raw type
        Set<?> s = (Set<?>) o; // Wildcard type
        ...
    }
    ```

    总之，使用原始类型可能会在运行时导致异常，所以不要轻易使用它们。它们仅用于与引入泛型之前的遗留代码进行兼容和互操作。快速回顾一下，`Set` 是一个参数化类型，表示可以包含任何类型的对象的集合，`Set` 是一个通配符类型，表示只能包含某种未知类型的对象的集合，Set 是一个原始类型，它选择了泛型系统。前两个是安全的，后一个就不安全了。

27. 消除 unchecked 警告：使用泛型获得的经验越多，得到的警告就越少，但是不要期望新编写的代码能够完全正确地编译。力求消除所有 unchecked 警告。 如果你消除了所有警告，你就可以确信你的代码是类型安全的，这是一件非常好的事情。如果不能消除警告，但是可以证明引发警告的代码是类型安全的，那么（并且只有在那时）使用 SuppressWarnings("unchecked") 注解来抑制警告。SuppressWarnings 注解可以用于任何声明中，从单个局部变量声明到整个类。总是在尽可能小的范围上使用 SuppressWarnings 注解。 每次使用SuppressWarnings("unchecked") 注解时，要添加一条注释，说明这样做是安全的。

28. list 优于数组：数组与泛型有两个重要区别。首先，数组是协变的。这个听起来很吓人的单词的意思很简单，如果 Sub 是 Super 的一个子类型，那么数组类型 Sub[] 就是数组类型 Super[] 的一个子类型。数组和泛型之间的第二个主要区别：数组是具体化的。这意味着数组在运行时知道并强制执行他们的元素类型。相比之下，泛型是通过擦除来实现的。

    由于这些基本差异，数组和泛型不能很好地混合。例如，创建泛型、参数化类型或类型参数的数组是非法的。因此，这些数组创建表达式都不是合法的：`new List[]`、`new List[]`、`new E[]`。所有这些都会在编译时导致泛型数组创建错误。为了更具体，请考虑以下代码片段：

    ```java
    // Why generic array creation is illegal - won't compile!
    List<String>[] stringLists = new List<String>[1]; // (1)
    List<Integer> intList = List.of(42); // (2)
    Object[] objects = stringLists; // (3)
    objects[0] = intList; // (4)
    String s = stringLists[0].get(0); // (5)
    ```

    假设创建泛型数组的第 1 行是合法的。第 2 行创建并初始化一个包含单个元素的 `List`。第 3 行将 `List` 数组存储到 Object 类型的数组变量中，这是合法的，因为数组是协变的。第 4 行将 `List` 存储到 Object 类型的数组的唯一元素中，这是成功的，因为泛型是由擦除实现的：`List` 实例的运行时类型是 List，`List`[] 实例的运行时类型是 List[]，因此这个赋值不会生成 ArrayStoreException。现在我们有麻烦了。我们将一个 `List` 实例存储到一个数组中，该数组声明只保存 `List` 实例。在第 5 行，我们从这个数组的唯一列表中检索唯一元素。编译器自动将检索到的元素转换为 String 类型，但它是一个 Integer 类型的元素，因此我们在运行时得到一个 ClassCastException。为了防止这种情况发生，第 1 行（创建泛型数组）必须生成编译时错误。

    当你在转换为数组类型时遇到泛型数组创建错误或 unchecked 强制转换警告时，通常最好的解决方案是使用集合类型 `List`，而不是数组类型 E[]。

    总之，数组和泛型有非常不同的类型规则。数组是协变的、具体化的；泛型是不变的和可被擦除的。因此，数组提供了运行时类型安全，而不是编译时类型安全，对于泛型反之亦然。一般来说，数组和泛型不能很好地混合。如果你发现将它们混合在一起并得到编译时错误或警告，那么你的第一个反应该是将数组替换为 list。

29. 优先使用泛型：考虑一个泛型栈结构：

    ```java
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
    ```

    通常至少会得到一个错误或警告，这个类也不例外。幸运的是，这个类只生成一个错误：

    ```
    Stack.java:8: generic array creation
    elements = new E[DEFAULT_INITIAL_CAPACITY];
    ^
    ```

    每当你编写由数组支持的泛型时，就会出现这个问题。有两种合理的方法来解决它。第一个解决方案直接绕过了创建泛型数组的禁令：创建对象数组并将其强制转换为泛型数组类型。现在，编译器将发出一个警告来代替错误。这种用法是合法的，但（一般而言）它不是类型安全的：

    ```
    Stack.java:8: warning: [unchecked] unchecked cast
    found: Object[], required: E[]
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    ^
    ```

    消除 Stack 中泛型数组创建错误的第二种方法是将字段元素的类型从 E[] 更改为 Object[]。如果你这样做，你会得到一个不同的错误：

    ```
    Stack.java:19: incompatible types
    found: Object, required: E
    E result = elements[--size];
    ^
    ```

    通过将从数组中检索到的元素转换为 E，可以将此错误转换为警告，但你将得到警告：

    ```
    Stack.java:19: warning: [unchecked] unchecked cast
    found: Object, required: E
    E result = (E) elements[--size];
    ^
    ```

    消除泛型数组创建的两种技术都有其追随者。第一个更容易读：数组声明为 E[] 类型，这清楚地表明它只包含 E 的实例。它也更简洁：在一个典型的泛型类中，从数组中读取代码中的许多点；第一种技术只需要一次转换（在创建数组的地方），而第二种技术在每次读取数组元素时都需要单独的转换。因此，第一种技术是可取的，在实践中更常用。

    泛型比需要在客户端代码中转换的类型更安全、更容易使用。

30. 优先使用泛型方法：允许类型参数被包含该类型参数本身的表达式限制，尽管这种情况比较少见。这就是所谓的递归类型限定。递归类型边界的一个常见用法是与 Comparable 接口相关联，后者定义了类型的自然顺序：

    ```java
    public interface Comparable<T> {
        int compareTo(T o);
    }
    ```

    许多方法采用实现 Comparable 的元素集合，在其中进行搜索，计算其最小值或最大值，等等。要做到这些，需要集合中的每个元素与集合中的每个其他元素相比较，换句话说，就是列表中的元素相互比较。

    ```java
    // Using a recursive type bound to express mutual comparability
    public static <E extends Comparable<E>> E max(Collection<E> c);
    ```

    类型限定 `<E extends Comparable<E>>` 可以被理解为「可以与自身进行比较的任何类型 E」，这或多或少与相互可比性的概念相对应。

31. 使用有界通配符增加 API 的灵活性：假设我们想添加一个方法，该方法接受一系列元素并将它们全部推入堆栈。这是第一次尝试：

    ```java
    // pushAll method without wildcard type - deficient!
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }
    ```

    该方法能够正确编译，但并不完全令人满意。下面的代码将会产生错误：

    ```java
    Stack<Number> numberStack = new Stack<>();
    Iterable<Integer> integers = ... ;
    numberStack.pushAll(integers);
    ```

    错误信息：

    ```java
    StackTest.java:7: error: incompatible types: Iterable<Integer>
    cannot be converted to Iterable<Number>
            numberStack.pushAll(integers);
                        ^
    ```

    Java 提供了一种特殊的参数化类型，有界通配符类型来处理这种情况。pushAll 的输入参数的类型不应该是「E 的 Iterable 接口」，而应该是「E 的某个子类型的 Iterable 接口」，并且有一个通配符类型，它的确切含义是：`Iterable<? extends E>`：

    ```java
    // Wildcard type for a parameter that serves as an E producer
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
    }
    ```

    popAll方法的代码如下：

    ```java
    // popAll method without wildcard type - deficient!
    public void popAll(Collection<E> dst) {
        while (!isEmpty())
            dst.add(pop());
    }
    ```

    同样，如果目标集合的元素类型与堆栈的元素类型完全匹配，那么这种方法可以很好地编译。但这也不是完全令人满意：

    ```java
    Stack<Number> numberStack = new Stack<Number>();
    Collection<Object> objects = ... ;
    numberStack.popAll(objects);
    ```

    同样，有一个通配符类型，它的确切含义是：`Collection<? super E>`。

    ```java
    // Wildcard type for parameter that serves as an E consumer
    public void popAll(Collection<? super E> dst) {
      while (!isEmpty())
        dst.add(pop());
    }
    ```

    总而言之，PECS 表示生产者应使用 extends，消费者应使用 super。

32. 明智地合用泛型和可变参数：泛型和可变参数不能很好的结合起来。考虑如下代码：

    ```java
    // Mixing generics and varargs can violate type safety!
    // 泛型和可变参数混合使用可能违反类型安全原则！
    static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList; // Heap pollution
        String s = stringLists[0].get(0); // ClassCastException
    }
    ```

    此方法没有显式的强制类型转换，但在使用一个或多个参数调用时抛出 ClassCastException。它的最后一行有一个由编译器生成的隐式强制转换。此转换失败，表明类型安全性受到了影响，并且在泛型可变参数数组中存储值是不安全的。为什么使用泛型可变参数声明方法是合法的，而显式创建泛型数组是非法的？答案是，带有泛型或参数化类型的可变参数的方法在实际开发中非常有用，因此语言设计人员选择忍受这种不一致性。事实上，Java 库导出了几个这样的方法，包括 Arrays.asList(T... a)、Collections.addAll(Collection<? super T> c, T... elements) 以及 EnumSet.of(E first, E... rest)。

    在 Java 7 中添加了 SafeVarargs 注释，以允许使用泛型可变参数的方法的作者自动抑制客户端警告。本质上，SafeVarargs 注释构成了方法作者的一个承诺，即该方法是类型安全的。 

    可变参数方法和泛型不能很好地交互，因为可变参数工具是构建在数组之上的漏洞抽象，并且数组具有与泛型不同的类型规则。虽然泛型可变参数不是类型安全的，但它们是合法的。如果选择使用泛型（或参数化）可变参数编写方法，首先要确保该方法是类型安全的，然后使用 @SafeVarargs 对其进行注释。

33. 考虑类型安全的异构容器：

    ```java
    // Typesafe heterogeneous container pattern - implementation
    public class Favorites {
      private Map<Class<?>, Object> favorites = new HashMap<>();
    
      public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
      }
    
      public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
      }
    }
    ```

    这里发生了一些微妙的事情。每个 Favorites 实例都由一个名为 favorites 的私有 `Map<Class<?>, Object>` 支持。你可能认为由于通配符类型是无界的，所以无法将任何内容放入此映射中，但事实恰恰相反。需要注意的是，通配符类型是嵌套的：通配符类型不是 Map 的类型，而是键的类型。这意味着每个键都可以有不同的参数化类型：一个可以是 `Class<String>`，下一个是 `Class<Integer>`，等等。这就是异构的原理。

    接下来要注意的是 favorites 的值类型仅仅是 Object。换句话说，Map 不保证键和值之间的类型关系，即每个值都是其键所表示的类型。实际上，Java 的类型系统还没有强大到足以表达这一点。但是我们知道这是事实，当需要检索一个 favorite 时，我们会利用它。

    putFavorite 的实现很简单：它只是将从给定 Class 对象到给定 Favorites 实例的放入 favorites 中。如前所述，这将丢弃键和值之间的「类型关联」；将无法确定值是键的实例。但这没关系，因为 getFavorites 方法可以重新建立这个关联。

    getFavorite 的实现比 putFavorite 的实现更复杂。首先，它从 favorites 中获取与给定 Class 对象对应的值。这是正确的对象引用返回，但它有错误的编译时类型：它是 Object（favorites 的值类型），我们需要返回一个 T。因此，getFavorite 的实现通过使用 Class 的 cast 方法，将对象引用类型动态转化为所代表的 Class 对象。

    总之，以集合的 API 为例的泛型在正常使用时将每个容器的类型参数限制为固定数量。你可以通过将类型参数放置在键上而不是容器上来绕过这个限制。你可以使用 Class 对象作为此类类型安全异构容器的键。以这种方式使用的 Class 对象称为类型标记。还可以使用自定义键类型。例如，可以使用 DatabaseRow 类型表示数据库行（容器），并使用泛型类型 `Column<T>` 作为它的键。



## 第六章 枚举和注解

34. 用枚举类型代替 int 常量：在枚举类型被添加到 JAVA 之前，表示枚举类型的一种常见模式是声明一组 int 的常量，这种技术称为 int 枚举模式，它有许多缺点。它没有提供任何类型安全性，并且几乎不具备表现力。如果你传递一个苹果给方法，希望得到一个橘子，使用 == 操作符比较苹果和橘子时编译器并不会提示错误，或更糟的情况：

    ```java
    // Tasty citrus flavored applesauce!
    int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
    ```

    使用 String 常量代替 int 常量。这种称为 String 枚举模式的变体甚至更不可取。虽然它确实为常量提供了可打印的字符串，但是它可能会导致不知情的用户将字符串常量硬编码到客户端代码中，而不是使用字段名。使用枚举可以解决上述问题。

    从表面上看，Java 枚举类型可能与其他语言（如 C、c++ 和 c#）的枚举类型类似，但不能只看表象。Java 的枚举类型是成熟的类，比其他语言中的枚举类型功能强大得多，在其他语言中的枚举本质上是 int 值。除了纠正 int 枚举的不足之外，枚举类型还允许添加任意方法和字段并实现任意接口。

    编写一个富枚举类型很容易，如上述的 Planet。要将数据与枚举常量关联，可声明实例字段并编写一个构造函数，该构造函数接受数据并将其存储在字段中。 枚举本质上是不可变的，因此所有字段都应该是 final。字段可以是公共的，但是最好将它们设置为私有并提供公共访问器。

    有一种更好的方法可以将不同的行为与每个枚举常量关联起来，这些方法称为特定常量方法实现：

    ```java
    // Enum type with constant-specific method implementations
    public enum Operation {
        PLUS {public double apply(double x, double y){return x + y;}},
        MINUS {public double apply(double x, double y){return x - y;}},
        TIMES {public double apply(double x, double y){return x * y;}},
        DIVIDE{public double apply(double x, double y){return x / y;}};
        public abstract double apply(double x, double y);
    }
    ```

35. 使用实例字段替代序数：所有枚举都有一个 ordinal 方法，该方法返回枚举类型中每个枚举常数的数值位置。

    ```java
    // Abuse of ordinal to derive an associated value - DON'T DO THIS
    public enum Ensemble {
        SOLO, DUET, TRIO, QUARTET, QUINTET,SEXTET, SEPTET, OCTET, NONET, DECTET;
    
        public int numberOfMusicians() { return ordinal() + 1; }
    }
    ```

    虽然这个枚举可以工作，但维护却是噩梦。如果常量被重新排序，numberOfMusicians 方法将被破坏。有一个简单的解决方案：不要从枚举的序数派生与枚举关联的值；而是将其存储在实例字段中：

    ```java
    public enum Ensemble {
        SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),NONET(9), DECTET(10),TRIPLE_QUARTET(12);
    
        private final int numberOfMusicians;
    
        Ensemble(int size) { this.numberOfMusicians = size; }
    
        public int numberOfMusicians() { return numberOfMusicians; }
    }
    ```

    枚举规范对 ordinal 方法的评价是这样的：「大多数程序员都不会去使用这个方法。它是为基于枚举的通用数据结构（如 EnumSet 和 EnumMap）而设计的」。除非你使用这个数据结构编写代码，否则最好完全避免使用这个方法。

36. 用 EnumSet 替代位字段：位字段模式如下：

    ```java
    // Bit field enumeration constants - OBSOLETE!
    public class Text {
        public static final int STYLE_BOLD = 1 << 0; // 1
        public static final int STYLE_ITALIC = 1 << 1; // 2
        public static final int STYLE_UNDERLINE = 1 << 2; // 4
        public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
        // Parameter is bitwise OR of zero or more STYLE_ constants
        public void applyStyles(int styles) { ... }
    }
    ```

    允许你使用位运算的 OR 操作将几个常量组合成一个 Set：

    ```java
    text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    ```

    位字段表示方式允许使用位运算高效地执行 Set 操作，如并集和交集。但是位字段具有 int 枚举常量所有缺点，甚至更多。当位字段被打印为数字时，它比简单的 int 枚举常量更难理解。没有一种简单的方法可以遍历由位字段表示的所有元素。

    当之前的示例修改为使用枚举和 EnumSet 而不是位字段时。它更短，更清晰，更安全：

    ```java
    // EnumSet - a modern replacement for bit fields
    public class Text {
        public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
        // Any Set could be passed in, but EnumSet is clearly best
        public void applyStyles(Set<Style> styles) { ... }
    }
    ```

    下面是将 EnumSet 实例传递给 applyStyles 方法的客户端代码：

    ```java
    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    ```

    EnumSet 类结合了位字段的简洁性和性能。EnumSet 的一个真正的缺点是，从 Java 9 开始，它不能创建不可变的 EnumSet。但是，可以用 `Collections.unmodifiableSet` 包装 EnumSet，实现不可变性，但简洁性和性能将受到影响。

37. 使用 EnumMap 替换序数索引：如果想要使用 Enum 里面的美居元素来对一组对象进行分组的话，请不要使用序数索引：

    ```java
    // Using ordinal() to index into an array - DON'T DO THIS!
    Set<Plant>[] plantsByLifeCycle =(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    ```

    这样带来的问题是不便于维护。Java 提供了一种简单的方式实现该目的，EnumMap：

    ```java
    // Using an EnumMap to associate data with an enum
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =new EnumMap<>(Plant.LifeCycle.class);
    ```

    这个程序比原来的版本更短，更清晰，更安全，速度也差不多。没有不安全的转换；不需要手动标记输出，因为 Map 的键是能转换为可打印字符串的枚举；在计算数组索引时不可能出错。

38. 使用接口模拟可扩展枚举：利用枚举类型可以实现任意接口这一事实，为 opcode 类型定义一个接口，并为接口的标准实现定义一个枚举：

    ```java
    // Emulated extensible enum using an interface
    public interface Operation {
        double apply(double x, double y);
    }
    
    public enum BasicOperation implements Operation {
        PLUS("+") {
            public double apply(double x, double y) { return x + y; }
        },
        MINUS("-") {
            public double apply(double x, double y) { return x - y; }
        },
        TIMES("*") {
            public double apply(double x, double y) { return x * y; }
        },
        DIVIDE("/") {
            public double apply(double x, double y) { return x / y; }
        };
    
        private final String symbol;
    
        BasicOperation(String symbol) {
            this.symbol = symbol;
        }
    
        @Override
        public String toString() {
            return symbol;
        }
    }
    ```

39. 注解优于命名模式：使用命名模式来标明某些程序元素需要工具或框架特殊处理的方式是很常见的，例如，在版本 4 之前，JUnit 测试框架要求其用户通过以字符 test 开头的名称来指定测试方法。命名模式有几个问题：

    1. 首先，排版错误会导致没有提示的失败
    2. 无法确保只在相应的程序元素上使用它们
    3. 它们没有提供将参数值与程序元素关联的好方法

    假设我们声明了一个 Test 注解，那么我们需要相应的工具来解析这些注解：

    ```java
    // Program to process marker annotations
    import java.lang.reflect.*;
    
    public class RunTests {
        public static void main(String[] args) throws Exception {
            int tests = 0;
            int passed = 0;
            Class<?> testClass = Class.forName(args[0]);
            for (Method m : testClass.getDeclaredMethods()) {
                if (m.isAnnotationPresent(Test.class)) {
                    tests++;
                    try {
                        m.invoke(null);
                        passed++;
                    } catch (InvocationTargetException wrappedExc) {
                        Throwable exc = wrappedExc.getCause();
                        System.out.println(m + " failed: " + exc);
                    } catch (Exception exc) {
                        System.out.println("Invalid @Test: " + m);
                    }
            }
        }
        System.out.printf("Passed: %d, Failed: %d%n",passed, tests - passed);
        }
    }
    ```

    使用注解很简洁，同时也防止了使用命名模式所带来的一系列的问题。

40. 坚持使用 @Override 注解：在每个方法声明上都使用 `@Override` 注解来覆盖超类型声明，那么编译器可以帮助你减少受到有害错误的影响，如错误将重写实现为重载。在具体类中，可以不对覆盖抽象方法声明的方法使用该注解。

41. 使用标记接口定义类型：标记接口是一种不包含任何方法声明的接口，它只是指定一个类，该类实现了具有某些属性的接口。例如 Serializable 接口。与标记注解相比，标记接口有两个优点。首先，标记接口定义的类型由标记类的实例实现；标记注解不会。标记接口相对于标记注解的另一个优点是可以更精确地定位它们。相对于标记接口，标记注解的主要优势是它们可以是其他注解功能的一部分。



## 第七章 Lambda表达式和流

42. lambda 表达式优于匿名类：在历史上，带有单个抽象方法的接口被用作函数类型。它们的实例（称为函数对象）表示函数或操作。自从 JDK 1.1 在 1997 年发布以来，创建函数对象的主要方法就是匿名类：

    ```java
    // Anonymous class instance as a function object - obsolete!
    Collections.sort(words, new Comparator<String>() {
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
    });
    ```

    在 Java 8 中官方化了一个概念，即具有单个抽象方法的接口是特殊的，应该得到特殊处理。这些接口现在被称为函数式接口，允许使用 lambda 表达式创建这些接口的实例：

    ```java
    // Lambda expression as function object (replaces anonymous class)
    Collections.sort(words,(s1, s2) -> Integer.compare(s1.length(), s2.length()));
    ```

    一般来说，省略lambda中参数的类型，除非编译器不能自动推断出来。另外，在lambda表达式中this关键字指向的是外部的类的实例，但是匿名类指的是匿名类自己。

43. 方法引用优于 lambda 表达式：lambda 表达式与匿名类相比，主要优势是更简洁。Java 提供了一种方法来生成比 lambda 表达式更简洁的函数对象：方法引用。

    ```java
    map.merge(key, 1, Integer::sum);
    ```

    而是用lambda代码如下：

    ```java
    map.merge(key, 1, (count, incr) -> count + incr);
    ```

    方法引用通常为 lambda 表达式提供了一种更简洁的选择。如果方法引用更短、更清晰，则使用它们；如果没有，仍然使用 lambda 表达式。

44. 优先使用标准函数式接口：现在 Java 已经有了 lambda 表达式，编写 API 的最佳实践已经发生了很大的变化。java.util.function 包提供了大量的标准函数接口供你使用。如果一个标准的函数式接口可以完成这项工作，那么你通常应该优先使用它，而不是使用专门构建的函数式接口。六个基本的函数式接口总结如下：

    | Interface        | Function Signature    | Example               |
    | ---------------- | --------------------- | --------------------- |
    | `UnaryOperator`  | `T apply(T t)`        | `String::toLowerCase` |
    | `BinaryOperator` | `T apply(T t1, T t2)` | `BigInteger::add`     |
    | `Predicate`      | `boolean test(T t)`   | `Collection::isEmpty` |
    | `Function`       | `R apply(T t)`        | `Arrays::asList`      |
    | `Supplier`       | `T get()`             | `Instant::now`        |
    | `Consumer`       | `void accept(T t)`    | `System.out::println` |

    还有 6 个基本接口的 3 个变体，用于操作基本类型 int、long 和 double。Function 接口还有 9 个额外的变体，在结果类型为基本数据类型时使用。

45. 明智地使用流：在 Java 8 中添加了流 API，以简化序列或并行执行批量操作的任务。这个 API 提供了两个关键的抽象：流（表示有限或无限的数据元素序列）和流管道（表示对这些元素的多阶段计算）。流管道的计算是惰性的：直到调用 Terminal 操作时才开始计算，并且对完成 Terminal 操作不需要的数据元素永远不会计算。这种惰性的求值机制使得处理无限流成为可能。流 API 非常通用，实际上任何计算都可以使用流来执行，但这并不意味着你就应该这样做。如果使用得当，流可以使程序更短、更清晰；如果使用不当，它们会使程序难以读取和维护。由于 Java 不支持基本字符流：

    ```java
    "Hello world!".chars().forEach(System.out::print);
    ```

    你可能希望它打印 Hello world!，但如果运行它，你会发现它打印 721011081081113211911111410810033。这是因为 "Hello world!".chars() 返回的流元素不是 char 值，而是 int 值，因此调用了 print 的 int 重载。强制转换可以解决这个问题：

    ```java
    "Hello world!".chars().forEach(x -> System.out.print((char) x));
    ```

    有些事情你可以对代码块做，而你不能对函数对象（通常是 lambda 表达式或方法引用）做

    + 从代码块中，可以读取或修改作用域中的任何局部变量；在 lambda 表达式中，只能读取 final 或有效的 final 变量，不能修改任何局部变量。
    + 从代码块中，可以从封闭方法返回、中断或继续封闭循环，或抛出声明要抛出的任何已检查异常；在 lambda 表达式中，你不能做这些事情。

    相反，流使做一些事情变得非常容易：

    + 元素序列的一致变换
    + 过滤元素序列
    + 使用单个操作组合元素序列
    + 将元素序列累积到一个集合中，可能是按某个公共属性对它们进行分组
    + 在元素序列中搜索满足某些条件的元素

46. 在流中使用无副作用的函数：你可能会看到如下使用流的代码片段，它用于构建文本文件中单词的频率表：

    ```java
    // Uses the streams API but not the paradigm--Don't do this!
    Map<String, Long> freq = new HashMap<>();
    try (Stream<String> words = new Scanner(file).tokens()) {
        words.forEach(word -> {
            freq.merge(word.toLowerCase(), 1L, Long::sum);
        });
    }
    ```

    简单地说，它根本不是流代码，而是伪装成流代码的迭代代码。它没有从流 API 中获得任何好处，而且它（稍微）比相应的迭代代码更长、更难于阅读和更难以维护。这个问题源于这样一个事实：这段代码在一个 Terminal 操作中（forEach）执行它的所有工作，使用一个会改变外部状态的 lambda 表达式（频率表）。改进后的代码：

    ```java
    // Proper use of streams to initialize a frequency table
    Map<String, Long> freq;
    try (Stream<String> words = new Scanner(file).tokens()) {
        freq = words.collect(groupingBy(String::toLowerCase, counting()));
    }
    ```

    这个代码片段与前面的代码片段做了相同的事情，但是正确地使用了流 API。它更短更清晰。

    将流的元素收集到一个真正的 Collection 中的 collector 非常简单。这样的 collector 有三种：`toList()`、`toSet()` 和 `toCollection(collectionFactory)`。它们分别返回 List、Set 和程序员指定的集合类型。

    ```java
    // Pipeline to get a top-ten list of words from a frequency table
    List<String> topTen = freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
    ```

    另外还有 groupingBy 和 join。

47. 优先选择 Collection 而不是流作为返回类型：在编写返回元素序列的方法时，有些用户可能希望将它们作为流处理，而有些用户可能希望对它们进行迭代。如果可以返回集合，那么就这样做。如果你已经在一个集合中拥有了元素，或者序列中的元素数量足够小，可以创建一个新的元素，那么返回一个标准集合，例如 ArrayList 。否则，请考虑像对 power 集那样实现自定义集合。如果返回集合不可行，则返回流或 iterable，以看起来更自然的方式返回。

48. 谨慎使用并行流：在主流语言中，Java 一直走在提供简化并发编程任务工具的前列。当 Java 在 1996 年发布时，它内置了对线程的支持，支持同步和 wait/notify。Java 5 引入了 java.util.concurrent。具有并发集合和执行器框架的并发库。Java 7 引入了 fork-join 包，这是一个用于并行分解的高性能框架。Java 8 引入了流，它可以通过对 parallel 方法的一次调用来并行化。

    并行性带来的性能提升在 ArrayList、HashMap、HashSet 和 ConcurrentHashMap 实例上的流效果最好；int 数组和 long 数组也在其中。 这些数据结构的共同之处在于，它们都可以被精确且廉价地分割成任意大小的子程序，这使得在并行线程之间划分工作变得很容易。

    并行化流不仅会导致糟糕的性能，包括活动失败；它会导致不正确的结果和不可预知的行为（安全故障）。 如果管道使用映射器、过滤器和其他程序员提供的函数对象，而这些对象没有遵守其规范，则并行化管道可能导致安全故障。流规范对这些功能对象提出了严格的要求。例如，传递给流的 reduce 操作的累加器和组合器函数必须是关联的、不干扰的和无状态的。



## 第八章 方法

49. 检查参数的有效性：大多数方法和构造函数都对传递给它们的参数值有一些限制。例如，索引值必须是非负的，对象引用必须是非空的。如果一个无效的参数值被传递给一个方法，如果该方法在执行之前会检查它的参数，那么这个过程将迅速失败，并引发适当的异常。如果方法未能检查其参数，可能会发生以下几件事。该方法可能会在处理过程中出现令人困惑的异常而失败。更糟的是，该方法可以正常返回，但会静默计算错误的结果。在 Java 7 中添加的 Objects.requireNonNull 方法非常灵活和方便，因此不再需要手动执行空检查。在 Java 9 中，范围检查功能被添加到 java.util.Objects 中。这个功能由三个方法组成：checkFromIndexSize、checkFromToIndex 和 checkIndex。非公共方法可以使用断言检查它们的参数。

    在执行方法的计算任务之前，应该显式地检查方法的参数，这条规则也有例外。一个重要的例外是有效性检查成本较高或不切实际，或者检查是在计算过程中隐式执行了。例如，考虑一个为对象 List 排序的方法，比如 Collections.sort(List)。List 中的所有对象必须相互比较。在对 List 排序的过程中，List 中的每个对象都会与列表中的其他对象进行比较。如果对象不能相互比较，将抛出 ClassCastException，这正是 sort 方法应该做的。因此，没有必要预先检查列表中的元素是否具有可比性。

50. 在需要时制作防御性副本：即使使用一种安全的语言，如果你不付出一些努力，也无法与其他类隔离。你必须进行防御性的设计，并假定你的类的客户端会尽最大努力破坏它的不变量。 随着人们越来越多地尝试破坏系统的安全性，这个观点越来越正确。考虑这样的一个类：

    ```java
    // Broken "immutable" time period class
    public final class Period {
        private final Date start;
        private final Date end;
    
        public Period(Date start, Date end) {
            if (start.compareTo(end) > 0)
                throw new IllegalArgumentException(start + " after " + end);
            this.start = start;
            this.end = end;
        }
    
        public Date start() {
            return start;
        }
    
        public Date end() {
            return end;
        }
        ... // Remainder omitted
    }
    ```

    乍一看，这个类似乎是不可变的，并且要求一个时间段的开始时间不能在结束时间之后。然而，利用 Date 是可变的这一事实很容易绕过这个约束：

    ```java
    // Attack the internals of a Period instance
    Date start = new Date();
    Date end = new Date();
    Period p = new Period(start, end);
    end.setYear(78); // Modifies internals of p!
    ```

    为了防止这样的攻击，可以选择制作防御性副本：

    ```java
    // Repaired constructor - makes defensive copies of parameters
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(this.start + " after " + this.end);
    }
    
    // Repaired accessors - make defensive copies of internal fields
    public Date start() {
        return new Date(start.getTime());
    }
    
    public Date end() {
        return new Date(end.getTime());
    }
    ```

    防御性复制可能会带来性能损失，最好的方法是使用不可变类，比如Instant（或 Local-DateTime 或 ZonedDateTime）来代替 Date，Date 已过时，不应在新代码中使用。

51. 仔细设计方法签名：

    + 仔细选择方法名称。
    + 不要提供过于便利的方法
    + 避免长参数列表
    + 对于参数类型，优先选择接口而不是类
    + 双元素枚举类型优于 boolean 参数

52. 明智地使用重载：考虑下面的代码：

    ```java
    // Broken! - What does this program print?
    public class CollectionClassifier {
        public static String classify(Set<?> s) {
            return "Set";
        }
    
        public static String classify(List<?> lst) {
            return "List";
        }
    
        public static String classify(Collection<?> c) {
            return "Unknown Collection";
        }
    
        public static void main(String[] args) {
            Collection<?>[] collections = {
                new HashSet<String>(),new ArrayList<BigInteger>(),new HashMap<String, String>().values()
            };
            for (Collection<?> c : collections)
                System.out.println(classify(c));
        }
    ```

    你可能期望这个程序打印 Set，然后是 List 和 Unknown Collection，但是它没有这样做。它打印 Unknown Collection 三次。为什么会这样？因为 classify 方法被重载，并且 在编译时就决定了要调用哪个重载。

    这个程序的行为违反常规，因为重载方法的选择是静态的，而覆盖方法的选择是动态的。 在运行时根据调用方法的对象的运行时类型选择覆盖方法的正确版本。

    ```java
    class Wine {
        String name() { return "wine"; }
    }
    
    class SparklingWine extends Wine {
        @Override
        String name() { return "sparkling wine"; }
    }
    
    class Champagne extends SparklingWine {
        @Override
        String name() { return "champagne"; }
    }
    
    public class Overriding {
        public static void main(String[] args) {
            List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champagne());
        for (Wine wine : wineList)
            System.out.println(wine.name());
        }
    }
    ```

    正如你所期望的，这个程序打印出 wine、sparkling 和 champagne，即使实例的编译时类型是循环每次迭代中的 wine。

    应该避免混淆重载的用法。安全、保守的策略是永远不导出具有相同数量参数的两个重载。这些限制并不十分繁琐，因为你总是可以为方法提供不同的名称，而不是重载它们。

53. 明智地使用可变参数：可变参数方法接受指定类型的零个或多个参数。可变参数方法首先创建一个数组，其大小是在调用点上传递的参数数量，然后将参数值放入数组，最后将数组传递给方法。在性能关键的情况下使用可变参数时要小心。每次调用可变参数方法都会导致数组分配和初始化。如果你已经从经验上确定你负担不起这个成本，但是你仍需要可变参数的灵活性，那么有一种模式可以让你鱼与熊掌兼得。假设你已经确定对方法 95% 的调用只需要三个或更少的参数。可以声明该方法的 5 个重载，每个重载 0 到 3 个普通参数，当参数数量超过 3 个时引入可变参数：

    ```java
    public void foo() { }
    public void foo(int a1) { }
    public void foo(int a1, int a2) { }
    public void foo(int a1, int a2, int a3) { }
    public void foo(int a1, int a2, int a3, int... rest) { }
    ```

54. 返回空集合或数组，而不是 null：在几乎每次使用返回 null 来代替空集合或数组的方法时，都需要使用这种权宜之计。它很容易出错，因为编写客户端的程序员可能忘记编写特殊情况的代码来处理 null 返回。这样的错误可能会被忽略多年，因为这样的方法通常返回一个或多个对象。此外，在空容器中返回 null 会使返回容器的方法的实现复杂化。数组的情况与集合的情况相同。永远不要返回 null，而应该返回零长度的数组。永远不要用 null 来代替空数组或集合。它使你的 API 更难以使用，更容易出错，并且没有性能优势。

55. 明智地返回 Optional：在 Java 8 之前，在编写在某些情况下无法返回值的方法时，可以采用两种方法。要么抛出异常，要么返回 null（假设返回类型是对象引用类型）。这两种方法都不完美。应该为异常条件保留异常，并且抛出异常代价高昂，因为在创建异常时捕获整个堆栈跟踪。返回 null 没有这些缺点，但是它有自己的缺点。如果方法返回 null，客户端必须包含特殊情况代码来处理 null 返回的可能性，除非程序员能够证明 null 返回是不可能的。如果客户端忽略检查 null 返回并将 null 返回值存储在某个数据结构中，那么 NullPointerException 可能会在将来的某个时间，在代码中的某个与该问题无关的位置产生。

    在 Java 8 中，还有第三种方法来编写可能无法返回值的方法。`Optional<T>` 类表示一个不可变的容器，它可以包含一个非空的 T 引用，也可以什么都不包含。不包含任何内容的 Optional 被称为空。一个值被认为存在于一个非空的 Optional 中。Optional 的本质上是一个不可变的集合，它最多可以容纳一个元素。具备 Optional 返回值的方法比抛出异常的方法更灵活、更容易使用，并且比返回 null 的方法更不容易出错。

    如果你发现自己编写的方法不能总是返回确定值，并且你认为该方法的用户在每次调用时应该考虑这种可能性，那么你可能应该让方法返回一个 Optional。但是，你应该意识到，返回 Optional 会带来实际的性能后果；对于性能关键的方法，最好返回 null 或抛出异常。最后，除了作为返回值之外，你几乎不应该以任何其他方式使用 Optional。

56. 为所有公开的 API 元素编写文档注释：如果 API 要可用，就必须对其进行文档化。传统上，API 文档是手工生成的，保持与代码的同步是一件苦差事。Java 编程环境使用 Javadoc 实用程序简化了这一任务。Javadoc 使用特殊格式的文档注释（通常称为文档注释）从源代码自动生成 API 文档。

    要正确地编写 API 文档，必须在每个公开的类、接口、构造函数、方法和字段声明之前加上文档注释。 如果一个类是可序列化的，还应该记录它的序列化形式。方法的文档注释应该简洁地描述方法与其客户端之间的约定。



## 第九章 通用程序设计

57. 将局部变量的作用域最小化：通过最小化局部变量的范围，可以提高代码的可读性和可维护性，并降低出错的可能性。在做循环操作的时候，尽量使用for而不是while。下面是使用局部变量的方法技巧：

    + 将局部变量的作用域最小化，最具说服力的方式就是在第一次使用它的地方声明。
    + 每个局部变量声明都应该包含一个初始化表达式。
    + 最小化局部变量范围的最后一种技术是保持方法小而集中。

58. for-each 循环优于传统的 for 循环：通常我们使用迭代器来遍历集合，使用数组索引遍历数组，但是他们的缺点就是需要自己维护迭代器和索引变量，最好的方法是使用for-each循环。但是，下列情况不适合for-each循环：

    + 破坏性过滤，如果需要遍历一个集合并删除选定元素，则需要使用显式的迭代器，以便调用其 remove 方法。
    + 转换，如果需要遍历一个 List 或数组并替换其中部分或全部元素的值，那么需要 List 迭代器或数组索引来替换元素的值。
    + 并行迭代，如果需要并行遍历多个集合，那么需要显式地控制迭代器或索引变量，以便所有迭代器或索引变量都可以同步执行。

59. 了解并使用库：假设你想要生成 0 到某个上界之间的随机整数：

    ```java
    // Common but deeply flawed!
    static Random rnd = new Random();
    static int random(int n) {
        return Math.abs(rnd.nextInt()) % n;
    }
    ```

    有三个缺点：首先，如果 n 是小的平方数，随机数序列会在相当短的时间内重复。第二个缺陷是，如果 n 不是 2 的幂，那么平均而言，一些数字将比其他数字更频繁地返回。第三个缺陷是，在极少数情况下会返回超出指定范围的数字，这是灾难性的结果。

    从 Java 7 开始，就不应该再使用 Random。在大多数情况下，选择的随机数生成器现在是 ThreadLocalRandom。使用这些库能生成更高的随机数，同时，你不必浪费时间为那些与你的工作无关的问题编写专门的解决方案。

60. 若需要精确答案就应避免使用 float 和 double 类型：float 和 double 类型特别不适合进行货币计算，因为不可能将 0.1（或 10 的任意负次幂）精确地表示为 float 或 double。为了得到准确值，应该使用BigDecimal或者是long，int类型进行计算操作。

61. 基本数据类型优于包装类：将 == 操作符应用于包装类型几乎都是错误的，而在使用基本类型的时候则是正确的。在操作中混合使用基本类型和包装类型时，包装类型就会自动拆箱，如果包装类是null，可能就会导致NullPointerException。

    什么时候该使用包装类型？第一个是作为集合中的元素、键和值。在参数化类型和方法中，必须使用包装类型作为类型参数，因为 Java 不允许使用基本类型。最后，在进行反射方法调用时，必须使用包装类型。

    总之，只要有选择，就应该优先使用基本类型，而不是包装类型。基本类型更简单、更快。当你的程序使用 == 操作符比较两个包装类型时，它会执行标识比较，这几乎肯定不是你想要的。

62. 其他类型更合适时应避免使用字符串：字符串被设计用来表示文本，它们在这方面做得很好。下面是一些字符串不推荐使用的方式：

    + 字符串是枚举类型的糟糕替代品
    + 字符串是聚合类型的糟糕替代品

    总之，当存在或可以编写更好的数据类型时，应避免将字符串用来表示对象。如果使用不当，字符串比其他类型更麻烦、灵活性更差、速度更慢、更容易出错。字符串经常被误用的类型包括基本类型、枚举和聚合类型。

63. 当心字符串连接引起的性能问题：使用字符串串联运算符重复串联 n 个字符串需要 n 的平方级时间。这是字符串不可变这一事实导致的结果。当连接两个字符串时，将复制这两个字符串的内容。要获得能接受的性能，请使用 StringBuilder 代替 String。

64. 通过接口引用对象：应该优先使用接口而不是类来引用对象。如果存在合适的接口类型，那么应该使用接口类型声明参数、返回值、变量和字段。惟一真正需要引用对象的类的时候是使用构造函数创建它的时候。如果你养成了使用接口作为类型的习惯，那么你的程序将更加灵活。如果没有合适的接口存在，那么用类引用对象是完全合适的。

65. 接口优于反射：核心反射机制 java.lang.reflect 提供对任意类的编程访问。给定一个 Class 对象，你可以获得 Constructor、Method 和 Field 实例，分别代表了该 Class 实例所表示的类的构造器、方法和字段。使用反射是有代价的：

    + 你失去了编译时类型检查的所有好处，包括异常检查
    + 执行反射访问所需的代码既笨拙又冗长
    + 性能降低，反射方法调用比普通方法调用慢得多

    反射是一种功能强大的工具，对于某些复杂的系统编程任务是必需的，但是它有很多缺点。如果编写的程序必须在编译时处理未知的类，则应该尽可能只使用反射实例化对象，并使用在编译时已知的接口或超类访问对象。

66. 明智地使用本地方法：Java 本地接口（JNI）允许 Java 程序调用本地方法，这些方法是用 C 或 C++ 等本地编程语言编写的。为了提高性能，很少建议使用本地方法。使用本地方法有严重的缺点。由于本地语言不安全，使用本地方法的应用程序不再能免受内存毁坏错误的影响同时很难进行调试。

67. 明智地进行优化：不要过早地进行优化。为了获得良好的性能而改变 API 是一个非常糟糕的想法。同时，在每次尝试优化之前和之后测量性能。

68. 遵守被广泛认可的命名约定：

    | Identifier Type    | Example                                              |
    | ------------------ | ---------------------------------------------------- |
    | Package or module  | `org.junit.jupiter.api`, `com.google.common.collect` |
    | Class or Interface | Stream, FutureTask, LinkedHashMap, HttpClient        |
    | Method or Field    | remove, groupingBy, getCrc                           |
    | Constant Field     | MIN_VALUE, NEGATIVE_INFINITY                         |
    | Local Variable     | i, denom, houseNum                                   |
    | Type Parameter     | T, E, K, V, X, R, U, V, T1, T2                       |



## 第十章 异常

69. 仅在确有异常条件下使用异常：异常只适用于确有异常的情况；它们不应该用于一般的控制流程。下列代码不应该使用：

    ```java
    // Horrible abuse of exceptions. Don't ever do this!
    try {
        int i = 0;
        while(true)
            range[i++].climb();
        }
        catch (ArrayIndexOutOfBoundsException e) {
    }
    ```

    我们完全可以使用for-each循环实现。

70. 对可恢复情况使用 checked 异常，对编程错误使用运行时异常：Java 提供了三种可抛出项：checked 异常、运行时异常和错误。决定是使用 checked 异常还是 unchecked 异常的基本规则是：使用 checked 异常的情况是为了合理地期望调用者能够从中恢复。有两种 unchecked 的可抛出项：运行时异常和错误。它们在行为上是一样的：都是可抛出的，通常不需要也不应该被捕获。使用运行时异常来指示编程错误。 绝大多数运行时异常都表示操作违反了先决条件。

71. 避免不必要地使用 checked 异常：在 API 中过度使用 checked 异常会变得不那么令人愉快。如果一个方法抛出 checked 异常，调用它的代码必须在一个或多个 catch 块中处理它们；或者通过声明抛出，让它们向外传播。无论哪种方式，它都给 API 的用户带来了负担。消除 checked 异常的最简单方法是返回所需结果类型的 Optional 对象（Item-55）。该方法只返回一个空的 Optional 对象，而不是抛出一个 checked 异常。

    总之，如果谨慎使用，checked 异常可以提高程序的可靠性；当过度使用时，它们会使 API 难以使用。如果调用者不应从失败中恢复，则抛出 unchecked 异常。如果恢复是可能的，并且你希望强制调用者处理异常条件，那么首先考虑返回一个 Optional 对象。只有当在失败的情况下，提供的信息不充分时，你才应该抛出一个 checked 异常。

72. 鼓励复用标准异常：使你的 API 更容易学习和使用，因为它符合程序员已经熟悉的既定约定。最常见的可复用异常：

    | Exception                       | Occasion for Use                                             |
    | ------------------------------- | ------------------------------------------------------------ |
    | IllegalArgumentException        | Non-null parameter value is inappropriate（非空参数值不合适） |
    | IllegalStateException           | Object state is inappropriate for method invocation（对象状态不适用于方法调用） |
    | NullPointerException            | Parameter value is null where prohibited（禁止参数为空时仍传入 null） |
    | IndexOutOfBoundsException       | Index parameter value is out of range（索引参数值超出范围）  |
    | ConcurrentModificationException | Concurrent modification of an object has been detected where it is prohibited（在禁止并发修改对象的地方检测到该动作） |
    | UnsupportedOperationException   | Object does not support method（对象不支持该方法调用）       |

    另外，不要直接复用 Exception、RuntimeException、Throwable 或 Error。应当将这些类视为抽象类。你不能对这些异常进行可靠的测试，因为它们是方法可能抛出的异常的超类。

73. 抛出能用抽象解释的异常：当一个方法抛出一个与它所执行的任务没有明显关联的异常时，这是令人不安的。这种情况经常发生在由方法传播自低层抽象抛出的异常。为了避免这个问题，高层应该捕获低层异常，并确保抛出的异常可以用高层抽象解释。 这个习惯用法称为异常转换：

    ```java
    // Exception Translation
    try {
        ... // Use lower-level abstraction to do our bidding
    } catch (LowerLevelException e) {
        throw new HigherLevelException(...);
    }
    ```

    虽然异常转换优于底层异常的盲目传播，但它不应该被过度使用。在可能的情况下，处理低层异常的最佳方法是确保低层方法避免异常。

74. 为每个方法记录会抛出的所有异常：始终单独声明 checked 异常，并使用 Javadoc 的 @throw 标记精确记录每次抛出异常的条件。使用 Javadoc 的 @throw 标记记录方法会抛出的每个异常，但是不要对 unchecked 异常使用 throws 关键字。如果一个类中的许多方法都因为相同的原因抛出异常，你可以在类的文档注释中记录异常， 而不是为每个方法单独记录异常。

75. 异常详细消息中应包含捕获失败的信息：当程序由于未捕获异常而失败时，系统可以自动打印出异常的堆栈跟踪。堆栈跟踪包含异常的字符串表示，这是调用其 toString 方法的结果。这通常包括异常的类名及其详细信息。要捕获失败，异常的详细消息应该包含导致异常的所有参数和字段的值。因为许多人在诊断和修复软件问题的过程中可能会看到堆栈跟踪，所以不应包含密码、加密密钥等详细信息。

76. 尽力保证故障原子性：在对象抛出异常之后，通常希望对象仍然处于定义良好的可用状态，即使在执行操作时发生了故障。对于 checked 异常尤其如此，调用者希望从异常中恢复。一般来说，失败的方法调用应该使对象处于调用之前的状态。 具有此属性的方法称为具备故障原子性。有几种方式可以达到这种效果：

    + 最简单的方法是设计不可变对象
    + 对计算进行排序，以便可能发生故障的部分都先于修改对象的部分发生
    + 以对象的临时副本执行操作，并在操作完成后用临时副本替换对象的内容
    + 编写恢复代码，拦截在操作过程中发生的故障，并使对象回滚到操作开始之前的状态

77. 不要忽略异常：如果在方法调用的周围加上一条 try 语句，其 catch 块为空，可以很容易忽略异常，空 catch 块违背了异常的目的，它的存在是为了强制你处理异常情况。如果你选择忽略异常，catch 块应该包含一条注释，解释为什么这样做是合适的，并且应该将变量命名为 ignored。



## 第十一章 并发

78. 对共享可变数据的同步访问：synchronized 关键字确保一次只有一个线程可以执行一个方法或块。没有同步，一个线程所做的的更改可能对其他线程不可见。同步不仅阻止线程察觉到处于不一致状态的对象，而且确保每个进入同步方法或块的线程都能察觉由同一把锁保护的所有已修改的效果。

    ```java
    // Broken! - How long would you expect this program to run?
    public class StopThread {
        private static boolean stopRequested;
    
        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
            });
    
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
        }
    }
    ```

    在缺乏同步的情况下，无法保证后台线程何时（如果有的话）看到主线程所做的 stopRequested 值的更改。在缺乏同步的情况下，虚拟机可以很好地转换这段代码：

    ```java
    while (!stopRequested)
        i++;
    into this code:
    if (!stopRequested)
        while (true)
            i++;
    ```

    这种优化称为提升，这正是 OpenJDK 服务器 VM 所做的。结果是活性失败：程序无法取得进展。

    注意，写方法（requestStop）和读方法（stopRequested）都是同步的。仅同步写方法是不够的！除非读和写操作都同步，否则不能保证同步工作。虽然 volatile 修饰符不执行互斥，但它保证任何读取字段的线程都会看到最近写入的值。

    ```java
    // Broken - requires synchronization!
    private static volatile int nextSerialNumber = 0;
    
    public static int generateSerialNumber() {
        return nextSerialNumber++;
    }
    ```

    问题在于增量运算符 (++) 不是原子性的。它对 nextSerialNumber 字段执行两个操作：首先读取值，然后返回一个新值，旧值再加 1。如果第二个线程在读取旧值和写入新值之间读取字段，则第二个线程将看到与第一个线程相同的值，并返回相同的序列号。可以将 synchronized 添加到方法声明中，另外，也使用 AtomicLong 类，它是 `java.util.concurrent.atomic` 的一部分。

    总之，当多个线程共享可变数据时，每个读取或写入数据的线程都必须执行同步。 在缺乏同步的情况下，不能保证一个线程的更改对另一个线程可见。

79. 避免过度同步：过度的同步可能导致性能下降、死锁甚至不确定行为。作为规则，你应该在同步区域内做尽可能少的工作。获取锁，检查共享数据，根据需要进行转换，然后删除锁。如果你必须执行一些耗时的活动，请设法将其移出同步区域。

80. Executor、task、流优于直接使用线程：java.util.concurrent 已经添加到 Java 中。这个包有一个 Executor 框架，它是一个灵活的基于接口的任务执行工具。对于小程序或负载较轻的服务器，Executors.newCachedThreadPool 通常是一个不错的选择，因为它不需要配置，而且通常「做正确的事情」。但是对于负载沉重的生产服务器来说，缓存的线程池不是一个好的选择！在缓存的线程池中，提交的任务不会排队，而是立即传递给线程执行。如果没有可用的线程，则创建一个新的线程。如果服务器负载过重，所有 CPU 都被充分利用，并且有更多的任务到达，就会创建更多的线程，这只会使情况变得更糟。因此，在负载沉重的生产服务器中，最好使用 Executors.newFixedThreadPool，它为你提供一个线程数量固定的池，或者直接使用 ThreadPoolExecutor 类来实现最大限度的控制。

81. 并发实用工具优于 wait 和 notify：考虑到正确使用 wait 和 notify 的困难，你应该使用更高级别的并发实用工具。java.util.concurrent 中级别较高的实用工具可分为三类：Executor 框架，Item-80 简要介绍了该框架；并发集合；同步器。本条目简要介绍并发集合和同步器。

    并发集合是标准集合接口，如 List、Queue 和 Map 的高性能并发实现。为了提供高并发性，这些实现在内部管理它们自己的同步。一些集合接口使用阻塞操作进行了扩展，这些操作将等待（或阻塞）成功执行。例如，BlockingQueue 扩展了 Queue 并添加了几个方法，包括 take，它从队列中删除并返回首个元素，如果队列为空，则等待。

    同步器是允许线程彼此等待的对象，允许它们协调各自的活动。最常用的同步器是 CountDownLatch 和 Semaphore。较不常用的是 CyclicBarrier 和 Exchanger。最强大的同步器是 Phaser。

    始终使用 wait 习惯用法，即循环来调用 wait 方法；永远不要在循环之外调用它。 循环用于在等待之前和之后测试条件。

82. 文档应包含线程安全属性：类的线程安全的描述通常属于该类的文档注释，但是具有特殊线程安全属性的方法应该在它们自己的文档注释中描述这些属性。没有必要记录枚举类型的不变性。

83. 明智地使用延迟初始化：延迟初始化是延迟字段的初始化，直到需要它的值。与大多数优化一样，延迟初始化的最佳建议是「除非需要，否则不要这样做」。在大多数情况下，常规初始化优于延迟初始化。

84. 不要依赖线程调度器：任何依赖线程调度器来保证正确性或性能的程序都可能是不可移植的。如果线程没有做有用的工作，它们就不应该运行。线程优先级可以少量地用于提高已经工作的程序的服务质量，但绝不应该用于「修复」几乎不能工作的程序。



## 第十二章 序列化

85. Java 序列化的替代方案：序列化的一个根本问题是它的可攻击范围太大，且难以保护，而且问题还在不断增多：通过调用 ObjectInputStream 上的 readObject 方法反序列化对象图。这个方法本质上是一个神奇的构造函数，可以用来实例化类路径上几乎任何类型的对象，只要该类型实现 Serializable 接口。避免序列化利用的最好方法是永远不要反序列化任何东西。永远不要反序列化不可信的数据。

    序列化是危险的，应该避免。如果你从头开始设计一个系统，可以使用跨平台的结构化数据，如 JSON 或 protobuf。不要反序列化不可信的数据。如果必须这样做，请使用对象反序列化过滤，但要注意，它不能保证阻止所有攻击。避免编写可序列化的类。

86. 非常谨慎地实现 Serializable：使类的实例可序列化非常简单，只需实现 Serializable 接口即可。因为这很容易做到，所以有一个普遍的误解，认为序列化只需要程序员付出很少的努力。而事实上要复杂得多。虽然使类可序列化的即时代价可以忽略不计，但长期代价通常是巨大的：

    + 一旦类的实现被发布，它就会降低更改该类实现的灵活性
    + 增加了出现 bug 和安全漏洞的可能性
    + 增加了与发布类的新版本相关的测试负担

    为继承而设计的类（Item-19）很少情况适合实现 Serializable 接口，接口也很少情况适合扩展它。另外，内部类不应该实现 Serializable。

87. 考虑使用自定义序列化形式：当对象的物理表示与其逻辑数据内容有很大差异时，使用默认的序列化形式有四个缺点：

    + 它将导出的 API 永久地绑定到当前的内部实现
    + 它会占用过多的空间
    + 它会消耗过多的时间
    + 它可能导致堆栈溢出

    无论你是否使用默认的序列化形式，必须对对象序列化强制执行任何同步操作，就像对读取对象的整个状态的任何其他方法强制执行的那样。无论选择哪种序列化形式，都要在编写的每个可序列化类中声明显式的序列版本 UID。 这消除了序列版本 UID 成为不兼容性的潜在来源。这么做还能获得一个小的性能优势。如果没有提供序列版本 UID，则需要执行高开销的计算在运行时生成一个 UID。不要更改序列版本 UID，除非你想破坏与现有序列化所有实例的兼容性。

88. 防御性地编写 readObject 方法：当对象被反序列化时，对任何客户端不能拥有的对象引用的字段进行防御性地复制至关重要。 因此，对于每个可序列化的不可变类，如果它包含了私有的可变组件，那么在它的 readObjec 方法中，必须要对这些组件进行防御性地复制：

    ```java
    // readObject method with defensive copying and validity checking
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        // Defensively copy our mutable components
        start = new Date(start.getTime());
        end = new Date(end.getTime());
        // Check that our invariants are satisfied
        if (start.compareTo(end) > 0)
            throw new InvalidObjectException(start +" after "+ end);
    }
    ```

    下面是编写 readObject 方法的指导原则：

    + 对象引用字段必须保持私有的的类，应防御性地复制该字段中的每个对象
    + 检查任何不变量，如果检查失败，则抛出 InvalidObjectException
    + 如果必须在反序列化后验证整个对象图，那么使用 ObjectInputValidation 接口
    + 不要直接或间接地调用类中任何可被覆盖的方法

89. 对于实例控制，枚举类型优于 readResolve：在可能的情况下，使用枚举类型强制实例控制不变量。如果这是不可能的，并且你需要一个既可序列化又实例控制的类，那么你必须提供一个 readResolve 方法，并确保该类的所有实例字段都是基本类型，或使用 transient 修饰。

90. 考虑以序列化代理代替序列化实例：实现 Serializable 接口的决定增加了出现 bug 和安全问题的可能性，因为它允许使用一种超语言机制来创建实例，而不是使用普通的构造函数。然而，有一种技术可以大大降低这些风险。这种技术称为序列化代理模式。序列化代理模式相当简单。首先，设计一个私有静态嵌套类，它简洁地表示外围类实例的逻辑状态。这个嵌套类称为外围类的序列化代理。它应该有一个构造函数，其参数类型是外围类。这个构造函数只是从它的参数复制数据：它不需要做任何一致性检查或防御性复制。

    序列化代理模式有两个限制。它与客户端可扩展的类不兼容；序列化代理模式所增强的功能和安全性并不是没有代价的。

    当你发现必须在客户端不可扩展的类上编写 readObject 或 writeObject 方法时，请考虑序列化代理模式。要想稳健地将带有重要约束条件的对象序列化时，这种模式可能是最容易的方法。













































