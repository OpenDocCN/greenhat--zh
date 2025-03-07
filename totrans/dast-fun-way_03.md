# 3

动态数据结构

![](img/chapterart.png)

本章介绍了*动态数据结构*，它们会随着数据的变化而改变结构。这些结构的调整可能包括根据需要增长数据结构的大小，创建动态的、可变的链接以连接不同的值等。动态数据结构是几乎所有计算机程序的核心，并且是一些最令人兴奋、最有趣和最强大的计算机科学算法的基础。

前几章介绍的基本数据结构就像停车场——它们为我们提供了存储信息的地方，但不提供太多的适应性。当然，我们可以对数组中的值（或者说停车场中的车）进行排序，利用这种结构使得二分查找更加高效。但是，我们仅仅是改变了数组中数据的顺序。数据结构本身既没有发生变化，也没有对数据的变化做出响应。如果我们后来修改了排序数组中的数据，比如改变某个元素的值，我们就需要重新排序数组。更糟糕的是，当我们需要改变数据结构本身时——例如扩展或缩小数组——简单的静态数据结构并不会提供任何帮助。

本章将第一章中介绍的静态数据结构——数组，与一种简单的动态数据结构——链表进行比较，以展示后者的优势。在某些方面，这两种数据结构是相似的：它们都允许程序员通过一个引用（无论是数组还是链表的头部）来存储和访问多个值。然而，数组在创建时的结构是固定的，就像停车位的行列。而链表则可以在程序的内存中不断增长。它们更像是一个不断伸长或缩短的排队队伍，允许随时进行增减。理解这些差异为理解本书后续将讨论的更高级数据结构提供了基础。

## 数组的局限性

虽然数组是存储多个值的优秀数据结构，但它们有一个重要的局限性：它们的大小和内存布局在创建时是固定的。如果我们想要存储比数组所能容纳的更多的值，就需要创建一个新的、更大的数组，并将旧数组中的数据复制过来。这种固定大小的内存对于当我们有一个不变的上限时是可以接受的。如果我们有足够的空间来存储数据，我们可以随时设置各个条目，而不必担心数组在内存中的静态布局。然而，许多应用程序需要动态数据结构，这些结构能够随着程序的变化而增长和变化。

为了满足对动态数据结构的需求，许多现代编程语言提供了动态“数组”，它们会随着你添加元素而增长或缩小。然而，这些实际上是围绕静态数组或其他数据结构的包装器，隐藏了与它们动态特性相关的复杂性和成本。虽然这对程序员来说很方便，但它可能导致隐藏的低效。当我们向数组的末尾添加元素时，程序仍然需要增加所使用的内存。它只是在幕后完成这个操作。为了理解动态数据结构为何如此重要，我们需要讨论静态数据结构的限制。在本书中，我们将使用*数组*一词来指代简单的静态数组。

为了说明数组的限制，假设你花了一整周时间掌握最新的复古视频游戏现象《太空青蛙 2000》。每次主屏幕显示你的五个最高分时，你都会开心地笑出声。这些辉煌的成就代表了无数小时的汗水、眼泪、喊叫和更多的眼泪。然而，第二天，你的（很快就会成为前）最好的朋友来访，并连续五次击败了你的最高分。当你把背叛的前朋友赶出家门后，你回到游戏中，凝视着新显示的最高分，如图 3-1 所示，忍不住喊道：“为什么游戏不能保存更多的分数？真的很难保留一个前十名榜单，或者至少在最后加一个分数吗？”

![图示，显示一个索引，包含五个插槽，用于记录视频游戏的最佳分数。最高分 1025 位于索引 0，最低分 949 位于索引 4。](img/f03001.png)

图 3-1：一个包含视频游戏高分的五元素数组。遗憾的是，里面没有你的分数。

这是任何固定大小数据结构及其在内存中固定布局的基本限制之一——它无法随着数据的增加而增长。正如我们下面所看到的，这种限制使得某些常见操作变得昂贵。更实际的说，想象一下一个只能容纳固定字符数的文字处理器，一个只有固定行数的电子表格，一个只能存储有限数量照片的照片存储程序，或一个最多只能记录一千条记录的咖啡日记。

