---
layout: post
title: 使用 pythonnet 库开发 Windows 程序
image: /img/hello_world.jpeg
category: [ python ]
---

## 序
对于 Python 的 GUI 开发一直没有一个特别方便，适合学生快速掌握、使用的方法。无论是 wxPython 还是 PyQt，都要求较高的界面布局技能和编程技巧。直到发现了 pythonnet 库。

pythonnet 是一个 Python 和 dotnet 桥接的库，可以实现在 Python 中调用 dotnet 框架及在 dotnet 中调用 Python 程序。

## 安装
开发环境需要在 Windows + .net framework 4.0 以上，Python 方面需要通过以下命令安装 pythonnet 库：
```bash
pip install pythonnet
```

GUI 设计上使用 xaml 语言，通过 Microsoft Express Studio 4 可以很方便的绘制布局。安装程序在[这里](http://download.microsoft.com/download/B/4/E/B4E108E1-14EC-4E9D-96ED-379D687270C0/ExpressionStudio_UltimateTrial_zh-Hans.exe)

## Hello XAML
以下是使用 pythonnet 开发的 HelloXaml 程序：
```python
# pythonnet 包名叫做 clr，需要 import 此库才能加载 dotnet 的 dll 文件
import clr                                                          

# 添加 xaml 所在的 wpf 支持，这实际上是一个 dll 文件
clr.AddReference("wpf\PresentationFramework")
# 引入 xaml 解析器
from System.Windows.Markup import XamlReader
# 引入 Windows 的线程管理相关功能
from System.Threading import Thread, ThreadStart, ApartmentState
# 引入 Windows 应用程序，窗口等相关功能
from System.Windows import Application, Window
# 以上这些都是比较固定的，不需要特别记忆，后面的程序也会在此基础上封装起来，一般看不到的


'''
这里需要构建我们自己的一个 MyWindow 类，所有的交互控制操作都会写在这个类里。
其父类是 System.Windows.Window
'''
class MyWindow(Window):

    # 构造函数中需要实现加载 xaml 布局内容，并启动窗口程序
    def __init__(self):
        # xaml 是布局代码，也可以在单独的文件中编写，然后直接加载文件。
        xaml = '''<Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
                <Grid>
                    <Label Content="Hello XAML!" HorizontalAlignment="Center" VerticalAlignment="Center"/>
                </Grid>
            </Window>'''
        window = XamlReader.Parse(xaml)
        Application().Run(window)


if __name__ == '__main__':
    # Window 程序区别于命令行程序，有很多交互操作，所以界面部分必须要在一个线程中运行。
    # 这部分代码也不需要特别关注，后续会封装起来的
    thread = Thread(ThreadStart(MyWindow))
    thread.SetApartmentState(ApartmentState.STA)
    thread.Start()
    thread.Join()
```

运行成功你会看到这样的结果：

![HelloXaml 运行结果](/assets/python/pythonnet.1.png)

## 进阶
如果写个 Windows 程序都要写这么一大堆配置代码，实在是超级不 Pythonish。因此，封装了所有固定代码，同时还增加了一些约定方法，让 pythonnet 使用起来更加方便。见 [win_helper/helper.py](./win_helper/helper.py)

通过一个更有意思的程序看看 helper 都帮助我们做了什么：
```python
from win_helper import *
import math

class MyWindow(PythonWin):
    def __init__(self):
        self.i = 0

    def Window_SizeChanged_handler(self, obj, event):
        '''监听窗口尺寸变化
        '''
        label = self.controls['All']['Label1']
        if obj.WindowState == WindowState.Maximized:
            label.Content = '窗口最大化了'
        elif not math.isnan(obj.Width):
            label.Content = '窗口大小：{}x{}\n窗口位置：{}, {}'.format(obj.Width, obj.Height, obj.Top, obj.Left)

    def Label1_MouseUp_handler(self, obj, event):
        '''监听 Label1 上鼠标抬起事件
        '''
        self.i += 1
        obj.Content = '点击了 {} 次'.format(self.i)


if __name__ == '__main__':
    MyWindow.run('resizing.xaml')
```

```xml
<Window
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" Width="800" Height="600">
    <Grid>
        <Label Name="Label1" Content="这是一个窗口，点击文字试试，或者修改下窗口尺寸" HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </Grid>
</Window>
```


这个程序只需要引入 win_helper 模块就可以了，然后 MyWindow 类的父类需要改为在 helper.py 中定义的 `PythonWin`，主程序（`if __name__ == '__main__'`）下直接执行 `MyWindow.run(xaml 文件)` 就可以了。

同时，通过约定的方式，可以快速定义事件响应功能。比如：`Window_SizeChanged_handler` 表示窗口的尺寸变化事件的处理函数。约定的函数命名规则如下：

    * [对象名称]_[事件名称]_handler(self, 引发事件的对象, 事件对象)
    * 对象名称区分大小写，按照 xaml 中的组件 Name 来设置对象名称
    * 事件名称区分大小写，按照文档选择相应的事件

对象名称是在 xaml 中定义的控件名称，比如上面的 xaml 中 `<Label />` 标签的 Name 为 `Label1`，我们想在其上添加一个鼠标抬起的事件处理函数（Label 没有鼠标点击事件）。所以我们声明了一个 `Label1_MouseUp_handler(self, obj, event)` 函数，在其中对 Label 的内容进行了修改。


## 对控件的控制

xaml 控件都包含非常多的属性，比如文本内容啦，字体字号，前景色背景色，宽高位置等等。我们可以在事件函数中通过对 obj 操作当前引发事件的对象直接进行控制，还可以通过 `self.controls` 找到其他控件，`self.controls` 是一个二维字典（dict），第一维是控件的分类，根据 xaml 文件中涉及得有名称（包含 `Name` 属性）控件的类型进行分类。比如上面代码中的 Label 控件，我们就可以通过 `self.controls['Label']['Label1']` 找到这个 Label 控件，另外 `self.controls` 中还有一个特殊的分类 `All`，在其中包含了所有有名称的控件，也可以通过 `self.controls['All']['Label1']` 找到之前提到的 Label 控件。另外，程序的窗口对象可以通过 `self.window` 找到。

## Xaml 介绍

Xaml 是微软出品的一个布局设计语言，其官方的文档已经非常详细了，可以访问[这里](https://msdn.microsoft.com/zh-cn/library/windows/apps/mt228349.aspx)开始你的学习了解。

Xaml 可以通过 Microsoft Expression Studio 4 进行布局设计。

![Expression Blend 4](/assets/python/pythonnet.2.png)

## 自定义绑定事件

虽然 helper.py 可以简化很多操作，但是如果希望可以更加自由的管理窗口及控件，我们可以定义一个 `load` 方法。这个方法将在窗口建立后调用。举例，也许你希望把不同的按钮绑定到同一个事件处理方法上。在这个方法中，你就可以不按照约定的方式绑定事件；或者有某个控件，你的程序会经常使用，每次使用 `self.controls['All']['SomeControl']` 太啰嗦，你可以在这个函数里定义个 `self.some_control = self.controls['All']['SomeControl']`，这样以后使用上就方便很多了。

你可以参看 [calc.py](./calc.py)，学习一下一个比较完整的 Windows 程序的例子。
