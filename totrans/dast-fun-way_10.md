# 10

哈希表

![](img/chapterart.png)

本章介绍了*哈希表*，这是一种动态数据结构，经过高度优化，适用于插入和查找操作。哈希表通过数学函数帮助我们指向数据的位置。它们在纯存储的场景中尤为有用，在这些场景中，目标是快速地查找和获取信息。

这正是我们为咖啡储藏室所做的折衷。忘了试图按过期日期或口味排序咖啡——我们是资深的咖啡爱好者，轻松记住每一颗咖啡豆的最小细节。对于任何给定的属性（或属性组合），我们都能瞬间记住咖啡的名称。当我们走到咖啡储藏室时，我们已经决定了要喝哪种咖啡。把我们的咖啡选择按这些其他维度进行排序，只会拖慢我们的速度。我们需要的是高效的检索：只要知道我们想喝的咖啡的名字，就希望能够以最小的努力找到那些咖啡豆。哈希表正是实现这种快速通过名称检索的理想工具。

数组提供了一种紧凑的结构来存储单独的数据项，并提供高效检索的机制——但前提是我们知道该项的索引。有了索引，我们可以在常数时间内查找任何元素。正如我们在第二章中所看到的，如果没有索引，查找数组中的项的过程就变得更加复杂。如果我们只有项的值，那么就需要遍历数组，找到其正确的位置。我们可以通过二分查找高效地找到项，但代价是必须保持数组的有序性，这会导致插入和删除操作效率低下。

在前面的章节中，我们探讨了数据结构和算法，如何高效地搜索目标值。试想，如果我们能构建一个神奇的函数，直接将目标值映射到其索引（当然，有一些限制）。这就是哈希表的核心思想。哈希表通过数学函数计算值在数据结构中的索引，从而让我们可以直接从值映射到数组中的某个位置。缺点是，没有映射是完美的。我们将看到不同的值可能映射到相同的位置，导致冲突。接着我们将探讨解决冲突的两种方法。

和所有数据结构一样，哈希表并不是解决所有问题的完美方案——我们将探讨它的优点和折衷，包括内存使用和最坏情况下的性能。在此过程中，我们将研究一种通过使用数学映射来组织数据的新方式。

## 使用键进行存储和查找

在我们深入探讨哈希表的机制之前，先考虑一种理想化的索引方案，用于高效检索整数值——为每个*可能的*值维护一个单独的数组箱，并用该值本身作为索引。这个结构如图 10-1 所示。为了插入值 9，我们只需将其放入索引为 9 的箱子中。在这种安排下，我们可以在常数时间内插入或检索项目。

![一个包含十个箱子的数组，编号从 0 到 9。箱子 1、5 和 7 指向数组外的数据，剩余的箱子通过斜杠表示为空。](img/f10001.png)

图 10-1：一个大型数组，每个潜在条目都有一个箱子

我们理想化的数据结构的明显缺点是维护每个可能键的数组的荒谬成本。考虑存储所有可能的 16 位信用卡号码的情况。我们将需要几个千万亿个箱子，精确地说是 10¹⁶个箱子。这需要大量的内存。更糟糕的是，我们可能根本不会使用这么多箱子。如果我们正在编写一个程序来跟踪一个 1000 人公司的企业信用卡，我们只需要非常少的可用箱子——每 10¹³个箱子中只有一个箱子是用来存储的。其余的都是浪费。它们空着，等待着某天能有数据存储进来。同样，我们也不想为每本可能的书籍预留一个图书馆位置，为每位可能的客人预定一个酒店房间，或者为每种已知的咖啡预留一个咖啡储藏室。那太荒谬了（除非是咖啡的话）。

但是，作为一种思想实验，让我们考虑这个理想化的数据结构如何适用于其他类型的数据。我们立刻遇到了一个问题，那就是如何为更复杂的数据类型（如字符串甚至复合数据类型）选择值。假设我们正在创建一个简单的咖啡记录数据库。第三章展示了如何使用指针数组来存储这种动态大小的数据，如以下代码所示：

```py
CoffeeRecord {
    String: Name
    String: Brand
    Integer: Rating
    FloatingPoint: Cost_Per_Pound
    Boolean: Is_Dark_Roast
    String: Other_Notes
    Image: Barcode
}
```

