## **8

**修复空间站**

![image](img/common01.jpg)

当你在空间站内四处漫游时，你一定注意到有些地方看起来不太对。为了快速启动程序，我们使用`EXPLORER`部分来显示房间。然而，它也有一些缺点：

+   有时，场景下方会显示一片空白，因为那里没有地板。

+   当你走到房间前方时，前墙会遮挡住宇航员。

+   宇航员的腿在走到屏幕后方时消失。

+   所有房间都绘制在游戏窗口的左上角。这让它看起来不均匀且不一致，因为房间右边的空间远比左边的多，宽的房间在右侧的空间比窄房间少。

+   没有阴影，导致更难理解房间中物体的位置。

在本章中，我们将修复这些故障，并添加一个在窗口顶部显示信息的函数。这些信息将向玩家提供有关空间站以及他们在游戏中进展的情况。

在你阅读本章时，你将学会如何将信息传递给 Python 函数，并发现如何使用 Pygame Zero 绘制矩形。到本章结束时，空间站将变得很棒！

### **将信息传递给一个函数**

第一次，我们需要将信息传递给函数。你已经看过如何通过将信息放入括号中传递给`print()`函数。例如，你可以像这样输出一条信息：

```py
print("Learn your emergency evacuation drill")
```

当该指令运行时，`print()`函数接收你放入括号中的信息，并在命令行窗口或 Python Shell 中显示出来。

我们还可以将信息传递给我们自己创建的函数。

#### **创建一个接收信息的函数**

为了实验函数，我们将构建一个可以将两个数字相加的函数。点击**文件** ▸ **新建**打开一个新窗口，并在清单 8-1 中输入程序。

*listing8-1.py*

```py
➊ def add(first_number, second_number):
➋     total = first_number + second_number
➌     print(first_number, "+", second_number, "=", total)

➍ add(5, 7)
   add(2012, 137)
   add(1234, 4321)
```

*清单 8-1：将信息传递给一个函数*

将程序保存为*listing8-1.py*。因为它没有使用任何 Pygame Zero 功能，你可以通过点击**运行** ▸ **运行模块**或按 F5 来运行它。（如果使用 Pygame Zero 运行，结果会显示在命令行窗口，游戏窗口将为空。）

当你运行程序时，你应该能看到以下输出：

```py
5 + 7 = 12
2012 + 137 = 2149
1234 + 4321 = 5555
```

我们创建了一个名为`add()`的新函数➊。在定义了`add()`之后，我们可以通过使用其名称➍来运行它，并通过将数字放入括号中，使用逗号分隔它们➍，向它发送数字。然后，函数会将这两个数字相加。

#### **工作原理**

为了使函数能够接收数字，我们在定义它时给它提供了两个变量来存储这些数字。我将它们命名为`first_number`和`second_number`➊，以便让程序更容易理解，但这些变量名可以是任何名字。这些是局部变量：它们只在这个函数内部有效。

当你使用这个函数时，它会获取接收到的第一个项目，并将其放入变量`first_number`中。第二个项目会放入`second_number`中。

当然，两个数字相加的顺序并不重要，所以传递数字的顺序也不重要。指令`add(5, 7)`和`add(7, 5)`会得到相同的结果。但有些函数需要你按照函数预期接收信息的顺序来传递参数。例如，如果函数是做减法运算的，那么如果你传递数字的顺序错误，你将得到不同的结果。知道函数期望接收什么信息的唯一方法是查看它的代码。

函数体非常简单。它创建了一个新变量`total`，用于存储两个数字相加的结果➋。然后程序打印一行，内容包括第一个数字、加号、第二个数字、等号和总和➌。

在最后三条指令中，我们将三个数字对传递给函数进行加法运算➍。

这个简单的演示向你展示了如何将信息（或*参数*）传递给函数。你可以创建接受比两个参数更多参数的函数，甚至可以接受列表、字典或图像。函数使得重复使用指令集变得简单，而传递参数意味着我们可以使用不同的信息重复使用这些指令。例如，清单 8-1 使用了三次相同的`print()`指令，来显示三个不同数字对的和。在这种情况下，我们避免了重复使用`print()`指令和设置`total`变量的指令。更复杂的函数可以避免重复大量代码，这使得程序更容易编写和理解。

**训练任务 #1**

尝试修改程序，使其从一个数字中减去另一个数字，而不是加法运算。当你改变传递给新函数的数字顺序时，会发生什么呢？你可能需要修改的不仅仅是计算部分，确保函数易于使用。

现在我们准备向*Escape*游戏添加一些新函数，用来在空间站上绘制物体。

### **添加阴影、墙壁透明度和颜色的变量**

为了修复我们的空间站，我们将使用新学到的函数知识，为*Escape*游戏创建新的显示函数。在创建这些新函数之前，我们需要为函数使用设置新的变量。

打开 *listing7-6.py*，这是你在第七章保存的最后一个列表。在程序的开头找到 `VARIABLES` 部分，并添加列表 8-2 中显示的新行。将程序保存为 *listing8-2.py*。像往常一样，运行程序（使用 pgzrun listing8-2.py）检查是否有新的错误。

