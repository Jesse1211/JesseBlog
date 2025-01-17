---
title: C#
categories:
  - Back-End
  - C#
---

# Types

| 类别                    | 描述     |                                       |
| ----------------------- | -------- | ------------------------------------- |
| 值类型 value type       | 简单类型 | int, byte, char, float, decimal, bool |
|                         | 枚举类型 | enum                                  |
|                         | 结构类型 | struct                                |
| 引用类型 reference type | 类类型   | object, string, 自定义 class          |
|                         | 接口类型 | interface                             |
|                         | 数组类型 | int[]                                 |
|                         | 委托类型 | delegate                              |

- Enum
  - 简单的 type - Days of the Week, Payment Status
- Struct
  - 可以用于 Enum, can have constructors, properties, and methods, but cannot have an explicit parameterless constructor in C#

# Delegate

- type-safe function pointer
- 通过使用 `+=`, `-=`, invoke 会调用所有的 delegate functions

# 堆和栈

值类型的数据被保存在栈(stack)上，引用类型的数据被保存在堆(heap)上，当值类型作为参数传递给函数时，会将其复制到新的内存空间中，因此在函数中对该值类型的修改不会影响原始值类型

# 拆箱和装箱

- 装箱
  - 将栈 (value type) 中的内容迁移到堆上 (reference type)
    - int double 转换为对应的引用(object)类型
- 拆箱
  - 把引用转成值类型，把堆中的内容转移到栈上去

```CSharp
int i=3;
objcet o=i;//装箱
i=(int)o //拆箱
```

# == 和 Equals 的区别

- Equals 继承自 Object，一般用于比较对象内容是否相等
- == 比较 type value 值是否相等，reference value 比较地址是否相等

# 内存泄漏

- 静态引用：如果一个静态对象长时间存活且占用大量内存，并且该对象不会被释放或重置，可能导致内存泄漏
- 不使用的引用对象没有置 null
- 委托或事件没有解除注册

# Virtual & Abstract

- 相同点
  - 抽象方法和虚方法都可以供派生类重写, 派生类重写父类的方法都要使用关键字 override 来声明。
- 不同点
  - 虚方法必须有方法名称和方法实现；抽象方法是只有方法名称,没有方法实现；
  - 虚方法在派生类中可以实现,也可以不实现；抽象方法是一种强制派生类覆盖的方法,子类必须实现,否则派生类将不能被实例化;
  - 抽象方法只能在抽象类中声明,虚方法不是，如果类包含抽象方法,那么该类也必须声明为抽象类；
  - 抽象方法使用修饰符 abstract 声明,虚方法使用方法修饰符 virtual 声明；

```CSharp
public abstract class AbstProgram            // 必须声明为抽象类
{
  public abstract string abstMethod();     // 抽象方法

  public virtual void virMethod()          // 虚方法
  {
    Console.WriteLine("hello, virmethod");
  }
}

public class Subclass: AbstProgram
{
  // 抽象方法必须实现
  public override string abstMethod()
  {
    return "hello,abstMethod";
  }

  // 虚方法可以实现,也可以选择不实现
  // public override void virMethod()
  // {
  //   Console.WriteLine("hello,virmethod");
  // }
}
```

# Interface & abstract

- 相同点

  - 都可以被继承
  - 都不能直接被实例化
  - 都可以包含方法声明且都没有实现
  - 派生类必须实现未实现的成员

- 不同点

  - 接口支持多实现，抽象类只能被单一继承实现；
  - 接口中的成员访问类型默认为公共的，不能有其他的访问修饰符修饰；
  - 定义的关键字不一样， 抽象类需要 abstract，而接口则使用 interface；
  - 接口可以作用于值类型(枚举可以实现接口)和引用类型，抽象类只能作用于引用类型；
  - 接口不能包含字段和已实现的方法, 接口只包含方法、属性、索引器、事件的签名；抽象类可以定义字段、属性、包含有实现的方法；
  - 在抽象类中, 新增一个方法的话, 继承类中可以不用作任何处理；而对于接口来说, 则需要修改继承类, 提供新定义的方法；
  - 接口用于规范, 更强调契约, 抽象类用于共性, 强调父子。抽象类是一类事物的高度聚合, 那么对于继承抽象类的子类来说, 对于抽象类来说, 属于"Is A"的关系; 而接口是定义行为规范, 强调"Can Do"的关系, 因此对于实现接口的子类来说, 相对于接口来说, 是"行为需要按照接口来完成"；