我们仍然可以将所有项放入一个单一的大数组中，为每个*可能的*条目准备一个箱子。在这种情况下，箱子不仅包含单一的值，还包含指向更复杂数据结构的指针，如图 10-2 中的指针数组所示。

![一个包含十个箱子的数组，编号从 0 到 9。每个箱子里都有一个指向数组外部数据的箭头。](img/f10002.png)

图 10-2：一个指针数组

然而，这仍然留下了如何进行实际查找的问题。如果我们想要查找我们为“Jeremy 的高咖啡因体验：中度烘焙”所给的评分，这是第六章中介绍的一种咖啡，我们不能将整个复合数据结构作为值。我们没有这些完整的信息。即使我们确实有这些完整的信息，也不清楚我们如何使用复合数据结构甚至字符串作为索引。

计算机程序通常使用键来识别记录。*键*是一个与数据本身一起存储或从数据中派生出来的单一值，可以用来标识一个记录。在 RSVP 列表的情况下，键可能是包含受邀人姓名的字符串；在我们的咖啡记录中，键可以是咖啡的名称或条形码。在许多数据结构中，从排序数组到字典树，我们使用键来组织数据。对于本书早期的数值示例，键就是值本身。每次在排序数组或二叉搜索树中搜索一个特定的数值，都是通过查找匹配的数值键来检索记录。类似地，第六章介绍的字典树使用字符串作为键。

然而，这并不能解决索引问题。除非我们有一个整数键，否则我们仍然无法对数组桶进行索引。数组中没有名为“Jeremy 的高咖啡因体验：中度烘焙”的桶。我们可以在数据结构中搜索，寻找一个匹配的记录。线性扫描和二分查找都以这种方式工作，使用目标值作为键。然而，我们失去了理想数据结构的魔力。我们又回到了查找匹配键的问题。

在某些情况下，我们可能能够为我们的记录找到一个自然的数字键。在咖啡示例中，我们可以按我们第一次品尝咖啡的时间顺序列出我们曾经品尝过的每一款咖啡，并使用相应的日期作为键。如果“Jeremy 的高咖啡因体验：中度烘焙”首次品尝的日期是 2020 年 1 月 1 日（假设我们能神奇地记住这一点），我们可以通过二分查找来检索记录。或者，我们可以使用咖啡的条形码或它在*《世界咖啡、品牌与制造商大全》*中的页码。

更一般来说，我们希望有一个函数，能够从我们的键生成一个索引。在接下来的章节中，我们将介绍哈希函数，它正好解决了这个问题。

## 哈希表

哈希表使用数学映射来压缩键空间。它们通过使用*哈希函数*将原始键映射到表中的位置（也叫做*哈希值*），将大的键空间压缩成一个小的域。我们将映射键*k*到一个有*b*个桶的表的哈希函数表示为一个函数*f*(*k*)，其范围为[*0*, *b* − 1]*。这种映射解决了上一节中的两个问题。我们不再需要无限多个桶，只需要*b*个桶。正如我们将看到的，函数也可以将非整数映射到数值范围，从而解决了非整数键的问题。

一个简单的整数键哈希函数是使用除法方法通过数字键来计算哈希值。我们将键除以桶的数量并取余：

*f*(*k*) = *k* % *b*

其中，*%* 是模运算。每一个可能的（整数）键都被映射到正确范围[*0*, *b* − 1]*内的一个桶中。例如，对于一个 20 个桶的哈希表，该函数将产生表 10-1 中所示的映射。

表 10-1：使用 20 个容器的除法法哈希映射示例

| ***k*** | ***f*(*k*)** |
| --- | --- |
| 5 | 5 |
| 20 | 0 |
| 21 | 1 |
| 34 | 14 |
| 41 | 1 |

考虑将所有信用卡号码映射到 100 个容器的问题。除法方法通过使用卡号的最后两位数字，将关键字空间从 16 位压缩到 2 位。当然，这种简化的映射可能对某些关键字分布效果不佳。如果我们有很多卡号以 10 结尾，它们都会映射到同一个容器。然而，它解决了我们一个核心问题：通过一次（且高效的）数学运算，我们将大量的关键字压缩到有限的索引范围。