*listing8-2.py*

```py
   --snip--

   ###############
   ## VARIABLES ##
   ###############

   --snip--

   player_image = PLAYER[player_direction][player_frame]
   player_offset_x, player_offset_y = 0, 0

➊ PLAYER_SHADOW = {
       "left": [images.spacesuit_left_shadow, images.spacesuit_left_1_shadow,
                images.spacesuit_left_2_shadow, images.spacesuit_left_3_shadow,
                images.spacesuit_left_3_shadow
                ],
       "right": [images.spacesuit_right_shadow, images.spacesuit_right_1_shadow,
                 images.spacesuit_right_2_shadow,
                 images.spacesuit_right_3_shadow, images.spacesuit_right_3_shadow
                 ],
       "up": [images.spacesuit_back_shadow, images.spacesuit_back_1_shadow,
              images.spacesuit_back_2_shadow, images.spacesuit_back_3_shadow,
              images.spacesuit_back_3_shadow
              ],
       "down": [images.spacesuit_front_shadow, images.spacesuit_front_1_shadow,
                images.spacesuit_front_2_shadow, images.spacesuit_front_3_shadow,
                images.spacesuit_front_3_shadow
                ]
       }

➋ player_image_shadow = PLAYER_SHADOW["down"][0]

➌ PILLARS = [
       images.pillar, images.pillar_95, images.pillar_80,
       images.pillar_60, images.pillar_50
       ]

➍ wall_transparency_frame = 0

➎ BLACK = (0, 0, 0)
   BLUE = (0, 155, 255)
   YELLOW = (255, 255, 0)
   WHITE = (255, 255, 255)
   GREEN = (0, 255, 0)
   RED = (128, 0, 0)

   ###############
   ##    MAP    ##
   ###############

   --snip--
```

*列表 8-2：为新的显示函数添加所需的变量*

我们添加了一个与 `PLAYER` 字典类似的 `PLAYER_SHADOW` 字典 ➊。它包含了宇航员阴影的动画帧。当宇航员移动时，阴影的形状也会发生变化。`player_image_shadow` ➋ 存储着当前的宇航员阴影，就像 `player_image` 变量存储当前的宇航员动画帧（或站立图像）。

在本章后面，我们将添加动画，使得当你走到前墙后面时，前墙会渐隐，这样你仍然可以看到宇航员。在这里，我们设置了一个包含动画帧的列表 ➌ 和一个 `wall_transparency_frame` 变量，用来记住当前显示的动画帧 ➍。你稍后将学习更多关于这些是如何工作的。

我们还设置了一些名称，可以用来引用颜色编号 ➎。Pygame Zero 中的颜色存储为元组。元组像一个列表，但其中的内容不可更改，并且使用圆括号而不是方括号。你已经在绘制屏幕坐标时看到过元组的使用（见第一章）。颜色作为三个数字存储，分别指定了颜色中的红色、绿色和蓝色的量，顺序是这样的。每种颜色的范围从 0 到 255。这个颜色是亮红色：

```py
(255, 0, 0)
```

红色达到了最大值（255），而绿色（0）和蓝色（0）则没有出现在颜色中。

因为我们已经设置了这些颜色变量，现在可以使用名称`BLACK`来代替使用元组`(0, 0, 0)`来表示黑色。使用颜色名称会让程序更易于阅读。

表 8-1 展示了一些你可能在程序中使用的颜色组合。你也可以尝试不同的数字，发明你自己的颜色。

**表 8-1：** 一些示例 RGB 颜色值

| **红色** | **绿色** | **蓝色** | **描述** |
| --- | --- | --- | --- |
| 255 | 0 | 0 | 亮红色 |
| 0 | 255 | 0 | 亮绿色 |
| 0 | 0 | 255 | 亮蓝色 |
| 0 | 0 | 50 | 极暗蓝色（几乎是黑色！） |
| 255 | 255 | 255 | 白色（所有颜色的最大强度） |
| 255 | 255 | 150 | 奶油黄色（比白色少一点蓝色） |
| 230 | 230 | 230 | 银色（稍微调暗的白色） |
| 200 | 150 | 200 | 淡紫色 |
| 255 | 100 | 0 | 橙色（最大红色，略带绿色） |
| 255 | 105 | 180 | 粉色 |

### **删除资源管理器部分**

我们需要添加一个新的 `DISPLAY` 部分，其中包含一些新的函数，将改善游戏在屏幕上的显示效果。`EXPLORER` 部分帮助我们快速启动了程序，但我们将在本章中构建一个新的、更好的 `draw()` 函数，替代我们目前使用的那个。为了避免程序中仍然残留 `EXPLORER` 代码带来的问题，我们将删除它。你的 `EXPLORER` 部分可能比我的 图 8-1 中的多一些或少一些，具体取决于你是否在前几章删除了一些内容。

要删除整个 `EXPLORER` 部分，请按照以下步骤操作：

1.  找到代码接近结尾处的 `EXPLORER` 部分。

