## **A

ESCAPE: 完整游戏代码**

![image](img/common01.jpg)

本附录展示了 *Escape* 游戏的最终代码。你可以将其作为参考，查看在哪些位置放置特定的函数和代码块，或者如果你想一次性查看完整代码，也可以通读一遍。此代码不包括在构建游戏时编写的临时部分，比如 EXPLORER 部分。它只包含最终游戏中的代码。

请记住，你也可以下载 *escape.py* 文件并在 IDLE 中查看，通过按 CTRL-F 进行搜索。

我已将 PLAYER_NAME 更改为 “Captain” 在此代码中。当你构建或自定义游戏时，你可以使用你自己的名字（参见 Listing 4-1 中的 ➊，位于 第 63 页）。

为了测试这个项目，我使用本书中的说明重新构建了游戏。这个游戏代码来自已经在 Windows、Raspberry Pi 3 Model B+ 和 Raspberry Pi 2 Model B 上测试完成的游戏代码。

# Escape - 一个 Python 冒险游戏

# by Sean McManus / www.sean.co.uk

# Art by Rafael Pimenta

# Typed in by PUT YOUR NAME HERE

import time, random, math

###############

## 变量 ##

###############

WIDTH = 800 # 窗口大小

HEIGHT = 800

#PLAYER 变量

PLAYER_NAME = "Captain" # 将此修改为你的名字！

FRIEND1_NAME = "Karen" # 将此修改为朋友的名字！

FRIEND2_NAME = "Leo" # 将此修改为另一个朋友的名字！

current_room = 31 # 起始房间 = 31

top_left_x = 100

top_left_y = 150

DEMO_OBJECTS = [images.floor, images.pillar, images.soil]

LANDER_SECTOR = random.randint(1, 24)

LANDER_X = random.randint(2, 11)

LANDER_Y = random.randint(2, 11)

TILE_SIZE = 30

player_y，player_x = 2，5

game_over = False

PLAYER = {

"left": [images.spacesuit_left，images.spacesuit_left_1，

images.spacesuit_left_2，images.spacesuit_left_3，

images.spacesuit_left_4

],

"right": [images.spacesuit_right，images.spacesuit_right_1，

images.spacesuit_right_2，images.spacesuit_right_3，

images.spacesuit.right_4

],

"up": [images.spacesuit_back，images.spacesuit_back_1，

images.spacesuit_back_2，images.spacesuit_back_3，

images.spacesuit_back_4

],

"down": [images.spacesuit.front，images.spacesuit.front_1，

images.spacesuit.front_2，images.spacesuit.front_3，

images.spacesuit_front_4

]

}

player_direction = "down"

player_frame = 0

player_image = PLAYER[player_direction][player_frame]

player_offset_x，player_offset_y = 0，0

PLAYER_SHADOW = {

"left": [images.spacesuit_left_shadow，images.spacesuit_left_1_shadow，

images.spacesuit_left_2_shadow，images.spacesuit_left_3_shadow，

images.spacesuit_left_3_shadow

],

"right": [images.spacesuit_right_shadow，images.spacesuit_right_1_shadow，

images.spacesuit_right_2_shadow，

images.spacesuit_right_3_shadow，images.spacesuit_right_3_shadow

],

"up": [images.spacesuit_back_shadow，images.spacesuit_back_1_shadow，

images.spacesuit.back_2_shadow，images.spacesuit.back_3_shadow，

images.spacesuit_back_3_shadow

],

"down": [images.宇航服前面 _ 阴影, images.宇航服前面 _1_ 阴影,

images.宇航服前面 _2_ 阴影, images.宇航服前面 _3_ 阴影,

images.宇航服前面 _3_ 阴影

]

}

player_image_shadow = PLAYER_SHADOW["down"][0]

PILLARS = [

images.柱子, images.柱子 _95, images.柱子 _80,

images.柱子 _60, images.柱子 _50

]

wall_transparency_frame = 0

BLACK = (0, 0, 0)

BLUE = (0, 155, 255)

YELLOW = (255, 255, 0)

WHITE = (255, 255, 255)

GREEN = (0, 255, 0)

RED = (128, 0, 0)

air, energy = 100, 100

suit_stitched, air_fixed = False, False

launch_frame = 0

###############

##    MAP    ##

###############

MAP_WIDTH = 5

MAP_HEIGHT = 10

MAP_SIZE = MAP_WIDTH * MAP_HEIGHT

GAME_MAP = [ ["房间 0 - 存放闲置物品的地方", 0, 0, False, False] ]

outdoor_rooms = range(1, 26)

for planetsectors in range(1, 26): #rooms 1 to 25 are generated here

GAME_MAP.append( ["尘土飞扬的星球表面", 13, 13, True, True] )

GAME_MAP  += [

#["房间名称", 高度, 宽度, 上方出口?, 右方出口?]

["气闸", 13, 5, True, False], # room 26

["工程实验室", 13, 13, False, False], # room 27

["贵宾犬任务控制中心", 9, 13, False, True], # room 28

["观景厅", 9, 15, False, False], # room 29

["船员浴室", 5, 5, False, False], # room 30

["气闸入口区", 7, 11, True, True], # room 31

["左侧肘部房间", 9, 7, True, False], # room 32

["右侧肘部房间", 7, 13, True, True], # room 33

["科学实验室", 13, 13, False, True], # room 34

["温室", 13, 13, True, False], # room 35

[PLAYER_NAME + "的卧室", 9, 11, False, False], # room 36

["西走廊", 15, 5, True, True], # room 37

["简报室", 7, 13, False, True], # room 38

["船员的公共活动室", 11, 13, True, False], # room 39

["主任务控制中心", 14, 14, False, False], # room 40

["医务室", 12, 7, True, False], # room 41

["西走廊", 9, 7, True, False], # room 42

["公共设施控制室", 9, 9, False, True], # room 43

["系统工程区", 9, 11, False, False], # room 44

["通往任务控制中心的安全门", 7, 7, True, False], # room 45

[FRIEND1_NAME + "的卧室", 9, 11, True, True], # room 46

[FRIEND2_NAME + "的卧室", 9, 11, True, True], # room 47

["管道工厂", 13, 11, True, False], # room 48

["首席科学家的办公室", 9, 7, True, True], # room 49

["机器人工作坊", 9, 11, True, False] # room 50

]