怀疑的读者可能会对上述描述产生疑问：“我们不能将两个不同的项存储在数组的同一元素中。你在第一章就告诉我们这一点。哈希函数显然可以将两个不同的值映射到同一个容器中。看看表 10-1，21 和 41 都映射到容器 1。”这就是之前提到的警告。遗憾的是，哈希函数并非真正的魔法。正如我们将在下一节看到的那样，这种复杂性就是哈希表结构中其余部分的作用——处理碰撞。现在，我们可以注意到，哈希函数将我们的关键字划分为不相交的集合，我们只需要关心集合内部的碰撞问题。

哈希函数不限于数字。我们也可以定义一个哈希函数，将咖啡的名称映射到某个容器。这使我们可以通过两步直接访问任何咖啡记录，如图 10-3 所示。首先，我们使用咖啡的名称来计算其哈希值。关键字“House Blend”被映射为 6；我们将在本章稍后介绍一种简单的字符串哈希方法。其次，我们通过使用哈希值作为索引来查找我们表中的哈希值。我们甚至可以使用这种方法将我们庞大的现实世界咖啡收藏映射到咖啡储藏室中的固定架子上。

![左侧显示了一列咖啡名称，如“House Blend”和“Morning Shock”。中间列包含哈希函数，将字符串映射为数字值。House Blend 映射到 6。箭头指向哈希函数对应的数组中的容器。](img/f10003.png)

图 10-3：哈希函数将字符串映射到数组中的索引

哈希表的一个现实世界示例是我们在生活中常见的注册表，无论是在夏令营的第一天、比赛的早晨，还是学术会议的开始。需要存储的项目（注册包）会根据它们的关键字（名字）被分配到独特的容器中。人们可以通过应用一个哈希函数来找到正确的容器，通常这个哈希函数就像一个简单的符号，将字母表的某个范围映射到某一行。例如，名字以 A−D 开头的放到第 1 行，E−G 的放到第 2 行，依此类推。

### 碰撞

即使是世界上最好的哈希函数，也无法提供完美的键到桶的**一对一**映射。为了实现这一点，我们需要回到那个庞大的数组及其对内存的过度使用。任何将大量键映射到远小于它们的值集合的数学函数，都会遇到偶尔的*碰撞*——即两个键映射到相同的哈希值的情况。想象一下，如果将车牌号的第一个数字用于映射停车位，结果是将车牌号映射到 10 个停车位上。我们并不需要我们的同事的车牌号与我们的完全匹配才能为停车位发生争执。比如你去登记你的车，车牌号是“二叉搜索树是 #1”，却发现你的同事已经用车牌号“100,000 数据结构及计数”占了那个车位。这两个车牌号可能恰好哈希到 1，因此它们会被分配到同一个车位。

由于碰撞，我们在会议注册处排起了队。假设一个会议注册过程，将根据姓氏的首字母将与会者分配到八个队列中。姓氏首字母为 A 到 D 的人去第 1 桌，姓氏首字母为 E 到 G 的人去第 2 桌，依此类推。如果与会者超过几个人，我们几乎可以确定会出现碰撞。如果没有碰撞，每个人都会有自己专属的注册处。然而，由于姓氏为 A 到 D 的与会者会排在同一队列中，因为他们的键（姓氏）发生了碰撞。

我们可以通过增加哈希表的大小或选择更好的哈希函数来缓解一些碰撞。然而，只要我们的键空间大于桶的数量，完全消除碰撞是不可能的。我们需要一种优雅地处理两个数据项争夺同一位置的方式。如果这是一个幼儿园班级，我们或许可以使用“安安先坐的”或“你得学会分享”这样的策略。然而，这些方法在数据结构的上下文中是行不通的。我们不能忽略新的键值，也不能覆盖旧的数据。数据结构的目的就是存储所有必要的数据。在接下来的两节中，我们将讨论链式法和线性探测法，这两种常见的处理哈希表碰撞的方法。

### 链接法

链接法是一种通过在桶内使用额外结构来处理哈希表中碰撞的方法。我们不再在每个桶中存储固定的数据项（或指向单一数据项的指针），而是可以存储指向链表头的指针：

```py
HashTable {
    Integer: size
    Array of ListNodes: bins
}
```

在