```CSharp
public interface IMyInterface       // 接口
{
    public void intMethod();
}

public abstract class AbstProgram       // 抽象类
{
    public abstract void abstMethod();
}


public class SubClass: AbstProgram, IMyInterface
{
    public override void abstMethod()
    {
        Console.WriteLine("hello,abstMethod");
    }

    public void intMethod()
    {
        Console.WriteLine("hello,intMethod");
    }
}
```

# Interface & Class

- 相同点

  - 接口、类和结构体都可以从多个接口继承；
  - 接口似于抽象基类, 继承接口的任何非抽象类型都必须实现接口的所有成员；
  - 接口和类都可以包含事件、索引器、方法和属性；

- 不同点

  - 不能直接实例化接口；
  - 接口可以实现多继承，类只能被派生类单独继承；
  - 接口只包含方法或属性的声明，不包含方法的实现， 而类定义的方法必须实现；
  - 表达的含义不同,接口定义了所有类继承接口时应遵循的"语法合同"；接口定义了语法合同 "是什么" 部分,派生类定义了语法合同 "怎么做" 部分；

```CSharp
interface IMyInterface
{
    void MethodToImplement();           // 接口成员
}

class InterfaceImplementer: IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
    }

    // 类实现接口
    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }
}
```

# virtual、sealed、override 和 abstract 的区别

- virtual 声明虚方法的关键字，说明该方法可以被重写。
- sealed 说明该类不可被继承，也就是声明该类为密封类。
- override 重写基类/父类的方法 。
- abstract 声明抽象类和抽象方法的关键字，抽象方法不提供实现，由子类实现，抽象类不可实例化。

```CSharp
public abstract class Program
{
    public abstract string abstmethod();     // abstract

    public virtual void virmethod()          // virtual
    {
        Console.WriteLine("hello,virmethod");
    }
}

public sealed class SubClass: Program      // sealed
{
    public override string abstmethod()      // override
    {
        return "hello,abstmethod";
    }
}
```

# const 与 readonly 的区别

- const 字段只能在该字段的声明中初始化；而 readonly 字段可以在声明或构造函数中初始化。因此，根据所使用的构造函数，readonly 字段可能具有不同的值。
- const 字段是编译时常数，而 readonly 字段是运行时常数，是运行时确认的（一个在编译时就确定了，一个在运行时才确定）。
- const 默认就是静态的，而 readonly 如果设置成静态的就必须显示声明。
- const 对于引用类型的常数，可能的值只能是 string 和 null；而 readonly 可以是任何类型。

```CSharp
class Program
{
public static readonly int NumberA = NumberB \* 10;
public static readonly int NumberB = 10;

    public const int NumberC = NumberD * 10;
    public const int NumberD = 10;

    static void Main(string[] args)
    {
        Console.WriteLine("NumberA is {0}, NumberB is {1}.", NumberA, NumberB);     //NumberA is 0, NumberB is 10.
        Console.WriteLine("NumberC is {0}, NumberD is {1}.", NumberC, NumberD);     //NumberC is 100, NumberD is 10.
        Console.ReadLine();
    }
}
```

# 重载 (override) 和重写 (overload) 有什么区别

- 重载: 当类包含两个名称相同但签名不同(方法名相同,参数列表不相同)的方法时发生方法重载。用方法重载来提供在语义上完成相同而功能不同的方法。(一个类中、多个方法)
- 重写: 在类的继承中使用, 通过覆写子类方法可以改变父类虚方法的实现。(二个类以上)

```CSharp
public class Program
{
    // 重载(override)
    public void method(int num1)
    {
    }

    public void method(string str1)
    {
    }

    // 虚方法
    public virtual void virtmethod()
    {
        Console.WriteLine("hello,virtmethod");
    }
}

public class SubClass: Program
{
// 重写(overload)
public override void virtmethod()
    {
        Console.WriteLine("hello,overload");
    }
}
```

# 结构体和类的区别

- 结构体是值类型，类是引用类型。
- 结构体常用于数据存储，类多用于行为。
- 类支持继承， 而结构体不支持继承。
- 类可以为抽象类，结构体不支持抽象模式。
- 结构体不支持无参构造函数，也不支持析构函数，并且不能有 Protected 修饰符。

