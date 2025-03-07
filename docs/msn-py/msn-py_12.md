## **12  

危险！危险！**添加危险元素**  

![image](img/common01.jpg)  

当空间站系统故障时，各种威胁出现。在本章中，你将看到空气开始从空间站泄漏，并且会在一些房间中发现移动的危险，包括流氓机器人、能量球和有毒的水坑。  

我把危险放在最后，这样你可以在不担心时间或精力耗尽的情况下先测试游戏到这一点。在本章中，我们将开始漏气并绘制一个计时条，让你知道剩余多少空气。我们还将加入一些可能伤害你的危险，并消耗你的能量。最后，我们将清理游戏并准备好进行游戏！  

### **添加空气倒计时**  

玩家在游戏中有两种失败方式：空气用尽或能量用尽。在屏幕底部，两个条形图显示玩家剩余的空气和能量（见图 12-1）。  

![image](img/fig12-1.jpg)  

*图 12-1：屏幕底部的两个条形图显示剩余的空气和能量。*  

当你走过有毒的溢出物，或者被移动的危险物体击中时，你会失去能量，空气也会因为空间站墙壁的泄漏而逐渐耗尽。如果你穿上太空服，你可以争取更多的时间，但太空服的气瓶最终也会耗尽。你最艰难的决定之一可能是决定什么时候补充空气和使用食物来恢复能量。  

#### **显示空气和能量条**  

我们将创建一个新的程序部分，称为`AIR`，你需要将它放在`DOORS`部分之后，但在程序末尾的`START`部分之前。将清单 12-1 中的新代码添加到你上一个章节的最终清单中（*listing11-7.py*）。将文件保存为*listing12-1.py*。如果你运行程序，它不会做任何新的事情，但这段代码为绘制空气和能量条创建了函数。  

*listing12-1.py*  

```py
   --snip--
       objects[21][0] = frames[airlock_door_frame]
       objects[21][1] = shadow_frames[airlock_door_frame]

   ###############
   ##    AIR    ##
   ###############

   def draw_energy_air():
       box = Rect((20, 765), (350, 20))
➊     screen.draw.filled_rect(box, BLACK) # Clear air bar.
➋     screen.draw.text("AIR", (20, 766), color=BLUE)
➌     screen.draw.text("ENERGY", (180, 766), color=YELLOW)

➍     if air > 0:
➎         box = Rect((50, 765), (air, 20))
➏         screen.draw.filled_rect(box, BLUE) # Draw new air bar.

➐     if energy > 0:
           box = Rect((250, 765), (energy, 20))
           screen.draw.filled_rect(box, YELLOW) # Draw new energy bar.

   ###############
   ##   START   ##
   ###############
   --snip--
```

*清单 12-1：绘制空气和能量条*  

我们通过在屏幕底部的状态区域绘制一个黑色框来清除它，开始新的`draw_energy_air()`函数 ➊。然后我们添加蓝色的 AIR 标签 ➋，黄色的 ENERGY 标签 ➌。这个函数将使用已经在程序的`VARIABLES`部分设置为 100 的`air`和`energy`变量。  

如果玩家还有一些空气（如果变量`air`大于 0） ➍，将创建一个使用`air`变量作为宽度的框 ➎。然后，框将被填充为蓝色 ➏。这会绘制 AIR 指示条，它最初宽度为 100 像素，并随着`AIR`变量的减少而变小。

我们使用类似的指令来绘制能量条 ➐，但条形图的起始位置更靠右（*x*位置是 250，而不是 50）。  

#### **添加空气倒计时功能**  

我们将创建三个函数来启用空气倒计时。当你没有空气时，`end_the_game()`函数会运行。它会显示玩家失败任务的原因，播放一些音效，并在游戏窗口中央显示一个大的游戏结束消息。

`air_countdown()`函数消耗空气供应。我们还会添加一个`alarm()`函数，在游戏开始后不久运行，警告玩家他们的空气供应不足。

这三个函数位于清单 12-2 中。将此处显示的新代码添加到程序的`AIR`部分，在你刚刚添加的`draw_energy_air()`函数之后。将程序保存为*listing12-2.py*。你可以使用`pgzrun listing12-2.py`运行此程序，但此时你还看不到任何新内容。

*listing12-2.py*