1.  点击 `EXPLORER` 注释框的起始位置，按住鼠标按钮，并拖动鼠标至该部分的底部（见 图 8-1）。该部分结束的位置在 `START` 部分开始之前。

1.  按下键盘上的 DELETE 或 BACKSPACE 键。

在 `EXPLORER` 部分中有一个我们仍然需要的指令：它运行 `generate_map()` 函数来设置第一个房间的地图。你需要将该指令作为单独的一行添加到程序的末尾，如 清单 8-3 所示。

*listing8-3.py*

```py
--snip--
###############
##   START   ##
###############

clock.schedule_interval(game_loop, 0.03)
generate_map()
```

*清单 8-3：为第一个房间生成地图*

`generate_map()` 这一行将在变量设置好后运行，并为当前房间生成地图。

![image](img/fig8-1.jpg)

*图 8-1：删除 EXPLORER 部分*

将新清单保存为 *listing8-3.py*，并使用 `pgzrun listing8-3.py` 运行它。如果一切顺利，命令行窗口应该不会显示任何错误信息。游戏窗口显示的是漆黑的太空，因为我们还没有添加绘制任何内容的新代码。

### **添加 DISPLAY 部分**

现在，我们将添加新的 `DISPLAY` 部分，来替代已删除的 `EXPLORER` 部分。这个部分包含了更新屏幕显示的大部分代码。它包括绘制房间、显示信息和改变前墙透明度的代码。

#### **添加绘制对象的函数**

首先，我们将创建一些函数，用于在特定的瓦片位置绘制对象、阴影或玩家。在 `GAME LOOP` 和 `START` 部分之间，添加新的 `DISPLAY` 部分，如 清单 8-4 所示。将该程序保存为 *listing8-4.py* 并使用 `pgzrun listing8-4.py` 运行。再次提醒，游戏窗口中还不会显示任何内容。

如果命令行窗口中有任何错误，你可以利用它们来帮助修复程序。最好在添加代码时进行测试，而不是一次性添加大量代码后再去找错误。

*listing8-4.py*

```py
   --snip--

       if player_direction == "down" and player_frame > 0:
           player_offset_y = -1 + (0.25 * player_frame)

   ###############
   ##  DISPLAY  ##
   ###############

➊ def draw_image(image, y, x):
➋     screen.blit(
           image,
           (top_left_x + (x * TILE_SIZE),
            top_left_y + (y * TILE_SIZE) - image.get_height())
           )

➌ def draw_shadow(image, y, x):
       screen.blit(
           image,
           (top_left_x + (x * TILE_SIZE),
            top_left_y + (y * TILE_SIZE))
           )

   def draw_player():
➍     player_image = PLAYER[player_direction][player_frame]
➎     draw_image(player_image, player_y + player_offset_y,
                  player_x + player_offset_x)
➏     player_image_shadow = PLAYER_SHADOW[player_direction][player_frame]
➐     draw_shadow(player_image_shadow, player_y + player_offset_y,
                   player_x + player_offset_x)

   ###############
   ##   START   ##
   ###############

   clock.schedule_interval(game_loop, 0.03)
   generate_map()
```

*清单 8-4：在 DISPLAY 部分添加第一个函数*

第一个新的函数 `draw_image()` ➊ 用于在屏幕上绘制指定的图像。当我们使用它时，我们会传入要绘制的图像和物体在房间中的 *y* 和 *x* 瓷砖位置。该函数会根据房间中的瓷砖位置计算出在屏幕上绘制图像的位置（像素位置）。例如，我们可以这样使用这个函数：

```py
draw_image(player_image, 5, 2)
```

这行代码将玩家的图像绘制在房间中位置 *y* = 5 和 *x* = 2 处。

当我们定义 `draw_image()` 函数时，我们设置它来接收图像名称 `image`，将 *y* 位置赋值给 `y` 变量，将 *x* 位置赋值给 `x` 变量 ➊。虽然 `draw_image()` 函数有几行代码，但它的唯一指令是 `screen.blit()`，该指令根据我们指定的位置绘制图像 ➋。这个指令几乎与我们在旧版 `EXPLORER` 部分中使用的相同，所以请查看 第三章 以复习它的工作原理。

**提示**

确保所有括号的位置正确。你需要在所有 `screen.blit()` 参数周围加上一对括号，并且在 *y* 和 *x* 位置的括号周围也要加上一对，因为它们组成一个元组。你还需要在位置计算的乘法部分加上一对括号。如果程序无法运行，首先检查错误时，要数一数开括号和闭括号的数量，确保它们的数量相同。

然后我们添加了一个新的 `draw_shadow()` 函数 ➌。这个函数与绘制图像的函数类似，不同之处在于计算图像在屏幕上的位置时，并不会减去图像的高度。这就是将阴影绘制在主图像 *下方* 的原因。图 8-2 显示了基于相同瓷砖位置的宇航员和他们的阴影。请记住，传递给 `screen.blit()` 的 *y* 位置是图像的顶部边缘。

![image](img/fig8-2.jpg)

