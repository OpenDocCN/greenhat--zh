## 让自己变得有用**

![image](img/common01.jpg)

你已经将道具添加到游戏中，所以在本章中，你将添加代码来使宇航员能够 *使用* 物品并将它们组合成新的物品。这些技能将对你的任务至关重要。你将有机会进行演练，以便在任何情况下都能做好准备。

本章的代码比你最近看到的一些列表更简单，并且包含了 *Escape* 游戏中许多谜题的答案。为了避免剧透过多，我不会在这里解释每个项目和解决方案。例如，有时你可能会在代码中看到一个物品编号，但我不会告诉你该物品的名称。

如果你在玩游戏时遇到卡住的地方，可以阅读这段代码，并通过参考 `objects` 字典（参见 列表 5-6 和 列表 5-8）来帮助辨认物品。虽然这应该是最后的办法，但通过像宇航员一样思考，你是可以解决所有难题的。问问自己：你能接触到哪些对你有用的东西？你如何让某物变得更有用？

### **添加用于使用物品的键盘控制**

我们将从在 `game_loop()` 函数中添加键盘控制开始。打开 *listing9-10.py*，这是你在 第九章中的最后一个列表。我们将在这个列表的基础上进行构建。

列表 10-1 显示了需要添加到 `game_loop()` 函数中的新指令。将它们添加到你在上一章中添加的 *drop* 和 *examine* 键盘控制之后。当玩家按下 U 键时，这些指令会启动 `use_object()` 函数。将程序保存为 *listing10-1.py*。不要尝试运行程序：它不会做任何新事情，但如果按下 U 键，程序会崩溃。

*listing10-1.py*

```py
--snip--

    if keyboard.space:
        examine_object()

    if keyboard.u:
        use_object()
--snip--
```

*列表 10-1：添加用于使用物品的键盘控制*

### **添加用于使用物品的标准消息**

使用物品的函数较长，因此我为它提供了一个单独的程序部分。将新的 `USE OBJECTS` 部分放在你在 第九章 中添加的 `PROP INTERACTIONS` 部分之后。 列表 10-2 显示了这个新部分的开始。在 `examine_object()` 函数结束后但在 `START` 部分之前添加这段代码。

*listing10-2.py*

```py
   --snip--
       show_text(description, 0)
       time.sleep(0.5)

   #################
   ## USE OBJECTS ##
   #################

   def use_object():
       global room_map, props, item_carrying, air, selected_item, energy
       global in_my_pockets, suit_stitched, air_fixed, game_over

➊     use_message = "You fiddle around with it but don't get anywhere."
➋     standard_responses = {
           4: "Air is running out! You can't take this lying down!",
           6: "This is no time to sit around!",
           7: "This is no time to sit around!",
           32: "It shakes and rumbles, but nothing else happens.",
           34: "Ah! That's better. Now wash your hands.",
           35: "You wash your hands and shake the water off.",
           37: "The test tubes smoke slightly as you shake them.",
           54: "You chew the gum. It's sticky like glue.",
           55: "The yoyo bounces up and down, slightly slower than on Earth",
           56: "It's a bit too fiddly. Can you thread it on something?",
           59: "You need to fix the leak before you can use the canister",
           61: "You try signalling with the mirror, but nobody can see you.",
           62: "Don't throw resources away. Things might come in handy...",
           67: "To enjoy yummy space food, just add water!",
           75: "You are at Sector: " + str(current_room) + " // X: " \
               + str(player_x) + " // Y: " + str(player_y)    
           }

       # Get object number at player's location.
➌     item_player_is_on = get_item_under_player()
➍     for this_item in [item_player_is_on, item_carrying]:
➎         if this_item in standard_responses:
➏             use_message = standard_responses[this_item]
➐     show_text(use_message, 0)
       time.sleep(0.5)

   ###############
   ##   START   ##
   ###############
   --snip--
```

*列表 10-2：添加用于使用物品的第一条指令*

