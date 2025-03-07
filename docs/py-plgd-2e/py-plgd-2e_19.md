# [14](nsp-venkitachalam503045-0008.xhtml#rch14)

# 物联网花园

![](images/nsp-venkitachalam503045-circle-image.jpg)

我们生活在一个手机能与灯泡对话，牙刷也想接入互联网的时代。这一切都得益于 *物联网 (**IoT**)*，它是由嵌入传感器的日常设备组成的网络，这些设备相互之间以及与互联网进行通信，通常是无线的。在本章中，你将构建自己的物联网传感器网络，用于监控花园中的温湿度条件。该网络将由一个或多个运行 Python 代码并通过无线方式将实时传感器数据传输到树莓派的低功耗设备组成。树莓派将记录这些数据，并通过本地 web 服务器使其可用。你将能够通过浏览器查看传感器数据，并在出现极端天气情况时，实时接收移动设备上的警报。

通过这个项目，你将学习到一些概念：

+   • 构建一个低功耗物联网传感器网络

+   • 理解蓝牙低能耗（BLE）协议的基础

+   • 在树莓派上构建一个 BLE 扫描器

+   • 使用 SQLite 数据库存储传感器数据

+   • 在树莓派上使用 `Bottle` 运行一个 web 服务器

+   • 使用 If This Then That (IFTTT) 向手机发送警报

## [工作原理](nsp-venkitachalam503045-0008.xhtml#rah1601)