*图 8-2：计算图像和阴影的位置*

第三个新的函数 `draw_player()` 用于绘制宇航员。首先，它将正确的宇航员动画帧放入 `player_image` ➍。然后使用新的 `draw_image()` 函数来绘制 ➎。`draw_image()` 函数需要以下参数：

+   变量 `player_image` 包含需要绘制的图像。

+   将 `player_y` 和 `player_offset_y` 变量相加后的结果。这是 *y* 位置的瓷砖数，可能包括小数部分（例如 5.25）。

+   将 `player_x` 和 `player_offset_x` 相加后的结果，用于计算 *x* 位置的瓷砖数。（有关如何使用偏移变量进行动画的更多信息，请参见“理解移动代码”第 119 页）

我们使用类似的代码来绘制玩家的阴影：将正确的动画帧从 `PLAYER_SHADOW` 字典中取出并放入 `player_image_shadow` ➏。然后使用 `draw_shadow()` 函数将其绘制出来 ➐。`draw_shadow()` 函数使用与 `draw_image()` 函数相同的瓷砖位置。

#### **绘制房间**

现在我们已经创建了绘制对象和玩家的函数，可以添加绘制房间的代码了。列表 8-5 中的新`draw()`函数为场景和玩家添加了阴影，并修复了之前看到的视觉故障。

将新的代码添加到`DISPLAY`部分的末尾，将程序保存为*listing8-5.py*，并使用`pgzrun listing8-5.py`运行它。就像你打开了灯光，阴影出现在物体前面。游戏看起来还不完全正确，因为所有房间都会绘制在窗口的左上角，有时当你离开房间时，房间不会被正确清除。我们稍后会解决这个问题。在这一点上，你不应该看到任何错误信息。

*listing8-5.py*

```py
   --snip--

   def draw_player():
       player_image = PLAYER[player_direction][player_frame]
       draw_image(player_image, player_y + player_offset_y,
                  player_x + player_offset_x)
       player_image_shadow = PLAYER_SHADOW[player_direction][player_frame]
       draw_shadow(player_image_shadow, player_y + player_offset_y,
                   player_x + player_offset_x)
   def draw():
       if game_over:
           return

➊     # Clear the game arena area.
       box = Rect((0, 150), (800, 600))
       screen.draw.filled_rect(box, RED)
       box = Rect ((0, 0), (800, top_left_y + (room_height - 1)*30))

➋     screen.surface.set_clip(box)
       floor_type = get_floor_type()

➌     for y in range(room_height): # Lay down floor tiles, then items on floor.
           for x in range(room_width):
               draw_image(objects[floor_type][0], y, x)
               # Next line enables shadows to fall on top of objects on floor
               if room_map[y][x] in items_player_may_stand_on:
                   draw_image(objects[room_map[y][x]][0], y, x)

➍     # Pressure pad in room 26 is added here, so props can go on top of it.
       if current_room == 26:
           draw_image(objects[39][0], 8, 2)
           image_on_pad = room_map[8][2]
           if image_on_pad > 0:
               draw_image(objects[image_on_pad][0], 8, 2)

➎     for y in range(room_height):
           for x in range(room_width):
               item_here = room_map[y][x]
               # Player cannot walk on 255: it marks spaces used by wide objects.
               if item_here not in items_player_may_stand_on + [255]:
                   image = objects[item_here][0]

➏                 if (current_room in outdoor_rooms
                       and y == room_height - 1
                       and room_map[y][x] == 1) or \
                       (current_room not in outdoor_rooms
                       and y == room_height - 1
                       and room_map[y][x] == 1
                       and x > 0
                       and x < room_width - 1):
                       # Add transparent wall image in the front row.
                       image = PILLARS[wall_transparency_frame]

                   draw_image(image, y, x)

➐                 if objects[item_here][1] is not None: # If object has a shadow
                       shadow_image = objects[item_here][1]
                       # if shadow might need horizontal tiling
➑                     if shadow_image in [images.half_shadow,
                                           images.full_shadow]:
                           shadow_width = int(image.get_width() / TILE_SIZE)
                           # Use shadow across width of object.
                           for z in range(0, shadow_width):
                               draw_shadow(shadow_image, y, x+z)
                       else:
                           draw_shadow(shadow_image, y, x)

➒         if (player_y == y):
                   draw_player()

➓     screen.surface.set_clip(None)

   ###############
   ##   START   ##
   ###############

   clock.schedule_interval(game_loop, 0.03)
   generate_map()
```

*列表 8-5：新的* draw() *函数*

与第七章中的移动代码一样，即使你想自定义程序，你也不需要知道`draw()`函数是如何工作的。我将在下一部分解释`draw()`函数，所以如果你暂时不想知道它是如何工作的，可以跳到“在屏幕上定位房间”，第 141 页(page 141)。

#### **理解新的 DRAW() 函数**

你可以将新的`draw()`函数看作是之前`EXPLORER`部分代码的一个更复杂版本。我将为你概述每一部分的工作原理。

##### **清理游戏场地**

