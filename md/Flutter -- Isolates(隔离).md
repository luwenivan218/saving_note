> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7312330825206251571)

Dart 使用隔离模型来实现并发。Isolate 只不过是线程的包装器。但根据定义，线程可以共享内存， 而隔离不能共享内存。

适用场景：它们非常适合解析庞大的 JSON 文件、复杂的图像处理或复杂的计算等繁重任务。

实现
--

### 1、**创建隔离体**

#### 1.1、使用 Isolate.spawn 创建一个隔离

> Isolate.spawn(dataProcessingIsolate, data);

#### 1.2、使用 Isolate.run 创建一个隔离

[`Isolate.run`](https://link.juejin.cn?target=http%3A%2F%2Fisolate.run%2F "http://isolate.run/")是将任务推送到隔离区的简单方法。它非常适合快速卸载任务，节省大量时间。这有助于使用这几行代码来生成、错误处理、消息传递和终止隔离。

> Isolate.run((){});

#### 1.3、使用 compute 创建一个隔离

*   产生一个 _Isolate_，
*   在该隔离上运行_回调_函数，向其传递一些数据，
*   返回值，结果回调，
*   并在回调执行结束时终止 _Isolate 。_

**注意**

“回调” 函数**必须**是顶级函数， **不能**是闭包或类的方法（静态或非静态）。

```
import 'dart:io';  
import 'package:flutter/foundation.dart';  
import 'package:image/image.dart' as img;  
  
void main() async {  
File imageFile = File('image.jpg');  
  
// 在单独的隔离区中加载和处理图像
img.Image processedImage = await compute(_processImage, imageFile);  
  
// 将处理后的图像保存到新文件
File processedImageFile = File('processed_image.jpg');  
await processedImageFile.writeAsBytes(img.encodeJpg(processedImage));  
print('处理后的图像保存到: ${processedImageFile.path}');  
}  
  
img.Image _processImage(File imageFile) {  
// 获取图片信息
img.Image image = img.decodeImage(imageFile.readAsBytesSync());  
.... 执行图像处理操作
return processedImage;  
}


```

Isolate 和 Compute 之间的主要区别在于它们提供的控制级别和复杂性。Flutter Isolate 对隔离区的创建和管理提供了更细粒度的控制，使其适合需要与主隔离区进行广泛通信或涉及多个隔离区协同工作的复杂任务。另一方面，Compute 提供了一种更直接的方法将短期任务卸载到单独的隔离区，而无需手动管理隔离区。

#### 1.4、使用 Isolate.spawnUri 创建一个隔离

[**Isolate.spawnUri**](https://link.juejin.cn?target=https%3A%2F%2Fapi.dart.dev%2Fstable%2F2.18.2%2Fdart-isolate%2FIsolate%2FspawnUri.html "https://api.dart.dev/stable/2.18.2/dart-isolate/Isolate/spawnUri.html") 根据您传入的 URI 在 **新隔离组**中创建一个新隔离。此 URI 必须指向包含 main 函数的 dart 文件或 AOT 快照。

#### 1.5、何时使用新的 Run 方法，何时使用旧的 spawn 方法？

如果仅发生 1 次的消息传递，应该使用 run 方法，它大大减少了代码行数和测试用例。 但是，如果需要在隔离之间传递多条消息，使用 **Isolate.spawn()** 方法。例如，当您开始在工作隔离上下载文件并希望在 UI 上显示下载进度时。这意味着需要一次又一次地传递进度计数。

### 2、**隔离通信：**

发送任务

> SendPort.send(data)

接受任务

> var receivePort = ReceivePort(); Isolate.spawn(dataProcessingIsolate,receivePort.sendPort);

### 3、隔离组

当一个 Isolate 将一个对象传递给另一个 Isolate 时，该对象必须被深度复制。这意味着需要花费大量时间将对象复制到另一个隔离区，这可能会导致卡顿。为了避免这种情况，隔离被重新设计并发明了隔离组。隔离组，意味着一组隔离，它们共享一些表示正在运行的应用程序的通用内部数据结构。这意味着每次生成新的 Isolate 时，不需要再次构建新的内部数据结构。因为他们一起分享。

不要将这些内部数据结构与可变对象混淆。隔离者仍然无法彼此共享可变对象。消息传递还是需要的。但是，由于同一 Isolate 组中的 Isolate 共享相同的堆，这意味着生成新 Isolate 的速度要快 100 倍，并且消耗的内存要少 10-100 倍。

### 事件循环

当您启动 _Flutter_（或任何 _Dart_）应用程序时，将创建并启动一个新的 **Isolate** 该_隔离_将是您必须关心整个应用程序的唯一隔离。

所以，当这个隔离被创建时，Dart 会自动

1.  初始化 2 个_队列_，分别是 “ **MicroTask** ” 和 “ **Event** ”FIFO 队列；
2.  执行 **main()** 方法，一旦该代码执行完成，
3.  启动**事件循环**

在线程的整个生命周期中，一个称为**事件循环**的内部且不可见的进程将驱动代码的执行方式以及执行顺序，具体取决于 _MicroTask_ 和_事件_队列的内容。

事件_循环_对应于某种_无限_循环，由内部时钟控制，在每个_时钟周期_，**如果没有其他 Dart 代码被执行**，则执行如下操作：

```
void eventLoop(){
  while (microTaskQueue.isNotEmpty){
    fetchFirstMicroTaskFromQueue();
    executeThisMicroTask();
    return;
  }

  if (eventQueue.isNotEmpty){
    fetchFirstEventFromQueue();
    executeThisEventRelatedCode();
  }
}


```

正如我们所看到的，_MicroTask_ 队列优先于_事件_队列，但这两个队列是用来做什么的呢？

### 微任务队列

MicroTask 队列用于**非常短的**内部操作，这些操作需要异步运行。

_作为 MicroTask_ 的示例，您可以想象必须在资源关闭后立即对其进行处置。由于关闭过程可能需要一些时间才能完成，您可以编写如下内容：

```
  void closeAndRelease() {
    scheduleMicroTask(_dispose);
    _close();
  }
  void _close(){
    ...
  }

  void _dispose(){
  }


```

大多数时候，您不需要使用微任务队列。举个例子，整个 Flutter 源代码只引用了 [scheduleMicroTask()](https://link.juejin.cn?target=https%3A%2F%2Fwww.flutteris.com%2F "https://www.flutteris.com/") 方法 7 次。

### 事件队列

事件队列用于引用由以下结果产生的_操作_

*   外部事件，例如
    *   输入 / 输出；
    *   手势;
    *   绘画;
    *   定时器；
    *   流 Stream；
    *   ...
*   Future

事实上，每次触发外部事件时，都会将相应要执行的代码引用到事件队列中。 一旦不再有任何_微任务_要运行，_事件循环就会考虑_ _事件_队列中的第一项并执行它。 **Futures** 也是通过_事件_队列处理的。

### Future

Future 对应于_异步_运行并在未来某个时间点完成（或失败）的_任务_。

当你实例化一个新的 _Future_ 时：

*   _该 Future_ 的实例被创建并记录在由 _Dart_ 管理的内部数组中；
*   _这个 Future_ 需要执行的代码直接被压入 _Event_ Queue；
*   未来_的实例_返回状态（不完整）；
*   如果有，则执行下一个同步代码（**不是 Future 的代码**）

一旦 _事件循环_从_事件循环中获取_ _Future_ 引用的代码，就会像任何其他 _Event_ 一样执行 当该代码将被执行并完成（或失败）时，其 **then()** 或 **catchError()** 将直接执行。