列表 10-2 显示了 `use_object()` 函数的第一部分。我们将在本章的进一步列表中充实这个部分。在函数的末尾，程序会向玩家显示一条信息，告诉他们在尝试使用物品时发生了什么 ➐。该信息会保存在 `use_message` 变量中。在该函数的开始部分，我们将它设置为一个错误信息 ➊。稍后，如果玩家成功使用了物品，它将被更改为成功的消息。

一些对象在游戏中没有实际功能，但当玩家尝试使用它们时会奖励玩家一条信息。这些信息可能包括线索以及丰富游戏故事情节。字典`standard_responses`包含在玩家使用某些对象时显示的消息➋。字典的键是对象编号。例如，如果他们想使用床（懒骨头！），这是对象 4，他们会看到一条消息：“你不能躺着做这件事！”

变量`item_the_player_is_on`存储玩家所在房间中物体的位置编号➌。玩家可以使用他们携带或站在上的物体。我们设置了一个循环，遍历一个包含两个项目的列表：玩家站在上面的物体编号和玩家携带的物体编号➍。如果它们中的任何一个是`standard_responses`字典的键➎，则`use_message`将更新为该字典中该对象的消息➏。如果两者都具有标准消息，程序会优先使用玩家携带的物品。

将文件保存为*listing10-2.py*。使用`pgzrun listing10-2.py`运行它。为了测试它是否正常工作，按 U 键使用你携带的悠悠球。

### **添加游戏进度变量**

我们需要在程序中添加一些新变量，以存储关于玩家在游戏中进度的重要数据：

+   `air`，存储你拥有的空气量，按百分比表示

+   `energy`，存储你的能量，按百分比表示，并在你受伤时减少

+   `suit_stitched`，根据套装是否已修复，存储`True`或`False`值

+   `air_fixed`，根据气罐是否已修复，存储`True`或`False`值

将这些变量添加到`VARIABLES`部分的末尾，如列表 10-3 所示。将更新后的程序保存为*listing10-3.py*。运行时程序不会做任何新事情：我们设置了一些变量，但暂时没有对它们进行任何操作。

*listing10-3.py*

```py
--snip--
GREEN = (0, 255, 0)
RED = (128, 0, 0)

air, energy = 100, 100
suit_stitched, air_fixed = False, False
launch_frame = 0

###############
##    MAP    ##
###############
--snip--
```

*列表 10-3：添加游戏进度变量*

### **添加特定对象的操作**

`use_object()`函数的下一个阶段是检查特定对象，看看是否可以对其执行操作。这些检查将覆盖之前可能设置的任何标准消息，并显示在列表 10-4 中。因为这些指令位于`use_object()`函数内，所以它们的缩进至少为四个空格。将你的程序保存为*listing10-4.py*。使用`pgzrun listing10-4.py`运行它。

*listing10-4.py*

