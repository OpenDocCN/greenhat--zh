## 10

使用人脸识别限制访问权限

![图片](../images/common.jpg)

在前一章中，你是太空军的一名联军海军技术员。在这一章中，你依然是那个技术员，只是你的工作变得更加复杂。你现在的角色是*识别人脸*，而不仅仅是检测人脸。你的指挥官，德明上尉，发现了一个包含突变体生产的跨维度传送门的实验室，他希望仅限自己访问该实验室。

如同前一章所述，你需要快速行动，因此你将依赖 Python 和 OpenCV 来提高速度和效率。具体来说，你将使用 OpenCV 的局部二值模式直方图（LBPH）算法，这是一种最古老且最易使用的人脸识别算法，来帮助锁定实验室。如果你之前没有安装和使用过 OpenCV，可以查看 [第6页](ch01.xhtml#page_6)中的“安装 Python 库”。

### **使用局部二值模式直方图识别人脸**

LBPH 算法依赖特征向量来识别人脸。回想一下 [第5章](ch05.xhtml)中提到的，特征向量基本上是按特定顺序排列的一组数字。在 LBPH 中，这些数字代表了面部的一些特性。例如，假设你可以通过一些简单的测量来区分不同的面部，比如眼睛之间的距离、嘴巴的宽度、鼻子的长度和脸部的宽度。按照这个顺序，且以厘米为单位，这四个测量值可能组成如下的特征向量：（5.1，7.4，5.3，11.8）。将数据库中的人脸简化为这些向量，可以实现快速搜索，并且我们可以通过两个向量之间的数值差异，或者说*距离*，来表达它们之间的差异。

识别人脸在计算上需要多个特征，当然，许多可用的算法依赖于不同的特征。在这些算法中有特征脸（Eigenfaces）、LBPH、Fisherfaces、尺度不变特征变换（SIFT）、加速稳健特征（SURF）以及各种神经网络方法。当面部图像在受控条件下采集时，这些算法可以达到与人类相似的高准确率。

对于面部图像的受控条件，可能涉及每个面部的正面视图，表情自然、放松，并且为了让所有算法都能使用，还需要保持一致的光照条件和分辨率。面部应避免被胡须和眼镜遮挡，前提是算法已经学习了如何在这些条件下识别面部。

#### ***人脸识别流程图***

在深入了解 LBPH 算法的细节之前，让我们先来看看人脸识别的一般工作原理。这个过程包括三个主要步骤：捕获、训练和预测。

在捕获阶段，你需要收集将用于训练人脸识别器的图像（见 [图10-1](ch10.xhtml#ch010fig1)）。对于每个你想要识别的面孔，你应拍摄十张或更多张不同表情的图像。

![Image](../images/fig10_01.jpg)

图10-1：捕捉人脸图像以训练人脸识别器

捕捉过程的下一步是检测图像中的人脸，在人脸周围绘制一个矩形，裁剪图像到矩形区域，调整裁剪后的图像大小以匹配相同的尺寸（取决于算法），并将其转换为灰度图像。算法通常使用整数来跟踪人脸，因此每个对象需要一个唯一的ID号。处理后，所有人脸将存储在一个文件夹中，我们称之为数据库。

下一步是训练人脸识别器（见[图10-2](ch10.xhtml#ch010fig2)）。该算法——在我们的案例中是LBPH——分析每个训练图像，然后将结果写入YAML（*.yml*）文件，YAML是一种可读性强的数据序列化语言，用于数据存储。YAML最初代表“Yet Another Markup Language”，但现在表示“YAML Ain’t Markup Language”，以强调它不仅仅是一个文档标记工具。

![Image](../images/fig10_02.jpg)

图10-2：训练人脸识别器并将结果写入文件

训练好人脸识别器后，最后一步是加载一个新的、未经训练的人脸并预测其身份（见[图10-3](ch10.xhtml#ch010fig3)）。这些未知的人脸与训练图像的处理方式相同——即，裁剪、调整大小并转换为灰度图像。然后，识别器会对其进行分析，将结果与YAML文件中的人脸进行比对，并预测最匹配的人脸。

![Image](../images/fig10_03.jpg)

图10-3：使用训练过的识别器预测未知的人脸

请注意，识别器将对每个面孔的身份进行预测。如果YAML文件中只有一个已训练的人脸，识别器将为每个人脸分配该训练人脸的ID号。它还会输出一个*置信度*因素，实际上是新的人脸与训练人脸之间的距离度量。数字越大，匹配越差。我们稍后会更详细地讨论这个问题，但现在要知道，你将使用一个阈值来决定预测的人脸是否正确。如果置信度超过接受的阈值，程序将丢弃该匹配，并将人脸分类为“未知”（见[图10-3](ch10.xhtml#ch010fig3)）。

#### ***提取局部二值模式直方图***

你将使用的OpenCV人脸识别器是基于局部二值模式（LBP）的。这些纹理描述符最早于1994年左右被用来描述和分类表面纹理，例如区分混凝土和地毯。人脸也是由纹理组成的，因此该技术也适用于人脸识别。

在提取直方图之前，你需要先生成二值模式。LBP算法通过将每个像素与其周围的邻居进行比较，计算出纹理的局部表示。第一个计算步骤是将一个小窗口滑动到人脸图像上并捕捉像素信息。[图10-4](ch10.xhtml#ch010fig4)显示了一个示例窗口。

![图片](../images/fig10_04.jpg)

图 10-4：示例 3×3 像素滑动窗口，用于捕获局部二值模式

下一步是将像素转换为二进制数，使用中心值（在此情况下为 90）作为阈值。你通过将八个相邻值与阈值进行比较来实现这一点。如果某个相邻值等于或高于阈值，则赋值为 1；如果低于阈值，则赋值为 0。接下来，忽略中心值，将二进制值逐行连接（某些方法使用顺时针旋转）以形成新的二进制值（11010111）。最后，将该二进制数转换为十进制数（215），并将其存储在中心像素位置。

继续滑动窗口，直到所有像素都转换为 LBP 值。除了使用方形窗口捕获相邻像素外，该算法还可以使用半径，这一过程称为*圆形 LBP*。

现在是时候从上一步产生的 LBP 图像中提取直方图了。为此，你使用一个网格将 LBP 图像划分为矩形区域（见[图 10-5](ch10.xhtml#ch010fig5)）。在每个区域内，构建 LBP 值的直方图（在[图 10-5](ch10.xhtml#ch010fig5)中标记为“局部区域直方图”）。

![图片](../images/fig10_05.jpg)

图 10-5：提取 LBP 直方图

构建局部区域直方图后，按照预定顺序对它们进行归一化并将它们连接成一个长直方图（在[图 10-5](ch10.xhtml#ch010fig5)中显示为截断形式）。由于你使用的是灰度图像，其强度值在 0 和 255 之间，因此每个直方图中有 256 个位置。如果你使用的是 10×10 网格，如在[图 10-5](ch10.xhtml#ch010fig5)中所示，那么最终直方图中将有 10×10×256 = 25,600 个位置。假设这个复合直方图包含了识别面部所需的诊断特征。因此，它们是面部图像的*表示*，而面部识别是通过比较这些表示，而不是图像本身来实现的。

要预测一个新的未知面孔的身份，你需要提取其连接直方图，并将其与训练数据库中的现有直方图进行比较。比较的方式是衡量直方图之间的距离。这个计算可以使用各种方法，包括欧几里得距离、绝对距离、卡方距离等。该算法返回与最接近的直方图匹配的训练图像的 ID 号码，并附带置信度测量。然后，你可以对置信度值应用阈值，如在[图 10-3](ch10.xhtml#ch010fig3)中所示。如果新图像的置信度低于阈值，则认为匹配为正。

由于 OpenCV 封装了所有这些步骤，LBPH 算法易于实现。它还在受控环境中产生了出色的结果，并且不受光照条件变化的影响（见[图 10-6](ch10.xhtml#ch010fig6)）。

![图片](../images/fig10_06.jpg)

图10-6：LBP对光照变化具有鲁棒性

LBPH算法能够很好地应对光照条件变化，因为它依赖于像素强度之间的比较。即使一张图像的照明比另一张图像要亮得多，面部的相对反射率依然保持不变，LBPH能够捕捉到这一点。

### **项目 #14：限制外星文物的访问**

你的队伍已经打进了包含产生传送门的外星文物的实验室。德明上校命令立即锁定该实验室，仅他自己可以访问。另一个技术员将用军用笔记本电脑覆盖当前系统。德明上校将通过这台笔记本电脑使用两级安全验证：一个是输入密码，另一个是面部识别。考虑到你在OpenCV方面的技能，他命令*你*负责面部验证部分。

目标

编写一个Python程序，能够识别德明上校的面孔。

#### ***策略***

由于时间紧迫并且工作环境不佳，你希望使用一个快速且易用的工具，且具有良好的性能记录，比如OpenCV的LBPH人脸识别器。你知道LBPH在受控环境下表现最佳，因此你将使用相同的笔记本电脑摄像头来捕捉训练图像和任何试图访问实验室的人的面孔。

除了德明上校的面部照片外，你还需要捕捉一些不属于德明上校的面孔。你将使用这些面孔来确保所有的正面匹配确实属于上校。无需担心设置密码、隔离程序与用户的操作，或者破解现有系统；另一个技术员会处理这些任务，而你则负责外出打击一些变异体。

#### ***支持模块和文件***

在这个项目中，你将主要使用OpenCV和NumPy。如果你还没有安装它们，请参考[第6页](ch01.xhtml#page_6)的“安装Python库”部分。你还需要playsound模块来播放声音，以及pyttsx3模块来实现文本转语音功能。你可以在[第207页](ch09.xhtml#page_207)的“代码”部分找到这些模块的更多信息，包括安装说明。

代码和支持文件位于书籍网站上的*Chapter_10*文件夹，[https://nostarch.com/real-world-python/](https://nostarch.com/real-world-python/)。下载后请保持文件夹结构和文件名不变（见[图10-7](ch10.xhtml#ch010fig7)）。注意，*tester*和*trainer*文件夹稍后会创建，并不会包含在下载内容中。

![图片](../images/fig10_07.jpg)

图10-7：项目14的文件结构

*demming_trainer*和*demming_tester*文件夹包含德明上校及其他人的图片，你可以用来完成这个项目。当前代码已引用这些文件夹。

如果你想提供自己的图像——例如，使用你自己的面部来代表 Captain Demming——那么你将使用名为 *trainer* 和 *tester* 的文件夹。接下来的代码将为你创建 *trainer* 文件夹。你需要手动创建 *tester* 文件夹并添加一些你的照片，稍后会有详细说明。当然，你需要编辑代码，使其指向这些新的文件夹。

#### ***视频捕获代码***

第一步（由 *1_capture.py* 代码执行）是捕捉你将用于训练识别器的面部图像。如果你打算使用 *demming_trainer* 文件夹中提供的图像，可以跳过此步骤。

要使用你自己的面部来表示 Captain Demming，使用计算机的摄像头录制大约十几张不同表情的面部照片，且不戴眼镜。如果你没有摄像头，可以跳过此步骤，使用手机自拍，并将其保存到名为 *trainer* 的文件夹中，如 [图10-7](ch10.xhtml#ch010fig7) 所示。

##### **导入模块，设置音频、摄像头、说明和文件路径**

[清单 10-1](ch10.xhtml#ch010list1) 导入模块，初始化并设置音频引擎和 Haar 级联分类器，初始化摄像头，并提供用户说明。你需要 Haar 级联分类器，因为在识别面部之前，必须先检测到它。关于 Haar 级联和面部检测的复习内容，请参阅 [第204页](ch09.xhtml#page_204) 中的“照片中的面部检测”。

```py
1_capture.py, part 1
   import os
   import pyttsx3
   import cv2 as cv
   from playsound import playsound

   engine = pyttsx3.init()
➊ engine.setProperty('rate', 145)
   engine.setProperty('volume', 1.0)

   root_dir = os.path.abspath('.')
   tone_path = os.path.join(root_dir, 'tone.wav')

➋ path = "C:/Python372/Lib/site-packages/cv2/data/"
   face_detector = cv.CascadeClassifier(path + 
                                        'haarcascade_frontalface_default.xml')
   cap = cv.VideoCapture(0)
   if not cap.isOpened(): 
       print("Could not open video device.")
➌ cap.set(3, 640)  # Frame width.
   cap.set(4, 480)  # Frame height.

   engine.say("Enter your information when prompted on screen. \
              Then remove glasses and look directly at webcam. \
              Make multiple faces including normal, happy, sad, sleepy. \
              Continue until you hear the tone.")
   engine.runAndWait()

➍ name = input("\nEnter last name: ")
   user_id = input("Enter assigned ID Number: ")
   print("\nCapturing face. Look at the camera now!")
```

清单 10-1：导入模块并设置音频和检测器文件、摄像头以及说明

导入的模块与上一章中用于检测面部的模块相同。你将使用操作系统（通过 os 模块）来操作文件路径，使用 pyttsx3 播放文本转语音音频说明，使用 cv 处理图像并运行面部检测器和识别器，使用 playsound 播放音效，提示用户程序何时完成捕获其图像。

接下来，设置文本转语音引擎。你将使用它告诉用户如何运行程序。默认的语音取决于你所使用的操作系统。该引擎的速率参数目前已针对 Windows 系统上的美国“David”语音进行了优化➊。如果你发现语速太快或太慢，可以编辑该参数。如果你想更改语音，请参阅 [清单 9-1](ch09.xhtml#ch09list1) 及 [第209页](ch09.xhtml#page_209) 上的说明。

你将使用音效提醒用户视频捕获过程已结束。设置 *tone.wav* 音频文件的路径，方法与 [第9章](ch09.xhtml) 中一样。

现在，提供 Haar 级联文件的路径 ➋，并将分类器分配给名为 face_detector 的变量。这里显示的路径是我的 Windows 计算机的路径；你的路径可能会有所不同。例如，在 macOS 上，你可以在 *opencv/data/haarcascades* 文件夹下找到这些文件。你也可以在网上找到它们，网址为 *[https://github.com/opencv/opencv/tree/master/data/haarcascades/](https://github.com/opencv/opencv/tree/master/data/haarcascades/)*。

在 [第 9 章](ch09.xhtml) 中，你学习了如何使用计算机的网络摄像头捕获你的面部图像。你将在本程序中使用类似的代码，从调用 cv.VideoCapture(0) 开始。0 参数指的是活动摄像头。如果你有多个摄像头，可能需要使用其他数字，例如 1，这可以通过尝试不同的数字来确定。使用条件语句检查摄像头是否成功打开，如果成功，则分别设置帧的宽度和高度 ➌。这两个方法中的第一个参数指的是宽度或高度参数在参数列表中的位置。

出于安全考虑，你将在视频捕获过程中进行监督。然而，你可以使用 pyttsx3 引擎来向用户解释该过程（这样你就不需要记住这些步骤了）。为了控制获取条件并确保后续的识别准确，用户需要摘掉眼镜或面部遮挡物，并展示多种表情。其中应包括他们每次进入实验室时计划使用的表情。

最后，他们需要按照屏幕上打印的指示操作。首先，他们将输入自己的姓氏 ➍。目前不需要担心重复问题，因为 Demming 阁下将是唯一的用户。此外，你将为用户分配一个唯一的 ID 号。OpenCV 将使用这个变量 user_id 来跟踪训练和预测过程中的所有面部图像。以后，你将创建一个字典，以便能够跟踪每个用户 ID 对应的是哪个人，假设未来会有更多人获得访问权限。

一旦用户输入他们的 ID 号并按下回车键，摄像头将开启并开始捕获图像，因此通过另一个调用 print() 来通知用户。记住在前一章中提到的 Haar 级联人脸检测器对头部姿态非常敏感。为了让它正常工作，用户必须直接面对网络摄像头并尽量保持头部直立。

##### **捕获训练图像**

[列表 10-2](ch10.xhtml#ch010list2) 使用网络摄像头和一个 while 循环来捕获指定数量的面部图像。代码将图像保存到一个文件夹中，并在操作完成时发出音调。

```py
1_capture.py, part 2
if not os.path.isdir('trainer'):
    os.mkdir('trainer')
os.chdir('trainer')

frame_count = 0

while True:
    # Capture frame-by-frame for total of 30 frames.
    _, frame = cap.read()
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
 ➊ face_rects = face_detector.detectMultiScale(gray, scaleFactor=1.2,
                                                minNeighbors=5)     
    for (x, y, w, h) in face_rects:
        frame_count += 1
        cv.imwrite(str(name) + '.' + str(user_id) + '.'
                   + str(frame_count) + '.jpg', gray[y:y+h, x:x+w])
        cv.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
        cv.imshow('image', frame)
        cv.waitKey(400)
 ➋ if frame_count >= 30:
        break

print("\nImage collection complete. Exiting...")
playsound(tone_path, block=False)
cap.release()
cv.destroyAllWindows()
```

列表 10-2：使用循环捕获视频图像

首先检查是否有名为 *trainer* 的目录。如果没有，使用操作系统模块的 mkdir() 方法创建该目录。然后，将当前工作目录更改为 *trainer* 文件夹。

现在，初始化 frame_count 变量为 0。只有当检测到人脸时，代码才会捕捉并保存视频帧。为了知道何时结束程序，你需要跟踪已捕捉帧的数量。

接下来，启动一个设置为 True 的 while 循环。然后调用 cap 对象的 read() 方法。如前一章所述，该方法返回一个元组，包含一个布尔值返回码和一个 numpy ndarray 对象，表示当前帧。返回码通常用于检查在从文件读取时是否已读完所有帧。由于我们这里不是从文件读取，所以将其赋值给一个下划线，以表示这是一个未使用的变量。

人脸检测和人脸识别都在灰度图像上工作，因此将帧转换为灰度图像，并将结果数组命名为 gray。接着，调用 detectMultiscale() 方法检测图像中的人脸 ➊。有关该方法如何工作的详细信息，请参考 [Listing 9-2](ch09.xhtml#ch09list2) 中的讨论，见 [第 212 页](ch09.xhtml#page_212)。由于你通过让用户注视笔记本的摄像头来控制条件，你可以放心地认为算法会表现良好，尽管你当然需要检查结果。

前面的方法应该输出围绕人脸的矩形框坐标。接着，开始一个 for 循环，遍历每一组坐标，并立即将 frame_count 变量加 1。

使用 OpenCV 的 imwrite() 方法将图像保存到 *trainer* 文件夹中。文件夹采用以下命名规则：*name.user_id.frame_count.jpg*（例如 *demming.1.9.jpg*）。只保存人脸矩形区域内的图像部分，这将有助于确保你不会训练算法识别背景特征。

接下来的两行代码会在原始帧上绘制一个人脸矩形，并显示它。这是为了让用户——Demming 阁下——能够检查自己的头部是否直立，表情是否适当。waitKey() 方法延迟了捕捉过程，给用户足够时间切换不同的表情。

即使 Demming 阁下在验证身份时总是采用放松的中性表情，对软件进行一系列表情的训练将导致更为健壮的结果。从这个角度来看，如果用户在捕捉阶段稍微*左右*摇动头部，也会有帮助。

接下来，检查目标帧数是否已达到，如果已经达到，则跳出循环 ➋。请注意，如果没有人对着摄像头，循环将永远运行下去。只有当级联分类器检测到人脸并返回人脸矩形时，才会计数帧数。

通过打印消息并发出提示音，让用户知道相机已经关闭。然后，释放相机并销毁所有图像窗口，从而结束程序。

此时，*trainer* 文件夹中应该包含 30 张用户的紧密裁剪人脸图像。在接下来的部分中，你将使用这些图像——或 *demming_trainer* 文件夹中提供的图像集——来训练 OpenCV 的人脸识别器。

#### ***面部训练器代码***

下一步是使用OpenCV创建一个基于LBPH的面部识别器，使用训练图像对其进行训练，并将结果保存为可重复使用的文件。如果你使用自己的面部图像来代表Captain Demming的面部，你需要将程序指向*trainer*文件夹。否则，你将需要使用*demming_trainer*文件夹，和包含代码的*2_train.py*文件，这些都位于可下载的*Chapter_10*文件夹中。

[列表 10-3](ch10.xhtml#ch010list3)设置了用于面部检测的Haar级联路径和前一个程序捕捉的训练图像。OpenCV通过标签整数跟踪面部，而不是使用名称字符串，列表还初始化了用于存储标签及其相关图像的列表。接着，它遍历训练图像，加载它们，从文件名中提取用户ID号，并进行面部检测。最后，它训练识别器并将结果保存到文件中。

```py
2_train.py
   import os
   import numpy as np
   import cv2 as cv

   cascade_path = "C:/Python372/Lib/site-packages/cv2/data/"
   face_detector = cv.CascadeClassifier(cascade_path +
                                       'haarcascade_frontalface_default.xml')

➊ train_path = './demming_trainer'  # Use for provided Demming face.   
   #train_path = './trainer'  # Uncomment to use your face.
   image_paths = [os.path.join(train_path, f) for f in os.listdir(train_path)]
   images, labels = [], []

   for image in image_paths:
       train_image = cv.imread(image, cv.IMREAD_GRAYSCALE)
    ➋ label = int(os.path.split(image)[-1].split('.')[1])
       name = os.path.split(image)[-1].split('.')[0]
       frame_num = os.path.split(image)[-1].split('.')[2]
    ➌ faces = face_detector.detectMultiScale(train_image)
       for (x, y, w, h) in faces:
           images.append(train_image[y:y + h, x:x + w])
           labels.append(label)
           print(f"Preparing training images for {name}.{label}.{frame_num}")
           cv.imshow("Training Image", train_image[y:y + h, x:x + w])
           cv.waitKey(50) 

   cv.destroyAllWindows()

➍ recognizer = cv.face.LBPHFaceRecognizer_create()
   recognizer.train(images, np.array(labels))
   recognizer.write('lbph_trainer.yml')
   print("Training complete. Exiting...")
```

列表 10-3：训练并保存LBPH面部识别器

你之前已经看到过导入和面部检测器的代码。虽然你已经在*1_capture.py*中将训练图像裁剪成面部矩形，但重复这个过程并不会有坏处。由于*2_train.py*是一个独立程序，最好不要对任何事情掉以轻心。

接下来，你必须选择使用哪一组训练图像：你自己在*trainer*文件夹中捕捉的图像，还是*demming_trainer*文件夹中提供的图像集 ➊。注释掉或删除不使用的那一行。记住，由于你没有提供文件夹的完整路径，你需要从包含该文件夹的上一级文件夹启动程序，即*trainer*和*demming_trainer*文件夹的父文件夹。

使用列表推导创建一个名为image_paths的列表。它将保存训练文件夹中每个图像的目录路径和文件名。然后创建空列表来存储图像及其标签。

开始一个for循环，遍历图像路径。以灰度模式读取图像；然后从文件名中提取其数字标签，并将其转换为整数➋。记住，标签对应的是通过*1_capture.py*输入的用户ID，恰好是在它捕捉视频帧之前。

让我们花一点时间来解析一下这个提取和转换过程发生了什么。os.path.split()方法接受一个目录路径并返回一个包含目录路径和文件名的元组，如下所示：

```py
>>> import os
>>> path = 'C:\demming_trainer\demming.1.5.jpg'
>>> os.path.split(path)
('C:\\demming_trainer', 'demming.1.5.jpg')
```

然后，你可以使用索引-1选择元组中的最后一个项目，并根据点分割它。这将产生一个包含四个项目的列表（用户的姓名、用户ID、帧编号和文件扩展名）。

```py
>>> os.path.split(path)[-1].split('.')
['demming', '1', '5', 'jpg']
```

为了提取标签值，你可以通过索引1选择列表中的第二个项目。

```py
>>> os.path.split(path)[-1].split('.')[1]
'1'
```

重复此过程以提取每个图像的名称和frame_num。这些目前都是字符串，因此你需要将用户ID转换为整数，以便作为标签使用。

现在，调用人脸检测器处理每一张训练图像 ➌。这将返回一个`numpy.ndarray`，我们称之为`faces`。开始循环遍历数组，它包含了检测到的人脸矩形的坐标。将矩形区域内的图像部分添加到之前创建的图像列表中。同时，将图像的用户ID添加到标签列表中。

通过在终端打印信息来让用户了解当前的进展。然后，作为检查，显示每张训练图像50毫秒。如果你曾看过彼得·加布里尔1986年流行的音乐视频《Sledgehammer》，你会喜欢这种展示方式。

现在是训练人脸识别器的时候了。就像使用OpenCV的人脸检测器一样，你首先要实例化一个识别器对象 ➍。接着，调用`train()`方法，将图像列表和标签列表传递给它，并实时将其转换为NumPy数组。

你不希望每次有人验证他们的面孔时都重新训练识别器，所以将训练过程的结果写入名为*lbph_trainer.yml*的文件中。然后让用户知道程序已结束。

#### ***人脸预测代码***

现在是时候开始识别人脸了，这个过程我们称之为*预测*，因为它归结为概率问题。*3_predict.py*中的程序首先会计算每张人脸的连接LBP直方图。然后，它会计算该直方图与训练集中所有直方图之间的距离。接着，如果该距离低于你指定的阈值，它将为新的面孔分配与训练面孔最接近的标签和名称。

##### **导入模块并准备人脸识别器**

[清单 10-4](ch10.xhtml#ch010list4)导入了模块，准备了一个字典来保存用户ID号和姓名，设置了人脸检测器和识别器，并建立了测试数据的路径。测试数据包括德明队长的图像，以及其他几张人脸图像。为了测试结果，还包含了训练文件夹中的一张德明队长的图像。如果一切正常，算法应该能准确识别这张图像，并给出较低的距离度量。

```py
3_predict.py, part 1
   import os
   from datetime import datetime
   import cv2 as cv

   names = {1: "Demming"}
   cascade_path = "C:/Python372/Lib/site-packages/cv2/data/"
   face_detector = cv.CascadeClassifier(cascade_path +
                                        'haarcascade_frontalface_default.xml')

➊ recognizer = cv.face.LBPHFaceRecognizer_create()
   recognizer.read('lbph_trainer.yml')

   #test_path = './tester'
➋ test_path = './demming_tester'
   image_paths = [os.path.join(test_path, f) for f in os.listdir(test_path)]
```

清单 10-4：导入模块并为人脸检测与识别做准备

在导入一些常见的模块后，创建一个字典，将用户ID号与用户名关联起来。尽管目前只有一个条目，但这个名称字典使得将来可以方便地添加更多条目。如果你使用的是自己的照片，可以随意更改最后一个名字，但ID号要保持为1。

接下来，重复设置`face_detector`对象的代码。你需要输入自己的`cascade_path`（参见[清单 10-1](ch10.xhtml#ch010list1)在[第233页](ch10.xhtml#page_233)）。

如同在*2_train.py*代码中一样创建一个识别器对象 ➊。然后使用`read()`方法加载包含训练信息的*.yml*文件。

你需要使用文件夹中的人脸图像来测试识别器。如果你使用的是提供的Demming图像，请设置指向*demming_tester*文件夹的路径 ➋。否则，使用你之前创建的*tester*文件夹。你可以将自己的图像添加到这个空白文件夹中。如果你用自己的脸来代表Demming队长的脸，你不应该在这里重复使用训练图像，尽管你可以考虑使用其中一张作为对照。相反，使用*1_capture.py*程序生成一些新的图像。如果你戴眼镜，可以包括一些戴眼镜和不戴眼镜的照片。你还需要包括一些来自*demming_tester*文件夹中的陌生人的图像。

##### **人脸识别与访问日志更新**

[清单10-5](ch10.xhtml#ch010list5)循环遍历测试文件夹中的图像，检测任何存在的人脸，将人脸的直方图与训练文件中的进行比较，命名人脸，分配置信度值，然后将姓名和访问时间记录到一个持久化的文本文件中。在此过程中，程序理论上会在ID为正时解锁实验室，但由于我们没有实验室，所以我们会跳过这部分。

```py
3_predict.py, part 2
for image in image_paths:
    predict_image = cv.imread(image, cv.IMREAD_GRAYSCALE)
    faces = face_detector.detectMultiScale(predict_image,
                                          scaleFactor=1.05,
                                          minNeighbors=5)
    for (x, y, w, h) in faces:
        print(f"\nAccess requested at {datetime.now()}.")
 ➊ face = cv.resize(predict_image[y:y + h, x:x + w], (100, 100))   
    predicted_id, dist = recognizer.predict(face)
 ➋ if predicted_id == 1 and dist <= 95:
        name = names[predicted_id]
        print("{} identified as {} with distance={}"
              .format(image, name, round(dist, 1)))
     ➌ print(f"Access granted to {name} at {datetime.now()}.",
              file=open('lab_access_log.txt', 'a'))
    else:
        name = 'unknown'
        print(f"{image} is {name}.")

    cv.rectangle(predict_image, (x, y), (x + w, y + h), 255, 2)
    cv.putText(predict_image, name, (x + 1, y + h - 5),
               cv.FONT_HERSHEY_SIMPLEX, 0.5, 255, 1)        
    cv.imshow('ID', predict_image)        
    cv.waitKey(2000)
    cv.destroyAllWindows()
```

清单10-5：运行人脸识别并更新访问日志文件

从测试文件夹中的图像开始循环。这将是*demming_tester*文件夹或*tester*文件夹。将每张图像读取为灰度图像，并将结果数组赋给名为predict_image的变量。然后对其运行人脸检测器。

现在像之前一样循环遍历人脸矩形。打印出一个关于访问请求的消息；然后使用OpenCV将人脸子数组调整为100×100像素 ➊。这接近*demming_trainer*文件夹中训练图像的尺寸。同步图像的大小严格来说不是必需的，但根据我的经验，有助于提高结果。如果你使用自己的图像代表Demming队长的脸，你应该检查训练图像和测试图像的尺寸是否相似。

现在是时候预测人脸的身份了。这样做只需要一行代码。只需在识别器对象上调用predict()方法，并传入人脸子数组。该方法将返回一个ID号和一个距离值。

距离值越低，预测的人脸被正确识别的可能性就越大。你可以使用距离值作为阈值：所有被预测为Demming队长并且得分*等于或低于*阈值的图像将被确定为Demming队长。其他所有图像将被分配为“未知”。

要应用阈值，使用if语句➋。如果你使用的是自己的训练和测试图像，第一次运行程序时，将距离值设置为1000。查看测试文件夹中所有图像的距离值，包括已知和未知图像。找到一个阈值，在该阈值以下，所有面孔都能正确识别为Demming队长。这将成为你接下来使用的区分值。对于*demming_trainer*和*demming_tester*文件夹中的图像，阈值距离应该是95。

接下来，使用预测的_id值作为名称字典的键来获取图像的名称。 在命令行中打印一条消息，表明图像已被识别，并包括图像文件名、字典中的名称和距离值。

对于日志，打印一条消息，表明某个名字（在本例中是Demming队长）已获得实验室访问权限，并包括使用datetime模块的时间➌。

你会想保留一份人员进出记录的持久文件。这里有一个很巧妙的技巧：只需使用print()函数写入文件。打开*lab_access_log.txt*文件，并包含一个“附加”参数。这样，程序每次处理新图像时，不会覆盖文件，而是在文件底部添加一行。以下是文件内容的示例：

```py
Access granted to Demming at 2020-01-20 09:31:17.415802.
Access granted to Demming at 2020-01-20 09:31:19.556307.
Access granted to Demming at 2020-01-20 09:31:21.644038.
Access granted to Demming at 2020-01-20 09:31:23.691760.
--snip--
```

如果条件不满足，将name设置为'unknown'并打印相应的消息。然后在面部周围绘制矩形，并使用OpenCV的putText()方法显示用户的名字。显示图像两秒钟后销毁它。

#### ***结果***

你可以在[图 10-8](ch10.xhtml#ch010fig8)中查看一些来自*demming_tester*文件夹的20张图片的示例结果。预测器代码正确识别了8张Demming队长的照片，没有出现误识别。

![Image](../images/fig10_08.jpg)

图 10-8：Demming队长和非Demming队长

为了使LBPH算法具有高准确性，你需要在受控条件下使用它。记住，通过强制用户通过笔记本电脑获取访问权限，你控制了他们的姿势、面部大小、图像分辨率和光照条件。

### **总结**

在本章中，你将使用OpenCV的局部二值模式直方图算法来识别人脸。通过仅几行代码，你就创建了一个强大的面部识别器，能够轻松应对不同的光照条件。你还使用了标准库的os.path.split()方法来分割目录路径和文件名，从而生成自定义的变量名。

### **进一步阅读**

“应用局部二值模式于人脸检测与识别”（加泰罗尼亚理工大学，2010年），由Laura María Sánchez López编写，是对LBPH方法的全面回顾。PDF可以在像* [https://www.semanticscholar.org/](https://www.semanticscholar.org/)*这样的网站上找到。

“查看 LBP 直方图”，在 AURlabCVsimulator 网站上 (*[https://aurlabcvsimulator.readthedocs.io/en/latest/](https://aurlabcvsimulator.readthedocs.io/en/latest/)*)，包含了可以让你可视化 LBPH 图像的 Python 代码。

如果你是 macOS 或 Linux 用户，一定要查看 Adam Geitgey 的 face_recognition 库，这是一个简单易用且高精度的人脸识别系统，利用深度学习技术。你可以在 Python 软件基金会网站上找到安装说明和概述：*[https://pypi.org/project/face_recognition/](https://pypi.org/project/face_recognition/)*

“机器学习很有趣！第四部分：使用深度学习进行现代人脸识别”（Medium，2016），作者 Adam Geitgey，是一篇简短且有趣的概述，介绍了如何使用 Python、OpenFace 和 dlib 进行现代人脸识别。

“使用 OpenCV 进行活体检测”（PyImageSearch，2019），作者 Adrian Rosebrock，是一篇在线教程，教你如何保护人脸识别系统免受伪造面孔的欺骗，例如将 Captain Demming 的照片举到网络摄像头前。

世界各地的城市和大学已开始禁止使用人脸识别系统。发明家们也纷纷行动起来，设计出能够迷惑系统、保护身份的衣物。“这些衣服使用离奇的设计来欺骗面部识别软件，让它认为你不是人类”（《商业内幕》，2020），作者 Aaron Holmes，以及“如何黑掉你的面孔，躲避人脸识别技术的崛起”（《连线》，2019），作者 Elise Thomas，回顾了这一问题的一些近期实用——也有不太实用的——解决方案。

“使用 OpenCV 进行深度学习年龄检测”（PyImageSearch，2020），作者 Adrian Rosebrock，是一篇在线教程，介绍如何使用 OpenCV 从照片中预测一个人的年龄。

### **挑战项目：添加密码和视频捕捉**

你在项目 14 中编写的 *3_predict.py* 程序，循环处理一个文件夹中的照片进行人脸识别。现在，重写该程序，使其能够动态识别网络摄像头视频流中的人脸。人脸矩形框和名称应该与文件夹中的图像一样出现在视频帧中。

要启动程序，让用户输入一个密码并进行验证。如果密码正确，则添加音频指示，告诉用户看向摄像头。如果程序成功识别出 Captain Demming，则用音频宣布授权访问。否则，播放一条音频消息，表示访问被拒绝。

如果你需要帮助识别视频流中的人脸，请参见附录中的 *challenge_video_recognize.py* 程序。注意，你可能需要为视频帧使用比静态照片更高的置信度值。

为了方便你跟踪谁尝试进入实验室，将一帧图像保存到与*lab_access_log.txt*文件相同的文件夹中。使用从datetime.now()获取的日志结果作为文件名，这样你就可以将面部与访问尝试进行匹配。注意，你需要重新格式化datetime.now()返回的字符串，以使其仅包含操作系统允许用作文件名的字符。

### **挑战项目：相似面孔和双胞胎**

使用第14项目中的代码比较名人相似面孔和双胞胎。用互联网上的图像进行训练，看看你能否欺骗LBPH算法。可以考虑的一些搭配包括：斯嘉丽·约翰逊与安柏·赫德、艾玛·沃森与基尔南·希普卡、利亚姆·海姆斯沃斯与卡伦·哈恰诺夫、罗布·洛与伊恩·萨默霍尔德、希拉里·达夫与维多利亚·佩德雷蒂、布莱斯·达拉斯·霍华德与杰西卡·查斯坦、威尔·法瑞尔与查德·史密斯。

对于著名的双胞胎，可以看看宇航员双胞胎马克·凯利和斯科特·凯利，或者名人双胞胎玛丽-凯特·奥尔森和艾希丽·奥尔森。

### **挑战项目：时光机**

如果你曾经观看过老节目重播，你会看到一些著名演员年轻时——有时是*非常*年轻时的模样。尽管人类在面部识别方面表现出色，我们仍然可能很难认出年轻时的伊恩·麦凯伦或帕特里克·斯图尔特。这就是为什么有时需要某种语调的变化或奇特的举止，才能促使我们赶紧去Google查找演员表。

面部识别算法在跨时间识别面孔时也容易出错。为了查看LBPH算法在这些条件下的表现，可以使用第14项目中的代码，并用你自己（或你亲戚）某个年龄阶段的面部图像进行训练。然后，使用不同年龄段的图像进行测试。
