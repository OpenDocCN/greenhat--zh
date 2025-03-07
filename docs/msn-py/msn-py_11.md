## **11

**激活安全门**

![image](img/common01.jpg)

在太空站中，门限制了对某些区域的访问，确保宇航员只能进入他们有资格工作的区域。许多门需要个人通行证才能打开，而工程舱的门只能通过任务控制中的按钮打开。工程舱的门还设置了定时器，自动关闭它们以提高安全性。

门还执行安全规则，要求宇航员在进入气闸前必须穿着有效的宇航服，并且在打开通往行星表面的门之前，必须有一名伙伴陪伴。监控录像显示，一些宇航员已经找到了绕过伙伴要求的方法，这样他们就能享受在行星表面独自漫步的宁静。

你在安装道具时就已经在太空站安装了门。在这一章中，你将添加打开和关闭门的代码，并且添加一些其他的技巧和谜题，使游戏更加有趣。

### **规划安全门的位置**

门显然是太空站设计中的一个重要组成部分，但它们对游戏设计也至关重要。最明显的是，它们呈现了一个具有挑战性的谜题：玩家需要找到打开上锁门的方法。

门还帮助我们讲述一个故事，其中有障碍，英雄必须利用他们的生存训练和逻辑思维来克服这些障碍。只有当玩家需要稍微思考一下时，游戏的谜题才会让人满意。因此，能够控制玩家何时看到不同的谜题元素非常重要。想象一下，你进入一个房间，另一侧的出口被肆虐的火焰阻挡。如果你已经携带了灭火器，你只需把它拿出来使用，根本没有什么挑战。如果你先看到威胁（或谜题），然后才需要想办法解决，那就更有趣了。通过封锁地图的一些区域，我们可以引导玩家先看到问题，再看到它的解决方案。我们不能确定他们会注意到我们为他们设置的每一个障碍，但我们可以给他们一个机会，体验到游戏的最佳状态。

门还使我们能够从地图中获得更多的价值。尽管输入地图后可能感觉不到，游戏地图其实并不大。通过要求玩家多次穿越困难的房间，我们可以提供更丰富的体验和更长的游戏时间。例如，如果我们把一把钥匙放在走廊的尽头，我们可以引导玩家沿着走廊倒回去，使用他们在途中经过的门上的钥匙。

图 11-1 显示了游戏中门的位置。在不剧透太多的前提下，玩家在进入空间站的右上部分（通过房间 34）之前是无法进入房间 36 的。在进入房间 40 之前，他们也无法访问房间 27。通过在锁定的房间中战略性地放置物品，包括访问卡，我们可以引导玩家通过游戏和故事的发展。

![image](img/fig11-1.jpg)

*图 11-1：显示红色门的游戏地图*

当你设计自己的游戏时，仔细考虑物品的放置位置非常重要。这是确保游戏能为玩家提供一个有趣挑战的关键元素之一。

### **门的位置**

我将*逃脱*游戏中的所有门都放在房间的顶部或底部出口，这是因为游戏采用的是自上而下的视角。如果门位于侧面出口，玩家只能看到门的顶部表面，我们需要确保像门这样重要的元素能被清晰地看到。

大多数门位于房间的顶部，并在玩家打开后保持打开状态。唯一的例外是房间 32 和房间 27 之间的门，它有一个定时器机制，会自动关闭。这种定时器提供了额外的挑战：玩家必须迅速从打开门的开关处赶到房间，才能在门关闭之前进入。

*逃脱*中的门是对象 20 到 26。它们的图像和描述在 objects 字典中设置（参见 “制作空间站对象字典” 第 85 页）。门的位置在 props 字典中设置（参见 “添加道具信息” 第 151 页）。每个门都有一个 *x* 位置，将其放置在房间的门口。要计算门的 *x* 位置，只需将房间宽度除以 2，向下取整，再减去 1。

现在让我们添加一些控制，让玩家能够打开门。