```py
   --snip--
           if this_item in standard_responses:
               use_message = standard_responses[this_item]

➊     if item_carrying == 70 or item_player_is_on == 70:
           use_message = "Banging tunes!"
           sounds.steelmusic.play(2)

➋     elif item_player_is_on == 11:
➌         use_message = "AIR: " + str(air) + \
                         "% / ENERGY " + str(energy) + "% / "
           if not suit_stitched:
               use_message += "*ALERT* SUIT FABRIC TORN / "
           if not air_fixed:
               use_message += "*ALERT* SUIT AIR BOTTLE MISSING"
           if suit_stitched and air_fixed:
               use_message += " SUIT OK"
           show_text(use_message, 0)
           sounds.say_status_report.play()
           time.sleep(0.5)
           # If "on" the computer, player intention is clearly status update.
           # Return to stop another object use accidentally overriding this.
➍         return

       elif item_carrying == 60 or item_player_is_on == 60:
➎         use_message = "You fix " + objects[60][3] + " to the suit"
           air_fixed = True
           air = 90
           air_countdown()
           remove_object(60)

       elif (item_carrying == 58 or item_player_is_on == 58) \
          and not suit_stitched:
           use_message = "You use " + objects[56][3] + \
                         " to repair the suit fabric"
           suit_stitched = True
           remove_object(58)

       elif item_carrying == 72 or item_player_is_on == 72:
           use_message = "You radio for help. A rescue ship is coming. \
   Rendezvous Sector 13, outside."
           props[40][0] = 13

       elif (item_carrying == 66 or item_player_is_on == 66) \
               and current_room in outdoor_rooms:
           use_message = "You dig..."
           if (current_room == LANDER_SECTOR
               and player_x == LANDER_X
               and player_y == LANDER_Y):
               add_object(71)
               use_message = "You found the Poodle lander!"

       elif item_player_is_on == 40:
           clock.unschedule(air_countdown)
           show_text("Congratulations, "+ PLAYER_NAME +"!", 0)
           show_text("Mission success! You have made it to safety.", 1)
           game_over = True
           sounds.take_off.play()
           game_completion_sequence()

       elif item_player_is_on == 16:
           energy += 1
           if energy > 100:
               energy = 100
           use_message = "You munch the lettuce and get a little energy back"
           draw_energy_air()

       elif item_carrying == 68 or item_player_is_on == 68:
           energy = 100
           use_message = "You use the food to restore your energy"
           remove_object(68)
           draw_energy_air()

       if suit_stitched and air_fixed: # open airlock access
           if current_room == 31 and props[20][0] == 31:
               open_door(20) # which includes removing the door
               sounds.say_airlock_open.play()
               show_text("The computer tells you the airlock is now open.", 1)
           elif props[20][0] == 31:
               props[20][0] = 0 # remove door from map
               sounds.say_airlock_open.play()
               show_text("The computer tells you the airlock is now open.", 1)

       show_text(use_message, 0)
       time.sleep(0.5)

   ###############
   ##   START   ##
   ###############
   --snip--
```

*列表 10-4：添加使用特定对象的功能*

列表 10-4 包括一系列指令，检查正在使用的对象是否是特定的对象编号。如果是，便执行该对象的操作。

例如，如果玩家携带或站在物品 70 ➊上，它是一个 MP3 播放器，玩家会看到一条消息“Banging tunes!”并听到一些音乐。如果玩家正在使用计算机➋，则显示的消息是通过组合`air`和`energy`变量中的信息生成的，并在西装或气瓶出现故障时添加警报。这里还有一个计算机语音音效，说“状态报告！”

我在这组指令的结尾添加了`return`指令➍，防止玩家在本意使用计算机时意外使用其他物品。如果没有包含这个`return`指令，玩家可能会使用他们携带的其他道具，而不是计算机。保持控制简洁意味着玩家可能会对他们想要使用的物品产生一些歧义，但游戏的设计是优先考虑那些帮助玩家完成游戏的结果。

在几个地方，我使用了`objects`字典中的简短描述，而不是直接将物品名称输入到字符串中➎。这样做是为了防止你在代码中看到任何剧透！