#简单的有效性检查，确保数据录入正确

assert len(GAME_MAP)-1 == MAP_SIZE, "地图大小与 GAME_MAP 不匹配"

###############

##  OBJECTS  ##

###############

objects = {

0: [images.地板, None, "地板光滑且干净"],

1: [images.柱子, images.全阴影, "墙壁光滑而冰冷"],

2: [images.土壤, None, "这就像一个沙漠。还是应该说是甜点？"],

3: [images.柱子低, images.半阴影, "墙壁光滑而冰冷"],

4: [images.床, images.半阴影, "一张整洁舒适的床"],

5: [images.桌子, images.半阴影, "它是由强塑料制成的。"],

6: [images.chair_left, None, "一把带有软垫的椅子"],

7: [images.chair_right, None, "一把带有软垫的椅子"],

8: [images.bookcase_tall, images.full_shadow,

"书架，堆满了参考书籍"],

9: [images.bookcase_small, images.half_shadow,

"书架，堆满了参考书籍"],

10: [images.cabinet, images.half_shadow,

"一个小储物柜，用于存放个人物品"],

11: [images.desk_computer, images.half_shadow,

"一台计算机。用来运行生命支持诊断"],

12: [images.plant, images.plant_shadow, "一种太空浆果植物，已在这里生长"],

13: [images.electrical1, images.half_shadow,

"用于为空间站提供电力的电气系统"],

14: [images.electrical2, images.half_shadow,

"用于为空间站提供电力的电气系统"],

15: [images.cactus, images.cactus_shadow, "哎呀！小心仙人掌！"],

16: [images.shrub, images.shrub_shadow,

"一种太空生菜。虽然有点萎靡，但它竟然在这里生长！"],

17: [images.pipes1, images.pipes1_shadow, "水处理管道"],

18: [images.pipes2, images.pipes2_shadow,

"生命支持系统的管道"],

19: [images.pipes3, images.pipes3_shadow,

"生命支持系统的管道"],

20: [images.door, images.door_shadow, "安全门。自动打开\

为穿着太空服的宇航员准备的。"],

21: [images.door, images.door_shadow, "气密门。\

"出于安全原因，需要两人操作。"],

22: [images.door, images.door_shadow, "一扇锁住的门。它需要" \

+ PLAYER_NAME + "的访问卡"],

23: [images.door, images.door_shadow, "一扇锁住的门。它需要" \

+ FRIEND1_NAME + "的访问卡"],

24: [images.door, images.door_shadow, "一扇锁住的门。它需要" \

+ FRIEND2_NAME + "的访问卡"],

25: [images.door, images.door_shadow,

"一扇锁住的门。它从主任务控制室打开"],

26: [images.door, images.door_shadow,

"工程区的一个锁住的门。"],

27: [images.map, images.full_shadow,

"屏幕显示，坠毁地点为：\

+ str(LANDER_SECTOR) + " // X: " + str(LANDER_X) + \

" // Y: " + str(LANDER_Y)],

28: [images.rock_large, images.rock_large_shadow,

"一块石头。它的粗糙表面感觉像是磨刀石", "这块石头"],

29: [images.rock_small, images.rock_small_shadow,

"一块小而重的火星岩"],

30: [images.crater, None, "行星表面上的一个陨石坑"],

31: [images.fence, None,

"一张细网状的围栏。它有助于保护站点免受沙尘暴"],

32: [images.contraption, images.contraption_shadow,

"一个科学实验。它轻微震动"],

33: [images.robot_arm, images.robot_arm_shadow,

"一只机器人手臂，用于重物搬运"],

34: [images.toilet, images.half_shadow, "一间闪闪发光的干净厕所"],

35: [images.sink, None, "一只有自来水的洗手池", "水龙头"]

36: [images.globe, images.globe_shadow,

"一个巨大的行星地球仪。它从内部轻轻发光"],

37: [images.science_lab_table, None,

"一张实验台，分析行星土壤和尘土"],

38: [images.vending_machine, images.full_shadow,  

“一台自动售货机。需要信用卡。”，“自动售货机”],

39: [images.floor_pad, None,

“一个压力传感器，确保没有人单独外出。”],

40: [images.rescue_ship, images.rescue_ship_shadow, “一艘救援船！”],

41: [images.mission_control_desk, images.mission_control_desk_shadow,  

“任务控制站。”],

42: [images.button, images.button_shadow,  

“用于打开工程区时间锁门的按钮。”],

43: [images.whiteboard, images.full_shadow,

“白板用于头脑风暴和规划会议。”],

44: [images.window, images.full_shadow,

“窗户提供了向外看到行星表面的视角。”],

45: [images.robot, images.robot_shadow, “一台关闭的清洁机器人。”],

46: [images.robot2, images.robot2_shadow,

“一台行星表面探索机器人，等待安装。”],

47: [images.rocket, images.rocket_shadow, “一艘正在修理的单人飞行器。”],

48: [images.toxic_floor, None, “有毒地面 - 切勿踩踏！”],

49: [images.drone, None, “一台配送无人机”],

50: [images.energy_ball, None, “一个能量球 - 危险！”],

51: [images.energy_ball2, None, “一个能量球 - 危险！”],

52: [images.computer, images.computer_shadow,

“一台计算机工作站，用于管理空间站系统。”],

53: [images.clipboard, None,

“一块写字板。有人在上面涂鸦。”，“写字板”],

54: [images.bubble_gum, None,

“一块粘性的泡泡糖。太空莓口味。”，“泡泡糖”],

55: [images.yoyo, None, “一款由细而坚固的绳子和塑料制成的玩具。”],

用于反重力实验。”，PLAYER_NAME + "的溜溜球" ],

56: [images.thread, None,

“一根精细而坚固的绳子”，“一根绳子”],

57: [images.needle, None,

“一个仙人掌的尖针”，“一根仙人掌针”],

58: [images.threaded_needle, None,

“一根仙人掌针，穿过一段绳子”，“针和绳子”],

59: [images.canister, None,

“空气罐漏气。”，“一个漏气的空气罐”],

60: [images.canister, None,

“看起来密封能保持！”，“一个密封的空气罐”],

61: [images.mirror, None,

“镜子将光圈投射到墙上。”，“一面镜子”],

62: [images.bin_empty, None,

“一个很少使用的垃圾桶，由轻质塑料制成”，“一个垃圾桶”],

63: [images.bin_full, None,

“一个装满水的重垃圾桶”，“一个装满水的垃圾桶”],

64: [images.rags, None,

“一块油腻的抹布。如果必须捡起，请抓住角落！”，“一块油腻的抹布”],

65: [images.hammer, None,

“一把锤子。也许适合用来砸开东西...”，“一把锤子”],

66: [images.spoon, None, “一个大汤勺”，“一把勺子”],

67: [images.food_pouch, None,

“一个脱水食品袋。需要水。”，“一个干食品包”],

68: [images.food, None,  

“一个食物袋。用它恢复 100%的能量。”，“即食食品”],

69: [images.book, None, “这本书上写着‘别惊慌’”],

书的封面上用大字写着友好的文字，“一本书”],