### **添加访问控制**

为了让玩家能够打开门，我们需要在程序的 USE OBJECTS 部分的 use_object() 函数中添加一些指令。一段新的代码片段将在玩家按下某个房间的按钮时打开通往工程舱的定时门。你将把这段代码添加到处理对象 16 和 68 的指令之间。

另一段新的代码将使玩家能够使用访问卡打开门：把它放在使用配方的代码后面。

清单 11-1 显示了需要添加的新代码。因为这些指令是 use_object() 函数的一部分，所以第一个指令缩进了四个空格。你的新 elif 指令应该与上面的 elif 指令对齐。

打开*listing10-7.py*，并将这些新行添加到其中。将程序保存为 *listing11-1.py*。你可以通过 pgzrun listing11-1.py 来运行它，但我们还没有添加所有必要的代码来使门正常工作。不过，你应该不会看到任何错误信息。

*listing11-1.py*

--snip--

elif item_player_is_on == 16:

energy += 1

如果 energy > 100:

energy = 100

use_message = "你吃掉了生菜，恢复了一些能量"

draw_energy_air()

➊      elif item_player_is_on == 42:

➋          如果 current_room == 27:

➌              open_door(26)

➍          props[25][0] = 0 # 从 RM32 到工程舱的门

props[26][0] = 0 # 工程舱内的门

➎          clock.schedule_unique(shut_engineering_door, 60)

use_message = "你按下了按钮"

show_text("工程舱的门已经打开 60 秒", 1)

sounds.say_doors_open.play()

sounds.doors.play()

elif item_carrying == 68 或 item_player_is_on == 68:

energy = 100

use_message = "你使用食物恢复了能量"

remove_object(68)

draw_energy_air()

--snip--

for recipe in RECIPES:

ingredient1 = recipe[0]

ingredient2 = recipe[1]

--snip--

add_object(combination)

sounds.combine.play()

# {钥匙物品编号: 门物品编号}

➏      ACCESS_DICTIONARY = { 79:22, 80:23, 81:24 }

➐      如果 item_carrying 在 ACCESS_DICTIONARY 中:

door_number = ACCESS_DICTIONARY[item_carrying]

➑          如果 props[door_number][0] == current_room:

use_message = "你解锁了门！"

➒              sounds.say_doors_open.play()

sounds.doors.play()

open_door(door_number)

show_text(use_message, 0)

time.sleep(0.5)

--snip--

*Listing 11-1：添加打开门的功能*

打开通往工程舱的门的按钮是物品 42。工程舱外面有一个这样的按钮用于提供访问权限，工程舱内也有一个按钮，以确保玩家不会被困在里面。

如果玩家正在使用按钮 ➊，则运行打开门的代码。如果他们正在使用房间内的按钮 ➋，则使用 open_door() 函数显示门打开 ➌。我们会很快添加这个函数。

props 字典被更新，以将门的房间号改为 0，从而将门移除房间（并从游戏中移除） ➍。这扇门是定时工作的，因此程序安排了一个函数，在 60 秒后关闭门 ➎。如果你觉得很难及时到达房间，你可以将数字 60 改为更大的数字。这个数字应该足够让你有足够的时间，无论你是在使用 PC 还是 Raspberry Pi 3，或者是 Raspberry Pi 2，那时游戏会运行得稍微慢一些。

第二段代码使玩家能够使用钥匙打开门。我们创建了一个新的字典，叫做 ACCESS_DICTIONARY，它使用访问卡号作为字典键，门号作为数据➏。例如，物品 79（访问卡）用于打开门 22。

**提示**

在 *Escape* 中，用于打开门的物品都是门禁卡，但如果你修改游戏，可以使用任何物品。你可以用撬棍撬开门，或者（如果你做的是一款奇幻世界设定的游戏）可以使用不同的魔法咒语。只要确保玩家能合理地推断出该使用什么物品。