行尾的`\`符号➌告诉 Python 代码将在下一行继续。有些行比较长，因此我使用这个符号将它们分开，以便它们能够适应书页。

尝试通过走到其中一个计算机终端并按下 U 键来测试一些新代码。你将看到状态更新。如果你能找到 MP3 播放器，你也可以听听看。

**红色警报**

*当你输入物品编号和清单 10-4 中的其他代码时，要特别小心。如果你在这里犯了错误，可能无法完成游戏中的谜题！*

### **物品组合**

游戏中的一些谜题要求你将物品一起使用。例如，你可能会用一个物品作为工具对另一个物品进行操作，或者将两个物品组合在一起。例如，其中一个谜题要求你将 GPS 模块插入定位系统。当你找到这两个部件时，你需要将它们组合成一个工作中的定位系统。要将两个物品一起使用，你需要在你的背包中选择一个，并走到或走进另一个物品。你可能需要将物品从背包中丢到地上，这样你才能用你携带的其他物品来操作它。

在*Escape*游戏引擎中，组合物品被称为*配方*。一个配方包含三个物品编号，按顺序列出。前两个是被组合的物品，而第三个是它们组合后产生的物品编号。这里有一个例子：

```py
[73, 74, 75]
```

物品 73（GPS 模块）加上物品 74（定位系统）组合成物品 75（一个工作中的定位系统）。

当你组合物品时，新物品会进入你的背包。如果组合的物品是道具，它们将从游戏中移除。有时其中一个物品是场景道具，因此仍会保留在游戏中。

清单 10-5 展示了食谱列表。将其添加到程序中`PROPS`部分的末尾，在那里设置道具的信息。将文件保存为*listing10-5.py*。如果你运行该清单，它目前不会做任何新操作，但它会检查新数据是否正确。

*listing10-5.py*

```py
--snip--
in_my_pockets = [55]
selected_item = 0 # the first item
item_carrying = in_my_pockets[selected_item]

RECIPES = [
    [62, 35, 63], [76, 28, 77], [78, 38, 54], [73, 74, 75],
    [59, 54, 60], [77, 55, 56], [56, 57, 58], [71, 65, 72],
    [88, 58, 89], [89, 60, 90], [67, 35, 68]
    ]

checksum = 0
check_counter = 1
for recipe in RECIPES:
    checksum += (recipe[0] * check_counter
                 + recipe[1] * (check_counter + 1)
                 + recipe[2] * (check_counter + 2))
    check_counter += 3
print(len(RECIPES), "recipes")
assert len(RECIPES) == 11, "Expected 11 recipes"
assert checksum == 37296, "Error in recipes data"
print("Recipe checksum:", checksum)

#######################
## PROP INTERACTIONS ##
#######################
--snip--
```

*清单 10-5：为*逃脱*游戏添加食谱*

现在，在`use_object()`函数的末尾添加代码来使用食谱，如清单 10-6 所示。将其添加到你的`use_object()`函数中，并将程序保存为*listing10-5.py*。当你运行程序时，使用 pgzrun listing10-5.py，你将能够合并物品。

*listing10-6.py*

```py
   --snip--
               sounds.say_airlock_open.play()
               show_text("The computer tells you the airlock is now open.", 1)

➊     for recipe in RECIPES:
➋         ingredient1 = recipe[0]
           ingredient2 = recipe[1]
           combination = recipe[2]
➌         if (item_carrying == ingredient1
               and item_player_is_on == ingredient2) \
➍             or (item_carrying == ingredient2
                   and item_player_is_on == ingredient1):
➎             use_message = "You combine " + objects[ingredient1][3] \
                             + " and " + objects[ingredient2][3] \
                             + " to make " + objects[combination][3]
➏             if item_player_is_on in props.keys():
➐                 props[item_player_is_on][0] = 0
➑                 room_map[player_y][player_x] = get_floor_type()
➒             in_my_pockets.remove(item_carrying)
➓             add_object(combination)
               sounds.combine.play()

       show_text(use_message, 0)
       time.sleep(0.5)
   --snip--
