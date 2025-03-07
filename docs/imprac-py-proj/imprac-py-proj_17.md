## 实践项目解决方案

![image](../images/common01.jpg)

本附录提供了每一章实践项目的解决方案。数字版本可在本书网站上获得，网址为*[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)*。

### **第1章：傻名字生成器**

#### ***猪拉丁语***

*pig_Latin_practice.py*

"""将一个单词转换为其猪拉丁语等价词。"""

import sys

VOWELS = 'aeiouy'

while True:

word = input("输入一个单词，获取其猪拉丁语翻译：")

if word[0] in VOWELS:

pig_Latin = word + 'way'

else:

pig_Latin = word[1:] + word[0] + 'ay'

print()

print("{}".format(pig_Latin), file=sys.stderr)

try_again = input("\n\n再试一次吗？（按回车键，或者按n停止）\n ")

if try_again.lower() == "n":

sys.exit()

#### ***穷人的条形图***

*EATOIN_practice.py*

"""将字母映射到字典中，并打印频率的条形图。"""

import sys

import pprint

from collections import defaultdict

# 注意：文本应该是简短的短语，以便条形图适应IDLE窗口

text = '就像中世纪游戏角落里的城堡一样，我预见到可怕的 \

trouble and I stay here just the same.'

ALPHABET = 'abcdefghijklmnopqrstuvwxyz'

# defaultdict模块让你动态生成字典键！

mapped = defaultdict(list)

for character in text:

character = character.lower()

if character in ALPHABET:

mapped[character].append(character)

# pprint可以让你打印堆叠输出

print("\n如果文本换行，请可能需要拉伸控制台窗口。\n")

print("text = ", end='')

print("{}\n".format(text), file=sys.stderr)

pprint.pprint(mapped, width=110)

### **第2章：寻找回文拼写**

#### ***字典清理***

*dictionary_cleanup_practice.py*

"""如果不是'a'或'i'，从列表中移除单字母单词。"""

word_list = ['a', 'nurses', 'i', 'stack', 'b', 'c', 'cat']

word_list_clean = []

permissible = ('a', 'i')

for word in word_list:

if len(word) > 1:

word_list_clean.append(word)

elif len(word) == 1 and word in permissible:

word_list_clean.append(word)

else:

continue

print("{}".format(word_list_clean))

### **第3章：解决字谜**

#### ***查找字母对***

*count_digrams_practice.py*

"""生成伏地魔的字母对，并查找它们在字典中的频率。

需要load_dictionary.py模块来加载英文词典文件。

"""

import re

from collections import defaultdict

from itertools import permutations

import load_dictionary

word_list = load_dictionary.load('2of4brif.txt')

name = 'Voldemort'  #(tmvoordle)

name = name.lower()

# 生成名字的唯一字母对

digrams = set()

perms = {''.join(i) for i in permutations(name)}

for perm in perms:

for i in range(0, len(perm) - 1):

digrams.add(perm[i] + perm[i + 1])

print(*sorted(digrams), sep='\n')

print("\n字母对数量 = {}\n".format(len(digrams)))

# 使用正则表达式查找单词中重复的字母对

mapped = defaultdict(int)

for word in word_list:

word = word.lower()

for digram in digrams:

for m in re.finditer(digram, word):

mapped[digram] += 1

print("双字频率计数：")

count = 0

for k in sorted(mapped):

print("{} {}".format(k, mapped[k]))

### **第4章：解码美国内战密码**

#### ***破解林肯密码***

| **代码词** | **明文** |
| --- | --- |
| WAYLAND | captured |
| NEPTUNE | Richmond |

**明文：** 请确认在里士满被捕的《论坛报》记者为何被拘留，如果可以，请将其释放

#### ***识别密码类型***

*identify_cipher_type_practice.py*

"""加载密文并使用ETAOIN出现的分数来分类密码类型。"""

import sys

from collections import Counter

# 设置英语中最常见字母的6个字母的任意截止分数

# 密文中包含目标分数或更高的部分 = 换位密码

CUTOFF = 0.5

# 加载密文

def load(filename):

"""打开文本文件并返回列表。"""

with open(filename) as f:

return f.read().strip()

try:

ciphertext = load('cipher_a.txt')

except IOError as e:

print("{}. 程序终止.".format(e),

file=sys.stderr)

sys.exit(1)

# count 6 个最常见字母在密文中

six_most_frequent = Counter(ciphertext.lower()).most_common(6)

print("\n英语中最常用的六个字母 = ETAOIN")