当玩家按下 U 键时，如果他们已选择字典中用于解锁门的某个物品 ➐，并且站在与门解锁位置相同的房间内 ➑，门就会打开。同时，我们会播放一段计算机语音效果，播报“门已打开” ➒。这只是一个录音，就像游戏中的其他任何声音一样。

### **制作门的开关动画**

我们将把开门、关门和动画制作的函数放入程序的新 DOORS 部分。你需要在 USE OBJECTS 部分之后、START 部分之前添加这个新部分。

Listing 11-2 展示了你需要添加的前两个函数，以启动 DOORS 部分。添加新行并将程序保存为 *listing11-2.py*。DOORS 部分仍然不完整：你可以运行程序（使用 pgzrun listing11-2.py）来检查错误，但门的功能尚未启用。

*listing11-2.py*

--snip--

sounds.completion.play()

sounds.say_mission_complete.play()

###############

##   DOORS   ##

###############

➊ def open_door(opening_door_number):

global door_frames, door_shadow_frames

global door_frame_number, door_object_number

➋     door_frames = [images.door1, images.door2, images.door3,

images.door4, images.floor]

# （最终帧恢复阴影，为门重新出现做准备）。

door_shadow_frames = [images.door1_shadow, images.door2_shadow,

images.door3_shadow, images.door4_shadow,

images.door_shadow]

door_frame_number = 0

door_object_number = opening_door_number

➌     do_door_animation()

➍ def close_door(closing_door_number):

global door_frames, door_shadow_frames

global door_frame_number, door_object_number, player_y

➎     door_frames = [images.door4, images.door3, images.door2,

images.door1, images.door]

door_shadow_frames = [images.door4_shadow, images.door3_shadow,

images.door2_shadow, images.door1_shadow,

images.door_shadow]

door_frame_number = 0

door_object_number = closing_door_number

# 如果玩家与门处于同一行，则必须处于开门口

➏     如果 player_y == props[door_object_number][1]:

➐         如果 player_y == 0: # 如果在上方门口

➑             player_y = 1 # 将玩家向下移动

else:

➒             player_y = room_height - 2 # 将玩家向上移动

➓     do_door_animation()

###############

##   START   ##

###############

--snip--

*Listing 11-2: 设置门的动画*

open_door() 和 close_door() 函数设置了开门和关门的动画效果。你已经在 Listing 11-1 中见过 open_door() ➊ 的描述。在 Listing 11-2 中，我们定义了这个函数，以便它在玩家使用钥匙开门时运行。

门的动画有五帧，编号从 0 到 4，如表 11-1 所示。我们将动画的图像存储在名为 door_frames ➋➎的列表中，并将帧编号存储在变量 door_frame_number 中。在 open_door()和 close_door()函数中，我们将帧编号设置为 0，即第一帧。

在变量 door_object_number 中，我们存储将要开关的门的对象编号。在变量和列表设置完成后，启动 do_door_animation()函数来执行动画 ➌➓。我们稍后会添加这个函数。

关闭门的函数➍与开门函数➊类似，有两个不同点：动画帧不同，并且有检查来防止门关闭时覆盖玩家。

如果玩家与门处于相同的*y*位置 ➏，则意味着玩家站在门口。在这种情况下，如果玩家在最上面一行 ➐，我们将他们的*y*位置设置为 1 ➑，将他们移到下一行。如果玩家不在最上面一行，我们将他们的*y*位置设置为倒数第二行 ➒，也就是门的内部。

这意味着宇航员会主动跳开门，更加逼真，而不是直接站在门里！

**表 11-1：** 门的动画帧

| **帧编号** | **0** | **1** | **2** | **3** | **4** |
| --- | --- | --- | --- | --- | --- |
| **开门** | ![image](img/f0189-01.jpg) | ![image](img/f0189-02.jpg) | ![image](img/f0189-03.jpg) | ![image](img/f0189-04.jpg) | **最后一帧是地板砖（没有门）。** |
| **关门** | ![image](img/f0189-05.jpg) | ![image](img/f0189-06.jpg) | ![image](img/f0189-07.jpg) | ![image](img/f0189-08.jpg) | ![image](img/f0189-09.jpg) |

