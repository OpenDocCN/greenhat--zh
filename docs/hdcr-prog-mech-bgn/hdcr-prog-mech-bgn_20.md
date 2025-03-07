## 15

结构模型

![Image](../images/common01.jpg)

在本书的这一部分，我们将重点解决桁架结构问题。桁架结构用于支撑工业仓库的屋顶（见[图 15-1](ch15.xhtml#ch15fig1)）以及大跨度桥梁。这是一个实际的工程问题，作为构建一个应用程序的好例子，程序可以从文件中读取数据，基于这些数据构建模型，求解线性方程组，并通过图表呈现结果。

由于解决桁架结构问题是一个庞大的课题，我们将其分解成几章内容。第一章将为你粗略介绍材料力学的基础知识；这并不是从零开始解释概念，而是作为一种复习。当我们掌握了基础知识后，我们将实现两个类来模拟桁架结构：节点和杆件。正如我们在前面的章节中看到的，解决一个问题的第一步是拥有一组基本元素来表示解决方案中涉及的实体。

![Image](../images/15fig01.jpg)

*图 15-1：仓库屋顶是桁架结构的一个典型例子。*

### **解决结构问题**

我们先来定义几个术语。*结构*是由一系列抗力构件组成，用于承受外部负荷的作用及其自身的重量。*桁架结构*是一种由两端通过销钉连接的杆件构成的结构，外部力仅作用于这些杆件连接的地方，即节点。

在解决结构问题时，我们最关心的两件事是：第一，结构中的杆件能否承受作用在其上的力并避免坍塌？第二，当结构在外力作用下变形时，其位移有多大？第一个问题显而易见：如果结构中的任何杆件断裂，结构可能会坍塌，造成灾难性的后果（例如：仓库屋顶或桥梁坍塌）。我们的分析应该确保这种情况永远不会发生。

第二个问题不那么显而易见，但同样重要。如果一个结构发生了足够明显的变形，哪怕结构本身是安全的，不会坍塌，周围或下面的人可能会感到不安。想象一下，如果你看到客厅的天花板明显弯曲，你会有什么感觉。将结构的变形限制在一定范围内对其使用者的舒适度有着重要影响。

我们要寻找的解决方案应该包括每根杆件上的应力大小，以及结构的整体位移。在下一章，我们将编写实际的解决方案代码；在这里，我们将定义解决方案模型。我们可以预计，解决方案模型将包括这两个量：每根杆件上的机械应力和节点位移。

然而，在我们能进行分析之前，我们需要深入研究结构分析的世界。准备好编写大量代码吧。我们将要解决一个重大的工程问题，因此我们辛苦工作的回报将是巨大的。

### **结构构件内力**

让我们快速回顾一下弹性体如何对外力的作用做出反应。这是材料力学中通常会讲授的一个主题，是机械工程课程中的经典内容。如果你已经深入学习过这个主题，可以跳过这一部分，或者快速浏览一下作为复习。如果没有，这一部分就是为你准备的。你所掌握的力学知识应该足够理解本文的内容，但我们不可能详细覆盖所有内容。你可以参考[3]，这是我个人非常喜欢的书籍之一。静力学相关的书籍也有一定程度的讲解，推荐你阅读[9]或[11]。

#### ***受外力作用的弹性体***

让我们以 I 型梁作为弹性体的例子，并对其施加一个外部平衡力系统。这些力的总和为零： ![Image](../images/f00389-p1.jpg)。[图 15-2](ch15.xhtml#ch15fig2) 显示了梁的样子。

![Image](../images/15fig02.jpg)

*图 15-2：受外力作用的梁*

当外部力作用于这个弹性体时，它的原子会反抗，以保持它们之间的相对距离。如果外部载荷试图分开这些原子，它们会尽量把彼此拉得更紧。如果它们被推在一起，它们会尽量避免靠得太近。这种“反抗”构成了*内力*：在体内存在的力，是对外力施加作用的反应。

为了研究这些力对物体的影响，我们可以以[图 15-2](ch15.xhtml#ch15fig2)中的梁为例，虚拟地用一个平面将其切割，如同[图 15-3](ch15.xhtml#ch15fig3)中所示。

![Image](../images/15fig03.jpg)

*图 15-3：受外力作用的梁的横截面*

让我们去掉梁的右侧部分，分析左侧部分的横截面发生了什么。由于整个梁在我们切割之前是静力平衡的，因此左侧部分也应当保持静力平衡。为了保持这个平衡，我们必须考虑右侧被去除的部分对左侧部分施加的内力分布。这些力的出现是因为左侧部分的原子已被从与右侧部分的邻近原子中分离开。拉紧它们的力需要加到切割的部分上，以保持原子的平衡状态不变。

这些力分布在整个切割面上，并在[图 15-4](ch15.xhtml#ch15fig4)中表示。

![Image](../images/15fig04.jpg)

*图 15-4：分析截面中的平衡*

力在某一面积上的分布被称为 *应力*。应力的净效应可以用一个等效系统来代替，其中包括一个结果力 ![Image](../images/Rvictor_caps.jpg) 和一个力矩 ![Image](../images/mvictor.jpg)。这个等效力和力矩的每个分量对梁产生不同的效果。让我们来分解这些分量。

#### ***轴向力和剪切力***

等效内力 ![Image](../images/Rvictor_caps.jpg) 可以分解为两个力的等效系统，一个垂直于截面 ![Image](../images/RNvictorit_Cap.jpg)，另一个平行于截面 ![Image](../images/Rtvictorit_Cap.jpg)（参见[图15-5](ch15.xhtml#ch15fig5)）。

![Image](../images/15fig05.jpg)

*图15-5：梁截面内的等效内力*

如果弹性体具有棱柱形状（其中一边远长于其他两边），并且我们切割与其直线方向垂直的截面，那么得到的法向力 ![Image](../images/RNvictorit_Cap.jpg) 就被称为 *轴向力*。这个名称反映了该力与棱柱的主轴或直线方向对齐。棱柱体在结构分析中很常见；梁和柱就是很好的例子。

轴向力可以拉长或压缩物体。拉伸物体的轴向力称为 *拉力*，而压缩物体的轴向力则被称为 *压力*。[图15-6](ch15.xhtml#ch15fig6)展示了受到这些力作用的两个棱柱体。

![Image](../images/15fig06.jpg)

*图15-6：拉力与压力*

*剪切力* 是作用在截面上的切向力（参见[图15-7](ch15.xhtml#ch15fig7)），因此可以进一步分解为两个分量：![Image](../images/rtxvictor_cap.jpg) 和 ![Image](../images/rtyvictor_cap.jpg)（参见[图15-5](ch15.xhtml#ch15fig5)右侧的示意图）。这两个分量具有相同的效果：它们试图剪切物体。 [图15-7](ch15.xhtml#ch15fig7)展示了剪切力作用在棱柱体上的效果。

![Image](../images/15fig07.jpg)

*图15-7：剪切力*

总结来说，物体截面的等效内力可能具有一个法向分量，该分量可以拉长或压缩物体；它还可能具有一个切向分量，该分量使物体发生剪切。这是内力对物体产生变形的两种方式。

#### ***弯曲和扭转力矩***

我们研究了由此产生的内力对给定截面的可能影响。那么，产生的力矩会有什么效果呢？如[图15-8](ch15.xhtml#ch15fig8)所示，产生的力矩 ![Image](../images/mvictor.jpg) 可以分解为垂直于截面的力矩 ![Image](../images/mnvictorit_cap.jpg) 和与截面切线方向平行的力矩 ![Image](../images/mtvictorit_cap.jpg)。

![Image](../images/15fig08.jpg)

*图15-8：梁截面内的等效内力矩*

这些力矩以任意方式弯曲物体，但如果我们选择一个棱柱体并沿着其主轴方向切割（与我们之前对力的处理方式相同），那么我们得到的力矩会有可预测且明确的效果。垂直于表面的力矩，![Image](../images/mnvictorit_cap.jpg)，会对棱柱体产生扭转（旋转）效应，因此被称为*扭转力矩*。

再次强调，垂直于截面的力矩可以进一步分解为两个子分量：![Image](../images/mtxvictorit_cap.jpg) 和 ![Image](../images/mtyvictorit_cap.jpg)（见[图 15-8](ch15.xhtml#ch15fig8)右侧插图）。这两个力矩的效果相似：它们会使棱柱体发生弯曲，因此被称为*弯曲力矩*。[图 15-9](ch15.xhtml#ch15fig9)展示了这一效果。

![Image](../images/15fig09.jpg)

*图 15-9：弯矩*

总结来说，物体横截面上的等效内部力矩可能包含一个会绕其主轴扭转的法向分量（即扭转力矩），也可能有两个切向力矩，它们倾向于弯曲棱柱体（即弯曲力矩）。

现在我们来详细分析棱柱形杆件在轴向力作用下的行为。然后，我们将看到如何利用一组这些抗力棱柱体，构建能够承受重载荷的结构。

### **拉力与压缩力**

让我们将分析重点放在轴向力上：即沿着棱柱体抗力体轴线方向的力。如我们在下一节所见，我们解决的结构仅由受轴向力作用的棱柱形元素（杆件）组成。

#### ***胡克定律***

实验已证明，在一定范围内，棱柱体杆件的伸长量与施加的轴向力成正比。这个线性关系被称为*胡克定律*。假设一根长度为*l*、截面为*A*的杆件，受一对外力作用，力的大小分别为![Image](../images/fvictorit.jpg)和–![Image](../images/fvictorit.jpg)，如[图 15-10](ch15.xhtml#ch15fig10)所示。

![Image](../images/15fig10.jpg)

*图 15-10：受轴向力作用的杆件*

[公式 15.1](ch15.xhtml#ch15eqa01)给出了胡克定律。

![Image](../images/15eqa01.jpg)

在这个公式中，

*δ*    是杆件的总伸长量。

*F*    是![Image](../images/fvictorit.jpg)力的大小。

*E*    是比例常数或杨氏模量，它是材料特有的。

胡克定律指出，受一对外力作用的杆件的总伸长量*δ*，（1）与力的大小和杆件的长度成正比，并且（2）与其横截面和杨氏模量成反比。杆件越长或施加的力越大，伸长量就越大。相反，横截面值或杨氏模量越大，伸长量则越小。

回想一下，当力分布在一个面积上时，单位面积上的力强度被称为*应力*。应力通常用希腊字母*σ*表示（参见[方程式 15.2](ch15.xhtml#ch15eqa02)）。

![图片](../images/15eqa02.jpg)

按惯例，拉伸力的应力为正，压缩力的应力为负。应力在机械设计中是一个有用的量；它用于判断给定部件（例如结构或机器中的部件）在运行中是否会断裂。给定材料在失效前能承受的应力值已被充分研究。

我们定义*应变*为单位长度的伸长，是一个无量纲的量，表示为希腊字母*ϵ*（参见[方程式 15.3](ch15.xhtml#ch15eqa03)）。

![图片](../images/15eqa03.jpg)

使用应力和应变的方程，胡克定律可以从[方程式 15.1](ch15.xhtml#ch15eqa01)重新写成[方程式 15.4](ch15.xhtml#ch15eqa04)所示的形式。

![图片](../images/15eqa04.jpg)

有趣的是，通过引入应力和应变，外部作用力（力）与其效果（伸长）之间的关系不再依赖于物体的面积或长度。我们实际上已从方程中去除了所有维度参数。[方程式 15.4](ch15.xhtml#ch15eqa04)中的比例常数（*E*）是杨氏模量，这是材料的一个特性。例如，对于结构钢，*E*约为200 GPa，即200 ⋅ 10⁹ *Pa*。因此，我们可以通过应用针对所用材料获得的实验结果来预测物体的机械行为。为此，我们使用*应力-应变图*，它绘制了给定材料的应力与应变的关系。

#### ***应力-应变图***

应力-应变图绘制了给定材料的应力与应变的关系，并通过进行拉伸或压缩试验获得（更多详情请参见[3]）。我们利用这些图表预测由相同材料制成的抗力体的行为。回想一下，自从我们引入了应力和应变这两个量后，胡克定律中的所有维度项都已经消失，这意味着一旦我们通过实验确定了材料在特定载荷下所经历的应力和应变，就可以将这些结果应用于任何由相同材料制成的物体，无论它们的形状或大小如何。

[图 15-11](ch15.xhtml#ch15fig11)是结构钢的近似应力-应变图。注意，该图并非按比例绘制。

![图片](../images/15fig11.jpg)

*图 15-11：结构钢的应力-应变图*

该图有一个初始的线性区域，能够承受一个已知的应力值，称为*比例极限*，由点A表示。对于大于比例极限的应力值，应力-应变关系不再是线性的。比例极限通常在结构钢的210 MPa到350 MPa之间—比杨氏模量小三个数量级。这个区域由胡克定律建模，线性关系为*σ* = *Eϵ*。我们将在这里集中分析。

在点A之后，即比例极限，通过一个小的应力增量，我们到达了点B，*屈服应力*或*屈服强度*。在屈服应力之后，尽管应力不再增加，材料却会发生较大的伸长。这个现象被称为材料的*屈服*。

在经历了明显的应变后，我们到达了点C，材料似乎开始硬化。应力必须继续增加，直到达到点D，这是结构钢能够承受的最大应力。我们将这个应力值称为*极限应力*或*极限强度*。从这个点开始，材料会在应力值减小的情况下承受更大的应变。

点E是材料发生断裂的地方。材料在断裂前能承受的应变量可以称为*断裂应变*。这是完全的机械失效点，但如果你仔细想想，达到极限应力（点D）后，材料很可能会发生断裂。极限应力通常作为材料在失效前能够承受的最大应力值。

现在我们对承受拉伸应力的承重体的反应有了很好的理解，让我们来看一下桁架结构。

### **平面桁架**

结构类型有很多种，但我们将重点分析其中最简单的一种：平面桁架。

*平面桁架*结构是指位于一个平面内的结构，其承重体为仅受轴向力作用的杆件，并且其自重可以忽略。使其成为平面桁架的有两个条件。

+   杆件的端部必须通过铰接销连接。

+   外部载荷必须始终作用于节点。

*节点*是多个杆件端点相交的地方。节点将杆件端部通过无摩擦的连接连接在一起，这意味着杆件绕节点的旋转不受限制。

平面桁架由三角形构成：三根杆件在其端部铰接。三角形是最简单的刚性框架；连接成四边形或更多边形的杆件形成的是非刚性框架。[图 15-12](ch15.xhtml#ch15fig12)展示了由四根杆件组成的平面桁架如何从原始位置移动，因此不被认为是刚性的。通过增加一根新的杆件并形成两个子三角形，结构变得刚性。

![图像](../images/15fig12.jpg)

*图 15-12：多边形平面桁架示例*

[图 15-13](ch15.xhtml#ch15fig13)是一个平面桁架的示例。该结构由八个节点（N1、N2、...、N8）和十三根杆件组成。节点1和节点5有外部支撑或约束。节点6、7和8承受外部载荷。

![Image](../images/15fig13.jpg)

*图 15-13：平面桁架结构*

[图 15-14](ch15.xhtml#ch15fig14)是通过对[图 15-13](ch15.xhtml#ch15fig13)中描述的平面桁架进行结构分析得到的图示。它是通过我们将在本书这一部分构建的应用程序生成的。

![Image](../images/15fig14.jpg)

*图 15-14：平面桁架结构解答图*

在这个图示中，我们可以看到结构的变形几何形状，因为它已经被缩放到足够显眼。节点的位移通常非常小（大约是结构杆件尺寸的两个数量级），因此没有缩放的节点位移图示可能难以与原始几何形状区分开来。

你会注意到在[图 15-14](ch15.xhtml#ch15fig14)中有很多信息。每根杆件都标注了它所承受的应力，尽管图中的标签字体较小，因此可能不容易阅读。正数表示拉伸应力，负数表示压缩应力。杆件也根据它们所承受的载荷以绿色或红色进行着色：绿色表示拉伸，红色表示压缩。由于本书是黑白印刷的，你无法分辨颜色，但一旦你开发完成应用程序，你将使用自己的代码生成图形，并能够探索其中的所有细节。

现在让我们研究一下组成平面桁架的杆件的机械响应。它们有一个我们之前提到的有趣特性：它们只会产生轴向应力。

### **双力构件**

如我们之前讨论的，平面桁架杆件的端部是铰接的，载荷总是施加在节点上；因此，杆件仅承受轴向力。我们只能在杆件的端部施加外力，通过铰接接头与节点的接触。由于这些接头是无摩擦的，它们只能沿杆件的主方向传递力。

[图 15-15](ch15.xhtml#ch15fig15)展示了外力如何通过节点传递到杆件上。这些力与杆件的主方向对齐，因此只会产生轴向应力。

![Image](../images/15fig15.jpg)

*图 15-15：节点中力的传递*

由于杆两端均有固定点并施加外力，因此它们受两个力的作用。为了保持平衡，身体需要这两个力共线，大小相等且方向相反。在杆（一个长的棱柱体）的情况下，这两个力必须沿着杆的主方向（见[图 15-16](ch15.xhtml#ch15fig16)）作用，因此只会产生轴向应力。我们将这些施加有两个共线力的杆称为 *双力构件*（见[图 15-16](ch15.xhtml#ch15fig16)）。

![Image](../images/15fig16.jpg)

*图 15-16：一个双力构件*

在[图 15-16](ch15.xhtml#ch15fig16)中施加在杆上的力标记为 ![Image](../images/fvictorit.jpg) 和 –![Image](../images/fvictorit.jpg)，表示这两个力的大小相等并且方向相反。在这种情况下，力会在杆上产生拉伸应力。

多亏了霍克定律，我们知道材料如何响应外部荷载的作用。我们还探讨了双力构件，并且看到平面桁架中的杆是双力构件。现在我们来推导一组方程，将这两个力与它们在这种双力构件上产生的位移关联起来。

### **全局坐标系中的刚度矩阵**

回到[方程 15.1](ch15.xhtml#ch15eqa01)中霍克定律的原始公式，我们可以将力项隔离，得到以下结果：

![Image](../images/f0397-01.jpg)

这里，术语 ![Image](../images/f00397-p1.jpg) 是杆的比例常数，它将施加的力 *F* 与其产生的伸长 *δ* 相关联。这个术语也称为 *刚度*。如你所见，刚度依赖于杆的杨氏模量 (*E*)，它是材料特性，和几何形状 (*A* 和 *l*)。

现在看看[图 15-17](ch15.xhtml#ch15fig17)中的杆。如果我们考虑一个局部坐标系，其 x 轴与杆的主方向对齐，则该杆具有两个 *自由度（DOF）*，换句话说，它有两种不同的独立运动方式。这些是两个节点在局部 x 轴上的位移，分别用 ![Image](../images/f00397-p2.jpg) 和 ![Image](../images/f00397-p3.jpg) 表示。每个节点都有一个施加的力：*F*[1] 和 *F*[2]。

**注意**

*命名法说明：我们将使用撇号来标记杆局部坐标系中参考的自由度（DOF）。例如，*![Image](../images/ui1victor.jpg)* 指的是节点 1 在杆局部坐标系中的 x 位移：(x^′,y^′)*。相比之下，没有撇号的值，如* u[1]*，是指全球坐标系的参考：(x,y)。*

![Image](../images/15fig17.jpg)

*图 15-17：具有两个自由度的杆*

使用之前的方程，我们可以将每个节点的力与位移 ![Image](../images/f00397-p2.jpg) 和 ![Image](../images/f00397-p3.jpg) 关联起来，如下所示：

![Image](../images/f0398-01.jpg)

上述两个方程可以用矩阵表示法写出 ([方程 15.5](ch15.xhtml#ch15eqa05))，

![Image](../images/15eqa05.jpg)

其中，[*k*^′] 被称为杆件的局部*刚度矩阵*。这个刚度矩阵将杆件两节点的位移与施加在它们上的外力联系起来，所有内容都在杆件的局部参考系中。使用这个局部参考系，杆件只有两个自由度，即每个节点在局部 x 轴方向上的位移 (![Image](../images/f00397-p2.jpg) 和 ![Image](../images/f00397-p3.jpg))。

现在我们来考虑一个相对于全局坐标系旋转的杆件。以 [图 15-18](ch15.xhtml#ch15fig18) 为例。该杆件有自己的局部参考系 (*x*^′, *y*^′)，它与全局参考系 (*x, y*) 形成一个角度 *θ*。

![Image](../images/15fig18.jpg)

*图 15-18：杆件的局部参考系*

从全局参考系的角度来看，杆件的每个节点有两个自由度：每个节点可以在 x 和 y 方向上移动。在这个参考系下，四个自由度分别是 *u*[1]、*v*[1]、*u*[2] 和 *v*[2]。

为了将杆件的局部刚度矩阵 [*k*^′] 转换为全局的 [*k*] 刚度矩阵，我们必须应用一个变换矩阵。我们可以通过将局部位移 ![Image](../images/f00397-p2.jpg) 和 ![Image](../images/f00397-p3.jpg) 分解为它们的全局分量来找到这样一个矩阵。 [图 15-19](ch15.xhtml#ch15fig19) 展示了这个操作。

![Image](../images/15fig19.jpg)

*图 15-19：局部位移投影*

现在我们来找一个数学表达式，通过它来计算基于局部位移的全局位移：

![Image](../images/f0399-01.jpg)

以矩阵形式写出，它的形式如下：

![Image](../images/f0399-02.jpg)

其中，[*L*] 是变换矩阵。为了从局部的 [*k*^′] 计算全局刚度矩阵，我们可以使用以下方程（有关如何推导这个表达式的详细信息，请参见 [2] 或 [10]）。

[*k*] = [*L*]^′[*k*]^′[*L*]

通过简化符号为 *c* = cos*θ* 和 *s* = sin*θ*，得到 [方程 15.6](ch15.xhtml#ch15eqa06)。

![Image](../images/15eqa06.jpg)

现在我们有一个方程组，它将施加在杆件节点上的外力与它们在全局坐标系中的位移联系起来（参见 [方程 15.7](ch15.xhtml#ch15eqa07)）。

![Image](../images/15eqa07.jpg)

现在我们利用这些知识开始在代码中构建我们的结构模型。

### **原始结构模型**

在我们的*力学*项目中，创建一个新的 Python 包，命名为*structures*。在*structures*中，再创建一个包：*model*。在这里我们将定义构成结构模型的类。再在*structures*中创建一个名为*solution*的包。在这里，我们将定义模拟已解析结构的类。还需要在*structures*中创建一个*tests*文件夹，包含我们将要开发的单元测试。你项目的结构应当是这样的：

力学

|- 应用

|- eqs

|- geom2d

|- graphic

|- 结构

|    |- model

|    |   |- __init__.py

|    |- solution

|    |   |- __init__.py

|    |- tests

|    |   |- __init__.py

|    |- __init__.py

|- 工具

下一步是创建一个表示结构节点的类。

#### ***节点类***

在*model*中创建一个新文件，命名为*node.py*，并输入[列表 15-1](ch15.xhtml#ch15lis1)中的代码。这是结构节点的基本定义。

```py
import operator
from functools import reduce

from geom2d import Point, Vector

class StrNode:

    def __init__(
        self,
     ➊ _id: int,
        position: Point,
        loads=None,
        dx_constrained=False,
        dy_constrained=False
    ):
        self.id = _id
        self.position = position
     ➋ self.loads = loads or []
        self.dx_constrained = dx_constrained
        self.dy_constrained = dy_constrained

    @property
    def loads_count(self):
        return len(self.loads)

    @property
    def net_load(self):
     ➌ return reduce(
            operator.add,
            self.loads,
            Vector(0, 0)
        )
```

*列表 15-1：结构节点类*

在这个列表中，我们定义了新的类 StrNode。这个类定义了一个 id，用来标识它的每个实例。

请注意，传递给构造函数的参数使用了下划线：_id ➊。Python 已经定义了一个名为 id 的全局函数，因此如果我们将参数命名为 id（而不是使用下划线），我们就会在构造函数内覆盖这个全局 id 函数的定义。这意味着 id 在构造函数中不再指代 Python 的函数，而是指代我们传入的值。虽然我们在这个类的构造函数中并没有使用 Python 的 id 函数，但我们会尽量避免覆盖全局函数。

StrNode 还包含一个 Point 类的实例，用来确定节点的位置，并且包含一个默认值为 None 的加载列表。结构可能有很多节点没有外部加载，因此我们将加载参数设为可选（并提供默认值 None）。当加载参数为 None 时，我们将 self.loads 属性赋为空列表（[]） ➋。

你可能会想知道或运算符在 ➋ 中是如何工作的：

```py
self.loads = loads or []
```

或运算符返回其操作数中第一个“真实”的值或 None。看以下示例：

```py
>>> 'Hello' or 'Good bye'
'Hello'

>>> None or 'Good bye'
'Good bye'

>>> False or True
True

>>> False or 'Hello'
'Hello'

>>> False or None
# nothing returned here

>>> False or None or 'Hi'
'Hi'
```

正如你可能已经猜到的，在布尔上下文中，None 被评估为“假”。

还有两个属性我们需要传递给构造函数，它们在构造函数中有默认值：dx_constrained 和 dy_constrained。这些属性决定了在 x 和 y 方向上的位移是否受到外部约束。我们将它们初始化为 False，这意味着节点不会受到外部约束，除非我们明确指出。

我们在类中定义了两个属性：loads_count 和 net_load。第一个属性 loads_count 只是返回 loads 列表的长度。

**注意**

*如果你记得[第五章](ch05.xhtml#ch05)中的德梅特法则，任何想要知道施加在节点上的载荷数量的外部人员，都应该能直接向* StrNode *类询问。但如果让* StrNode *返回载荷列表，再使用* len *函数来获取其长度，就会违反这一重要原则。*

net_load 属性使用 reduce 来计算所有载荷的总和 ➌。注意，我们向 reduce 函数传递了第三个参数：Vector(0, 0)。这个第三个参数是归约的初始值。在载荷列表为空的完全有效的情况下，我们将返回这个初始值。否则，归约过程的第一步将把这个初始值与列表的第一个元素结合起来。如果没有提供初始值，减少载荷列表将引发以下错误：

```py
TypeError: reduce() of empty sequence with no initial value
```

接下来，我们将添加一个方法来向节点的载荷列表中添加载荷；在[列表 15-2](ch15.xhtml#ch15lis2)中输入该方法。

```py
class StrNode:
   --snip--

   def add_load(self, load: Vector):
       self.loads.append(load)
```

*列表 15-2：向节点添加载荷*

最后，我们来实现 StrNode 类的相等性比较。类中有一些属性，但我们认为只有当两个节点在平面上的位置相同时，它们才是相等的。这个比较将认为重叠的节点是相等的，而不管它们的其他属性。

如果我们希望结构中的节点是真正唯一的，可以依赖于一个相等性比较，比较节点的所有属性，包括载荷列表和外部约束。在我们的情况下，我们只关心确保没有重叠的节点。如果我们在相等性检查中包括更多的字段，可能会发生两个重叠的节点（具有相同位置）被评估为不同的，因为它们有不同的载荷列表。这样我们就会允许两个重叠的节点在结构中共存。

在[列表 15-3](ch15.xhtml#ch15lis3)中输入 __eq__ 方法的实现。

```py
class StrNode:
   --snip--

    def __eq__(self, other):
        if self is other:
            return True

        if not isinstance(other, StrNode):
            return False

        return self.position == other.position
```

*列表 15-3：节点相等性*

我们的 StrNode 类现在准备好了！[列表 15-4](ch15.xhtml#ch15lis4)包含了最终的 StrNode 类。

```py
import operator
from functools import reduce

from geom2d import Point, Vector

class StrNode:

    def __init__(
            self,
            _id: int,
            position: Point,
            loads=None,
            dx_constrained=False,
            dy_constrained=False
    ):
        self.id = _id
        self.position = position
        self.loads = loads or []
        self.dx_constrained = dx_constrained
        self.dy_constrained = dy_constrained

    @property
    def loads_count(self):
        return len(self.loads)

    @property
    def net_load(self):
        return reduce(
            operator.add,
            self.loads,
            Vector(0, 0)
        )

    def add_load(self, load: Vector):
        self.loads.append(load)

    def __eq__(self, other):
        if self is other:
            return True

        if not isinstance(other, StrNode):
            return False

        return self.position == other.position
```

*列表 15-4：节点类结果*

现在，我们将实现一个类来表示结构杆件。

#### ***杆件类***

结构杆件是由两个由 StrNode 类建模的节点之间定义的。杆件需要存储计算刚度矩阵所需的两个抗性属性的值（[方程 15.6](ch15.xhtml#ch15eqa06)）：杨氏模量和截面。

##### **实现杆件类**

在 *model* 中创建一个名为 *bar.py* 的新文件，并输入 StrBar 类的初始定义（[列表 15-5](ch15.xhtml#ch15lis5)）。

```py
from geom2d import Segment
from .node import StrNode

class StrBar:

    def __init__(
            self,
            _id: int,
            start_node: StrNode,
            end_node: StrNode,
            cross_section: float,
            young_mod: float
    ):
        self.id = _id
        self.start_node = start_node
        self.end_node = end_node
        self.cross_section = cross_section
        self.young_mod = young_mod

    @property
    def geometry(self):
        return Segment(
            self.start_node.position,
            self.end_node.position
        )

    @property
    def length(self):
        return self.geometry.length
```

*列表 15-5：结构杆件类*

在此列表中，我们定义了 StrBar 类，包含五个属性：作为标识符的 ID，起始节点和终止节点，截面值，以及杨氏模量值。这些值传递给构造函数并存储在类中。

我们还使用 @property 装饰器定义了两个属性：geometry 和 length。杆件的几何形状是一个从起始节点位置到结束节点位置的线段，杆件的长度就是这个线段的长度。

我们需要实现的最后一件事是一个方法，用来计算杆件在全局坐标系中的刚度矩阵，正如[方程 15.6](ch15.xhtml#ch15eqa06)所定义的那样。在[清单 15-6](ch15.xhtml#ch15lis6)中输入该方法。

```py
from eqs import Matrix
from geom2d import Segment
from .node import StrNode

class StrBar:
    --snip--

    def global_stiffness_matrix(self) -> Matrix:
        direction = self.geometry.direction_vector
        eal = self.young_mod * self.cross_section / self.length
        c = direction.cosine
        s = direction.sine

        c2_eal = (c ** 2) * eal
        s2_eal = (s ** 2) * eal
        sc_eal = (s * c) * eal

        return Matrix(4, 4).set_data([
            c2_eal, sc_eal, -c2_eal, -sc_eal,
            sc_eal, s2_eal, -sc_eal, -s2_eal,
            -c2_eal, -sc_eal, c2_eal, sc_eal,
            -sc_eal, -s2_eal, sc_eal, s2_eal
        ])
```

*清单 15-6：全局坐标系中的杆件刚度矩阵*

别忘了导入 Matrix，如下所示：

```py
from eqs import Matrix
```

我们添加了 global_stiffness_matrix 方法。该方法创建一个 4 × 4 的矩阵，并将其值设置为适当的刚度项，正如[方程 15.6](ch15.xhtml#ch15eqa06)中所给出的，并为方便起见在此重复：

![Image](../images/f0406-01.jpg)

为了计算每个值，我们首先获取杆件几何形状的方向向量，并计算它的正弦和余弦。由于 [*k*] 中的每一项都乘以 ![Image](../images/f00397-p1.jpg)，我们计算它并将结果存储在 eal 变量中。矩阵中的十六个项实际上只有三个不同的值需要计算。这些值分别存储在 c2_eal、s2_eal 和 sc_eal 中，后续会在 set_data 方法中引用。

##### **测试 Bar 类**

刚度矩阵的计算是我们结构分析问题的核心；如果这段代码有 bug，会导致完全错误的结果，比如，杆件出现巨大的变形。让我们添加一个单元测试，确保刚度矩阵中的所有项都能正确计算。我们首先需要在 *structures/tests* 目录中创建一个新的测试文件，命名为 *bar_test.py*。在文件中输入[清单 15-7](ch15.xhtml#ch15lis7)中的代码。

```py
import unittest
from math import sqrt

from eqs import Matrix
from geom2d import Point
from structures.model.node import StrNode
from structures.model.bar import StrBar

class BarTest(unittest.TestCase):
    section = sqrt(5)
    young = 5

    node_a = StrNode(1, Point(0, 0))
    node_b = StrNode(2, Point(2, 1))
    bar = StrBar(1, node_a, node_b, section, young)

    def test_global_stiffness_matrix(self):
        expected = Matrix(4, 4).set_data([
            4, 2, -4, -2,
            2, 1, -2, -1,
            -4, -2, 4, 2,
            -2, -1, 2, 1
        ])
        actual = self.bar.global_stiffness_matrix()
        self.assertEqual(expected, actual)
```

*清单 15-7：测试杆件的刚度矩阵*

在这个测试中，我们创建了一根节点位于 (0, 0) 和 (2, 1) 之间的杆件，截面如 ![Image](../images/f00407-p1.jpg)，杨氏模量为 5。我们选择这些数字是为了让期望的刚度矩阵中的所有值都是整数，这样可以方便我们编写断言，特别是在这个情况下：![Image](../images/f00407-p2.jpg)，![Image](../images/f00407-p3.jpg)，和 ![Image](../images/f00407-p4.jpg)。

你可以通过点击绿色播放按钮或从终端运行测试。

```py
$ python3 -m unittest structures/tests/bar_test.py
```

这应该会产生以下输出：

```py
Ran 1 test in 0.000s

OK
```

你的 StrBar 类应该与[清单 15-8](ch15.xhtml#ch15lis8)类似。

```py
from eqs import Matrix
from geom2d import Segment
from .node import StrNode

class StrBar:

    def __init__(
            self,
            _id: int,
            start_node: StrNode,
            end_node: StrNode,
            cross_section: float,
            young_mod: float
    ):
        self.id = _id
        self.start_node = start_node
        self.end_node = end_node
        self.cross_section = cross_section
        self.young_mod = young_mod

    @property
    def geometry(self):
        return Segment(
            self.start_node.position,
            self.end_node.position
        )

    @property
    def length(self):
        return self.geometry.length

    def global_stiffness_matrix(self) -> Matrix:
        direction = self.geometry.direction_vector
        eal = self.young_mod * self.cross_section / self.length
        c = direction.cosine
        s = direction.sine

        c2_eal = (c ** 2) * eal
        s2_eal = (s ** 2) * eal
        sc_eal = (s * c) * eal

        return Matrix(4, 4).set_data([
            c2_eal, sc_eal, -c2_eal, -sc_eal,
            sc_eal, s2_eal, -sc_eal, -s2_eal,
            -c2_eal, -sc_eal, c2_eal, sc_eal,
            -sc_eal, -s2_eal, sc_eal, s2_eal
        ])
```

*清单 15-8：Bar 类结果*

我们需要最后一个类来将节点和杆件组合在一起：结构本身。

#### ***Structure 类***

在 *structures/model* 中创建一个新的 Python 文件，命名为 *structure.py*，并输入 Structure 类的代码([清单 15-9](ch15.xhtml#ch15lis9))。

```py
from functools import reduce

from .node import StrNode
from .bar import StrBar

class Structure:
    def __init__(self, nodes: [StrNode], bars: [StrBar]):
        self.__bars = bars
        self.__nodes = nodes

    @property
    def nodes_count(self):
        return len(self.__nodes)

    @property
    def bars_count(self):
        return len(self.__bars)

    @property
    def loads_count(self):
        return reduce(
            lambda count, node: count + node.loads_count,
            self.__nodes,
            0
        )
```

*清单 15-9：Structure 类*

这个类目前非常简单，但在后续章节中，我们将编写代码来组装结构的全局刚度矩阵，生成方程组，求解它，并创建解。现在，这个类仅存储传递给构造函数的节点列表和杆件列表，以及一些处理其包含项数量的计算。

loads_count 属性会计算每个节点的荷载总数。为了实现这一点，我们将一个 lambda 函数作为第一个参数传递给 reduce 函数。这个 lambda 函数有两个参数：当前的荷载计数和 self.__nodes 列表中的下一个节点。这个归约操作需要一个初始值（即第三个参数 0），我们将第一个节点的计数值加到它上面。如果没有这个初始值，归约操作将无法进行，因为 reduce 函数不知道 lambda 的第一个参数 count 在第一次迭代时的值。

现在我们已经有了完整的模型来定义结构！

#### ***从 Python Shell 创建结构***

让我们尝试使用我们的模型类构建[图 15-20](ch15.xhtml#ch15fig20)中的桁架结构。

![图像](../images/15fig20.jpg)

*图 15-20：示例桁架结构*

要定义结构，首先在 Python shell 中导入以下类：

```py
>>> from geom2d import Point, Vector
>>> from structures.model.node import StrNode
>>> from structures.model.bar import StrBar
>>> from structures.model.structure import Structure
```

然后输入以下代码：

```py
>>> node_one = StrNode(1, Point(0, 0), None, True, True)
>>> node_two = StrNode(2, Point(100, 0), None, False, True)
>>> node_three = StrNode(3, Point(100, 100), (Vector(50, -100)))

>>> bar_one = (1, node_one, node_two, 20, 20000000)
>>> bar_two = (2, node_two, node_three, 20, 20000000)
>>> bar_three = (3, node_three, node_one, 20, 20000000)

>>> structure = Structure(
    (node_one, node_two, node_three),
    (bar_one, bar_two, bar_three)
)
```

如你所见，在代码中创建桁架结构的模型非常简单。无论如何，我们最常见的做法是从外部定义文件加载模型，正如我们将在[第 17 章](ch17.xhtml#ch17)中学到的那样。然而，手动操作一个例子是理解我们模型类如何工作的一个很好的练习。

为了完成这一章，让我们创建结构解算模型：存储节点位移和杆件应力的类。

### **结构解算模型**

我们将在下一章解决结构的解算问题，但在这里我们会准备类来存储解算值。现在，假设我们已经准备好了求解算法，需要使用解算类来存储解算的数据。

当我们解算一个结构时，首先获得节点在全局坐标系中的位移。通过结构节点的新位置，我们可以计算出其余的所有数据（如应变、应力和反作用力值）。我们需要一个新类来表示位移节点，这些节点类似于我们刚刚使用 StrNode 类定义的节点，只是多了一个位移向量。

这些节点位移会使结构的杆件发生拉伸或压缩。记住，杆件会产生应变和应力，这是它们对拉伸或压缩的机械反应。应变和应力值是结构解算中的重要数据，它们将决定结构是否能够承受施加的荷载。

我们还将创建一个新类来表示解算的杆件。这个类将引用位移后的节点，并计算应变和应力值。

#### ***解法节点***

让我们创建一个表示结构解法中节点的类。在 *structures/solution* 包中，创建一个名为 *node.py* 的新文件，并输入[列表 15-10](ch15.xhtml#ch15lis10)中的代码。

```py
from geom2d import Vector
from structures.model.node import StrNode

class StrNodeSolution:
    def __init__(
            self,
            original_node: StrNode,
            global_disp: Vector
    ):
        self.__original_node = original_node
        self.global_disp = global_disp

    @property
 ➊ def id(self):
        return self.__original_node.id

    @property
 ➋ def original_pos(self):
        return self.__original_node.position

    @property
 ➌ def is_constrained(self):
        return self.__original_node.dx_constrained \
               or self.__original_node.dy_constrained

    @property
 ➍ def loads(self):
        return self.__original_node.loads

    @property
 ➎ def is_loaded(self):
       return self.__original_node.loads_count > 0

    @property
 ➏ def net_load(self):
        return self.__original_node.net_load
```

*列表 15-10：解法节点类*

这个代码段声明了 StrNodeSolution 类。如你所见，该类的构造函数接受原始节点及其在全局坐标系中的位移向量——这就是我们所需要的。原始节点被设置为类的私有属性 (__original_node)，但它的一些属性被暴露出来。例如，id 属性 ➊ 仅返回原始节点的 ID，加载（loads）也是如此。

original_pos 属性 ➋ 返回原始节点的位置：即在应用作为结构解法一部分获得的位移之前的位置。这里的命名很重要，因为我们稍后将添加另一个属性，用于暴露节点在被位移后的新位置。

is_constrained 属性 ➌ 检查原始节点是否有任何自由度（在 x 或 y 方向上的位移）被外部约束。我们将使用此信息来确定是否需要为节点计算反作用力。*反作用力*是支撑或约束在节点上施加的外力。我们需要知道支撑所承受的力的大小，以便正确设计和确定支撑的尺寸。

最后，我们有三个与外部载荷相关的属性：loads ➍、is_loaded ➎ 和 net_load ➏。第一个属性仅返回原始节点的力的列表。我们将在绘制像[图 15-14](ch15.xhtml#ch15fig14)那样的向量图时使用此信息。属性 is_loaded 让我们知道节点是否施加了任何载荷。这个属性在我们需要检查哪些解法节点上施加了载荷并将这些载荷绘制到结果图中时非常有用。属性 net_load 返回原始节点的净载荷，我们将使用它来计算节点的反作用力。

##### **位移位置**

让我们将位移位置作为一个属性。由于位移通常比结构的尺寸小几个数量级，我们希望包含一个方法来缩放位移向量，以便绘制出变形后的几何图形。这确保了我们能够在结果图中区分变形后的几何图形与原始几何图形。

将[列表 15-11](ch15.xhtml#ch15lis11)中的代码输入到 StrNodeSolution 类中。

```py
class StrNodeSolution:
   --snip--

   @property
   def displaced_pos(self):
       return self.original_pos.displaced(self.global_disp)

   def displaced_pos_scaled(self, scale=1):
       return self.original_pos.displaced(self.global_disp, scale)
```

*列表 15-11：解法节点位移*

displaced_pos 方法返回在应用全局位移向量后，原始节点的位置。displaced_pos_scaled 方法做类似的事情，但带有一个缩放值，允许我们增加位移的大小。

##### **最终结果**

如果你按照步骤操作，你的 StrNodeSolution 类应该与[列表 15-12](ch15.xhtml#ch15lis12)中显示的一样。

```py
from geom2d import Vector
from structures.model.node import StrNode

class StrNodeSolution:
    def __init__(
            self,
            original_node: StrNode,
            global_disp: Vector
    ):
        self.__original_node = original_node
        self.global_disp = global_disp

    @property
    def id(self):
        return self.__original_node.id

    @property
    def original_pos(self):
        return self.__original_node.position

    @property
    def is_constrained(self):
        return self.__original_node.dx_constrained \
               or self.__original_node.dy_constrained 

    @property
    def loads(self):
        return self.__original_node.loads

    @property
    def is_loaded(self):
        return self.__original_node.loads_count > 0

    @property
    def displaced_pos(self):
        return self.original_pos.displaced(self.global_disp)

    def displaced_position_scaled(self, scale=1):
        return self.original_pos.displaced(self.global_disp, scale)
```

*列表 15-12：解法节点类结果*

现在让我们实现杆的解类。

#### ***解杆***

知道杆的节点位移是我们计算其应变和轴向应力所需的全部。我们将在开发 `StrBarSolution` 类时解释为什么如此。

在 *structures/solution* 目录下创建一个新文件，命名为 *bar.py*，并输入 [Listing 15-13](ch15.xhtml#ch15lis13) 中的代码。

```py
from structures.model.bar import StrBar
from .node import StrNodeSolution

class StrBarSolution:
    def __init__(
            self,
            original_bar: StrBar,
            start_node: StrNodeSolution,
            end_node: StrNodeSolution
    ):
        if original_bar.start_node.id != start_node.id:
            raise ValueError('Wrong start node')

        if original_bar.end_node.id != end_node.id:
            raise ValueError('Wrong end node')

        self.__original_bar = original_bar
        self.start_node = start_node
        self.end_node = end_node

    @property
    def id(self):
        return self.__original_bar.id

    @property
    def cross_section(self):
        return self.__original_bar.cross_section

    @property
    def young_mod(self):
        return self.__original_bar.young_mod
```

*Listing 15-13: 解杆类*

`StrBarSolution` 类使用原始杆和两个解节点进行初始化。在构造函数中，我们通过将解节点的 ID 与原始杆节点的 ID 进行比较，检查是否传入了正确的解节点。如果检测到传入了错误的节点，我们将引发 `ValueError`，并终止程序执行。如果继续执行，结果将是错误的，因为解杆将与原始定义中未连接的节点相连。这将防止我们在构建结构的解类时犯错误。

该类还定义了 `id`、`cross_section` 和 `young_mod` 属性。这些属性只是返回原始杆的值。

##### **伸长、应力与应变**

现在让我们一步一步地计算应变和应力。应力可以通过应变推导出来（使用 [Equation 15.4](ch15.xhtml#ch15eqa04)），所以我们从应变开始。应变是杆的每单位长度的伸长（见 [Equation 15.3](ch15.xhtml#ch15eqa03)），因此我们需要找出这个伸长值。为此，我们首先要了解杆的原始几何形状和结果几何形状。输入 [Listing 15-14](ch15.xhtml#ch15lis14) 中显示的属性。

```py
from geom2d import Segment
from structures.model.bar import StrBar
from .node import StrNodeSolution

class StrBarSolution:
   --snip--

   @property
   def original_geometry(self):
       return self.__original_bar.geometry

   @property
   def final_geometry(self):
       return Segment(
           self.start_node.displaced_pos,
           self.end_node.displaced_pos
       )
```

*Listing 15-14: 解杆几何形状*

原始几何形状已经是 `StrBar` 类中的一个属性。最终几何形状也是一个线段，这次是位移后的起始节点和结束节点之间的线段。需要理解的是，由于桁架结构的杆是二力构件，它们只受轴向力的作用。因此，杆的导线将始终保持为一条直线。[Figure 15-21](ch15.xhtml#ch15fig21) 描绘了原始杆和由于位移原始节点位置而产生的变形杆 ![Image](../images/u1victorit.jpg) 和 ![Image](../images/u2victorit.jpg)。

![Image](../images/15fig21.jpg)

*Figure 15-21: 杆的长度增量*

假设原始杆的长度为 *l*[*o*]，*l*[*f*] 是最终长度，则杆的伸长简单地为 *Δl* = *l*[*f*] – *l*[*o*]。如果杆被拉伸，则伸长值为正；如果杆被压缩，则为负。请注意，这与我们的应力符号约定一致：拉伸为正，压缩为负。输入 [Listing 15-15](ch15.xhtml#ch15lis15) 中的属性。

```py
class StrBarSolution:
   --snip--

   @property
   def original_length(self):
       return self.original_geometry.length

   @property
   def final_length(self):
       return self.final_geometry.length

   @property
   def elongation(self):
       return self.final_length - self.original_length
```

*Listing 15-15: 解杆长度*

现在我们知道了杆的伸长，可以轻松地计算应变和应力。在 `StrBarSolution` 类中输入应变和应力属性，如 [Listing 15-16](ch15.xhtml#ch15lis16) 中所示。

```py
class StrBarSolution:
   --snip--

   @property
   def strain(self):
       return self.elongation / self.original_length

   @property
   def stress(self):
       return self.young_mod * self.strain
```

*列表 15-16：梁的应变与应力*

最后！如你所见，所给的应变由[方程 15.3](ch15.xhtml#ch15eqa03)给出，它是梁的伸长与原始长度的商。通过应变值，我们可以通过与材料的杨氏模量简单相乘来获得应力。这就是[方程 15.4](ch15.xhtml#ch15eqa04)中表述的胡克定律。

##### **内力**

为了计算反作用力，我们将使用每个节点的静力平衡条件：一个节点的合力始终为零。在这个力的总和中，连接到该节点的每根梁都施加一个大小相等、方向相反的力（如[图 15-23](ch15.xhtml#ch15fig23)所示）。这个内力是根据梁的应力乘以其截面计算得出的（见[方程 15.2](ch15.xhtml#ch15eqa02)）。

我们需要梁每个节点内力的大小和方向，因为如果你还记得，为了使这个二力杆件处于平衡状态，两端的力需要大小相等且方向相反。让我们看看如何实现这一点。

输入代码：[列表 15-17](ch15.xhtml#ch15lis17)。

```py
from geom2d import Segment, make_vector_between
from structures.model.bar import StrBar
from .node import StrNodeSolution

class StrBarSolution:
    --snip--

    @property
    def internal_force_value(self):
        return self.stress * self.cross_section

    def force_in_node(self, node: StrNodeSolution):
     ➊ if node is self.start_node:
            return make_vector_between(
                self.end_node.displaced_pos,
                self.start_node.displaced_pos
            ).with_length(
                self.internal_force_value
            )
     ➋ elif node is self.end_node:
            return make_vector_between(
                self.start_node.displaced_pos,
                self.end_node.displaced_pos
            ).with_length(
                self.internal_force_value
            )

        raise ValueError(
            f'Bar {self.id} does not know about node {node.id}'
        )
```

*列表 15-17：梁的内力*

在这段代码中，我们首先定义了 internal_force_value 属性，该属性表示根据[方程 15.2](ch15.xhtml#ch15eqa02)计算的内力的大小（正值或负值）。

接下来是 force_in_node 方法，它根据梁的起始或结束节点，返回该节点的力向量。无论是哪种情况，力向量的大小都是 internal_force_value。只有方向会根据传入的节点发生变化。

我们的符号约定是将拉力视为正值，压缩力视为负值。如果我们选择每个节点的内力方向为正，则力向量始终会朝着正确的方向。这是因为稍后我们会为其分配一个内力值，这个值对于压缩力来说是负的，正如你所知道的那样，将负值分配给我们的 Vector 实例之一会反转其方向。

回顾一下代码。如果传入的节点是起始节点 ➊，则力向量从结束节点的最终位置指向起始节点的位置。然后，结果向量会根据 internal_force_value 进行缩放。

相反，如果传入的节点是结束节点 ➋，力向量是相反的，但缩放部分保持不变。

最后，如果传入的节点不是这两个梁的节点之一，我们将引发错误。

##### **梁有节点吗？**

我们快完成梁解决方案类了；只需要再添加两个方法，我们的类就准备好了。第一个方法检查结构中的任何节点是否为梁的端节点之一。我们将使用此方法来绘制结果。输入方法：[列表 15-18](ch15.xhtml#ch15lis18)。

```py
class StrBarSolution:
   --snip--

   def has_node(self, node: StrNodeSolution):
       return node is self.start_node or node is self.end_node
```

*列表 15-18：梁有节点吗？*

最后，我们需要一个方法来生成条形的最终几何图形，但要对位移应用缩放。

##### **比例最终几何图形**

如果你还记得，我们已经在StrNodeSolution类中实现了一个方法，它返回带有位移缩放的节点位置。让我们利用这个实现来构建表示变形条形几何图形的段，并对其应用缩放。输入[列表 15-19](ch15.xhtml#ch15lis19)中的代码。

```py
class StrBarSolution:
   --snip--

   def final_geometry_scaling_displacement(self, scale: float):
       return Segment(
           self.start_node.displaced_pos_scaled(scale),
           self.end_node.displaced_pos_scaled(scale)
       )
```

*列表 15-19: 条形比例几何图形*

final_geometry_scaling_displacement方法返回一个段，其端点是条形节点在应用位移缩放后的位置。我们将在结果图上绘制这个段，以可视化原始条形如何从其原始位置发生位移。

同样，因为与结构本身的大小相比，位移相对较小，所以我们希望对节点位移进行缩放，以便清楚地看到结构在解图中如何变形。

##### **最终结果**

如果你跟着操作，那么你的StrBarSolution应该像[列表 15-20](ch15.xhtml#ch15lis20)一样。

```py
from geom2d import Segment, make_vector_between
from structures.model.bar import StrBar
from .node import StrNodeSolution

class StrBarSolution:
    def __init__(
            self,
            original_bar: StrBar,
            start_node: StrNodeSolution,
            end_node: StrNodeSolution
    ):
        if original_bar.start_node.id != start_node.id:
            raise ValueError('Wrong start node')

        if original_bar.end_node.id != end_node.id:
            raise ValueError('Wrong end node')

        self.__original_bar = original_bar
        self.start_node = start_node
        self.end_node = end_node

    @property
    def id(self):
        return self.__original_bar.id

    @property
    def cross_section(self):
        return self.__original_bar.cross_section

    @property
    def young_mod(self):
        return self.__original_bar.young_mod

    @property
    def original_geometry(self):
        return self.__original_bar.geometry

    @property
    def final_geometry(self):
        return Segment(
            self.start_node.displaced_pos,
            self.end_node.displaced_pos
        )

    @property
    def original_length(self):
        return self.original_geometry.length

    @property
    def final_length(self):
        return self.final_geometry.length

    @property
    def elongation(self):
        return self.final_length - self.original_length

    @property
    def strain(self):
        return self.elongation / self.original_length

    @property
    def stress(self):
        return self.young_mod * self.strain

    @property
    def internal_force_value(self):
        return self.stress * self.cross_section

    def force_in_node(self, node: StrNodeSolution):
        if node is self.start_node:
            return make_vector_between(
                self.end_node.displaced_pos,
                self.start_node.displaced_pos
            ).with_length(
                self.internal_force_value
            )
        elif node is self.end_node:
            return make_vector_between(
                self.start_node.displaced_pos,
                self.end_node.displaced_pos
            ).with_length(
                self.internal_force_value
            )

        raise ValueError(
            f'Bar {self.id} does not know about node {node.id}'
        )

    def has_node(self, node: StrNodeSolution):
        return node is self.start_node or node is self.end_node

    def final_geometry_scaling_displacement(self, scale: float):
        return Segment(
            self.start_node.displaced_position_scaled(scale),
            self.end_node.displaced_position_scaled(scale)
        )
```

*列表 15-20: 解条形类结果*

还有一个最后需要定义的类：结构解。

#### ***结构解***

就像我们为原始结构模型定义了一个类一样，我们也想为结构的解定义一个类。这个类的目的是将解的节点和条形组合在一起。

在*structures/solution*文件夹中创建一个名为*structure.py*的新文件。在文件中输入类的基本定义（[列表 15-21](ch15.xhtml#ch15lis21)）。

```py
from .bar import StrBarSolution
from .node import StrNodeSolution

class StructureSolution:
    def __init__(
            self,
            nodes: [StrNodeSolution],
            bars: [StrBarSolution]
    ):
        self.nodes = nodes
        self.bars = bars
```

*列表 15-21: 结构解类*

StructureSolution类通过节点和条形的列表初始化，这些节点和条形构成了解。这与原始结构的定义类似。但由于我们使用这个类来生成结果——报告和图表——所以我们需要一些额外的属性。

##### **结构矩形边界**

在绘制结构分析结果时，我们需要知道绘制完整结构所需的空间。了解整个结构的矩形边界将帮助我们稍后计算SVG图形的viewBox。让我们先计算这些边界并添加一些边距（参见[图 15-22](ch15.xhtml#ch15fig22)），这样可以为绘制如表示荷载的箭头等元素留出额外的空间。

![Image](../images/15fig22.jpg)

*图 15-22: 结构的边界*

在类中，输入bounds_rect方法（[列表 15-22](ch15.xhtml#ch15lis22)）。

```py
from geom2d import make_rect_containing_with_margin
from .bar import StrBarSolution
from .node import StrNodeSolution

class StructureSolution:
   --snip--

    def bounds_rect(self, margin: float, scale=1):
        d_pos = [
            node.displaced_pos_scaled(scale)
            for node in self.nodes
        ]
        return make_rect_containing_with_margin(d_pos, margin)
```

*列表 15-22: 结构图形边界*

我们首先导入make_rect_containing_with_margin函数。我们在本书的[第二部分](part02.xhtml#part02)中实现了这个函数，它创建一个包含所有传入点的矩形原始图形，并带有一些边距。

我们编写的bounds_rect方法初始化了d_pos变量，作为一个包含所有结构节点位移位置的列表，并将其传递给函数，后者生成矩形。请注意，我们使用的是位移的缩放版本，以确保矩形边界包含所有将被绘制的节点位置。

##### **节点反应力**

最后，由于StructureSolution类能够访问结构的所有节点和杆件，它将负责计算每个节点的反应力。StrNodeSolution类无法自行进行此计算，因为它无法访问与该节点相交的杆件列表。

那么，我们如何计算一个节点的反应力呢？假设我们有一个像[图15-23](ch15.xhtml#ch15fig23)中的节点。两个杆件，杆件1和杆件2，在此节点相交，并且分别受内部力 ![Image](../images/f1victorit.jpg) 和 ![Image](../images/f2victorit.jpg) 的作用。同时，外部载荷 ![Image](../images/qvictorit.jpg) 也施加在该节点上。该节点受到外部约束，! [Image](../images/Rvictor_caps.jpg) 是我们要计算的反应力。

![Image](../images/15fig23.jpg)

*图15-23：节点中的反应力*

从这些量中，只有 ![Image](../images/Rvictor_caps.jpg) 是未知的。杆件的内部力 ![Image](../images/f1victorit.jpg) 和 ![Image](../images/f2victorit.jpg) 是通过我们在[清单15-17](ch15.xhtml#ch15lis17)中实现的force_in_node方法计算的，而外部载荷 ![Image](../images/qvictorit.jpg) 是题目陈述中给出的。

假设节点处于静态平衡状态，必须满足以下条件。

![Image](../images/f0424-01.jpg)

你可能已经注意到，在这种情况下，杆件力的符号是负号。这些是节点从杆件的力中接收到的反应力，符合牛顿第三定律。如果一根杆件受到一对压缩力，它会将节点拉向自己。另一方面，如果一根杆件倾向于扩张，它会将节点推离自己。

我们可以轻松地从前面的方程中孤立出 ![Image](../images/Rvictor_caps.jpg)。

![Image](../images/f0424-02.jpg)

或者以更通用的方式表示（[方程式15.8](ch15.xhtml#ch15eqa08)），

![Image](../images/15eqa08.jpg)

其中 ![Image](../images/f00424-p1.jpg) 是所有杆件力的总和，! [Image](../images/f00424-p2.jpg) 是施加在该节点上的所有外部载荷的总和（节点的净载荷）。

让我们在我们的类中实现这一点。输入代码到[清单15-23](ch15.xhtml#ch15lis23)。

```py
import operator
from functools import reduce

from geom2d import make_rect_containing_with_margin, Vector
from .bar import StrBarSolution
from .node import StrNodeSolution

class StructureSolution:
   --snip--

    def reaction_for_node(self, node: StrNodeSolution):
     ➊ if not node.is_constrained:
            return Vector(0, 0)

     ➋ forces = [
            bar.force_in_node(node)
            for bar in self.bars
            if bar.has_node(node)
        ]

        if node.is_loaded:
         ➌ forces.append(node.net_load.opposite())

     ➍ return reduce(operator.add, forces)
```

*清单15-23：节点反应力*

我们定义了 reaction_for_node 方法，该方法根据给定的节点计算其反作用力。不要忘记，反作用力只存在于那些具有外部支撑或约束的节点。事实上，这就是我们首先检查的内容 ➊：如果节点没有约束，我们返回一个零向量（表示没有反作用力）。

第二步是搜索结构中与传入节点相连接的所有杆件，并获取它们在该节点的内力 ➋。我们使用列表推导式来实现这一点，遍历结构中的所有杆件，筛选出通过 bar.has_node(node) 测试的那些，最后将每个杆件映射到其在给定节点的内力。这就是[方程 15.8](ch15.xhtml#ch15eqa08)中的![Image](../images/f00424-p1.jpg)。

接下来，如果节点有外部载荷，我们就将净外部载荷添加到力列表中 ➌。请注意，来自节点的净载荷在[方程 15.8](ch15.xhtml#ch15eqa08)中以负号出现，这也是为什么我们对其调用相反的方法。还要注意，我们不需要对这些载荷求和（正如[方程 15.8](ch15.xhtml#ch15eqa08)中的![Image](../images/f00424-p2.jpg)所示），因为 StrNodeSolution 类已经为我们完成了这项工作，并提供了净载荷。

最后，使用 reduce 函数和 operator.add 运算符 ➍ 将列表中的所有力求和。

##### **最终结果**

供您参考，[清单 15-24](ch15.xhtml#ch15lis24)展示了完整的 StructureSolution 类实现。

```py
import operator
from functools import reduce

from geom2d import make_rect_containing_with_margin, Vector
from .bar import StrBarSolution
from .node import StrNodeSolution

class StructureSolution:
    def __init__(
            self,
            nodes: [StrNodeSolution],
            bars: [StrBarSolution]
    ):
        self.nodes = nodes
        self.bars = bars

    def bounds_rect(self, margin: float, scale=1):
        d_pos = [
            node.displaced_pos_scaled(scale)
            for node in self.nodes
        ]
        return make_rect_containing_with_margin(d_pos, margin)

    def reaction_for_node(self, node: StrNodeSolution):
        if not node.is_constrained:
            return Vector(0, 0)

        forces = [
            bar.force_in_node(node)
            for bar in self.bars
            if bar.has_node(node)
        ]

        if node.is_loaded:
            forces.append(node.net_load.opposite())

        return reduce(operator.add, forces)
```

*清单 15-24：结构解法类结果*

进行单元测试非常重要，以确保我们没有犯任何错误。尽管如此，为了进行测试，我们需要了解一种高级测试技术：模拟。我们将在下一章深入探讨这个话题，所以我们会回到这个实现。

### **总结**

我们在本章开始时回顾了一些材料力学的内容，比如弹性体在外力作用下所产生的内力。我们介绍了应力和应变的概念，这两者对于结构分析至关重要。我们特别关注了棱柱体中的轴向应力，因为这些应力在平面桁架结构中至关重要，而本书这一部分的重点就是平面桁架。

接下来，我们研究了平面桁架及其特点，并利用刚度矩阵的概念公式化了杆件上力与位移之间的关系。正如我们将在下一章中看到的，这些矩阵在结构求解中起着至关重要的作用。

最后，我们实现了结构的建模类：StrNode、StrBar 和 Structure。我们还实现了结构的求解类：StrNodeSolution、StrBarSolution 和 StructureSolution。这两组类分别表示原始设计的结构和结构求解，包括每根杆件的应力值和每个节点的位移。我们将在下一章中介绍如何从原始定义到求解的过程。
