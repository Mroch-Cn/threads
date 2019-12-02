# threads

## 线程
进程是操作系统中进行保护和资源分配的基本单位，操作系统分配资源以进程为基本单位。  而线程是进程的组成部分，它代表了一条顺序的执行流。
进程从操作系统获得基本的内存空间，所有的线程共享着进程的内存地址空间。当然，每个  线程也会拥有自己私有的内存地址范围，其他线程不能访问。

由于所有的线程共享进程的内存地址空间，所以线程间的通信就容易的多，通过共享进程级  全局变量即可实现。

同时，在没有引入多线程概念之前，所谓的『并发』是发生在进程之间的，每一次的进程上  下文切换都将导致系统调度算法的运行，以及各种 CPU 上下文的信息保存，非常耗  时。而线程级并发没有系统调度这一步骤，进程分配到 CPU 使用时间，并给其内部的各个  线程使用。

在分时系统中，进程中的每个线程都拥有一个时间片，时间片结束时保存 CPU 及寄存器中  的线程上下文并交出 CPU，完成一次线程间切换。当然，当进程的 CPU 时间使用结束  时，所有的线程必然被阻塞。  

JAVA API 中用 Thread 这个类抽象化描述线程，线程有几种状态：  NEW：线程刚被创建
RUNNABLE：线程处于可执行状态  
BLOCKED、WAITING：线程被阻塞  
TERMINATED：线程执行结束，被终止      
其中 RUNNABLE 表示的是线程可执行，但不代表线程一定在获取 CPU 执行中，可能由于时间片使用结  束而等待系统的重新调度。BLOCKED、WAITING 都是由于线程执行过程中缺少某些条件而暂时阻塞，一旦它们等  待的条件满足时，它们将回到 RUNNABLE 状态重新竞争 CPU。

## 协程
协程是什么？  
和线程类似，是一种在程序开发中处理多任务的组件。  
协程就像是一种轻量级的线程  
协程很像线程，但他不是线程。  
协程是用户态的，他的切换不需要和操作系统交互，因此协程切换的成本笔线程低。  
协程由于是协作式的，所以不需要线程的同步操作。  

其实协程就是一套官方封装的线程API，方便使用。线程框架
它可以用同步的方式写出异步的代码  (非阻塞式挂起)
```kotlin
val user = api.getUser()
nameTv.text = user.name
```

最基本的功能是并发
```
launch(Dispachers.IO){
    saveToDatabase(data)
}
launch(Dispacher.Main){
    updateViews(data)
}
```

最大的优势是把不同的代码写在同一个代码块中
```
launch(Dispachers.Main){//开始：主线程
    val user = api.getUser()//网络请求：后台线程
    nameTv.text = user.name //更新ui：主线程
}
```
有两个接口,无相关性，都得到的话请求需要回调先后请求（网络事件增长），在协程中就可以并发执行,消除并发代码的难度
```
//API 1：获取用户头像
api.getAvatar(user)
//API 2: 获取用户所在公司的logo
api.getCompanyLogo(user)
```
```协程
launch(Dispachers.Main){
    val avatar = async{api.getAvatar(user)}//获取用户头像
    val logo = async{api.getCompanyLogo(user)}//获取公司的logo
    val merge = suspendingMerge(avatar, logo)//合并
    show(merge)//显示
}
```
介绍协程使用，上下关系代码实现多线程之间协作
```
launch(Dispachers.Main){
    val image = withContext(Dispachers.IO){//指定在IO线程上执行代码
        getImage(imageId)
    }//执行完之后自动切回来，继续执行
    avatarIv.setImageBitmao(image)
}
```
## suspend函数
挂起函数，协程中最核心的关键词。非阻塞式挂起，不会阻塞线程，兵分两路（暂时切走，稍后切回来）  
线程跳过往下执行，协程把任务切到指定线程中运行，运行完之后自动切回来。  
稍后会被自动切回来的线程切换  
要么在协程中被调用要么在另一个挂起函数中被调用。（为了线程在执行协程的过程中切过去之后再切回来）  
作用是提示这是一个耗时函数。 

## 非阻塞式
本质是不卡线程。切走线程，主线程就不卡了。用thread或者线程池也不卡（也是非阻塞式）  
单线程的耗时操作卡线程。  
主线程空置出来，让IO线程去在后台操作


```
import kotlinx.coroutines.*

fun mail() = runBlocking{
    repeat(100000){
        launch{
            delay(1000L)
            print(".")
        }
    }
}
```
用线程多半会内存溢出，但是本质上是把这一万次任务放进线程池来做事。对比线程代码

```
repeat(100000){
    thread{
        Thread.sleep(1000L)
        print(".")
    }
}
```
但其实协程使用应该和ExecutorService那几个类来做比较