```py
   --snip--
       if energy > 0:
           box = Rect((250, 765), (energy, 20))
           screen.draw.filled_rect(box, YELLOW) # Draw new energy bar.

➊ def end_the_game(reason):
       global game_over
➋     show_text(reason, 1)
➌     game_over = True
       sounds.say_mission_fail.play()
       sounds.gameover.play()
➍     screen.draw.text("GAME OVER", (120, 400), color = "white",
                        fontsize = 128, shadow = (1, 1), scolor = "black")

➎ def air_countdown():
       global air, game_over
       if game_over:
           return # Don't sap air when they're already dead.
➏     air -= 1
➐     if air == 20:
           sounds.say_air_low.play()
       if air == 10:
           sounds.say_act_now.play()
➑     draw_energy_air()
➒     if air < 1:
           end_the_game("You're out of air!")

➓ def alarm():
       show_text("Air is running out, " + PLAYER_NAME
                 + "! Get to safety, then radio for help!", 1)
       sounds.alarm.play(3)
       sounds.say_breach.play()

   ###############
   ##   START   ##
   ###############
   --snip--
```

*清单 12-2：添加空气倒计时*

`air_countdown()`函数 ➎ 每次运行时都会减少`air`变量的值 1 ➏。如果值等于 20 ➐或 10，会播放一个警告音效，提醒玩家他们的空气已经很低。

你在清单 12-1 中添加的`draw_energy_air()`函数会更新空气和能量的显示 ➑。如果空气耗尽 ➒，`end_the_game()`函数会运行并显示字符串`"You're out of air!"`。

**提示**

音频文件必须存储在*sounds*文件夹中，格式应为*.wav*或*.ogg*。要播放名为*bang.wav*的声音，可以使用`sounds.bang.play()`。与图像一样，你不需要告诉 Pygame Zero 音频文件的扩展名或其存储位置。为什么不尝试为游戏中的不同点录制并添加你自己的音效呢？

在`end_the_game()`函数 ➊ 中，我们使用变量`reason`来存储接收到的信息，并将其作为死亡原因显示在屏幕上 ➋。`game_over`变量被设置为`True` ➌。其他函数使用这个变量来判断游戏是否结束，从而使一切停止。`end_the_game()`函数随后在屏幕中央绘制“游戏结束”的大字。文字的位置是*x* = 120，*y* = 400，使用白色文字和 128 的字体大小 ➍。我们还为文字添加了一个投影效果，投影偏移了 1 像素，并且是黑色的（见图 12-2）。

![image](img/fig12-2.jpg)

*图 12-2：哦不！你没氧气了！*

本节中的最后一个函数`alarm()` ➓，播放警报声音并显示一条消息，提醒你呼叫无线电寻求帮助。它在警告中使用玩家的名字，以便个性化。

`sounds.alarm.play()`命令中括号里的数字表示声音应该播放的次数（在清单 12-2 中，它是三次）。

#### **启动空气倒计时并触发警报**

我们还没有设置这三个新函数的运行。为此，我们需要在程序的 `START` 部分添加一些指令，这部分（也许让人困惑的是）位于程序清单的末尾。添加 清单 12-3 中显示的新指令，并将其保存为 *listing12-3.py*。

*listing12-3.py*

```py
--snip--
###############
##   START   ##
###############

clock.schedule_interval(game_loop, 0.03)
generate_map()
clock.schedule_interval(adjust_wall_transparency, 0.05)
clock.schedule_unique(display_inventory, 1)
clock.schedule_unique(draw_energy_air, 0.5)
clock.schedule_unique(alarm, 10)
# A higher number below gives a longer time limit.
clock.schedule_interval(air_countdown, 5)
```

*清单 12-3：启动空气倒计时*

现在游戏有了时间限制。当空气用尽时，游戏结束。使用 `pgzrun listing12-3.py` 运行程序，你应该能看到空气供应逐渐下降。

如果你发现游戏的最终版本太难，可以通过将 清单 12-3 中最后一行的 5 改为更高的数字，来给自己更多时间。这个数字决定了 `air_countdown()` 函数消耗空气供应的频率，单位是秒。特别是，如果你使用的是 Raspberry Pi 2，时间限制可能会显得很具挑战性，因为该平台上游戏运行较慢。尽管如此，完成游戏仍然是可能的，但你可以将数字 5 增大一点，给自己更多的、咳咳、呼吸空间。

**训练任务 #1**