程序通过清理游戏场地 ➊ 来开始绘制太空站的位置。它通过绘制一个大红色矩形来清除之前的屏幕显示。顶部和底部的区域提供玩家信息是独立的，因此它们不会被更改。

将矩形显示在屏幕上有两个步骤。首先，你使用一个名为*Rect*的 Pygame 对象来创建这个形状，它的工作方式如下（不要键入此内容）：

```py
box = Rect((left position, top position), (width, height))
```

名称几乎可以是任何你喜欢的名字，但在我的程序中，我使用`box`这个名字。位置和大小是元组，因此它们用括号括起来。

然后，你通过使用类似于以下指令的代码在屏幕上绘制你创建的矩形（再次提醒，不要键入此内容）：

```py
screen.draw.filled_rect(box, color)
```

括号中的第一个项目是你之前创建的 `box` 矩形。第二个项目是你想要绘制的矩形的颜色。这可以是由红色、绿色和蓝色构成的数字元组。在列表 8-5 中，我使用了我们在`VARIABLES`部分之前设置的`RED`名称。

你还可以使用矩形形状创建一个*裁剪区域* ➋。这就像一个不可见的窗口，你通过它查看屏幕。如果程序绘制的东西超出了这个窗口，它就无法被看到。我设置了一个裁剪区域，区域的高度与房间一致，以防止玩家的阴影在站在前门口时从游戏底部溢出。

##### **绘制房间**

房间分为两个阶段绘制。首先，程序绘制地板瓷砖和玩家可以走动的区域 ➌。先绘制它们能使景物、玩家和阴影绘制在其上面。这解决了景物下出现黑洞的问题，因为在景物绘制之前，地板瓷砖已经存在于这些空间中。

接着，程序使用新的循环添加房间中的景物，包括它的阴影 ➎。由于这些是在整个房间的地板绘制完之后绘制的，阴影会覆盖在地板瓷砖和地面上的物品之上。阴影是透明的，因此你仍然可以看到阴影下方的物体。景物绘制的循环还会添加透明墙 ➏，并将玩家绘制在地面之上 ➒。

一如既往，房间是从后到前绘制的，以确保房间前方的物体看起来位于房间后方物体之前。

我们还添加了一小段代码，用于一个在游戏中只会在一个地方使用的特殊物体。26 号房间的地板上有一个压力垫，你在游戏中可能想要将物品放在上面（也许是重物，或者是你可以让它变重的物品...）。这里的特殊代码确保了地板垫和它上面的物体都能被显示出来。

在地板瓷砖绘制完后，`draw()`函数会检查当前房间是否是 26 号房间：如果是，它会先绘制地板垫，然后绘制任何在其上的物体 ➍。

**红色警报**

*如果你正在用自己的地图自定义游戏，请删除这段代码以移除游戏中的地板垫。从注释行* ➍*开始，并删除直到（包括）*draw_image(objects[image_on_pad][0], 8, 2) *指令。*

##### **使前墙透明**

当程序绘制房间的前排（当`y`循环等于`room_height - 1`时），它会检查是否需要绘制半透明墙壁，而不是从房间地图中获取的实体墙物体 ➏。如果玩家站在墙后面，则使用半透明墙壁（见图 8-3）。

在行星表面，程序使整个墙体透明。在太空站内部，只有当透明墙面板不位于底部角落位置时才使用透明墙面板（见图 8-3）。角落位置总是使用实体墙面板。原因是，如果你看到实体墙面板从倒数第二行开始，效果看起来会很奇怪。

稍后我们将添加代码，通过改变`wall_transparency_frame`中的数值来为墙壁添加透明度动画。此时在游戏中你还看不到半透明的墙壁。

![image](img/fig8-3.jpg)

*图 8-3：如最终游戏中所见，房间前面的透明墙*

##### **添加阴影**

如果一个物体有阴影，那么阴影会从`objects`字典中取出并放入`shadow_image` ➐。然后程序检查是否应该使用`half_shadow`或`full_shadow`，分别填充半块或整块瓷砖。这两种标准阴影适用于不需要特殊阴影轮廓的方块状物品（如电气单元和墙壁）。程序会检查`shadow_image`是否位于一个包含这两种标准图像的列表中 ➑。

这是一种简单且易于阅读的方式，用来检查`shadow_image`是否是两种情况之一。如果你检查的是三种或更多的情况，使用这种技巧可以让程序比使用大量`if`比较和结合`==`与`or`更加易读。

如果阴影是标准图像之一，程序将计算阴影应该有多宽，以瓷砖为单位。这是通过获取投射阴影的物体的宽度，并将其除以瓷砖的宽度（30 像素）来计算的。例如，一个宽度为 90 像素的图像，将是 3 个瓷砖宽。

程序接着创建一个循环，用变量`z`来绘制标准阴影图像。`z`从 0 开始，一直到阴影宽度减去 1。因为`range`不包括最后一项：`range(0, 3)`会返回 0、1 和 2 这三个数值。`z`值会加到主循环中的*x*位置，并用于绘制阴影瓷砖。图 8-4 展示了一个宽度为 3 个瓷砖的物体。`z`循环取值 0、1 和 2，用来把阴影绘制在正确的位置。