```CSharp
// Books 结构体实现
struct Books
{
    public string title;
    public string author;
    public string subject;
    public int book_id;
};
```

# ref 与 out 的区别

- 使用 ref 型参数时，传入的参数必须先被初始化；对 out 而言，必须在方法中对其完成初始化。
- 使用 ref 和 out 时，在方法的参数和执行方法时，都要加 ref 或 out 关键字，以满足匹配。
- out 适合用在需要 retrun 多个返回值的地方，而 ref 则用在需要被调用的方法修改调用者的引用的时候。

```CSharp
class Program
{
    public void test1(ref int i)
    {
        i = 99;
    }

    public void test2(out int i)
    {
        i = 11;
    }

    static void Main(string[] args)
    {
        Program p = new Program();

        int i = 1;
        p.test1(ref i);     // i = 99

        int j;
        p.test2(out j);     // j = 10
    }
}
```

# 值类型和引用类型的区别

- 值类型: 就是一个包含实际数据的对象。即当定义一个值类型的变量时，C#会根据它所声明的类型，以栈方式分配一块大小相适应的存储区域给这个变量，随后对这个变量的读或写操作就直接在这块内存区域进行；
- 引用类型: 一个引用类型的变量不存储它们所代表的实际数据，而是存储实际数据的引用。引用类型分两步创建：首先在栈上创建一个引用变量，然后在堆上创建对象本身，再把这个内存的句柄（也是内存的首地址）赋给引用变量；

```CSharp
//引用类型（由于使用了'Class'）
class SomeRef { public Int32 x; }

//值类型（由于使用了'Struct'）
struct SomeVal { public Int32 x; }

class Program
{
    static void ValueTypeDemo()
    {
        SomeRef r1 = new SomeRef();
        SomeVal v1 = new SomeVal();
        r1.x = 5;
        v1.x = 5;
        Console.WriteLine(r1.x); //显示"5"
        Console.WriteLine(v1.x); //同样显示"5"

        SomeRef r2 = r1;
        SomeVal v2 = v1;
        r1.x = 8;
        v1.x = 9;

        Console.WriteLine(r1.x);    //显示"8"
        Console.WriteLine(r2.x);    //显示"8"
        Console.WriteLine(v1.x);    //显示"9"
        Console.WriteLine(v2.x);    //显示"5"
    }
}
```

# 拆箱和装箱的定义及拆箱和装箱的性能影响？怎么解决?

- 装箱: 值类型转换为引用类型;
- 拆箱：引用类型转换为值类型；
- 影响: 都涉及到内存的分配和对象的创建，有较大的性能影响；
- 解决：使用泛型;

```CSharp
public class Solution
{
    static void Main(string[] args)
    {
        int i = 123; //声明一个 int 类型的变量 i，并初始化为 123
        object obj = i; //执行装箱操作
        Console.WriteLine("装箱操作：值为{0}，装箱之后对象为{1}", i, obj);
        int j = (int)obj; //执行拆箱操作
        Console.WriteLine("拆箱操作：装箱对象为{0}，值为{1}", obj, j);
    }
}
```

# 委托是什么？事件是不是委托？

c# 中的委托（Delegate）类似于 c 或 c++ 中函数的指针。委托（Delegate） 是存有对某个方法的引用的一种引用类型变量。引用可在运行时被改变。委托（Delegate）特别用于实现事件和回调方法。所有的委托（Delegate）都派生自 System.Delegate 类。

- 事件是特殊的委托，事件内部是基于委托来实现的。

```CSharp
delegate int NumberChanger(int n);
class TestDelegate
{
    static int num = 10;
    public static int AddNum(int p)
    {
        num += p;
        return num;
    }

    public static int MultNum(int q)
    {
        num *= q;
        return num;
    }
    public static int GetNum()
    {
        return num;
    }

    static void Main(string[] args)
    {
        // 创建委托实例
        NumberChanger nc1 = new NumberChanger(AddNum);
        NumberChanger nc2 = new NumberChanger(MultNum);
        // 使用委托对象调用方法
        nc1(25);
        Console.WriteLine("Value of Num: {0}", GetNum());   // Value of Num: 35
        nc2(5);
        Console.WriteLine("Value of Num: {0}", GetNum());   // Value of Num: 175
        Console.ReadKey();
    }
}
```