print('\n密文中最常见的六个字母 =')

print(*six_most_frequent, sep='\n')

# 将元组列表转换为字母集进行比较

cipher_top_6 = {i[0] for i in six_most_frequent}

TARGET = 'etaoin'

count = 0

for letter in TARGET:

if letter in cipher_top_6:

count += 1

if count/len(TARGET) >= CUTOFF:

print("\n此密文很可能是由换位密码加密生成的")

else:

print("此密文很可能是由替代密码加密生成的")

#### ***将键存储为字典***

*key_dictionary_practice.py*

"""输入密码键字符串，获取用户输入的路由方向作为字典值。"""

col_order = """1 3 4 2"""

key = dict()

cols = [int(i) for i in col_order.split()]

for col in cols:

while True:

key[col] = input("读取列 {} 的方向 (u = 上, d = 下): "

.format(col).lower())

if key[col] == 'u' or key[col] == 'd':

break

else:

print("输入应为 'u' 或 'd'")

print("{}, {}".format(col, key[col]))

#### ***自动化可能的键***

*permutations_practice.py*

"""对于总列数，找出所有独特的列排列。

构建一个包含所有可能唯一排列的列表列表

包括负值表示路由方向的各个列号

(向上读取列与向下读取列)。

输入：

-总列数

返回值：

-唯一列顺序的列表列表，包括负值表示路由方向

路由密码加密方向

"""

import math

from itertools import permutations, product

#------开始输入-----------------------------------------------------------

# 输入总列数：

num_cols = 4

#------以下内容不要编辑--------------------------------------------

# 生成单个列号的列表

columns = [x for x in range(1, num_cols+1)]

print("columns = {}".format(columns))

# 构建列号组合的列表列表

# itertools product计算输入可迭代对象的笛卡尔积

def perms(columns):

"""获取列数整数并生成正负排列。"""

results = []

for perm in permutations(columns):

for signs in product([-1, 1], repeat=len(columns)):

results.append([i*sign for i, sign in zip(perm, signs)])

return results

col_combos = perms(columns)

print(*col_combos, sep="\n")  # 如果num_cols > 4，请注释掉此行！

print("没有负数的列数的阶乘 = {}"

.format(math.factorial(num_cols)))

print("列组合的数量 = {}".format(len(col_combos)))

#### ***路线置换密码：暴力破解攻击***

