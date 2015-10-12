# Hello Word

本文示范来如何连通leap motion控制器以及如何获取基本追踪数据。阅读本文和自己实践后，你会为开始你的开发之路打下一个坚实的基础。

> This article demonstrates how to connect to the Leap Motion controller and access basic tracking data. After reading this article and following along with your own basic program, you should have a solid foundation for beginning your own application development.

首先是个小背景资料：

## Leap Motion控制器是如何工作的
LM控制器包含硬件和软件两部分。
> The Leap Motion controller encompasses both hardware and software components.

LM硬件部分基本上由一对立体红外相机和照明LED组成。相机传感器朝向上方（当设备处于它的标准位置时）。下图表明了LM传感器如何监测用户的手。
> The Leap Motion hardware consists primarily of a pair of stereo infrared cameras and illumination LEDs. The camera sensors look upward (when the device is in its standard orientation). The following illustration shows how a user’s hands look from the perspective of the Leap Motion sensor:

![](https://di4564baj7skl.cloudfront.net/documentation/images/Leap_View.jpg)

LM软件接收到传感器的数据并分析这些有关手、手指、手臂、工具的数据（追踪其它物体的功能在以后可能会被加入，目前这些就是全部能被追踪的）。软件会将人手内模与接收到的传感器数据进行对比来确定最佳匹配。传感器数据以帧为单位，将每一帧的数据发送给LM应用程序。你的程序收到的Frame object包含了全部的已知位置、向量以及全部追踪到的身份（比如手、手指、工具）。像手势、动作这种横跨多个帧的也会在每一帧被更新。想要总览一下控制器获取的追踪数据，可以看[API Overview](https://developer.leapmotion.com/documentation/python/devguide/Leap_Overview.html)。

> The Leap Motion software receives the sensor data and analyzes this data specifically for hands, fingers, arms, and tools. (Tracking for other types of objects could be added in the future, but this is the current set of tracked entities). The software maintains an internal model of the human hand and compares that model to the sensor data to determine the best fit. Sensor data is analyzed frame-by-frame and the service sends each frame of data to Leap Motion-enabled applications. The Frame object received by your application contains all the known positions, velocities and identities of tracked entities, such as hands, fingers, and tools. Constructs such as gestures and motions that span multiple frames are also updated each frame. For an overview for the tracking data provided by the controller, read API Overview.

LM软件会作为一个服务（windows上）或后台（MAC和Linux上）运行在电脑上。原生LM应用可以通过Leap Motion动态库（SDK里的一部分）用API连接到这个服务。网页应用可以连接一个WebSocket服务器。WebSocket给出一个JSON格式消息的追踪数据，每个消息是一帧数据。LeapJS，一个javascript库，有个包含这些数据的API。更多信息可以看[系统架构](https://developer.leapmotion.com/documentation/python/devguide/Leap_Architecture.html)。

> The Leap Motion software runs as a service (Windows) or daemon (Mac and Linux) on the client computer. Native Leap Motion-enabled applications can connect to this service using the API provide by the Leap Motion dynamic libraries (provided as part of the Leap Motion SDK). Web applications can connect to a WebSocket server hosted by the service. The WebSocket provides tracking data as a JSON-formatted message – one message per frame of data. A JavaScript library, LeapJS, provides an API wrapping this data. For more information read System Architecture.

## 设置文件

本教程使用命令行编译器和接口（需要时）以便专注于代码而非环境。关于如何在IDE里用SDK建立项目的细节，可以看[项目建立](https://developer.leapmotion.com/documentation/python/devguide/Project_Setup.html)。

> This tutorial also uses command line compilers and linkers (where needed) in order to focus on the code rather than the environment. For details on how to set up projects using the Leap Motion SDK in popular IDEs, see Setting Up a Project.

### 1.
先从[开发者网站](https://developer.leapmotion.com/)下载解压最新的SDK然后安装最新的Leap Motion服务。 If you haven’t already, download and unzip the latest Leap Motion SDK from the developer site and install the latest Leap Motion service.

### 2.
打开终端或者控制台，进入SDK示例文件夹。Open a terminal or console window and navigate to the SDK samples folder.

### 3.
`Sample.py`包含本教程的最终代码。但是你也可以重命名文件并在这个文件夹中创建一个空的 Sample.py文件。`Sample.py` contains the finished code for this tutorial, but to get the most out of this lesson, you can rename the existing file, and create a new, blank Sample.py file in this folder.

### 4.
在你的新Sample.py程序中，引入LeapMotion库。下面的代码用来判断你是用的32位还是64位Python并引入正确的库。 In your new Sample.py program, add code to import the Leap Motion libraries. The following code detects whether you are running 32- or 64-bit Python and loads the proper library:

```
import os, sys, inspect, thread, time
src_dir = os.path.dirname(inspect.getfile(inspect.currentframe()))
arch_dir = '../lib/x64' if sys.maxsize > 2**32 else '../lib/x86'
sys.path.insert(0, os.path.abspath(os.path.join(src_dir, arch_dir)))

import Leap
from Leap import CircleGesture, KeyTapGesture, ScreenTapGesture, SwipeGesture
```

### 5.
加上结构语句来定义python命令。
Add the “structural” code to define a Python command-line program:

```
def main():

    # Keep this process running until Enter is pressed
    print "Press Enter to quit..."
    try:
        sys.stdin.readline()
    except KeyboardInterrupt:
        pass

if __name__ == "__main__":
    main()
```
要注意`sys.stdin.readline()`在IDLE和其他一些IDE中执行的不是很好，你必须用其他方式来保证程序能执行到main()的结尾使程序能立即退出。
> Note that the statement: `sys.stdin.readline()` does not play nicely with IDLE and possibly other IDEs. To use IDLE, you must implement a different way to prevent the program from reaching the end of the main() subroutine and thus exiting immediately.

这段代码简单地显示了一条信息然后等待键盘输入来退出。文章末尾的“运行示例”会教你怎么运行它。
> This code simply prints a message and then waits for keyboard input before exiting. See Running the Sample for instructions on running the program.

## 进行连接

下一步就是在程序中加入一个 `Controller` object ，它能连接上Leap Motion服务/后台。
> The next step is to add a Controller object to the program – which serves as our connection to the Leap Motion service/daemon.

```
def main():
     controller = Leap.Controller()

     # Keep this process running until Enter is pressed
     print "Press Enter to quit..."
     try:
         sys.stdin.readline()
     except KeyboardInterrupt:
         pass
```

当你创建了`Controller` object，它会自动与LM服务连接，一旦连接建立，你就可以用`Controller.frame()`函数来获得追踪的数据。
> When you create a `Controller` object, it automatically connects to the Leap Motion service and, once the connection has been established, you can get tracking data from it using the `Controller.frame()` method.

连接进程是异步的，因此你不可能在一行创建`Controller`然后希望在下一行获得数据。你必须等待连接完成。但是，这要多长时间？
> The connection process is asynchronous, so you can’t create the Controller in one line and expect to get data in the next line. You have to wait for the connection to complete. But for how long?

## 听还是不听

你可以给`Controller`加入一个`Listener` object ，这提供了一个基于事件的机制来响应重要的Controller状态变化。这就是我们这个教程中用到的通道，但它也不总是最好的通道。
> You can add a Listener object to the Controller, which provides an event-based mechanism for responding to important Controller state changes. This is the approach used in this tutorial – but it is not always the best approach.


**Listener的问题:** `Listener` object 用了独立的线程来执行事件中的代码。因此，使用Listener机制会将复杂的线程带入你的程序中。你有责任确保有一个线程安全的方式让你的Listener子类中实现代码访问程序的其他部分。比如，你一般不能从除主线外其他地方访问和GUI控制相关的变量。创建和清理进程也有附加的上层联系。
> The Problem with Listeners: Listener objects use independent threads to invoke the code that you implement for each event. Thus, using the listener mechanism can introduce the complexities of threading into your program. It becomes your responsibility to make sure that the code you implement in your Listener subclass accesses other parts of your program in a thread-safe manner. For example, you might not be able to access variables related to GUI controls from anything except the main thread. There can also be additional overhead associated with the creation and cleanup of threads.

**避免Listener：** 当方便的情况下，你也可以只是通过简单地查看每一帧的Controller object来避免使用Listener。很多程序已经有事件循环或动态循环来驱动用户输入和动态。如果是这样，你可以每一个循环获取一次追踪数据，这样一般也跟你使用数据的速度一样快。

> Avoiding Listeners: You can avoid using Listener objects by simply polling the Controller object for frames (or other state) when convenient for your program. Many programs already have a event- or animation-loop to drive user input and animation. If so, you can get the tracking data once per loop – which is often as fast as you can use the data anyway.

Listener class在API中定义了一个功能的签名，这个功能会在Controller事件发生时被引用。你通过创建一个Listener的子类并执行回调函数来为你感兴趣的事件创建一个listener。

> The Listener class in the API defines the signatures for a function that will be called when a Controller event occurs. You create a listener by creating a subclass of Listener and implementing the callback functions for the events you are interested in.

继续教程，加入`SampleListener` class到你的程序中
To continue this tutorial, add the SampleListener class to your program:

```
class SampleListener(Leap.Listener):

    def on_connect(self, controller):
        print "Connected"


    def on_frame(self, controller):
        print "Frame available"

def main():
    #...
```

如果你已经看了完成的文件，你可能注意到它有很多回调函数。你可以也加上它们，如果你想的话，但是要保证简洁，我们会着眼于`on_connect()`和`on_frame()`。

> If you have already taken a look at the finished file, you may have noticed that several more callback functions are present. You can add those too, if you wish, but to keep things simple, we will concentrate on [on_connect()](https://developer.leapmotion.com/documentation/python/api/Leap.Listener.html#Leap.Listener.on_connect) and [on_frame()](https://developer.leapmotion.com/documentation/python/api/Leap.Listener.html#Leap.Listener.on_frame).

现在用新的类创建一个SampleListener object并把它加入你的controller中 Now create a SampleListener object using your new class and add it to your controller:

```
def main():
    # Create a sample listener and controller
    listener = SampleListener()
    controller = Leap.Controller()

    # Have the sample listener receive events from the controller
    controller.add_listener(listener)

    # Keep this process running until Enter is pressed
    print "Press Enter to quit..."
    try:
        sys.stdin.readline()
    except KeyboardInterrupt:
        pass
    finally:
        # Remove the sample listener when done
        controller.remove_listener(listener)
```
现在该测试你的程序了。
Now is a good time to test your sample program.

如果一切都正确，你会看到终端窗口上出现“connected”然后是一堆“Frame Available”。如果出错了你也弄不明白，可以上我们的开发者网站的论坛问问。<br>
当你的开发出现问题时，可以打开可视化诊断看看。它会显示LM的追踪数据。你可以对比你的程序的现实和可视化中的（这俩使用的API是同样的）来判断出了什么问题。

> If everything is correct (and your Leap Motion hardware is plugged in), then you should see the string “Connected” printed to the terminal window followed by an rapid series of “Frame available”. If things go wrong and you cannot figure out why, you can get help on our developer forum at developer.leapmotion.com.<br>
> Whenever you run into trouble developing a Leap Motion application, try opening the diagnostic visualizer. This program displays a visualization of the Leap Motion tracking data. You can compare what you see in your program to what you see in the visualizer (which uses the same API) to isolate and identify many problems.


## 连接成功后

当你的`Controller` object成功连接上LM服务，同时硬件部分也成功连接后，Controller object会将它的`is_connected`属性变为true，并让你`Listener.on_connect()`的执行回调（如果有的话）。

> When your Controller object successfully connects to the Leap Motion service/daemon AND the Leap Motion hardware is plugged in, The Controller object changes its is_connected property to true and invokes your Listener.on_connect() callback (if there is one).

当控制器连接了，你可以用`Controller.enable_gesture()`和`Controller.set_policy()`来设置控制器属性。比如你可以用下面的`onConnect()`函数来让扫动手势能被识别：

> When the controller connects, you can set controller properties using the Controller.enable_gesture() and Controller.set_policy() methods. For example, you could enable the swipe gesture with the following onConnect() function:

```
def on_connect(self, controller):
    print "Connected"
    controller.enable_gesture(Leap.Gesture.TYPE_SWIPE);
```

## 帧

LM系统中的所有追踪数据都是通过`Frame` object来传递的。你可以通过调用`Controller.frame()`从你的控制器（当你连接成功后）来获得`Frame` objects。当新的一帧数据可用时，`on_frame()`会被调用来回调你的Listener子类。当你不用listener时，你可以比较你处理的最后一帧的id值来看是否有新一帧。通过设置`frame()`函数的历史参数，你可以得到比目前这一帧更早的帧（最多到60帧）。因此，即使监测的速度慢于Leap Motion的帧率，你也可以处理每一帧，如果你愿意的话。

> All the tracking data in the Leap Motion system arrives through the Frame object. You can get Frame objects from your controller (after it has connected) by calling the Controller.frame() method. The on_frame() callback of your Listener subclass is called when a new frame of data becomes available. When you aren’t using a listener, you can compare the id value to that of the last frame you processed to see if there is a new frame. Note that by setting the history parameter of the frame() function, you can get earlier frames than the current one (up to 60 frames are stored). Thus, even when polling at a slower rate than the Leap Motion frame rate, you can process every frame, if desired.

把`frame()`加到你的`on_frame()`来获取帧：<br>
To get the frame, add this call to frame() to your on_frame() callback:

```
def on_frame(self, controller):
    frame = controller.frame()
```

然后打出一些`Frame` object的属性:<br>
Then, print out some properties of the Frame object:

```
def on_frame(self, controller):
    frame = controller.frame()

    print "Frame id: %d, timestamp: %d, hands: %d, fingers: %d, tools: %d, gestures: %d" % (
          frame.id, frame.timestamp, len(frame.hands), len(frame.fingers), len(frame.tools), len(frame.gestures()))
```

再次运行你的示例，把你的手放在LM设备上方，你可以看到每一帧的基本数据出现在控制台窗口中。
> Run your sample again, put a hand or two over the Leap Motion device and you should now see the basic statistics of each frame printed to the console window.

本教程到此结束，但是你可以从原本自带的示例程序中看到其余的代码，它们用来获取每一帧中的[Hand](https://developer.leapmotion.com/documentation/python/api/Leap.Hand.html),[Finger](https://developer.leapmotion.com/documentation/python/api/Leap.Finger.html),[Arm](https://developer.leapmotion.com/documentation/python/api/Leap.Arm.html),[Bone](https://developer.leapmotion.com/documentation/python/api/Leap.Bone.html),[Gesture](https://developer.leapmotion.com/documentation/python/api/Leap.Gesture.html) object。

> I’ll end this tutorial here, but you can look at the rest of the code in the original sample program for examples on how to get all the Hand, Finger, Arm, Bone and Gesture objects in a frame.

## 运行示例
在终端中输入下面的命令（当前所处位置为SDK samples文件夹）：
Type the following at your command prompt (with the current directory set the SDK samples folder):

```
python Sample.py
```

当程序初始化并连接到Leap后你可以看到“Connected”消息出现。然后你会看到onFrame发出的每一帧的信息。
> You should see the message “Connected” printed to standard output when the application initializes and connects to the Leap. You should then see frame information printed each time the Leap dispatches the onFrame event.