# 构造函数 Constructor 是否可以被继承？是否可以被 Override 重载？

Constructor 不可以被继承，因此不能被重写（Overriding），但可以被重载(Overloading)。

```CSharp
public class Student
{
    public int Age { get; }
    public string Name { get; }

    public Student()
    {
        Console.WriteLine("对象被创建。");
    }

    public Student(string name) : this()
    {
        this.Name = name;
    }

    public Student(string name, int age) : this(name)
    {
        this.Age = age;
    }
}

public class Program
{
    static void Main(string[] args)
    {
        Student student_1 = new Student();
        Student student_2 = new Student("姓名", 14);
        Console.WriteLine(student_2.Name + ' ' + student_2.Age);
    }
}
```

# String 类是否可以被继承？

- String 类是 sealed 类故不可以继承。
- 当对一个类应用 sealed 修饰符时，此修饰符会阻止其他类从该类继承。 在下面的示例中，类 SealedProgram 从类 DemoProgram 继承，但是任何类都不能从类 SealedProgram 继承。

```CSharp
class DemoProgram { };
sealed class SealedProgram: DemoProgram { };
```

# Task 和 Thread 的区别

- Task 比较新，发布于.NET 4.5 版本，而 Thread 在.NET 1.1 版本。
- Task 能结合新的 async/await 代码模型写代码, 而 Thread 则不支持。
- Task 可以创建线程池，返回主线程，管理 api， 而 Thread 则不能。
- Task 是多核多线程，Thread 是单核多线程。

```CSharp
public class Solution
{
  static void Main()
  {
    var testTask = new Task(() =>
    {
      Console.WriteLine("hello, Task");
    });

    Console.WriteLine(testTask.Status); // Created
    testTask.Start();
  }
}

public class Solution
{
    static void Main(string[] args)
    {
        ThreadTest test = new ThreadTest(); //创建 ThreadTest 类的一个实例
        Thread thread = new Thread(new ThreadStart(test.MyThread)); //调用 test 实例的 MyThread 方法
        thread.Start(); //启动线程
        Console.ReadKey();
    }
}

public class ThreadTest
{
    public void MyThread()
    {
      Console.WriteLine("这是一个实例方法");
    }
}
```

# 死锁的必要条件？怎么克服?

- 死锁的原因主要是:
  - 因为系统资源不足。
  - 进程运行推进的顺序不合适。
  - 资源分配不当等。如果系统资源充足,进程的资源请求都能够得到满足,死锁出现的可能性就很低,否则就会因争夺有限的资源而陷入死锁。其次,进程运行推进顺序与速度不同,也可能产生死锁。
- 必要条件
  - 互斥条件：一个资源每次只能被一个进程使用。
  - 不剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。
  - 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。
  - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 处理死锁的措施

  - 预防死锁：通过设置某些限制条件，去破坏产生死锁的四个必要条件中的一个或几个条件，来防止死锁的发生。
  - 避免死锁：在资源的动态分配过程中，用某种方法去防止系统进入不安全状态，从而避免死锁的发生。
  - 检测死锁：允许系统在运行过程中发生死锁，但可设置检测机构及时检测死锁的发生，并采取适当措施加以清除。
  - 解除死锁：当检测出死锁后，便采取适当措施将进程从死锁状态中解脱出来。

# Error 和 Exception 有什么区别？

- Error 是不可捕捉到的，无法采取任何恢复的操作。Except 表示可恢复的例外，这是可捕捉到的。 举个例子： 跟儿子逛街，儿子看中一个奥特曼是 Exception，没带钱是 Error。
- Error 更多的表示系统方面的问题,如系统崩溃,虚拟机错误,内存空间不足,方法调用栈溢出，内存溢出等；而 Exception 类表示程序可以捕获到的异常（比方说 c#的 try-catch 语句），可以捕获且可能恢复，出现这类异常，应该及时处理，使得程序正常运行，从而避免影响整个项目的运行。

# UDP 和 TCP 连接有什么区别?

- TCP 是传输控制协议，提供的是面向连接的，是可靠的，字节流服务，TCP 提供超时重拨，检验数据功能。
- UDP 是用户数据报协议，是一个简单的面向数据报的传输协议，是不可靠的连接。
- TCP 保证数据正确性，UDP 可能丢包，TCP 保证数据顺序，UDP 不保证。
- TCP 对系统资源的要求较多, UDP 较少。
