## **7

**进入空间站**

![image](../images/common01.jpg)

现在，我们已经为空间站配备了景观、生命支持系统和其他设备，是时候开始移动了。在这一章中，你将第一次看到自己在空间站的样子，并且你将能够四处移动并探索房间。刚开始你可能会感觉有些僵硬，但很快你就能在基地四处走动了。

你将学习如何为宇航员添加动画，并使用键盘控制让他们四处移动。你还将添加代码，使宇航员能够在房间之间移动。火星上有生命吗？现在有了。

### **到达空间站**

在本章中，我们将以[第6-6节](ch06.xhtml#ch06list6)中的代码作为起点，参考[第6章](ch06.xhtml#ch06)，因此请打开*listing6-6.py*。我们将添加代码，让你在空间站的太空服中显示自己。最终，你将能够使用箭头键移动。

#### **禁用探险者区的房间导航控制**

到目前为止，我们一直在使用EXPLORER区的箭头键来显示地图上的不同房间。接下来，我们将开始使用这些按键来让宇航员在房间之间移动。首先，我们需要禁用现有的控制。向下滚动到程序中的EXPLORER部分，突出显示[Listing 7-1](ch07.xhtml#ch07list1)中显示的指令。点击**格式** ▸ **注释区域**，将这些指令变成注释，以便程序忽略它们。（如果你愿意，也可以直接删除它们。）将程序保存为*listing7-1.py*。

*listing7-1.py*

--snip--

##def movement():

##    global current_room

##    old_room = current_room

##

##    if keyboard.left:

##        current_room -= 1

##    if keyboard.right:

##        current_room += 1

##    if keyboard.up:

##        current_room -= MAP_WIDTH

##    if keyboard.down:

##        current_room += MAP_WIDTH

##

##    if current_room > 50:

##        current_room = 50

##    if current_room < 1:

##        current_room = 1

##

##    if current_room != old_room:

##        print("进入房间：" + str(current_room))

##

##clock.schedule_interval(movement, 0.08)

--snip--

*Listing 7-1: 关闭* EXPLORER *区的键盘控制*

现在我们可以添加代码，使用箭头键来移动宇航员了。

#### **添加新变量**

让我们先设置一些变量。最重要的变量是你开始传送时的坐标。和之前一样，我们将变量添加到程序的VARIABLES部分，靠近开始的地方。请按照[Listing 7-2](ch07.xhtml#ch07list2)中所示添加新行。将程序保存为*listing7-2.py*。

*listing7-2.py*

--snip--

TILE_SIZE = 30

➊ player_y, player_x = 2, 5

➋ game_over = False

➌ PLAYER = {

"left": [images.spacesuit_left, images.spacesuit_left_1,

images.spacesuit_left_2, images.spacesuit_left_3,

images.spacesuit_left_4

],

"right": [images.spacesuit_right, images.spacesuit_right_1,

images.spacesuit_right_2, images.spacesuit_right_3,

images.spacesuit_right_4

],

"up": [images.spacesuit_back, images.spacesuit_back_1,

images.spacesuit_back_2, images.spacesuit_back_3,

images.spacesuit_back_4

],

"down": [images.spacesuit_front, images.spacesuit_front_1,

images.spacesuit_front_2, images.spacesuit_front_3,

images.spacesuit_front_4

]

}

➍ player_direction = "down"

➎ player_frame = 0

➏ player_image = PLAYER[player_direction][player_frame]

player_offset_x, player_offset_y = 0, 0

--snip--

*列表 7-2：添加玩家变量*

VARIABLES 部分已经包括了当前房间（current_room）的值，即你将开始的房间。（如果你在[第6章](ch06.xhtml#ch06)实验时更改了 current_room 的值，请确保将其恢复为31。）我们创建了新的 player_y 和 player_x 变量 ➊，用来保存你在房间中的起始位置坐标。在这里，我们在一行代码中设置了两个变量。数字按顺序放入变量中，因此2放入 player_y（第一个数字放入第一个变量），5放入 player_x。随着你在空间站中移动，这些变量会发生变化，并用于检查你的位置以及在正确的位置绘制你。你的位置是使用与景物位置相同的瓦片坐标来衡量的。

我们还设置了一个 game_over 变量 ➋，用来告诉程序游戏是否结束。程序开始时，我们将该变量设为 False。直到游戏结束，它才会变为 True。程序通过检查这个变量来判断玩家是否允许继续移动。如果玩家在死后还在继续移动，那就太奇怪了！

接下来，我们将设置玩家走动动画的图像。动画是一种视觉技巧。你从一系列相似的图片开始，每张图像有微小的差异，显示出小的运动。当你迅速切换它们时，可以欺骗眼睛，让它以为图像在动。在我们的游戏中，我们将使用一系列宇航员走动的图像，显示出不同位置的腿部。当我们快速切换它们时，宇航员的腿部看起来像是在移动。

**提示**

动画能够生效的关键是确保图像足够相似。如果图像差异太大，效果就不会起作用。

动画中的每一帧图像称为 *帧*。[表7-1](ch07.xhtml#ch07tab1)展示了我们将使用的动画帧。我们将从0开始编号我们的帧，这时宇航员处于静止状态，当宇航员在屏幕上方走动时，我们会看到他们的背部，因为他们背对我们在房间中走动。

**表7-1：** 宇航员的动画帧

| **键** | **帧 0** | **帧 1** | **帧 2** | **帧 3** | **帧 4** |
| --- | --- | --- | --- | --- | --- |
| left | ![image](../images/f0114-01.jpg) | ![image](../images/f0114-02.jpg) | ![image](../images/f0114-03.jpg) | ![image](../images/f0114-04.jpg) | ![image](../images/f0114-05.jpg) |
| 向右 | ![image](../images/f0114-06.jpg) | ![image](../images/f0114-07.jpg) | ![image](../images/f0114-08.jpg) | ![image](../images/f0114-09.jpg) | ![image](../images/f0114-10.jpg) |
| 向上 | ![image](../images/f0114-11.jpg) | ![image](../images/f0114-12.jpg) | ![image](../images/f0114-13.jpg) | ![image](../images/f0114-14.jpg) | ![image](../images/f0114-15.jpg) |
| 向下 | ![image](../images/f0114-16.jpg) | ![image](../images/f0114-17.jpg) | ![image](../images/f0114-18.jpg) | ![image](../images/f0114-19.jpg) | ![image](../images/f0114-20.jpg) |

PLAYER 字典 ➌ 存储动画帧。方向名称——上、下、左、右——是字典的键。每个字典项都是一个列表，包含玩家站立时的图像，以及该方向的四个动画帧（见 [表 7-1](ch07.xhtml#ch07tab1)）。PLAYER 字典将与玩家面朝的方向 ➍ 和动画帧的编号 ➎ 一起使用，以显示玩家走路或站立时的正确图像。player_image 变量 ➏ 存储当前的宇航员图像。

**提示**

[附录 B](appb.xhtml#appb) 在书的末尾描述了 *Escape* 程序中的重要变量，如果你记不住某个变量的功能，可以查看那里。

#### **传送到太空站**

准备好传送下来！在确定起始坐标后，让我们添加代码，使你出现在太空站中。

[Listing 7-3](ch07.xhtml#ch07list3) 显示了你需要添加到程序 EXPLORER 部分的代码行。如前所述，你只需要添加新的代码行。不要修改其他行，只用它们来帮助你理解程序代码。第一行新代码 ➊ 被缩进了八个空格，因为它位于一个函数内部且也在一个循环内。将你的程序保存为 *listing7-3.py*。

*listing7-3.py*

--snip--

for y in range(room_height):

for x in range(room_width):

if room_map[y][x] != 255:

image_to_draw = objects[room_map[y][x]][0]

screen.blit(image_to_draw,

(top_left_x + (x*30),

top_left_y + (y*30) - image_to_draw.get_height()))

➊         if player_y == y:

➋             image_to_draw = PLAYER[player_direction][player_frame]

➌             screen.blit(image_to_draw,

(top_left_x + (player_x*30)+(player_offset_x*30),

top_left_y + (player_y*30)+(player_offset_y*30)

- image_to_draw.get_height()))

--snip--

*Listing 7-3: 绘制房间中的玩家*

这些新指令将在房间中绘制你。y 循环从后向前绘制房间。x 循环则从左到右绘制每行的景象。

每绘制一行后，程序检查玩家是否站在该行 ➊。这个指令应该与 for x in range(room_width) 这一行对齐，而不是进一步缩进，因为它不在 x 循环内。它将在 x 循环完成后执行一次。

如果玩家 *确实* 在程序刚刚绘制的行中，下一行 ➋ 会将玩家的图片放入 image_to_draw 变量中。该图片来自 PLAYER 字典中的动画帧，使用玩家的方向和动画帧编号。

最后一行新代码 ➌ 使用你刚刚设置的 image_to_draw 变量绘制玩家，这个变量包含了图片。它还使用了玩家的 *x* 和 *y* 位置变量来计算在屏幕上绘制图片的位置。[第3章](ch03.xhtml#ch03) 解释了屏幕位置是如何计算的（参见 “[计算每个物品绘制位置](ch03.xhtml#lev58)” 在 [第56页](ch03.xhtml#page_56)）。player_offset_x 和 player_offset_y 变量是在 [Listing 7-2](ch07.xhtml#ch07list2) 中设置的，用于在玩家走过瓷砖时将玩家位置放置在瓷砖之间。你将很快了解这些变量。

准备传送！做好准备！深呼吸。

使用 `pgzrun listing7-3.py` 运行你的程序。如果你的传送成功，你应该在空间站上（参见[图7-1](ch07.xhtml#ch07fig1)）。如果没有，检查你在本章中所做的程序更改。

传送的一个副作用是，刚开始时你不能移动。随着代码的添加，你会发现这个副作用会消失。

![image](../images/fig7-1.jpg)

*图7-1：宇航员到达！*

### **添加移动代码**

现在我们将添加一个全新的部分，称为 GAME LOOP。这是程序的核心。game_loop() 函数将每秒运行多次，使你能够移动。稍后在书中，我们将添加更多指令，使你能够与找到的物品进行互动。

在 MAKE MAP 和 EXPLORER 部分之间添加这一新部分。[Listing 7-4](ch07.xhtml#ch07list4) 显示了它的样子。将程序保存为 *listing7-4.py*。

*listing7-4.py*

--省略--

for tile_number in range(1, image_width_in_tiles):

room_map[scenery_y][scenery_x + tile_number] = 255

###############

## 游戏循环 ##

###############

➊ def game_loop():

➋     全局变量 player_x, player_y, current_room

全局变量 from_player_x, from_player_y

全局变量 player_image, player_image_shadow

全局变量 selected_item, item_carrying, energy

全局变量 player_offset_x, player_offset_y

全局变量 player_frame, player_direction

➌     如果 game_over:

返回

➍     如果 player_frame > 0:

player_frame += 1

time.sleep(0.05)

如果 player_frame == 5:

player_frame = 0

player_offset_x = 0

player_offset_y = 0

➎ # 保存玩家当前位置

old_player_x = player_x

old_player_y = player_y

➏ # 按键时移动

如果 player_frame == 0:

如果 keyboard.right:

from_player_x = player_x

from_player_y = player_y

player_x += 1

player_direction = "right"

player_frame = 1

elif keyboard.left: # elif 防止玩家做出对角移动

from_player_x = player_x

from_player_y = player_y

player_x -= 1

player_direction = "left"

player_frame = 1

elif keyboard.up:

from_player_x = player_x

from_player_y = player_y

player_y -= 1

player_direction = "up"

player_frame = 1

elif keyboard.down:

from_player_x = player_x

from_player_y = player_y

player_y += 1

player_direction = "down"

player_frame = 1

➐      # 如果玩家站在不该站的位置，将其移回。

# 保留以下两条注释 - 你稍后会用到它们

if room_map[player_y][player_x] not in items_player_may_stand_on: #\

#           or hazard_map[player_y][player_x] != 0:

player_x = old_player_x

player_y = old_player_y

➑          player_frame = 0

➒      if player_direction == "right" and player_frame > 0:

player_offset_x = -1 + (0.25 * player_frame)

if player_direction == "left" and player_frame > 0:

player_offset_x = 1 - (0.25 * player_frame)

if player_direction == "up" and player_frame > 0:

player_offset_y = 1 - (0.25 * player_frame)

if player_direction == "down" and player_frame > 0:

player_offset_y = -1 + (0.25 * player_frame)

###############

## EXPLORER  ##

###############

--snip--

*Listing 7-4: 添加玩家移动*

在程序的最后，你还需要添加一个名为 START 的新部分，它将使 game_loop() 函数每 0.03 秒运行一次。[Listing 7-5](ch07.xhtml#ch07list5) 显示了你需要添加的代码行。这个指令没有缩进，因为它不属于任何函数。Python 会按程序中出现的顺序从上到下运行那些*不*在函数内的指令。这个指令会在所有变量、地图、场景和道具数据设置好，函数在上面的指令中定义好之后执行。保存你的程序为 *listing7-5.py*。

*listing7-5.py*

--snip--

###############

##   START   ##

###############

clock.schedule_interval(game_loop, 0.03)

--snip--

*Listing 7-5: 设置 game_loop() 函数定期运行*

使用 pgzrun 运行程序 listing7-5.py。你应该在房间里（如[图 7-1](ch07.xhtml#ch07fig1)所示），并能够使用箭头键移动！你可能会注意到当你向上走时，腿部会消失。这是瞬移的副作用，我们将在[第 8 章](ch08.xhtml#ch08)改进绘制房间的代码时解决这个问题。

到此为止，如果你走出门外，程序不会正常工作，但它应该能阻止你穿过墙壁或家具。如果你*能*穿过物体，请仔细检查你刚刚添加的新代码。如果问题仍然存在，仔细检查程序中 OBJECTS 部分末尾设置 items_player_may_stand_on 列表的那一行。

### **理解运动代码**

如果你想玩这个游戏并用自己的设计进行自定义，你并不需要理解本章代码的工作原理。你只需要替换图像以及地图、景物和道具的数据。这段移动代码和稍后将在本章中添加的房间间移动代码应该能继续工作。不过，如果你想理解代码是如何工作的，并且希望看到如何将动画添加到你的程序中，我现在就来给你分解一下。这段代码才是游戏的真正引擎，所以从许多方面来看，它是最激动人心的部分！

如果你有种似曾相识的感觉，那是因为你已经看到过大部分的代码。在[第2章](ch02.xhtml#ch02)中，你使用了代码通过键盘控制来改变玩家的位置，并用一个名为game_loop()的函数来控制运动。让我们回顾一下，看看在[清单7-4](ch07.xhtml#ch07list4)中有什么新的内容。

在[清单7-4](ch07.xhtml#ch07list4)中，我们在这一新章节的开始定义了一个名为game_loop()的函数➊。我们在程序末尾添加的clock.schedule_interval()函数（见[清单7-5](ch07.xhtml#ch07list5)）使得game_loop()函数每0.03秒执行一次。每次game_loop()函数执行时，它都会检查你是否按下了箭头键或正在走动，如果是，它就会更新你的位置信息。

在game_loop()的开始部分，我们告诉Python哪些变量是全局变量➋（关于为何需要这样做，请参考[《理解太空漫步清单》](ch01.xhtml#lev30)，以及[第27页](ch01.xhtml#page_27)）。其中有些变量还没有被使用，但我们稍后会用到它们。

然后我们检查game_over变量。如果它被设置为True➌，game_loop()函数会在没有执行任何其他指令的情况下结束，因为游戏已经结束。这个变量会在游戏结束时阻止玩家继续移动。目前它不会有任何作用，因为我们的程序中还没有任何东西会导致游戏结束。

game_loop()函数会检查玩家是否已经在走路➍。玩家走一步需要四帧动画。如果玩家在移动，player_frame变量的值将介于1到4之间，表示当前使用的动画帧。如果玩家在走动，程序会将player_frame变量加1，跳到下一帧动画。这意味着EXPLORER部分的draw()函数将在下次运行时显示下一帧动画。

当player_frame达到5时，意味着所有的动画帧都已经显示完毕，动画也就结束了。此时，程序将player_frame重置为0以结束动画。当动画结束时，程序还会重置player_offset_x和player_offset_y变量。我稍后会告诉你这些变量的作用。

接下来，我们检查玩家是否按下了一个键来启动新的行走动画。在允许玩家移动之前，我们通过将 *x* 位置存储在 `old_player_x` 变量中、将 *y* 位置存储在 `old_player_y` 变量中，来保存玩家的当前位置 ➎。如果玩家试图走到不该去的地方（比如撞到墙柱），我们将使用这些变量将玩家移回。

然后，程序使用一段熟悉的代码块来改变玩家的 *x* 和 *y* 位置变量，如果按下了箭头键 ➏。我们以瓦片为单位来测量玩家的位置，这与我们在[第一章](ch01.xhtml#ch01)中使用像素作为度量单位不同。

当玩家按下右箭头键时，程序会将 *x* 位置加 1。如果玩家按下左箭头键，程序会将 *x* 位置减 1。我们使用类似的代码来改变 *y* 位置，当玩家按下上或下箭头键时。

当玩家移动时，`from_player_x` 和 `from_player_y` 这两个全局变量存储玩家起始位置。这些变量稍后将用来检查玩家在行走过程中是否被障碍物击中。`player_direction` 变量也会被设置为玩家的移动方向，而 `player_frame` 被设置为 1，这是动画序列中的第一帧。

如同在[第一章](ch01.xhtml#ch01)中一样，我们使用 `elif` 来组合检查按键的操作。这确保了玩家不能同时更改 *x* 和 *y* 位置进行对角线移动。在我们的 3D 房间中，对角线行走会让玩家穿过障碍物，挤过无法通过的缝隙。

在移动玩家后，我们会检查新位置是否是玩家可以站立的地方 ➐。我们通过使用 `room_map` 来查看玩家所站的位置处有什么物品，并将其与 `items_player_may_stand_on` 列表进行比对。这里有一些我已注释掉的代码，我们稍后需要启用它，以防止玩家穿过障碍物。

我们可以使用 `in` 关键字来检查某个元素是否在列表中。通过与 `not` 结合使用，我们可以检查某个元素是否缺失在列表中。以下代码的意思是：“如果玩家所在的地图位置的数字不在物品列表中，玩家被允许站在 ...”

if room_map[player_y][player_x] not in items_player_may_stand_on:

如果玩家站在的物品不在 `items_player_may_stand_on` 列表中，我们会将玩家的 *x* 和 *y* 位置重置为他们移动前的位置。

这一切发生得非常快，以至于玩家没有察觉。如果他们试图走进一堵墙，看起来就像他们根本没有走动！这种方式比在每次移动前检查是否允许移动更简单地阻止玩家穿墙。

如果玩家的位置需要重置，程序还会将 `player_frame` 变量设置为 0 ➑。这样就会关闭玩家动画。

当你按下右箭头键时，宇航员向右迈出一个瓷砖。动画需要四个帧来完成，因此在动画播放过程中，宇航员会显示在瓷砖的部分位置上。player_offset_x 和 player_offset_y 变量用于计算应该在哪里绘制宇航员。这些变量在 game_loop() 函数的末尾计算出来 ➒。draw() 函数（见[代码清单 7-3](ch07.xhtml#ch07list3)）将偏移量值乘以瓷砖的大小（30 像素），因为图像是以像素为单位绘制的。例如，如果偏移量为 0.25 瓷砖，宇航员会大约离新瓷砖的中心 7 个像素。计算机会四舍五入，因为你不能用半个像素来定位某物。

看看[图 7-2](ch07.xhtml#ch07fig2)的左侧。当宇航员向左走的第一个动画帧时，我们需要在玩家的新瓷砖位置上加上三分之四个瓷砖（0.75）。对于第二个动画帧，我们在绘制前将半个瓷砖（0.5）加到玩家的新瓷砖位置上。对于第三个动画帧，我们在玩家的新瓷砖位置上加上一四分之一瓷砖。

![image](../images/fig7-2.jpg)

*图 7-2：理解宇航员在动画中的位置*

我们可以使用帧编号来计算这些偏移量。下面是向左走的计算：

player_offset_x = 1 - (0.25 * player_frame)

通过自己计算这些数字，检查这个计算是否有道理。例如，下面是动画帧为 2 时的计算：

0.25 × 2 = 0.5

1 − 0.5 = 0.5

在[图 7-2](ch07.xhtml#ch07fig2)中，0.5 是帧 2 的正确偏移量。

当玩家向右走时，我们需要从玩家的位置中减去一部分瓷砖，因此偏移量是负值。看看[图 7-2](ch07.xhtml#ch07fig2)的右侧。对于帧 1，添加 −0.75 会把宇航员移到其新位置的*左侧*三分之四的瓷砖位置。

我们也可以使用帧编号来计算向右走的 *x* 偏移量。下面是公式：

player_offset_x = -1 + (0.25 * player_frame)

**训练任务 #1**

你能检查这个公式是否有效吗？用它来找到帧 1 和帧 3 的偏移量值，并检查它们是否与[图 7-2](ch07.xhtml#ch07fig2)中的偏移量值相匹配。

*y* 方向的偏移量计算方式相同。当宇航员向上移动时，我们使用与向左偏移量相同的公式计算 *y* 偏移量。当宇航员向下移动时，我们使用与向右偏移量相同的公式计算 *y* 偏移量。

总结一下，game_loop() 函数做了这些事情：

+   如果你没有走路，当你按下一个键时，它会开始走路动画。

+   如果你在走路，它会计算下一个动画帧以及在绘制你时使用的部分跨越瓷砖的位置。

+   如果你已经到达动画序列的末尾，它会重置以便你可以再次移动。移动是流畅的，因此如果你按住一个键，你会在动画帧 1 到 4 之间循环，直到你停止行走，才会看到站立姿势。

### **在房间之间移动**

现在你已经站起来了，想要充分探索空间站。让我们在 `game_loop()` 函数中添加一些代码，使你可以进入下一个房间。在 [清单 7-6](ch07.xhtml#ch07list6) 中添加新代码，位置在检查键盘按键之后，并在检查玩家是否站在不该站的位置之前。确保你包括以注释符号（#）开头的指令，我们以后会用到它们。

[清单 7-6](ch07.xhtml#ch07list6) 中灰色的行显示了你可以在哪里添加新代码。将你的程序保存为 *listing7-6.py*。使用 `pgzrun listing7-6.py` 运行它，然后在空间站中四处走动！这是一个很好的时机，在门被安装之前，看看周围的环境，因为某些区域会被锁住。

*listing7-6.py*

--snip--

def game_loop():

--snip--

player_direction = "down"

player_frame = 1

# 检查是否退出房间

➊     如果 player_x == room_width: # 通过右侧的门

#clock.unschedule(hazard_move)

➋         current_room += 1

➌         generate_map()

➍         player_x = 0 # 从左侧进入

➎         player_y = int(room_height / 2) # 从门口进入

➏         player_frame = 0

➐         #start_room()

➑         return

➒     如果 player_x == -1: # 通过左侧的门

#clock.unschedule(hazard_move)

current_room -= 1

generate_map()

player_x = room_width - 1  # 从右侧进入

player_y = int(room_height / 2) # 从门口进入

player_frame = 0

#start_room()

return

➓     如果 player_y == room_height: # 通过底部的门

#clock.unschedule(hazard_move)

current_room += MAP_WIDTH

generate_map()

player_y = 0 # 从顶部进入

player_x = int(room_width / 2) # 从门口进入

player_frame = 0

#start_room()

return

如果 player_y == -1: # 通过顶部的门

#clock.unschedule(hazard_move)

current_room -= MAP_WIDTH

generate_map()

player_y = room_height - 1 # 从底部进入

player_x = int(room_width / 2) # 从门口进入

player_frame = 0

#start_room()

return

# 如果玩家站在不该站的位置，将其移回。

如果 room_map[player_y][player_x] 不在 items_player_may_stand_on 中：#\

#           或者 hazard_map[player_y][player_x] != 0:

player_x = old_player_x

--snip--

*清单 7-6：让玩家在房间之间移动*

要查看此代码的工作原理，我们使用一个示例房间地图。[图 7-3](ch07.xhtml#ch07fig3) 显示了一个宽 9 个瓦片、高 9 个瓦片的房间，四面都有出口。我们将使用此图像来理解玩家在离开房间时的位置。

正如你所知道的，地图上的位置是从左上角的 0 开始编号的。黄色的方块表示玩家如果走出房间，可能的位置：

+   如果玩家的 *y* 位置为 −1，则表示他们已走出顶部出口。

+   如果玩家的*x*位置为-1，说明他们走出了左侧出口。

+   如果玩家的*y*位置与room_height变量相同，说明他们已经走出了底部。瓦片位置是从0开始编号的，因此，如果玩家进入一个有9行的房间的第9行，他们已经离开了房间。

+   同样，如果玩家的*x*位置与room_width变量相同，说明他们走出了右侧出口。

![image](../images/fig7-3.jpg)

*图 7-3：判断玩家是否走过出口*

新的代码行检查玩家的位置是否意味着他们已经走出了房间。如果玩家的*x*位置与room_width ➊相同，他们就走出了右侧出口，如[图 7-3](ch07.xhtml#ch07fig3)所示。

当玩家离开房间时，我们需要更改他们所在房间的房间编号，这个编号存储在current_room变量中。当他们从右侧的门走出去时，房间编号会增加1 ➋。再次查看房间地图（请翻到[图 4-1](ch04.xhtml#ch04fig1)页面[60](ch04.xhtml#page_60)），看看这是否合理：房间编号是从左到右递增的。例如，如果玩家在房间33并通过右侧出口走出去，他们就会进入房间34。

程序接着生成一个新的room_map列表 ➌，用来显示和导航新的房间。玩家的位置会被重新设置到房间的另一侧 ➍，看起来就像是他们穿过了门口。如果玩家从房间的右侧出口出去，他们会从左侧进入下一个房间 ➍。

房间大小各不相同，所以我们还需要更改玩家的*y*位置，将其放置在门口的正中间。否则，玩家可能会从墙壁中穿出来！我们将玩家的位置设置为房间高度的一半 ➎，这样他们就位于门口的正中间。当他们进入房间时，我们也会重置玩家动画 ➏。

我在这里加入了几个我们稍后会用到的功能，所以请确保你包含了clock.unschedule(hazard_move) ➊和start_room() ➐指令。start_room()函数会在玩家进入新房间时显示房间名称。我们将在本书后面讲解这些指令。

最后，return指令会退出game_loop()函数 ➑。该函数中的任何进一步指令在本次循环中都不会执行。当函数重新开始时，它将像往常一样从顶部开始。

接下来的代码块 ➒会检查玩家是否通过了左侧门。要通过左侧门，程序会执行以下操作：

+   检查player_x变量是否包含-1（参见[图 7-3](ch07.xhtml#ch07fig3)）。

+   从当前房间编号减去1以进入左侧房间。

+   设置玩家的*x*位置为正好在右侧门口内侧的位置。这个位置是room_width减去1。（你可以在[图 7-3](ch07.xhtml#ch07fig3)中检查。在一个room_width为9的房间中，玩家的*x*位置应该是8。）

+   使用room_height将玩家的*y*位置设置到中间。这与走过右侧出口的方式相同。

顶部和底部出口使用相同的代码结构➓。但是，程序会检查玩家的*y*位置，看他们是否使用了出口，并将他们的新位置设置为通过顶部或底部门进入。

这次，我们将房间编号改变了5，而不是1，因为游戏地图的宽度就是5个房间（参见[图 4-1](ch04.xhtml#ch04fig1)）。例如，如果你在房间37并通过顶部出口，你将进入房间32（37减去5）。如果你在房间37并通过底部出口，你将进入房间42（37加上5）。我们之前将数字5存储在变量MAP_WIDTH中，程序在这里使用了它。

现在，你可以自由探索太空站了。在下一章，我们将修复房间显示中的剩余几个bug。

### **你准备好飞行了吗？**

勾选以下选项框以确认你已经学习了本章的关键内容。

![图片](../images/box.jpg) 玩家在*Escape*游戏中的位置以格子为单位，就像风景一样。

![图片](../images/box.jpg) game_loop()函数控制玩家移动，并计划每0.03秒运行一次。

![图片](../images/box.jpg) 如果玩家移动到一个不允许进入的位置，他们会被迅速送回到上一个位置，你几乎看不见他们的移动。

![图片](../images/box.jpg) 程序检查玩家的*x*和*y*位置，以确定他们是否走出了一个出口。如果是，他们将出现在下一个房间相反出口的中间。

![图片](../images/box.jpg) 动画帧被存储在PLAYER字典中，并为每个方向提供了一系列图像。字典的键是方向名称，通过索引数字可以获得特定的帧。

![图片](../images/box.jpg) 第0帧是静止位置。第1、2、3和4帧显示了宇航员行走的动作。

![图片](../images/box.jpg) game_loop()函数增加了玩家行走时使用的动画帧数。

![图片](../images/box.jpg) player_offset_x和player_offset_y变量用于在玩家进入新格子时正确定位宇航员。

![图片](../images/f0126-01.jpg)