通过在铺设地板后绘制玩家角色，确保了宇航员的腿部在屏幕上移动时不会消失 ➒。

![image](img/fig8-4.jpg)

*图 8-4：宽度为 3 个瓷砖的物体下方可以有一个标准阴影，这个阴影会被使用三次。*

`draw()`函数的最后，关闭了限制区域，避免阴影溢出游戏区域的底部 ➓。

### **将房间定位到屏幕上的位置**

现在让我们解决房间出现在屏幕左上角的问题。程序使用两个变量来定位房间：`top_left_x` 和 `top_left_y`。目前，这两个变量的值分别为 100 和 150，这意味着房间总是绘制在窗口的左上角。我们将添加一些代码，根据房间的大小改变这些变量的值，使房间居中显示在窗口中（见图 8-5）。这样屏幕布局看起来会更好，也会让游戏更容易操作。

![image](img/fig8-5.jpg)

*图 8-5：房间居中显示在窗口中*

将清单 8-6 中所示的新行添加到`generate_map()`函数的末尾，该函数位于程序的`MAKE MAP`部分。由于它们位于一个函数内，你需要将每行缩进四个空格。

将程序保存为*listing8-6.py*并使用 pgzrun listing8-6.py 运行。如图 8-5 所示，现在每个房间应该已经居中显示在屏幕上。

*listing8-6.py*

```py
   --snip--

   def generate_map():

   --snip--

               for tile_number in range(1, image_width_in_tiles):
                   room_map[scenery_y][scenery_x + tile_number] = 255

➊     center_y = int(HEIGHT / 2) # Center of game window
➋     center_x = int(WIDTH / 2)
➌     room_pixel_width = room_width * TILE_SIZE # Size of room in pixels
➍     room_pixel_height = room_height * TILE_SIZE
➎     top_left_x = center_x - 0.5 * room_pixel_width
➏     top_left_y = (center_y - 0.5 * room_pixel_height) + 110
```

*Listing 8-6：创建变量将房间放置在游戏窗口的中央*

这些指令位于`generate_map()`函数内，该函数在玩家进入每个房间时设置`room_map`列表。`generate_map()`函数现在还会设置`top_left_x`和`top_left_y`变量，记住房间应当在窗口中的绘制位置。

Listing 8-6 中的新代码首先计算出窗口的中心位置。`HEIGHT`和`WIDTH`变量存储窗口的尺寸（像素）。将它们除以 2 可以得到窗口中心的坐标。我们将这些坐标存储在`center_y` ➊和`center_x` ➋变量中。

程序接着计算房间图像的宽度（像素）➌。它将是房间的瓷砖数乘以单个瓷砖的大小。计算结果存储在`room_pixel_width`中。对于房间高度，程序进行类似的计算➍。

为了将房间图像放置在房间的中央，我们希望房间的左半部分在中心线的左侧，右半部分在右侧。因此，我们从中心线➎开始，减去房间宽度的一半像素，然后在那里开始绘制房间。

对于`top_left_y`，我们使用类似的计算方法，只是我们在结果上加上了 110➏。我们需要加上 110，因为最终的屏幕布局会在屏幕的顶部使用一个信息面板区域。我们稍微将房间图像向下移动一些，为面板腾出空间。

### **使前墙逐渐显现与消失**

此时，游戏中有一些死角，玩家在这些地方无法被看到。在房间中央，我们可以通过确保物体不太高以免遮挡玩家来避免这种情况。然而，我们仍然需要在房间前面放置一面高墙。

用墙壁将玩家挡在房间前面可能会引发各种问题：如果你掉了什么东西，你将无法找到它；或者如果有什么东西在伤害你，你将看不见它！解决方法是让墙壁在玩家接近时逐渐消失。

`draw()`函数已经使用动画帧绘制了前墙的柱子。墙壁动画共有五帧（编号从 0 到 4），存储在`PILLARS`列表中。第一帧是实心墙，而最后一帧显示的是墙壁最透明的状态（见表 8-2）。随着动画帧数的增加，墙壁变得越来越透明。当前帧存储在变量`wall_transparency_frame`中。

由于透明度在图像中的表现方式，当透明墙被绘制在玩家上方时，玩家将透过墙壁可见。

**表 8-2：前墙的动画帧**

| **帧编号** | **0** | **1** | **2** | **3** | **4** |
| --- | --- | --- | --- | --- | --- |
| 图像 | ![image](img/f0143-01.jpg) | ![image](img/f0143-02.jpg) | ![image](img/f0143-03.jpg) | ![image](img/f0143-04.jpg) | ![image](img/f0143-05.jpg) |

Listing 8-7 展示了一个新函数 `adjust_wall_transparency()`，它会使墙壁渐显或渐隐。将它添加到 `DISPLAY` 部分的末尾，紧接着你刚完成的 `draw()` 函数，并放在 `START` 部分之前。你还需要在程序的末尾、函数外部添加一行代码，安排它定期运行。这一行也在 Listing 8-7 中。