由于数组的大小在创建时是固定的，如果我们想要扩展数组以存储更多的数据，就必须创建一个新的、更大的内存块。考虑最简单的情况，向数组的末尾添加一个元素。由于数组是一个单一的固定大小的内存块，我们不能只是将另一个值插入到末尾。内存中可能已经有另一个变量占用了这个空间。为了避免覆盖该变量的值，我们必须分配一个新的（更大的）内存块，将原数组中的所有值复制到新块中，并将新值写入末尾。这对于一个简单的增加来说是大量的开销，正如图 3-2 所示。

![图示显示了一个包含六个数据元素的数组被复制，接着在复制的数组末尾添加了第七个数据元素。](img/f03002.png)

图 3-2：向满载的数组末尾添加元素。

可以将数组比作一个有固定数量插槽的加热酒店自助餐台。很容易把空的炒蛋托盘拿掉，换上新的托盘。但你不能简单地把一个新托盘加到末尾。没有空间。如果厨师决定把煎饼加入菜单，其他的东西必须被拿走。

如果你知道你需要插入大量的新值，你可能会将成本分摊到多个更新中，*摊销*成本。你可能会采用像*数组扩容*这样的策略，即每当数组扩展时，其大小会翻倍。例如，如果我们尝试向一个大小为 128 的数组添加第 129 个元素，我们首先分配一个大小为 256 的新数组，并将原来的 128 个元素复制过来。这让我们可以在不担心再次分配新空间之前继续扩展数组一段时间。然而，这种方法的代价可能是浪费空间。如果我们只需要 129 个元素，总共多分配了 127 个空间。

数组扩容提供了在高昂的数组复制成本和浪费内存之间的合理平衡。随着数组的增长，扩容的频率会越来越低。同时，通过在数组满时进行扩容，我们可以确保浪费的空间不会超过一半。然而，即使采用这种平衡方法，我们仍然能清晰地看到使用固定大小数组在复制成本和内存使用方面的代价。

```py
ArrayDouble(Array: old_array):
    Integer: length = length of old_array
    Array: new_array = empty array of size length * 2

    Integer: j = 0
    WHILE j < length:
        new_array[j] = old_array[j]
        j = j + 1
    return new_array
```

数组扩容的代码首先分配一个新数组，新数组的大小是当前数组的两倍。一个单一的`WHILE`循环遍历当前数组中的元素，将它们的值复制到新数组中。新数组随后被返回。

想象将这种策略应用到书架空间上。我们在某个地点开了一家名为“数据结构与更多”的书店，安装了五个简单的书架。开业当天需求超出预期，大家要求更多的品种：我们需要扩展库存。慌乱之中，我们搬到了一个有 10 个书架的新地点，并迁移了书籍。需求暂时得到满足。由于零售市场上缺乏全面的数据结构书店，这家店大获成功，需求继续增长。我们可能会将店面升级几次，搬到拥有 20、40，再到 80 个书架的地点。每次我们都需要支付一定成本来确保新的地点并迁移书籍。

数组值在内存中的固定位置带来了另一个限制。我们不能轻松地在数组中间插入新的元素。即使在原始数组末尾有足够的空闲空间来容纳新元素，因此我们不需要将整个数组移动到新的内存块中，我们仍然需要逐个移动每个现有元素，为新值腾出空间。与书架上的书籍不同，我们不能通过一次用力就将所有元素推到一边。如果我们有 10,000 个元素，并且想要在第二个位置添加一个元素，我们需要移动 9,999 个元素。为了插入一个元素，这需要付出很大的努力。

当我们试图将新值插入到已经满的数组的中间时，问题变得更加复杂。不仅需要分配一个新的内存块并复制旧值，还需要将新值之后的元素向下移动一个位置，为新值腾出空间。例如，假设我们想将值 23 插入到一个已有六个元素的数组的第四个位置，如图 3-3 所示。

