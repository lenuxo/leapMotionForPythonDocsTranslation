# 建立项目

本文教你如何把python库引入python程序

## 库Libraries

LeapMotion Python的API是一个Python扩展模块，它包括Python代码和源码。Python API支持Python 2.7。

The Leap Motion Python API is provided as a Python extension module containing both Python and native code. The Python API supports Python 2.7.

||Windows|Mac|Linux|
|:--|:--|:--|:--|
|Python模块|lib/Leap.py|lib/Leap.py|lib/Leap.py|
|32位原生库|lib/x86/LeapPython.pyd<br>lib/x86/Leap.dll||lib/x86/LeapPython.so<br>lib/x86/libLeap.so|
|64位原生库|lib/x64/LeapPython.pyd<br>lib/x64/Leap.dll||lib/x64/LeapPython.so<br>lib/x64/libLeap.so|
|通用二进制原生库||lib/LeapPython.so<br>lib/libLeap.dylib||

你可以在下载到的SDK中的lib文件夹中找到这些库文件。在windows和linux上，32位python要对应x86的库，64位python要用x64的库。
You can find these libraries in the lib folder of the Leap Motion SDK download package. On Windows and Linux, you must use the x86 versions of the native libraries for 32-bit versions of Python and the x64 versions for 64-bit versions of Python.

LM没有把库做成一个标准python包或者直接安装到用户电脑上。你的把库作为你程序里的内部模块。你应该把库和你的程序一起发布。
Leap Motion does not provide these libraries in a standard Python package or install them on end-user computers. Treat the libraries as internal modules of your application. It is your responsibility to distribute the libraries with your application.

## 引入LM模块 Importing the Leap Motion Module

要引入Leap模块，必须把库文件放到Python运行时可以找到的地方。最简单的方法就是把库和程序文件放到同一个目录里。Python会在同一目录里找导入模块。
To import the Leap module, the library files must be placed in a location where the Python runtime can find them. The easiest way to do this is to put them in the same directory as your application source code. Python looks in the same directory as the importing file for imported modules.

如果你想把库放在与你的源码不同的目录，你可以在引入模块前把库的路径加到`sys.path`列表中
If you prefer to keep the libraries in a separate directory from your source code, you can add the path to the Leap Motion libraries to the Python sys.path list before importing the Leap module:

```
import sys
sys.path.insert(0, "path/to/leap/libraries")
import Leap
```

For example, if your project file structure looked like:

```
SnakesAndLadders/
    src/
        Snakes.py
        Ladders.py
    lib/
```
你也可以拷贝LM库到子文件夹中（确保复制的是适合你的平台和架构的文件）。比如在mac上你的项目可能是这样的：
You could copy the Leap Motion libraries to the lib subfolder (making sure to copy the appropriate files for your platform and architecture). For example, on a Mac, your project would look like:

```
SnakesAndLadders/
    src/
        Snakes.py
        Ladders.py
    lib/
        Leap.py
        LeapPython.so
        libLeap.dylib
```
 
你可以用下面的方法引入Leap模块
From the Snakes.py or Ladders.py source, you could then import the Leap module with the following:

```
import sys
sys.path.insert(0, "../lib")
import Leap
```

这种方法假定你的Python文件总是在当前包含你Python主程序的工作目录运行的。要想让主程序从任何位置运行时引入都能正常工作，你可以用Python检查模块来得到包含源码的目录，然后把sys.path关联到那个文件夹：
This import method assumes that your Python file is always run with the current working directory set to the folder containing your main Python file. To allow the import to work when the main application file is invoked from any directory, you can use the Python inspect module to get the directory containing your source file and then set the  sys.path relative to that folder:

```
import os, sys, inspect
src_dir = os.path.dirname(inspect.getfile(inspect.currentframe()))
lib_dir = os.path.abspath(os.path.join(src_dir, '../lib'))
sys.path.insert(0, lib_dir)
import Leap
```
*注意*：如果你开发的只是单独为你自己使用，你可以把Leap模块和其他需要的库放到任何方便的位置：比如其中一个文件夹放`sys.path`变量，一个文件夹是你的`PYTHONPATH`环境变量等等。只要在更新SDK的时候你还能记得你把它们放到哪了。但是，自从Leap模块不再在标准Python包管理器中之后，不推荐把LM库拷贝到Python标准模块索引位置中。这样做，当多个程序尝试安装它们各自不同版本的LM库时，会发生冲突。

