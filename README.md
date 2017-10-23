# Java8
Lambda表达式
λ表达式可以被当做是一个Object（注意措辞）。λ表达式的类型，叫做“目标类型（target type）”。
λ表达式的目标类型是“函数接口（functional interface）”，这是Java8新引入的概念。
它的定义是：一个接口，如果只有一个显式声明的抽象方法，那么它就是一个函数接口。一般用@FunctionalInterface标注出来（也可以不标）。
@FunctionalInterface
    public interface Runnable { void run(); }
    
    public interface Callable<V> { V call() throws Exception; }
    
    public interface ActionListener { void actionPerformed(ActionEvent e); }
    
    public interface Comparator<T> { int compare(T o1, T o2); boolean equals(Object obj); }
注意最后这个Comparator接口。它里面声明了两个方法，貌似不符合函数接口的定义，但它的确是函数接口。
这是因为equals方法是Object的，所有的接口都会声明Object的public方法——虽然大多是隐式的。
所以，Comparator显式的声明了equals不影响它依然是个函数接口。

你可以用一个λ表达式为一个函数接口赋值：
 
    Runnable r1 = () -> {System.out.println("Hello Lambda!");};
    
然后再赋值给一个Object：

    Object obj = r1;
    
但却不能这样干：

    Object obj = () -> {System.out.println("Hello Lambda!");}; // ERROR! Object is not a functional interface!

必须显式的转型成一个函数接口才可以：

    Object o = (Runnable) () -> { System.out.println("hi"); }; // correct
    
一个λ表达式只有在转型成一个函数接口后才能被当做Object使用。所以下面这句也不能编译：

    System.out.println( () -> {} ); //错误! 目标类型不明
    
必须先转型:

    System.out.println( (Runnable)() -> {} ); // 正确

一个λ表达式其实就是定义了一个匿名方法，只不过这个方法必须符合至少一个函数接口。

二、λ表达式用在何处

1、λ表达式主要用于替换以前广泛使用的内部匿名类，各种回调，比如事件响应器、传入Thread类的Runnable等。看下面的例子：

    Thread oldSchool = new Thread( new Runnable () {
        @Override
        public void run() {
            System.out.println("This is from an anonymous class.");
        }
    } );
    
    Thread gaoDuanDaQiShangDangCi = new Thread( () -> {
        System.out.println("This is from an anonymous method (lambda exp).");
    } );

注意第二个线程里的λ表达式，你并不需要显式地把它转成一个Runnable；
因为Java能根据上下文自动推断出来：一个Thread的构造函数接受一个Runnable参数，而传入的λ表达式正好符合其run()函数，所以Java编译器推断它为Runnable。
从形式上看，λ表达式只是为你节省了几行代码。但将λ表达式引入Java的动机并不仅仅为此。
Java8有一个短期目标和一个长期目标。短期目标是：配合“集合类批处理操作”的内部迭代和并行处理（下面将要讲到）；
长期目标是将Java向函数式编程语言这个方向引导（并不是要完全变成一门函数式编程语言，只是让它有更多的函数式编程语言的特性）；
也正是由于这个原因，Oracle并没有简单地使用内部类去实现λ表达式，而是使用了一种更动态、更灵活、易于将来扩展和改变的策略（invokedynamic）。

2、λ表达式与集合类批处理操作（或者叫块操作）

上文提到了集合类的批处理操作。这是Java8的另一个重要特性，它与λ表达式的配合使用乃是Java8的最主要特性。
集合类的批处理操作API的目的是实现集合类的“内部迭代”，并期望充分利用现代多核CPU进行并行计算。
Java8之前集合类的迭代（Iteration）都是外部的，即客户代码。而内部迭代意味着改由Java类库来进行迭代，而不是客户代码。例如：
    for(Object o: list) { // 外部迭代
        System.out.println(o);
    }
可以写成：
    list.forEach(o -> {System.out.println(o);}); //forEach函数实现内部迭代
集合类（包括List）现在都有一个forEach方法，对元素进行迭代（遍历），所以我们不需要再写for循环了。
forEach方法接受一个函数接口Consumer做参数，所以可以使用λ表达式。

3、Lambda表达式与匿名内部类的共同点
Lambda表达式是匿名内部类的一种简化，因此它可以部分取代匿名内部类的作用，Lambda表达式与匿名内部类存在如下相同点： 
（1）Lambda表达式与匿名内部类一样，都可以直接访问“effectively final”的局部变量，以及外部类的成员变量（包括实例变量和类变量）。 
（2）Lambda表达式创建于匿名内部类生成的对象一样，都可以直接调用接口中集成的默认方法。

4、Lambda表达式与匿名内部类区别
（1）匿名内部类可以为任意接口创建实例——不管接口包含多少个抽象方法，只要匿名内部类实现了所有的抽象方法即可；但Lambda表达式只能为函数式接口创建实例。 
（2）匿名内部类可以为抽象类甚至普通类创建实例； 
（3）匿名内部类实现的抽象方法的方法体允许调用接口中定义的默认方法；但Lambda表达式的代码块不允许调用接口中的默认方法。