当你的空气供应降到 0 时，你应该会看到 "GAME OVER" 信息，并且发现你无法再控制宇航员。每 5 秒你的能量下降 1%，因此大约 8.5 分钟（500 秒）后空气会耗尽。你能想出如何让空气泄漏得更频繁一点，以便更轻松地测试空气耗尽时会发生什么吗？

完成训练任务后，确保将程序恢复原状：否则你会发现任务完成起来相当困难！

### **添加移动危险物**

游戏中有三种移动的危险物：两种类型的能量球和一架失控的飞行无人机。

图 12-3 显示了这些移动危险物使用的方向编号。

危险物会沿直线移动，直到碰到某物，然后我们会添加一个数字来改变它们的方向。我们添加的数字决定了危险物的移动模式。例如，如果我们向方向编号添加 `1`，危险物将按顺时针方向移动（上、右、下、左）。如果我们向方向编号添加 `-1`，危险物将按逆时针方向移动（左、下、右、上）。如果我们添加 `2`，它将在左右（2 和 4）或上下（1 和 3）之间反弹。看看 图 12-3，确认这是否能理解。每个危险物都可以有自己的移动模式。

![image](img/fig12-3.jpg)

*图 12-3：移动危险物使用的方向编号按顺时针顺序编号。*

如果加法结果超过 4，我们就减去 4。例如，如果一个危险物体按顺时针方向移动，每次碰到物体时我们都会将方向数加 1。如果它是向下移动（方向 3），碰到物体时我们加 1，使得它开始向左移动（方向 4）。下一次碰到物体时，我们加 1，这使得方向数变为 5。因此，我们减去 4，得到方向数 1。如图 12-3 所示，这是 4 之后的下一个方向数，按顺时针方向旋转。

表 12-1 总结了我们可以使用的数字，以获得不同的移动模式。

**表 12-1：** 当危险物体碰到物体时如何改变方向

| **移动模式** |  | **要加到方向数上的数字** |
| --- | --- | --- |
| 顺时针 | ![image](img/f0203-01.jpg) | 1 |
| 逆时针 | ![image](img/f0203-02.jpg) | -1 |
| 左/右 | ![image](img/f0203-03.jpg) | 2 |
| 上/下 | ![image](img/f0203-04.jpg) | 2 |

**红色警报**

*请小心不要混淆描述移动的两个数字。方向数（见图 12-3）告诉程序危险物体的移动方向。我们加到方向数上的数字（见表 12-1）则告诉程序当危险物体碰到物体时应该如何反弹。*

#### **添加危险数据**

在`AIR`和`START`部分之间，我们将在程序中添加一个名为`HAZARDS`的新部分。列表 12-4 展示了危险数据。将其添加到程序中，并保存为*listing12-4.py*。如果运行程序，它暂时不会做任何新操作，但你可以检查命令行窗口中是否没有出现错误信息。

*listing12-4.py*

```py
   --snip--
       sounds.alarm.play(3)
       sounds.say_breach.play()    

   ###############
   ##  HAZARDS  ##
   ###############

   hazard_data = {
       # room number: [[y, x, direction, bounce addition to direction]]
➊     28: [[1, 8, 2, 1], [7, 3, 4, 1]], 32: [[1, 5, 1, 1]],
       34: [[5, 1, 1, 1], [5, 5, 1, 2]], 35: [[4, 4, 1, 2], [2, 5, 2, 2]],
       36: [[2, 1, 2, 2]], 38: [[1, 4, 3, 2], [5, 8, 1, 2]],
       40: [[3, 1, 3, 1], [6, 5, 2, 2], [7, 5, 4, 2]],
       41: [[4, 5, 2, 2], [6, 3, 4, 2], [8, 1, 2, 2]],
       42: [[2, 1, 2, 2], [4, 3, 2, 2], [6, 5, 2, 2]],
       46: [[2, 1, 2, 2]],
       48: [[1, 8, 3, 2], [8, 8, 1, 2], [3, 9, 3, 2]]
       }

   ###############
   ##   START   ##
   ###############
   --snip--
```

*列表 12-4：添加危险数据*

我们创建了一个`hazard_data`字典，使用房间编号作为字典的键。对于每个房间，都会有一个包含所有危险数据的列表。每个危险的数据以一个包含*y*位置、*x*位置、起始方向和碰到物体时要加的数字的列表形式表示。

例如，房间 28 ➊ 存在一个包含列表数据`[7, 3, 4, 1]`的危险。这意味着该危险从*y* = 7，*x* = 3 开始。它开始向左移动（方向 4），当它碰到物体时会顺时针转动，因为我们将 1 加到它的方向数上。