![一个包含六个盒子的数组被分成两部分。每一半的值被复制并移动到一个新的七盒子数组中，中间插入一个新值。](img/f03003.png)

图 3-3：在满数组的中间添加一个元素

为了弥补数组的不足，我们需要转向更灵活的数据结构，这些结构能够随着新数据的加入而增长：动态数据结构。在我们深入细节之前，让我们先介绍指针这一变量类型，它是重新配置和增长数据结构的关键。

## 指针与引用

有一种变量类型在其强大的功能和使新程序员困惑的能力上，超越了其他类型：*指针*。指针是一个只存储计算机内存中地址的变量。因此，指针指向内存中的第二个位置，实际数据存储在该位置，如图 3-4 所示。

![八个内存地址，每个地址对应一个盒子。第四个盒子地址是 2103，值是 109，标记为“指针指向的地址。”第七个盒子地址是 2106，值是 2103，标记为“指针。”](img/f03004.png)

图 3-4：指针表示计算机内存中的一个地址

精明的读者可能会问：“一个变量只是指向内存中另一个位置，它有什么意义？我以为变量的名字已经起到了这个作用。为什么不把数据像正常人一样直接存储在变量里？为什么你们总是要把事情搞得这么复杂？”不要听那些怀疑者的声音。指针是动态数据结构的核心组成部分，稍后我们会看到这一点。

假设我们正在办公室进行一个大型建筑项目，并且已经整理了一份示例图纸文件夹与团队分享。很快，项目文件夹里便包含了多个楼层平面图、成本估算和艺术效果图。与其复制这份沉重的文件并公开放置，我们留下了一张纸条，告诉合作者们去三楼的档案室找文件，档案柜 #3，第二个抽屉，第五个文件夹。这张纸条就扮演了指针的角色。它并没有详细列出文件里的所有信息，而是允许我们的同事们找到并提取其中的信息。更重要的是，我们可以与每个同事共享这一个“地址”，而不必为他们复制整个文件。他们每个人都可以利用这个信息，在需要时查找并修改文件夹。我们甚至可以在每个团队成员的桌上留下单独的便签，提供 10 个指向相同信息的变量。

除了存储内存块的位置外，指针还可以拥有一个空值（在某些编程语言中用 None、Nil 或 0 表示）。空值仅表示指针当前没有指向一个有效的内存位置。换句话说，它表示指针实际上还没有指向任何东西。

不同的编程语言提供了不同的机制来实现指针的功能，并且并非所有语言都会向程序员提供原始的内存地址。像 C 和 C++ 这样的低级语言提供原始指针，允许你直接访问它们存储的内存位置。其他编程语言，如 Python，则使用引用，这种引用的语法与普通变量相似，但仍然允许你引用另一个变量。这些不同的实现方式有不同的行为和用法（如解除引用、指针运算、空值的形式等）。为了简化起见，在本书中我们将使用*指针*一词来涵盖所有通过指针、引用或索引访问预分配内存块实现的变量。我们不必担心访问内存块所需的复杂语法（这曾让不少编程爱好者泪如雨下）。我们在定义伪代码中的指针变量时，也将使用最终数据类型（而非更通用的类型指针）。对于我们来说，关键概念是指针提供了一种机制，用于链接到一块内存，这正是我们第一个动态数据结构——链表——的核心。

## 链表

*链表* 是最简单的动态数据结构示例，也是数组的近亲。像数组一样，它们是一种用于存储多个值的数据结构。不同于数组，链表由一系列由指针链接在一起的节点组成。链表中的基本*节点*是一个复合数据结构，包含两个部分：一个值（可以是任何类型）和指向列表中下一个节点的指针：

```py
LinkedListNode {
    Type: value
    LinkedListNode: next
}
```

我们可以将链表看作是一系列相互连接的容器，如图 3-5 所示。每个容器存储一个值，并包含指向下一个容器的指针。