### **添加门动画**

do_door_animation()函数将管理门的开关动画。

将 do_door_animation()函数放在程序的 DOORS 部分，在你在清单 11-2 中添加的 close_door()函数之后。添加清单 11-3 中的新行，并将程序保存为*listing11-3.py*。你可以使用 pgzrun listing11-3.py 运行这个版本的游戏。现在，使用钥匙打开的门应该可以正常工作。稍后我会告诉你如何在训练任务#1 中测试它们。

*listing11-3.py*

--snip--

player_y = room_height - 2 # 将他们移动到上面

do_door_animation()

def do_door_animation():

global door_frames, door_frame_number, door_object_number, objects

➊     objects[door_object_number][0] = door_frames[door_frame_number]

objects[door_object_number][1] = door_shadow_frames[door_frame_number]

➋     door_frame_number += 1

➌     如果 door_frame_number == 5:

➍         如果 door_frames[-1] == images.floor:

➎             props[door_object_number][0] = 0 # 从道具列表中移除门

# 从道具重新生成房间地图

# 如果需要将门放入房间

➏          generate_map()

➐      else:

➑          clock.schedule(do_door_animation, 0.15)

###############

##   开始   ##

###############

--snip--

*Listing 11-3: 添加门的动画*

对象字典包含了用于特定对象的图像等内容。这个新函数首先通过将字典中的门图像更改为当前的动画帧 ➊。重新绘制房间时，它将使用该动画帧。

然后该函数将动画帧数加 1 ➋，以便下一次运行此函数时显示下一个动画帧。如果帧数现在是 5，说明我们已经到了动画的最后一帧 ➌。在这种情况下，我们检查门是否已打开（而不是关闭），通过检查最后一帧是否为地板砖，显示没有门 ➍。（-1 的索引值表示列表中的最后一个项。）

如果门现在已经完全打开，props 数据将被更新，通过将其房间编号更改为 0 来移除这扇门 ➎。如果当前的动画帧是最后一帧，无论门是打开还是关闭，都会生成一个新的房间地图 ➏，以确保门在当前房间内正确添加或移除。

如果当前帧不是最后一个动画帧 ➐，函数会在 0.15 秒后重新运行 ➑，以显示序列中的下一帧。

你可能会想，为什么我没有将两个 if 语句 ➌➍ 合并在一起。原因是 generate_map()函数需要在动画结束时运行，无论门是打开还是关闭。如果我们将两个 if 语句合并，这个函数只会在门打开时运行。

**训练任务 #1**

在程序的这一部分，门应该已经完全正常工作了。您能测试它是否能正常工作吗？找到社区房间中门的通行卡并使用它。站在社区房间里，通过在物品栏中选择通行卡并按 U 来使用它。如果您需要提示，请查看图 11-1 中的地图。社区房间是编号 39，它的钥匙在 41 号房间。记住，有时人们会整理东西，钥匙可能不会显眼地放着。

### **关闭定时门**

接下来，我们需要添加一个名为 shut_engineering_door()的新函数，用于自动关闭通往工程舱的门。该函数设置为在门打开后延迟 60 秒运行（参见 Listing 11-1），给玩家一分钟的时间从按钮跑到门口，才会关闭门！

将此函数放入程序中的 DOORS 部分，位于您刚才添加的 do_door_animation()函数之后。在 Listing 11-4 中添加新行，并将程序保存为*listing11-4.py*。然后使用 pgzrun listing11-4.py 运行此程序。您应该不会看到任何错误消息。定时门现在应该可以正常工作了，但稍后我会向您展示一种更简单的测试方法。

*listing11-4.py*

--snip--

else:

clock.schedule(do_door_animation, 0.15)