房间 41 包含三个危险（分别在三个列表中），它们从左向右并来回移动。我们知道这一点，因为它们的方向初始为 2 或 4（向右或向左），并在碰到物体时将方向加 2（变为 4 或 6：我们知道 6 减去 4 会变成 2）。

#### **消耗玩家能量**

在危害数据之后，我们需要添加一个名为`deplete_energy()`的函数，当危害击中玩家时会减少玩家的能量。列表 12-5 展示了这个新函数。将它添加到程序的`HAZARDS`部分中列表 12-4 后面，并将程序保存为*listing12-5.py*。你可以使用 pgzrun listing12-5.py 运行程序来检查错误，但它不会做任何新的操作。

*listing12-5.py*

```py
   --snip--
       46: [[2, 1, 2, 2]],
       48: [[1, 8, 3, 2], [8, 8, 1, 2], [3, 9, 3, 2]]
       }

➊ def deplete_energy(penalty):
       global energy, game_over
       if game_over:
           return # Don't sap energy when they're already dead.
➋     energy = energy - penalty
       draw_energy_air()
       if energy < 1:
           end_the_game("You're out of energy!")

   ###############
   ##   START   ##
   ###############
   --snip--
```

*列表 12-5：减少玩家的能量*

`deplete_energy()`函数接受一个数字 ➊，并使用这个数字来减少玩家的`energy`变量 ➋。因此，我们可以用这个函数来处理消耗不同能量值的危害。

#### **启动和停止危害**

当玩家进入新房间时，函数`hazard_start()`会将危害添加到房间中。列表 12-6 展示了这个函数，你需要将它添加到程序中`HAZARDS`部分的`deplete_energy()`函数后面。将程序保存为*listing12-6.py*。如果你使用 pgzrun listing12-6.py 运行它，你应该不会注意到任何变化，因为我们还没有设置这个函数来运行。

*listing12-6.py*

```py
   --snip--
       if energy < 1:
           end_the_game("You're out of energy!")

   def hazard_start():
       global current_room_hazards_list, hazard_map
➊     if current_room in hazard_data.keys():
➋         current_room_hazards_list = hazard_data[current_room]
➌         for hazard in current_room_hazards_list:
               hazard_y = hazard[0]
               hazard_x = hazard[1]
➍             hazard_map[hazard_y][hazard_x] = 49 + (current_room % 3)
➎         clock.schedule_interval(hazard_move, 0.15)

   ###############
   ##   START   ##
   ###############
   --snip--
```

*列表 12-6：将危害添加到当前房间*

`hazard_start()`函数会在每次玩家进入新房间时运行，因此它首先检查当前房间在`hazard_data`字典中是否有条目 ➊。如果有，这个房间应该有移动的危害，接下来会执行函数的其余部分。我们将房间的危害数据放入一个名为`current_room_hazards_list`的列表 ➋。然后函数使用一个循环 ➌依次处理列表中的每个危害。

危害使用它们自己的房间地图，称为`hazard_map`，这样它们可以轻松地飞越地板上的物体，而不会覆盖房间地图中的物品。如果危害使用与道具相同的房间地图，它们就会在飞越道具时将道具抹去，或者我们需要一个复杂的方法来记住危害下方的内容。

这三个危害对象在`objects`字典中分别对应数字 49、50 和 51。程序使用简单的计算来确定哪个危害对象应该进入特定的房间。正如你之前所见，Python 的`%`操作符会给出除法后的余数。当你将任何数字除以 3 时，余数将是 0、1 或 2。所以，程序会将房间号除以 3，然后将余数加到 49 上，从而选出一个危害对象的编号 ➍。例如，如果我们在房间 34，程序会计算出`34 % 3`的余数是 1，然后将 1 加到 49 上，选择编号为 50 的危害对象。

这种选择危害编号的方式确保了玩家进入房间时，危害始终是相同类型的。由于地图宽度为五个房间，它还保证了两个直接相连的房间不能有相同的危害。这为地图增加了多样性，尽管并非所有房间都有危害，因此在实践中，玩家可能会在走过一些空房间后，连续两次遇到相同的危害。

该函数的最后，通过安排每隔 0.15 秒执行一次 `hazard_move()` 函数来完成 ➎。