![六个节点，每个节点包含一个数值。指向右的箭头连接节点。最后一个箭头指向右侧的斜线，表示链表的结束。](img/f03005.png)

图 3-5：一个由指针连接的节点序列所表示的链表

链表末尾的斜线表示空值，指示链表的结束。实际上，我们是在说最后一个节点的`next`指针不指向一个有效的节点。

链表就像我们最喜欢的咖啡店前排队的长队伍。人们很少知道自己在队伍中的绝对位置——“我在离柜台五十三块地砖的位置。”他们关注的是自己相对的位置，也就是排在自己前面的人，这一点我们通过指针来存储。即使队伍在店内（甚至停车场）绕来绕去，形成复杂的环状结构，我们仍然可以通过询问每个人前面是谁来重建队伍的顺序。我们可以通过询问每个人前面是谁来向柜台移动。

因为链表包含指针和数值，它们比数组在存储相同数据时需要更多的内存。如果我们有一个大小为*K*的数组，每个元素的大小为*N*字节，那么我们只需要*K* × *N*字节。相比之下，如果每个指针需要额外的*M*字节，那么我们的数据结构的内存开销将是*K* × (*M* + *N*)字节。除非指针的大小远小于数据值的大小，否则这种开销是显著的。然而，增加的内存使用通常是值得的，因为指针提供的灵活性。

虽然教科书通常将链表表示为整齐有序的结构（如图 3-5 所示，或在我们的人队列示例中所暗示的那样），但实际上我们的链表可能分散在程序的内存中。正如图 3-6 所示，链表的节点仅通过它们的指针连接。

这就是指针和动态数据结构的真正威力。我们不必将整个链表保存在一个连续的内存块中。我们可以在任何有空闲空间的地方为新的节点分配空间。

![与图 3-5 中相同的六个节点和表示链表结束的斜线，不过顺序不同，并且节点之间有多个空的容器。箭头表示节点连接的顺序。](img/f03006.png)

图 3-6：计算机内存中的链表。节点不一定彼此相邻。

程序通常通过保留指向链表前端或*头部*的指针来存储链表。然后，程序可以从头部开始，通过指针遍历节点，从而访问链表中的任何元素：

```py
LinkedListLookUp(LinkedListNode: head, Integer: element_number):
  ❶ LinkedListNode: current = head
    Integer: count = 0

  ❷ WHILE count < element_number AND current != null:
        current = current.next
        count = count + 1
    return current
```

代码从列表的头部开始❶。我们维护一个第二个变量`count`来跟踪当前节点的索引。然后，`WHILE`循环遍历列表中的每个节点，直到找到正确的数字，`count == element_number`，或者遍历到列表末尾，`current == null`❷。无论哪种情况，代码都会返回`current`。如果循环由于遍历到列表边缘而终止，那么索引不在列表中，代码会返回`null`。

例如，如果我们想访问链表的第四个元素，程序将依次访问头节点、第二、第三和第四个元素，以找到正确的内存位置。图 3-7 展示了这个过程，其中值为 3 的节点是列表的头部。

![六个节点的链表，程序首先访问列表头节点，然后访问直接连接到它的下一个节点，以此类推。](img/f03007.png)

图 3-7：遍历链表需要沿着指针链从一个节点移动到下一个节点。

然而，值得注意的是，这存在一个权衡：链表比数组有更高的计算开销。当访问数组中的元素时，我们只需计算一个偏移量并查找正确的内存地址。无论选择哪个索引，数组访问只需要一次数学计算和一次内存查找。链表则需要我们从列表的开头开始遍历，直到找到感兴趣的元素。对于较长的列表，缺乏直接访问可能会增加显著的开销。

初看之下，这种受限的访问模式似乎是链表的一大缺点。我们大幅增加了查找任意元素的成本！想一想这对二分查找意味着什么。一次查找需要遍历多个元素，失去了排序列表的优势。

