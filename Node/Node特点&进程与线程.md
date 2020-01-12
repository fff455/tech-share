# Node 优势

Node在2009年被创造出来，最初的目的是作为一个轻量级的WEB服务器框架   
WEB服务器很明显的特点是事件驱动以及非阻塞的IO处理，因此JavaScript作为了Node的实现语言  

## Node的特点

1. 异步IO
Node的最大特点是它的异步IO，相对其他语言，以python，Java以及C++为例，它们在处理文件读写时的操作如下：

- python

```python
f = open("sxl.txt")
lines = f.read()
print lines
f.close()
```

- Java

```java
InputStreamReader read = new InputStreamReader(new FileInputStream(file), encoding);
BufferedReader bufferedReader = new BufferedReader(read);
String lineTxt = null;

while ((lineTxt = bufferedReader.readLine()) != null){
  list.add(lineTxt);
}
bufferedReader.close();
read.close();
```

- C++

```c++
char buffer[256];
ifstream in("test.txt");
if (! in.is_open()){
  cout << "Error opening file"; exit (1); 
}
while (!in.eof()){
  in.getline (buffer,100);
  cout << buffer << endl;
}
```

毫无疑问，在读写文件时，上述语言都是通过阻塞式的操作来处理，在读写完成之前，是会阻塞之后的操作，直到完成后才会继续执行  
我们再来看一下Node的读写文件

- Node

```javascript
fs.readFile('input.txt', function (err, data) {
  if (err) {
      return console.error(err);
  }
  console.log("读取完成: " + data.toString());
});
console.log("发起读取文件");
```

Node提供了两种读取文件的模式，一种是上述异步的模式，一种是同步的模式。同步的模式类似其他的语言是会阻塞掉后面的操作，而上述异步的操作是会异步执行，当执行到读写文件的操作时，Node可以继续执行之后的代码，在读写文件完成的时候，将返回的结果传入第二个参数传入的回调函数之中，这样就实现了Node的异步IO

2. 事件和回调函数

在前端的代码中，事件和回调函数的出现非常频繁，以绑定一个点击事件为例：
```javascript
document.querySelect('body').addEventListener('click', () => {console.log('clicked')})
```
因为此类接口非常契合Node异步IO的思想，Node依旧保留了这一类接口。像之前Node读取文件的时的`fs.readFile`，在读取完成时触发完成事件，将与之相关的参数传入回调函数之中完成操作。  
使用异步思想来编程时，需要对业务进行划分，将不同类的业务划分到不同的“回调函数”之中，由于异步的操作，对流程的控制以及对事件之间的协作可能会造成一些障碍

3. 进程与线程

首先回顾下操作系统进程与线程的概念
- 进程
拥有独立的内存空间和端口，通过进程间通信来完成进程之间的协作
- 线程
任务执行的基本单元，在进程容器中执行，线程之间共享进程的资源比如内存，使用信号量和互斥锁来进行线程之间的协作

Node是单进程多线程的，在V8工作和文件读写的时候都会启动多个线程来进行工作，在Node v10.5.0之后，可以使用`worker_thread`模块来进行线程级的开发。Node的线程与操作系统中的线程不同的是，线程间无法共享内存，线程之间传递数据只能使用进程通信的机制（`postmessage()` / `on('message', callback)`）来传递  

虽然Node是单进程的，但是我们可以通过`child_process`来创建一个子进程，使用Master-Worker的模式，通过父进程和多个子进程的模式来将计算拆分到子进程中计算，通过进程之间的通信来传输数据，这样可以利用起多核CPU，带来更强的健壮性  

## Node的应用场景

1. IO密集型

Node面向网络擅长IO操作，在IO的密集处理上是首屈一指的，Node无需为单独一个请求开启线程，而是使用异步IO和事件循环的能力，能更少的占用资源

2. CPU密集型

在执行效率上，V8的效率是非常高的，在处理CPU密集型的业务时，有以下几个方式来充分利用CPU

- 编写C/C++扩展，将一些V8不用做到极致的计算使用C++来计算
- 使用子进程，将IO与计算分离，利用CPU核数
- 使用`worker_thread`进行多线程的开发