要在玩家进入新房间时启动 `hazard_start()` 函数，在 `start_room()` 函数中添加一条指令，如 列表 12-7 所示。将你的程序保存为 *listing12-7.py*。该版本的程序将在你离开起始房间时冻结，因为我们还没有完成添加危害的代码。

*listing12-7.py*

```py
--snip--
###############
## GAME LOOP ##
###############

def start_room():
    global airlock_door_frame
    show_text("You are here: " + room_name, 0)
    if current_room == 26: # Room with self-shutting airlock door
        airlock_door_frame = 0
        clock.schedule_interval(door_in_room_26, 0.05)
    hazard_start()
--snip--
```

*列表 12-7：玩家进入房间时启动危害*

并非所有房间都有危害，因此当玩家离开房间时，我们会停止危害的移动。我们之前在 `game_loop()` 函数中添加了指令，用于在玩家改变房间时关闭让危害移动的函数。我们将它们注释掉了，因为那时我们还没有准备好使用它们。

现在我们准备好添加它们了！按照以下步骤取消注释这些指令（你在 第八章 中做过类似的操作）：

1.  在 IDLE 中点击 **编辑** ▸ **替换**（或按 CTRL-H）以显示替换文本对话框。

1.  在“查找”框中输入 #clock.unschedule(hazard_move)。

1.  在“替换为”框中输入 clock.unschedule(hazard_move)。

1.  点击 **全部替换**。IDLE 应该会在四个地方替换该指令，并跳转到列表中的最后一项。列表 12-8 显示了过程结束时将被高亮的新行（你无需输入此列表）。在这段代码之前，有三个类似的代码块，现在也会在玩家离开房间时通过出口停止危害的移动。

*listing12-8.py*

```py
--snip--
    if player_y == -1: # through door at TOP
        clock.unschedule(hazard_move)
        current_room -= MAP_WIDTH
        generate_map()
        player_y = room_height - 1 # enter at bottom
        player_x = int(room_width / 2) # enter at door
        player_frame = 0
        start_room()
        return
--snip--
```

*列表 12-8：当玩家离开房间时停止危害移动*

将你的更新程序保存为 *listing12-8.py*。如果你运行此版本的程序，控制台将显示错误信息，且当你离开房间时，游戏将会冻结。原因是我们还没有添加 `hazard_move()` 函数。

#### **设置危害地图**

我们现在需要确保，当生成房间地图、景物和道具时，也会生成一个空的危害地图。`hazard_start()` 函数将用房间中的任何危害来填充这个地图。

在程序的 `MAKE MAP` 部分的 `generate_map()` 函数末尾，添加 列表 12-9 中显示的新代码。将此新代码放在 `GAME LOOP` 部分之前，并确保将第一行缩进四个空格，因为它位于函数内部。

保存程序为 *listing12-9.py*。当你运行它时，程序还不能正常工作，因为它还不完整。

*listing12-9.py*

```py
--snip--
                for tile_number in range(1, image_width_in_tiles):
                    room_map[prop_y][prop_x + tile_number] = 255    

    hazard_map = [] # empty list
    for y in range(room_height):
        hazard_map.append( [0] * room_width )

###############
## GAME LOOP ##
###############
--snip--
```

*清单 12-9：创建空的危险物体地图*

这些新指令为危险物体地图创建了一个空列表，并用与房间宽度相同的 0 填充每一行。

#### **让危险物体移动**

现在，让我们添加缺失的 `hazard_move()` 函数，使危险物体能够移动。将其放在程序中 `HAZARDS` 部分的 `hazard_start()` 函数之后，正如 清单 12-10 所示。保存程序为 *listing12-10.py*。

*listing12-10.py*