然而，尽管存在这些成本，链表在实际程序中仍然可以成为宝贵的资源。数据结构几乎总是涉及复杂性、效率和使用模式之间的权衡。使得某个数据结构不适合某种用途的行为，可能使它成为支持其他算法的完美选择。理解这些权衡是有效结合算法和数据结构的关键。以链表为例，访问元素的开销增加的权衡是整体数据结构灵活性的显著提升，正如我们将在下一节中看到的那样。

## 链表的操作

尽管有些人抱怨链表的杂乱无章，相较于紧凑数组的美学，但正是这种跨不同内存块链接的能力使得数据结构如此强大，允许我们*动态*地重新排列数据结构。让我们将向数组中插入新值与向链表中添加值进行比较。

### 插入到链表中

正如我们所见，将新元素插入数组中可能需要我们分配一个新的（更大的）内存块，并将原数组中的所有值复制到新块中。此外，插入操作本身可能需要我们遍历数组并移动元素。

另一方面，链表不需要保持在一个单一的连续内存块中——它可能一开始就不在单一块中。我们只需要知道新节点的位置，更新前一个节点的 `next` 指针指向新节点，并将新节点的 `next` 指针指向正确的节点。如果我们想将值为 23 的节点添加到 图 3-5 中链表的前端，我们只需将新节点的 `next` 指针指向原链表的起始节点（值 = 3）。这个过程如 图 3-8 所示。任何之前指向链表起始位置（第一个节点）的变量也需要更新为指向新的第一个节点。

![图 3-5 中的链表，新增节点已添加到前端。右箭头将新的第一个节点链接到现在的第二个节点。](img/f03008.png)

图 3-8：通过将新节点添加到前端来扩展链表

同样，我们可以通过遍历链表直到末尾，然后更新最后一个节点的 `next` 指针（值 = 8）指向新节点，并将新节点的 `next` 指针设为 `null`，来将节点添加到链表的末尾，正如 图 3-9 所示。这个方法天真地实现时，需要遍历整个数组到达末尾，但正如我们在下一章将看到的那样，有方法可以避免这种额外的成本。

![图 3-5 中的链表，新增节点已添加到末尾。右箭头将原本的最后一个节点链接到新的最后一个节点，右箭头将新的最后一个节点链接到表示链表结束的斜杠。](img/f03009.png)

图 3-9：通过将一个附加节点附加到末尾来扩展链表

如果我们想在中间插入一个值，需要更新两个指针：前一个节点和插入的节点。例如，要在节点 *X* 和节点 *Y* 之间添加节点 *N*，我们有两个步骤：

1.  将 *N* 的 `next` 指针指向 *Y*（即当前 *X* 的 `next` 指针指向的地方）。

1.  将 *X* 的 `next` 指针指向 *N*。

这两个步骤的顺序非常重要。指针就像所有其他变量一样，只能保存一个单一的值——在这个案例中是内存中的一个地址。如果我们先设置 *X* 的 `next` 指针，那么我们将丧失 *Y* 所在位置的数据。

完成后，*X* 指向 *N*，*N* 指向 *Y*。图 3-10 展示了这个过程。

![将新节点 *N* 插入到链表中，位于节点 *X* 和节点 *Y* 之间的过程示意图。插入之前，*X* 指向 *Y*；插入过程中，*N* 和 *X* 都指向 *Y*；插入后，*X* 指向 *N*，*N* 指向 *Y*。](img/f03010.png)

图 3-10：在链表中插入新节点 *N* 到节点 *X* 和 *Y* 之间的过程。

尽管指针有了重新排列，这种操作的代码相对简单：

```py
LinkedListInsertAfter(LinkedListNode: previous,
                      LinkedListNode: new_node):
    new_node.next = previous.next
    previous.next = new_node
```

假设我们想将一个值为 23 的节点插入到当前链表中节点 9 和 37 之间。结果指针链将如图 3-11 所示。

![图 3-5 中的链表，节点 9 曾指向节点 37。现在，节点 9 到新插入的节点 23 有一条右箭头指向，节点 23 到节点 37 也有一条右箭头指向。](img/f03011.png)

