

客户端是Unity2017.3.1f1, 采用c#语言，服务器端使用的是c++。

客户端采取的protobuf版本是:3.5.1 服务器对应的版本3.5.1


<b>客户端</b>

我们客户端protobuf来源于google官方的github仓库，对应本项目protobuf-lib文件夹，我们在生成的dll的时候通过后处理事件，同时生成对应的mdb文件，且copy到对应的unity目录。所以需要你配置环境变量：
UnityInstallPath，即unity的安装路径，不包含到Editor

关于unity protobuf的信息可以参考：https://github.com/google/protobuf/issues/644




通过修改上面的源码，在这里我们优化消除了protobuf在序列化和反序列化时候产生gc的状况。原生的代码使用了大量的.net4和.net6的语法， 我们也直接修改成net3以下 unity直接支持的语法。

之前的产生gc

<img src="tools/img/gc.jpg">

修改之后消除gc：

<img src="tools/img/gc_no.jpg">

优化操作：

在优化的时候，为了避免反序列化产生的实例（new），我们在每一个proto Message对应的类中加了一个共享的对象，这个对象只会new 一次，下次使用的时候直接改对象里的成员值，而不是直接new一个对象。

还有就是重写了CodeInputStream里的代码，内存中也是只会存在一个共享的CodeInputStream, 而不是每次都根据bytes[]或者stream来new一个新对象。


<b>工具</b>

tools目录：

在tools/ProtobufTool/目录中：

运行build.bat 会生成cs proto文件，并将拷贝到对应的unity目录中

运行build_server.bat 会生成.h .cc文件， 并拷贝到Server目录


<b>服务器</b>

请将项目需要的.proto 全部定义在game.proto文件中，自动生成的代码全部保存在PBMessage.cs中

运行build—server.bat 会生成服务器端对应的代码。

服务端采用的protobuf版本是：protobuf-cpp-3.5.1， 对应的下载地址：https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-cpp-3.5.1.zip

服务器用到的protobuf编译生成的libprotocd.lib、libprotobufd.lib、protoc.exe由于文件太大， 没有上传。
读者自行生成，可以参照csdn这篇paper: https://blog.csdn.net/program_anywhere/article/details/77365876  

服务器socket测试只支持windows平台，其他平台（linux）没有测试


运行效果如下图：
<img src="tools/img/show.gif">


<b>最后</b>

我们在提供unity使用protobuff v3.0的同时，也提供了对protobuff v2.0的支持，对应的分支是： client_protobuf_v2

可以在终端上敲击下面命令，进入到

```sh

git clone https://github.com/huailiang/game_socket_protobuf.git

git checkout client_protobuf_v2

```

其他分支：

ierator_rcv：接收的消息的时候在协程里处理，这样就不会卡住主线程，处理回调的时候也不用考虑回调里使用unity api的问题

```bash
git checkout ierator_rcv

```

asyn_recv: 使用Socket自带的api异步方法： BeginReceive和EndReceive 这两个是对偶出现的。 c#内部实现是基于线程池（thread pool)的，所以不能直接再receive回调里使用unity api

```bash
git checkout asyn_recv

```


mutthread: 手动开启一个线程用来接收socket消息，不能直接再receive回调里使用unity api

```bash
git checkout mutthread

```