```py
   --snip--
               hazard_map[hazard_y][hazard_x] = 49 + (current_room % 3)
           clock.schedule_interval(hazard_move, 0.15)

   def hazard_move():
       global current_room_hazards_list, hazard_data, hazard_map
       global old_player_x, old_player_y

       if game_over:
           return
       for hazard in current_room_hazards_list:
           hazard_y = hazard[0]
           hazard_x = hazard[1]
           hazard_direction = hazard[2]

➊         old_hazard_x = hazard_x
           old_hazard_y = hazard_y
           hazard_map[old_hazard_y][old_hazard_x] = 0

➋         if hazard_direction == 1: # up
               hazard_y -= 1
           if hazard_direction == 2: # right
               hazard_x += 1
           if hazard_direction == 3: # down
               hazard_y += 1
           if hazard_direction == 4: # left
               hazard_x -= 1

           hazard_should_bounce = False

➌         if (hazard_y == player_y and hazard_x == player_x) or \
              (hazard_y == from_player_y and hazard_x == from_player_x
               and player_frame > 0):
               sounds.ouch.play()
               deplete_energy(10)
               hazard_should_bounce = True

➍         # Stop hazard going out of the doors
           if hazard_x == room_width:
               hazard_should_bounce = True
               hazard_x = room_width - 1
           if hazard_x == -1:
               hazard_should_bounce = True
               hazard_x = 0
           if hazard_y == room_height:
               hazard_should_bounce = True
               hazard_y = room_height - 1
           if hazard_y == -1:
               hazard_should_bounce = True
               hazard_y = 0

➎         # Stop when hazard hits scenery or another hazard.
           if room_map[hazard_y][hazard_x] not in items_player_may_stand_on \
                  or hazard_map[hazard_y][hazard_x] != 0:
               hazard_should_bounce = True

➏         if hazard_should_bounce:
               hazard_y = old_hazard_y # Move back to last valid position.
               hazard_x = old_hazard_x
➐             hazard_direction += hazard[3]
➑             if hazard_direction > 4:
                   hazard_direction -= 4
               if hazard_direction < 1:
                   hazard_direction += 4
➒             hazard[2] = hazard_direction
➓         hazard_map[hazard_y][hazard_x] = 49 + (current_room % 3)
           hazard[0] = hazard_y
           hazard[1] = hazard_x

   ###############
   ##   START   ##
   ###############
   --snip--
```

*清单 12-10：添加危险物体移动函数*

`hazard_move()` 函数使用了类似于玩家移动的思想。危险物体的位置存储在 `old_hazard_x` 和 `old_hazard_y` 变量中 ➊。然后，危险物体被移动 ➋。

然后，我们检查危险物体是否撞到了玩家 ➌、是否出了门 ➍，或者撞到了景物或另一个危险物体 ➎。如果是 ➏，则将其位置重置为原来的值，并通过将数据列表中的最后一个数字加到方向编号上来改变其方向 ➐。如果加上这个数字后，方向编号超过 4 ➑，函数会减去 4，正如我们在本章之前讨论的，因为 4 是最大的有效方向编号。另一方面，如果加上这个数字后，方向编号小于 1，函数则会加上 4。最后，新的方向会保存在危险物体数据中 ➒。

在函数的末尾 ➓，危险物体被放入了危险物体地图。

你可以通过运行 `pgzrun listing12-10.py` 来启动这个程序。第一个有危险物体的房间是在你起始房间的右侧。当你进入时，尽管你看不见任何危险物体，但你的能量有时会神秘地下降。这是因为我们还没有添加绘制危险物体的代码。

**提示**

当危险物体碰到你时 ➌，`deplete_energy()` 函数会减少你 10% 的能量。如果你觉得游戏太难，可以将这个数字改为 5。如果你完成游戏并希望下次有更大的挑战，可以将它改为 20！

#### **在房间中显示危险物体**

让我们为房间中的危险物体添加几行代码，以便显示它们，因为不可见的危险似乎不太公平。清单 12-11 显示了要添加到程序 `DISPLAY` 部分的 `draw()` 函数中的三行新代码。将这些代码放在函数末尾，绘制玩家代码之前。

由于这些指令在 `draw()` 函数内（缩进 4 个空格）、在 `y` 循环内（再缩进 4 个空格）以及在 `x` 循环内（再缩进 4 个空格），因此总共需要缩进 12 个空格。保存程序为 *listing12-11.py*。

*listing12-11.py*

```py
--snip--
                        # Use shadow across width of object.
                        for z in range(0, shadow_width):
                            draw_shadow(shadow_image, y, x+z)
                    else:
                        draw_shadow(shadow_image, y, x)

            hazard_here = hazard_map[y][x]
            if hazard_here != 0: # If there's a hazard at this position
                draw_image(objects[hazard_here][0], y, x)

        if (player_y == y):
                draw_player()
--snip--
```

*清单 12-11：显示移动中的危险物体*

这个列表完成了移动的危险物体。使用 pgzrun listing12-11.py 来运行你的程序。然后开始奔命吧！你现在应该能看到移动的危险物体，比如图 12-4 中展示的能量球。