图 3-11：将节点 23 插入到链表中需要更新来自前一个节点（9）和下一个节点（37）的指针。

同样地，当一位顾客让他们的朋友插到队伍的中间时，两个指针会发生变化。回想一下，在这个类比中，每个人都会“指向”或记录前面的人。这位过于慷慨的顾客现在指向了站在他们前面的插队朋友。与此同时，开心的插队者则指向了之前在他们帮忙的朋友前面的人。队伍中其他人在他们后面纷纷投来不满的目光，并低声抱怨不友善的话语。

再次强调，图示和咖啡馆排队的类比掩盖了插入过程的真正复杂性。虽然我们并没有将新节点插入到紧挨着最后一个节点的内存位置，但我们在逻辑上确实是将它插入到队列的下一个位置。这个节点本身可能位于计算机内存的另一端，紧挨着记录我们拼写错误或每日咖啡杯数的变量。只要我们保持列表指针的更新，我们就可以把它们和它们指向的节点当作一个整体列表来处理。

当然，在插入节点到头节点前（`index == 0`）或插入到超出列表末尾的索引时，我们必须格外小心。如果我们在头节点前插入一个节点，我们需要更新头指针本身；否则，它将继续指向旧的列表开头，我们将无法访问新的第一个元素。如果我们尝试将一个节点插入到超出列表末尾的索引，那么在`index - 1`处就没有有效的前一个节点。在这种情况下，我们可能会失败插入、返回错误，或者将元素追加到列表的末尾（在一个较小的索引处）。无论你选择哪种方法，清晰地记录你的代码至关重要。我们可以将这额外的逻辑打包成一个辅助函数，将线性查找代码和在给定位置插入新节点的功能结合起来：

```py
LinkedListInsert(LinkedListNode: head, Integer: index,
                 Type: value):
    # Special case inserting a new head node.
  ❶ IF index == 0:
        LinkedListNode: new_head = LinkedListNode(value)
        new_head.next = head
        return new_head

    LinkedListNode: current = head
    LinkedListNode: previous = null
    Integer: count = 0
  ❷ WHILE count < index AND current != null:
        previous = current
        current = current.next
        count = count + 1

    # Check if we've run off the end of the list before
    # getting to the necessary index.
  ❸ IF count < index:
        Produce an invalid index error.

  ❹ LinkedListNode: new_node = LinkedListNode(value)
    new_node.next = previous.next
    previous.next = new_node

  ❺ return head
```

插入的代码从特殊情况开始，即在`index = 0`（列表的开头）插入新节点 ❶。它创建一个新的头节点，将新头节点的`next`指针设置为列表之前的头节点，并返回新的头节点。由于新头节点前面没有节点，因此我们在这种情况下无需更新前一个节点的`next`指针。

对于列表中间的元素，代码需要遍历列表以找到正确的位置❷。这类似于`LinkedListLookUp`查找：代码沿着每个节点的`next`指针前进，同时跟踪`current`节点和已看到的`count`，直到找到列表末尾或正确的位置。代码还会跟踪一个额外的信息，即`previous`，它是指向当前节点*之前*的节点的指针。跟踪`previous`可以让我们更新指向新插入节点的指针。

代码接着检查是否到达了所需的插入索引❸。通过进行`count < index`的检查，我们仍然允许在列表的末尾进行插入。只有在我们试图插入列表末尾*之后*至少一个位置时，才会导致错误。

如果代码找到了正确的位置来插入节点，它会将其拼接在`previous`和`current`之间。代码通过创建一个新节点，设置该节点的`next`指针指向`previous.next`所指示的地址，然后将`previous.next`指向新节点，从而执行插入❹。这个逻辑同样适用于我们将新节点立即添加到列表最后一个节点之后的情况。因为在这种情况下，`previous.next == null`，所以新节点的`next`指针被赋值为`null`，正确地表示了列表的末尾。