本项目中使用的物联网设备是 Adafruit BLE Sense 板，它们内置了温度和湿度传感器。设备定期进行测量，并通过低功耗蓝牙（BLE）无线传输传感器数据，我们稍后将讨论这个技术。这些数据会被运行 BLE 扫描器的树莓派接收。树莓派使用数据库存储和检索数据，同时运行 web 服务器，以便在网页上显示数据。此外，树莓派还有逻辑检测温湿度数据中的异常，并在发生异常时通过 IFTTT 服务将警报发送到用户的移动设备。[图 14-1](nsp-venkitachalam503045-0029.xhtml#fig14-1) 总结了该项目的架构。

![](images/nsp-venkitachalam503045-f14001.jpg)

图 14-1：物联网花园系统架构

你可能会问，为什么需要树莓派呢？为什么传感器设备不能直接与互联网通信，而非通过树莓派？答案在于功耗。如果设备通过类似 Wi-Fi 的协议直接与互联网通信，它们的功耗通常是使用 BLE 的 10 倍以上。这一点很重要，因为物联网设备通常由电池供电，我们希望它们能够长时间使用。[图 14-1](nsp-venkitachalam503045-0029.xhtml#fig14-1) 中的方案，其中低功耗无线设备通过网关（在此为树莓派）进行通信，是物联网世界中的常见架构。

这个项目架构的另一个特点是你的数据始终掌握在自己手中——准确来说，保存在你的Pi上的数据库中。你并没有将数据通过互联网发送，而是将其局限于本地网络中。这对于基本的花园数据可能并不重要，但知道物联网设备并不总是*需要*将所有数据传送到互联网以保持其有用性，仍然是一个好消息。隐私和安全是将数据仅在确有必要时暴露到互联网的两个重要理由。在这种情况下，你仍然会稍微使用互联网来发送IFTTT警报。

### [蓝牙低能耗](nsp-venkitachalam503045-0008.xhtml#rbh1601)

BLE是与蓝牙耳机和扬声器相同无线技术标准的一个子集，但它针对低功耗、可由电池供电的设备进行了优化。例如，BLE就是智能手机与智能手表或健身追踪器之间通信的方式。通过BLE进行通信的设备可以分为*中央*设备和*外围*设备。通常，中央设备是更强大的硬件，如笔记本电脑和手机，而外围设备则是能力较弱的硬件，如健身带和信标。在这个项目中，树莓派是中央设备，Adafruit BLE Sense开发板是外围设备。

注意 中央设备和外围设备之间的区别并不总是非常明确。现代的BLE芯片允许同一硬件既能作为中央设备，也能作为外围设备，或者两者的组合。

一个BLE外围设备通过*广告包*来表明其存在，如[图 14-2](nsp-venkitachalam503045-0029.xhtml#fig14-2)所示。这些数据包通常每隔几毫秒发送一次，包含的信息有外围设备的名称、其传输功率、制造商数据、中央设备是否能连接到它等。中央设备会持续扫描广告包，并可利用包中的信息与外围设备建立通信。

![](images/nsp-venkitachalam503045-f14002.jpg)

图 14-2：BLE广告方案

广告包中的数据量仅限于31字节，以节省外围设备的电池电量，但外围设备可以选择通过名为*扫描响应*的单独传输来发送附加的数据。外围设备在其正常的广告包中指示是否有扫描响应可用。如果中央设备需要额外数据，它会发送扫描响应请求，这会促使外围设备暂停发送广告包，改为发送扫描响应。不过对于这个项目，你所需的传感器数据非常少，因此你可以直接将温度和湿度数据放入广告包中。

在 Raspberry Pi 端，你将使用 BlueZ 构建一个 BLE 扫描器，BlueZ 是 Linux 上的官方 Bluetooth 协议栈。具体来说，你将使用以下三个命令行程序：

`hciconfig` 在程序初始化时重置 Pi 上的 BLE

hcitool       扫描 BLE 外设

hcidump       读取 BLE 外设的广告数据

`hciconfig` 和 `hcitool` 是 Raspberry Pi OS 安装的一部分，但你需要从终端安装 `hcidump`，命令如下：

$ `sudo apt-get install bluez-hcidump`

下面是 Raspberry Pi 上典型的命令行会话，使用这些工具。首先，在一个终端中运行 `hcidump`，以准备在扫描开始时输出数据包：

pi@iotsensors:~ $ `sudo hcidump --raw`

HCI 嗅探器 - 蓝牙数据包分析仪 ver 5.50

device: hci0 snap_len: 1500 filter: 0xffffffff

这表示 `hcidump` 正在等待 BLE 输入。接下来，在另一个终端中运行 `lescan` 命令，通过 `hcitool` 开始扫描 BLE 设备：

pi@iotsensors:~ $ `sudo hcitool lescan`

LE 扫描中...

DE:74:03:D9:3D:8B (未知)

DE:74:03:D9:3D:8B IOTG1

36:D2:35:5A:BF:B0 (未知)

8C:79:F5:8C:AE:DA (未知)

5D:9F:EC:A0:09:51 (未知)

5D:9F:EC:A0:09:51 (未知)

60:80:0A:83:18:40 (未知)

--`snip`--

这表示扫描器已检测到大量的 BLE 设备（现在它们到处都是！）。一旦你运行 `lescan` 命令，`hcidump` 开始打印广告数据包，所以你的 `hcidump` 命令行窗口应该充满了类似以下的信息：

< 01 0B 20 07 01 10 00 10 00 00 00

> 04 0E 04 01 0B 20 00

< 01 0C 20 02 01 01

> 04 0E 04 01 0C 20 00
> 
> 04 3E 1B 02 01 02 01 8B 3D D9 03 74 DE 0F 0E FF 22 08 0A 31

FE 49 4F 54 47 31 1B 36 30 CB

> 04 3E 16 02 01 04 01 8B 3D D9 03 74 DE 0A 02 0A 00 06 09 49

4F 54 47 31 CB

> 04 3E 23 02 01 03 01 03 58 0A 00 6A 35 17 16 FF 06 00 01 09

21 0A 13 71 DA 7D 1A 00 52 6F 63 69 6E 61 6E 74 65 C5

> 04 3E 1F 02 01 03 01 B9 D4 AE 7E 01 0E 13 12 FF 06 00 01 09

21 0A 9E 54 20 C5 51 48 6D 61 6E 64 6F BE

这些消息以十六进制字节的形式输出（而不是人类可读的文本），因为你使用 `--raw` 选项启动了 `hcidump`。

这个示例展示了如何在命令行中手动使用 BlueZ 工具。在这个项目中，你将从 Pi 上运行的 Python 代码内部执行这些命令。Python 代码还会读取广告数据包并获取传感器数据。

### [Bottle Web 框架](nsp-venkitachalam503045-0008.xhtml#rbh1602)

要通过 Web 界面监控传感器数据，你需要让 Pi 运行一个 Web 服务器。为此，你将使用 `Bottle`，一个 Python Web 框架，具有简单的接口。（事实上，整个库由一个名为 *bottle.py* 的单一源文件组成。）下面是使用 `Bottle` 从 Pi 提供简单网页所需的代码：

from bottle import route, run

❶ @route('/hello')

def hello():

❷ return "Hello Bottle World!"

❸ run(host='`iotgarden.local`', port=`8080`, debug=True)

这段代码首先定义了一个指向 URL 或路径（在此例中为 `/hello`）的路由，客户端可以在此路由上发送数据请求 ❶，并使用 `Bottle` 中的 `route()` 方法作为 Python 装饰器绑定到 `hello()` 函数，该函数将作为该路由的处理函数。这样，当用户访问该路由时，`Bottle` 将调用 `hello()` 函数，返回一个字符串 ❷。`run()` 方法 ❸ 启动 `Bottle` 服务器，服务器现在可以接受来自客户端的连接。这里假设服务器运行在一个名为 `iotgarden` 的树莓派上，端口为 8080。注意，`debug` 标志已设置为 `True`，以便更容易诊断问题。

在你的 Wi-Fi 连接的树莓派上运行此代码，打开任何连接到本地网络的计算机上的浏览器，访问 *http://<iotgarden>.local:8080/hello/*，并根据需要替换为你的树莓派名称。`Bottle` 应该为你提供一个包含“Hello Bottle World!”文本的网页。只需几行代码，你就创建了一个 web 服务器。

注意，你将在项目中稍微不同地使用 `Bottle` 路由功能，与这个简单的例子不同，因为你会将路由绑定到类方法，而不是像之前所展示的那样绑定到自由函数（例如 `hello()`）。稍后我们会详细讨论。

### [SQLite 数据库](nsp-venkitachalam503045-0008.xhtml#rbh1603)

你需要一个地方来存储传感器数据，以便以后能够检索。你可以将数据写入文本文件，但检索过程会很快变得繁琐。相反，你将使用 SQLite 存储数据，SQLite 是一个轻量级、易于使用的数据库，非常适合像树莓派这样的嵌入式系统。要在 Python 中访问 SQLite，你将使用 `sqlite3` 库。

SQLite 数据库通过 SQL 进行操作，SQL 是一种用于数据库系统的标准语言。SQL 语句以字符串形式写入到 Python 代码中。然而，使用 SQLite 进行此项目时，你不需要成为 SQL 专家。你只需掌握几个命令，我们会在需要时进行讨论。为了了解它是如何工作的，我们来看看一个简单的例子，展示如何在 Python 解释器会话中使用 SQLite。首先，这里是如何创建一个数据库并添加一些条目：

>>> `import sqlite3`

>>> `con = sqlite3.``connect('test.db')` ❶

>>> `cur = con.``cursor()` ❷

>>> `cur.``execute("``CREATE TABLE sensor_data (``TS datetime,` `ID text,` `VAL numeric)")` ❸

>>> `for i in range(10):`

...   `cur.execute("INSERT into sensor_data VALUES (datetime('now'),'ABC', ?) ", (i, ))` ❹

>>> `con.``commit()` ❺

>>> `con.``close()`

>>> `exit()`

这里你通过调用`sqlite3.connect()`方法，传入数据库名称（在此例中为`test.db`）❶。该方法返回一个连接到现有数据库的连接对象，如果指定名称的数据库不存在，则会创建一个新数据库。接着，你使用连接对象创建一个*游标*❷。游标是一个可以让你与数据库交互的结构，用来创建表、添加新条目并检索数据。你使用游标执行一个SQL语句，创建一个名为`sensor_data`的数据库表，包含以下列：`TS`（时间戳），类型为`datetime`；`ID`，类型为`text`；`VAL`，类型为`numeric`❸。接下来，你通过在`for`循环中执行SQL `INSERT`语句，向数据库中添加10条记录。每个语句添加当前时间戳、字符串`'ABC'`以及循环索引`i`❹。`?`是SQLite使用的格式占位符，实际值通过元组指定。最后，你提交更改❺，使其在数据库中永久保存，然后关闭数据库连接。

现在让我们从数据库中检索一些值：

>>> `con = sqlite3.``connect('test.db')`

>>> `cur = con.``cursor()`

❶ >>> `cur.``execute("``SELECT * FROM sensor_data WHERE VAL > 5")`

>>> `print(cur.``fetchall())`

[('2021-10-16 13:01:22', 'ABC', 6), ('2021-10-16 13:01:22', 'ABC', 7),

('2021-10-16 13:01:22', 'ABC', 8), ('2021-10-16 13:01:22', 'ABC', 9)]

再次建立与数据库的连接，并创建一个游标与其交互。然后，你执行一个`SELECT` SQL查询来检索一些数据❶。在这个查询中，你请求从`sensor_data`表中选择所有行（`SELECT *`），条件是`VAL`列中的值大于`5`。你打印查询结果，这些结果可以通过游标的`fetchall()`方法访问。

## [要求](nsp-venkitachalam503045-0008.xhtml#rah1602)

在树莓派上，你需要`bottle`模块来创建一个Web服务器，`sqlite3`模块来操作SQLite数据库，`matplotlib`来绘制传感器数据。BLE Sense板没有足够的计算能力来运行完整版本的Python，因此你将使用CircuitPython编程，它是Adafruit维护的MicroPython的开源衍生版本。我们在这个项目中使用CircuitPython，而不是[第12章](nsp-venkitachalam503045-0027.xhtml#ch12)中提到的MicroPython，因为前者对Adafruit制造的设备有更多的库支持。

本项目还需要以下硬件：

+   • 一块或多块Adafruit Feather Bluefruit nRF52840 Sense板，根据需要选择

+   • 一块带SD卡和电源适配器的树莓派3B+（或更新版本）主板

+   • 每块BLE Sense板需要一块3.7V锂聚合物电池或USB电源供应

[图14-3](nsp-venkitachalam503045-0029.xhtml#fig14-3)展示了所需的硬件。

![](images/nsp-venkitachalam503045-f14003.jpg)

图14-3：项目所需的硬件

您可以将树莓派放置在室内，靠近您的花园，并将您的BLE Sense开发板放置在花园中，配备适当的电源单元和保护外壳。

### [树莓派设置](nsp-venkitachalam503045-0008.xhtml#rbh1604)

要开始这个项目，您需要设置您的树莓派。请按照[附录B](nsp-venkitachalam503045-0032.xhtml#appb)中的说明操作。接下来的项目代码假设您已经将树莓派命名为`iotgarden`，这样您就可以通过网络以`iotgarden.local`访问它。

### [CircuitPython 设置](nsp-venkitachalam503045-0008.xhtml#rbh1605)

要安装CircuitPython，请按照以下步骤为每个Adafruit BLE Sense开发板操作：

1.  1. 访问[https://circuitpython.org/downloads](https://circuitpython.org/downloads)，搜索您的Bluefruit Sense开发板，并下载与该开发板对应的CircuitPython安装文件，该文件扩展名为*.uf2*。请注意您下载的CircuitPython版本号。

1.  2. 将Adafruit开发板连接到计算机的USB端口，并双击开发板上的重置按钮。开发板上的LED灯应变为绿色，并且您的计算机上应该出现一个名为FTHRSNSBOOT的新驱动器。

1.  3. 将*.uf2*文件拖入FTHRSNSBOOT驱动器中。文件复制完成后，开发板上的LED灯将开始闪烁，您的计算机上会出现一个名为CIRCUITPY的新驱动器。

接下来，您需要在开发板上安装所需的Adafruit库。操作方法如下：

1.  1. 访问[https://circuitpython.org/libraries](https://circuitpython.org/libraries)并下载与您所用CircuitPython版本相对应的库包*.zip*文件。

1.  2. 解压下载的文件，将以下文件/文件夹复制到CIRCUITPY驱动器中的名为*lib*的文件夹内。（如果没有*lib*文件夹，请创建一个。）

    +   ◦ *adafruit_apds9960*

    +   ◦ *adafruit_ble*

    +   ◦ *adafruit_bme280*

    +   ◦ *adafruit_bmp280.mpy*

    +   ◦ *adafruit_bus_device*

    +   ◦ *adafruit_lis3mdl.mpy*

    +   ◦ *adafruit_lsm6ds*

    +   ◦ *adafruit_register*

    +   ◦ *adafruit_sht31d.mpy*

    +   ◦ *neopixel.mpy*

1.  3. 按下开发板上的重置按钮，您就可以开始使用该开发板进行此项目了。

默认情况下，CircuitPython会运行CIRCUITPY中任何名为*code.py*的文件。您需要将接下来部分讨论的*ble_sensors.py*文件复制到驱动器中，并将其重命名为*code.py*以运行项目。

### [If This Then That 设置](nsp-venkitachalam503045-0008.xhtml#rbh1606)

IFTTT是一个网络服务，允许您创建自动响应特定操作的规则。您将使用IFTTT在传感器检测到温度或湿度异常时，向您的手机发送警报。请按照以下步骤设置以接收IFTTT警报：

1.  1. 访问IFTTT网站([https://ifttt.com](https://ifttt.com))并注册一个账户。

1.  2. 下载IFTTT应用程序并在智能手机上进行设置。

1.  3. 在浏览器中登录到您的IFTTT账户后，点击**创建**。然后在“If This”框中点击**添加**按钮。

1.  4\. 在弹出的选择服务页面中，搜索并选择**Webhooks**。然后选择**接收带 JSON 负载的 Web 请求**。

1.  5\. 在事件名称下，输入**TH_alert**（注意大小写），然后点击**创建触发器**。

1.  6\. 现在您应该回到创建页面。在“Then That”框中点击**添加**按钮。

1.  7\. 搜索并选择**通知**。然后点击**从 IFTTT 应用发送通知**。

1.  8\. 在弹出的页面中，将文本“T/H Alert！”添加到消息框中。然后点击**添加成分**并选择**OccuredAt**。再次点击**添加成分**并选择**JsonPayload**。

1.  9\. 点击**创建动作**按钮返回创建屏幕。然后点击**继续**和**完成**以完成警报设置。

您需要 IFTTT 密钥才能从 Python 代码触发警报。要查找密钥，请按照以下步骤操作：

1.  1\. 在 IFTTT 网站上，点击屏幕右上角的圆形账户图标，并选择**我的服务**。

1.  2\. 点击**Webhooks**链接，然后点击**文档**按钮。

1.  3\. 页面加载时，您的密钥会显示在顶部。请记下该密钥。您还会在此页面找到如何向智能手机发送测试警报的信息。如果运行测试，确保将事件名称填写为**TH_alert**。

## [代码](nsp-venkitachalam503045-0008.xhtml#rah1603)

该项目的代码分布在以下 Python 文件中：

ble_sensors.py 运行在 Adafruit BLE Sense 板上的 CircuitPython 代码。它读取温湿度传感器的数据，并将数据放入 BLE 广告包中。

BLEScanner.py 在 Raspberry Pi 上实现了 BLE 扫描器，使用 BlueZ 工具读取广告数据。此代码还会发送 IFTTT 警报。

server.py 实现了 Raspberry Pi 上的 `Bottle` Web 服务器。

iotgarden.py 主要程序文件。此代码设置 SQLite 数据库，并启动 BLE 扫描器和 Web 服务器。

除了 Python 文件，项目还包含一个名为*static*的子文件夹，里面有一些 `Bottle` Web 服务器使用的额外文件：

static/style.css 服务器返回的 HTML 的样式表

static/server.js 服务器返回的 JavaScript 代码，用于获取传感器数据

项目的完整代码可以在 [https://github.com/mkvenkit/pp2e/tree/main/iotgarden](https://github.com/mkvenkit/pp2e/tree/main/iotgarden) 查看。

### [CircuitPython 代码](nsp-venkitachalam503045-0008.xhtml#rbh1607)

运行在BLE Sense板上的CircuitPython代码有一个直接的目的：它从内置的温湿度传感器读取数据，并将这些数据放入板子的BLE广告包中。这个看似简单的任务需要导入出乎意料的多个模块。要查看完整的代码清单，请跳转到[“完整的CircuitPython代码”](nsp-venkitachalam503045-0029.xhtml#ah1607)在[第343页](nsp-venkitachalam503045-0029.xhtml#p343)。代码也可以在[https://github.com/mkvenkit/pp2e/blob/main/iotgarden/ble_sensors/ble_sensors.py](https://github.com/mkvenkit/pp2e/blob/main/iotgarden/ble_sensors/ble_sensors.py)找到。

import time, struct

import board

import adafruit_bmp280

import adafruit_sht31d

from adafruit_ble import BLERadio

❶ from adafruit_ble.advertising import Advertisement, LazyObjectField

❷ from adafruit_ble.advertising.standard import ManufacturerData,

ManufacturerDataField

import _bleio

import neopixel

你需要导入Python的内建`time`和`struct`模块，分别用于睡眠和打包数据。`board`模块让你可以访问`I2C`库，从而使板上的BLE芯片能够通过I²C协议（发音为“eye-squared-C”）与传感器进行通信。`adafruit_bmp280`和`adafruit_sht31d`模块是与传感器通信所必需的，而`BLERadio`类则用于启用BLE并发送广告包。❶和❷处的导入让你可以使用Adafruit BLE广告模块，创建包含传感器数据的自定义广告包。此外，你还会使用`_bleio`模块来获取设备的MAC地址，使用`neopixel`来控制板上的LED。
