# 日志规范

对于分布式并发系统，日志是最重要的调试方法，很多时候是唯一手段。一个典型的程序包括三分之一正常业务处理逻辑，三分之一异常业务处理，还有三分之一是 log 代码。 Log 代码在不同级别/不同详细程度记录了系统的运维运行状态和调试数据。

鉴于日志的重要性及不改变运行代码的要求，所有日志手工禁止用 AOP 这类工具来写日志信息。

## 1 基本原则

- 日志也是程序的基本组成部分，和业务代码/错误处理代码一样。
- 日志的主要用户是运维人员和程序员，和业务人员无关。
- 日志有五个级别：Error， Warning， Info， Debug， Trace。
- Debug 日志记录完整的执行路径和关键运行数据，避免重复和不必要的细节。
- 任何错误/异常发生的地方都要用日志记录。
- 仔细规划日志的级别，如果下面的通用指南不够清楚，请在业务模块文档或前后端给出特别的日志级别指南。

## 2 基本功能

日志是运行时代码调试工具。日志的用户有二种：运维和程序员。相应的也有二个主要功能，通过不同的日志级别和运行状态来完成。

### 2.1 运维人员：系统状态

- 运行状态：缺省运行级别是 Info，可以知道运行的状态（系统的启动、停止、重启等）、性能汇总 （每个小时/每天的吞吐量和响应时间）等。
- 错误报警：程序员和运维人员用 Error 和 Warning来知道错误发生和错误现场信息。
- 立即响应：当出现 Error 级别的错误，需要相关人员立刻修复。

### 2.2 程序员：运行时跟踪调试

- 跟踪/定位：当错误出现后不能通过静态代码检查发现错误原因，需要在运行时打开 Debug。运行时没有 Debug 级别的信息输出，需要在要诊断的业务模块设置 Debug 级别来产生日志信息，用于事后跟踪调试下一次错误的发生。
- Trace 级别仅用于循环内或大的数据结构比如数组等给出详细的错误信息。

## 3 日志级别的使用指南

五个日志级别的定义和用途如下。

### 3.1 Error

Error 表示严重错误，系统异常或应用程序功能无法执行。比如未知的系统运行错误、不能连接到数据库、调用参数错误或严重业务数据错误。Error 级别的错误属于高优先级 bug，需要开发人员立即修复。

系统出现意料之外的异常也要用 Error 处理。因为 Unknown 的异常很可能非常严重，需要搞清楚每个 Unknown 的异常。

### 3.2 Warning

Warning 表示不影响程序继续运行或可以重试的各种系统错误，比如网络超时、不重要的数据错误、外部服务请求失败等。运维人员需要每周留意 Warning 信息，看是否有异常情况。

### 3.3 Info

Info 表示一个重要的系统事件。可以给系统运维人员提供重要的系统运行状态和性能统计数据。Info 事件不包括业务层面的事件，比如用户创建、订单的增删该查等。常见的 Info 事件有：

- 系统的生命周期：启动、初始化、停止等。
- 系统动态状态改变：动态配置改变、切换备用服务等。
- 过去一小时/一天的请求数目，平均请求时间等。

### 3.4 Debug

Debug 是调试的主要级别。这个级别的信息应该给出完整的执行路径和重要的执行结果。具体要求如下：

- Debug 应该涵盖代码执行的所有分支。如：
  - while 语句代码块
  - if-else 语句的 if 代码块和 else 代码块
  - throw exception 代码前
- 执行路径指函数调用和执行分支。函数调用时要么调用者，要么被调用者记录日志，但是不用重复记录。同时各个重要执行分支都用 Debug 日志记录分支的判断条件。
- 打印的信息不应该太详细（比如有十个以上的属性），也不应该用在重复十次以上的循环内部数据。

### 3.5 Trace

Trace 给出详细的程序运行状态。Trace 可以用在循环的内部或用于打印完整的详细信息。当输出详细信息时，通常也先有一个 Debug 级别的摘要信息。比如，Debug 信息给出数组的尺寸，而 Trace 级别给出具体的数组数据（所有元素或一部分元素）。在非常底层不重要的分支，也可以不用 Debug 而用 Trace 输出日志。

## 4 日志的使用效果

一个投入运行的生产系统，缺省的日志运行级别是 Info。可以看到系统的大概运行状态。

- 平常应该很少见到 Error 级别的错误。正式运行时，应该是一个月难得一见。
- Warning 级别的日志可能每天碰到，但是应该反应当时的网络状态或外部服务的稳定性。
- Info 级别的日志代表了系统的状态改变或环境的变化。必要时也可以给出运行性能的统计数据。
- Debug 用于反映出完整的程序执行路径和所用到的数据，但是不包含过数据细节。
- Trace 用于补充 Debug 数据的细节。比如很大的数据或循环里的数据。

## 5 最佳实践

### 5.1 日志格式

Java/Scala 日志采用 sl4j + logback 记录日志。

- 日志格式：`%date [%thread] [%X{TraceId}] %-5level %logger{80}.%M - %msg%n`
- 日志记录的地点：生产系统输出到 Console 或日志平台。本地开发/测试则可以记录到本地文件。
- 记录颗粒度：不同类型日志，记录到不同文件，每日分割，最后日志由日志分析平台收集。
- 日志记录级别：线上只开通 info 级别，测试环境可以开通其他级别，后期开启动态调级的设置。
- 分布式日志跟踪：每个请求进入系统时，都会在其处理线程中添加一个线程本地变量`TraceId`，记录日志时，自动采用本变量，如果需要创建线程做一些异步操作，或者需要调用其他服务，都请将 traceId 带上。

### 5.2 日志写法

日志是给系统运维和开发人员看的。所以给出的信息也是以程序调试为主。常见二种格式

```java
// 格式一
'user 1234 clicked on the save button on the sign-up page'

// 格式二
'userId:1234 clicked on buttonId:save on pageId:sign-up'
```

第二种格式给出了具体的变量名称和对应状态值，是推荐的日志格式。即参数名和参数值之间用':'分隔。

Debug 级别的日志在跨进程函数出入口进行记录时应成对出现。 推荐格式如下：

```java
// "Enter. "作为推荐的函数进入点的日志格式标准，后面可以加上关键参数的信息
"Enter GetOrder::orderId: 1234, employeeId:37"

// "Exit"作为推荐的函数退出点的日志格式标准
"Exit GetOrder"

// 当有返回值时，也可以记录返回的参数描述
"Exit GetOrderCount. Return value:42"
```

### 5.2 注意事项

- 对于失败的状态，在抛出异常的地方要记录日志，根据错误程度级别分别为 Error、Warning、Info、Debug。具体级别参考上面的解释。

- 跨进程服务的 API 需要有 Debug 级别日志成对记录请求参数和返回结果，这样也提供了相应时间记录。

- 日志语句中不要调用耗时的方法（在关闭日志以后，日志对性能的影响应该可以忽略不记）

```java
// 关闭日志以后以会有函数调用 toJson 或 toString，会对性能造成影响，避免使用；
logger.debug("Enter. request:{}", JsonUtils.toJson(params));
logger.debug("Enter. request:{}", params);

// 建议的方法
if (logger.isDebugEnabled()) {
    "Enter. request:{}", JsonUtils.toJson(params));
}
// 如果用 toString，则建议
if (logger.isDebugEnabled()) {
    "Enter. request:{}", params);
}
```

## 6 例外

任何规定都是要被打破的，只有有足够好的理由。

上面的规定，可以适当不遵守，但是需要有比较好的理由，经过同事审查同意并且写入文档。