![image](img/fig12-4.jpg)

*图 12-4：这个能量球沿逆时针方向在房间内弹跳。*

**训练任务 #2**

测试移动的危险物体是否正常工作。进入你起始房间右侧的房间（或者必要时传送到房间 32）。当能量球撞到你时，你的能量是否减少？能量球会弹开吗？你能把能量球弹到两个门口，检查它是否停留在房间内吗？当你的能量用完时，游戏会结束吗？

#### **阻止玩家穿过危险物体**

我们还需要添加一行代码来防止玩家走进或穿过危险区域。实际上，危险物体通常会从玩家身上弹开，但如果没有在列表 12-12 中做出修复，有时玩家是可以穿过危险的。

我们已经在`game_loop()`函数中添加了需要的代码，但我们将其注释掉了。现在是时候取消注释了，通过删除`\`末尾行前的`#`符号，以及删除下一行开头的`#`符号。

我们还需要删除`items_player_may_stand_on`后面的冒号。一个快速找到正确部分的方式是按 CTRL-F 打开搜索框，然后输入`#\`。 列表 12-12 展示了需要修改的行。

*listing12-12.py*

```py
--snip--
  # If the player is standing somewhere they shouldn't, move them back.
    if room_map[player_y][player_x] not in items_player_may_stand_on \
               or hazard_map[player_y][player_x] != 0:
        player_x = old_player_x
        player_y = old_player_y
        player_frame = 0
--snip--
```

*列表 12-12：阻止玩家穿过危险物体*

将你的程序保存为*listing12-12.py*，并通过运行 pgzrun listing12-12.py 来测试它。你能追踪到空间站中所有三种飞行危险吗？

### **添加有毒泄漏**

你可能已经注意到图 12-4 中地板上的绿色污点。这是有毒的泄漏物，当你走在上面时会消耗你的能量。你需要进行战略思考。你应该冲过去以更快到达某个地方吗？还是应该小心绕过它，节省能量以便后续使用，尽管这样可能会让你变慢？

列表 12-13 展示了当你在有毒地面上行走时，如何耗尽你的能量的指令。这些指令需要添加到`game_loop()`函数中，紧接在你在列表 12-12 中修复的指令之后。

将你的程序保存为*listing12-13.py*。你可以通过运行 pgzrun listing12-13.py 并在有毒地面上行走来测试它是否有效。有毒地面是对象 48，并作为景物放置在房间中。

*listing12-13.py*

```py
--snip--
  # If the player is standing somewhere they shouldn't, move them back.
    if room_map[player_y][player_x] not in items_player_may_stand_on \
               or hazard_map[player_y][player_x] != 0:
        player_x = old_player_x
        player_y = old_player_y
        player_frame = 0

    if room_map[player_y][player_x] == 48: # toxic floor
        deplete_energy(1)

    if player_direction == "right" and player_frame > 0:
        player_offset_x = -1 + (0.25 * player_frame)
