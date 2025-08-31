

## 第三十五章：完全功能的批处理改进版



![](img/chapter.jpg)

在第十五章中，我构建了一个简单的*BatchImprov.bat*版本，它只共享一个笑话、谜语或双关语。在后来的章节中，你学到了几种工具来增强这个过程，例如读取文件、加载数组和使用随机数从数组中选择一个任意元素。

现在让我们将所有内容结合到一个增强版的批处理文件中，它首先读取包含任意数量笑话、谜语和双关语的库文件，并将它们加载到数组中。然后，BUI 会随机检索并分享一个用户请求的幽默示例，并询问他们是否想要另一个示例。

从数据开始，以下是*BatJokes.txt*的完整内容：

```
Why are bats so active at night?|They charge their bat-teries by day.
How do bats flirt?|They bat their eyes.
What's a good pick-up line for a bat?|Let's hang.
Why did the bat cross the road?|To prove he wasn't chicken. 
```

*BatRiddles.txt*类似地包含谜语：

```
This type of bat is silly.|A Dingbat.
This circus performer can see in the dark.|An Acro-bat.
This is the strongest and meanest bat in the cave.|The Alpha-bat.
This sport uses bats and is also food for bats.|Cricket. 
```

这些双关语的格式不同，因此每个*BatPuns.txt*记录中不包含由管道分隔的答案：

```
Crossing a vampire bat with a computer means love at first byte.
The first thing bat pups learn at school is the alpha-bat.
Bat pups are trained to go potty in the bat-room. 
```

最后，将这三个库文件放置在与此版本的*BatchImprov.bat*相同的目录中：

```
 @setlocal EnableExtensions EnableDelayedExpansion
 @echo off
 color 1F
 title Batch Improv Theater

 call :LoadArray joke
 call :LoadArray riddle
 call :LoadArray pun
 pause

:Again
 cls
 > con echo.
 > con choice /C:JPR /M:"Do you want a Joke, Pun, or Riddle"
 > con echo.
 if %errorlevel% equ 1 (
    call :Joke
 ) else if %errorlevel% equ 2 (
    call :Pun
 ) else if %errorlevel% equ 3 (
    call :Riddle
 )
 > con echo.
 > con choice /M:"Do you want to try again"
 if %errorlevel% equ 1  goto :Again
 goto :eof

:Joke
 call :GetRandNbr joke
 > con echo Please give an answer to the joke:
 > con set /P yourAns=!joke[%randNbr%]!  &
 > con echo ** !jokeAns[%randNbr%]!
 > con echo ** You said: "%yourAns%"
 goto :eof

:Pun
 call :GetRandNbr pun
 > con echo We hope you find this punny:
 > con echo !pun[%randNbr%]!
 goto :eof

:Riddle
 call :GetRandNbr riddle
 > con echo Please give an answer to the riddle:
 > con set /P yourAns=!riddle[%randNbr%]!  &
 > con echo ** !riddleAns[%randNbr%]!
 > con echo ** You said: "%yourAns%"
 goto :eof

:LoadArray
 set %1sTot=0
 for /F "tokens=1-2 delims=|" %%b in (Bat%1s.txt) do (
    set %1[!%1sTot!]=%%~b
    set %1Ans[!%1sTot!]=%%~c
    set /A %1sTot += 1
 )
 > con echo.
 > con echo Results of array load of %1s:
 > con set %1
 goto :eof

:GetRandNbr
 set nbrPossVal=!%1sTot!
 set /A maxRandNbr = 32768 / %nbrPossVal% * %nbrPossVal% - 1
:GetAnotherRand
 set randNbr=%random%
 if %randNbr% gtr %maxRandNbr%  goto :GetAnotherRand
 set /A randNbr = %randNbr% %% %nbrPossVal%
 goto :eof 
```

这部分批处理文件的大部分内容应该看起来很熟悉，但也有很多新的部分。我多次调用:LoadArray，并将笑话、谜语或双关语作为参数传递给它。这个例程类似于第二十九章中的一些代码，使用这些文本查找并读取当前目录中的特定文件，并构建适当命名的数组。

一个不太熟练的程序员可能会先让笑话部分工作，然后才将其克隆到谜语和双关语部分。相反，我使用了通用代码，其中第一次调用填充了笑话和笑话答案数组，并将 jokesTot 设置为加载到数组中的笑话总数，尽管实际的变量名从未出现在批处理文件中。我通过将参数解析为%1sTot 的一部分来创建该变量。

第二次调用类似地填充了谜语和谜语答案数组，并设置了 riddlesTot 变量。但双关语的格式不同。由于没有管道符号且没有答案，因此没有第二个参数，代码不会填充答案数组。相反，相同的逻辑构建了双关语数组，并将 punsTot 设置为数组中双关语的数量。

你可以稍后移除它，但出于测试目的，我将每次加载的结果显示到控制台：

```
Results of array load of jokes:
jokeAns[0]=They charge their bat-teries by day.
jokeAns[1]=They bat their eyes.
jokeAns[2]=Let's hang.
jokeAns[3]=To prove he wasn't chicken.
jokesTot=4
joke[0]=Why are bats so active at night?
joke[1]=How do bats flirt?
joke[2]=What's a good pick-up line for a bat?
joke[3]=Why did the bat cross the road?

Results of array load of riddles:
riddleAns[0]=A Dingbat.
riddleAns[1]=An Acro-bat.
riddleAns[2]=The Alpha-bat.
riddleAns[3]=Cricket.
riddlesTot=4
riddle[0]=This type of bat is silly.
riddle[1]=This circus performer can see in the dark.
riddle[2]=This is the strongest and meanest bat in the cave.
riddle[3]=This sport uses bats and is also food for bats.

Results of array load of puns:
punsTot=3
pun[0]=Crossing a vampire bat with a computer means love at first byte.
pun[1]=The first thing bat pups learn at school is the alpha-bat.
pun[2]=Bat pups are trained to go potty in the bat-room.
Press any key to continue ... 
```

cls 命令在开始批处理文件的用户界面部分之前清除屏幕。

:Again 标签下的主要逻辑与之前版本的批处理文件没有变化。:Joke、:Riddle 和:Pun 例程通过调用:GetRandNbr 获取一个随机数。为了获得适当数组中的元素总数，例程将其参数解析为!%1sTot!的一部分。其余的逻辑与你在第二十一章中看到的类似。

在获取它们数组（或数组）的指针后，这些例程与之前的版本相似，不同之处在于它们从数组中获取内容。例如，!joke[%randNbr%]! 解析为一个笑话，!jokeAns[%randNbr%]! 解析为其答案。（延迟扩展真是太棒了。）

现在你可以运行*BatchImprov.bat*来获取多个笑话、谜语和双关语。你甚至可以在不修改代码的情况下向库文件中添加更多内容。更棒的是，可以将其作为使用 BUI、数组、分隔数据文件和随机数的应用程序模板。尽情享受吧。