def shut_engineering_door():

global current_room, door_room_number, props

➊     props[25][0] = 32 # 从 32 号房间到工程舱的门。

➋     props[26][0] = 27 # 工程舱内的门。

➌     generate_map() # 如果在受影响的房间中，向 room_map 中添加门。

➍     if current_room == 27:

➎         close_door(26)

➏     if current_room == 32:

➐         close_door(25)

show_text("计算机告诉你门已关闭。", 1)

sounds.say_doors_closed.play()

###############

##   开始   ##

###############

--snip--

*列表 11-4：自动关闭工程门的代码*

shut_engineering_door()函数有两个门的道具来处理，即物体 25 和 26，因为玩家可以根据他们所处的房间从任一侧看到这扇门。我们首先做的是更新道具字典，以便这些门出现在房间中➊➋。

然后我们调用 generate_map()函数➌。如果玩家在有这些门的房间中，这个函数会更新当前房间的房间地图。在其他情况下，generate_map()函数仍然会运行，但没有任何变化。

如果玩家在工程舱（27 号房间）中➍，他们需要看到 26 号门关闭➎，所以程序会启动动画。如果玩家在门的另一侧，处于 32 号房间➏，我们需要显示 25 号门关闭➐。

**红色警报**

*不要混淆门的编号和房间的编号。门编号是物体编号，与它所在的房间无关。*

为了测试工程舱门是否正常工作，我们必须运行游戏，按下按钮，并冲向工程舱。所以为了节省时间，让我们设计一个解决方案，使我们能够更快速地在太空站中移动。

### **添加一个传送门**

当你还在建造太空站时，能够瞬间跳转到任何房间会非常有帮助。利用最新的分子传输技术，我们可以安装一个*传送门*，让你输入房间号并直接传送过去。在测试游戏时，这对你来说是一个巨大的便利，但这项技术属于受限技术，在太空站的真实任务中不允许使用。你需要在完成游戏之前移除它。我将这项高度机密的技术交给你使用。

将传送门代码与游戏循环（game_loop()）中的其他玩家控制代码放在一起，位于程序的游戏循环部分。我建议你将其添加到开始使用物品（use_object）函数的指令后面。因为这些指令在函数内部，你需要将 if 指令缩进四个空格，然后将它下面的指令再缩进四个空格。

将新的指令添加到列表 11-5，然后将文件保存为*listing11-5.py*。你可以使用 pgzrun listing11-5.py 来运行这个程序。

*listing11-5.py*

--snip--

if keyboard.u:

use_object()

## 用于测试的传送门

## 对于真实的游戏，移除这一部分

➊     if keyboard.x:

➋         current_room = int(input("请输入房间号："))

➌         player_x = 2

player_y = 2

➍         generate_map()

➎         start_room()

sounds.teleport.play()

## 传送部分结束

--省略--

*清单 11-5：添加传送门*

当你按下 X 键 ➊ 时，程序会要求你输入一个房间号 ➋。此请求会显示在你输入 pgzrun 指令以运行程序的命令行窗口中。你可能需要点击该窗口将其置于前面，然后在之后需要点击游戏窗口重新开始游戏。

input() 函数会将你输入的内容作为字符串处理。由于我们需要的是数字输入，因此我们使用 int() 函数将其转换为整数（或整数型） ➋。

你输入的数字会被存入 current_room 变量。这里没有错误检查，所以如果你输入无效的房间号，程序可能会崩溃。例如，如果你输入的是文字而不是数字，程序会卡死。

你会被传送到房间内的 *y* = 2，*x* = 2 ➌ 位置。这个位置通常是一个相对安全的地方，但如果传送门将你传送到某个景物内，你通常可以直接走出去。房间地图会被重新生成 ➍，房间会重新启动 ➎，完成你到达新目的地的传送。

**训练任务 #2**