这个实践项目使用了两个程序。第二个程序，*perms.py*，作为第一个程序*route_cipher_hacker.py*的模块。它是基于之前描述的程序*permutations_practice.py*构建的，该程序在[第371页](appendix.xhtml#page_371)的“[自动化可能的密钥](appendix.xhtml#lev408)”中进行了介绍。

##### ***route_cipher_hacker.py***

*route_cipher_hacker.py*

"""暴力破解联合路径密码（route_cipher_hacker.py）。"""

设计用于具有可变行列的全词置换密码。

假设加密从列的顶部或底部开始。

基于输入的列数和行数自动生成可能的密钥。

Key表示读取列的顺序和遍历的方向。

负的列号意味着从底部开始，向上读取。

正的列号意味着从顶部开始，向下读取。

下面的示例适用于4x4矩阵，密钥为-1 2 -3 4。

注意，“0”不允许使用。

箭头显示加密路径；对于负数密钥值，从上读。

1   2   3   4

___ ___ ___ ___

| ^ | | | ^ | | | 消息已写入

|_|_|_v_|_|_|_v_|

| ^ | | | ^ | | | 每行横向读取

|_|_|_v_|_|_|_v_|

| ^ | | | ^ | | | 以这种方式

|_|_|_v_|_|_|_v_|

| ^ | | | ^ | | | 最后一行填充了虚拟词

|_|_|_v_|_|_|_v_|

开始        结束

所需输入 - 一条文本信息，列数，行数，密钥字符串

需要自定义的“perms”模块来生成密钥

输出所用的密钥和解密后的明文

"""

import sys

import perms

#==============================================================================

# 用户输入：

# 要解密的字符串（请输入或粘贴在三引号之间）：

ciphertext = """REST TRANSPORT YOU GODWIN VILLAGE ROANOKE WITH ARE YOUR IS JUST

提供免费的雪，前往南方填充器

"""

# 假定的置换矩阵中的列数：

COLS = 4

# 假定的置换矩阵中的行数：

ROWS = 5

# 用户输入结束 - 以下内容请勿编辑！

#==============================================================================

def main():

"""将密文转换为列表，调用验证和解密函数。"""

cipherlist = list(ciphertext.split())

validate_col_row(cipherlist)

decrypt(cipherlist)

def validate_col_row(cipherlist):

"""检查输入的列和行是否与消息长度有效对比。"""

factors = []

len_cipher = len(cipherlist)

for i in range(2, len_cipher):  # 范围排除单列密码

if len_cipher % i == 0:

factors.append(i)

print("\n密码长度 = {}".format(len_cipher))

print("可接受的列/行值包括：{}".format(factors))

print()

if ROWS * COLS != len_cipher:

print("\n错误 - 输入的列和行不是长度的因数 "

"密码的终止程序。", file=sys.stderr)

sys.exit(1)

def decrypt(cipherlist):

"""将列转换为列表列表中的项并解密密文。"""

col_combos = perms.perms(COLS)

for key in col_combos:

translation_matrix = [None] * COLS

plaintext = ''

start = 0

stop = ROWS

for k in key:

if k < 0: # 从下到上读取列

col_items = cipherlist[start:stop]

elif k > 0: # 从上到下读取列

col_items = list((reversed(cipherlist[start:stop])))

translation_matrix[abs(k) - 1] = col_items

start += ROWS

stop += ROWS

# 遍历嵌套列表，将最后一个项弹出到新列表：

for i in range(ROWS):

for matrix_col in translation_matrix:

word = str(matrix_col.pop())

plaintext += word + ' '

print("\n使用密钥 = {}".format(key))

print("翻译结果 = {}".format(plaintext))

print("\n密钥数量 = {}".format(len(col_combos)))

if __name__ == '__main__':

main()

##### ***perms.py***

*perms.py*

"""对于列的总数，找出所有独特的列排列。

构建包含所有可能唯一排列的列表列表

包括负值的单独列号用于路线方向

输入：

-列总数

返回：

-包含负值的唯一列顺序列表列表

路线密码加密方向

"""

from itertools import permutations, product

# 构建列号组合的列表列表

# itertools的product计算输入迭代对象的笛卡尔积

def perms(num_cols):

"""接受列数整数并生成正负排列。"""

results = []

columns = [x for x in range(1, num_cols+1)]

for perm in permutations(columns):

for signs in product([-1, 1], repeat=len(columns)):

results.append([i*sign for i, sign in zip(perm, signs)])

return results

### **第五章：编码英国内战密码**

#### ***保存玛丽***

*save_Mary_practice.py*

"""在一系列名字中隐藏一个空密码，使用变量模式。"""

import load_dictionary

# 写一个简短的消息，且不要使用标点符号或数字！

message = "给出你的词，我们崛起"

message = "".join(message.split())

# 打开名字文件

names = load_dictionary.load('supporters.txt')

name_list = []

# 用未在密码中使用的空单词初始化列表

name_list.append(names[0])

# 将空密码的字母添加到名字的第二个字母，然后是第三个字母，然后重复

count = 1

for letter in message:

for name in names:

if len(name) > 2 and name not in name_list:

if count % 2 == 0 and name[2].lower() == letter.lower():

name_list.append(name)

count += 1

break

elif count % 2 != 0 and name[1].lower() == letter.lower():

name_list.append(name)

count += 1

break

# 在消息开头添加两个空单词来迷惑密码分析师

name_list.insert(3, 'Stuart')

name_list.insert(6, 'Jacob')

# 显示封面信和带有空白密码的列表

print("""

尊贵的殿下： \n

非常高兴向您呈现支持您事业并请求篡位者的贵族家族名单

已承诺支持您的事业，并向篡位者请愿

释放陛下从当前悲剧性境况中。

""")

print(*name_list, sep='\n')

#### ***科尔切斯特的骗局***

*colchester_practice.py*

"""根据每隔第 n 个字母在每隔第 n 个单词中的位置解密。"""

import sys

def load_text(file):

"""加载文本文件作为字符串。"""

with open(file) as f:

return f.read().strip()

# 加载并处理消息：

filename = input("\n请输入要翻译的消息的完整文件名：")

try:

loaded_message = load_text(filename)

except IOError as e:

print("{}. 程序终止.".format(e), file=sys.stderr)

sys.exit(1)

# 检查加载的消息和行数

print("\n原始消息 = {}\n".format(loaded_message))

# 将消息转换为列表并获取长度

message = loaded_message.split()

end = len(message)

# 获取用户输入的检查间隔

increment = int(input("输入最大单词和字母位置以 \

检查（例如，每 1 of 1, 2 of 2 等）："))

print()

# 查找指定间隔的字母

for i in range(1, increment + 1):

print("\n使用增量字母 {} 的单词 {}".format(i, i))

print()

count = i - 1

location = i - 1

for index, word in enumerate(message):

if index == count:

if location < len(word):

print("letter = {}".format(word[location]))

count += i

else:

print("间隔不起作用", file=sys.stderr)

### **第6章：用隐形墨水写作**

#### ***检查空白行的数量***

*elementary_ink_practice.py*

"""添加代码检查假消息中的空白行与实际消息中的行数。"""

import sys

import docx

from docx.shared import RGBColor, Pt

# 获取假消息的文本并将每行作为列表项

fake_text = docx.Document('fakeMessage.docx')

fake_list = []

for paragraph in fake_text.paragraphs:

fake_list.append(paragraph.text)

# 获取实际消息的文本并将每行作为列表项

real_text = docx.Document('realMessageChallenge.docx')

real_list = []

for paragraph in real_text.paragraphs:

if len(paragraph.text) != 0:  # 删除空白行

real_list.append(paragraph.text)

# 定义函数检查可用隐藏空间：

def line_limit(fake, real):

"""比较假消息中的空白行数与实际消息中的行数，并

如果没有足够的空白行来容纳实际消息，则警告用户。

注意： 需要导入 'sys'

"""

num_blanks = 0

num_real = 0

for line in fake:

if line == '':

num_blanks += 1

num_real = len(real)

diff = num_real - num_blanks

print("\n假消息中的空白行数 = {}".format(num_blanks))

print("实际消息中的行数 = {}\n".format(num_real))

if num_real > num_blanks:

print("假消息需要 {} 更多空白行。"

.format(diff), file=sys.stderr)

sys.exit()

line_limit(fake_list, real_list)

# 加载设置样式、字体、边距等的模板

doc = docx.Document('template.docx')

# 添加信头

doc.add_heading('Morland Holmes', 0)

subtitle = doc.add_heading('全球咨询与谈判', 1)

subtitle.alignment = 1

doc.add_heading('', 1)

doc.add_paragraph('2015年12月17日')

doc.add_paragraph('')

def set_spacing(paragraph):

"""使用 docx 设置段落间行距。"""

paragraph_format = paragraph.paragraph_format

paragraph_format.space_before = Pt(0)

paragraph_format.space_after = Pt(0)

length_real = len(real_list)

count_real = 0  # 当前行在真实（隐藏）消息中的索引

# interleave real and fake message lines

for line in fake_list:

if count_real < length_real and line == "":

paragraph = doc.add_paragraph(real_list[count_real])

paragraph_index = len(doc.paragraphs) - 1

# 设置真实消息的颜色为白色

run = doc.paragraphs[paragraph_index].runs[0]

font = run.font

font.color.rgb = RGBColor(255, 255, 255)  # 测试时将其设置为红色

count_real += 1

else:

paragraph = doc.add_paragraph(line)

set_spacing(paragraph)

doc.save('ciphertext_message_letterhead.docx')

print("完成"))

### **第8章：为俳句诗歌计数音节**

#### ***音节计数器与字典文件***

*test_count_syllables_w_dict.py*

"""加载字典文件，挑选随机单词，运行音节计数模块。"""

import sys

import random

from count_syllables import count_syllables

def load(file):

"""打开文本文件并返回小写字符串列表。"""

with open(file) as in_file:

loaded_txt = in_file.read().strip().split('\n')

loaded_txt = [x.lower() for x in loaded_txt]

return loaded_txt

try:

word_list = load('2of4brif.txt')

except IOError as e:

print("{}\n打开文件时出错。程序终止.".format(e),

file=sys.stderr)

sys.exit(1)

test_data = []

num_words = 100

test_data.extend(random.sample(word_list, num_words))

for word in test_data:

try:

num_syllables = count_syllables(word)

print(word, num_syllables, end='\n')

except KeyError:

print(word, end='')

print(" 未找到", file=sys.stderr)

### **第10章：我们孤单吗？探索费米悖论**

#### ***遥远的银河***

*galaxy_practice.py*

"""使用螺旋公式构建银河展示。"""

import math

from random import randint

import tkinter

root = tkinter.Tk()

root.title("银河 BR549")

c = tkinter.Canvas(root, width=1000, height=800, bg='black')

c.grid()

c.configure(scrollregion=(-500, -400, 500, 400))

oval_size = 0

# build spiral arms

num_spiral_stars = 500

angle = 3.5

core_diameter = 120

spiral_stars = []

for i in range(num_spiral_stars):

theta = i * angle

r = math.sqrt(i) / math.sqrt(num_spiral_stars)

spiral_stars.append((r * math.cos(theta), r * math.sin(theta)))

for x, y in spiral_stars:

x = x * 350 + randint(-5, 3)

y = y * 350 + randint(-5, 3)

oval_size = randint(1, 3)

c.create_oval(x-oval_size, y-oval_size, x+oval_size, y+oval_size,

fill='white', outline='')

# build wisps

wisps = []

for i in range(2000):

theta = i * angle

# 通过 num_spiral_stars 来划分，得到更好的尘埃带

r = math.sqrt(i) / math.sqrt(num_spiral_stars)

spiral_stars.append((r * math.cos(theta), r * math.sin(theta)))

对于 x, y 在 spiral_stars 中：

x = x * 330 + randint(-15, 10)

y = y * 330 + randint(-15, 10)

h = math.sqrt(x**2 + y**2)

if h < 350:

wisps.append((x, y))

c.create_oval(x-1, y-1, x+1, y+1, 填充='white', 外轮廓='')

# 构建银河核心

core = []

对于 i 在 range(900) 中：

x = randint(-core_diameter, core_diameter)

y = randint(-core_diameter, core_diameter)

h = math.sqrt(x**2 + y**2)

if h < core_diameter - 70:

core.append((x, y))

oval_size = randint(2, 4)

c.create_oval(x-oval_size, y-oval_size, x+oval_size, y+oval_size,

填充='white', 外轮廓='')

elif h < core_diameter:

core.append((x, y))

oval_size = randint(0, 2)

c.create_oval(x-oval_size, y-oval_size, x+oval_size, y+oval_size,

填充='white', 外轮廓='')

root.mainloop()

#### ***构建银河帝国***

*empire_practice.py*

"""构建银河二维模型，扩展环用于银河帝国。"""

import tkinter as tk

import time

from random import randint, uniform, random

import math

#=============================================================================

# 主输入

# 银河帝国母星位置：

HOMEWORLD_LOC = (0, 0)

# 最大模拟年份数：

最大年份 = 10000000

# 平均扩展速度与光速的比值：

速度 = 0.005

# 缩放单位

单位 = 200

#======================================================================

# 设置显示画布

root = tk.Tk()

root.title("银河系")

c = tk.Canvas(root, width=1000, height=800, bg='black')

c.grid()

c.configure(scrollregion=(-500, -400, 500, 400))

# 实际银河系尺寸（光年）

DISC_RADIUS = 50000

disc_radius_scaled = round(DISC_RADIUS/UNIT)

def polar_coordinates():

"""在圆盘内生成均匀随机的 x, y 坐标，用于二维显示。"""

r = random()

theta = uniform(0, 2 * math.pi)

x = round(math.sqrt(r) * math.cos(theta) * disc_radius_scaled)

y = round(math.sqrt(r) * math.sin(theta) * disc_radius_scaled)

返回 x, y

def spirals(b, r, rot_fac, fuz_fac, arm):

"""使用对数螺旋公式为 tkinter 显示构建螺旋臂。

b = 对数螺旋方程中的任意常数

r = 缩放的银河盘半径

rot_fac = 旋转因子

fuz_fac = 星体位置的随机偏移量，应用于“fuzz”变量

arm = 螺旋臂（0 = 主臂，1 = 后续星体）

"""

spiral_stars = []

fuzz = int(0.030 * abs(r))  # 随机偏移星体位置

theta_max_degrees = 520

对于 i 在 range(theta_max_degrees) 中：  # range(0, 700, 2) 用于没有黑洞的情况

theta = math.radians(i)

x = r * math.exp(b*theta) * math.cos(theta + math.pi * rot_fac)

+ randint(-fuzz, fuzz) * fuz_fac

y = r * math.exp(b*theta) * math.sin(theta + math.pi * rot_fac)

+ randint(-fuzz, fuzz) * fuz_fac

spiral_stars.append((x, y))

对于 x, y 在 spiral_stars 中：

if arm == 0 and int(x % 2) == 0:

c.create_oval(x-2, y-2, x+2, y+2, 填充='white', 外轮廓='')

elif arm == 0 and int(x % 2) != 0:

c.create_oval(x-1, y-1, x+1, y+1, 填充='white', 外轮廓='')

elif arm == 1:

c.create_oval(x, y, x, y, 填充='white', 外轮廓='')

def star_haze(scalar):

"""在银河盘面中随机分布微弱的 tkinter 星星。

disc_radius_scaled = 银河盘面半径缩放至无线电气泡直径

scalar = multiplier 用于调整星星数量

"""

for i in range(0, disc_radius_scaled * scalar):

x, y = polar_coordinates()

c.create_text(x, y, fill='white', font=('Helvetica', '7'), text='.')

def model_expansion():

"""模拟从家园开始的帝国扩展，带有同心圆环。"""

r = 0 # 从家园到的半径

text_y_loc = -290

x, y = HOMEWORLD_LOC

c.create_oval(x-5, y-5, x+5, y+5, fill='red')

increment = round(MAX_YEARS / 10) # 年份间隔以发布圆圈

c.create_text(-475, -350, anchor='w', fill='red', text='增量 = {:,}'

.format(increment))

c.create_text(-475, -325, anchor='w', fill='red',

text='速度作为光速的比例 = {:,}'.format(SPEED))

for years in range(increment, MAX_YEARS + 1, increment):

time.sleep(0.5) # 在发布新扩展圆圈之前的延迟

traveled = SPEED * increment / UNIT

r = r + traveled

c.create_oval(x-r, y-r, x+r, y+r, fill='', outline='red', width='2')

c.create_text(-475, text_y_loc, anchor='w', fill='red',

text='年份 = {:,}'.format(years))

text_y_loc += 20

# 更新画布以适应新的圆圈；不再需要 mainloop()

c.update_idletasks()

c.update()

def main():

"""生成银河显示，模拟帝国扩展，运行 mainloop。"""

spirals(b=-0.3, r=disc_radius_scaled, rot_fac=2, fuz_fac=1.5, arm=0)

spirals(b=-0.3, r=disc_radius_scaled, rot_fac=1.91, fuz_fac=1.5, arm=1)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=2, fuz_fac=1.5, arm=0)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=-2.09, fuz_fac=1.5, arm=1)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=0.5, fuz_fac=1.5, arm=0)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=0.4, fuz_fac=1.5, arm=1)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=-0.5, fuz_fac=1.5, arm=0)

spirals(b=-0.3, r=-disc_radius_scaled, rot_fac=-0.6, fuz_fac=1.5, arm=1)

star_haze(scalar=9)

model_expansion()

# 运行 tkinter 循环

root.mainloop()

if __name__ == '__main__':

main()

#### ***一种预测可检测性的曲折方法***

*rounded_detection_practice.py*

"""计算给定 15.6 M 无线电气泡直径的 32 LY 探测概率

在银河中随机分布文明。"""

import math

from random import uniform, random

from collections import Counter

# length units in light-years

DISC_RADIUS = 50000

DISC_HEIGHT = 1000

NUM_CIVS = 15600000

DETECTION_RADIUS = 16

def random_polar_coordinates_xyz():

"""在 3D 圆盘内生成均匀的随机 xyz 点。"""

r = random()

theta = uniform(0, 2 * math.pi)

x = round(math.sqrt(r) * math.cos(theta) * DISC_RADIUS, 3)

y = round(math.sqrt(r) * math.sin(theta) * DISC_RADIUS, 3)

z = round(uniform(0, DISC_HEIGHT), 3)

return x, y, z

def rounded(n, base):

"""将数字四舍五入到由 base 参数指定的最接近的数字。"""

return int(round(n/base) * base)

def distribute_civs():

"""在银河盘面模型中分布 xyz 位置并返回列表。"""

civ_locs = []

while len(civ_locs) < NUM_CIVS:

loc = random_polar_coordinates_xyz()

civ_locs.append(loc)

return civ_locs

def round_civ_locs(civ_locs):

"""四舍五入xyz位置并返回四舍五入后的地点列表。"""

# 将半径转换为立方体维度：

detect_distance = round((4 / 3 * math.pi * DETECTION_RADIUS**3)**(1/3))

print("\n检测半径 = {} 光年".format(DETECTION_RADIUS))

print("立方体检测距离 = {} 光年".format(detect_distance))

# 将文明xyz四舍五入到检测距离

civ_locs_rounded = []

for x, y, z in civ_locs:

i = rounded(x, detect_distance)

j = rounded(y, detect_distance)

k = rounded(z, detect_distance)

civ_locs_rounded.append((i, j, k))

return civ_locs_rounded

def calc_prob_of_detection(civ_locs_rounded):

"""计算位置并计算重复值的概率。"""

overlap_count = Counter(civ_locs_rounded)

overlap_rollup = Counter(overlap_count.values())

num_single_civs = overlap_rollup[1]

prob = 1 - (num_single_civs / NUM_CIVS)

return overlap_rollup, prob

def main():

"""调用函数并打印结果。"""

civ_locs = distribute_civs()

civ_locs_rounded = round_civ_locs(civ_locs)

overlap_rollup, detection_prob = calc_prob_of_detection(civ_locs_rounded)

print("四舍五入前的civ_locs长度 = {}".format(len(civ_locs)))

print("四舍五入后civ_locs_rounded的长度 = {}".format(len(civ_locs_rounded)))

print("overlap_rollup = {}\n".format(overlap_rollup))

print("检测概率 = {0:.3f}".format(detection_prob))

# QC步骤：检查四舍五入

print("\n前三个位置的四舍五入前后：\n")

for i in range(3):

print("四舍五入前：{}".format(civ_locs[i]))

print("四舍五入后：{} \n".format(civ_locs_rounded[i]))

if __name__ == '__main__':

main()

### **第11章：蒙提·霍尔问题**

#### ***生日悖论***

*birthday_paradox_practice.py*

"""计算每x个人有相同生日的概率。"""

import random

max_people = 50

num_runs = 2000

print("\n至少有2人生日相同的概率：\n")

for people in range(2, max_people + 1):

found_shared = 0

for run in range(num_runs):

bdays = []

for i in range(0, people):

bday = random.randrange(0, 365)  # 忽略闰年

bdays.append(bday)

set_of_bdays = set(bdays)

if len(set_of_bdays) < len(bdays):

found_shared += 1

prob = found_shared/num_runs

print("人数 = {} 概率 = {:.4f}".format(people, prob))

print("""

根据生日悖论，如果房间里有23个人，

有50%的机会这2个人会有相同的生日。

""")

### **第13章：模拟外星火山**

#### ***走向远方***

*practice_45.py*

import sys

import math

import random

import pygame as pg

pg.init()  # 初始化pygame

# define color table

BLACK = (0, 0, 0)

WHITE = (255, 255, 255)

LT_GRAY = (180, 180, 180)

GRAY = (120, 120, 120)

DK_GRAY = (80, 80, 80)

class Particle(pg.sprite.Sprite):

"""构建火山模拟的喷发物质颗粒。"""

gases_colors = {'SO2': LT_GRAY, 'CO2': GRAY, 'H2S': DK_GRAY, 'H2O': WHITE}

VENT_LOCATION_XY = (320, 300)

IO_SURFACE_Y = 308

GRAVITY = 0.5  # 像素每帧

VELOCITY_SO2 = 8  # 像素每帧

# 标量（SO2 原子量/粒子原子量）用于速度

vel_scalar = {'SO2': 1, 'CO2': 1.45, 'H2S': 1.9, 'H2O': 3.6}

def __init__(self, screen, background):

super().__init__()

self.screen = screen

self.background = background

self.image = pg.Surface((4, 4))

self.rect = self.image.get_rect()

self.gas = 'SO2'

self.color = ''

self.vel = Particle.VELOCITY_SO2 * Particle.vel_scalar[self.gas]

self.x, self.y = Particle.VENT_LOCATION_XY

self.vector()

def vector(self):

"""计算粒子发射时的向量。"""

angles = [65, 55, 45, 35, 25]  # 90 为垂直

orient = random.choice(angles)

if orient == 45:

self.color = WHITE

else:

self.color = GRAY

radians = math.radians(orient)

self.dx = self.vel * math.cos(radians)

self.dy = -self.vel * math.sin(radians)  # y 轴向下增加时为负

def update(self):

"""应用重力，绘制路径并处理边界条件。"""

self.dy += Particle.GRAVITY

pg.draw.line(self.background, self.color, (self.x, self.y),

(self.x + self.dx, self.y + self.dy))

self.x += self.dx

self.y += self.dy

if self.x < 0 or self.x > self.screen.get_width():

self.kill()

if self.y < 0 or self.y > Particle.IO_SURFACE_Y:

self.kill()

def main():

"""设置并运行游戏画面与循环。"""

screen = pg.display.set_mode((639, 360))

pg.display.set_caption("Io 火山模拟器")

background = pg.image.load("tvashtar_plume.gif")

# 设置颜色编码的图例

legend_font = pg.font.SysFont('None', 26)

text = legend_font.render('白色 = 45 度', True, WHITE, BLACK)

particles = pg.sprite.Group()

clock = pg.time.Clock()

while True:

clock.tick(25)

particles.add(Particle(screen, background))

for event in pg.event.get():

if event.type == pg.QUIT:

pg.quit()

sys.exit()

screen.blit(background, (0, 0))

screen.blit(text, (320, 170))

particles.update()

particles.draw(screen)

pg.display.flip()

if __name__ == "__main__":

main()

### **第16章：使用本福特定律寻找欺诈**

#### ***击败本福特***

*beat_benford_practice.py*

"""操作投票计数，使最终结果符合本福特定律。"""

# 下面的示例是针对特朗普与希拉里，2016年伊利诺伊州总统选举

def load_data(filename):

"""打开一个包含数字的文本文件，并将内容转换为整数列表。"""

with open(filename) as f:

lines = f.read().strip().split('\n')

return [int(i) for i in lines]  # 将字符串转换为整数

def steal_votes(opponent_votes, candidate_votes, scalar):

"""使用标量来减少一个投票数并增加另一个投票数，返回为列表。

Arguments:

opponent_votes – 被盗的投票

candidate_votes - 被盗投票的增加量

scalar - 分数百分比，< 1，用于减少投票数

Returns:

改变后的对手投票列表

改变后的候选人投票列表

"""

new_opponent_votes = []

new_candidate_votes = []

对于`opp_vote`和`can_vote`，使用`zip(opponent_votes, candidate_votes)`配对：

new_opp_vote = round(opp_vote * scalar)

new_opponent_votes.append(new_opp_vote)

stolen_votes = opp_vote - new_opp_vote

new_can_vote = can_vote + stolen_votes

new_candidate_votes.append(new_can_vote)

return new_opponent_votes, new_candidate_votes

def main():

"""运行程序。

加载数据，设置目标获胜选票数，调用函数，显示

结果以表格形式显示，将新的合并选票总数写入文本文件

用于本福特定律分析的输入。

"""

# 加载选票数据

c_votes = load_data('Clinton_votes_Illinois.txt')

j_votes = load_data('Johnson_votes_Illinois.txt')

s_votes = load_data('Stein_votes_Illinois.txt')

t_votes = load_data('Trump_votes_Illinois.txt')

total_votes = sum(c_votes + j_votes + s_votes + t_votes)

# 假设特朗普以49%的选票获得多数

t_target = round(total_votes * 0.49)

print("\n特朗普胜利目标 = {:,} 票".format(t_target))

# 计算特朗普胜利所需的额外选票

extra_votes_needed = abs(t_target - sum(t_votes))

print("所需额外选票 = {:,}".format(extra_votes_needed))

# 计算生成额外选票所需的标量

scalar = 1 - (extra_votes_needed / sum(c_votes + j_votes + s_votes))

print("标量 = {:.3}".format(scalar))

print()

# 根据标量翻转选票计数并构建新的合并选票列表

fake_counts = []

new_c_votes, new_t_votes = steal_votes(c_votes, t_votes, scalar)

fake_counts.extend(new_c_votes)

new_j_votes, new_t_votes = steal_votes(j_votes, new_t_votes, scalar)

fake_counts.extend(new_j_votes)

new_s_votes, new_t_votes = steal_votes(s_votes, new_t_votes, scalar)

fake_counts.extend(new_s_votes)

fake_counts.extend(new_t_votes)  # 添加最后的，因为到现在为止一直在变化

# 以表格形式比较旧的和新的选票计数与总数

# 根据需要切换“特朗普”和“克林顿”

for i in range(0, len(t_votes)):

print("旧特朗普: {} \t 新特朗普: {} \t 旧克林顿: {} \t " \

"新克林顿: {}".

format(t_votes[i], new_t_votes[i], c_votes[i], new_c_votes[i]))

print("-" * 95)

print("总计:")

print("旧特朗普: {:,} \t 新特朗普: {:,} \t 旧克林顿: {:,}  " \

"新克林顿: {:,}".format(sum(t_votes), sum(new_t_votes),

sum(c_votes), sum(new_c_votes)))

# 写出文本文件，作为benford.py程序的输入

# 本程序将检查伪造选票与本福特定律的一致性

with open('fake_Illinois_counts.txt', 'w') as f:

for count in fake_counts:

f.write("{}\n".format(count))

if __name__ == '__main__':

main()