```py
ListNode {
    Type: key
 Type: value
    ListNode: next
}
```

这些列表就像我们会议注册处的排队人员。每个人都是独立的个体，但都映射到同一注册桌。

如图 10-4 所示，每个桶的列表包含所有映射到该桶的数据。这允许我们在每个桶中存储多个元素。我们链表中的每个项对应于插入该桶的一个元素。

![一个包含 5 个桶的数组垂直排列在左侧。每个桶都有一个箭头指向一个列表。第一个桶的箭头指向一个包含三个节点的链表的起始位置。](img/f10004.png)

图 10-4：使用链表将同一桶中的条目存储在哈希表中

将新项插入哈希表（使用链地址法）的代码相对简单：

```py
HashTableInsert(HashTable: ht, Type: key, Type: value):
  ❶ Integer: hash_value = HashFunction(key, ht.size)

    # If the bin is empty, create a new linked list.
  ❷ IF ht.bins[hash_value] == null:
        ht.bins[hash_value] = ListNode(key, value)
        return

    # Check if the key already exists in the table.
  ❸ ListNode: current = ht.bins[hash_value]
 WHILE current.key != key AND current.next != null:
        current = current.next
    IF current.key == key:
      ❹ current.value = value
    ELSE:
      ❺ current.next = ListNode(key, value)
    return
```

我们首先计算 `key` ❶ 的哈希值，并检查对应的桶。如果桶为空（指针为 `null`），我们会创建一个新的链表节点，保存插入的 `key` 和 `value` ❷。否则，我们需要扫描桶中的链表，检查每个元素是否与目标键匹配 ❸。`WHILE` 循环确保我们既没有找到正确的键（`current.key != key`），也没有超出链表的末尾（`current.next != null`）。如果链表中已经包含匹配的键，我们更新与该键关联的值 ❹。否则，我们将新键及其对应的值追加到链表的末尾 ❺。

查找采用类似的方法。然而，逻辑更简单，因为我们不再需要插入新的节点：

```py
HashTableLookup(HashTable: ht, Type: key):
  ❶ Integer: hash_value = HashFunction(key, ht.size)
  ❷ IF ht.bins[hash_value] == null:
        return null

    ListNode: current = ht.bins[hash_value]
  ❸ WHILE current.key != key AND current.next != null:
        current = current.next
    IF current.key == key:
      ❹ return current.value
  ❺ return null
```

查找的代码首先计算 `key` ❶ 的哈希值，检查对应的桶，如果桶为空则返回 `null` ❷。否则，它会使用 `WHILE` 循环遍历链表中的每个元素 ❸，并返回匹配键的值 ❹。如果我们在链表末尾仍未找到匹配的键，则代码返回 `null`，表示该键不在哈希表中 ❺。

最后，在删除项时，我们需要在列表中查找它，并且如果它存在，则将其删除。以下代码既删除也返回与目标键匹配的链表节点：

```py
HashTableRemove(HashTable: ht, Type: key):
  ❶ Integer: hash_value = HashFunction(key, ht.size)
    IF ht.bins[hash_value] == null:
        return null

    ListNode: current = ht.bins[hash_value]
    ListNode: last = null
  ❷ WHILE current.key != key AND current.next != null:
 last = current
        current = current.next
  ❸ IF current.key == key:
        IF last != null:
          ❹ last.next = current.next
        ELSE:
          ❺ ht.bins[hash_value] = current.next
        return current
    return null
```

代码再次从计算 `key` 的哈希值开始，检查对应的桶，如果桶为空则返回 `null` ❶。如果桶不为空，我们使用 `WHILE` 循环扫描它，并查找匹配的键 ❷。为了删除正确的元素，我们需要跟踪额外的信息：当前节点之前的最后一个链表节点。如果我们找到匹配项 ❸，需要检查我们是否正在删除链表中的第一个元素（`last` 为 `null`）。如果不是，我们可以修改 `last` 节点的 `next` 指针，跳过要删除的节点 ❹。否则，我们需要修改哈希桶开始位置的指针，跳过该节点 ❺。最后，如果未找到匹配的节点，则返回 `null`。

