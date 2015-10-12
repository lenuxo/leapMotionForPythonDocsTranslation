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

LM软件会作为一个服务（windows上）或后台（MAC和Linux上）运行在电脑上。原生LM应用可以通过Leap Motion动态库（SDK里的一部分）用API连接到这个服务。网页应用可以连接一个WebSocket服务器。

> The Leap Motion software runs as a service (Windows) or daemon (Mac and Linux) on the client computer. Native Leap Motion-enabled applications can connect to this service using the API provide by the Leap Motion dynamic libraries (provided as part of the Leap Motion SDK). Web applications can connect to a WebSocket server hosted by the service. The WebSocket provides tracking data as a JSON-formatted message – one message per frame of data. A JavaScript library, LeapJS, provides an API wrapping this data. For more information read System Architecture.