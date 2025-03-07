## **10

让自己有用**

![image](../images/common01.jpg)

你已经将道具添加到游戏中，因此在本章中，你将添加代码，使宇航员能够*使用*物品并将它们组合成新的物品。这些技能对你的任务至关重要。你将有机会进行练习，确保自己准备好应对任何情况。

本章中的代码比你最近看到的一些代码简单，并且包含了*逃脱*游戏中许多谜题的答案。为了避免剧透，我不会在这里解释每一个项目和解决方案。例如，有时你可能会看到代码中的一个物品编号，但我不会告诉你那个物品的名称。

如果你在玩游戏时遇到困难，你可以查看这段代码，并通过参考物品字典（见[Listing 5-6](ch05.xhtml#ch05list6)和[Listing 5-8](ch05.xhtml#ch05list8)在[第5章](ch05.xhtml#ch05)中）来弄清楚哪些是哪些物品。不过，这应该是最后的手段。你可以通过像宇航员一样思考来解决所有的谜题。问问自己：你能使用什么物品？如何才能让某样东西更有用？

### **添加使用物品的键盘控制**

我们将从在 game_loop() 函数中添加键盘控制开始。打开 *listing9-10.py*，这是你在[第9章](ch09.xhtml#ch09)中的最后一个示例。我们将在这个示例的基础上进行扩展。

Listing 10-1 显示了要添加到 game_loop() 函数中的新指令。将它们添加到你在上一章中为*丢弃*和*检查*添加的键盘控制之后。这些指令会在玩家按下 U 键时启动 use_object() 函数。将程序保存为 *listing10-1.py*。不要急着运行程序：它不会做任何新操作，但如果你按下 U 键，它会崩溃。

*listing10-1.py*

--snip--

if keyboard.space:

examine_object()

if keyboard.u:

use_object()

--snip--

*Listing 10-1: 为使用物品添加键盘控制*

### **添加使用物品的标准消息**

使用物品的函数比较长，因此我将它放在程序中的一个独立部分。将新的 USE OBJECTS 部分放在你在[第9章](ch09.xhtml#ch09)中添加的 PROP INTERACTIONS 部分之后。[Listing 10-2](ch10.xhtml#ch10list2) 显示了这一新部分的开头。将这段代码添加到 examine_object() 函数结束后，但在 START 部分之前。

*listing10-2.py*

--snip--

show_text(description, 0)

time.sleep(0.5)

#################

## 使用物品 ##

#################

def use_object():

global room_map, props, item_carrying, air, selected_item, energy

global in_my_pockets, suit_stitched, air_fixed, game_over

➊     use_message = "你摆弄了一下它，但没有任何进展。"

➋     standard_responses = {

4: "空气快没了！你不能坐视不管！",

6: "现在可不是坐着不动的时候！",

7: "现在可不是坐着不动的时候！",

32: "它晃动并发出轰鸣声，但没有其他反应。",

34: "啊！好多了。现在洗手吧。",

35: "你洗了手，并把水抖掉。",

37: "你摇动试管时，它们稍微冒烟。",

54: "你嚼了口香糖。它像胶水一样粘。",

55: "悠悠球上下弹跳，比地球上稍慢"

56: "有点太麻烦了。你能把它穿过某个东西吗？",

59: "在使用罐子之前，你需要修复漏水",

61: "你试着用镜子打信号，但没有人能看到你。",

62: "不要浪费资源。以后可能会派上用场...",

67: "要享受美味的太空食物，只需加水！",

75: "你位于区域: " + str(current_room) + " // X: " \

+ str(player_x) + " // Y: " + str(player_y)

}

# 获取玩家位置上的物品编号。

➌     item_player_is_on = get_item_under_player()

➍     for this_item in [item_player_is_on, item_carrying]:

➎         if this_item in standard_responses:

➏             use_message = standard_responses[this_item]

➐     show_text(use_message, 0)

time.sleep(0.5)

###############

##   开始   ##

###############

--snip--

*Listing 10-2: 添加使用物品的第一个指令*

[Listing 10-2](ch10.xhtml#ch10list2) 显示了 use_object() 函数的第一部分。我们将在本章的后续列表中进一步完善它。在函数的最后，程序会向玩家展示一条消息，告诉他们尝试使用物品时发生了什么 ➐。该消息将存储在 use_message 变量中。在函数开始时，我们将其设置为错误消息 ➊。稍后，如果玩家成功使用物品，它会变成一条成功消息。

游戏中的一些物品实际上没有实际功能，但当玩家尝试使用它们时，会通过消息奖励玩家。这些消息可能包括线索，也可能会增加游戏的故事性。字典 standard_responses 存储了玩家使用特定物品时需要显示的消息 ➋。字典的键是物品编号。例如，如果玩家想要使用床（懒骨头！），它是物品 4，他们会看到一条消息：“你不能躺着做这件事！”

变量 item_the_player_is_on 存储了玩家在房间 ➌ 中的位置的物品编号。玩家可以使用他们携带的或站在上的物品。我们设置了一个循环，遍历一个包含两个物品的列表：玩家站立的物品编号和玩家携带的物品编号 ➍。如果其中任何一个是 standard_responses 字典的键 ➎，那么 use_message 会更新为该物品在字典中的消息 ➏。如果两者都有标准消息，程序会优先使用你携带的物品。

将文件保存为 *listing10-2.py*。使用 `pgzrun listing10-2.py` 运行它。要测试它是否正常工作，按 U 使用你正在携带的悠悠球。

### **添加游戏进度变量**

我们需要向程序添加一些新的变量，用于存储关于玩家在游戏中进展的重要数据：

+   air，表示你剩余的空气量，按百分比显示

+   energy, 表示玩家的能量，以百分比表示，如果受伤会减少

+   suit_stitched, 表示宇航服是否已修复，存储 True 或 False 的值

+   air_fixed，表示气瓶是否已修复，存储 True 或 False 的值

将变量添加到 VARIABLES 部分的末尾，如[列表 10-3](ch10.xhtml#ch10list3)所示。将更新后的程序保存为 *listing10-3.py*。如果运行它，程序不会做任何新事情：我们设置了一些变量，但还没有用它们做任何事情。

*listing10-3.py*

--snip--

GREEN = (0, 255, 0)

RED = (128, 0, 0)

air, energy = 100, 100

suit_stitched, air_fixed = False, False

launch_frame = 0

###############

##    地图    ##

###############

--snip--

*列表 10-3：添加游戏进度变量*

### **添加特定对象的操作**

use_object() 函数中的下一个阶段是检查特定对象，看看是否可以用它们执行某些操作。这些检查会覆盖任何之前可能已设置的标准信息，并显示在[列表 10-4](ch10.xhtml#ch10list4)中。因为这些指令位于 use_object() 函数内部，所以它们至少缩进四个空格。保存你的程序为 *listing10-4.py*。使用 pgzrun listing10-4.py 运行它。

*listing10-4.py*

--snip--

if this_item in standard_responses:

use_message = standard_responses[this_item]

➊     if item_carrying == 70 or item_player_is_on == 70:

use_message = "节奏感十足！"

sounds.steelmusic.play(2)

➋     elif item_player_is_on == 11:

➌         use_message = "空气： " + str(air) + \

"% / 能量 " + str(energy) + "% / "

if not suit_stitched:

use_message += "*警告* 宇航服面料撕裂 / "

if not air_fixed:

use_message += "*警告* 宇航服气瓶丢失"

if suit_stitched and air_fixed:

use_message += " 宇航服正常"

show_text(use_message, 0)

sounds.say_status_report.play()

time.sleep(0.5)

# 如果“在”电脑上，玩家的意图显然是更新状态。

# 返回以防止另一个物体的使用意外覆盖此操作。

➍         return

elif item_carrying == 60 or item_player_is_on == 60:

➎         use_message = "你将 " + objects[60][3] + " 固定到宇航服上"

air_fixed = True

air = 90

air_countdown()

remove_object(60)

elif (item_carrying == 58 or item_player_is_on == 58) \

and not suit_stitched:

use_message = "你使用了 " + objects[56][3] + \

" 修理宇航服面料"

suit_stitched = True

remove_object(58)

elif item_carrying == 72 or item_player_is_on == 72:

use_message = "你发出无线电求救，救援船正在赶来。"

Rendezvous Sector 13，外面。"

props[40][0] = 13

elif (item_carrying == 66 or item_player_is_on == 66) \

and current_room in outdoor_rooms:

use_message = "你在挖掘..."

if (current_room == LANDER_SECTOR

and player_x == LANDER_X

and player_y == LANDER_Y):

add_object(71)

use_message = "你找到了贵宾犬着陆器！"

elif item_player_is_on == 40:

clock.unschedule(air_countdown)

show_text("恭喜你，"+ PLAYER_NAME +"!", 0)

show_text("任务成功！你已安全抵达。", 1)

game_over = True

sounds.take_off.play()

game_completion_sequence()

elif item_player_is_on == 16:

energy += 1

if energy > 100:

energy = 100

use_message = "你吃了生菜，恢复了一点能量"

draw_energy_air()

elif item_carrying == 68 or item_player_is_on == 68:

energy = 100

use_message = "你使用食物恢复了能量"

remove_object(68)

draw_energy_air()

if suit_stitched and air_fixed: # 打开气闸

if current_room == 31 and props[20][0] == 31:

open_door(20) # 这包括移除门

sounds.say_airlock_open.play()

show_text("计算机告诉你气闸现在已打开。", 1)

elif props[20][0] == 31:

props[20][0] = 0 # 从地图中移除门

sounds.say_airlock_open.play()

show_text("计算机告诉你气闸现在已打开。", 1)

show_text(use_message, 0)

time.sleep(0.5)

###############

##   开始   ##

###############

--省略--

*清单 10-4：添加使用某些物体的功能*

[清单 10-4](ch10.xhtml#ch10list4) 包含一系列指令，用于检查正在使用的物体是否是特定的物品编号。如果是，这些物体的指令将被执行。

例如，如果玩家携带或站在物品 70 ➊ 上，那个是一个 MP3 播放器，玩家会看到“炸裂的音乐！”并听到一些音乐。如果玩家正在使用计算机 ➋，则显示的消息是通过组合空气和能量变量的信息并加入警告（如果太空服或空气瓶有问题）制作的。这里还有一个计算机语音效果，内容是“状态报告！”

我在这组指令的末尾加入了一个返回指令➍，这样可以防止玩家在打算使用计算机时，误操作其他物品。如果我们不加入这个返回指令，玩家可能会使用他们携带的其他道具，而不是计算机。保持控制简单意味着有时玩家意图使用的物品可能会有些模糊，但游戏的设计是优先考虑帮助玩家完成任务的结果。

在某些地方，我使用了物品字典中的简短描述，而不是将物品名称直接写入字符串 ➎。这样可以避免你在代码中看到剧透！

行末的 \ 符号 ➌ 告诉 Python 代码将在下一行继续。一些行比较长，所以我使用了这个符号来将它们拆开，以便适应书页。

尝试运行一些新代码，走到一个计算机终端并按下 U 键。你会看到状态更新。如果你能找到 MP3 播放器，你也可以听听它。

**红色警报**

*特别小心在 [清单 10-4](ch10.xhtml#ch10list4) 中输入物品编号和其余代码。如果你在这里犯错，可能无法完成游戏中的谜题！*

### **合并物品**

游戏中的一些谜题需要你将物体组合使用。例如，你可能会使用一个物体作为工具，对另一个物体进行操作，或者将两个物体合并在一起。例如，其中一个谜题要求你将GPS模块插入定位系统。当你找到这两个部分时，你需要将它们组合成一个可用的定位系统。要一起使用两个物体，你需要在背包中选择一个，然后走到另一个物体上或进入它。有时你需要将一个物体从背包中丢到地面上，这样你才能与另一个你携带的物体一起操作。

在*Escape*游戏引擎中，物体组合被称为*食谱*。一个食谱包含一个列表中的三个物体编号。前两个是被组合的物品，第三个是它们合并后的物体编号。以下是一个示例：

[73, 74, 75]

物体 73（一个GPS模块）加上物体 74（一个定位系统）会生成物体 75（一个工作定位系统）。

当你组合物体时，新的物体会进入你的背包。如果它们是道具，已组合的物品将从游戏中移除。有时其中一个可能是景物，这样它就会留在游戏中。

[Listing 10-5](ch10.xhtml#ch10list5)展示了食谱的列表。将其添加到程序中PROPS部分的末尾，那里是道具信息的设置位置。将文件保存为*listing10-5.py*。如果你运行该列表，它不会做任何新的操作，但它会检查新数据是否正确。

*listing10-5.py*

--省略--

in_my_pockets = [55]

selected_item = 0 # 第一个物品

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

print(len(RECIPES), "个食谱")

assert len(RECIPES) == 11, "预期有 11 个食谱"

assert checksum == 37296, "食谱数据错误"

print("食谱校验和:", checksum)

#######################

## PROP INTERACTIONS ##

#######################

--省略--

*Listing 10-5: 将食谱添加到* Escape *游戏中*

现在，在`use_object()`函数的末尾添加代码来使用食谱，如[列表 10-6](ch10.xhtml#ch10list6)所示。将其添加到你的`use_object()`函数中，并将程序保存为*listing10-5.py*。当你运行程序时，使用`pgzrun listing10-5.py`，你将能够合并物体。

*listing10-6.py*

--省略--

sounds.say_airlock_open.play()

show_text("计算机告诉你空气锁现在已打开。", 1)

➊     for recipe in RECIPES:

➋         ingredient1 = recipe[0]

ingredient2 = recipe[1]

combination = recipe[2]

➌         if (item_carrying == ingredient1

and item_player_is_on == ingredient2) \

➍             or (item_carrying == ingredient2

and item_player_is_on == ingredient1):

➎             use_message = "你组合了 " + objects[ingredient1][3] \

+ " 和 " + objects[ingredient2][3] \

+ " 以制作 " + objects[combination][3]

➏             if item_player_is_on in props.keys():

➐                 props[item_player_is_on][0] = 0

➑                 room_map[player_y][player_x] = get_floor_type()

➒             in_my_pockets.remove(item_carrying)

➓             add_object(combination)

sounds.combine.play()

show_text(use_message, 0)

time.sleep(0.5)

--snip--

*示例 10-6：在游戏中组合物品*

你可能会发现，你可以理解这段新代码的逻辑：它大部分结合了你之前见过的概念。我们使用一个循环来遍历RECIPES列表中的所有物品 ➊，每次都有一个新的食谱加入食谱列表。我们将原料和组合物体的编号存入变量，以使函数更易理解 ➋。

程序检查玩家是否携带着第一个原料并站在第二个原料上 ➌，或者反过来 ➍。如果是，使用消息会更新，告诉他们组合了什么物品，以及他们做了什么 ➎。

当组合物体被创建时，它通常会替换原料物体。然而，如果其中一个物体是景物而不是道具，它会继续保留在游戏中。所以程序会检查玩家所站的物体是否是道具 ➏，如果是，它的房间号会被设置为0，从游戏中移除它 ➐。如果它是道具，它还会从当前房间的房间地图中删除 ➑。

被携带的物体从玩家的物品栏中移除 ➒，新创建的物体会被添加到物品栏中 ➓。

**训练任务 #1**

我们来做一个简单的测试，检查组合代码是否有效。为了进行这个测试，我们需要稍微修改一下代码。在PROPS部分，将设置in_my_pockets的那一行改成使你携带物品73和74：

in_my_pockets = [55, 73, 74]

现在运行程序：你将携带GPS模块和定位系统。丢弃其中一个，站在它上面。选择背包中的另一个物品，然后按下U键。物品应该被组合成一个有效的GPS系统！你可以用它来查看你在游戏中的位置。为了确认代码是否正常工作，尝试交换物品，让自己这次站在另一个物体上。

确保在之后将代码改回去：

in_my_pockets = [55]

### **添加游戏完成序列**

在程序的USE OBJECTS部分有一个最终函数，这是一个短动画，在玩家完成游戏时播放：宇航员乘坐救援飞船起飞。将此函数添加到USE OBJECTS部分的末尾，如[示例 10-7](ch10.xhtml#ch10list7)所示：

*listing10-7.py*

--snip--

show_text(use_message, 0)

time.sleep(0.5)

def game_completion_sequence():

global launch_frame #(初始值为0，在VARIABLES部分设置)

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

screen.draw.text("任务", (200, 380), color = "white",

fontsize = 128, shadow = (1, 1), scolor = "black")

screen.draw.text("完成", (145, 480), color = "white",

fontsize = 128, shadow = (1, 1), scolor = "black")

sounds.completion.play()

sounds.say_mission_complete.play()

###############

##   开始   ##

###############

--snip--

*清单 10-7: 发射！*

### **探索物品**

现在，你可以探索在空间站中找到的物品，并尝试使用它们看它们有什么作用。不过，在你找到所有道具并开始在空间站工作之前，你需要想办法打开封闭空间站某些部分的安全门。在下一章中，你将通过设计门的机制，在使用正确的通行证时打开这些安全门，从而完成空间站的设置。

你还可以运用本章所学的知识，向 *逃脱* 游戏代码中添加你自己的谜题。最简单的方法是使用标准消息（[清单 10-2](ch10.xhtml#ch10list2)）作为线索，并使用食谱（[清单 10-5](ch10.xhtml#ch10list5)）来组合物品。你还可以添加简单的指令（[清单 10-4](ch10.xhtml#ch10list4)），查看玩家是否携带某个特定物品，然后增加他们的空气或能量变量，显示一条消息，或者在游戏中执行其他操作。祝你冒险愉快！

### **你准备好飞行了吗？**

勾选以下框以确认你已经掌握本章的关键知识点。

![图片](../images/box.jpg) 使用物品的说明被写入 use_object() 函数中。

![图片](../images/box.jpg) standard_responses 字典包含当玩家使用特定物品时的消息。

![图片](../images/box.jpg) 对于许多物品，当玩家使用它们时，会有特定的说明来更新不同的列表或变量。

![图片](../images/box.jpg) RECIPES 列表存储了玩家在游戏中如何组合物品的详细信息。

![图片](../images/box.jpg) 在食谱中，前两个项目是原料，第三个项目是它们做成的东西。
