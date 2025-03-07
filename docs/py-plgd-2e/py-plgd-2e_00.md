# [引言](nsp-venkitachalam503045-0008.xhtml#rintro)

![](images/nsp-venkitachalam503045-circle-image.jpg)

欢迎来到《*Python Playground*》的第二版！在这本书中，你将找到15个令人兴奋的项目，旨在鼓励你用Python探索编程的世界。这些项目涵盖了广泛的主题，如绘制类似Spirograph的图案、3D渲染、根据音乐同步投影激光图案，以及使用机器学习进行语音识别。除了本身非常有趣，这些项目还提供了大量的扩展空间，旨在成为你探索自己想法的跳板。

## [这本书适合谁？](nsp-venkitachalam503045-0008.xhtml#rahintro01)

《*Python Playground*》是为任何对如何通过编程理解和探索创意感到好奇的人写的。本书中的项目假设你已经掌握了基本的Python语法和编程概念，并且对高中数学有所了解。我已经尽力详细解释了所有项目所需的数学知识。

本书并不打算成为你学习Python的第一本书。我不会带你走过基础知识的学习。然而，我会向你展示如何使用Python解决一系列实际的非平凡项目。在完成这些项目的过程中，你将深入探索Python编程语言的细微差别，并学习如何使用一些流行的Python库。但或许更重要的是，你将学会如何将问题分解成多个部分，开发出解决问题的算法，然后从头到尾使用Python实现解决方案。

解决现实世界的问题可能是困难的，因为这些问题通常是开放式的，并且需要多领域的专业知识。但Python提供了工具来促进问题解决。克服困难并找到现实问题的解决方案是你成为专家程序员之路上最重要的部分。

## [这本书的内容是什么？](nsp-venkitachalam503045-0008.xhtml#rahintro02)