使用传送门传送到 27 号房间，以便测试工程舱的门。使用房间顶部的按钮来打开门（在走向按钮时按下 U），并在房间内等待直到门关闭。再次打开门，但这次离开房间并检查从另一侧看门是否仍然关闭。门的动画应该能正常工作。

### **激活气闸安全门**

作为一项安全功能，通往行星表面的气闸门使用压力传感器来开启。一名宇航员必须站在压力垫上才能打开门，从而允许另一个人通过。这种设计确保了宇航员在空间站内没有支持的情况下无法进入行星表面。

为了启用这一安全功能，我们需要在程序的 DOORS 部分添加一个新函数。清单 11-6 显示了新函数的代码，该函数用于动画化门的开关。将此代码添加到你在清单 11-4 中添加的 shut_engineering_door() 函数之后。将更新后的程序保存为 *listing11-6.py*。你可以使用 pgzrun listing11-6.py 来运行程序，但气闸门尚未被激活。

*listing11-6.py*

--省略--

show_text("电脑告诉你门已关闭。", 1)

sounds.say_doors_closed.play()

def door_in_room_26():

global airlock_door_frame, room_map

➊     frames = [images.door, images.door1, images.door2,

images.door3, images.door4, images.floor

]

shadow_frames = [images.door_shadow, images.door1_shadow,

images.door2_shadow, images.door3_shadow,

images.door4_shadow, None]

➋     if current_room != 26:

clock.unschedule(door_in_room_26)

return

# 属性 21 是房间 26 中的门。

➌     if ((player_y == 8 and player_x == 2) or props[63] == [26, 8, 2]) \

且 props[21][0] == 26:

➍         airlock_door_frame += 1

➎         if airlock_door_frame == 5:

props[21][0] = 0 # 当门完全打开时，将门从地图中移除。

room_map[0][1] = 0

room_map[0][2] = 0

room_map[0][3] = 0

➏     if ((player_y != 8 or player_x != 2) and props[63] != [26, 8, 2]) \

and airlock_door_frame > 0:

if airlock_door_frame == 5:

# 将门添加到属性和地图中，以便显示动画。

props[21][0] = 26

room_map[0][1] = 21

room_map[0][2] = 255

room_map[0][3] = 255

airlock_door_frame -= 1

➐     objects[21][0] = frames[airlock_door_frame]

objects[21][1] = shadow_frames[airlock_door_frame]

###############

##   开始   ##

###############

--snip--

*列表 11-6：在气闸中添加压力感应门*

我已将 `door_in_room_26()` 函数添加到游戏中，以启用一个特定的谜题。为了避免透露解决方案而破坏谜题的乐趣，我不会在此详细讲解代码中的所有内容，但我相信如果你愿意，自己一定能搞明白！

我们将门的动画帧存储在 `frames` 列表中，包括展示门关闭的第一帧和展示没有门的地板瓷砖的最后一帧 ➊。

我们将气闸门的动画帧存储在 `airlock_door_frame` 变量中。如果玩家站在压力垫上（位置 *y* = 8，*x* = 2），并且门已经在房间 ➌ 中，动画帧的数字会增加，使门继续打开 ➍。如果动画帧现在是 5 ➎，那么门就完全打开了，属性字典和房间地图将更新以将门从房间中移除。

我们添加了另一段代码，当玩家 *没有* 站在压力垫上且门已经至少部分打开时 ➏，门会关闭，因此当玩家离开压力垫时，门会关闭。程序只显示当前房间地图中存在的物品，所以前面的指令会将门（物品 21）放入房间地图中，尽管第一帧的动画会显示门完全打开。

最后，我们将 `objects` 字典中门的图片文件更改为当前的动画帧 ➐。门的阴影图像也会更新。因此，当房间被绘制时，门的图片会显示其当前的动画帧。

这个气闸例程创建了一种平滑效果，当玩家站在压力垫上时，门会滑开，而当玩家走开时门会再次滑闭。如果玩家在门关闭时重新站回垫子上，门会重新打开。