将更新后的程序保存为 *listing8-7.py* 并使用 pgzrun listing8-7.py 运行它。如果你走到前面的墙后面，它现在会渐变成透明，这样你就可以被看到（请参见本章前面的 Figure 8-3）。当你再次走开时，墙壁会恢复为不透明。

*listing8-7.py*

```py
   --snip--

   ###############
   ##  DISPLAY  ##
   ###############

   --snip--

       screen.surface.set_clip(None)

   def adjust_wall_transparency():
       global wall_transparency_frame

➊     if (player_y == room_height - 2
➋         and room_map[room_height - 1][player_x] == 1
➌         and wall_transparency_frame < 4):  
➍         wall_transparency_frame += 1 # Fade wall out.

➎     if ((player_y < room_height - 2
➏             or room_map[room_height - 1][player_x] != 1)
➐             and wall_transparency_frame > 0):
➑         wall_transparency_frame -= 1 # Fade wall in.

   ###############
   ##   START   ##
   ###############

   clock.schedule_interval(game_loop, 0.03)
   generate_map()
➒ clock.schedule_interval(adjust_wall_transparency, 0.05)
```

*Listing 8-7: 当你靠近前墙时让它变得透明*

我们在 Listing 8-7 中添加的最后一行让 `adjust_wall_``transparency()` 函数每 0.05 秒运行一次 ➒。这使得墙壁在玩家在房间里走动时根据需要渐显或渐隐。

让我们看看这个新函数是如何工作的。如果玩家站在墙后面，以下两个语句成立：

+   它们的 *y* 位置将等于 `room_height - 2` ➊。正如 Figure 8-6 所示，地图的底行是 `room_height - 1`。所以我们检查玩家是否在这一行之上。

+   在房间底行中，有一块墙与玩家的 *x* 位置对齐 ➋。 在 Figure 8-6 中，红色方块标记了一个我们无法看到玩家的位置。它们前方的底行包含一个表示墙壁的 1。绿色方块显示了我们能看到玩家的位置，因为他们在门口。这里，房间底行地图包含 0。

![image](img/fig8-6.jpg)

*Figure 8-6: 判断玩家是否在墙后面*

如果玩家在墙后面 ➊ ➋ 且墙壁的透明度没有设置为最大值 ➌，则墙壁的透明度增加 1 ➍。

如果以下任一条件成立，则意味着玩家 *没有* 被墙壁遮挡：

+   它们的 *y* 位置小于 `room_height - 2` ➎。如果玩家离房间后面远一些，至少部分是可以被看到的。

+   在与它们的 *x* 位置对齐的房间底行没有墙体 ➏。

在这些情况下，如果墙壁的透明度设置超过最低值 ➐，它会减少 ➑。

`draw()` 函数使用 `wall_transparency_frame` 的值来计算从 `PILLARS` 列表中的动画帧中选择哪个图像用于前排。

效果是墙壁会逐渐淡入淡出，取决于玩家是否站在墙后。这种淡入淡出的速度足够快，玩家不会被延迟，但也不会太快到让它瞬间消失，这样就不会分散注意力。

### **显示提示、技巧和警告**

有时候，*Escape*游戏会使用文本告诉你发生了什么。例如，它可能会使用文本告诉你当你对某个物体做了什么时发生了什么，或者为你提供该物体的描述。

程序中`DISPLAY`部分的最终函数将消息写在游戏窗口的顶部。这里有两行文本：

+   第一行，位于窗口顶部 15 像素的位置，告诉玩家他们正在做什么。例如，它显示物体描述，并告诉玩家当他们使用物体时会发生什么。

+   第二行，位于窗口顶部 50 像素的位置，用于显示重要消息。

这些文本行被这样分隔，以免重要消息被不那么重要的消息覆盖。如果游戏需要告诉你一个生死攸关的情况，你肯定不希望那个消息被告知你进入的新房间的消息所取代！

将列出 8-8 中的新代码添加到`DISPLAY`部分的末尾，在你添加墙壁透明代码之后（在列出 8-7 中）。将该列表保存为*listing8-8.py*。你可以通过运行 pgzrun listing8-8.py 来测试它，但目前你还看不出任何区别。稍后，我们将添加一些指令，使用这个新的`show_text()`函数。

*listing8-8.py*

```py
   --snip--

       if ((player_y < room_height - 2
               or room_map[room_height - 1][player_x] != 1)
               and wall_transparency_frame > 0):
           wall_transparency_frame -= 1 # Fade wall in.

➊ def show_text(text_to_show, line_number):
       if game_over:
           return
➋     text_lines = [15, 50]
➌     box = Rect((0, text_lines[line_number]), (800, 35))
➍     screen.draw.filled_rect(box, BLACK)
➎     screen.draw.text(text_to_show,
                        (20, text_lines[line_number]), color=GREEN)

   ###############
   ##   START   ##
   ###############

   --snip--
```

*列出 8-8：添加文本显示功能*

我们将像这样使用`show_text()` ➊函数（不要直接输入此内容）：