```py`The skeptical reader might pause here and ask, “How does this help? We still must scan through a bunch of elements of a linked list. We have lost the ability to directly map to a single entry. We’re back where we started.” However, the primary advantage to this new approach is that we are no longer scanning through a linked list of *all* the entries. We only scan through those entries whose hash values match. Instead of searching through a giant list, we search through a single tiny list for this bin. In our coffee pantry, where our hash function maps the coffee’s name to its corresponding shelf, we might be able to cull our search from 1,000 varieties to the 20 varieties on that one shelf. Back in the computational realm, if we maintain enough bins in our hash table, we can keep the size of these lists small, perhaps with only one or two elements.    Of course, the worst-case time for a lookup can be linear in the number of elements. If we choose a terrible hash function, such as *f*(*k*) = 1, we’re basically implementing a single linked list with extra overhead. It’s vital to be careful when selecting a hash function and sizing the hash table, as we’ll discuss later.    ### Linear Probing    An alternate approach to handling collisions is to make use of adjacent bins. If we are trying to insert data into a bin that already contains another key, we simply move on and check the next bin. If that neighbor is also full, we move onto the next. We keep going until we find an empty bin or a bin containing data with the same key. Linear probing extends this basic idea to the hash table’s operations. We start at the index corresponding to the key’s hash value and progress until we find what we are looking for or can conclude it is not in the hash table.    Hash tables using linear probing need a slightly different structure. Because we are not using a linked list of nodes, we use a wrapper data structure to store the combination of keys and values:    ``` HashTableEntry {     类型: 键     类型: 值 } ```py    We also include an additional piece of data in the hash table itself, the number of keys currently stored in the table:    ``` HashTable {     整数: 大小     整数: 键的数量     HashTableEntry 数组: 桶 } ```py    This information is critical, because when the number of keys reaches the size of the hash table, there are no available bins left. Often hash tables will increase the array size when they start to get too full, although care must be taken here. Because we are using a hash function that maps keys onto the range of the current array, keys may map to different locations in a larger array. In this section, we will only consider a simplified implementation of a fixed size table for illustration purposes.    Consider a hash table with linear probing where we have inserted a few of our favorite coffees, as shown in Figure 10-5.  ![The left column shows a list of three keys: Morning Shock, Liquid Alarm Clock, and Pure Caffeine. Arrows map these keys to hash functions and then to the corresponding bins in the array. Morning Shock maps to bin 1, Liquid Alarm Clock to bin 2, and Pure Caffeine to Bin 5.](img/f10005.png)    Figure 10-5: A hash table with three entries      After we have inserted these initial three entries, we try to insert “Morning Zap” as shown in Figure 10-6. The insertion function finds another key, Morning Shock, in bin 1\. It proceeds to bin 2, where it finds Liquid Alarm Clock. The insertion function finally finds an opening at bin 3.  ![The left column shows a single key, Morning Zap, which maps to a hash value of 1\. In the array on the right, bins 1 and 2 contain other keys. Arrows show our search through the array for an empty bin, found at bin 3.](img/f10006.png)    Figure 10-6: Inserting the entry “Morning Zap” requires scanning through several bins in the hash table.      The code for inserting items into a fixed-size hash table is shown below. As noted previously, it is possible to increase the size of the array when the hash table is full, but this adds complexity to ensure the items are mapped correctly to new bins. For now, we will return a Boolean to indicate whether the item could be inserted.    ``` HashTableInsert(HashTable: ht, 类型: 键, 类型: 值):   ❶ 整数: 索引 = HashFunction(键, ht.size)   ❷ 整数: 计数 = 0      HashTableEntry: 当前 = ht.bins[索引]   ❸ 当 当前 != null 且 当前.key != 键 且 计数 != ht.size:         索引 = 索引 + 1       ❹ 如果 索引 >= ht.size:             索引 = 0         当前 = ht.bins[索引]         计数 = 计数 + 1    ❺ 如果 计数 == ht.size:         返回 False    ❻ 如果 当前 == null:         ht.bins[索引] = HashTableEntry(键, 值)         ht.num_keys = ht.num_keys + 1     否则:       ❼ ht.bins[索引].值 = 值     返回 True ```py    The code starts the search at the new key’s hash value ❶. The code also maintains a count of bins it has checked to prevent infinite loops if the table is full ❷. This count is not needed if we use resizing or otherwise ensure there is always at least one empty bin. The code then loops through each bin using a `WHILE` loop ❸. The loop tests three conditions: (1) whether it found an empty bin, (2) whether it found the target key, and (3) whether it has searched all the bins. The first and third conditions test whether the key is not in the table. After incrementing the index, the code checks whether the index should wrap back to the beginning of the table ❹, which allows the code to search the entire table.    After the loop completes, the code first checks whether it has examined every bin in the table without finding the key. If so, the table is full and does not contain the key ❺, so the code returns `False`. The code then checks whether it has found an empty bin or matching key. If the bin is empty ❻, a new `HashTableEntry` is created and stored. Otherwise, the value of the entry with the matching key is updated ❼. The code returns `True` to indicate that it successfully inserted the key and value.    Search follows the same pattern. We start at the index of the key’s hash value and iterate over bins from there. At each step, we check whether the current entry is empty (`null`), in which case the key is not in the table, or whether the current entry matches the target key.    ``` HashTableLookup(HashTable: ht, 类型: 键):   ❶ 整数: 索引 = HashFunction(键, ht.size)   ❷ 整数: 计数 = 0      HashTableEntry: 当前 = ht.bins[索引]   ❸ 当 当前 != null 且 当前.key != 键 且 计数 != ht.size:         索引 = 索引 + 1       ❹ 如果 索引 >= ht.size:             索引 = 0         当前 = ht.bins[索引]         计数 = 计数 + 1      # 如果找到匹配项，则返回值。   ❺ 如果 当前 != null 且 当前.key == 键:         返回 当前.值   ❻ 返回 null ```py    The code starts by computing the hash value for the `key` to get the starting location of the search ❶. As with insertion, the code also maintains a count of bins it has checked to prevent infinite loops if the table is full ❷. The code then loops through each bin using a `WHILE` loop ❸. The loop tests three conditions: (1) whether it found an empty bin, (2) whether it found the target key, and (3) whether it has searched all the bins. After incrementing `index`, the code tests whether the search has run off the end of the table and, if so, wraps the search back to the start ❹. Once the loop terminates, the code checks whether it has found the matching key ❺. If so, it returns the corresponding value. Otherwise, the key is not in the table, and the code returns `null` ❻.    In contrast to search and insertion, deletion with linear probing requires more than a simple scan. If we remove an arbitrary element such as “Liquid Alarm Clock,” shown in Figure 10-6, we might break the chain of comparisons needed for other elements. If we replace “Liquid Alarm Clock” with null in Figure 10-6, we can no longer find “Morning Zap.” Different implementations use different solutions to this problem, from scanning through the table and fixing later entries to inserting dummy values into the bin.    The advantage of linear probing over chaining is that we make fuller use of the initial array bins and do not add the overhead of linked lists within the bins. The downside is that, as our table gets full, we might iterate over many entries during a search, and, unlike with chaining, these entries are not restricted to ones with matching keys.    ## Hash Functions    The difference between a good and bad hash function is effectively the difference between a hash table and a linked list. We’d want our hash function to map keys uniformly throughout the space of bins, rather than pile them into a few overloaded bins. A good hash function is like a well-run conference registration. Any clumping of attendees will lead to collisions, and collisions lead to more linear scanning (more time waiting in line). Similarly, in our conference registration example, bad hash functions, such as dividing the table into two lines for names starting with A−Y and names starting with Z, leads to long waits and annoyed attendees.    In addition, any hash function must be meet a couple of key criteria. It must:    1.  Be deterministic The hash function needs to map the same key to the same hash bin every time. Without this attribute, we might insert items into a hash table only to lose the ability to retrieve them. 2.  Have a predefined range for a given size The hash function needs to map any key into a limited range, corresponding to the number of hash buckets. For a table with *b* buckets, we need the function to map onto the range [0, *b* − 1]. We’d also like to be able to vary the hash function’s range with the size of our hash table. If we need a larger table, we need a correspondingly larger range to address all the buckets. Without this ability to adjust the range, we may be stuck with a limited number of viable hash table sizes.    In our conference registration example, these criteria correspond to people being able to find their packets (deterministic for all users) and having everyone map to a line (correct range). It would be wasteful to set up a conference check-in with lines for empty parts of the range (all names starting with Zzza through Zzzb), and it would be rude to map some people’s names to no line at all (no line for names starting with K). The best hash functions are like in-person organizers, holding clipboards and directing people to the correct lines.    ### Handling Non-numeric Keys    For numeric keys, we can often use a range of mathematical functions, such as the division method above. However, we may often need to handle non-numeric keys as well. Most importantly, we need to be able to compute the hash of a string containing the coffee name. The general approach to handling non-numeric keys is to use a function that first transforms the non-numeric input into a numeric value. For example, we could map each character to a numeric value (such as the ASCII value), compute the sum of values in a word, and modulo the resulting sum into the correct number of buckets. This approach, while straightforward and easy to implement, may not provide a good distribution of hash values due to how letters occur. For example, the hash function does not take into account the order of the letters, so words with the same letters, such as *act* and *cat*, will always be mapped to the same bin.    A better approach to hashing strings is an approach often called *Horner’s method*. Instead of directly adding the letters’ values, we multiply the running sum of letters by a constant after each addition:    ``` StringHash(字符串: 键, 整数: 大小):     整数: 总计 = 0     对每个 字符 在 键 中:         总计 = CONST * 总计 + CharacterToNumber(字符)     返回 总计 % 大小 ```py    where `CONST` is our multiplier, typically a prime number that is larger than the largest value for any character. Of course, by multiplying our running sum, we greatly increase the size of the value, so we need to be careful to handle overflow.    There are a wide variety of hash functions, each with their own trade-offs. A full survey of the potential hash functions and their relative merits is a topic worthy of its own book; this chapter presents just a few simple functions for the point of illustration. The key takeaway is that we can use these mathematical functions to reduce the range of our key space.    ### An Example Use Case    Hash tables are particularly useful when tracking a set of items, which is why Python uses them to implement data structures such as dictionary and set. We can use them to aid in tracking metadata for the searches such as those seen in Chapter 4.    In both the depth-first and breadth-first search, we maintained a list of future options to explore. We used a queue for breadth-first search and a stack for depth-first. However, there is an additional set of information we need to track: the set of items we have already visited. In both searches, we avoid adding items to the list that we have already seen, allowing us to avoid loops or, at least, wasted effort. Hash tables provide an excellent mechanism for storing this set.    Consider the sixth step of our example breadth-first search from Chapter 4, as shown on the left side of Figure 10-7. The search has already visited the gray nodes (A, B, F, G, H, and I), and the circled node (G) is our current node. We do not want to re-add either of G’s two neighbors to our list of items to explore; we have already visited both F and I.  ![Diagram of a breadth‐first search in progress on the left. A hash table on the right stores the visited nodes.](img/f10007.png)    Figure 10-7: The visited nodes of a breadth-first search tracked in a hash table      We could store the visited items in a hash table as shown on the right of Figure 10-7. This hash table uses a simple function: the division function maps the letter’s index in the alphabet into 5 bins. The letter *A* has index 0 and maps to bin 0\. The letter *G* has index 6 and maps to bin 1\. We insert a key for each item when we visit it. Before adding new items to our list of future topics, we check whether their key is in the hash table. If so, we skip the addition. Hash tables are particularly well suited to tracking this type of data because they are optimized for fast insertions and lookups.    ## Why This Matters    Hash tables provide a new way of organizing data by using a mathematical function. As opposed to tree-based structures, which structure the data around the keys themselves, hash tables introduce the intermediate step of mapping the keys to a reduced range. As with all things in computer science, this mapping comes with tradeoffs. Collisions mean that we can’t always access the desired data directly. We must add another level of indirection, such as a linked list, to store items that map to the same location. As such, hash tables provide another example of how we can creatively combine data structures (in this case, an array and linked lists).    Understanding hash functions and how they map keys to bins provides a side benefit. We can use these types of mappings to partition items or spread out work, such as the lines at a conference registration or the coffees on our shelves; in the computational domain, we might use hashing to assign tasks to servers in a simple load balancer. In the next chapter, we see how hash tables can be used to create caches. Hash tables are used throughout computer science, since they provide a handy data structure with (on average) fast access time and reasonable memory tradeoffs. They are a vital tool for every computer scientist’s toolbox.````