为了使气闸例程正常工作，我们还需要添加指令，使得 door_in_room_26()函数每 0.05 秒运行一次，当玩家进入房间时。如果玩家离开房间，Listing 11-6 中➋处的指令将停止该函数的定期运行并退出该函数（使用 return 指令），这样门的动画就会停止。

我们将把启动 door_in_room_26()函数的代码放入 GAME LOOP 部分顶部的 start_room()函数中。start_room()函数在玩家进入房间时运行。Listing 11-7 显示了需要添加的新指令。

*listing11-7.py*

--省略--

###############

## 游戏循环 ##

###############

def start_room():

global airlock_door_frame

show_text("你在这里：" + room_name, 0)

if current_room == 26: # 自关闭气闸门的房间

airlock_door_frame = 0

clock.schedule_interval(door_in_room_26, 0.05)

--省略--

*Listing 11-7：为气闸调度门动画*

将程序保存为*listing11-7.py*并使用 pgzrun listing11-7.py 运行。在游戏中，按 X 键使用传送器并传送到 26 号房间。现在你可以测试压力垫是否按预期工作（参见图 11-2）。尝试走上它、离开它和穿过它，看看门的行为如何变化。

请注意，如果你从这个房间底部的出口离开，门会出现并阻挡你重新进入的路。（通常，你只会通过打开那扇门并将其从游戏中移除来进入气闸。）当你传送到房间时，像这样的奇怪事情可能会发生。这会干扰时空连续性。

![图片](img/fig11-2.jpg)

*图 11-2：站在压力垫上打开门。*

### **为你自己的游戏设计移除出口**

如果你正在为自己的地图设计封闭出口，你可能还需要移动或移除这些出口中的门。要从游戏中移除门，请更改道具字典中该门的条目，使其第一个数字为 0，或者从字典中删除该条目。

如果你正在定制游戏，你可能还需要移除一些自定义代码，这些代码启用了工程舱和气闸的特殊门。要禁用压力垫门，请移除 Listings 11-6 和 11-7 中的新代码。要移除通往工程舱的定时门，请移除 Listing 11-4 中显示的代码，并额外移除 Listing 11-1 中按下按钮（使用对象 42）的第一段新代码。

### **任务完成了吗？**

你现在已经完成了空间站的建造，并且它已经完全投入使用。看来你现在可以开始你的新生活，进行实验并探索红色星球。

但是，等等！这是什么？可能有麻烦在前方。

### **你准备好飞行了吗？**

勾选以下选项框以确认你已经学习了本章的关键内容。

![Images](img/box.jpg)  门可以封锁游戏地图的某些部分，让玩家按正确的顺序发现解谜元素。

![Images](img/box.jpg)  门需要放置在房间的顶部或底部。

![Images](img/box.jpg)  用访问卡打开的门将保持开启状态。

![Images](img/box.jpg)  你可以使用提供的函数添加自动关闭的门，比如工程区的门。

![Images](img/box.jpg)  门的位置通过 props 字典进行设置。它们的图片和描述存储在 objects 字典中。

![Images](img/box.jpg)  为了给门添加动画效果，程序会在 objects 字典中更改其图片。当房间被重绘时，会使用新的图片显示门。

![Images](img/box.jpg)  如果一扇门从两侧都能看到，则需要用两个门道具来表示：每个房间中都放置一个门道具。

![Images](img/box.jpg)  ACCESS_DICTIONARY 用于记住哪些访问卡解锁哪些门。你可以通过修改这个字典，使用其他对象来打开门。

![Images](img/box.jpg)  为了调整游戏的难度，你可以改变工程门关闭前的延迟时间。

![Images](img/box.jpg)  传送门允许你传送到任何房间进行测试。

![Images](img/box.jpg)  Python 中的 input()函数将你输入的内容当作字符串处理。为了让玩家输入数字，可以使用 int()函数将输入的内容转换为整数。
