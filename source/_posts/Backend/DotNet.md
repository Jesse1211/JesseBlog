---
title: DotNet
categories:
  - Back-End
  - DotNet
---

- NET Core 和.NET Framework 的区别和.NET Core 优点
  - .NET Core 和.NET Framework 都是用于构建 Windows 和 Web 应用程序的跨平台框架。.NET Core 是开源的，跨平台的，它可以在 Windows，macOS，Linux 等操作系统上运行。而.NET Framework 只能运行在 Windows 系统上。
  - 优点：.NET Core 具有更小的文件大小、更快的启动时间和更好的性能表现，同时还可以使用新的 C#语言功能。
- 在.NET Core 中，内置依赖注入服务的生命周期
  - .NET CROE 内置依赖注入的三种生命周期：
  1.  Transient（瞬时）：即用即建，用后即弃。就是每次获取这个服务的实例时都要创建一个这个服务的实例。
  2.  Scoped（作用域）：这种类型的服务实例保存在当前依赖注入容器(IServiceProvider)上。在同作用域,服务每个请求只创建一次。
  3.  Singleton（单例）：服务请求时只创建实例化一次，其后相同请求都延用这个服务
- 请简述.NET Core 中的中间件（Middleware）的作用及其使用方法
  - 中间件（Middleware）是.NET Core 中的一种特殊组件，它可以处理 HTTP 请求和响应，并把请求传递到下一个中间件或终止请求。中间件在 ASP.NET 应用程序中扮演着非常重要的角色，能够为应用程序提供丰富的功能和服务，例如路由、认证、授权、缓存、日志、异常处理等。
  - 使用中间件在.NET Core 应用程序中添加组件或服务非常简单。ASP.NET Core 加载中间件的顺序与它们添加到中间件管道的顺序相同，因此可以按照需要添加中间件并调整它们的顺序。

# 如何在 Docker 中部署.NET Core 应用程序

在 Docker 中部署.NET Core 应用程序需要以下步骤：

1. 创建 Dockerfile 文件。可以在该文件中选择可用的.NET Core 官方镜像，设置工作目录、复制应用程序到容器中以及执行运行指令等
2. 在应用程序根目录下构建 Docker 镜像。使用 docker build 命令从 Dockerfile 文件中创建镜像
3. 运行应用程序容器。使用 docker run 命令从新构建的镜像中启动容器

# 在 Docker 中如何对容器进行监控和管理？

在 Docker 中，可以使用以下工具和方法对容器进行监控和管理：

1. Docker CLI：可以使用 Docker CLI 来查看所有正在运行的容器、停止或删除容器、以及查看容器日志等。例如，使用 docker ps 命令可以列出所有正在运行的容器，使用 docker stop 命令可以停止指定的容器。2. Docker Dashboard：是一个基于 Web 的 UI 管理工具，可以用于查看所有的 Docker 容器、镜像和网络等，以及启动、停止、重启和删除这些容器。Docker Dashboard 还提供了一些基本的容器监控功能，例如 CPU、内存和网络使用情况等。
2. cAdvisor（Container Advisor）：是一个开源的容器监控工具，可以监控每个容器的资源使用情况，例如 CPU、内存、网络和磁盘 I/O 等。cAdvisor 可以与 Docker 集成，通过 Docker API 获取各个容器的资源使用情况，同时还可以将这些数据导出到其他监控系统中。

# .NET Core 和.NET Framework 的区别和.NET Core 优点：

.NET Core 和.NET Framework 都是用于构建 Windows 和 Web 应用程序的跨平台框架。.NET Core 是开源的，跨平台的，它可以在 Windows，macOS，Linux 等操作系统上运行。而.NET Framework 只能运行在 Windows 系统上。

- 优点：.NET Core 具有更小的文件大小、更快的启动时间和更好的性能表现，同时还可以使用新的 C#语言功能。

# 在.NET Core 中，内置依赖注入服务的生命周期？

.NET CROE 内置依赖注入的三种生命周期：

1. Transient（瞬时）：即用即建，用后即弃。就是每次获取这个服务的实例时都要创建一个这个服务的实例。
2. Scoped（作用域）：这种类型的服务实例保存在当前依赖注入容器(IServiceProvider)上。在同作用域,服务每个请求只创建一次。
3. Singleton（单例）：服务请求时只创建实例化一次，其后相同请求都延用这个服务。详解-->小白面试：之.NET CROE 依赖注入的生命周期

# 请简述.NET Core 中的中间件（Middleware）的作用及其使用方法。

- 中间件（Middleware）是.NET Core 中的一种特殊组件，它可以处理 HTTP 请求和响应，并把请求传递到下一个中间件或终止请求。中间件在 ASP.NET 应用程序中扮演着非常重要的角色，能够为应用程序提供丰富的功能和服务，例如路由、认证、授权、缓存、日志、异常处理等。

- 使用中间件在.NET Core 应用程序中添加组件或服务非常简单。ASP.NET Core 加载中间件的顺序与它们添加到中间件管道的顺序相同，因此可以按照需要添加中间件并调整它们的顺序。

# 你在.NET Core 开发中使用过哪些 ORM 框架？在使用过程中遇到的问题？

我使用过 Entity Framework Core 和 Dapper ORM 框架。Entity Framework Core 提供了强大的对象关系映射（ORM）功能，可以简化数据库操作。它的 LINQ 支持非常好，同时也支持复杂查询和事务。但是它的性能可能不如 Dapper。Dapper 是一个轻量级的 ORM 框架，专注于执行 SQL 查询并将结果映射到对象。因为它使用纯 ADO.NET，所以它的性能比 EF Core 更快。在使用 ORM 框架时，可能会遇到一些问题。例如，在使用 Entity Framework Core 时，可能会出现查询性能不佳、内存占用过高等问题；在使用 Dapper 时，可能会出现错误的映射、SQL 注入等问题。针对这些问题，我们可以通过优化查询语句、调整缓存策略等方式来提升查询性能；通过设置对象跟踪机制、关闭缓存等方式来控制内存使用；通过参数化查询等方式来减少 SQL 注入的风险