--snip--
```

*列表 12-13：玩家在有毒地面上行走时减少能量*

### **完成最后的修整**

游戏现在差不多完成了。在你开始探索空间站之前，我们需要移除一些在构建和测试游戏时使用的指令。

#### **禁用传送器**

任务规则禁止在太空站工作开始后使用传送器。找到`game_loop()`函数中的相关指令，使用鼠标高亮它们，然后点击**格式** ▸ **注释掉区域**来禁用这些指令。你的代码现在应该像清单 12-14 那样。

*listing12-14.py*

```py
--snip--
#### Teleporter for testing
#### Remove this section for the real game
##    if keyboard.x:
##        current_room = int(input("Enter room number:"))
##        player_x = 2
##        player_y = 2
##        generate_map()
##        start_room()
##        sounds.teleport.play()
#### Teleport section ends
--snip--
```

*清单 12-14：传送器已关闭。*

#### **清理数据**

在测试游戏时，你可能已经修改了某些变量和列表的内容。游戏开始时应显示像图 12-5 中的样子。如果没有，请查看程序中的`VARIABLES`部分，确保`current_room`变量设置为 31。

![image](img/fig12-5.jpg)

*图 12-5：你的任务开始了*

如果你携带的不只是悠悠球，请查看程序中的`PROPS`部分，并检查这行代码是否正确：

```py
in_my_pockets = [55]
```

#### **你的冒险开始了**

这是一个激动人心的时刻：你的训练已经完成；太空站已经准备就绪；你的火星任务即将开始。让我们设置一个科幻风格的开场音乐，在游戏开始时播放。清单 12-15 展示了你将添加到*Escape*中的最终指令。

*listing12-15.py*

```py
--snip--
clock.schedule_unique(alarm, 10)
clock.schedule_interval(air_countdown, 11) # A higher number gives a longer
time limit.
sounds.mission.play() # Intro music
```

*清单 12-15：游戏开始时播放科幻风格的开场音乐。*

将最终程序保存为*escape.py*。现在，你可以使用 pgzrun escape.py 来玩游戏。有关说明，请参见《玩游戏》第 11 页。

恭喜你完成了太空站的建设。你确实已经赢得了在这个任务中的一席之地。现在，是时候开始你在星球表面的工作了！

### **你的下一次任务：自定义游戏**

你成功逃脱《逃脱》游戏了吗？真是惊险万分！在下一次任务中，尝试自定义游戏。使用本书的方式有很多种，所以你可能在构建游戏时已经做了一些自定义。以下是一些修改游戏的建议，从最简单的开始：

+   将游戏中的角色名字改为你朋友的名字。请参阅《清单 4-1》第 63 页和第四章。

+   自定义图像。你可以编辑我们的图像，或者创建自己的图像。游戏中包括一张白板图像，你可以使用自己喜欢的艺术软件进行编辑。如果你制作的图像与我们的尺寸相同、使用相同的文件名，并将其存储在*images*文件夹中，它们应该可以直接放入游戏世界中而不会有任何问题。

+   重新设计房间布局。第六章解释了如何在房间中布置景物。

+   向游戏中添加你自己的物品。首先创建它们的图像。道具应该是 30 像素见方。景物项可以更大，并且应当接触其瓷砖空间的左右两侧，以便当玩家无法更靠近景物时，不会显得不自然（例如，如果你的图像宽度为 30、60 或 90 像素，并且两侧都接触地面，它应该看起来没问题）。你需要将新物品添加到 `objects` 字典中（请参见第五章）。有关景物定位的帮助，请参见第六章。有关道具定位的建议，请参见第九章。

+   创建你自己的太空站地图（请参见第四章）。

+   使用游戏引擎制作你自己的游戏。你可以替换图像和地图，并编写你自己的谜题，基于 *Escape* 代码制作新游戏。`USE OBJECTS` 部分是游戏谜题的编程区域。它详细描述了物品单独使用或与其他物品组合时发生的情况。保存合成物品的代码（配方）并进行更新可能会很有用（请参见第十章）；保存标准响应的显示代码（请参见第十章）；以及保存开门的代码（请参见第十一章）。

如果你对第 26 房间进行任何更改，你需要禁用其压力垫的代码（请参见第十一章）。

请记住，你所做的任何更改可能会破坏原版 *Escape* 游戏中的谜题，使其无法完成。例如，可能会变得无法找到重要工具。我建议你将所有更改另存为不同的文件，这样你就可以随时回到原始代码。

**分享你的自定义修改**

我很想了解你的自定义修改！你可以在 Twitter 上找到我，用户名是 @musicandwords，或者访问我的网站 *[www.sean.co.uk](http://www.sean.co.uk)*，该网站包括本书的附加内容。如果你将修改后的 *Escape* 游戏与他人分享，或者分享你使用其代码、声音或图像构建的游戏，请注明本书及其作者，并明确表示你已修改了代码。谢谢！

### **你适合飞行吗？**

勾选以下选项框以确认你已掌握本章的关键内容。

![图片](img/box.jpg)  你可以使用 Pygame Zero 来绘制带有阴影的文本，并可以调整显示的文本大小。

![图片](img/box.jpg)  你可以通过在 `sounds.sound_name.play()` 指令中的括号里写入播放次数来多次播放声音。

![图片](img/box.jpg)  移动障碍物的方向从顶部开始编号为 1，按顺时针方向移动。要为障碍物创建一个移动模式，提供你希望在碰到物体时增加的方向编号。

![图片](img/box.jpg)  `deplete_energy()` 函数会减少玩家的能量。

![图片](img/box.jpg) 危险物体使用自己的房间地图，称为`hazard_map`。这使得它们能够更轻松地在地面上的物体之间移动。

![图片](img/box.jpg) 在开始游戏之前，请检查起始变量是否正确。

![图片](img/f0216-01.jpg)
