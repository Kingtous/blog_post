---
layout: post
title: 安卓跨进程通信-Binder_AIDL
subtitle: aidl
author: Kingtous
date: 2021-01-11 18:50:05
header-img: img/home-bg-o.jpg
catalog: True
tags:
- Android
---

此处简单讲一个例子：

通过跨进程通信，获取一个自增的数字，3s一次轮询，显示在另一个进程的屏幕上。

安卓是基于Linux，[那为什么Android要采用Binder作为IPC机制呢 By GitYuan](https://www.zhihu.com/question/39440766)

#### 定义AIDL统一接口

此处定义一个`queryNumber`的函数，函数调用后返回一个int值

```aidl
// IMyAidlInterface.aidl
package com.example.mediatestactivity;

// Declare any non-default types here with import statements

interface IMyAidlInterface {

    int queryNumber();
}
```

点击AS的build后，会自动生成对应的java类（AIDL简化编程，当然也可以自己写Binder机制，实现onTransact等方法，这个就是高级运用了 ，此处不涉及）

**注意：**除下参数可以为java的基础数据类型，所传的类都应该是可以序列化的。这也符合C/S架构的通信原理

#### 定义服务

Binder是典型的C/S架构的体现，相比Linux原生的方法相比，由于数据拷贝次数只需要1次，故性能仅次于共享内存，同时针对安卓平台增强了安全性，相比共享内存来说，约束了数据权限。

一个简单的服务定义如下：

```kotlin
/**
 * 服务端
 */
class MyAidlInterfaceService : Service() {

    companion object {
        val number : AtomicInteger = AtomicInteger(0)
    }

    private val interfaceService: IMyAidlInterface.Stub by lazy {
        object : IMyAidlInterface.Stub() {
            override fun queryNumber(): Int {
                return number.incrementAndGet();
            }
        }
    }

    override fun onBind(intent: Intent?): IBinder {
        return interfaceService.asBinder()
    }

}
```

number使用提供的进程/线程安全的AtomicInteger，从0开始自增。（在自增时，会添加额外的无意义的JVM指令控制执行逻辑）

onBind方法在Service被连接时调用，给用户返回一个针对AIDL接口的Binder客户端。

此外，需要在AndroidManifest.xml中注册该Service，并定义合适的action以及filter。其中action要告知客户端，进行连接

```xml
<service android:name=".MyAidlInterfaceService" android:process="${applicationId}.countServer" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.countServer"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</service>
```

#### 启动服务

启动服务与Activity相同，使用startxxx来代替

```kotlin
val intent = Intent(this,MyAidlInterfaceService::class.java)
startService(intent)
```

#### 客户端连接与逻辑配置

- 连接上Binder服务

使用Context提供的bindService进行服务绑定

```kotlin
...
bindService(Intent(SERVER_ADDRESS).apply {
    `package` = packageName
}, connection, Context.BIND_AUTO_CREATE)
...
```

注意：安卓5.0+后需要apply客户端包名，进一步增加了安全性。由此服务端可以通过包名来限制部分非法访问

connection参数为连接监听，此处设置：

```kotlin
private val connection: ServiceConnection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
        server = IMyAidlInterface.Stub.asInterface(service)
        executor.scheduleWithFixedDelay(runnable, 0, 3, TimeUnit.SECONDS)
    }

    override fun onServiceDisconnected(name: ComponentName?) {
        executor.shutdown()
    }
}
```

在service连接后，IBinder转化为可供调用的接口IMyAidlInterface，并通过提供的ScheduleWithFixedDelay方法进行周期性调度

定义的执行Runnable：

```kotlin
private val runnable: Runnable = Runnable {
        server?.let {
            runOnUiThread {
                showText.text =  "${it.queryNumber()}"
            }
        }
    }
```

设置好了，来运行起来即可。