让我们快速浏览一下这本书中的章节。[第一部分](nsp-venkitachalam503045-0011.xhtml#pt01)包含了一些入门项目，帮助你开始。

[第1章](nsp-venkitachalam503045-0012.xhtml#ch01): 科赫雪花 利用递归函数和`turtle`图形绘制有趣的分形图形

[第2章](nsp-venkitachalam503045-0013.xhtml#ch02): Spirographs 使用参数方程和`turtle`图形绘制类似玩具Spirograph生成的曲线

[第二部分](nsp-venkitachalam503045-0014.xhtml#pt02)包含了利用数学模型创建计算机模拟现实世界现象的项目。

[第3章](nsp-venkitachalam503045-0015.xhtml#ch03): 康威的生命游戏 使用`numpy`和`matploblib`实现著名的元胞自动机，基于简单的规则生成一个不断演变的生命系统

[第四章](nsp-venkitachalam503045-0016.xhtml#ch04)：卡普鲁斯-斯特朗的音乐泛音真实地模拟了弹拨弦乐器的声音，并通过`pyaudio`回放这些声音

[第五章](nsp-venkitachalam503045-0017.xhtml#ch05)：鸟群聚集使用`numpy`和`matplotlib`实现了Boids算法，模拟了鸟群的集结行为

[第三部分](nsp-venkitachalam503045-0018.xhtml#pt03)的项目将介绍如何使用Python读取和操作2D图像

[第六章](nsp-venkitachalam503045-0019.xhtml#ch06)：ASCII艺术通过将图像转换为基于文本的ASCII艺术，介绍了`Pillow`（Python图像库PIL的一个分支）

[第七章](nsp-venkitachalam503045-0020.xhtml#ch07)：照片马赛克通过拼接较小的图像网格，创建出一个可识别的大图像

[第八章](nsp-venkitachalam503045-0021.xhtml#ch08)：自动立体图使用深度图和像素操作，将3D图像的错觉嵌入到2D图像中

在[第四部分](nsp-venkitachalam503045-0022.xhtml#pt04)，你将学习如何使用着色器和OpenGL库，通过直接与图形处理单元（GPU）合作，快速高效地渲染3D图形

[第九章](nsp-venkitachalam503045-0023.xhtml#ch09)：理解OpenGL介绍了使用OpenGL创建简单3D图形的基础知识

[第十章](nsp-venkitachalam503045-0024.xhtml#ch10)：环面上的康威生命游戏重新审视了[第三章](nsp-venkitachalam503045-0015.xhtml#ch03)的模拟，并将其呈现在3D环面表面上

[第十一章](nsp-venkitachalam503045-0025.xhtml#ch11)：体积渲染实现了一个体积光线投射算法来渲染体积数据，这是一种常用于医学成像（如MRI和CT扫描）的技术

最后，在[第五部分](nsp-venkitachalam503045-0026.xhtml#pt05)，你将与Raspberry Pi及其他电子组件一起工作，将Python部署到嵌入式系统上

[第十二章](nsp-venkitachalam503045-0027.xhtml#ch12)：在Raspberry Pi Pico上的卡普鲁斯-斯特朗通过在一个微小的Raspberry Pi Pico微控制器上使用MicroPython实现[第四章](nsp-venkitachalam503045-0016.xhtml#ch04)的卡普鲁斯-斯特朗算法，创造一个可演奏的电子乐器

[第十三章](nsp-venkitachalam503045-0028.xhtml#ch13)：使用Raspberry Pi的激光音频显示通过在Raspberry Pi上利用Python控制两个旋转镜子和激光器，产生一个响应声音的激光灯光秀

[第十四章](nsp-venkitachalam503045-0029.xhtml#ch14)：物联网花园使用低功耗蓝牙将Raspberry Pi与运行CircuitPython的Adafruit硬件连接，创建一个物联网系统，用于监测花园中的温度和湿度

[第十五章](nsp-venkitachalam503045-0030.xhtml#ch15)：Pi上的音频机器学习通过在Raspberry Pi上实现一个语音识别系统，介绍了使用TensorFlow进行机器学习的激动人心的世界

每一章的结尾都会有一个“实验！”部分，提供如何扩展项目或进一步探索该主题的建议。

## [第二版新增内容](nsp-venkitachalam503045-0008.xhtml#rahintro03)

第二版新增了五个项目，包括[Koch雪花项目](nsp-venkitachalam503045-0012.xhtml#ch01)和[第10章](nsp-venkitachalam503045-0024.xhtml#ch10)中的康威生命游戏3D版。最重要的更新在硬件部分，已精简为专注于基于Raspberry Pi的系统，而不再是Raspberry Pi与Arduino的混合。因此，[第V部分](nsp-venkitachalam503045-0026.xhtml#pt05)中的每个项目要么是新项目（[第12章](nsp-venkitachalam503045-0027.xhtml#ch12)，[第14章](nsp-venkitachalam503045-0029.xhtml#ch14)，[第15章](nsp-venkitachalam503045-0030.xhtml#ch15)），要么是经过彻底重新设计的（[第13章](nsp-venkitachalam503045-0028.xhtml#ch13)）。

仅依赖Raspberry Pi简化了硬件项目的设置过程，同时保持了书籍对Python编程的专注。无需再在Python和Arduino编程语言（C++的一个版本）之间切换。随着更新后的[第V部分](nsp-venkitachalam503045-0026.xhtml#pt05)，你还将体验使用MicroPython和CircuitPython进行编程，这两种Python版本针对资源受限的嵌入式系统进行了优化。

第二版的其他重要更新包括：

+   • 在[第4章](nsp-venkitachalam503045-0016.xhtml#ch04)中，使用`pyaudio`而不是`pygame`来播放WAV文件。

+   • 在[第7章](nsp-venkitachalam503045-0020.xhtml#ch07)中，比较了线性搜索算法与k-d树数据结构在寻找最佳图像匹配时的性能，应用于照片拼贴。

+   • 在[第8章](nsp-venkitachalam503045-0021.xhtml#ch08)中，指导你如何创建自己的深度图以生成自立体图。

+   • 在[附录A](nsp-venkitachalam503045-0031.xhtml#appa)中，精简了使用标准Anaconda Python分发版的安装说明。

除了这些具体的更新外，整个文本已被审阅、纠正和澄清，代码也根据需要进行了更新，以反映自第一版发布以来Python的变化。

## [为什么选择Python？](nsp-venkitachalam503045-0008.xhtml#rahintro04)

Python是探索编程的理想语言。作为一种多范式语言，它为你提供了很多灵活性，可以根据需要构建程序。你可以将Python用作*脚本*语言来执行代码，作为*过程式*语言将程序组织成相互调用的函数集合，或作为*面向对象*语言使用类、继承和模块来构建层次结构。这种灵活性使你能够选择最适合特定项目的编程风格。

当你使用像C或C++这样的传统语言进行开发时，你必须在运行代码之前先编译并链接它。而使用Python，你可以在编辑后直接运行它。（从内部来看，Python将你的代码编译成中间字节码，然后由Python解释器运行，但这些过程对你，用户来说是透明的。）实际上，使用Python修改并一遍又一遍地运行代码，过程要轻松得多。

Python有一组非常小且简单但功能强大的数据结构。如果你已经理解了字符串、列表、元组、字典、列表推导式以及基本的控制结构如`for`和`while`循环，那么你已经迈出了一个良好的开端。Python简洁且富有表现力的语法，使得用几行代码就能执行复杂的操作。而且，一旦你熟悉了Python的内置模块和第三方模块，你将拥有一个工具库来解决现实中的问题，就像本书中涵盖的那些问题一样。Python有标准的方法调用C/C++代码，反之亦然，并且因为你可以找到几乎做任何事情的Python库，所以在更大的项目中将Python与其他语言模块结合使用是很容易的。这就是为什么Python被认为是一种出色的*粘合*语言——它使得结合不同的软件组件变得简单。[第IV部分](nsp-venkitachalam503045-0022.xhtml#pt04)中的3D图形项目展示了Python如何与类似C的OpenGL着色语言并肩工作，在[第14章](nsp-venkitachalam503045-0029.xhtml#ch14)中，你甚至会混合使用一些HTML、CSS和JavaScript，为你的物联网花园监控器创建一个Web界面。实际的软件项目通常会使用混合的软件技术，Python非常适合这种分层架构。

Python还提供了一个方便的工具——Python解释器。它让你轻松检查代码语法，进行快速计算，甚至测试正在开发中的代码。当我编写Python代码时，我会同时打开三个窗口：一个文本编辑器，一个Shell，和一个Python解释器。在编辑器中开发代码时，我将我的函数或类导入到解释器中，并在开发过程中进行测试。

我还使用解释器在将新模块集成到代码中之前，先对其进行测试。例如，在开发[第14章](nsp-venkitachalam503045-0029.xhtml#ch14)的物联网花园项目代码时，我想测试`sqlite3`数据库模块。我启动了Python解释器，并尝试了以下操作，以确保我理解如何创建和向数据库添加条目：

>>> `import sqlite3`

>>> `con = sqlite3.connect('test.db')`

>>> `cur = con.cursor()`

>>> `cur.execute("CREATE TABLE sensor_data (TS datetime, ID text, VAL numeric)")`

>>> `for i in range(10):`

...   `cur.execute("INSERT into sensor_data VALUES (datetime('now'),'ABC',  ?)", (i, ))`

>>> `con.commit()`

>>> `con.close()`

>>> `exit()`

然后，为了确保它有效，我做了以下操作来检索一些我添加的数据：

>>> `con = sqlite3.connect('test.db')`

>>> `cur = con.cursor()`

>>> `cur.execute("SELECT * FROM sensor_data WHERE VAL > 5")`

>>> `print(cur.fetchall())`

[('2021-10-16 13:01:22', 'ABC', 6), ('2021-10-16 13:01:22', 'ABC', 7),

('2021-10-16 13:01:22', 'ABC', 8), ('2021-10-16 13:01:22', 'ABC', 9)]

这个示例展示了如何使用 Python 解释器作为强大的开发工具。你不需要编写完整的程序来进行快速实验，只需打开解释器并开始操作。这只是我喜欢 Python 的众多原因之一，也是我认为你也会喜欢它的原因。

## [本书中的代码](nsp-venkitachalam503045-0008.xhtml#rahintro05)

我在本书中尽力详细逐步地为你讲解每个项目的代码。你可以自己输入代码，也可以通过[https://github.com/mkvenkit/pp2e](https://github.com/mkvenkit/pp2e)下载书中所有程序的完整源代码（使用下载 Zip 选项）。

你将在接下来的页面中找到几个令人兴奋的项目。我希望你能像我在创建它们时一样享受这些项目的乐趣。别忘了在每个项目的结尾探索更多的实验。我祝你在使用*Python Playground*时度过许多愉快的编程时光！
