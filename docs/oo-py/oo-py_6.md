self.gameOverImage = pygwidgets.Image(self.window, (140, 180),

'images/gameOver.png')

self.titleText = pygwidgets.DisplayText(self.window,

(70, GAME_HEIGHT + 17),

'得分：最高得分：',

fontSize=24, textColor=WHITE)

self.scoreText = pygwidgets.DisplayText(self.window,

(80, GAME_HEIGHT + 47), '0',

fontSize=36, textColor=WHITE,

justified='right')

self.highScoreText = pygwidgets.DisplayText(self.window,

(270, GAME_HEIGHT + 47), '',

fontSize=36, textColor=WHITE,

justified='right')

pygame.mixer.music.load('sounds/background.mid')

self.dingSound = pygame.mixer.Sound('sounds/ding.wav')

self.gameOverSound = pygame.mixer.Sound('sounds/gameover.wav')

# 实例化对象

2 self.oPlayer = Player(self.window)

self.oBaddieMgr = BaddieMgr(self.window)

self.oGoodieMgr = GoodieMgr(self.window)

self.highestHighScore = 0

self.lowestHighScore = 0

self.backgroundMusic = True

self.score = 0

3 self.playingState = STATE_WAITING

4 def getSceneKey(self):

return SCENE_PLAY

*Listing 16-3: ScenePlay 类的 __init__() 和 getSceneKey() 方法*

当游戏运行时，主代码实例化所有场景。在

Play 场景中，__init__() 方法创建了所有按钮和文本显示

窗口底部的字段 1，然后加载声音。非常重要-

同时，我们使用组合，详见第 4 章和第 10 章，来创建

玩家对象 (oPlayer)，一个坏蛋管理器对象 (oBaddieMgr)，和一个好蛋

管理器对象 (oGoodieMgr) 2. Play 场景对象创建这些管理器并期望它们创建并管理所有的坏蛋和好蛋。

这些管理器并期望它们创建并管理所有的坏蛋和好蛋。

__init__() 方法在程序启动时运行，但实际上并不

启动游戏。而不是这样，它实现了一个状态机（如章节中所讨论）