通过返回列表的头节点❺，我们可以处理插入发生在头节点之前的情况。或者，我们也可以将头节点封装在一个`LinkedList`复合数据结构中，并直接在其上操作。我们将在本书后面使用这种替代方法来处理二叉搜索树。

### 从链表中删除

删除链表中的任何元素时，我们只需要删除该节点并调整前一个节点的指针，如图 3-12 所示。

![图 3-9 中的七节点链表。节点 37 位于中间，节点 9 在它之前，节点 7 在它之后。节点 37 被删除，节点 9 现在指向节点 7。](img/f03012.png)

图 3-12：从链表中移除节点（37）需要更新前一个节点（9）的指针，以跳过并指向下一个节点（7）。

这就像是有人做出一个可疑的决定，认为咖啡排队不值得等待。他们看看手表，嘀咕着自己家里有速溶咖啡，然后离开了。只要离开后的顾客后面的人知道自己现在排在谁后面，队列的完整性就得以保持。

对于数组，我们删除一个元素的成本要高得多，因为我们需要将包含 37 的节点后面的所有元素向前移动一个位置以填补空缺。这可能需要我们遍历整个数组。

同样，当删除链表中的第一个元素或删除超出链表末尾的节点时，我们必须特别小心。删除第一个节点时，我们需要更新链表的头指针，指向新的头节点地址，从而有效地将该节点设为链表的新头节点。当删除超出链表末尾的节点时，我们有类似于插入时的选择：我们可以跳过删除或返回错误。下面的代码执行了后者：

```py
LinkedListDelete(LinkedListNode: head, Integer: index):
  ❶ IF head == null:
        return null

  ❷ IF index == 0:
        new_head = head.next
        head.next = null
        return new_head

    LinkedListNode: current = head
    LinkedListNode: previous = null
    Integer: count = 0
  ❸ WHILE count < index AND current != null:
        previous = current
        current = current.next
        count = count + 1

  ❹ IF current != null:
      ❺ previous.next = current.next
      ❻ current.next = null
    ELSE:
        Produce an invalid index error.
  ❼ return head
```

这段代码遵循与插入相同的方法。这次我们开始时做了额外的检查 ❶。如果链表为空，则没有节点可删除，我们可以返回`null`表示链表仍然为空。否则，我们检查是否正在删除第一个节点 ❷，如果是，删除链表中的第一个节点，并返回新头节点的地址。

要删除任何后续节点（`index > 0`），代码必须遍历到列表中的正确位置。使用与插入相同的逻辑，代码在迭代节点时跟踪`current`、`count`和`previous`，直到它找到正确的位置或遇到列表的末尾 ❸。如果代码在正确的索引处找到一个节点 ❹，它通过将`previous.next`指向当前节点之后的一个节点来删除要移除的节点 ❺。然而，如果`WHILE`循环在列表末尾超出并且`current`为`null`，则没有节点可删除，因此代码会抛出错误。该函数还将被删除节点的`next`指针设置为`null`，既是为了确保一致性（该节点不再有`next`节点），也是为了让具备内存管理功能的编程语言能够正确地释放不再使用的内存 ❻。函数最后返回列表头节点的地址 ❼。

我们可以调整这段代码，使用节点的其他信息来删除。如果我们知道要删除的节点的值，我们可以更新循环条件 ❸，删除第一个具有该值的节点：

```py
 WHILE current != null AND current.value != value:
```

在这种情况下，我们需要反转比较顺序，并在访问`current`的值之前检查它是否为`null`。类似地，如果我们需要根据指针删除一个节点，我们可以将该指针存储的地址与当前节点的地址进行比较。