```py
show_text("message", line number)
```

行号将是 0，表示第一行，或者是 1，表示第二行，该行保留给重要消息。在函数开始时，消息被放入变量`text_to_show`中，行号放入`line_number`中➊。

我们使用一个名为`text_lines`的列表来记录两行文本的垂直位置（以像素为单位）➋。我们还定义了一个框➌，并用黑色填充它➍，以便在绘制新消息之前清除文本行。

最后，我们使用 Pygame Zero 中的`screen.draw.text()`函数将文本显示在屏幕上➎。此函数需要文本、文本的* x *和* y *位置，以及文本颜色。位置数字放在括号内（它们组成一个元组）。

在列出 8-8 中➎，* x *位置从左边起 20 像素，垂直位置取自`text_lines`列表，使用`line_number`中的数字作为列表索引。

### **进入房间时显示房间名称**

为了测试 `show_text()` 函数，让我们添加 `start_room()` 函数，它会在你走进房间时显示房间名称。将此函数放在 `GAME LOOP` 部分，位于 `game_loop()` 函数之前，如 列表 8-9 所示。保存你的程序为 *listing8-9.py*。当你运行它时，暂时不会看到任何新内容。

*listing8-9.py*

```py
--snip--

###############
## GAME LOOP ##
###############

def start_room():
    show_text("You are here: " + room_name, 0)

def game_loop():
--snip--
```

*列表 8-9：添加* start_room() *函数*

此函数使用 `room_name` 变量，这是我们在 `generate_map()` 函数中设置的。它包含当前房间的名称，该名称来自 `GAME_MAP` 列表。房间名称与文本 `"You are here: "` 结合，并发送到 `show_text()` 函数。

现在，我们需要设置新的 `start_room()` 函数，以便每当玩家进入新房间时运行。在 第七章 的 列表 7-6 中，我们已经包含了这段代码，但将其注释掉了。现在，我们准备好了！在我们有 `#start_room()` 代码的地方，我们希望将其替换为 `start_room()`。那个 `#` 充当了“关闭开关”，告诉 Python 忽略该指令。为了启用该指令，我们需要移除 `#` 符号。

我们不会手动查找所有需要更改的行，而是让 IDLE 为我们完成这项工作。请按照以下步骤操作，并参考 图 8-7：

1.  在 IDLE 中点击 **编辑** ▸ **替换**（或按 CTRL-H）以显示替换文本对话框。

1.  在“查找”框中输入 #start_room()。

1.  在“替换为”框中输入 start_room()。

1.  点击 **全部替换**。

![image](img/fig8-7.jpg)

*图 8-7：当玩家进入新房间时启用* start_room() *函数*

IDLE 应该在四个地方替换指令，并跳转到列表中的最后一处，如 列表 8-10 所示（无需输入此列表）。

将列表保存为 *listing8-10.py*，并使用 `pgzrun listing8-10.py` 运行程序。当你进入每个新房间时，应该会显示一条消息。这是通过走过门触发的，所以在第一个房间时不会生效。

*listing8-10.py*

```py
--snip--
    if player_y == -1: # through door at TOP
        #clock.unschedule(hazard_move)
        current_room -= MAP_WIDTH
        generate_map()
        player_y = room_height - 1 # enter at bottom
        player_x = int(room_width / 2) # enter at door
        player_frame = 0
        start_room()
        return
--snip--
```

*列表 8-10：当玩家离开房间时启用* start_room() *函数*

这完成了 *Escape* 游戏的 `DISPLAY` 部分！稍后我们将做一些小改动来显示敌人，但除此之外，我们已经为游戏的其余部分打下了基础。

在下一章中，我们将开始拆解你的个人物品，并为游戏添加道具。

### **你准备好飞行了吗？**

勾选以下框来确认你已经学会了本章的关键课程。

![Images](img/box.jpg)  发送给函数的信息称为 *参数*。

![Images](img/box.jpg)  要将信息发送到函数，你需要将其放在函数名后的括号中。如果需要传递多个参数，可以用逗号分隔它们。例如：`add(5, 7)`。

![图片](img/box.jpg)  为了使一个函数能够接收信息，在定义函数时，你需要设置局部变量来接收参数。

![图片](img/box.jpg)  程序的`DISPLAY`部分绘制房间，动画化透明墙，并显示文本信息。

![图片](img/box.jpg)  `show_text()`函数需要两个参数：你想显示的字符串和行号（0 或 1）。第 1 行保留给重要信息。

![图片](img/box.jpg)  你通过为矩形的位置和大小提供 Python 元组来定义一个 Rect。

![图片](img/box.jpg)  `screen.draw.filled_rect()`函数绘制一个填充的矩形。

![图片](img/box.jpg)  Pygame Zero 中的颜色使用 RGB（红、绿、蓝）格式。例如`(255, 100, 0)`是橙色：最大红色，一点绿色，没有蓝色。

![图片](img/box.jpg)  如果你想在整个程序中替换某些代码，可以使用 IDLE 中的“全部替换”选项。

![图片](img/f0149-01.jpg)