[第 15 章](index_split_006.html#p340) 在等待状态 3 中开始。游戏的一轮开始时，用户按下新游戏按钮。

完整游戏：Dodger **351**

所有场景必须有一个 getSceneKey() 方法 4，返回一个表示当前场景的字符串。Listing 16-4 显示了获取

分数并根据请求重置游戏。

1 def enter(self, data):

self.getHiAndLowScores()

2 def getHiAndLowScores(self):

# 向高分场景请求一个分数字典

# 看起来像这样：

# {'highest': highestScore, 'lowest': lowestScore}

3 infoDict = self.request(SCENE_HIGH_SCORES, HIGH_SCORES_DATA)

self.highestHighScore = infoDict['highest']

self.highScoreText.setValue(self.highestHighScore)

self.lowestHighScore = infoDict['lowest']

4 def reset(self): # 开始一个新游戏

self.score = 0

self.scoreText.setValue(self.score)

self.getHiAndLowScores()

# 告诉管理器重置自己

5 self.oBaddieMgr.reset()

self.oGoodieMgr.reset()

if self.backgroundMusic:

pygame.mixer.music.play(-1, 0.0)

6 self.startButton.disable()

self.highScoresButton.disable()

self.soundCheckBox.disable()

self.quitButton.disable()

pygame.mouse.set_visible(False)

*Listing 16-4: ScenePlay 类的 enter()、getHiAndLowScores() 和 reset() 方法*

当导航到 Play 场景时，场景管理器调用 enter() 1，

该方法依次调用 getHiAndLowScores() 方法 2。该方法发出

请求前往高分榜场景 3，获取最高和最低分数

从高分榜表中获取，因此我们可以从该表中绘制最高分

在窗口底部的条形区域。在每场游戏结束时，它会

将游戏的分数与最低的前 10 名分数进行比较，看看这场游戏是否排名

前 10 名\。

当用户点击“新游戏”按钮时，reset() 方法 4

被调用以重新初始化所有在开始新一轮游戏前需要重置的内容

新一轮游戏的开始。reset() 方法告诉 Baddie 管理器和

通过调用自己的 reset() 方法，通知 Goodie 管理器重新初始化自己

方法 5，禁用屏幕底部的按钮，这样它们就无法

游戏中不能按下 6，并且隐藏光标。在播放时，

用户通过移动鼠标来控制窗口中的 Player 图标。

清单 16-5 中的代码处理用户输入。

**352** 第 16 章

1 定义 handleInputs(self, eventsList, keyPressedList):

2 如果 self.playingState == STATE_PLAYING:

返回 # 在播放时忽略按钮事件

for event in eventsList:

3 如果 self.startButton.handleEvent(event):

self.reset()

self.playingState = STATE_PLAYING

4 如果 self.highScoresButton.handleEvent(event):

self.goToScene(SCENE_HIGH_SCORES)

5 如果 self.soundCheckBox.handleEvent(event):

self.backgroundMusic = self.soundCheckBox.getValue()

6 如果 self.quitButton.handleEvent(event):

self.quit()

*清单 16-5：ScenePlay 类的 handleInputs() 方法*

handleInputs() 方法 1 负责点击事件。如果状态

在机器处于播放状态时，用户无法点击按钮，因此我们

不用检查事件 2。如果用户点击“新游戏”3，我们

调用 reset() 重新初始化变量并将状态机更改为

播放状态。如果用户点击“前往高分榜”4，我们导航到

使用继承自 self.goToScene() 方法的 High Scores 场景。如果用户

切换背景音乐复选框 5，我们调用其 getValue() 方法

获取其新设置；reset() 方法利用这个设置决定是否

应播放背景音乐。如果用户点击“退出”6，我们调用

继承自基类的 self.quit() 方法。清单 16-6 显示了

游戏实际玩法的代码。

1 定义 update(self):

如果 self.playingState != STATE_PLAYING:

返回 # 仅在播放时更新

# 将 Player 移动到鼠标位置，并获取其矩形区域

2 mouseX, mouseY = pygame.mouse.get_pos()

playerRect = self.oPlayer.update(mouseX, mouseY)

# 告诉 GoodieMgr 移动所有 Goodies

# 返回玩家接触到的 Goodies 数量

3 nGoodiesHit = self.oGoodieMgr.update(playerRect)

如果 nGoodiesHit > 0:

self.dingSound.play()

self.score = self.score + (nGoodiesHit * POINTS_FOR_GOODIE)

# 告诉 BaddieMgr 移动所有 Baddies

# 返回从底部掉落的 Baddies 数量

4 nBaddiesEvaded = self.oBaddieMgr.update()

self.score = self.score + (nBaddiesEvaded * POINTS_FOR_BADDIE_EVADED)

self.scoreText.setValue(self.score)

完整游戏：Dodger **353**

# 检查 Player 是否击中任何 Baddie

5 如果 self.oBaddieMgr.hasPlayerHitBaddie(playerRect):

pygame.mouse.set_visible(True)

pygame.mixer.music.stop()

self.gameOverSound.play()

self.playingState = STATE_GAME_OVER

6 self.draw() # 强制绘制游戏结束消息

7 如果 self.score > self.lowestHighScore:

scoreAsString = ′您的得分：′ + str(self.score) + ′\n′

如果 self.score > self.highestHighScore:

dialogText = (scoreString +

'是新的高分，恭喜！'

否则：

dialogText = (scoreString +

'让你进入高分榜。'

result = showCustomYesNoDialog(self.window, dialogText)

if result: # 导航

self.goToScene(SCENE_HIGH_SCORES, self.score)

self.startButton.enable()

self.highScoresButton.enable()

self.soundCheckBox.enable()

self.quitButton.enable()

*列表 16-6：ScenePlay 类的 update() 方法*

场景管理器调用 ScenePlay 类的 update() 方法 1

每一帧都处理。此方法处理游戏过程中发生的所有事情。

游戏正在进行。首先，它告诉 Player 对象移动 Player 的图标到

鼠标的位置。然后它调用 Player 的 update() 方法 2，

它返回图标在窗口中的当前矩形。我们用这个

看 Player 的图标是否接触到任何 Goodies 或 Baddies。

接下来，它调用 Goodie 管理器的 update() 方法 3 来移动所有

Goodies。该方法返回 Player 获得的 Goodies 数量。

已接触，我们用它来增加得分。

接下来调用 Baddie 管理器的 update() 方法 4

移动所有的 Baddies。该方法返回 Baddies 的数量。

已经从游戏区域底部掉落的物体。

然后我们检查 Player 是否接触到任何 Baddies 5。如果有，

游戏结束时，我们展示一个游戏结束图形。我们还会特别

特别调用我们的 draw() 方法 6，因为我们可能会弹出一个对话框给

用户，游戏的主循环不会绘制游戏结束图形，直到

用户点击对话框中的任意按钮。

最后，当游戏结束时，如果当前游戏得分高于

第十名得分 7，我们会弹出一个对话框，给用户选择

将他们的得分记录到高分榜中。如果当前的分数

如果游戏的得分是新的历史最高分，我们会在对话框中显示特别的消息。

列表 16-7 中的代码绘制了游戏角色。

**354** 第 16 章

1 def draw(self):

self.window.fill(BLACK)

# 告诉管理器绘制所有的 Baddies 和 Goodies

self.oBaddieMgr.draw()

self.oGoodieMgr.draw()

# 告诉 Player 绘制自身

self.oPlayer.draw()

# 绘制窗口底部的所有信息

2 self.controlsBackground.draw()

self.titleText.draw()

self.scoreText.draw()

self.highScoreText.draw()

self.soundCheckBox.draw()

self.quitButton.draw()

self.highScoresButton.draw()

self.startButton.draw()

3 如果 self.playingState == STATE_GAME_OVER:

self.gameOverImage.draw()

4 def leave(self):

pygame.mixer.music.stop()

*列表 16-7：ScenePlay 类的 draw() 和 leave() 方法*

draw() 方法告诉玩家绘制自己和 Goodie， 

Baddie 管理器绘制所有 Goodies 和 Baddies 1\。然后我们绘制

窗口的底部区域 2 包含所有按钮和文本显示字段。

如果我们处于游戏结束状态 3，我们绘制游戏结束图像。

当用户离开当前场景时，场景管理器调用

leave() 方法 4 并停止任何音乐。

**文件：Dodger/Baddies.py**

*Baddies.py* 文件包含两个类：Baddie 和 BaddieMgr。Play 场景创建了单个 Baddie 管理器对象，后者创建并维护一个

所有 Baddie 的列表。Baddie 管理器实例化来自 Baddie 的对象

类每隔几帧更新一次，基于计时器。列表 16-8 包含了

Baddie 类。

# Baddie 类

--- 省略导入 ---

class Baddie():

MIN_SIZE = 10

MAX_SIZE = 40

MIN_SPEED = 1

MAX_SPEED = 8

# 仅加载一次图像

完整游戏：Dodger **355**

1 BADDIE_IMAGE = pygame.image.load('images/baddie.png')

def __init__(self, window):

self.window = window

# 创建图像对象

size = random.randrange(Baddie.MIN_SIZE, Baddie.MAX_SIZE + 1)

self.x = random.randrange(0, WINDOW_WIDTH - size)

self.y = 0 - size # 从窗口上方开始

2 self.image = pygwidgets.Image(self.window, (self.x, self.y),

Baddie.BADDIE_IMAGE)

# 缩放它

percent = (size * 100) / Baddie.MAX_SIZE

self.image.scale(percent, False)

self.speed = random.randrange(Baddie.MIN_SPEED,

Baddie.MAX_SPEED + 1)

3 def update(self): # 将 Baddie 向下移动

self.y = self.y + self.speed

self.image.setLoc((self.x, self.y))

if self.y > GAME_HEIGHT:

return True # 需要删除

else:

return False # 保持在窗口内

4 def draw(self):

self.image.draw()

5 def collide(self, playerRect):

collidedWithPlayer = self.image.overlaps(playerRect)

return collidedWithPlayer

*列表 16-8：Baddie 类*

我们将 Baddie 的图像作为类变量加载 1，这样就只有一个

图像被 *所有* Baddie 共享。

__init__() 方法 2 为每个新的 Baddie 选择一个随机大小，

这样用户就能看到不同大小的 Baddie。它选择一个随机的 x 坐标

和一个将图像放置在窗口上方的 y 坐标。它接着

创建一个图像对象并将图像缩放到所选大小 2\。

最后，它选择一个随机速度。

Baddie 管理器，我稍后会展示它的代码，调用

update() 方法 3 在每一帧中执行：这里的代码会移动位置

通过表示其速度的像素数将 Baddie 向下移动。如果

如果 Baddie 已经移动出了游戏区域底部，我们返回 True 以表示

当这个 Baddie 准备被移除时。否则，我们返回 False 以告诉

Baddie 管理器将这个 Baddie 留在窗口中。

draw() 方法 4 在新的位置绘制 Baddie。

collide() 方法 5 检查玩家与 Baddie 是否发生碰撞

相交。

**356** 第16章

BaddieMgr 类，如列表 16-9 所示，创建并管理一个 Baddie 对象列表；这是一个典型的对象管理器对象的例子。

# BaddieMgr 类

class BaddieMgr():

ADD_NEW_BADDIE_RATE = 8  # 添加新 Baddie 的频率

1 def __init__(self, window):

self.window = window

self.reset()

2 def reset(self): # 游戏开始时调用

self.baddiesList = []

self.nFramesTilNextBaddie = BaddieMgr.ADD_NEW_BADDIE_RATE

3 def update(self):

# 告诉每个 Baddie 更新自身

# 计算有多少 Baddie 掉出了窗口底部

nBaddiesRemoved = 0

4 baddiesListCopy = self.baddiesList.copy()

对于 baddiesListCopy 中的每个 oBaddie：

5 deleteMe = oBaddie.update()

如果 deleteMe:

self.baddiesList.remove(oBaddie)

nBaddiesRemoved = nBaddiesRemoved + 1

# 检查是否该添加一个新 Baddie

6 self.nFramesTilNextBaddie = self.nFramesTilNextBaddie - 1

如果 self.nFramesTilNextBaddie == 0:

oBaddie = Baddie(self.window)

self.baddiesList.append(oBaddie)

self.nFramesTilNextBaddie = BaddieMgr.ADD_NEW_BADDIE_RATE

# 返回已删除的 Baddie 数量

return nBaddiesRemoved

7 def draw(self):

对于 self.baddiesList 中的每个 oBaddie：

oBaddie.draw()

8 def hasPlayerHitBaddie(self, playerRect):

对于 self.baddiesList 中的每个 oBaddie：

如果 oBaddie 与 playerRect 碰撞：

return True

return False

*列表 16-9：BaddieMgr 类*

__init__() 方法调用了 BaddieMgr 类的重置方法（reset()）

设置 Baddie 对象列表为空列表。我们使用帧计数器

创建新 Baddie 的方法是比较频繁的，以保持游戏的趣味性。

我们使用实例变量 self.nFramesTilNextBaddie 来计数帧数。

完整游戏：Dodger **357**

reset() 方法 2 在开始新一轮游戏时调用。

它会清空 Baddie 列表，并重置帧计数器。

update() 方法 3 是 Baddie 管理的实际发生地

针对发生的情况，我们的目的是遍历所有 Baddie，并告知每个 Baddie

更新自身的位置，并移除掉任何掉出窗口底部的 Baddie

窗口。然而，这里有一个潜在的 bug。如果你仅仅是遍历

列出并删除符合删除条件的元素，该列表

当元素被删除后，列表会立即压缩。此时，紧跟着被删除元素的元素会

删除的元素将被跳过；在这个循环中，该元素会

不会被告知更新自身。虽然当时我没有深入讲解，但我们

我们在第 11 章的气球游戏中遇到过类似问题，当时我们需要去除那些漂浮到窗口顶部的气球。

在这里我使用了一个解决方案，使用 reversed() 函数对列表进行反转

列表，按反向顺序迭代（参见列表 11-6）。

在这里我实现了一个更加通用的解决方案 4。该方法

在 BaddieMgr 类中使用的方式是制作列表的副本并遍历

*复制* 列表；然后，如果我们找到符合删除条件的元素（例如一个掉出了窗口底部的 Baddie），我们

从*原始*列表中移除该元素（即特定的Baddie）。通过这种方法，我们遍历的是一个不同于删除元素的列表。

我们正在移除元素。

当我们遍历Baddies时，调用每个Baddie的update()方法，

每个Baddie的update()方法返回一个布尔值：False表示它仍在向下移动

窗口，或者True表示它已经掉出底部。我们统计

Baddies掉出屏幕底部的数量，并从

列表。在方法结束时，我们将计数返回给主代码，以便它

可以更新得分。

每帧我们还会检查是否到了创建新

Baddie 6。当我们遍历常量ADD_NEW_BADDIE_RATE指定的帧数时，

帧数达到一定数量时，我们创建一个新的Baddie对象并将其添加到Baddies列表中。

draw()方法7遍历Baddie列表并调用

draw()方法用于让每个Baddie在适当的位置绘制自己。

最后，hasPlayerHitBaddie()方法8会检查Player是否与

矩形与任何Baddie发生碰撞时，代码遍历Baddie列表。

Baddies并调用每个Baddie的collide()方法。如果发生了碰撞，

与任何Baddie发生碰撞（重叠）时，我们将碰撞信息报告回主代码，

这将结束游戏。

**文件：Dodger/Goodies.py**

GoodieMgr和Goodie类与BaddieMgr和Baddie类非常相似。

类。Goodie管理器是一个对象管理器对象，负责维护一个

Goodie列表的不同之处在于它会随机

随机地将一个Goodie放置在窗口的左边缘（在这种情况下它

向右移动）或者右边缘（因此它向左移动）。它还会创建新的Goodie

在经过随机的帧数后，当Player与

如果Player与Goodie碰撞，用户会得到25分奖励。Goodie的update()方法

Goodie管理器使用前面章节描述的技术：它

它会复制Goodies列表并遍历副本。

**358** 第16章

**文件：Dodger/Player.py**

Player类，如示例16-10所示，管理Player图标并跟踪

它应该出现在游戏窗口中的位置。

# Player类

--- 截取导入部分 ---

class Player():

1 def __init__(self, window):

self.window = window

self.image = pygwidgets.Image(window,

(-100, -100), 'images/player.png')

playerRect = self.image.getRect()

self.maxX = WINDOW_WIDTH - playerRect.width

self.maxY = GAME_HEIGHT - playerRect.height

# 每帧，将Player图标移动到鼠标位置

# 限制x和y坐标在游戏窗口的区域内

2 def update(self, x, y):

if x < 0:

x = 0

elif x > self.maxX:

x = self.maxX

if y < 0:

y = 0

elif y > self.maxY:

y = self.maxY

self.image.setLoc((x, y))

return self.image.getRect()

3 def draw(self):

self.image.draw()

*示例16-10：Player类*

__init__()方法1加载Player图标图像并设置

用于后续的多个实例变量。

update()方法2在每帧中由Play场景调用。

基本思路是将玩家图标显示在鼠标位置，这是通过传递来实现的

中。我们进行了一些检查，以确保图标保持在矩形范围内。

可玩区域的角度。在每一帧中，update() 方法返回

更新玩家图标的矩形区域，这样主游戏代码（在列表 16-6 中）就

可以检查玩家的矩形是否与任何坏人或好人相交。

最后，draw() 方法 3 会在新位置绘制玩家图标。

使用好人管理器、坏人管理器和玩家

该对象清楚地展示了面向对象编程（OOP）的力量。我们只需向

这些对象，要求它们更新或重置自己，它们会根据需要执行

每当它们需要做出响应时。好人和坏人管理器传递

将这些消息传递给它们所管理的所有好人和坏人。

完整游戏：Dodger **359**

**文件：Dodger/SceneHighScores.py**

高分场景显示前10名的高分（以及

玩家）在表格中。它还允许排名前10的用户

用户可以选择将他们的名字和分数输入到表格中。场景实例化

一个 HighScoresData 对象来管理实际数据，其中包括读取

和写入数据文件。这使得高分场景能够更新

表格并响应来自游戏场景的当前高分请求。

以及低分。

列表 16-11 至 16-13 包含了 SceneHighScores

类的代码。我们从 __init__() 和 getSceneKey() 方法开始

列表 16-11\。

# 高分场景

--- 截取导入部分，showCustomAnswersDialog 和 showCustomResetDialog ---

class SceneHighScores(pyghelpers.Scene):

def __init__(self, window):

self.window = window

1 self.oHighScoresData = HighScoresData()

self.backgroundImage = pygwidgets.Image(self.window,

(0, 0)，

'images/highScoresBackground.jpg')

self.namesField = pygwidgets.DisplayText(self.window,

(260, 84)，''，fontSize=48，

textColor=BLACK，

width=300, justified='left')

self.scoresField = pygwidgets.DisplayText(self.window，

(25, 84)，''，fontSize=48，

textColor=BLACK,

width=175, justified='right')

self.quitButton = pygwidgets.CustomButton(self.window，

(30, 650)，

up='images/quitNormal.png',

down='images/quitDown.png',

over='images/quitOver.png',

disabled='images/quitDisabled.png')

self.backButton = pygwidgets.CustomButton(self.window,

(240, 650)，

up='images/backNormal.png',

down='images/backDown.png',

over='images/backOver.png',

disabled='images/backDisabled.png')

self.resetScoresButton = pygwidgets.CustomButton(self.window，

(450, 650)，

up='images/resetNormal.png',

down='images/resetDown.png',

over='images/resetOver.png',

**第 360 章**

disabled='images/resetDisabled.png') 2 self.showHighScores()

3 def getSceneKey(self):

return SCENE_HIGH_SCORES

*列表 16-11：SceneHighScores 类的 __init__() 和 getSceneKey() 方法* __init__() 方法 1 创建一个 HighScoresData 类的实例，

该数据维护着高分场景的所有数据。然后我们创建所有

场景的图片、字段和按钮。在初始化结束时—

在我们调用 `self.showHighScores()` 时，用 2 来填充姓名和分数字段。

getSceneKey() 方法 3 返回该场景的唯一标识符。

所有场景中都必须实现此方法。

Listing 16-12 显示 SceneHighScores 类的 enter() 方法的代码。

SceneHighScores 类。

1 def enter(self, newHighScoreValue=None):

# 这个方法可以有两种不同的调用方式：

# 1\. 如果没有新的最高分，newHighScoreValue 将为 None。

# 2\. newHighScoreValue 是当前游戏的分数，位于前 10 名内。

2 如果 newHighScoreValue 为 None：

return # 没有操作

3 self.draw() # 在显示对话框前绘制

# 我们从游戏场景收到了新的最高分。

dialogQuestion = ('录入你得分的 ' +

str(newHighScoreValue) + ',\n' +

'请输入你的名字：'

4 playerName = showCustomAnswerDialog(self.window,

dialogQuestion)

5 如果 playerName 为 None：

return # 用户按下了取消

# 将用户和分数添加到最高分列表。

如果 playerName == ''：

playerName = '匿名'

6 self.oHighScoresData.addHighScore(playerName,

newHighScoreValue)

# 显示更新后的最高分表格。

self.showHighScores()

*Listing 16-12：SceneHighScores 类的 enter() 方法*

场景管理器调用 High Scores 场景的 enter() 方法 1

当从游戏场景导航到此场景时。如果用户刚刚完成的游戏...

如果用户完成的游戏没有进入前 10 名，这个方法将直接返回 2。否则，

如果用户确实获得了前 10 名的分数，enter() 方法会额外带有一个参数调用。

值—用户刚刚完成的游戏的得分。

在这种情况下，我们调用 draw() 3 来绘制 High Scores 场景的内容。

在显示对话框并让用户选择是否将其分数添加到列表之前的场景。

完整游戏：Dodger **361**

![Image 53](index-391_1.png)

将分数添加到列表中。然后我们调用一个中间函数，showCustom。

AnswerDialog()，它构建并显示自定义对话框 4，如图所示。

如图 16-7 所示。

*图 16-7：一个自定义的 AnswerDialog，让用户将他们的名字添加到最高分列表中* 如果用户选择 "No Thanks"，我们得到的返回值为 None，之后继续执行。

我们跳过此方法的其余部分 5\. 否则，我们获取用户输入的名字。

返回并通过调用方法将姓名和分数添加到表格 6 中。

在 HighScoresData 对象中。最后，通过调用该方法来更新字段。

showHighScores() 方法。如果该方法调用中没有分数 2，

没有操作，因为当前列表已经显示。

Listing 16-13 包含此类其余方法的代码。

def showHighScores(self): 1

# 获取分数和姓名，并在两个字段中显示它们。

scoresList, namesList = self.oHighScoresData.getScoresAndNames()

self.namesField.setValue(namesList)

self.scoresField.setValue(scoresList)

def handleInputs(self, eventsList, keyPressedList): 2

对于事件列表中的每个事件：

if self.quitButton.handleEvent(event):

self.quit()

elif self.backButton.handleEvent(event):

self.goToScene(SCENE_PLAY)

elif self.resetScoresButton.handleEvent(event):

confirmed = showCustomResetDialog(self.window, 3

'你确定要\n重置高分吗？')

如果确认：

self.oHighScoresData.resetScores()

self.showHighScores()

def draw(self): 4

self.backgroundImage.draw()

self.scoresField.draw()

self.namesField.draw()

self.quitButton.draw()

self.resetScoresButton.draw()

self.backButton.draw()

**362** 第16章

def respond(self, requestID): 5

如果 requestID == HIGH_SCORES_DATA:

# 来自 Play 场景的最高和最低分数请求

# 构建一个字典并返回给 Play 场景

highestScore, lowestScore = self.oHighScoresData.getHighestAndLowest()

return {'highest':highestScore, 'lowest':lowestScore}

*第16-13号：SceneHighScores 类中的 showHighScores()、handleInputs()、draw() 和 respond() 方法*

showHighScores() 方法 1 首先通过询问 HighScoresData

对象包含两个列表：前10名的姓名和分数。它接收这些列表，并

返回并将它们发送到两个显示字段进行展示。如果你传递一个列表

给 DisplayText 对象的 setValue() 方法，它会显示每个元素

在单独的行上。我们使用两个 DisplayText 对象，因为 self.namesField 是

左对齐，而 self.scoresField 是右对齐的。

handleInputs() 方法 2 只需要检查并响应

用户点击退出、返回和重置分数按钮时。因为

重置分数按钮会清除数据，因此我们应该在操作前要求确认—

执行此操作。因此，当用户点击此按钮时，我们会调用一个

中间函数，showCustomResetDialog() 3，用于弹出一个对话框，询问

提示用户确认他们是否真的想清除所有当前分数。

draw() 方法 4 绘制窗口中的所有元素。

最后，respond() 方法 5 允许另一个场景向此场景请求

信息。这使得 Play 场景能够请求当前的最高

分数和第十高分数——合格玩家的最低分数—

高分列表。调用者发送一个值，指示请求哪种信息

它正在寻找的内容。在这种情况下，请求的信息是 HIGH_SCORES_DATA，这是一个常量

这些信息来自 *Constants.py* 文件。这个方法建立了一个字典，包含请求的两个值，并将其返回给调用场景。

**文件：Dodger/HighScoresData.py**

最终类是 HighScoresData，负责管理高分信息。

它以 JSON 格式读取和写入数据。数据

始终保持顺序，从最高分到最低分。例如，

代表前10名高分的文件可能是这样的：

[['Moe', 987], ['Larry', 812], ... ['Curly', 597]]

第16-14号列展示了 HighScoresData 类的代码。

# HighScoresData 类

from Constants import *

来自 pathlib 的 Path

import json

class HighScoresData():

"""数据文件以 JSON 格式存储为一个列表的列表。

每个列表由一个姓名和一个分数组成：

[[name, score], [name, score], [name, score] ...]

在此类中，所有分数都保存在 self.scoresList 中。

完整游戏：Dodger **363**

列表按分数从高到低排序。

"""

1 def __init__(self):

self.BLANK_SCORES_LIST = N_HIGH_SCORES * [['-----', 0]]

2 self.oFilePath = Path('HighScores.json')

# 尝试打开并从数据文件中加载数据

try:

3 data = self.oFilePath.read_text()

except FileNotFoundError: # 文件未找到，设置为空白分数并保存

4 self.scoresList = self.BLANK_SCORES_LIST.copy()

self.saveScores()

return

# 文件存在，从 JSON 文件中加载分数

5 self.scoresList = json.loads(data)

6 def addHighScore(self, name, newHighScore):

# 找到适当的位置添加新的高分

placeFound = False

for index, nameScoreList in enumerate(self.scoresList):

thisScore = nameScoreList[1]

if newHighScore > thisScore:

# 插入到正确的位置，移除最后一项

self.scoresList.insert(index, [name, newHighScore])

self.scoresList.pop(N_HIGH_SCORES)

placeFound = True

break

if not placeFound:

return # 分数不应添加到列表中

# 保存更新后的分数

self.saveScores()

7 def saveScores(self):

scoresAsJson = json.dumps(self.scoresList)

self.oFilePath.write_text(scoresAsJson)

8 def resetScores(self):

self.scoresList = self.BLANK_SCORES_LIST.copy()

self.saveScores()

9 def getScoresAndNames(self):

namesList = []

scoresList = []

for nameAndScore in self.scoresList:

thisName = nameAndScore[0]

thisScore = nameAndScore[1]

namesList.append(thisName)

scoresList.append(thisScore)

return scoresList, namesList

a def getHighestAndLowest(self):

# 元素 0 是最高项，元素 -1 是最低项

**364** 第16章

highestEntry = self.scoresList[0]

lowestEntry = self.scoresList[-1]

# 获取每个子列表的分数（元素 1）

highestScore = highestEntry[1]

lowestScore = lowestEntry[1]

return highestScore, lowestScore

*Listing 16-14: The HighScoresData class*

在 __init__() 方法 1 中，我们首先创建一个所有空白项的列表。

使用 Path 模块创建一个路径对象，指向数据的存放位置

文件 2\.

**注释**

*此示例中显示的路径与代码位于同一文件夹。这对于学习文件输入输出的概念是可以的。然而，如果你打算将程序分享给其他人，在他们的电脑上运行，最好使用用户主目录中的其他路径。可以像这样构建此路径：*

import os.path

DATA_FILE_PATH = os.path.expanduser('~/DodgerHighScores.json')

*或者：*

from pathlib import Path

DATA_FILE_PATH = Path('~/DodgerHighScores.json').expanduser()

接下来，我们通过检查文件是否存在来检查是否已经保存了高分

检查数据文件是否存在 3。如果文件未找到 4，则设置

将分数添加到空白项的列表中，调用 saveScores() 保存分数，

返回。如果未找到，则读取文件内容 5 并进行转换

将 JSON 格式转换为列表列表。

addHighScores() 方法 6 负责添加新的高分

分数添加到列表中。由于数据始终保持有序，我们会遍历

直到找到合适的索引并插入新

姓名和分数。由于该操作会扩展列表，我们会移除

最后一个元素仅保留前10名。我们还检查新分数

应该被插入到列表中。最后，我们调用saveScores()保存

将分数保存到数据文件。

saveScores()方法7将分数数据保存为JSON格式的

文件。它从各个地方调用。

调用resetScores()方法8，当用户表示他们希望

重置所有姓名和分数为起始点（所有姓名为空白，分数为零）。

所有分数设置为零）。我们调用saveScores()来重写数据文件。

getScoresAndNames()方法9由高分场景调用

获取前10名的分数和姓名。我们遍历高分榜的列表列表。

分数数据创建一个分数列表和一个姓名列表；这两个列表都是

返回。

最后，getHighestAndLowest()方法a由高分场景调用

场景以获取表中的最高分和最低分。它使用这些结果

判断用户的分数是否符合条件以输入姓名和

将分数记录到高分榜中。

完整游戏：Dodger **365**

***游戏扩展***

整体架构是模块化的，便于修改。每个

场景封装自己的数据和方法，同时通信和

导航由场景管理器处理。扩展可以由

在一个场景中进行，而不会影响其他场景。

例如，而不是游戏在玩家图标

当击中一个反派时，你可能希望玩家从一定数量的生命开始；

当玩家图标击中反派时，生命数会减少

一旦玩家生命耗尽，游戏结束。这种

这样的更改相对容易实现，只会影响

游戏场景。

作为另一个想法，玩家可能从少量炸弹开始

以便玩家可以在困境中引爆炸弹，消灭所有反派

在玩家图标周围的指定半径范围内。炸弹的数量会

每次使用时都减少，直到减少到零。这个更改

只会影响游戏场景和反派管理器的代码。

或者你可能希望追踪更多的高分——比如20名，而不是

10名的一个更改可以在高分场景中进行，而不会

影响游戏或开场画面。

**总结**

本章演示了如何创建和使用是/否和答案

对话框—包括文本和可定制版本。然后我们专注于构建

一个完整的面向对象游戏程序，Dodger。

我们使用pygwidgets模块处理所有按钮、文本显示和

输入文本框。我们使用了pyghelpers模块来处理所有对话框。该

SceneMgr使我们能够将游戏拆分为更小、更易管理

片段（场景对象）并在场景之间导航。

游戏使用或展示了以下面向对象的

概念：

**封装** 每个场景仅处理特定的事物

场景。

**多态** 每个场景实现相同的方法。

**继承** 每个场景都继承自Scene基类。

**对象管理器对象** 游戏场景使用组合来创建一个

Baddie管理对象self.oBaddieMgr和Goodie管理对象，

self.oGoodieMgr，每个管理其对象列表。

**共享常量** 我们为Goodies和Baddies使用独立模块，

一个*Constants.py*文件使我们可以轻松地在各模块之间共享常量。

**366** 第16章

**17**

**设计模式**

**总结**

在本章的最后，我将介绍

面向对象编程的概念

*设计模式*，它们是可重用的面向对象编程解决方案

形式和常见软件问题-

问题。我们已经在本章中看到一个设计模式

书籍：使用对象管理器对象来管理一个列表

或对象字典。许多完整的书籍都围绕这个主题写过

设计模式，因此显然我们不能讨论所有的模式。在本章中

我们将重点介绍用于分解系统的模型-视图-控制器模式-

将系统分解成更小、更易管理和修改的部分。最后，我将

给出面向对象编程的总结。

**模型-视图-控制器**

*模型-视图-控制器 (MVC)* 设计模式强制明确分离

数据集合和数据展示方式之间的关系

用户。该模式将功能分为三部分：模型、

![图 54](index-397_1.png)

![图 55](index-397_2.png)

视图和控制器。每个部分都有明确的责任，

每个部分由一个或多个对象实现。

模型存储数据。视图负责绘制信息-

以一种或多种方式从模型中获取信息的控制器

通常创建模型和视图对象，处理所有用户交互，

通知模型更改，并告诉视图显示

数据。这样的分离使得整个系统高度可维护和

可修改的。

***文件显示示例***

作为MVC模式的一个好例子，考虑文件显示的方式

在macOS Finder或Windows文件资源管理器中。例如，我们有一个包含四个文件和一个子文件夹的文件夹。最终用户可以选择以

包含四个文件和一个子文件夹。最终用户可以选择以图标形式显示这些

项目以列表形式显示，如图 17-1所示\。

*图 17-1：文件夹中的文件以列表形式显示*

或者，用户可以选择将相同的项目以图标的形式显示，如

如图 17-2所示\。

*图 17-2：文件夹中的文件以图标形式显示*

两种显示的底层数据是相同的，但表现-

信息展示方式的不同。在这个例子中，数据是

文件和子文件夹的列表，它保存在一个模型对象中。视图对象

以用户选择的任何方式显示数据：列表、图标、或详细列表等。

详细列表，等等。控制器告诉视图以某种方式显示来自模型的信息，

用户选择的布局方式下的呈现

***统计显示示例***

作为MVC模式的更广泛示例，我们考虑一个程序

该程序模拟多次掷骰并显示结果。在

**368** 第17章

![Image 56](index-398_1.png)

每次掷骰时，我们将两个骰子的值相加，因此总和—我们称之为

一个*结果*—必须在2到12之间。数据包括掷骰2,500次的计数。

每个结果出现的次数和输入框中指定的掷骰次数的百分比。

显示每个结果在总掷骰次数中所占的百分比。程序可以显示

该数据有三种不同的表示形式：条形图、饼图和

文本表格。默认情况下显示为条形图，并在模拟后展示结果

由于该程序只是为了展示

演示MVC模式的工作示例，我们将生成输出

使用pygame和pygwidgets。对于更专业的图表和

如果您感兴趣，我建议您了解Python的数据可视化库

例如Matplotlib、Seaborn、Plotly、Bokeh等，都是为此类任务设计的

目的而言。

图17-3显示了条形图形式的数据。

*图17-3：掷骰数据（条形图形式）*

每个条形图下方是结果，显示该结果出现的次数

结果被掷出的次数，以及该结果占总掷骰次数的百分比

计数表示每个条形图的高度对应于计数（或百分比）。

百分比）。点击“掷骰”按钮再次运行模拟，使用掷骰次数

用户可以点击不同的单选按钮

按钮展示相同数据的不同视图。如果用户选择饼图

如果用户选择饼图单选按钮，数据将如图17-4所示。

设计模式与总结 **369**

![Image 57](index-399_1.png)

![Image 58](index-399_2.png)

*图17-4：掷骰数据（饼图形式）*

如果用户选择文本单选按钮，数据将以文本方式显示，如下所示

图17-5\。

*图17-5：掷骰数据（文本形式）*

**370** 第17章

用户可以更改“掷骰次数”字段中的值，按需掷骰多次。此程序中的数据基于统计数据

和随机性。随着样本大小的不同，具体的计数显然会有所变化

结果可能会有所不同，但百分比应该始终大致相同。

我不会在这里展示完整的程序清单，而是聚焦于几个

关键代码行展示了MVC模式中的设置和控制流程

模式。完整的程序可与书籍的其他资源一起下载。

本书的资源位于*MVC_RollTheDice*文件夹中。该文件夹包含

以下文件：

***Main_MVC.py***主Python文件

***Controller.py***包含Controller类

***Model.py***包含Model类

***BarView.py***包含显示条形图的BarView类

***Bin.py***包含绘制条形图单个条形的Bin类

***PieView.py***包含显示饼图的PieView类

***TextView.py***包含显示文本视图的TextView类

***Constants.py*** 包含多个模块共享的常量 主程序实例化一个控制器对象并运行主程序

循环。主循环中的代码将所有事件（除了pygame.QUIT事件）

事件）传递给控制器进行处理。

**控制器**

控制器是整个程序的主管。它首先通过实例化

初始化模型对象。然后实例化不同类型的视图对象

视图对象：BarView、PieView 和 TextView。以下是启动代码

控制器类的__init__()方法：

# 实例化模型

self.oModel = Model()

# 实例化不同的视图对象

self.oBarView = BarView(self.window, self.oModel)

self.oPieView = PieView(self.window, self.oModel)

self.oTextView = TextView(self.window, self.oModel)

当控制器对象实例化这些视图对象时，它传入

模型对象，这样每个视图对象就可以直接从

模型。MVC模式的不同实现可能会以不同的方式处理

这三者之间的通信方式可能不同；例如，一个

控制器可以充当中介，向模型请求数据

并将其转发到当前视图，而不是允许模型

和视图直接通信。

控制器绘制并响应黑色矩形外部的所有内容—

窗口中的所有内容，包括标题、骰子图像和单选按钮

按钮。它绘制退出和掷骰按钮，并在它们被点击时做出反应

被点击时，它会处理用户对掷骰次数的任何更改。

设计模式与总结 **371**

控制器对象保持一个当前的视图对象，决定当前显示的是哪个视图。我们默认将其设置为BarView对象

（条形图）：

self.oView = self.oBarView

当用户点击单选按钮时，控制器设置其当前视图

将对象传递给新选择的视图，并告知新的视图对象更新

通过调用其update()方法来更新自身：

if self.oBarButton.handleEvent(event):

self.oView = self.oBarView

self.oView.update()

elif self.oPieButton.handleEvent(event):

self.oView = self.oPieView

self.oView.update()

elif self.oTextButton.handleEvent(event):

self.oView = self.oTextView

self.oView.update()

启动时，以及每次用户点击掷骰时，控制器会验证—

更新“掷骰次数”字段中指定的掷骰次数，并告知

模型生成新数据：

self.oModel.generateRolls(nRounds)

所有视图都是多态的，因此在每一帧中，控制器对象

调用当前视图对象的draw()方法：

self.oView.draw() # 告诉当前视图绘制自身

**模型**

模型负责获取（并可能更新）信息—

在本程序中，模型对象很简单：它模拟掷两个骰子的过程

多次掷骰子，将结果存储在实例变量中，并报告

当视图对象请求数据时，控制器提供数据。

当请求生成数据时，模型运行一个循环模拟掷骰子

骰子并将其数据存储在两个字典中：self.rollsDict，它使用

每个结果作为一个键，计数作为一个值，以及 self.percentsDict，它

使用每个结果作为键，投掷的百分比作为值。

在更复杂的程序中，模型可以从数据源获取其数据，

基础，互联网或其他来源。例如，一个模型对象可能包含

维护股票信息、人口数据、城市住房数据、温度

读取等。

在这个模型中，getRoundsRollsPercents() 方法是由

视图对象一次性获取所有数据。然而，模型可能包含

比任何一个视图所需的更多信息。因此，不同的视图

对象可以调用模型对象中的不同方法来请求不同的

从同一个模型中获取信息。为了支持这一点，在示例程序中

**第 372 章**

我已包括一些额外的 getter 方法（getNumberOfRounds(), getRolls(), 和 getPercents()），程序员在构建时可以使用它们

新的视图对象可以获取只有新视图可能想展示的数据。

**视图**

视图对象负责将数据展示给用户。在我们的示例

在我们的程序中，有三个不同的视图对象展示相同的

以三种不同的形式呈现信息；每个显示信息的方式

在窗口中的黑色矩形框内。启动时，当用户

点击“Roll Dice”时，控制器调用当前视图的 update() 方法

对象。所有视图对象随后都向模型对象发出相同的请求来获取

当前数据：

nRounds, resultsDict, percentsDict = self.oModel.getRoundsRollsPercents()

然后，视图对象以其自己的方式格式化数据并展示给

用户。

***MVC 模式的优点***

MVC 设计模式将责任分配到不同的类中，

独立操作但共同协作。通过将组件构建为不同的

不同类之间的交互最小化，从而避免结果对象的交互

使每个单独的组件变得不那么复杂，且更少出错。

一旦定义了每个组件的接口，类的代码

甚至可以由不同的程序员编写。

使用 MVC 方法，每个组件展示了核心的面向对象编程

封装和抽象的概念。使用 MVC 对象结构，

模型可以改变它在内部表示数据的方式，而不会

不影响控制器或视图。如前所述，模型

可能包含比任何单一视图所需更多的数据。只要

控制器不改变它与模型的通信方式，并且

模型继续向视图返回请求的信息， 

一致的方式，模型可以添加新数据而不会破坏系统。

MVC 模型还使得添加增强功能变得更加容易。例如，

在我们的骰子投掷程序中，模型可以跟踪

两颗骰子的不同组合形成每个结果，

比如通过掷出 1 和 4 或 2 和 3 获得 5。然后我们可以修改

使得 BarChart 视图能够从模型获取这些附加信息。

并展示每个条形图分割成更小的条形图，显示每个部分的百分比。

组合。

每个视图对象都是完全可定制的。TextView 可以使用

不同的字体和字体大小，或不同的布局。PieView 可以显示

不同颜色的楔形。BarView 中的条形图可以更粗或

更高、更宽，或者以不同的颜色显示，甚至可以水平显示。

任何此类变化都只会在适当的视图对象中进行修改，com-

完全独立于模型或控制器。

设计模式与总结 **373**

MVC 模式还使得添加新的数据视图变得容易，只需编写一个新的视图类。唯一需要的额外更改是

为了让控制器绘制另一个单选按钮，实例化新的视图

对象，并在用户选择时调用新视图对象的 update() 方法。

新视图。

**注**

*MVC 和其他设计模式与任何特定的编程语言无关*，*并且可以在任何支持 OOP 的语言中使用。如果你有兴趣深入了解*，*我建议你在网上搜索 OOP 设计模式，如工厂模式、享元模式、观察者模式和访问者模式；这些模式有许多视频和文字教程（以及书籍）可供参考。关于一般介绍，*《设计模式：可复用面向对象软件的元素》*(Addison-Wesley) 作者是*

*埃里希·伽玛、理查德·赫尔姆、拉尔夫·约翰逊和约翰·弗利西德斯（四人帮）* *被认为是设计模式的经典之作。*

**总结**

在思考面向对象编程时，记住我最初的

对象的定义：数据加上作用于这些数据的代码。

OOP 为你提供了一种全新的编程思维方式，提供了一种简单

一种便捷的方式将数据和作用于这些数据的代码组合在一起。

你编写类并从这些类实例化对象。每个对象都获得

一个类中定义的所有实例变量的集合，但实例变量

不同的对象可以包含不同的数据，并且保持彼此独立

彼此。对象的方法可以因为它们

在不同数据上进行工作。对象可以随时实例化，并且可以

随时被销毁。

当从一个类实例化多个对象时，通常会构建

一个对象列表或字典，然后可以稍后遍历该列表或字典，

调用每个对象的方法。

最后提醒，OOP 的三个主要原则是：

**封装** 一切都集中在一个地方，对象拥有它们的数据。

**多态** 不同的对象可以实现相同的方法。

**继承** 一个类可以扩展或修改另一个类的行为。

对象通常在层次结构中工作；它们可以使用组合来实例化

创建其他对象，并可以调用低级别对象的方法，要求它们

执行工作或提供信息。

为了给你一个清晰的面向对象编程实际操作的可视化表示，大部分

本书中的示例集中在小部件和其他可以

在游戏环境中有用。我开发了 pygwidgets 和 pyghelpers

使用包来展示许多不同的 OOP 技巧，并允许你

在 pygame 程序中轻松使用 GUI 小部件。我希望你发现这些包

年龄有用，并且继续使用它们开发有趣或有用的程序

你自己的。

**374** 第17章

更重要的是，我希望你能认识到面向对象编程是一种通用方法，可以应用于广泛的领域

在各种情况下。每当你看到两个或更多的函数需要

操作共享数据集的对象上，你应该考虑构建一个类并

实例化对象。你也可以考虑构建一个对象

管理员对象管理一组对象。

话虽如此，我想向你表示祝贺：你做到了

直到书的最后！不过，实际上，这应该被认为是

面向对象编程旅程的开始。希望，

本书中描述的概念为你提供了一个框架，你

可以在此基础上进行构建——但唯一真正理解 OOP 如何工作

的方法是写大量的代码。随着时间的推移，你会开始注意到在你的代码中

你将一次又一次地使用的代码。理解如何构建

你的类设计是一个困难的过程，只有通过经验才能开始

为了更容易确保你有正确的方法和实例

在正确的类中使用变量。

练习，练习，再练习！

设计模式和总结 **375**

**索引**

**符号**

动画类，149，304–306

+ 操作符示例，多态，

动画类

193–194

建设，296–304

*（星号），与导入一起使用

合并，304

语句，61

在 pygwidgets 中，304–309

@（at）属性，174–177

SimpleAnimation 类，296–300

==（等于），魔法方法

SimpleSpriteSheetAnimation 类，

名称，196

300–304

> （大于），魔法方法

动画程序，pygwidgets

名称，196

包，308–309

>=（大于或等于），魔法

答复对话框，Dodger 游戏，345–347

方法名称，196

抗锯齿线条，绘制，118

<（小于），魔法方法

API（应用程序编程

名称，196

接口），137，158

<=（小于或等于），魔法

弧线，绘制，118

方法名称，196

参数

!=（不等于），魔法方法

方法和参数，144

名称，196

传递给方法，40–41

在调用方法时重新排列，

**A**

53

abc（抽象基类）模块，

星号（*），与导入一起使用

Python 标准库，232

语句，61

AbortTransaction 异常，79，81

at (@) 属性，174–177

绝对路径，在 pygame 中使用，101

抽象基类（abc）模块，

**B**

Python 标准库，232

背景音乐，在 pygame 中播放，

抽象类和方法，231–234。

115–116

*另见* 类；方法

坏人和好人，347

抽象，179–181

Ball 类，122–125

账户类

球对象，创建，125–127

错误处理，在，59

气球游戏

使用异常，78–79

主代码，252–256

测试，62

常量模块，253

账户对象，59，71

对象图，252

字典，66–67

项目文件夹，252

在列表中，64–66

截图，251

账户，创建，62–64

源文件，252

警报对话框，Dodger 游戏，343

气球管理器，256–258

尖括号（<>），其中的值，16

气球示例程序，251–261

气球类和对象，

内部与外部，164–165

258–261

使其可用，29–30

银行账户类，58–60

和方法，51

银行账户模拟。*见* 

和对象，23–25

过程实现

对象和实例化，25–33

操作和数据，7–8

将物理对象表示为，

数据表，15–16

35–44

银行类，70，79–81

在使用中，45

银行对象，70–71，82

编写，26–27

银行程序，使用异常，

类层次结构，继承，236–238

78–83

类作用域，27

基类，继承，212–214，

类声明，继承，216

227–231

类变量。*见* 变量

扑克牌组，278

常量，249–250

blit() 方法，102，114，124，137

用于计数，250

布尔值 True，105

创建，248–249

按钮类，构建，128–130

客户端代码

按钮，构建，131–132

解释，164

使用直接访问，170

**C**

collidepoint()，104

回调函数，137–141

比较运算符魔法方法，

CanastaDeck 类，279

195–196

卡片类，268–270

*组合*，71

扑克牌游戏。*见* 高低游戏

*组合* 和继承，238

扑克牌游戏

控制器对象，371–372

扑克牌，278

CountDownTimer 类，293–294

卡片类，268–270

计数对象，250

Deck 类，270–272

CountUpTimer 类，pyghelpers 包，

高低游戏，272–276

291–293

使用 __name__ 测试，276–278

*CPython*，242

使用不寻常的牌组，279

customAnswerDialog 对话框，Dodger

笛卡尔坐标系，

游戏，347

91–95，201

CustomButton 类，148，235–236

捕获异常，76

CustomCheckBox 类，pygwidgets

子类，继承，212

包，149

圆形，绘制，119

CustomRadioButton 类，pygwidgets

圆形类，187–190，227，230

包，149

类代码，导入，60–61

customYesNoDialog 对话框，Dodger

类。*见* 抽象类和

游戏，345

方法

构建，33–35

**D**

从中创建实例，31–32

数据，验证，168–170

从中创建对象，28–30

数据库，使用对象访问和

形式，26

XTRAS，178–179

实现数据类型为，32–33

数据类型，作为类的实现，

和继承，212–213

32–33

从相同的基类继承，

调试方法，203

227–231

Deck 类，270–272

**378** 索引

装饰器和 @ 属性，

使用函数，164

174–177

的解释，165–172

__del__() 方法，246–248，260

使实例变量更加

设计模式，MVC（模型-视图

私有，172–173

控制器)，367–374

使用对象，164–165

骰子投掷数据，MVC（模型-视图

在 pygwidgets 类中，177

控制器）设计模式，

使用 enter() 方法与场景，325

369–370

__eq__() 魔法方法名，196，

__dict__ 字典，261–263

198–199

字典

等于（==），魔法方法名，196

账户对象，66–67

错误处理

使用实例变量，

在 Account 类中，59

261–263

带异常，76–78

DimmerSwitch 类，33，48–50，52–53

事件驱动程序，95–96

创建 DimmerSwitch 对象，50，53

事件循环，99

避免直接访问，166–170

except 和 try，76–77

来自 Macromedia 项目的 Director，178

异常

DisplayMoney 类，222–227

在银行程序中，78–83

DisplayText 类，149，222

错误处理，76–78

Dodger 游戏

处理，81–83

警告对话框，343

答案对话框，345–347

**F**

自定义 Answer 对话框，347

文件显示示例，368

自定义 YesNo 对话框，345

Fraction 类，魔法方法，

扩展，366

205–208

实现，348

函数

模态对话框，342–347

使用封装，164

概述，347

len()，164

ScenePlay 类，351–355

与方法对比，28

textYesNo 对话框，343

super()，216

是/否对话框，344

vars()，52

不重复自己（DRY），253

Dragger 类，pygwidgets 包，149

**G**

绘制形状，116–120

Game 类，274

与 pygwidgets 一起使用的 draw() 方法，

Game 对象，272

150，155，157，193，308

游戏中的瞬态对象，242

与场景一起使用的 draw() 方法，325

垃圾回收，248

draw.rect()，187

__ge__() 魔法方法名，196

DRY（不要重复自己），253

getrefcount() 函数，244

与 getSceneKey() 方法一起使用

**E**

场景，328

教育项目，178–179

获取器和设置器，170–171，175–176

椭圆，绘制，119

*Ghostbusters*，138

Ellipse 类，192

访问 GitHub 仓库，157

Employee 类，继承，218

全局作用域，27

封装

好人和坏人，347

装饰器和 @property，174–177

与 goToScene() 方法一起使用

直接访问，166–172

场景，326

索引 **379**

图形文件格式，使用时

员工和经理示例，

pygame，100–101

214–218

大于（>），魔法方法

示例用法，224–227

名称，196

实现，213–214

大于或等于（>=），魔法

InputNumber 类，219–222，

方法名，196

224–227

__gt__() 魔法方法名，196，198

“是一个”关系，213

GUI 程序，事件驱动模型，

德梅特法则，238

95–96

在面向对象编程中，

212–213

**H**

和 pygwidgets，234–236

与 handleEvent() 方法一起使用

真实世界示例，219–227

pygwidgets，150，192，221，

来自同一基类，227–231

307，312

子类，212

与 handleInputs() 方法一起使用

测试代码，217–218

场景，363

pygwidgets 的使用，234–236

help() 函数，152

初始化参数，43–44

高低牌游戏，268\.

__init__() 方法，28，37，43，216

*参见* 纸牌游戏

Account 类，59，79

Game 对象，274–276

Ball 类，123

实现，4–7

Bank 类，73

主程序，272–274

继承示例，228–229，

表示数据，4

232–233

可重用代码，7

InputNumber，221

HighScoresData 类，363

发音，194

属性装饰器，175–176

**I**

SceneMgr 类，335

IDLE 开发环境，

继承中的子类，216

90，100–101

使用，27

Image 类，pygwidgets 包，

input() 函数，133，155

149。*参见* 子图像

InputNumber 类，219–222，224–227

ImageCollection 类，pygwidgets

InputText 类，149，219，222

包，149，157

安装

实现与接口，

pygame，90–91

84–85，137

pyghelpers，287

导入类代码，60–61

pygwidgets 包，149–150

导入语句，98

实例和作用域变量，27–28

继承。*参见* 多重

实例，26，31–32，41–43。*参见*

继承

多个实例

抽象类和方法，

实例作用域，27

231–234

实例变量。*参见* 插槽

基类，212

转换为计算，

类层次结构，236–238

167–168

客户端对子类的视图，218–219

更改名称，166–167

与组合，238

使用，27，165

编程难度，

实例化，解释，26

238–239

实例化过程，25–33

DisplayMoney 类，222–227

构建交互菜单，68–70

**380** 索引

接口与实现，84–85，137

与类，51

*Invent Your Own Computer Games with*

与函数，28

*Python*，341

向其传递参数，40–41

“是一个”关系，继承，213

模态对话框，Dodger 游戏，342–347

isInstance() 函数，196

Model 对象，371–372

items() 方法，271

常量模块，气球游戏，253

鼠标点击，在 pygame 中检测，

**J**

102–104

JSON 格式，363–365

MOUSEDOWN 事件，257

多重继承，239。*参见*

**K**

继承

关键字参数，pygame，145–146

多个实例，41–43。*参见*

实例

**L**

在 pygame 中播放音乐，115–116

德梅特法则，继承，238

MVC（模型-视图-控制器）设计

__le__() 魔法方法名称，196

模式

leave() 方法与场景一起使用，325

优势，373–374

len() 函数，15，164

控制器对象，371–372

小于 (<)，魔法方法名称，196

掷骰子数据，369–370

小于或等于 (<=)，魔法

文件显示示例，368

方法名称，196

Model 对象，372–373

LIFO（后进先出）顺序，179

概述，367–368

LightSwitch 类及测试代码，30

统计显示示例，368–371

灯光开关示例，22–23，25–31

View 对象，373

实例化 LightSwitch 对象，29

行，绘图，119

**N**

Lingo 语言，178

__name__，使用测试卡片游戏，

局部作用域，27

276–278

__lt__() 魔法方法名称，196，

命名约定，26

198，200

__ne__() 魔法方法名称，196

不等于（!=），魔法方法

**M**

名称，196

Macromedia 项目，178

魔法方法，194–201。*参见*

**O**

方法

对象组合，71

Manager 类，继承，219

对象生命周期

内存管理，插槽，261–263

级联删除，246–248

对象使用的内存。*参见* 对象

死亡通知，246–248

气球示例程序，251–261

垃圾回收，246–248

类变量，248–250

引用计数，242–246

使用插槽管理，261–263

事务对象，242

心智模型，49–52

瞬态对象，242

菜单，制作交互式，68–70

对象管理器对象，创建，70–76  

方法。*另见* 抽象类和  

面向对象编程（OOP）  

方法；魔法方法  

解释，3  

调用，30，41  

作为解决方案，45  

调用对象，30–31  

原则，374  

调用对象列表，83–84  

总结，374–375  

索引 **381**  

面向对象的 pygame。  

**P**  

*另见* pygame  

父类，继承，212  

Ball 类，122–125  

路径，使用 pygame，100  

Ball 对象，125–127  

路径名，使用 pygame，100  

回调，137–141  

模式，扩展与  

演示球与 SimpleText 和  

多态，192  

SimpleButton，135–137  

物理对象。*另见* 对象  

接口与实现，137  

构建软件模型，  

带按钮的程序，131–132  

22–23  

可重用的面向对象按钮，  

类和对象，23–25，45  

127–132  

类、对象与实例化，  

可重用的面向对象文本  

25–33  

显示，133–135  

复杂类，33–44  

SimpleButton，130–131  

OOP 作为解决方案，45  

SimpleText 类，133–135  

作为类表示，35–44  

显示文本的步骤，133  

PIE（多态、继承，  

面向对象解决方案  

封装），161  

类，19–20  

PinochleDeck 类，279  

对象。*另见* 内存使用  

像素  

对象；物理对象  

颜色，94–95  

调用其方法，30–31  

在窗口坐标系中  

调用方法列表，  

系统，91  

83–84  

播放声音，114–116  

和类，23–25  

游戏场景，石头、剪刀、布  

计数，250  

游戏，315  

从类创建，28–30  

绘制多边形，119  

定义，33  

多态  

封装，164  

经典示例，184–185  

垃圾回收，248  

扩展模式，192  

内部与外部，164–165  

带魔法方法的分数类，  

拥有数据，165  

205–208  

引用计数，242–248  

魔法方法，194–201  

向其发送消息，184  

主程序创建形状，  

值的字符串表示，  

190–192  

203–205  

运算符，193–203  

瞬态类型，242  

pygame 形状，185–192  

具有唯一标识符，66  

与 pygwidgets 一起使用，192–193  

变量引用，244  

向现实世界发送消息  

对象范围，27  

对象，184  

面向对象编程（OOP）  

值的字符串表示，  

解释，3  

对象，203–205  

作为解决方案，45  

向量示例，201–203  

原则，374  

弹出操作，堆栈使用，  

总结，374–375  

堆栈，179  

运算符  

pygame 中的位置参数，145  

魔法方法，194–201  

pygame 绘制的基本形状，  

多态应用于，193–203  

116–120  

o 前缀，26  

print() 函数，133，205  

**382** 索引  

程序化实现。*另见*  

pygame.Rect()，104  

银行账户模拟  

pygame 形状  

类，19–20  

Circle 和 Triangle 形状类，

高低卡牌游戏，3–4  

187–190  

问题，18–19，45  

方形形状类，186–187  

属性  

PygAnimation 基类，304，307–308  

@（at）和装饰器，174–177  

pyghelpers 包  

与抽象相关，181  

用于跟踪时间的类，290  

解释，174  

CountDownTimer 类，293–294  

推操作，堆栈使用，179  

CountUpTimer 类，291–293  

PyCharm IDE，100–101  

安装，287  

**.py** 文件扩展名，61

pygwidgets 类，封装在其中，177

pygame。*另见* 面向对象的 pygame

pygwidgets 包。*另见* pygame

抗锯齿线条，118

GUI 控件

弧线，119

添加图像，151

弹出空白窗口，97–100

Animation 类，304–309

笛卡尔坐标系，91–94

动画程序，308–309

圆形，119

按钮对象，154

颜色，94–95

按钮，152–154

检测鼠标点击，102–105

复选框，152–154

绘制图像，100–102

类，148–149, 157

绘制形状，116–120

类层次结构，237

椭圆，119

API 一致性，158

事件驱动程序，95–96

CustomButton 类，153–154

处理键盘输入，105–109

设计方法，150–151

安装，90–91

DisplayText 类，155, 222

线条，119

示例程序，157–158

基于位置的动画，109–111

目标和类，148–149

像素颜色，94–95

图像，151

播放声音，114–116

和继承，234–236

多边形，119

InputText 类，219

基本形状，118–120

多态，192–193

识别按键，105–107

PygAnimation 基类，307–308

rectobjects，104, 107, 111–114, 119

单选按钮，152–154

连续按键重复

设置，149–150

模式，107–109

sprite 模块，151

Splash 场景，314

SpriteSheetAnimation 类，306–307

状态机示例，314–319

TextButton 类，152–153

窗口坐标系，91–95

文本输出和输入，154–157

pygame.display.set_mode() 函数，98

使用按钮，154

pygame GUI 控件。*另见*

PyPI（Python 包索引），149

pygwidgets 包

Python 的哲学，242

参数，函数和

Python 软件基金会，242

方法，144–148

Python 标准库

关键字和默认值，148

abc（抽象基类）模块，232

None 作为默认值，146–147

调用，76–77

位置参数和关键字参数

getrefcount() 函数，244

参数，145–146

Python Tutor 网站，50, 243

索引 **383**

**R**

在场景之间导航，326

raise 语句和自定义异常，

项目文件夹，321

77–78

退出程序，326

RAM 内存空间，262

剪刀石头布，328–332

random 包，104

run() 方法，336–337

现实世界物体，发送消息

使用，319–320

到，184

SceneMgr 类，337

receive() 方法与场景一起使用，334

ScenePlay 类，Dodger 游戏，

矩形，绘制，120

351–355

Rectangle 类

场景

继承示例，233–234

之间的通信，332–334

使用魔法方法，196–198

当前和目标，332

引用计数，242–246

状态机方法，312–319

递减，245–246

测试场景之间的通信，

递增，245

334

相对路径，使用 pygame，

范围和实例变量，

100–101

27–28

respond() 方法与场景一起使用，333

屏幕保护程序球，构建使用

结果场景，剪刀石头布

面向对象的 pygame，121–127

游戏，315–316

“self” 的含义，52–55

可重用的面向对象按钮，

self 参数，41–42

构建，127–132

self. 前缀，27

可重用的面向对象文本显示，

send() 方法与场景一起使用，333

构建，133–135

在场景中使用 sendAll() 方法，334

RGB（红色，绿色，蓝色），94

设置器和获取器，170–171，175–176

剪刀石头布游戏

Shape 类

Play 场景，315

继承示例，232–233

Results 场景，315–316

作为基类使用，228

Splash 场景，314

在 pygame 中绘制图形，116–120

使用场景，328–332

SimpleAnimation 类，296–300

场景管理器的 run() 方法，

SimpleButton 类，129，131–132，

336，349

139–141

SimpleButton 对象，130–131，

**S**

137–138，141

Sample 类，250

SimpleText 类，133–137

Scene 基类，322

SimpleText 对象，135

场景管理器

Slider 类，193

构建场景，323–326

滑块拼图用户界面，290，293

场景之间的通信，

插槽，作为内存管理使用，

338–340

261–263\. *参见* 实例

演示程序，320–328

变量

示例场景，326–328

为物理演示构建软件模型

特性，319–320

对象，22–23

实现，334–340

在 pygame 中播放音效，

主方法，337–338

114–116

主程序，322–323

SpaceShip 类，249

实现场景的方法，

特殊方法，194

324–325

Splash 场景，313–314，347

**384** 索引

SpriteSheetAnimation 类，149，304，

定时器

306–307

构建至主循环，286

精灵表图像，300

计算经过时间，

Square 类，195，227

285–287

继承示例，229

帧计数，283

pygame 图形，186–187

演示程序，282

用于引用计数，243

实现，283–287

Square 对象，243

安装 pyghelpers，287

Stack 类，181

概览，281–282

栈操作，179–180

切换，使用，38

状态图，316

事务对象，242

状态机，pygame 示例，

瞬态对象，242

312–319

Triangle 类，187，227，230–231

统计显示示例，368–373

try/except 技巧，76–81

__str__() 方法，203–204

元组，设置 x 和 y 坐标

子类

作为，151

客户端视图，218–219

TV 类，创建，35–40

继承，212–213，215–217

TV 对象，创建，42

从基类继承，227

type() 函数，32

创建子图像，300\。

*参见* Image 类

**U**

超类，继承，212

用于场景的 update() 方法，325

super() 函数，216

Sweigart，Al，341

**V**

变量。 *参见* 类变量

**T**

指代相同对象，244

临时变量的使用，66

临时使用，66

测试代码

vars() 函数，52

账户，62–64

向量示例，多态，

创建，61–62

201–203

继承，217–218

View 对象，371，373

测试编程，177

textAnswerDialog 对话框，Dodger

**W**

游戏，346

WidgetWithFrills 类，214

TextButton 类，148，235

pygame 窗口坐标系统，

TextCheckBox 类，pygwidgets

91–95

包，149

工作目录，使用时

构建文本显示，133–135

pygame，100

TextRadioButton 类，pygwidgets

包，149

**X**

textYesNoDialog 对话框，Dodger

x 和 y 坐标，设置为

游戏，343

元组，151

抛出异常，76

XTRAs 和对象，访问

时间，显示，290–294

使用数据库，178–179

Timer 类，287–290

定时器演示程序，282

**Y**

定时器事件，284–285

是/否 和警告对话框，Dodger

定时器对象，288

游戏，342–345

索引 **385**

![图片 59](index-416_1.png)

从未有过，世界如此依赖互联网

保持连接并获取信息。这使得电子

前沿基金会的使命是确保技术支持所有人自由、正义和创新。

支持自由、正义和创新，为所有人带来福祉—

比以往任何时候都更加紧迫。

30 多年来，EFF 通过法律诉讼、技术活动及开发软件为科技用户争取权益。

活动主义，通过法院和开发软件应对技术带来的隐私、安全和自由表达等障碍。

克服障碍，保护隐私、安全和自由表达。

这一承诺使我们每个人在黑暗中获得力量。通过

在你的帮助下，我们可以共同迈向更光明的数字未来。

了解更多并加入 EFF，网址：EFF.ORG/NO-STARCH-PRESS

![图片 60](index-417_1.jpg)

![图片 61](index-417_2.png)

![图片 62](index-417_3.jpg)

![图片 63](index-417_4.jpg)

![图片 64](index-417_5.png)

![图片 65](index-417_6.jpg)

**资源**

访问 [*https://nostarch.com/object-oriented-python/*](https://nostarch.com/object-oriented-python/) 查找勘误和更多信息。

*更多实用书籍来自*

**无锈出版社**

**超越基础知识**

**严肃的 Python**

**缺失的 README**

**用 Python 构建**

**关于部署的黑带建议**

**新软件工程师指南**

**可扩展性、测试及更多**

*由* Chris Riccomini 和

**编写干净代码的最佳实践**

*由* Julien Danjou 编写

Dmitriy Ryaboy

*由* Al Sweigart 编写

240 页，$34.95

288 页，$24.99

384 页，$34.95

ISBN: 978-1-59327-878-6

ISBN: 978-1-7185-0183-6

ISBN: 978-1-59327-966-0

**算法思维**

**程序的秘密生活**

**编写优秀代码，第 3 卷**

**基于问题的入门介绍**

**了解计算机—编写更好的代码**

**工程软件**

*由* Daniel Zingaro 编写

*由* Jonathan E. Steinhart 编写

*由* Randall Hyde 编写

408 页，$49.95

504 页，$44.95

376 页，$49.95

ISBN: 978-1-7185-0080-8

ISBN: 978-1-59327-970-7

ISBN: 978-1-59327-979-0

**电话：**

**电子邮件：**

800.420.7240 或

sales@nostarch.com

415.863.9900

**网址：**

www.nostarch.com

![图片 66](index-418_1.png)

![图片 67](index-418_2.png)

![图片 68](index-418_3.png)

![图片 69](index-418_4.png)

![图片 70](index-418_5.png)

![图片 71](index-418_6.png)

![图片 72](index-418_7.png)

![图片 73](index-418_8.png)

![图片 74](index-418_9.png)

![图片 75](index-418_10.png)

**构建与**

**掌控**

**Python 3.x 和**

**你的 Python**

**P Y G A M E 2 . x**

**对象**

**面向对象**

**面向对象 Python**

**P Y T H O N**

面向对象编程（OOP）是一种编程范式

你将学习如何：

**掌握面向对象编程**

将数据和代码结合成一个有机的单元，

• 使用对象创建和管理多个对象

**构建游戏和图形用户界面**

让你从不同的角度思考计算问题

解决问题并以高度可重用的方式处理它们。

管理器对象

面向中级程序员的*对象-*

• 使用封装隐藏对象的内部细节

*面向对象 Python* 是一个深入的实践教程

从客户端代码

**I R V K A L B**

融入 OOP 的核心理念，展示如何使用

封装、多态和继承来编写

• 使用多态定义一个接口并

游戏和应用程序，使用 Python。

在多个类中实现

书籍通过展示关键问题开始

• 应用继承扩展现有代码

过程式编程中的固有问题，然后引导你

通过创建类和对象的基础知识

*面向对象 Python* 是一本直观的视觉指南

Python。你将在此基础上通过开发

完全理解面向对象编程（OOP）的运作方式以及你如何

按钮、文本框以及其他 GUI 元素

可以使用它使你的代码更易于维护，

在事件驱动环境中，标准化。你还将使用

可读且高效的代码——不牺牲

多个真实世界的代码示例和两个 pygame

功能性。

基于包的开发帮助将理论转化为实践，

使你能够轻松编写互动游戏和

**关于作者**

完成带有 GUI 小部件、动画的应用程序，

Irv Kalb 是 UCSC 硅谷分校的兼职教授

多个场景，以及可重用的游戏逻辑。在最后

硅谷扩展及硅谷大学，

本章，你将通过构建一个

他在 UCSC 硅谷分校及硅谷大学教授

完整功能的视频游戏，融入许多

Python 编程课程的作者之一。他也是

OOP 技术和 GUI 元素的应用

来自 *Python 3 编程入门：一步一步的指南*

本书中。

*编程指南。*

***极客娱乐的最佳选择 ™***

www.nostarch.com

**KALB**

**$44.99（$59.99 CDN）**

# 文档大纲

+   [致谢](index_split_000.html#p20)

+   [简介](index_split_000.html#p22)

+   [第一部分：介绍面向对象编程](index_split_000.html#p30)

    +   [第1章：过程式 Python 示例](index_split_000.html#p32)

        +   [高低牌游戏](index_split_000.html#p33)

        +   [银行账户模拟](index_split_000.html#p36)

        +   [过程式实现中的常见问题](index_split_000.html#p47)

        +   [面向对象解决方案——首次了解类](index_split_000.html#p48)

        +   [总结](index_split_000.html#p49)

    +   [第2章：使用面向对象编程建模物理对象](index_split_000.html#p50)

        +   [构建物理对象的软件模型](index_split_000.html#p51)

        +   [类与对象简介](index_split_000.html#p52)

        +   [类、对象与实例化](index_split_000.html#p54)

        +   [构建稍微复杂一些的类](index_split_001.html#p62)

        +   [将更复杂的物理对象表示为类](index_split_001.html#p64)

        +   [类的应用](index_split_001.html#p74)

        +   [OOP 作为解决方案](index_split_001.html#p74)

        +   [总结](index_split_001.html#p75)

    +   [第3章：对象的心理模型与“self”的意义](index_split_001.html#p76)

        +   [重新审视DimmerSwitch类](index_split_001.html#p77)

        +   [高级思维模型#1](index_split_001.html#p78)

        +   [更深层的思维模型#2](index_split_001.html#p78)

        +   [“self”是什么意思？](index_split_001.html#p81)

        +   [总结](index_split_001.html#p84)

    +   [第4章：管理多个对象](index_split_001.html#p86)

        +   [银行账户类](index_split_001.html#p87)

        +   [导入类代码](index_split_001.html#p89)

        +   [创建一些测试代码](index_split_001.html#p90)

        +   [创建一个对象管理器对象](index_split_001.html#p99)

        +   [通过异常处理更好的错误管理](index_split_001.html#p105)

        +   [在我们的银行程序中使用异常](index_split_001.html#p107)

        +   [在一系列对象上调用相同的方法](index_split_002.html#p112)

        +   [接口与实现](index_split_002.html#p113)

        +   [总结](index_split_002.html#p114)

+   [第二部分：使用Pygame的图形用户界面](index_split_002.html#p116)

    +   [第5章：Pygame介绍](index_split_002.html#p118)

        +   [安装Pygame](index_split_002.html#p119)

        +   [窗口详情](index_split_002.html#p120)

        +   [事件驱动的程序](index_split_002.html#p124)

        +   [使用Pygame](index_split_002.html#p125)

        +   [播放声音](index_split_002.html#p143)

        +   [绘制形状](index_split_002.html#p145)

        +   [总结](index_split_002.html#p149)

    +   [第6章：面向对象的Pygame](index_split_002.html#p150)

        +   [使用OOP Pygame构建屏幕保护程序球](index_split_002.html#p150)

        +   [构建可重用的面向对象按钮](index_split_002.html#p156)

        +   [构建可重用的面向对象文本显示](index_split_002.html#p162)

        +   [带有SimpleText和SimpleButton的Demo球](index_split_003.html#p164)

        +   [接口与实现](index_split_003.html#p166)

        +   [回调函数](index_split_003.html#p166)

        +   [总结](index_split_003.html#p170)

    +   [第7章：Pygame GUI小部件](index_split_003.html#p172)

        +   [将参数传递给函数或方法](index_split_003.html#p173)

        +   [pygwidgets包](index_split_003.html#p177)

        +   [一致的API的重要性](index_split_003.html#p187)

        +   [总结](index_split_003.html#p187)

+   [第三部分：封装、多态和继承](index_split_003.html#p190)

    +   [第8章：封装](index_split_003.html#p192)

        +   [使用函数的封装](index_split_003.html#p193)

        +   [使用对象的封装](index_split_003.html#p193)

        +   [封装的解释](index_split_003.html#p194)

        +   [让实例变量更私有](index_split_003.html#p201)

        +   [装饰器和@property](index_split_003.html#p203)

        +   [pygwidgets类中的封装](index_split_003.html#p206)

        +   [来自现实世界的故事](index_split_003.html#p207)

        +   [抽象](index_split_003.html#p208)

        +   [总结](index_split_003.html#p211)

    +   [第9章：多态](index_split_003.html#p212)

        +   [向现实世界的对象发送消息](index_split_003.html#p213)

        +   [编程中多态的经典示例](index_split_003.html#p213)

        +   [使用Pygame形状的示例](index_split_003.html#p214)

        +   [pygwidgets 展示多态性](index_split_004.html#p221)

        +   [运算符的多态性](index_split_004.html#p222)

        +   [创建对象值的字符串表示](index_split_004.html#p232)

        +   [具有魔法方法的分数类](index_split_004.html#p234)

        +   [总结](index_split_004.html#p237)

    +   [第10章：继承](index_split_004.html#p240)

        +   [面向对象编程中的继承](index_split_004.html#p241)

        +   [实现继承](index_split_004.html#p242)

        +   [员工与经理示例](index_split_004.html#p243)

        +   [客户端对子类的视图](index_split_004.html#p247)

        +   [继承的现实世界示例](index_split_004.html#p248)

        +   [多个类继承自同一基类](index_split_004.html#p256)

        +   [抽象类与方法](index_split_004.html#p260)

        +   [pygwidgets 如何使用继承](index_split_004.html#p263)

        +   [类层次结构](index_split_004.html#p265)

        +   [使用继承编程的困难](index_split_004.html#p267)

        +   [总结](index_split_004.html#p268)

    +   [第11章：管理对象占用的内存](index_split_004.html#p270)

        +   [对象生命周期](index_split_005.html#p271)

        +   [类变量](index_split_005.html#p277)

        +   [综合应用：气球示例程序](index_split_005.html#p280)

        +   [内存管理：槽](index_split_005.html#p290)

        +   [总结](index_split_005.html#p292)

+   [第四部分：在游戏开发中使用OOP](index_split_005.html#p294)

    +   [第12章：卡牌游戏](index_split_005.html#p296)

        +   [卡牌类](index_split_005.html#p297)

        +   [牌组类](index_split_005.html#p299)

        +   [更高或更低游戏](index_split_005.html#p301)

        +   [使用 __name__ 进行测试](index_split_005.html#p305)

        +   [其他卡牌游戏](index_split_005.html#p307)

        +   [总结](index_split_005.html#p308)

    +   [第13章：定时器](index_split_005.html#p310)

        +   [定时器演示程序](index_split_005.html#p311)

        +   [实现定时器的三种方法](index_split_005.html#p312)

        +   [安装 pyghelpers](index_split_005.html#p316)

        +   [定时器类](index_split_005.html#p316)

        +   [显示时间](index_split_005.html#p319)

        +   [总结](index_split_005.html#p323)

    +   [第14章：动画](index_split_005.html#p324)

        +   [构建动画类](index_split_005.html#p325)

        +   [pygwidgets中的动画类](index_split_006.html#p333)

        +   [总结](index_split_006.html#p339)

    +   [第15章：场景](index_split_006.html#p340)

        +   [状态机方法](index_split_006.html#p341)

        +   [使用状态机的 pygame 示例](index_split_006.html#p343)

        +   [用于管理多个场景的场景管理器](index_split_006.html#p348)

        +   [使用场景管理器的演示程序](index_split_006.html#p349)

        +   [使用场景的石头、剪子、布游戏](index_split_006.html#p357)

        +   [场景之间的通信](index_split_006.html#p361)

        +   [场景管理器的实现](index_split_006.html#p363)

        +   [总结](index_split_006.html#p369)

    +   [第16章：完整游戏：躲避者](index_split_006.html#p370)

        +   [模态对话框](index_split_006.html#p371)

        +   [构建完整游戏：躲避者](index_split_006.html#p376)

        +   [总结](index_split_007.html#p395)

    +   [第17章：设计模式与总结](index_split_007.html#p396)

        +   [模型视图控制器](index_split_007.html#p396)

        +   [总结](index_split_007.html#p403)

+   [索引](index_split_007.html#p406)