```py`The strength of linked lists is that they allow us to insert or remove elements without shifting those elements around in the computer’s memory. We can leave the nodes where they are and just update the pointers to indicate their movement.    ## Doubly Linked Lists    There are many additional ways we can add structure with pointers, many of which we’ll examine in later chapters. For now, we’ll discuss just one simple extension of the linked list: the *doubly linked list*, which includes backward as well as forward pointers, as shown in Figure 3-13.  ![The list from Figure 3‐5, now with right‐pointing and left‐pointing arrows between each node. ](img/f03013.png)    Figure 3-13: A doubly linked list contains pointers to both the next and previous entries.      For algorithms that need to iterate lists in both directions, or just for adventurous programmers looking to expand the number of pointers in their data structures, it is easy to adapt a linked list to a doubly linked one:    ``` DoublyLinkedListNode {     类型: 值     DoublyLinkedListNode: next     DoublyLinkedListNode: previous } ```py    The code for operating on doubly linked lists is similar to the code for singly linked lists. Lookups, insertions, and deletions often require traversing the list to find the correct element. Updating the appropriate nodes’ `previous` pointers in addition to their `next` pointers requires additional logic. Yet this small amount of additional information can enable shortcuts to some of the operations. Given the pointer to any node in a doubly linked list, we can also access the node before it without having to traverse the entire list from the beginning, as we would have to do for a singly linked list.    ## Arrays and Linked Lists of Items    So far, we have primarily used arrays to store individual (numeric) values. We might be storing a list of top scores, a list of reminder times for a smart alarm clock, or a log of how much coffee we consume each day. This is useful in a variety of applications but is only the most basic way to use an array. We can use the concept of pointers to store more complex and differently sized items.    Suppose you’re planning a party. We will make the generous assumption that, unlike many parties thrown by the author, your gathering is popular enough to require an RSVP list. As you begin to receive responses to your invitations, you write a new program using an array to keep track of the guests. You’d like to store at least a single string in each entry, indicating the name of the person who has responded. However, you immediately run into the problem that strings might not be fixed size, so you can’t guarantee they will fit in the array’s fixed-size bin. You could expand the bin size to fit all possible strings. But how much is enough? Can you reliably say all your invitees will have fewer than 1,000 characters in their name? And if we allow for 1,000 characters, what about the waste? If we are reserving space for 1,000 characters per invitee, then entries for “John Smith” are using only a tiny fraction of their bins. What if we want to include even more dynamic data with each record, such as a list of each guest’s music preferences or nicknames?    The natural solution is to combine arrays and pointers, as shown in Figure 3-14. Each bin in the array stores a single pointer to the data of interest. In this case, each bin stores a pointer to a string located somewhere else in memory. This allows the data for each entry to vary in size. We can allocate as much space as we need for each string and point to those strings from the array. We could even create a detailed composite data structure for our RSVP records and link those from the array.  ![Array in which each value points to an ellipse (three dots) outside of the array, indicating that the values link to larger data structures. ](img/f03014.png)    Figure 3-14: Arrays can store a series of pointers, allowing them to link to larger data structures.      The RSVP records don’t need to fit into the array bins, because their data lives somewhere else in memory. The array bins only hold (fixed-size) pointers. Similarly, a linked list’s nodes can contain pointers to other data. Unlike the `next` pointers in a linked list, which are pointing to other nodes, these pointers can point to arbitrary other blocks of data.    The rest of the book includes many cases where individual “values” are actually pointers to complex and even dynamic data structures.    ## Why This Matters    Linked lists and arrays are only the simplest example of how we can trade off among complexity, efficiency, and flexibility in our data structures. By using a pointer, a variable that stores addresses in memory, we can link across blocks of memory. A single fixed-size array bin can point to complex data records or strings of different lengths. Further, we can use pointers to create dynamically linked structures through the computer’s memory. By changing a pointer’s value to point to a new address, we can change this structure as needed at any time.    Over the remaining chapters, we will see numerous examples of how dynamic data structures can be used to both improve organization of the data and make certain computations more efficient. However, it is important to keep the relative tradeoffs in mind. As we saw with arrays and linked lists, each data structure comes with its own advantages and disadvantages in terms of flexibility, space requirements, efficiency of operations, and complexity. In the next chapter, we will show how can build on these fundamental concepts to create two data structures, stacks and queues, that enable different behavior.```
