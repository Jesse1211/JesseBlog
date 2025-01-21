---
title: DotNet
categories:
  - Back-End
  - DotNet
---

# 跨语言

CLS 从类型、命名、事件、属性、数组等方面对语言进行了共性的定义及规范。这些东西被提交给欧洲计算机制造联合会 ECMA，称为：共同语言基础设施。

# .NET Core 和.NET Framework 的区别和.NET Core 优点：

.NET Core 和.NET Framework 都是用于构建 Windows 和 Web 应用程序的跨平台框架。.NET Core 是开源的，跨平台的，它可以在 Windows，macOS，Linux 等操作系统上运行。而.NET Framework 只能运行在 Windows 系统上。

- 优点：.NET Core 具有更小的文件大小、更快的启动时间和更好的性能表现，同时还可以使用新的 C#语言功能。

# 在.NET Core 中，内置依赖注入服务的生命周期？

.NET CROE 内置依赖注入的三种生命周期：

1. Transient（瞬时）：即用即建，用后即弃。就是每次获取这个服务的实例时都要创建一个这个服务的实例。
2. Scoped（作用域）：这种类型的服务实例保存在当前依赖注入容器(IServiceProvider)上。在同作用域,服务每个请求只创建一次。
3. Singleton（单例）：服务请求时只创建实例化一次，其后相同请求都延用这个服务。

# 请简述.NET Core 中的中间件（Middleware）的作用及其使用方法。

中间件在 ASP.NET 应用程序中扮演着非常重要的角色，能够为应用程序提供丰富的功能和服务，处理 HTTP 请求和响应，并把请求传递到下一个中间件或终止请求, 例如路由、认证、授权、缓存、日志、异常处理等

职责

1. 选择是否将请求传递给管道中的下一个中间件
2. 在管道中的下一个中间件的前后执行工作。

# Task

表示异步操作的执行结果, 用于处理耗时任务

- Task 可能涉及多线程: 避免 .Result 或 .Wait()：尽量避免同步等待 Task 结果，这会导致线程阻塞。
- Async 和 Await 本身不创建线程, 他们把人物挂起并等待完成

# 泛型 Generic

## Genetic Unions

```csharp
List<T>
Dictionary<TKey, TValue>
HashSet<T>
Queue<T>
Stack<T>
SortedList<TKey, TValue>
SortedSet<T>
```

## Generic 接口

继承层次结构:

```plaintext
IEnumerable<T>
   └── ICollection<T>
          ├── IList<T>
          └── IDictionary<TKey, TValue>
```

接口：适合代码的灵活性、扩展性需求，关注行为而非实现。
具体实现：适合性能优化和特定功能需求，代码更直接。

### `IEnumerable<T>`

提供对集合的遍历能力.

### `ICollection<T>`

在 IEnumerable 的基础上定义通用集合的方法（如添加、删除、计数）。

### `IList<T>`

提供列表的索引访问功能.

### `IDictionary<TKey, TValue>`

提供键值对的集合接口。

## Generic Delegate

定义可以处理多种数据类型的回调或事件。

- `Func<T, TResult>`
  - Action 是 Delegate, 相当于 void, 表示没有返回值但有输入参数的方法。
    - 和 Func 的主要区别就是有没有返回值
    - 用于事件处理程序 / 回调函数
- `Action<T>` 表示没有返回值但有输入参数的方法。
- `Predicate<T>` 表示返回布尔值的条件判断方法。

```csharp
Func<int, int, int> add = (a, b) => a + b;
Action<string> print = message => Console.WriteLine(message);
Predicate<int> isEven = num => num % 2 == 0;

print($"10 + 20 = {add(10, 20)}");
Console.WriteLine($"Is 4 even? {isEven(4)}");
```

## Generic Class

- `IActionResult`: 接口，用于表示 ASP.NET Core 的控制器操作返回的各种结果类型: `BadRequest, NotFound...`
- `ActionResult<T>`: 表示返回一个具体类型的结果 T, 或其他 ActionResult 类型。

```csharp
public ActionResult<Item> GetItem(int id)
{
    if (id <= 0)
        return BadRequest();

    var item = GetItemFromDatabase(id);
    if (item == null)
        return NotFound();

    return item; // 强类型返回
}
```

# 如何在 Docker 中部署.NET Core 应用程序

在 Docker 中部署.NET Core 应用程序需要以下步骤：

1. 创建 Dockerfile 文件。可以在该文件中选择可用的.NET Core 官方镜像，设置工作目录、复制应用程序到容器中以及执行运行指令等
2. 在应用程序根目录下构建 Docker 镜像。使用 docker build 命令从 Dockerfile 文件中创建镜像
3. 运行应用程序容器。使用 docker run 命令从新构建的镜像中启动容器

# 在 Docker 中如何对容器进行监控和管理？

在 Docker 中，可以使用以下工具和方法对容器进行监控和管理：

1. Docker CLI：可以使用 Docker CLI 来查看所有正在运行的容器、停止或删除容器、以及查看容器日志等。例如，使用 docker ps 命令可以列出所有正在运行的容器，使用 docker stop 命令可以停止指定的容器。2. Docker Dashboard：是一个基于 Web 的 UI 管理工具，可以用于查看所有的 Docker 容器、镜像和网络等，以及启动、停止、重启和删除这些容器。Docker Dashboard 还提供了一些基本的容器监控功能，例如 CPU、内存和网络使用情况等。
2. cAdvisor（Container Advisor）：是一个开源的容器监控工具，可以监控每个容器的资源使用情况，例如 CPU、内存、网络和磁盘 I/O 等。cAdvisor 可以与 Docker 集成，通过 Docker API 获取各个容器的资源使用情况，同时还可以将这些数据导出到其他监控系统中。