> Note: If you are developing applications solely for your own use, you can put the Leap module and its supporting native libraries in any convenient location: for example, in one of the folders listed in the Python sys.path variable, a folder referenced in your PYTHONPATH environment variable, etc – just remember where you put them when it comes time to update your Leap Motion SDK. However, since the Leap module isn’t currently available through standard Python package managers, copying the Leap Motion libraries to one of the standard Python module search locations isn’t recommended. Doing so could create conflicts if multiple applications try to install their own version of the Leap Motion libraries.

## 支持32位和64位Python架构

要同时支持windows和linux的32位和64位架构，你可以用运行时检查架构然后把sys.path设置到正确的文件。在mac上，LM库是通用二进制的，支持两种架构，没必要管它。

> To support both 32-bit and 64-bit architectures at the same time on Windows and Linux, you can use a run-time check for the architecture and then set the sys.path to reference the correct files. On Mac, the Leap Motion libraries are universal binaries that support both architectures, so this technique is not needed.

再用假定的SnakesAndLadders项目作为例子，在程序里你应把两种库都分别放到两个文件夹中，如下：

> Using the hypothetical SnakesAndLadders project as an example again, you would copy both sets of library files to separate folders within the application, for example:

```
SnakesAndLadders/
    src/
        Snakes.py
        Ladders.py
    lib/
        x86/
            Leap.py
            LeapPython.so
            libLeap.so
        x64/
            Leap.py
            LeapPython.so
            libLeap.so
            
```
然后进入合适的Leap模块：
And then reference the proper Leap module at run time: 

```
import os, sys, inspect
src_dir = os.path.dirname(inspect.getfile(inspect.currentframe()))
arch_dir = '../lib/x64' if sys.maxsize > 2**32 else '../lib/x86'
sys.path.insert(0, os.path.abspath(os.path.join(src_dir, arch_dir)))

import Leap
```

要做多平台支持可以用同样的技术。
The same technique could be extended to support multiple platforms at the same time.

### 使用不同的Python发行版

在Mac或linux中的或者自建的Python中使用Leap Motion Python库，你必须更新`LeapPython.so`读取路径到相应的Python。
> To use the Leap Motion Python libraries with an alternate Python 2.7 distribution on Mac or Linux (i.e HomeBrew or MacPorts, etc.) or with a self-built version of Python, you must update the LeapPython.so loader path to reference the desired instance of Python.

**首先**，运行`otool`来显示当前读取路径：
First, run the otool utility to display the current loader paths:

```
otool -L LeapPython.so

```

显示出来差不多是这样：

```
LeapPython.so:
    @loader_path/LeapPython.so (compatibility version 0.0.0, current version 0.0.0)
    /Library/Frameworks/Python.framework/Versions/2.7/Python (compatibility version 2.7.0, current version 2.7.0)
    @loader_path/libLeap.dylib (compatibility version 0.7.0, current version 2.0.1)
    /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 169.3.0)
    /usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 56.0.0)
```

Python就在`/Library/Frameworks/Python.framework/Versions/2.7/Python` 这行，它需要用`install_name_tool`来改变。
The line starting with, /Library/Frameworks/Python.framework/Versions/2.7/Python is the reference to Python, which needs to be changed with the install_name_tool utility.

**然后**，运行`install_name_tool`工具来把Python的路径变更到你需要的位置，如下：
> Second, run the install_name_tool utility to update the Python reference to the desired location. For example:

```
install_name_tool -change /Library/Frameworks/Python.framework/Versions/2.7/Python \
/usr/local/Cellar/python/2.7.5/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib \
LeapPython.so
```

注意：`otool`和`install_name_tool`都是标准的Linux和osx命令行工具。
Note: otool and install_name_tool are standard Linux and OS X command line utilities.

## 为Python3重编译LeapPython

SDK中的LeapPython库只支持Python2.7。但是，SDK也包括SWIG接口文件，它可以用来生成LeapPython源码。这样高级用户就可以生成并编译他们自己的LeapPython。[这个地方](https://support.leapmotion.com/entries/39433657-Generating-a-Python-3-3-0-Wrapper-with-SWIG-2-0-9)可以看到相关信息。

> The LeapPython library included in the Leap Motion SDK supports only Python 2.7. However, the SDK also includes the SWIG interface file used to generate the LeapPython source code, so advanced users can generate and compile their own version of LeapPython. For instructions, refer to Generating a Python 3.3.0 Wrapper with SWIG 2.0.9 in our support knowledge base.


-----

[原文地址](https://developer.leapmotion.com/documentation/python/devguide/Project_Setup.html#id12)