```

*清单 10-6：在游戏中合并物品*

你可能会发现你可以理解这段新代码：它主要是将你之前见过的想法结合起来。我们使用一个循环来遍历`RECIPES`列表中的所有物品 ➊，每次都会将一个新的食谱添加到`recipe`列表中。我们将配料和合成物品的编号存入变量中，以便更容易理解函数 ➋。

程序会检查玩家是否携带了第一个配料并站在第二个配料上 ➌，或者反过来 ➍。如果是这样，使用消息将更新，告诉玩家他们合成了什么以及做出了什么 ➎。

当合成物品制作完成时，它通常会替换原来的配料物品。然而，如果其中一个物品是景物而不是道具，它将保留在游戏中。所以程序会检查玩家所站的物品是否为道具 ➏，如果是，它的房间号将被设置为 0，从游戏中移除 ➐。如果是道具，它还会从当前房间的房间地图中删除 ➑。

玩家携带的物品会从玩家的背包中移除 ➒，而新创建的物品会被添加到背包中 ➓。

**训练任务 #1**

让我们做一个简单的测试来检查合成代码是否正常工作。我们将需要对代码做一些修改来进行这个测试。在`PROPS`部分，修改设置`in_my_pockets`的那一行，以便你携带了物品 73 和 74：

```py
in_my_pockets = [55, 73, 74]
```

现在运行程序：你将携带 GPS 模块和定位系统。丢掉其中一个并站在上面。选择背包中的另一个物品，然后按 U 键。物品应该合成一个工作中的 GPS 系统！你可以用它查看游戏中的位置。为了确保代码正常工作，试着交换物品，这次站在另一个物品上。

确保之后将代码改回：

```py
in_my_pockets = [55]
```

### **添加游戏完成序列**

程序的`USE OBJECTS`部分还有一个最终功能，这是当玩家完成游戏时播放的一个简短动画：宇航员乘坐救援船起飞。将此功能添加到`USE OBJECTS`部分的末尾，如清单 10-7 所示：

*listing10-7.py*

```py
--snip--
    show_text(use_message, 0)
    time.sleep(0.5)

def game_completion_sequence():
    global launch_frame #(initial value is 0, set up in VARIABLES section)
    box = Rect((0, 150), (800, 600))
    screen.draw.filled_rect(box, (128, 0, 0))
    box = Rect ((0, top_left_y - 30), (800, 390))
    screen.surface.set_clip(box)

    for y in range(0, 13):
        for x in range(0, 13):
            draw_image(images.soil, y, x)

    launch_frame += 1
    if launch_frame < 9:
        draw_image(images.rescue_ship, 8 - launch_frame, 6)
        draw_shadow(images.rescue_ship_shadow, 8 + launch_frame, 6)
        clock.schedule(game_completion_sequence, 0.25)
    else:
        screen.surface.set_clip(None)
        screen.draw.text("MISSION", (200, 380), color = "white",
                     fontsize = 128, shadow = (1, 1), scolor = "black")
        screen.draw.text("COMPLETE", (145, 480), color = "white",
                     fontsize = 128, shadow = (1, 1), scolor = "black")
        sounds.completion.play()
        sounds.say_mission_complete.play()

###############
##   START   ##
###############
--snip--
```

*清单 10-7：发射！*

### **探索物品**

现在你可以探索在太空站中找到的物品，并尝试使用它们来看看它们的作用。不过，在你找到所有道具并开始工作之前，你需要先解决如何打开那些封闭太空站各个部分的安全门。在下一章中，你将通过设计门机制来完成太空站的设置，确保在使用正确的通行证时门会打开。

你还可以运用本章学到的知识，在*Escape*游戏代码中添加你自己的谜题。最简单的方法是使用标准消息（Listing 10-2）作为线索，并使用配方（Listing 10-5）来组合物品。你还可以添加简单的指令（Listing 10-4）来检查玩家是否携带某个特定物品，然后增加他们的`air`或`energy`变量，显示消息，或在游戏中执行其他操作。祝你冒险愉快！

### **你准备好飞行了吗？**

勾选以下框以确认你已经学会了本章的关键内容。

![图片](img/box.jpg)  使用物品的说明进入`use_object()`函数。

![图片](img/box.jpg)  `standard_responses`字典包含当玩家使用特定物品时的消息。

![图片](img/box.jpg)  对于许多物品，当玩家使用它们时，会有特定的说明来更新不同的列表或变量。

![图片](img/box.jpg)  `RECIPES`列表储存了玩家如何在游戏中组合物品的详细信息。

![图片](img/box.jpg)  在一个配方中，前两个项目是原料，第三个项目是它们所制成的物品。