70: [images.mp3_player, None,

“一台 MP3 播放器，装载着最新的歌曲”，“一台 MP3 播放器”],

71: [images.lander, None, "贵宾犬，一个小型太空探索飞行器。 \

它的黑匣子里有一个无线电被封存其中。", "贵宾犬着陆器"],

72: [images.radio, None, "一个无线电通信系统，来自 \

Poodle", "一个通信无线电"],

73: [images.gps_module, None, "一个 GPS 模块", "一个 GPS 模块"],

74: [images.positioning_system, None, "定位系统的一部分。 \

需要一个 GPS 模块。", "一个定位接口"],

75: [images.positioning_system, None,

"一个工作的定位系统", "一个定位计算机"],

76: [images.scissors, None, "剪刀。它们太钝，无法剪切 \

anything. Can you sharpen them?", "钝的剪刀"],

77: [images.scissors, None,

"锋利的剪刀。小心！", "磨锐的剪刀"],

78: [images.credit, None,

"一个小硬币，用于车站的自动售货系统",

"一张车站信用卡"]

79: [images.access_card, None,

"这张访问卡属于" + PLAYER_NAME, "一张访问卡"],

80: [images.access_card, None,

"这张访问卡属于" + FRIEND1_NAME, "一张访问卡"],

81: [images.access_card, None,

"这张访问卡属于" + FRIEND2_NAME, "一张访问卡"]

}

items_player_may_carry = list(range(53, 82))

# Numbers below are for floor, pressure pad, soil, toxic floor.

items_player_may_stand_on = items_player_may_carry + [0, 39, 2, 48]

###############

##  场景  ##

###############

# Scenery describes objects that cannot move between rooms.

# 房间号: [[物体编号，y 位置，x 位置]...]

scenery = {

26: [[39,8,2]],

27: [[33,5,5], [33,1,1], [33,1,8], [47,5,2],

[47,3,10], [47,9,8], [42,1,6]],

28: [[27,0,3], [41,4,3], [41,4,7]],

29: [[7,2,6], [6,2,8], [12,1,13], [44,0,1],

[36,4,10], [10,1,1], [19,4,2], [17,4,4]],

30: [[34,1,1], [35,1,3]],

31: [[11,1,1], [19,1,8], [46,1,3]],

32: [[48,2,2], [48,2,3], [48,2,4], [48,3,2], [48,3,3],

[48,3,4], [48,4,2], [48,4,3], [48,4,4]],

33: [[13,1,1], [13,1,3], [13,1,8], [13,1,10], [48,2,1],

[48,2,7], [48,3,6], [48,3,3]],

34: [[37,2,2], [32,6,7], [37,10,4], [28,5,3]],

35: [[16,2,9], [16,2,2], [16,3,3], [16,3,8], [16,8,9], [16,8,2], [16,1,8],

[16,1,3], [12,8,6], [12,9,4], [12,9,8],

[15,4,6], [12,7,1], [12,7,11]],

36: [[4,3,1], [9,1,7], [8,1,8], [8,1,9],

[5,5,4], [6,5,7], [10,1,1], [12,1,2]],

37: [[48,3,1], [48,3,2], [48,7,1], [48,5,2], [48,5,3],

[48,7,2], [48,9,2], [48,9,3], [48,11,1], [48,11,2]],

38: [[43,0,2], [6,2,2], [6,3,5], [6,4,7], [6,2,9], [45,1,10]],

39: [[38,1,1], [7,3,4], [7,6,4], [5,3,6], [5,6,6],

[6,3,9], [6,6,9], [45,1,11], [12,1,8], [12,1,4]],

40: [[41,5,3], [41,5,7], [41,9,3], [41,9,7],

[13,1,1], [13,1,3], [42,1,12]],

41: [[4,3,1], [10,3,5], [4,5,1], [10,5,5], [4,7,1],

[10,7,5], [12,1,1], [12,1,5]],

44: [[46,4,3], [46,4,5], [18,1,1], [19,1,3],

[19,1,5], [52,4,7], [14,1,8]],

45: [[48,2,1], [48,2,2], [48,3,3], [48,3,4], [48,1,4], [48,1,1]],

46: [[10,1,1], [4,1,2], [8,1,7], [9,1,8], [8,1,9], [5,4,3], [7,3,2]],

47: [[9,1,1], [9,1,2], [10,1,3], [12,1,7], [5,4,4], [6,4,7], [4,1,8]],

48: [[17,4,1], [17,4,2], [17,4,3], [17,4,4], [17,4,5], [17,4,6], [17,4,7],

[17,8,1], [17,8,2], [17,8,3], [17,8,4],

[17,8,5], [17,8,6], [17,8,7], [14,1,1]],

49: [[14,2,2], [14,2,4], [7,5,1], [5,5,3], [48,3,3], [48,3,4]],

50: [[45,4,8], [11,1,1], [13,1,8], [33,2,1], [46,4,6]]

}

checksum = 0

check_counter = 0

for key, room_scenery_list in scenery.items():

for scenery_item_list in room_scenery_list:

checksum += (scenery_item_list[0] * key

+ scenery_item_list[1] * (key + 1)

+ scenery_item_list[2] * (key + 2))

check_counter += 1

print(check_counter, "景观项")

assert check_counter == 161, "预期 161 个景观项"

assert checksum == 200095, "景观数据错误"

print("景观校验和: " + str(checksum))

for room in range(1, 26): # 在行星位置添加随机景观。

if room != 13: # 跳过房间 13。

scenery_item = random.choice([16, 28, 29, 30])

scenery[room] = [[scenery_item, random.randint(2, 10),

random.randint(2, 10)]]

# 使用循环将围栏添加到行星表面房间。

for room_coordinate in range(0, 13):

for room_number in [1, 2, 3, 4, 5]: # 添加顶部围栏

scenery[room_number] += [[31, 0, room_coordinate]]

for room_number in [1, 6, 11, 16, 21]: # 添加左侧围栏

scenery[room_number] += [[31, room_coordinate, 0]]

for room_number in [5, 10, 15, 20, 25]: # 添加右侧围栏

scenery[room_number] += [[31, room_coordinate, 12]]

del scenery[21][-1] # 删除房间 21 中的最后一块围栏面板

del scenery[25][-1] # 删除房间 25 中的最后一块围栏面板

###############

## 制作地图##

###############

def get_floor_type():

if current_room in outdoor_rooms:

return 2 # 土壤

else:

return 0 # 瓷砖地板

def generate_map():

# 该函数用于为当前房间生成地图，

# 使用房间数据、景观数据和道具数据。

global room_map, room_width, room_height, room_name, hazard_map

global top_left_x, top_left_y, wall_transparency_frame

room_data = GAME_MAP[current_room]

room_name = room_data[0]

room_height = room_data[1]

room_width = room_data[2]

floor_type = get_floor_type()

if current_room in range(1, 21):

bottom_edge = 2 #土壤

side_edge = 2 #土壤

if current_room in range(21, 26):

bottom_edge = 1 #墙壁

side_edge = 2 #土壤

if current_room > 25:

bottom_edge = 1 #墙壁

side_edge = 1 #墙壁

# 创建房间地图的顶部边界。

room_map=[[side_edge] * room_width]

# 添加房间地图的中间行（墙壁，地板填充宽度，墙壁）。

for y in range(room_height - 2):

room_map.append([side_edge]

+ [floor_type]*(room_width - 2) + [side_edge])

# 添加房间地图的底部边界。

room_map.append([bottom_edge] * room_width)

# 添加门。

middle_row = int(room_height / 2)

middle_column = int(room_width / 2)

if room_data[4]: # 如果此房间右侧有出口

room_map[middle_row][room_width - 1] = floor_type

room_map[middle_row+1][room_width - 1] = floor_type

room_map[middle_row-1][room_width - 1] = floor_type

if current_room % MAP_WIDTH != 1: # 如果房间不在地图左侧

room_to_left = GAME_MAP[current_room - 1]

# 如果左侧房间有右出口，则在此房间添加左出口

if room_to_left[4]:

room_map[middle_row][0] = floor_type

room_map[middle_row + 1][0] = floor_type

room_map[middle_row - 1][0] = floor_type

if room_data[3]: # 如果房间顶部有出口

room_map[0][middle_column] = floor_type

room_map[0][middle_column + 1] = floor_type

room_map[0][middle_column - 1] = floor_type

if current_room <= MAP_SIZE - MAP_WIDTH: # 如果房间不在最底行

room_below = GAME_MAP[current_room+MAP_WIDTH]

# 如果下方房间有上方出口，则在当前房间底部添加出口

if room_below[3]:

room_map[room_height-1][middle_column] = floor_type

room_map[room_height-1][middle_column + 1] = floor_type

room_map[room_height-1][middle_column - 1] = floor_type

if current_room in scenery:

for this_scenery in scenery[current_room]:

scenery_number = this_scenery[0]

scenery_y = this_scenery[1]

scenery_x = this_scenery[2]

room_map[scenery_y][scenery_x] = scenery_number

image_here = objects[scenery_number][0]

image_width = image_here.get_width()

image_width_in_tiles = int(image_width / TILE_SIZE)

for tile_number in range(1, image_width_in_tiles):

room_map[scenery_y][scenery_x + tile_number] = 255

center_y = int(HEIGHT / 2) # 游戏窗口的中心

center_x = int(WIDTH / 2)

room_pixel_width = room_width * TILE_SIZE # 房间的像素大小

room_pixel_height = room_height * TILE_SIZE

top_left_x = center_x - 0.5 * room_pixel_width

top_left_y = (center_y - 0.5 * room_pixel_height) + 110

for prop_number, prop_info in props.items():

prop_room = prop_info[0]

prop_y = prop_info[1]

prop_x = prop_info[2]

if (prop_room == current_room and

room_map[prop_y][prop_x] in [0, 39, 2]):

room_map[prop_y][prop_x] = prop_number

image_here = objects[prop_number][0]

image_width = image_here.get_width()

image_width_in_tiles = int(image_width / TILE_SIZE)

for tile_number in range(1, image_width_in_tiles):

room_map[prop_y][prop_x + tile_number] = 255

hazard_map = [] # 空列表

for y in range(room_height):

hazard_map.append( [0] * room_width )

###############

## GAME LOOP ##

###############

def start_room():

global airlock_door_frame

show_text("你当前所在房间: " + room_name, 0)

if current_room == 26: # 自闭气闸门的房间

airlock_door_frame = 0

clock.schedule_interval(door_in_room_26, 0.05)

hazard_start()

def game_loop():

global player_x, player_y, current_room

global from_player_x, from_player_y

global player_image, player_image_shadow

global selected_item, item_carrying, energy

global player_offset_x, player_offset_y

global player_frame, player_direction

if game_over:

return

if player_frame > 0:

player_frame += 1

time.sleep(0.05)

if player_frame == 5:

player_frame = 0

player_offset_x = 0

player_offset_y = 0

# save player's current position

old_player_x = player_x

old_player_y = player_y

# move if key is pressed

if player_frame == 0:

if keyboard.right:

from_player_x = player_x

from_player_y = player_y

player_x += 1

player_direction = "right"

player_frame = 1

elif keyboard.left: # elif 阻止玩家做出对角移动

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

# 检查是否退出房间

if player_x == room_width: # 从右侧门进入

clock.unschedule(hazard_move)

current_room += 1

generate_map()

player_x = 0 # 从左侧进入

player_y = int(room_height / 2) # 从门口进入

player_frame = 0

start_room()

return

if player_x == -1: # 从左侧门进入

clock.unschedule(hazard_move)

current_room -= 1

generate_map()

player_x = room_width - 1  # 从右侧进入

player_y = int(room_height / 2) # 从门口进入

player_frame = 0

start_room()

return

if player_y == room_height: # 从底部门进入

clock.unschedule(hazard_move)

current_room += MAP_WIDTH

generate_map()

player_y = 0 # 从顶部进入

player_x = int(room_width / 2) # 从门口进入

player_frame = 0

start_room()

return

if player_y == -1: # 从顶部门进入

clock.unschedule(hazard_move)

current_room -= MAP_WIDTH

generate_map()

player_y = room_height - 1 # 从底部进入

player_x = int(room_width / 2) # 从门口进入

player_frame = 0

start_room()

return

if keyboard.g:

捡起物品()

if keyboard.tab and len(in_my_pockets) > 0:

selected_item += 1

if selected_item > len(in_my_pockets) - 1:

selected_item = 0

item_carrying = in_my_pockets[selected_item]

显示背包()

if keyboard.d and item_carrying:

drop_object(old_player_y, old_player_x)

if keyboard.space:

examine_object()

if keyboard.u:

使用物品()

#### 用于测试的传送门

#### 移除这一部分以进行实际游戏

##    if keyboard.x:

##        current_room = int(input("请输入房间号:"))

##        player_x = 2

##        player_y = 2

##        generate_map()

##        start_room()

##        sounds.teleport.play()

#### 传送部分结束

# 如果玩家站在不该站的位置，将其移回

if room_map[player_y][player_x] not in items_player_may_stand_on \

or hazard_map[player_y][player_x] != 0:

player_x = old_player_x

player_y = old_player_y

player_frame = 0

if room_map[player_y][player_x] == 48: # 有毒地板

消耗能量(1)

if player_direction == "right" and player_frame > 0:

player_offset_x = -1 + (0.25 * player_frame)

if player_direction == "left" and player_frame > 0:

player_offset_x = 1 - (0.25 * player_frame)

if player_direction == "up" and player_frame > 0:

player_offset_y = 1 - (0.25 * player_frame)

if player_direction == "down" and player_frame > 0:

player_offset_y = -1 + (0.25 * player_frame)

###############

##  显示  ##

###############

def draw_image(image, y, x):

screen.blit(

image,

(top_left_x + (x * TILE_SIZE),

top_left_y + (y * TILE_SIZE) - image.get_height())

)

def draw_shadow(image, y, x):

screen.blit(

image,

(top_left_x + (x * TILE_SIZE),

top_left_y + (y * TILE_SIZE))

)

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

# 清除游戏竞技场区域。

box = Rect((0, 150), (800, 600))

screen.draw.filled_rect(box, RED)

box = Rect ((0, 0), (800, top_left_y + (room_height - 1)*30))

screen.surface.set_clip(box)

floor_type = get_floor_type()

for y in range(room_height): # 铺设地板瓷砖，然后铺设地面物品。

for x in range(room_width):

draw_image(objects[floor_type][0], y, x)

# 下一行使阴影能够落在地面物体上方

if room_map[y][x] in items_player_may_stand_on:

draw_image(objects[room_map[y][x]][0], y, x)

# 房间 26 中的压力垫在此处添加，以便道具可以放置在其上。

if current_room == 26:

draw_image(objects[39][0], 8, 2)

image_on_pad = room_map[8][2]

if image_on_pad > 0:

draw_image(objects[image_on_pad][0], 8, 2)

for y in range(room_height):

for x in range(room_width):

item_here = room_map[y][x]

# 玩家不能走到 255 位置：它标记了宽物体占据的空间。

if item_here not in items_player_may_stand_on + [255]:

image = objects[item_here][0]

if (current_room in outdoor_rooms

and y == room_height - 1

and room_map[y][x] == 1) or \

(current_room not in outdoor_rooms

and y == room_height - 1

and room_map[y][x] == 1

and x > 0

and x < room_width - 1):

# 在前排添加透明墙壁图像。

image = PILLARS[wall_transparency_frame]

draw_image(image, y, x)

if objects[item_here][1] is not None: # 如果物体有阴影

shadow_image = objects[item_here][1]

# 如果阴影可能需要水平平铺

if shadow_image in [images.half_shadow,

images.full_shadow]:

shadow_width = int(image.get_width() / TILE_SIZE)

# 使用跨越物体宽度的阴影。

for z in range(0, shadow_width):

draw_shadow(shadow_image, y, x+z)

else:

draw_shadow(shadow_image, y, x)

hazard_here = hazard_map[y][x]

if hazard_here != 0: # 如果此位置有危险

draw_image(objects[hazard_here][0], y, x)

if (player_y == y):

draw_player()

screen.surface.set_clip(None)

def adjust_wall_transparency():

global wall_transparency_frame

if (player_y == room_height - 2

and room_map[room_height - 1][player_x] == 1

and wall_transparency_frame < 4):

wall_transparency_frame += 1 # 墙壁渐变消失。

if ((player_y < room_height - 2

or room_map[room_height - 1][player_x] != 1)

and wall_transparency_frame > 0):

wall_transparency_frame -= 1 # 墙壁渐变出现。

def show_text(text_to_show, line_number):

if game_over:

return

text_lines = [15, 50]

box = Rect((0, text_lines[line_number]), (800, 35))

screen.draw.filled_rect(box, BLACK)

screen.draw.text(text_to_show,

(20, text_lines[line_number]), color=GREEN)

###############

##   道具   ##

###############

# 道具是可能在房间间移动、出现或消失的物体。

# 所有道具必须在这里设置。游戏中尚未出现的道具进入房间 0。

# object number : [room, y, x]

props = {

20: [31, 0, 4], 21: [26, 0, 1], 22: [41, 0, 2], 23: [39, 0, 5],

24: [45, 0, 2],

25: [32, 0, 2], 26: [27, 12, 5], # 同一扇门的两侧

40: [0, 8, 6], 53: [45, 1, 5], 54: [0, 0, 0], 55: [0, 0, 0],

56: [0, 0, 0], 57: [35, 4, 6], 58: [0, 0, 0], 59: [31, 1, 7],

60: [0, 0, 0], 61: [36, 1, 1], 62: [36, 1, 6], 63: [0, 0, 0],

64: [27, 8, 3], 65: [50, 1, 7], 66: [39, 5, 6], 67: [46, 1, 1],

68: [0, 0, 0], 69: [30, 3, 3], 70: [47, 1, 3],

71: [0, LANDER_Y, LANDER_X], 72: [0, 0, 0], 73: [27, 4, 6],

74: [28, 1, 11], 75: [0, 0, 0], 76: [41, 3, 5], 77: [0, 0, 0],

78: [35, 9, 11], 79: [26, 3, 2], 80: [41, 7, 5], 81: [29, 1, 1]

}

checksum = 0

for key, prop in props.items():

if key != 71: # 71 被跳过，因为每局游戏都不同。

checksum += (prop[0] * key

+ prop[1] * (key + 1)

+ prop[2] * (key + 2))

print(len(props), "道具")

assert len(props) == 37, "预期有 37 个道具"

print("道具校验和:", checksum)

assert checksum == 61414, "道具数据错误"

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

print(len(RECIPES), "配方")

assert len(RECIPES) == 11, "预期有 11 个配方"

assert checksum == 37296, "配方数据错误"

print("配方校验和:", checksum)

#######################

## PROP INTERACTIONS ##

#######################

def find_object_start_x():

checker_x = player_x

while room_map[player_y][checker_x] == 255:

checker_x -= 1

return checker_x

def get_item_under_player():

item_x = find_object_start_x()

item_player_is_on = room_map[player_y][item_x]

return item_player_is_on

def pick_up_object():

global room_map

# 获取玩家位置的物品编号。

item_player_is_on = get_item_under_player()

if item_player_is_on in items_player_may_carry:

# 清除地面空间。

room_map[player_y][player_x] = get_floor_type()

add_object(item_player_is_on)

show_text("现在携带 " + objects[item_player_is_on][3], 0)

sounds.pickup.play()

time.sleep(0.5)

else:

show_text("你无法携带这个！", 0)

def add_object(item): # 将物品添加到背包中。

global selected_item, item_carrying

in_my_pockets.append(item)

item_carrying = item

# 减一因为索引从 0 开始。

selected_item = len(in_my_pockets) - 1

display_inventory()

props[item][0] = 0 # 携带的物品进入房间 0（地图外）。

def display_inventory():

box = Rect((0, 45), (800, 105))

screen.draw.filled_rect(box, BLACK)

if len(in_my_pockets) == 0:

return

start_display = (selected_item // 16) * 16

list_to_show = in_my_pockets[start_display : start_display + 16]

selected_marker = selected_item % 16

for item_counter in range(len(list_to_show)):

item_number = list_to_show[item_counter]

image = objects[item_number][0]

screen.blit(image, (25 + (46 * item_counter), 90))

box_left = (selected_marker * 46) - 3

box = Rect((22 + box_left, 85), (40, 40))

screen.draw.rect(box, WHITE)

item_highlighted = in_my_pockets[selected_item]

description = objects[item_highlighted][2]

screen.draw.text(description, (20, 130), color="white")

def drop_object(old_y, old_x):

global room_map, props

if room_map[old_y][old_x] in [0, 2, 39]: # 可以丢东西的地方

props[item_carrying][0] = current_room

props[item_carrying][1] = old_y

props[item_carrying][2] = old_x

room_map[old_y][old_x] = item_carrying

show_text("你丢下了 " + objects[item_carrying][3], 0)

sounds.drop.play()

remove_object(item_carrying)

time.sleep(0.5)

else: # 如果这里已经有物品，则执行这个

show_text("你不能把它丢在这里。", 0)

time.sleep(0.5)

def remove_object(item): # 从背包中移除物品

global selected_item, in_my_pockets, item_carrying

in_my_pockets.remove(item)

selected_item = selected_item - 1

if selected_item < 0:

selected_item = 0

if len(in_my_pockets) == 0: # 如果背包中没有物品

item_carrying = False # 将 item_carrying 设置为 False

else: # 否则，将其设置为新选择的物品

item_carrying = in_my_pockets[selected_item]

display_inventory()

def examine_object():

item_player_is_on = get_item_under_player()

left_tile_of_item = find_object_start_x()

if item_player_is_on in [0, 2]: # 不描述地板

return

description = "你看到： " + objects[item_player_is_on][2]

for prop_number, details in props.items():

# props = 物品编号：[房间编号，y，x]

if details[0] == current_room: # 如果物品在房间内

# 如果物品是隐藏的（即在玩家位置但不在地图上）

if (details[1] == player_y

和 details[2] == left_tile_of_item

和 room_map[details[1]][details[2]] != prop_number):

add_object(prop_number)

description = "你找到了 " + objects[prop_number][3]

sounds.combine.play()

show_text(description, 0)

time.sleep(0.5)

#################

## 使用物品 ##

#################

def use_object():

global room_map, props, item_carrying, air, selected_item, energy

global in_my_pockets, suit_stitched, air_fixed, game_over

use_message = "你摆弄了一阵子，但没有什么进展。"

standard_responses = {

4: "空气快用完了！你不能坐视不管！",

6: "现在不是坐着不动的时候！",

7: "现在不是坐着不动的时候！",

32: "它摇晃并发出轰鸣声，但什么也没发生。",

34: "啊！好多了。现在洗洗手吧。",

35: "你洗了洗手，并把水甩掉。",

37: "试管在你摇晃时微微冒烟。",

54: "你嚼了口口香糖，黏黏的像胶水。",

55: "溜溜球上下弹跳，速度比地球上稍慢",

56: "有点麻烦。你能把它穿到什么东西上去吗？",

59: "你需要修理泄漏才能使用罐子。",

61: "你试着用镜子示意，但没有人能看到你。",

62: "别把资源浪费掉，东西可能会派上用场...",

67: "想享受美味的太空食物，只需加水！",

75: "你位于区域: " + str(current_room) + " // X: " \

+ str(player_x) + " // Y: " + str(player_y)

}

# 获取玩家位置的物品编号。

item_player_is_on = get_item_under_player()

for this_item in [item_player_is_on, item_carrying]:

if this_item in standard_responses:

use_message = standard_responses[this_item]

if item_carrying == 70 or item_player_is_on == 70:

use_message = "震撼的音乐！"

sounds.steelmusic.play(2)

elif item_player_is_on == 11:

use_message = "空气: " + str(air) + \

"% / 能量 " + str(energy) + "% / "

if not suit_stitched:

use_message += "*警告* 套装面料撕裂 / "

if not air_fixed:

use_message += "*警告* 套装空气瓶丢失"

if suit_stitched and air_fixed:

use_message += " 套装正常"

show_text(use_message, 0)

sounds.say_status_report.play()

time.sleep(0.5)

# 如果玩家“在”计算机上，玩家的意图显然是更新状态。

# 返回以防止其他物品使用意外覆盖此项。

return

elif item_carrying == 60 or item_player_is_on == 60:

use_message = "你将" + objects[60][3] + "修复到套装上"

air_fixed = True

air = 90

air_countdown()

remove_object(60)

elif (item_carrying == 58 or item_player_is_on == 58) \

and not suit_stitched:

use_message = "你使用了" + objects[56][3] + \

" 修复套装面料"

suit_stitched = True

remove_object(58)

elif item_carrying == 72 or item_player_is_on == 72:

use_message = "你呼叫了救援。救援船正在来。\

会合区域 13，外部。"

props[40][0] = 13

elif (item_carrying == 66 or item_player_is_on == 66) \

and current_room in outdoor_rooms:

use_message = "你开始挖掘..."

if (current_room == LANDER_SECTOR

and player_x == LANDER_X

and player_y == LANDER_Y):

add_object(71)

use_message = "你找到了贵宾犬着陆器！"

elif item_player_is_on == 40:

clock.unschedule(air_countdown)

show_text("恭喜，"+ PLAYER_NAME +"!", 0)

show_text("任务成功！你已经安全到达。", 1)

game_over = True

sounds.take_off.play()

game_completion_sequence()

elif item_player_is_on == 16:

energy += 1

if energy > 100:

energy = 100

use_message = "你吃了生菜，恢复了一些能量"

draw_energy_air()

elif item_player_is_on == 42:

if current_room == 27:

open_door(26)

props[25][0] = 0 # 从 RM32 到工程舱的门

props[26][0] = 0 # 工程舱内的门

clock.schedule_unique(shut_engineering_door, 60)

use_message = "你按下了按钮"

show_text("通往工程舱的门已打开 60 秒", 1)

sounds.say_doors_open.play()

sounds.doors.play()

elif item_carrying == 68 or item_player_is_on == 68:

energy = 100

use_message = "你使用食物恢复了能量"

remove_object(68)

draw_energy_air()

if suit_stitched and air_fixed: # 打开气闸门

if current_room == 31 and props[20][0] == 31:

open_door(20) # 这包括移除门

sounds.say_airlock_open.play()

show_text("计算机告诉你气闸门现在已打开。", 1)

elif props[20][0] == 31:

props[20][0] = 0 # 从地图中移除门

sounds.say_airlock_open.play()

show_text("计算机告诉你气闸现在已开启。", 1)

for recipe in RECIPES:

ingredient1 = recipe[0]

ingredient2 = recipe[1]

combination = recipe[2]

if (item_carrying == ingredient1

and item_player_is_on == ingredient2) \

or (item_carrying == ingredient2

and item_player_is_on == ingredient1):

use_message = "你将 " + objects[ingredient1][3] \

+ " 和 " + objects[ingredient2][3] \

+ " 来制造 " + objects[combination][3]

if item_player_is_on in props.keys():

props[item_player_is_on][0] = 0

room_map[player_y][player_x] = get_floor_type()

in_my_pockets.remove(item_carrying)

add_object(combination)

sounds.combine.play()

# {关键物体编号: 门物体编号}

ACCESS_DICTIONARY = { 79:22, 80:23, 81:24 }

if item_carrying in ACCESS_DICTIONARY:

door_number = ACCESS_DICTIONARY[item_carrying]

if props[door_number][0] == current_room:

use_message = "你解锁了门！"

sounds.say_doors_open.play()

sounds.doors.play()

open_door(door_number)

show_text(use_message, 0)

time.sleep(0.5)

def game_completion_sequence():

global launch_frame #(初始值为 0，在 VARIABLES 部分设置)

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

##   门   ##

###############

def open_door(opening_door_number):

global door_frames, door_shadow_frames

global door_frame_number, door_object_number

door_frames = [images.door1, images.door2, images.door3,

images.door4, images.floor]

# （最终帧恢复阴影，准备门重新出现时）。

door_shadow_frames = [images.door1_shadow, images.door2_shadow,

images.door3_shadow, images.door4_shadow,

images.door_shadow]

door_frame_number = 0

door_object_number = opening_door_number

do_door_animation()

def close_door(closing_door_number):

global door_frames, door_shadow_frames

global door_frame_number, door_object_number, player_y

door_frames = [images.door4, images.door3, images.door2,

images.door1, images.door]

door_shadow_frames = [images.door4_shadow, images.door3_shadow,

images.door2_shadow, images.door1_shadow,

images.door_shadow]

door_frame_number = 0

door_object_number = closing_door_number

# 如果玩家在与门同一行，他们必须在开启的门口

if player_y == props[door_object_number][1]:

if player_y == 0: # 如果在顶部门口

player_y = 1  # 将玩家移到下方

else:

player_y = room_height - 2  # 将玩家移到上方

do_door_animation()

def do_door_animation():

global door_frames, door_frame_number, door_object_number, objects

objects[door_object_number][0] = door_frames[door_frame_number]

objects[door_object_number][1] = door_shadow_frames[door_frame_number]

door_frame_number += 1

if door_frame_number == 5:

if door_frames[-1] == images.floor:

props[door_object_number][0] = 0  # 从 props 列表中移除门

# 从 props 重新生成房间地图

# 如有必要，将门放入房间。

generate_map()

else:

clock.schedule(do_door_animation, 0.15)

def shut_engineering_door():

global current_room, door_room_number, props

props[25][0] = 32  # 从 32 号房间到工程区的门。

props[26][0] = 27  # 工程区内部的门。

generate_map()  # 在受影响的房间中添加门到房间地图。

if current_room == 27:

close_door(26)

if current_room == 32:

close_door(25)

show_text("电脑告诉你门已关闭。", 1)

sounds.say_doors_closed.play()

def door_in_room_26():

global airlock_door_frame, room_map

frames = [images.door, images.door1, images.door2,

images.door3,images.door4, images.floor

]

shadow_frames = [images.door_shadow, images.door1_shadow,

images.door2_shadow, images.door3_shadow,

images.door4_shadow, None]

if current_room != 26:

clock.unschedule(door_in_room_26)

return

# prop 21 是 26 号房间的门。

if ((player_y == 8 and player_x == 2) or props[63] == [26, 8, 2]) \

and props[21][0] == 26:

airlock_door_frame += 1

if airlock_door_frame == 5:

props[21][0] = 0  # 当门完全打开时，从地图中移除门。

room_map[0][1] = 0

room_map[0][2] = 0

room_map[0][3] = 0

if ((player_y != 8 or player_x != 2) and props[63] != [26, 8, 2]) \

and airlock_door_frame > 0:

if airlock_door_frame == 5:

# 将门添加到 props 和地图中，以便显示动画。

props[21][0] = 26

room_map[0][1] = 21

room_map[0][2] = 255

room_map[0][3] = 255

airlock_door_frame -= 1

objects[21][0] = frames[airlock_door_frame]

objects[21][1] = shadow_frames[airlock_door_frame]

###############

##    AIR    ##

###############

def draw_energy_air():

box = Rect((20, 765), (350, 20))

screen.draw.filled_rect(box, BLACK)

screen.draw.text("空气", (20, 766), color=BLUE)

screen.draw.text("能量", (180, 766), color=YELLOW)

if air > 0:

box = Rect((50, 765), (air, 20))

screen.draw.filled_rect(box, BLUE)  # 绘制新的空气条。

if energy > 0:

box = Rect((250, 765), (energy, 20))

screen.draw.filled_rect(box, YELLOW)  # 绘制新的能量条。

def end_the_game(reason):

global game_over

show_text(reason, 1)

game_over = True

sounds.say_mission_fail.play()

sounds.gameover.play()

screen.draw.text("游戏结束", (120, 400), color = "white",

fontsize = 128, shadow = (1, 1), scolor = "black")

def air_countdown():

global air, game_over

if game_over:

return  # 当玩家已经死亡时，不消耗空气。

air -= 1

if air == 20:

sounds.say_air_low.play()

if air == 10:

sounds.say_act_now.play()

draw_energy_air()

if air < 1:

end_the_game("你没有空气了！")

def alarm():

show_text("空气快用完了，" + PLAYER_NAME

+ "! 快去安全地方，然后呼叫帮助！", 1)

sounds.alarm.play(3)

sounds.say_breach.play()

###############

##  危险物体  ##

###############

hazard_data = {

# 房间编号: [[y, x, 方向, 方向的反弹增量]]

28: [[1, 8, 2, 1], [7, 3, 4, 1]], 32: [[1, 5, 4, -1]],

34: [[5, 1, 1, 1], [5, 5, 1, 2]], 35: [[4, 4, 1, 2], [2, 5, 2, 2]],

36: [[2, 1, 2, 2]], 38: [[1, 4, 3, 2], [5, 8, 1, 2]],

40: [[3, 1, 3, -1], [6, 5, 2, 2], [7, 5, 4, 2]],

41: [[4, 5, 2, 2], [6, 3, 4, 2], [8, 1, 2, 2]],

42: [[2, 1, 2, 2], [4, 3, 2, 2], [6, 5, 2, 2]],

46: [[2, 1, 2, 2]],

48: [[1, 8, 3, 2], [8, 8, 1, 2], [3, 9, 3, 2]]

}

def deplete_energy(penalty):

global energy, game_over

如果 game_over:

返回 # 当它们已经死亡时，不消耗能量。

energy = energy - penalty

draw_energy_air()

如果能量 < 1:

end_the_game("你的能量耗尽了！")

def hazard_start():

global current_room_hazards_list, hazard_map

如果当前房间在 hazard_data 的键中:

current_room_hazards_list = hazard_data[current_room]

对于 current_room_hazards_list 中的 hazard:

hazard_y = hazard[0]

hazard_x = hazard[1]

hazard_map[hazard_y][hazard_x] = 49 + (current_room % 3)

clock.schedule_interval(hazard_move, 0.15)

def hazard_move():

global current_room_hazards_list, hazard_data, hazard_map

global old_player_x, old_player_y

如果 game_over:

返回

对于 current_room_hazards_list 中的 hazard:

hazard_y = hazard[0]

hazard_x = hazard[1]

hazard_direction = hazard[2]

old_hazard_x = hazard_x

old_hazard_y = hazard_y

hazard_map[old_hazard_y][old_hazard_x] = 0

如果 hazard_direction == 1: # 上

hazard_y -= 1

如果 hazard_direction == 2: # 右

hazard_x += 1

如果 hazard_direction == 3: # 下

hazard_y += 1

如果 hazard_direction == 4: # 左

hazard_x -= 1

hazard_should_bounce = False

如果 (hazard_y == player_y 和 hazard_x == player_x) 或者 \

(hazard_y == from_player_y 和 hazard_x == from_player_x

和 player_frame > 0):

sounds.ouch.play()

deplete_energy(10)

hazard_should_bounce = True

# 停止危险物体出门

如果 hazard_x == room_width:

hazard_should_bounce = True

hazard_x = room_width - 1

如果 hazard_x == -1:

hazard_should_bounce = True

hazard_x = 0

如果 hazard_y == room_height:

hazard_should_bounce = True

hazard_y = room_height - 1

如果 hazard_y == -1:

hazard_should_bounce = True

hazard_y = 0

# 当危险物体碰到景物或其他危险物体时停止。

如果 room_map[hazard_y][hazard_x] 不在 items_player_may_stand_on 中 \

或者 hazard_map[hazard_y][hazard_x] != 0:

hazard_should_bounce = True

如果 hazard_should_bounce:

hazard_y = old_hazard_y # 移动回最后一个有效位置。

hazard_x = old_hazard_x

hazard_direction += hazard[3]

如果 hazard_direction > 4:

hazard_direction -= 4

如果 hazard_direction < 1:

hazard_direction += 4

hazard[2] = hazard_direction

hazard_map[hazard_y][hazard_x] = 49 + (current_room % 3)

hazard[0] = hazard_y

hazard[1] = hazard_x

###############

##   开始   ##

###############

clock.schedule_interval(game_loop, 0.03)

generate_map()

clock.schedule_interval(adjust_wall_transparency, 0.05)

clock.schedule_unique(display_inventory, 1)

clock.schedule_unique(draw_energy_air, 0.5)

clock.schedule_unique(alarm, 10)

# 下方的更大数字会设置更长的时间限制。

clock.schedule_interval(air_countdown, 5)

sounds.mission.play() # 介绍音乐
