+++
title = "FreeRTOS学习笔记 - API规范记录"
date = "2022-07-22T23:09:17+08:00"
author = "AyalaKaguya"
tags = [
    "FreeRTOS",
    "ESP32",
    "单片机",
    "Arduino",
    "笔记"
]
categories = [
    "单片机",
]
keywords = [
    "FreeRTOS",
    "ESP32",
    "单片机",
    "Arduino"
]
+++

> ❗ 注意
>
> 文章内容较长且多为字典用途，请结合上面的目录使用。

Ayala看b站UP主[孤独的二进制](https://space.bilibili.com/1375767826)的[FreeRTOS](https://space.bilibili.com/1375767826/channel/collectiondetail?sid=419264)教学视频有感，打算对视频内容进行一些总结和归纳，方便以后的使用，遂有此篇。

## 为什么要使用RTOS

（Real-Time Operating System,RTOS）即实时操作系统，通常应用于嵌入式等对实时性要求较高的产品中，它会按照排序运行、管理系统资源，并为开发应用程序提供一致的基础。
实时操作系统与一般的操作系统相比，最大的特色就是“实时性”，如果有一个任务需要执行，实时操作系统会马上（在较短时间内）执行该任务，不会有较长的延时。这种特性保证了各个任务的及时执行。

## FreeRTOS与Arduino框架的区别

在ESP32平台上，Arduino是运行在RTOS上的一个任务（Task），分为设置代码setup()和基础循环loop()，其在RTOS中的表示大概是这样的：

```C
void task_arduino(void* ptParam) {
    setup()
    for (;;) {
        loop()
    }
}

void app_main() {
    xTaskCreatePinnedToCore(task_arduino,"arduino",1024*128,NULL,1,NULL,1)
    // 以上一条代码不需要管
}
```

而一个标准的Arduino的程序看起来是这样的：

```C
void setup() {
    ...
}
void loop() {
    ...
}
```

使用Arduino-ESP32这个库会将上述代码代入前一个代码，所以即便你可能用的是ArduinoIDE，但是实际上仍会被翻译成ESP-IDF项目，所以你可以在你的Arduino-ESP32项目中使用FreeRTOS。

## FreeRTOS有什么API需要注意

> 💬 引用：
>
> 1. b站 [孤独的二进制](https://space.bilibili.com/1375767826)：有相当部分的内容来自于此
>
> 2. ESP32文档和FreeRTOS的文档翻译：函数原型、参数、返回值等的描述
>
> 可能还有一些没标注到的，请在评论区联系我。

### 创建、删除任务，任务绑定核心

#### xTaskCreate

创建任务

```C
BaseType_t xTaskCreate( 
        TaskFunction_t pvTaskCode,
        const char * const pcName,
        configSTACK_DEPTH_TYPE usStackDepth,
        void *pvParameters,
        UBaseType_t uxPriority,
        TaskHandle_t *pxCreatedTask
        );
```

| 参数          | 功能       | 注意点                                                                                               |
| ------------- | ---------- | ---------------------------------------------------------------------------------------------------- |
| pvTaskCode    | 任务函数   | 传入一个函数指针，函数需要一个空类型指针作为参数                                                     |
| pcName        | 任务名     | 传入一个字符串                                                                                       |
| usStackDepth  | 任务栈大小 | 即内存空间，一般的写法为1024*n（n为任务需求的内存大小）                                              |
| pvParameters  | 任务参数   | 使用空类型指针，可以传结构体指针，只要前面加上`(void *)`进行强制类型转换，但是在任务内部仍需转换回来 |
| uxPriority    | 任务优先级 | 一般是0-24，24为最高优先级，如果高优先级任务不block或者暂停，那么低优先级任务永不执行                |
| pxCreatedTask | 任务句柄   | 用于控制这个任务，比如删除、暂停、设置优先级等                                                       |

| 返回值                                | 意义                                                                   |
| ------------------------------------- | ---------------------------------------------------------------------- |
| pdPass                                | 表示任务已经创建成功                                                   |
| errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY | 表示无法创建任务，因为FreeRTOS没有足够的堆内存来分配任务数据结构和堆栈 |

#### xTaskCreatePinnedToCore

在指定核心上创建任务。

当核心不存在时将会在核心0上创建任务。

```C
BaseType_t xTaskCreatePinnedToCore( 
        TaskFunction_t pvTaskCode,
        const char * const pcName,
        configSTACK_DEPTH_TYPE usStackDepth,
        void *pvParameters,
        UBaseType_t uxPriority,
        TaskHandle_t *pxCreatedTask,
        const BaseType_t xCoreID
        );
```

前七个参数等同于前面的`xTaskCreate`，而最后一个参数指定核心，返回值也等同于前面的`xTaskCreate`。

#### vTaskDelete

删除任务

```C
void vTaskDelete( TaskHandle_t pxTaskToDelete );
void vTaskDelete( NULL );
```

当传入指定任务的句柄，将会删除这个任务。当传入NULL，则会删除当前任务。

当对一个已经删除的任务执行`vTaskDelete`将会导致系统重启。

-----

### 任务阻塞（延时）、挂起及从挂起的恢复

#### vTaskDelay

相对延时（阻塞）

```C
void vTaskDelay( TickType_t xTicksToDelay );
```

FreeRTOS的“delay”是基于tick的，有时候1tick并不等于1ms，所以存在常量`portTICK_PERIOD_MS`，这个常量的意义是每毫秒的tick数，在ESP32平台上这个常量默认为1。

当我们需要延时5秒，我们可用写成 `vTaskDelay(5000/portTICK_PERIOD_MS);`

#### vTaskDelayUntil

绝对延时（阻塞）

控制任务能够周期性运行

```C
void vTaskDelayUntil(
        TickType_t *pxPreviousWakeTime, 
        TickType_t xTimeIncrement 
        );
```

| 参数                | 功能                                                                                                                  |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| *pxPreviousWakeTime | 指定一个变量来掌握任务最后开启的时间, 第一次使用时必须使用当前时间来初始化, 在vTaskDelayUntil中，这个变量是自动修改的 |
| xTimeIncrement      | 循环周期时间                                                                                                          |

举个例子：

```C
// 任务将会每10ticks运行一次。
void vTaskFunction( void * pvParameters )
{
    TickType_t xLastWakeTime;
    const TickType_t xFrequency = 10;
    // 用当前时间初始化 xLastWakeTime 变量。
    xLastWakeTime = xTaskGetTickCount(); // 这个函数会返回当前总共从开机经过的tick数
    for( ;; )
    {
        // 等待下一次循环.
        vTaskDelayUntil( &xLastWakeTime, xFrequency );
        // 代码....
    }
}
```

#### vTaskSuspend

挂起一个任务

```C
void vTaskSuspend( TaskHandle_t xTaskToSuspend );
```

xTaskToSuspend：欲挂起任务的句柄

#### vTaskResume

恢复一个挂起的任务

```C
void vTaskResume( TaskHandle_t xTaskToResume );
```

xTaskToResume：欲恢复任务的句柄

#### vTaskSuspendAll

挂起所有任务

```C
void vTaskSuspendAll();
```

#### xTaskResumeAll

恢复所有任务

```C
BaseType_t xTaskResumeAll();
```

-----

### 任务优先级

#### uxTaskPriorityGet

获取任务优先级

```C
UBaseType_t uxTaskPriorityGet( TaskHandle_t xTask );
UBaseType_t uxTaskPriorityGet(NULL);
```

当传入指定任务的句柄时，将会返回那个任务的优先级。而传入`NULL`时，将会返回当前任务的优先级。

#### vTaskPrioritySet

设置任务优先级

```C
void vTaskPrioritySet(  
        TaskHandle_t xTask,
        UBaseType_t uxNewPriority 
        );
```

#### taskYIELD yield

退让资源，任务调度器会重新评估任务，将资源分配给同等级或者更高等级任务

```C
taskYIELD();
yield();
```

-----

### 任务内存

#### ESP32获取总的Heap占用

```C
ESP.getHeapSize() //本程序Heap最大尺寸
ESP.getFreeHeap() //当前Free Heap最大尺寸
```

返回一个类似`uint32_t`类型的值，单位是Byte。

#### uxTaskGetStackHighWaterMark

检查任务从创建好到现在的历史剩余内存的最小值，记作“高水位线”。

```C
UBaseType_t uxTaskGetStackHighWaterMark( TaskHandle_t xTask );
UBaseType_t uxTaskGetStackHighWaterMark( NULL );
```

当传入指定任务的句柄时，将会返回那个任务的“高水位线”。而传入`NULL`时，将会返回当前任务的“高水位线”。

返回一个类似`uint32_t`类型的值，单位是Byte。

-----

### Software Timer - 软件定时器

作为实时操作系统的FreeRTOS给我们提供了一个软件实现的定时器。很多朋友，也许没有使用过Timer。这主要是开发板上硬件Timer太少了，轮不到我们最终用户使用。 FreeRTOS给我们打来了新的大门，只要内存允许，想要多少个Timer 就有多少个Timer。

#### xTimerCreate

创建一个软件定时器

```C
TimerHandle_t xTimerCreate(
        const char * const pcTimerName,
        const TickType_t xTimerPeriodInTicks,
        const UBaseType_t uxAutoReload,
        void * const pvTimerID,
        TimerCallbackFunction_t pxCallbackFunction
        );

void CallbackFunction(TimerHandle_t xTimer); // 回调函数的定义
```

| 参数                | 功能                             |
| ------------------- | -------------------------------- |
| pcTimerName         | 定时器名称（调试用）             |
| xTimerPeriodInTicks | 周期（单位tick）                 |
| uxAutoReload        | 自动装载（pdTURE）               |
| pvTimerID           | 定时器ID，可以判断是哪一个定时器 |
| pxCallbackFunction  | 回调函数                         |

返回定时器的句柄

#### xTimerStart

启动一个软件定时器

```C
BaseType_t xTimerStart(
        TimerHandle_t xTimer,
        TickType_t xToclsToWait
        );
```

| 参数         | 功能         |
| ------------ | ------------ |
| xTimer       | 定时器句柄   |
| xToclsToWait | 阻塞等待时间 |

| 返回值 | 意义                   |
| ------ | ---------------------- |
| pdPass | 表示定时器已经创建成功 |
| pdFAIL | 表示定时器队列已满     |

#### xTimerReset

重启定时器

```C
BaseType_t xTimerReset(
        TimerHandle_t xTimer,
        TickType_t xToclsToWait
        );
```

参数类型、返回值同 `xTimerStart`

- 重启软件定时器
- 如果定时器已经启动，重新计算时间
- 如果没有启动，则启动

#### pvTimerGetTimerID

获取定时器ID

```C
void* pvTimerGetTimerID(TimerHandle_t xTimer);
```

#### xTimerChangePeriod

更改定时器周期

```C
BaseType_t xTimerChangePeriod(
        TimerHandle_t xTimer,
        TickType_t xNewPeriod,
        TickType_t xToclsToWait
        );
```

| 参数         | 功能         |
| ------------ | ------------ |
| xTimer       | 定时器句柄   |
| xNewPeriod   | 新的定时周期 |
| xToclsToWait | 阻塞等待时间 |

如果定时器没启动，会启动定时器

-----

### Watchdog - 看门狗 （ESP平台独占）

看门狗，又叫 watchdog，从本质上来说就是一个定时器。将任务交给看门狗看管后，看门狗会不断的观察任务，如果任务不在指定时间内喂狗。那么，定时器到0，然后狗慌了，ESP32 就自动重启。

看门狗针对于Task任务

- Arduion-ESP32 默认在 Core 0 的 IDLE 任务开启了看门狗 时间为 5000 ticks = 5秒
- Core 0 和 Core 1 都运行了 FreeRTOS的IDLE任务，优先级为 0
- IDLE任务用于清理被删除任务的内存
- Core 1 loopBack任务就是Arduino的 setup 和 loop ，优先级为 1

#### esp_task_wdt.h

在Arduino-ESP32平台上使用看门狗（WDT）需要额外引入一个库：

```C
#include "esp_task_wdt.h"
```

#### esp_task_wdt_add

给任务添加看门狗

```C
esp_err_t esp_task_wdt_add(TaskHandle_t task_handle);
```

任务句柄为`NULL`时则代表本任务

#### esp_task_wdt_delete

取消任务订阅的看门狗

```C
esp_err_t esp_task_wdt_delete(TaskHandle_t task_handle);
```

任务句柄为`NULL`时则代表本任务

#### esp_task_wdt_reset

在任务中重置当前任务绑定看门狗的时间，简称喂狗。

```C
esp_err_t esp_task_wdt_reset(void);
```

#### 如何重置所有的WDT

```C
#include "soc/timer_group_struct.h"
#include "soc/timer_group_reg.h"
void feedTheDogInAllTasks() { //通过寄存器给所有任务的狗喂时
  // feed dog 0
  TIMERG0.wdt_wprotect = TIMG_WDT_WKEY_VALUE; // write enable
  TIMERG0.wdt_feed = 1;                     // feed dog
  TIMERG0.wdt_wprotect = 0;                 // write protect
  // feed dog 1
  TIMERG1.wdt_wprotect = TIMG_WDT_WKEY_VALUE; // write enable
  TIMERG1.wdt_feed = 1;                     // feed dog
  TIMERG1.wdt_wprotect = 0;                 // write protect
}
```

#### 删除指定核心上的所有WDT

```C
//手动关闭CPU上的TWDT - 慎重操作
disableCore0WDT();
disableCore1WDT();
```

-----

### Semaphore - 信号量

信号量通常用于任务间的同步和资源的保护。

#### 全局共享变量

当创建全局变量且要多线程访问时，我们应该使用`volatile`修饰符。

volatile关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改，比如：操作系统、硬件或者其他线程等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

```C
volatile int i=10;
uint32_t j = 12;
```

#### xSemaphoreCreateBinary 二进制信号量

二进制信号量，可以想成是一个布尔类型，只有0 和 1。

```C
SemaphoreHandle_t  xSemaphoreCreateBinary( void );
```

返回值为NULL表示信号量创建失败，否则返回信号量句柄。

新创建的信号量处于无效状态，这意味着使用API函数`xSemaphoreTake()`获取信号之前，需要先给出信号。

二进制信号量和互斥量非常相似，但也有细微的区别：互斥量具有优先级继承机制，二进制信号量没有这个机制。这使得二进制信号量更适合用于同步（任务之间或者任务和中断之间），互斥量更适合互锁。

一旦获得二进制信号量后不需要恢复，一个任务或中断不断的产生信号，而另一个任务不断的取走这个信号，通过这样的方式来实现同步。

低优先级任务拥有互斥量的时候，如果另一个高优先级任务也企图获取这个信号量，则低优先级任务的优先级会被临时提高，提高到和高优先级任务相同的优先级。这意味着互斥量必须要释放，否则高优先级任务将不能获取这个互斥量，并且那个拥有互斥量的低优先级任务也永远不会被剥夺，这就是操作系统中的优先级翻转。

互斥量和二进制信号量都是`SemaphoreHandle_t`类型，并且可以用于任何具有这类参数的API函数中。

#### xSemaphoreCreateCounting 计数信号量

从概念上来说，信号量是一个非负整数计数。 信号量通常用来协调对资源的访问，其中信号计数会初始化为可用资源的数目。 然后，线程在资源增加时会增加计数，在删除资源时会减小计数，这些操作都以原子方式执行。

当计数信号量释放时，如果有任务阻塞在该信号量阻塞队列上，那么任务解除阻塞；但是如果信号量释放时，没有任务阻塞在该信号量阻塞队列上，那么计数器加一。

```C
SemaphoreHandle_t xSemaphoreCreateCounting ( 
        UBaseType_t uxMaxCount,
        UBaseType_t uxInitialCount )
```

| 参数           | 功能                                         |
| -------------- | -------------------------------------------- |
| uxMaxCount     | 最大计数值，当信号到达这个值后，就不再增长了 |
| uxInitialCount | 创建信号量时的初始值                         |

返回值为NULL表示信号量创建失败，否则返回信号量句柄。

#### xSemaphoreCreateMutex 互斥量

有多任务同时写入，或者数据大小超过cpu内存通道时，或者对共享资源的访问时候，需要有防范机制，使用MUTEX对数据对Cirtical Section的内容进行保护，可以想象成MUTEX就是一把锁，共享的资源被锁在了一个箱子里，只有一把钥匙，有钥匙的任务才能对改资源进行访问。

```C
SemaphoreHandle_t xSemaphoreCreateMutex( void );
```

返回值为NULL表示信号量创建失败，否则返回信号量句柄。

二进制信号量和互斥量非常相似，但也有细微的区别：互斥量具有优先级继承机制，二进制信号量没有这个机制。这使得二进制信号量更适合用于同步（任务之间或者任务和中断之间），互斥量更适合互锁。

一旦获得二进制信号量后不需要恢复，一个任务或中断不断的产生信号，而另一个任务不断的取走这个信号，通过这样的方式来实现同步。

低优先级任务拥有互斥量的时候，如果另一个高优先级任务也企图获取这个信号量，则低优先级任务的优先级会被临时提高，提高到和高优先级任务相同的优先级。这意味着互斥量必须要释放，否则高优先级任务将不能获取这个互斥量，并且那个拥有互斥量的低优先级任务也永远不会被剥夺，这就是操作系统中的优先级翻转。

#### xSemaphoreCreateRecursiveMutex 递归互斥量

```C
SemaphoreHandle_t xSemaphoreCreateRecursiveMutex( void );
```

递归类型的互斥量可以被拥有者重复获取。拥有互斥量的任务必须调用API函数xSemaphoreGiveRecursive()将拥有的递归互斥量全部释放后，该信号量才真正被释放。比如，一个任务成功获取同一个互斥量5次，那么这个任务要将这个互斥量释放5次之后，其它任务才能获取到它。

- 递归互斥量具有优先级继承机制，因此任务获得一次信号后必须在使用完后做一个释放操作。
- 互斥量类型信号不可以用在中断服务例程中。

#### xSemaphoreTake

获取信号量，,不可用于中断

```C
BaseType_t xSemaphoreTake(
        SemaphoreHandle_t xSemaphore, 
        TickType_t xTicksToWait
        );
```

| 参数        | 功能                                                     |
| ----------- | -------------------------------------------------------- |
| xSemaphore  | 信号量句柄                                               |
| xTickToWait | 信号量无效时，任务最多等待的时间，单位是系统节拍周期个数 |

| 返回值  | 意义             |
| ------- | ---------------- |
| pdTRUE  | 成功获取到信号量 |
| pdFALSE | 信号量获取失败   |

#### xSemaphoreGive

释放信号量，不可用于中断

```C
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore)
```

| 参数       | 功能       |
| ---------- | ---------- |
| xSemaphore | 信号量句柄 |

| 返回值  | 意义           |
| ------- | -------------- |
| pdTRUE  | 成功释放信号量 |
| pdFALSE | 信号量释放失败 |

#### xSemaphoreTakeFromISR

在中断中获取信号量

```C
BaseType_t xSemaphoreTakeFromISR(
        SemaphoreHandle_t xSemaphore,
        signedBaseType_t *pxHigherPriorityTaskWoken
        );
```

| 参数                       | 功能           |
| -------------------------- | -------------- |
| xSemaphore                 | 信号量句柄     |
| *pxHigherPriorityTaskWoken | 一般设置为NULL |

| 返回值  | 意义             |
| ------- | ---------------- |
| pdTRUE  | 成功获取到信号量 |
| pdFALSE | 信号量获取失败   |

#### xSemaphoreGiveFromISR

在中断中释放信号量

```C
BaseType_t xSemaphoreGiveFromISR(
        SemaphoreHandle_t xSemaphore,
        signed BaseType_t *pxHigherPriorityTaskWoken 
        );
```

| 参数                       | 功能           |
| -------------------------- | -------------- |
| xSemaphore                 | 信号量句柄     |
| *pxHigherPriorityTaskWoken | 一般设置为NULL |

| 返回值  | 意义           |
| ------- | -------------- |
| pdTRUE  | 成功释放信号量 |
| pdFALSE | 信号量释放失败 |

#### xSemaphoreTakeRecursive

获取递归互斥量

```C
BaseType_t xSemaphoreTakeRecursive(
        SemaphoreHandle_t xMutex, 
        TickType_t xTicksToWait 
        );
```

| 参数        | 功能                                                               |
| ----------- | ------------------------------------------------------------------ |
| xMutex      | 信号量句柄，必须是由`xSemaphoreCreateRecursiveMutex()`创建的信号量 |
| xTickToWait | 信号量无效时，任务最多等待的时间，单位是系统节拍周期个数           |

| 返回值  | 意义             |
| ------- | ---------------- |
| pdTRUE  | 成功获取到信号量 |
| pdFALSE | 信号量获取失败   |

#### xSemaphoreGiveRecursive

释放递归互斥量

```C
BaseType_t xSemaphoreGiveRecursive(SemaphoreHandle_t xMutex );
```

| 参数   | 功能                                                             |
| ------ | ---------------------------------------------------------------- |
| xMutex | 信号量句柄，必须由`xSemaphoreCreateRecursiveMutex()`创建的信号量 |

| 返回值  | 意义           |
| ------- | -------------- |
| pdTRUE  | 成功释放信号量 |
| pdFALSE | 信号量释放失败 |

#### vSemaphoreDelete

删除信号量

```C
void vSemaphoreDelete( SemaphoreHandle_t xSemaphore );
```

-----

### Queue - 队列

队列是一种数据结构，可以包含一组固定大小的数据。在创建队列的同时，队列的长度和所包含数据类型的大小就确认下来了。一个队列可以有多个写入数据的任务和多个读取数据的任务。当一个任务试图从队列读取数据的时候，它可以设置一个阻塞时间（block time）。这是当队列数据为空时，任务会进入阻塞状态的时间。当有数据在队列或者到达阻塞时间的时候，任务都会进入就绪状态。如果有多个任务同时在阻塞状态等待队列数据，优先级高的任务会在数据到达时进入就绪状态；在优先级相同的时候，等待时间长的任务会进入就绪状态。同理可以推及多个任务写入数据时候的运行状态。

#### xQueueCreate

创建一个队列，为队列动态分配所需的内存空间。

```C
QueueHandle_t xQueueCreate( 
        UBaseType_t uxQueueLength,
        UBaseType_t uxItemSize 
        );
```

| 参数          | 功能                                   |
| ------------- | -------------------------------------- |
| uxQueueLength | 队列能够存储的最大单元数目，即队列深度 |
| uxItemSize    | 队列中数据单元的长度，以字节为单位     |

#### xQueueSend

向队列尾部发送一个队列消息。消息以拷贝的形式入队，而不是以引用的形式。

```C
BaseType_t xQueueSend(
        QueueHandle_t xQueue,
        const void * pvItemToQueue,
        TickType_t xTicksToWait
        );
```

| 参数          | 功能                                                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| xQueue        | 目标队列的句柄。这个句柄即是调用 `xQueueCreate()` 创建该队列时的返回值                                                                                       |
| pvItemToQueue | 发送数据的指针。其指向将要复制到目标队列中的数据单元。由于在创建队列时设置了队列中数据单元的长度，所以会从该指针指向的空间复制对应长度的数据到队列的存储区域 |
| xTicksToWait  | 队列满时，等待队列空闲的最大超时时间。如果队列满并且 xTicksToWait 被设置成 0，函数立刻返回                                                                   |

| 返回值  | 意义             |
| ------- | ---------------- |
| pdTRUE  | 消息发送成功成功 |
| pdFALSE | 消息发送成功失败 |

#### xQueueReceive

从一个队列中接收消息并把消息从队列中删除。接收的消息是以拷贝的形式进行的，所以必须提供一个足够大空间的缓冲区。具体能够拷贝多少数据到缓冲区，这个在队列创建的时候已经设定。

```C
BaseType_t xQueueReceive(
        QueueHandle_t xQueue,
        void *pvBuffer,
        TickType_t xTicksToWait
        );
```

| 参数         | 功能                                                                   |
| ------------ | ---------------------------------------------------------------------- |
| xQueue       | 被读队列的句柄。这个句柄即是调用 `xQueueCreate()` 创建该队列时的返回值 |
| pvBuffer     | 接收缓存指针。其指向一段内存区域，用于接收从队列中拷贝来的数据         |
| xTicksToWait | 队列空时，阻塞超时的最大时间。如果该参数设置为 0，函数立刻返回         |

| 返回值  | 意义           |
| ------- | -------------- |
| pdTRUE  | 队列项接收成功 |
| pdFALSE | 队列项接收失败 |

-----

### Stream Buffer - 流缓冲区

使用Stream Buffer 对流媒体数据，在任务间进行传输流媒体，读和写的大小都没有任何的限制，读和写的大小可以不一致， 比如写入100 bytes, 可以分成两次每次50 bytes读取出来。

适合于一个任务写，另外一个任务读，并不适合多任务读写（当然你也可以使用MUTEX）。

#### xStreamBufferCreate

创建流缓冲区

```C
StreamBufferHandle_t xStreamBufferCreate( 
        size_t xBufferSizeBytes,
        size_t xTriggerLevelBytes
        );
```

| 参数               | 功能                                                                         |
| ------------------ | ---------------------------------------------------------------------------- |
| xBufferSizeBytes   | 流缓冲区在任何时候都能够保存的总字节数                                       |
| xTriggerLevelBytes | 在流缓冲区上阻塞以等待数据的任务移出阻塞状态之前，流缓冲区中必须包含的字节数 |

#### xStreamBufferSend

将字节发送到流缓冲区

```C
size_t xStreamBufferSend(
        StreamBufferHandle_t xStreamBuffer,
        const void * pvTxData,
        size_t xDataLengthBytes,
        TickType_t xTicksToWait
        );
```

| 参数             | 功能                                                                                                                             |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| xStreamBuffer    | 正在向其发送流的流缓冲区的句柄                                                                                                   |
| pvTxData         | 指向缓冲区的指针，该缓冲区保存要复制到流缓冲区中的字节                                                                           |
| xDataLengthBytes | 从pvTxData复制到流缓冲区的最大字节数                                                                                             |
| xTicksToWait     | 如果流缓冲区包含的空间太小而无法容纳另一个`xDataLengthBytes`字节，则任务应保持在阻塞状态以等待流缓冲区中有足够空间可用的最长时间 |

返回写入流缓冲区的字节数。如果一个任务超时，它可以将所有`xDataLengthBytes`写入缓冲区，它仍将写入尽可能多的字节。

#### xStreamBufferReceive

从流缓冲区接收字节

```C
size_t xStreamBufferReceive(
        StreamBufferHandle_t xStreamBuffer,
        void * pvRxData,
        size_t xBufferLengthBytes,
        TickType_t xTicksToWait
        );
```

| 参数               | 功能                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| xStreamBuffer      | 要从中接收字节的流缓冲区的句柄                                                                                                                          |
| pvRxData           | 指向将要复制接收字节的缓冲区的指针                                                                                                                      |
| xBufferLengthBytes | `pvRxData`参数指向的缓冲区长度。这设置了一次调用中要接收的最大字节数。 `xStreamBufferReceive`将返回尽可能多的字节，直到`xBufferLengthBytes`设置的最大值 |
| xTicksToWait       | 如果流缓冲区为空，任务应保持在阻塞状态以等待数据变为可用的最长时间                                                                                      |

返回实际从流缓冲区读取的字节数，如果在`xBufferLengthBytes`可用之前对`xStreamBufferReceive()`的调用超时，则该字节数将小于`xBufferLengthBytes`。

#### xStreamBufferIsFull

查询流缓冲区以查看它是否已满。 如果流缓冲区没有任何可用空间，则它已满，因此无法再接受任何数据。

```C
BaseType_t xStreamBufferIsFull(StreamBufferHandle_t xStreamBuffer);
```

| 参数          | 功能                         |
| ------------- | ---------------------------- |
| xStreamBuffer | 正在查询的查询流缓冲区的句柄 |

| 返回值  | 意义         |
| ------- | ------------ |
| pdTRUE  | 流缓冲区已满 |
| pdFALSE | 流缓冲区未满 |

#### xStreamBufferBytesAvailable

查询流缓冲区以查看它包含多少数据，这等于在流缓冲区为空之前可以从流缓冲区读取的字节数。

```C
size_t xStreamBufferBytesAvailable(StreamBufferHandle_t xStreamBuffer);
```

| 参数          | 功能                         |
| ------------- | ---------------------------- |
| xStreamBuffer | 正在查询的查询流缓冲区的句柄 |

返回在查询流缓冲区为空之前可以从查询流缓冲区读取的字节数，即包含多少数据。

#### xStreamBufferSpacesAvailable

查询流缓冲区以查看它包含多少可用空间，这等于在流缓冲区满之前可以发送到流缓冲区的数据量。

```C
size_t xStreamBufferSpacesAvailable(StreamBufferHandle_t xStreamBuffer);
```

| 参数          | 功能                         |
| ------------- | ---------------------------- |
| xStreamBuffer | 正在查询的查询流缓冲区的句柄 |

返回在流缓冲区满之前可以写入流缓冲区的字节数。

-----

### Message Buffer - 消息缓冲区

Message Buffer基于Stream Buffer实现, 在传输的时候使用4个字节记录了发送内容大小，这样在读取时，也可以一次读取对应大小的数据，所以很适合串口接收和发送数据，每次的大小不定，但是接受和发送的数据量需要相同。

#### xMessageBufferCreate

创建消息缓冲区

```C
MessageBufferHandle_t xMessageBufferCreate(size_t xBufferSizeBytes);
```

| 参数             | 功能                                                                                                                             |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| xBufferSizeBytes | 消息缓冲区在任何时候都能保存的总字节数（而不是消息）。当消息写入消息缓冲区时，还会写入另外的sizeof（size_t）字节来存储消息的长度 |

#### xMessageBufferSend

将离散消息发送到消息缓冲区。消息可以是适合缓冲区可用空间的任何长度，并被复制到缓冲区中。

```C
size_t xMessageBufferSend(
        MessageBufferHandle_t xMessageBuffer,
        const void * pvTxData,
        size_t xDataLengthBytes,
        TickType_t xTicksToWait);
```

| 参数             | 功能                                                                                         |
| ---------------- | -------------------------------------------------------------------------------------------- |
| xMessageBuffer   | 正在向其发送消息的消息缓冲区的句柄                                                           |
| pvTxData         | 指向要复制到消息缓冲区中的消息的指针                                                         |
| xDataLengthBytes | 消息的长度                                                                                   |
| xTicksToWait     | 如果消息缓冲区空间不足，则调用任务应保持在阻塞状态以等待消息缓冲区中有足够空间可用的最长时间 |

返回写入消息缓冲区的字节数。如果在有足够的空间将消息写入消息缓冲区之前对`xMessageBufferSend()`的调用超时，则返回零。如果调用没有超时，则返回`xDataLengthBytes`。

#### xMessageBufferReceive

从消息缓冲区接收离散消息。消息可以是可变长度的，并从缓冲区中复制出来。

```C
size_t xMessageBufferReceive(
        MessageBufferHandle_t xMessageBuffer,
        void * pvRxData,
        size_t xMessageSizeMax,
        TickType_t xTicksToWait
        );
```

| 参数            | 功能                                                           |
| --------------- | -------------------------------------------------------------- |
| xMessageBuffer  | 从中接收消息的消息缓冲区的句柄                                 |
| pvRxData        | 指向要将复制的消息复制到的缓冲区的指针                         |
| xMessageSizeMax | 获取消息的最大长度                                             |
| xTicksToWait    | 如果消息缓冲区为空，则任务应保持在阻塞状态以等待消息的最长时间 |

返回从消息缓冲区读取的消息的长度（以字节为单位）（如果有）。如果`xMessageBufferReceive()`在消息可用之前超时，则返回零。如果消息的长度大于`xBufferLengthBytes`，则消息将保留在消息缓冲区中并返回零。

#### xMessageBufferIsFull

查询消息缓冲区以查看它是否已满

```C
BaseType_t xMessageBufferIsFull(MessageBufferHandle_t xMessageBuffer);
```

| 参数           | 功能                       |
| -------------- | -------------------------- |
| xMessageBuffer | 正在查询的消息缓冲区的句柄 |

| 返回值  | 意义         |
| ------- | ------------ |
| pdTRUE  | 消息缓冲区已满 |
| pdFALSE | 消息缓冲区未满 |

-----

### Direct Task Notification - 直接任务通知

Direct Task Notification是FreeRTOS 10版本以后的最重要的一个功能。可以实现二进制信号量、计数信号量、事件组、邮箱等功能，而且速度快45%，占用更少的内存。

- 一个任务可以有多个notification
- 每个notification包含4个字节的value 和 1个字节的stats
- stats用来记录当前的notification有没有被处理 pending or not pending
- 我们不能对stats进行直接的读写操作，是系统自动的
- 我们只能对notification value 进行操作

#### xTaskNotify

向一个任务发送通知

```C
BaseType_t xTaskNotify( 
        TaskHandle_t xTaskToNotify,
        uint32_t ulValue，
        eNotifyAction eAction 
        );
```

| 参数          | 功能             |
| ------------- | ---------------- |
| xTaskToNotify | 被通知任务的句柄 |
| ulValue       | 通知的操作值     |
| eAction       | 通知的操作方法   |

| eNotifyAction             | 作用                          |
| ------------------------- | ----------------------------- |
| eIncrement                | 累加                          |
| eNoAction                 | 什么也不做                    |
| eSetBits                  | 按位取与，实现设置某一位值为1 |
| eSetValueWithOverwrite    | 覆盖                          |
| eSetValueWithoutOverwrite | 如果前一个通知已处理则覆盖    |

| 返回值  | 意义             |
| ------- | ---------------- |
| pdTRUE  | 直接任务通知成功 |
| pdFALSE | 直接任务通知失败 |

#### xTaskNotifyWait

等待一个通知

```C
BaseType_t xTaskNotifyWait( 
        uint32_t ulBitsToClearOnEntry,
        uint32_t ulBitsToClearOnExit,
        uint32_t *pulNotificationValue,
        TickType_t xTicksToWait 
        );
```

| 参数                 | 功能                                                   |
| -------------------- | ------------------------------------------------------ |
| ulBitsToClearOnEntry | 接收通知前value区保存的内容，一般为`0x00`              |
| ulBitsToClearOnExit  | 接收通知后value区保存的内容，一般为`0x00`或`ULONG_MAX` |
| pulNotificationValue | 产生通知时，value区转存的缓冲区，用于接收通知的内容    |
| xTicksToWait         | 在没有通知时最大的等待时间                             |

| 返回值  | 意义         |
| ------- | ------------ |
| pdTRUE  | 等待通知成功 |
| pdFALSE | 等待通知失败 |

-----

### 常用公共API

接下来的一部分可能得等到我想到再填了（恼）

## 后记

这次的笔记真的很长呢，资料也东拼西凑了很多，虽然大部分我都亲自试验过了，但难免笔记中存在一些问题和谬误，如果您发现了这些问题，请及时在下方评论区中提出，我会第一时间更改，谢谢大家了。
