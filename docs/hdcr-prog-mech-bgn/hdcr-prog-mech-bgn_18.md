## 14

线性方程

![图片](../images/common01.jpg)

许多工程问题需要求解线性方程组。这些方程出现在结构分析、电路、统计和优化问题中，仅举几例。实现解决这些普遍存在的方程组的算法是我们 *力学* 项目应对现实工程问题的关键。

本章我们将探讨 *数值方法* 的概念：使用计算机求解方程组的现有算法。我们将实现一种强大的方法来求解线性方程组：Cholesky 分解。当我们需要解决书中[第五部分](part05.xhtml#part05)中的结构分析问题中的大型方程组时，我们将使用这种方法。

### **线性方程组**

具有 *n* 个未知数 *x*[1]、*x*[2]、…、*x*[*n*] 的线性方程可以表示为[方程 14.1](ch14.xhtml#ch14eqa01)所示。

![图片](../images/14eqa01.jpg)

在这里，*m*[1]、*m*[2]、…、*m*[*n*] 是方程的系数，即乘以每个未知数的已知数字，而 *b* 是一个已知数字，不与任何未知数相乘。我们将这个最后的数字 *b* 称为 *自由项*。

如果未知数只是与标量相乘、相加或相减，则我们称该方程为 *线性* 方程。系数始终是已知量。另一种表示线性方程的方式如下，

![图片](../images/f0360-01.jpg)

其中 *m*[*i*] 是系数，*x*[*i*] 是未知数，*b* 是自由项。

相比之下，非线性方程包括未知数带有指数（*x*³）、三角函数（sin(*x*)）或多个未知数的乘积（*x*[1] ⋅ *x*[2]）。这些方程比线性方程更难求解，因此我们将专注于线性方程。

线性方程组的形式如[方程 14.2](ch14.xhtml#ch14eqa02)所示。

![图片](../images/14eqa02.jpg)

在这里，系数 *m*[*i,j*] 是乘以第 *i* 个方程中第 *j* 个未知数的项。这些方程组可以方便地表示为矩阵形式，如[方程 14.3](ch14.xhtml#ch14eqa03)所示。

![图片](../images/14eqa03.jpg)

在这里，[*M*] 是系数矩阵，

![图片](../images/f0360-02.jpg)

并且 [*x*] 和 [*b*] 是未知数和自由项列向量：

![图片](../images/f0360-03.jpg)

[方程 14.3](ch14.xhtml#ch14eqa03)的解是一个由数字 *x*[1]、*x*[2]、…、*x*[*n*] 组成的集合，满足所有 *n* 个方程。手动求解大型方程组可能需要很长时间，但不用担心：有很多算法可以用计算机求解这样的方程组。

关于命名法的简要说明。我们将在方括号内使用大写字母表示矩阵：[*M*]。矩阵中的元素将使用与矩阵相同字母的小写形式命名，并且在下标中包含两个用逗号分隔的数字，表示它们在矩阵中的行列索引。例如，矩阵[*M*]中第3行第5列的数字将被称为*m*[3, 5]。列向量和行向量用方括号内的小写字母表示：[*x*]。请记住，列向量和行向量也是矩阵。

### **数值方法**

*数值方法*是使用计算机的计算能力找到方程组近似解的算法。

有许多数值方法旨在求解线性、非线性和微分方程系统。然而，大多数数值方法仅限于求解特定类型的方程组。例如，Cholesky分解仅适用于系数矩阵是对称且正定的线性方程组（稍后我们将了解这意味着什么）。如果我们需要求解一个非线性方程组，或者一个线性但系数矩阵非对称的方程组，Cholesky分解将无法使用。

数值方法有两大类：*直接法*和*迭代法*。直接法通过对原始系统进行代数修改来求解系统。另一方面，迭代法从方程组的近似解开始，通过逐步改进，直到解达到所需的精度为止。Cholesky分解是一种直接数值方法。

数值方法是一个庞大的主题：已有专门的书籍讨论它。数值方法涉及许多技术细节，我们这里不会一一介绍。但这不是一本理论书籍；我们更关心实践，因此我们将实现一个算法，用于求解在接下来的结构分析应用中可能出现的方程组。在这种情况下，这意味着我们将处理具有对称、正定系数矩阵的线性方程组。

### **Cholesky分解**

*Cholesky分解*是一种直接（非迭代）方法，用于求解线性方程组，前提是它们的[*M*]（系数矩阵）是*对称*和*正定*的。

对称矩阵[*M*]是指其等于其转置矩阵：[*M*] = [*M*]^′。这意味着矩阵中的数值相对于主对角线是对称的。在对称矩阵中，每一行包含与同一索引的列相同的值，反之亦然。注意，要成为对称矩阵，矩阵必须是方阵。以下是一个对称矩阵的例子：

![Image](../images/f0361-01.jpg)

一个具有 *n* 行和列的方阵 [*M*] 被称为 *正定矩阵*，如果对于任何由 *n* 个实数（除了一个全零向量）构成的列向量 [*x*]，[方程 14.4](ch14.xhtml#ch14eqa04) 中的表达式成立。

![Image](../images/14eqa04.jpg)

如果你找到一个不满足上述方程的非零向量 [*x*]，那么矩阵 [*M*] 就不是正定的。

我们也可以说，如果一个矩阵是对称的，并且它的所有特征值都是正的，那么它是正定的。如果你记得获得矩阵特征值的过程，你可能会同意这既痛苦又有点无聊。无论如何，证明对于每个可能的 [*x*]，[*x*]^′[*M*][*x*] > 0，或者获得矩阵的所有特征值并确保它们都为正，都是一个复杂的过程。

我们将跳过所有这些技术复杂性，不会演示我们将使用的矩阵是正定的。我们将应用 Cholesky 分解来解决一个众所周知的问题，该问题会产生一个对称正定矩阵的方程组：使用直接刚度法进行的桁架结构分析。如果你需要将这个算法应用到其他问题中，你首先需要确定为该问题导出的方程组是否具有 Cholesky 可以处理的矩阵。如果不是，也不用担心：你可以使用很多其他数值方法。

在我们一起实现 Cholesky 算法后，希望你能够自己实现其他数值方法。正如你所看到的，确保我们正确处理这些复杂算法的最强大工具就是单元测试。

#### ***LU 分解方法***

Cholesky 是属于所谓 *LU* 分解或分解方法家族中的一种计算方法。给定方阵 [*M*] 的 *LU* 分解形式如 [方程 14.5](ch14.xhtml#ch14eqa05) 所示。

![Image](../images/14eqa05.jpg)

在这里，[*L*] 是一个 *下三角矩阵*，而 [*U*] 是一个 *上三角矩阵*。下三角矩阵是指所有非零值都位于主对角线及其下方。相反，上三角矩阵的非零值位于主对角线及其上方。以下是一个下三角矩阵和上三角矩阵的示例：

![Image](../images/f0362-01.jpg)

每个 *非奇异矩阵*（具有逆矩阵的矩阵）总是可以进行 *LU* 分解。例如，前面的矩阵就是该矩阵的 *LU* 分解：

![Image](../images/f00363-p1.jpg)

你可以通过乘以以下内容来验证这一点：

![Image](../images/f00363-p2.jpg)

Cholesky算法将为我们提供一个下三角矩阵和一个上三角矩阵。除了Cholesky之外，还有两种著名的方法可以用于获得任何非奇异矩阵的分解：Doolittle算法和Crout算法。这些算法定义了计算下三角矩阵和上三角矩阵的*l*[*i,j*]和*u*[*i,j*]值所需的公式。这些方法的好处是它们适用于任何类型的矩阵，而不仅仅是对称正定矩阵。我们在这里不讨论这些方法，但我鼓励你了解它们，并尝试在我们的*力学*项目中自己实现其中之一。你可以在我们实现Cholesky算法之后，把它作为练习来尝试。

值得问一下，为什么不使用可以处理任何非奇异矩阵的Doolittle或Crout算法，而非使用更具限制性的Cholesky算法呢？对于对称正定矩阵，Cholesky分解的速度大约是这些其他算法的两倍。由于我们将使用适用此方法的矩阵类型，我们希望能够从Cholesky方法所提供的执行速度中获益。

一旦我们获得了矩阵的*LU*分解，我们就可以通过两步解决方程组。假设我们的原始系统如下：

[*M*][*x*] = [*b*]

在对[*M*]进行分解后，我们得到了[方程 14.6](ch14.xhtml#ch14eqa06)。

![Image](../images/14eqa06.jpg)

如果我们取[*U*][*x*]的乘积并用一个新的未知向量[*y*]代替它，就可以从[方程 14.6](ch14.xhtml#ch14eqa06)中提取出两个系统：

![Image](../images/f0363-02.jpg)

我们现在有了一个下三角矩阵系统，如[方程 14.7](ch14.xhtml#ch14eqa07)所示，

![Image](../images/14eqa07.jpg)

并且有一个上三角矩阵系统，如[方程 14.8](ch14.xhtml#ch14eqa08)所示。

![Image](../images/14eqa08.jpg)

通过首先求解[方程 14.7](ch14.xhtml#ch14eqa07)，我们得到[*y*]，然后将其代入[方程 14.8](ch14.xhtml#ch14eqa08)，可以计算出未知向量[*x*]：即系统的解。 [方程 14.7](ch14.xhtml#ch14eqa07)和[方程 14.8](ch14.xhtml#ch14eqa08)都是具有三角矩阵的方程组，且可以通过前向和后向替代法轻松求解。

取这个系数矩阵是下三角矩阵的方程组：

![Image](../images/f00364-p1.jpg)

第一个未知数*y*[1]可以通过以下方式从第一个方程计算得出：

![Image](../images/f0364-01.jpg)

从第二个方程我们得到了以下内容，

![Image](../images/f0364-02.jpg)

这个系统可以求解，因为我们已经在前一步计算出了*y*[1]的值。我们对第三个方程做同样的处理：

![Image](../images/f0364-03.jpg)

我们已经有了*y*[1]和*y*[2]的值，因此可以计算出*y*[3]的值。这个过程称为*前向替代*。使用前向替代法得到*y*^(*第i项*)解的公式见于[方程 14.9](ch14.xhtml#ch14eqa09)（采用零基索引）。

![Image](../images/14eqa09.jpg)

在一个系数矩阵为上三角矩阵的系统中，我们可以使用类似的替代过程，但这次从底部开始。这个过程叫做*逆向替代*。这时我们得到：

![Image](../images/f0364-04.jpg)

从最后一个方程开始，我们可以计算*x*[3]：

![Image](../images/f0364-05.jpg)

有了这个值，我们可以进入第二个方程，计算*x*[2]：

![Image](../images/f0365-01.jpg)

最后，从系统中的第一个方程我们得到：

![Image](../images/f0365-02.jpg)

对于逆向替代，计算*x*^(*ith*)项的公式由[公式14.10](ch14.xhtml#ch14eqa10)描述，其中*n*是系统的大小。

![Image](../images/14eqa10.jpg)

我们很快就需要在代码中实现这些公式。你会发现它比看起来简单。

#### ***理解Cholesky***

如我们所讨论的，Cholesky分解是一种与对称正定矩阵配合使用的*LU*方法。由于这些特性，矩阵[*M*]可以分解成[*L*][*U*]形式，其中上三角矩阵是下三角矩阵的转置：[*U*] = [*L*]^′。这意味着我们只需要计算下三角矩阵[*L*]：[*U*]只是其转置。使用Cholesky方法，[*M*]矩阵的分解形式如[公式14.11](ch14.xhtml#ch14eqa11)所示。

![Image](../images/14eqa11.jpg)

所以，方程组现在看起来像[公式14.12](ch14.xhtml#ch14eqa12)。

![Image](../images/14eqa12.jpg)

在这种情况下，我们通过将[*L*]^′[*x*]替换为[*y*]，得到我们需要求解的两个方程组：

![Image](../images/f00365-p1.jpg)

如我们已经知道的，这个变换会产生一个下三角系统，我们将首先使用前向替代法来求解（见[公式14.13](ch14.xhtml#ch14eqa13)），

![Image](../images/14eqa13.jpg)

然后，使用一个上三角系统，通过逆向替代法求解，得到解向量[*x*]（见[公式14.14](ch14.xhtml#ch14eqa14)）。

![Image](../images/14eqa14.jpg)

给定一个对称正定矩阵[*M*]，我们可以使用[公式14.15](ch14.xhtml#ch14eqa15)中的公式计算其Cholesky分解的下三角矩阵项，即*l*[*i,j*]项。

![Image](../images/14eqa15.jpg)

[公式14.15](ch14.xhtml#ch14eqa15)可能看起来令人畏惧，但实际上并没有那么复杂。最好的理解方式是通过手动做一个练习。拿起一支笔和一些纸，我们一起进行矩阵分解。

#### ***手动因式分解***

给定对称正定矩阵

![Image](../images/f0366-01.jpg)

让我们找到它的Cholesky分解，一个下三角矩阵[*L*]，

![Image](../images/f0366-02.jpg)

使得[*M*] = [*L*][*L*]^′。为了计算*l*[*i,j*]项，我们使用[公式14.15](ch14.xhtml#ch14eqa15)。别忘了，索引*i*表示矩阵的行，*j*表示矩阵的列。我们一步一步来。

**步骤 1：** *i* = 0，*j* = 0。由于 *i* = *j*，我们使用第一个公式：

![Image](../images/f0366-03.jpg)

请注意，求和符号被划掉了，因为它没有产生任何项。这是因为求和的结束值 *k* = –1 小于开始值 *k* = 0。如你所知，为了让求和产生任何项，*k*（迭代变量）的结束值必须等于或大于开始值。

**步骤 2：** *i* = 1，*j* = 0。此时，*i*≠*j*，所以我们使用第二个公式：

![Image](../images/f0366-04.jpg)

**步骤 3：** *i* = 1，*j* = 1。

![Image](../images/f0367-01.jpg)

**步骤 4：** *i* = 2，*j* = 0。

![Image](../images/f0367-02.jpg)

**步骤 5：** *i* = 2，*j* = 1。

![Image](../images/f0367-03.jpg)

**步骤 6：** *i* = 2，*j* = 2。

![Image](../images/f0367-04.jpg)

如果我们将所有计算出的 *l*[*i,j*] 值合并，得到的矩阵如下：

![Image](../images/f0367-05.jpg)

这意味着原始系统的矩阵 [*M*] 可以如下因式分解：

![Image](../images/f0367-06.jpg)

你可以进行矩阵乘法验证，*L*[*L*]^′ 的积实际上等于 *M*。为了完成这个练习，假设这个矩阵是一个方程组的系数矩阵，并使用前向和后向替代法来求解它。

#### ***手动解法***

假设我们之前分解的矩阵 [*L*][*L*]^′ 形式是以下方程组的一部分：

![Image](../images/f0367-07.jpg)

我们需要找到满足所有三个方程的 *x*[1]、*x*[2] 和 *x*[3] 的值。使用我们刚刚获得的 Cholesky 分解，我们可以将系统重写为如下形式：

![Image](../images/f0368-01.jpg)

我们需要解的两个子系统中的第一个是 [*L*][*y*] = [*b*]，这是通过将 [*L*]^′[*x*] 替换为新的未知向量 [*y*] 得到的：

![Image](../images/f0368-02.jpg)

这就得到了第一个系统（下系统）：

![Image](../images/f0368-03.jpg)

我们必须使用 [方程 14.9](ch14.xhtml#ch14eqa09) 中的前向替代公式来求解这个系统。

##### **下系统：前向替代**

让我们逐步应用 [方程 14.9](ch14.xhtml#ch14eqa09)：

**步骤 1：** *i* = 0。

![Image](../images/f0368-04.jpg)

**步骤 2：** *i* = 1。

![Image](../images/f0368-05.jpg)

**步骤 3：** *i* = 2。

![Image](../images/f0368-06.jpg)

因此，第一个系统的解如下：

![Image](../images/f0369-01.jpg)

有了这个解，我们可以使用后向替代法计算 [*x*]：即我们的方程组的解。

##### **上系统：后向替代**

让我们使用 [方程 14.10](ch14.xhtml#ch14eqa10) 逐步计算解向量。这一次，我们需要使用后向替代法来求解以下方程组：

![Image](../images/f0369-02.jpg)

由于替代过程是反向的，我们必须从最后一行（*i* = 2）开始，一直到第一行（*i* = 0）。

**步骤 1：** *i* = 2。

![Image](../images/f0369-03.jpg)

**步骤 2：** *i* = 1。

![Image](../images/f0369-04.jpg)

**步骤 3：** *i* = 0。

![Image](../images/f0369-05.jpg)

然后，初始系统的解如下：

![Image](../images/f0369-06.jpg)

你可以通过检查等式是否成立来测试解是否正确：

![Image](../images/f0369-07.jpg)

现在我们知道了Cholesky算法的工作原理，并且已经通过手算做了一个示例，让我们在代码中实现这个算法。

#### ***实现Cholesky***

首先在*eqs*包中创建一个名为*cholesky.py*的新文件。在该文件中，包含[清单 14-1](ch14.xhtml#ch14lis1)中的cholesky_solve函数。

```py
import math

from eqs.matrix import Matrix
from eqs.vector import Vector

def cholesky_solve(sys_mat: Matrix, sys_vec: Vector):
    validate_system(sys_mat, sys_vec)

    low_matrix = lower_matrix_decomposition(sys_mat)
    low_solution = solve_lower_sys(low_matrix, sys_vec)
    return solve_upper_sys(low_matrix, low_solution)
```

*清单 14-1：Cholesky分解算法*

此函数接受矩阵和向量作为输入。这些是系统的系数矩阵和自由向量：来自方程组[*M*][*x*] = [*b*]中的[*M*]和[*b*]。返回的向量是[*x*]，即通过Cholesky方法求得的系统解。

这个cholesky_solve函数定义了最高层的算法，包含三个主要步骤和一个输入系统的验证。我们还没有实现这些函数；我们稍后会实现。以下是算法的三个主要步骤：

lower_matrix_decomposition 通过应用[方程 14.15](ch14.xhtml#ch14eqa15)获得[*L*]，即下三角矩阵。

solve_lower_sys 通过应用前向代入法（见[方程 14.9](ch14.xhtml#ch14eqa09)）解第一个子系统，即下三角系统。

solve_upper_sys 通过应用回代法（见[方程 14.10](ch14.xhtml#ch14eqa10)）解第二个子系统，即上三角系统。

从函数名称来看，很容易看出cholesky_solve中的代码在做什么。请注意，我们将函数拆分成了几个较小的函数。如果我们把所有用于Cholesky分解的代码都丢进cholesky_solve函数中，结果将是一个没有明显结构的长代码块。这段代码将非常难以理解。

一般来说，你需要将大的算法分解成较小的子算法，每个子算法都包含在一个具有描述性名称的小函数中。

请注意cholesky_solve使用的子函数的可见性。所有子函数都是公开的。这是为了能够单独对它们进行单元测试。分解算法稍显复杂，如果我们知道每个子部分都能正确执行其功能，处理起来会更安全。

##### **验证系统**

让我们实现一个函数，验证系统是否是方阵，并且列数是否等于向量的大小。输入[清单 14-2](ch14.xhtml#ch14lis2)中的validate_system函数代码。

```py
--snip--

def validate_system(sys_matrix: Matrix, sys_vector: Vector):
    if sys_matrix.cols_count != sys_vector.length:
        raise ValueError('Size mismatch between matrix and vector')

    if not sys_matrix.is_square:
        raise ValueError('System matrix must be square')
```

*清单 14-2：系统验证*

我们首先检查矩阵的列数是否与向量的长度相同。如果这个条件不满足，系统就无法解，因此我们会抛出一个错误。如果系统的矩阵不是方阵，情况也是如此。

我们并没有检查矩阵是否对称或正定；如果传递给我们函数的矩阵不满足这些条件，函数最终会因为除零错误或类似问题而失败。添加这些保护措施是一个不错的主意，至少要检查对称性，但检查矩阵是否正定可能会更具挑战性。对称性检查容易实现，但缺点是计算开销大。我鼓励你考虑如何进行这些检查，并可能将它们添加到你的代码中。

现在我们要做点反向操作。我们将从单元测试开始，而不是直接从代码本身开始。这样我们就能知道何时代码准备好：一旦测试通过。我们可以不断运行测试来检查我们编写的逻辑是否已经准备好；我们可以重构代码，直到它变得可读，并且有测试的安全网，当我们做错了什么时，测试会警告我们。这种在编写代码之前编写测试的技术称为*测试驱动开发*，简称TDD。

我们将首先查看我们将在单元测试中使用的方程组。

##### **测试用方程组**

为了确保我们实现的所有逻辑没有错误，我们将为Cholesky算法中的每个子函数编写测试。我们还将实现一个测试，检查所有子函数是否能够协同工作以计算最终解。对于这些测试，我们希望使用一个我们事先知道解的方程组。

让我们使用以下大小为4的系统：

![Image](../images/f0372-01.jpg)

对于该系统的矩阵[*M*]，Cholesky [*L*][*L*]^′ 分解如下：

![Image](../images/f0372-02.jpg)

下层系统的解，

![Image](../images/f0372-03.jpg)

是以下向量：

![Image](../images/f0372-04.jpg)

最终解，来自上层系统的求解，

![Image](../images/f0372-05.jpg)

是以下向量：

![Image](../images/f0372-06.jpg)

花时间检查所有这些数字，并确保你理解解算过程是一个好主意。一旦你掌握了这个过程的基本原理，我们就开始编码，从单元测试开始。

##### **下三角矩阵分解**

由于我们即将实现本书迄今为止最复杂的算法，首先让我们编写一个单元测试。我们会知道我们的分解逻辑是否正确实现，一旦测试通过。很可能我们需要调试代码，拥有一个测试会帮助我们。

创建一个新的文件用于我们的测试，*cholesky_test.py*，并将其放入*eqs/tests*目录下。然后在[Listing 14-3](ch14.xhtml#ch14lis3)中输入测试代码。

```py
import unittest

from eqs.cholesky import lower_matrix_decomposition
from eqs.matrix import Matrix

class CholeskyTest(unittest.TestCase):
    sys_matrix = Matrix(4, 4).set_data([
        4, -2, 4, 2,
        -2, 10, -2, -7,
        4, -2, 8, 4,
        2, -7, 4, 7
    ])
    low_matrix = Matrix(4, 4).set_data([
        2, 0, 0, 0,
        -1, 3, 0, 0,
        2, 0, 2, 0,
        1, -2, 1, 1
    ])

    def test_lower_matrix_decomposition(self):
        actual = lower_matrix_decomposition(self.sys_matrix)
        self.assertEqual(self.low_matrix, actual)
```

*Listing 14-3：测试下三角矩阵分解*

这个测试定义了原始矩阵sys_matrix和期望的分解矩阵low_matrix。使用一个我们尚未定义的函数lower_matrix_decomposition，我们计算分解矩阵并将其与已知解进行比较。你的IDE应该会报错，提示你试图导入一个在*eqs.cholesky*模块中找不到的函数：

在'cholesky.py'中找不到引用'lower_matrix_decomposition'

让我们来实现这个函数。返回到 *cholesky.py* 文件，在validate_system之后，输入[Listing 14-4](ch14.xhtml#ch14lis4)中的代码。

```py
--snip--

def lower_matrix_decomposition(sys_mat: Matrix):
    size = sys_mat.rows_count
    low_mat = Matrix(size, size)

    for i in range(size):
        sq_sum = 0

        for j in range(i + 1): 
         ➊ m_ij = sys_mat.value_at(i, j)

           if i == j:
               # main diagonal value
            ➋ diag_val = math.sqrt(m_ij - sq_sum)
            ➌ low_mat.set_value(diag_val, i, j)

           else:
               # value under main diagonal
               non_diag_sum = 0
            ➍ for k in range(j):
                   l_ik = low_mat.value_at(i, k)
                   l_jk = low_mat.value_at(j, k)
                   non_diag_sum += l_ik * l_jk

               l_jj = low_mat.value_at(j, j)
            ➎ non_diag_val = (m_ij - non_diag_sum) / l_jj
            ➏ sq_sum += non_diag_val * non_diag_val

            ➐ low_mat.set_value(non_diag_val, i, j)

    return low_mat
```

*Listing 14-4：下三角矩阵分解*

我们首先将系统的大小存储在一个名为size的变量中。大小是行数或列数——由于矩阵是方阵，因此二者无关。接着我们创建一个相同大小的新方阵low_mat。回顾一下，我们的矩阵在实例化时是用零填充的。

该算法有两个主要的嵌套循环。这些循环遍历矩阵中主对角线及其下方的所有位置，也就是说，遍历所有*m*[*i,j*]，其中*i* ≥ *j*。

**注意**

*不要忘记，Python的* range(n) *函数生成的是从0到*n - 1*的序列，而不是*n*。

在j循环内部，我们将系统矩阵在位置(*i,j*)的值存储在m_ij ➊中。然后，我们通过if else语句区分是在主对角线（i == j）上，还是在其下方。回顾一下，计算分解矩阵主对角线上一个项的公式如下：

![Image](../images/f0374-01.jpg)

我们使用这个表达式来计算值，将其存储在diag_val ➋中，并设置在矩阵中 ➌。计算中我们使用了m_ij值和sq_sum。后者在每次新的*i*（每一行）迭代时初始化为0，并在主对角线下方的每个新值更新 ➏。

对于主对角线下方的情况（i > j，即else分支），计算*l*[*i,j*]项的公式如下：

![Image](../images/f0375-01.jpg)

请注意，为了计算这个*l*[*i,j*]值，我们需要有*l*[*j,j*]，这是来自前一行的一个值，因为*i* > *j*。我们计算的第一个项是*l*[*i,k*]*l*[*j,k*]的和，*k*从0到*j - 1*。步骤 ➍ 中的循环正是这样做的。在进入循环之前，我们将一个名为non_diag_sum的变量初始化为零。在循环内部，这个变量在每次循环中都会加上l_ik和l_jk的乘积。

计算出non_diag_sum后，我们就拥有了所需的一切。*l*[*j,j*]的值从low_mat中提取，并存储在变量l_jj中。然后，分解的值被计算出来并存储在变量non_diag_val ➎中。这个值首先用于更新sq_sum ➏，然后存储在分解矩阵 ➐中。

就这样。运行我们之前编写的测试，确保你的代码能够通过测试。如果没有通过，不用担心；事实上，第一次编写这个算法时，可能很难完全正确，但这正是我们先实现测试的原因。使用测试来调试代码，并仔细对比你编写的代码和本书中打印出的代码版本。你也可以参考书中附带的代码。

在 shell 中运行测试，使用以下命令：

```py
$ python3 -m unittest eqs/tests/cholesky_test.py
```

我们已经通过 Cholesky 算法得到了 [*L*] 分解矩阵。现在让我们实现下三角和上三角系统的求解。

##### **下三角系统求解**

为了使用前向代换法求解下三角系统，我们需要实现[方程 14.9](ch14.xhtml#ch14eqa09)中的算法。为了方便起见，我们在这里重复公式：

![Image](../images/f0375-02.jpg)

我们将采用与之前相同的方法，先编写测试，再编写主代码。在 *cholesky_test.py* 文件中，在[清单 14-5](ch14.xhtml#ch14lis5)中输入新的测试。

```py
import unittest

from eqs.cholesky import lower_matrix_decomposition, \
    solve_lower_sys
from eqs.matrix import Matrix
from eqs.vector import Vector

class CholeskyTest(unittest.TestCase):
    --snip--
  ➊ sys_vec = Vector(4).set_data([20, -16, 40, 28])
  ➋ low_solution = Vector(4).set_data([10, -2, 10, 4])

    def test_lower_matrix_decomposition(self):
        actual = lower_matrix_decomposition(self.sys_matrix)
        self.assertEqual(self.low_matrix, actual)

  ➌ def test_lower_system_resolution(self):
        actual = solve_lower_sys(self.low_matrix, self.sys_vec)
        self.assertEqual(self.low_solution, actual)
```

*清单 14-5：测试下三角系统求解*

我们首先从 eqs.vector 导入了 Vector 类。然后我们添加了两个新向量，这些向量是新测试所需的：sys_vec ➊，这是方程组的自由向量，以及 low_solution ➋，这是下三角系统的预期解。

在测试就位 ➌ 后，现在让我们实现缺失的 solve_lower_sys 函数。在 *cholesky.py* 文件中的分解函数后，输入[清单 14-6](ch14.xhtml#ch14lis6)中的代码。

```py
--snip--

def solve_lower_sys(low_mat: Matrix, vector: Vector):
    size = vector.length
    solution = Vector(size)

 ➊ for i in range(size):
        _sum = 0.0

     ➋ for j in range(i):
            l_ij = low_mat.value_at(i, j)
            y_j = solution.value_at(j)
            _sum += l_ij * y_j

        b_i = vector.value_at(i)
        l_ii = low_mat.value_at(i, i)
     ➌ solution_val = (b_i - _sum) / l_ii
        solution.set_value(solution_val, i)

    return solution
```

*清单 14-6：求解下三角系统*

我们做的第一件事是将系统的大小保存在一个变量 size 中，并创建一个该大小的解向量。主循环是 i 循环 ➊，它遍历 sys_vector 中的所有值。在循环中，我们首先将和初始化为零。j 循环 ➋ 遍历从 0 到 *i –* 1 的所有值，并在每次迭代时更新和。

得到方程的和部分后，我们可以计算出解值，并将其存储在 solution_val ➌ 中。然后我们在下一行设置解向量。

运行 *cholesky_test.py* 中的两个测试，确保都通过。第一个测试通过似乎是合理的：我们没有以任何方式修改分解函数，但最好运行文件中的所有测试，以防我们修改了不该修改的东西。我希望第二个测试对你来说也能通过，这样你就成功实现了新函数！否则，你需要调试你的代码。请花时间这样做，这是一个很好的练习。

从 shell 运行测试，使用以下命令：

```py
$ python3 -m unittest eqs/tests/cholesky_test.py
```

现在让我们来处理上三角系统的求解。

##### **上三角系统求解**

使用回代法求解上三角系统可以通过[方程 14.10](ch14.xhtml#ch14eqa10)来实现。为了提醒你，公式如下：

![Image](../images/f0377-01.jpg)

需要记住的一件重要事是，上三角矩阵 [*U*]，其值为 *u*[*i,j*]，是 Cholesky 下三角分解的转置：[*L*]^′。

我们再次从测试开始。打开你的 *cholesky_test.py* 文件，并在 [清单 14-7](ch14.xhtml#ch14lis7) 中输入新的测试。

```py
import unittest

from eqs.cholesky import lower_matrix_decomposition, \
    solve_lower_sys, solve_upper_sys
from eqs.matrix import Matrix
from eqs.vector import Vector

class CholeskyTest(unittest.TestCase):
    --snip--
 ➊ solution = Vector(4).set_data([1, 2, 3, 4])

    def test_lower_matrix_decomposition(self):
        actual = lower_matrix_decomposition(self.sys_matrix)
        self.assertEqual(self.low_matrix, actual)

    def test_lower_system_resolution(self):
        actual = solve_lower_sys(self.low_matrix, self.sys_vec)
        self.assertEqual(self.low_solution, actual)

 ➋ def test_upper_system_resolution(self):
        actual = solve_upper_sys(
            self.low_matrix,
            self.low_solution
        )
        self.assertEqual(self.solution, actual)
```

*清单 14-7：测试上三角系统的求解*

在这个新测试中 ➋，我们调用 solve_upper_sys（尚未编写），传入分解后的矩阵 low_matrix 和下三角系统的解 low_solution。然后，我们断言返回的向量是我们期望的那个，即我们在测试数据中的 solution 变量中定义的向量 ➊。

我们现在准备实现最后一部分，以完成 Cholesky 方法：求解上三角系统。再次打开 *cholesky.py* 文件，并进入 [清单 14-8](ch14.xhtml#ch14lis8) 中的 solve_upper_sys 函数。

```py
--snip--

def solve_upper_sys(up_matrix: Matrix, vector: Vector):
    size = vector.length
    last_index = size - 1
    solution = Vector(size)

 ➊ for i in range(last_index, -1, -1):
        _sum = 0.0

     ➋ for j in range(i + 1, size):
         ➌ u_ij = up_matrix.value_transposed_at(i, j)
            x_j = solution.value_at(j)
            _sum += u_ij * x_j

        y_i = vector.value_at(i)
     ➍ u_ii = up_matrix.value_transposed_at(i, i)
     ➎ solution_val = (y_i - _sum) / u_ii
        solution.set_value(solution_val, i)

    return solution
```

*清单 14-8：求解上三角系统*

该函数类似于之前的 solve_lower_sys 函数。我们首先初始化解向量 solution，大小与传入的 low_vector 相同。这一次，因为我们从最后一行开始迭代，所以我们将其索引保存在 last_index 变量中。

迭代所有行索引的循环从 last_index 一直到 -1（不包括） ➊。内部循环从 *i* + 1 到 size（同样不包括），计算 *u*[*i,j*]*x*[*j*] 的乘积之和 ➋。为了得到 u_ij，我们像访问转置矩阵一样访问下三角矩阵的值 ➌。多亏了这个巧妙的技巧，我们避免了转置 [*L*]，这一过程计算开销很大。这就是我们在前一章中讨论的优化方法。

为了得到 [公式 14.10](ch14.xhtml#ch14eqa10) 中的除数，我们再次使用 value_transposed_at 函数 ➍。有了这个值，我们就可以计算每一行的解 ➎，并将其存储在结果向量中。

运行文件中的所有测试，检查实现是否没有错误。仅供参考，[清单 14-9](ch14.xhtml#ch14lis9) 是完整的 *cholesky.py* 文件。

```py
import math

from eqs.matrix import Matrix
from eqs.vector import Vector

def cholesky_solve(sys_mat: Matrix, sys_vec: Vector) -> Vector:
    validate_system(sys_mat, sys_vec)

    low_matrix = lower_matrix_decomposition(sys_mat)
    low_solution = solve_lower_sys(low_matrix, sys_vec)
    return solve_upper_sys(low_matrix, low_solution)

def validate_system(sys_matrix: Matrix, sys_vector: Vector):
    if sys_matrix.cols_count != sys_vector.length:
        raise ValueError('Size mismatch between matrix and vector')

    if not sys_matrix.is_square:
        raise ValueError('System matrix must be square')

def lower_matrix_decomposition(sys_mat: Matrix) -> Matrix:
    size = sys_mat.rows_count
    low_mat = Matrix(size, size)

    for i in range(size):
        sq_sum = 0

        for j in range(i + 1):
            m_ij = sys_mat.value_at(i, j)

            if i == j:
                # main diagonal value
                diag_val = math.sqrt(m_ij - sq_sum)
                low_mat.set_value(diag_val, i, j)

            else:
                # value under main diagonal
                non_diag_sum = 0
                for k in range(j):
                    l_ik = low_mat.value_at(i, k)
                    l_jk = low_mat.value_at(j, k)
                    non_diag_sum += l_ik * l_jk

                l_jj = low_mat.value_at(j, j)
                non_diag_val = (m_ij - non_diag_sum) / l_jj
                sq_sum += non_diag_val * non_diag_val

                low_mat.set_value(non_diag_val, i, j)

    return low_mat

def solve_lower_sys(low_mat: Matrix, vector: Vector):
    size = vector.length
    solution = Vector(size)

    for i in range(size):
        _sum = 0.0

        for j in range(i):
            l_ij = low_mat.value_at(i, j)
            y_j = solution.value_at(j)
            _sum += l_ij * y_j

        b_i = vector.value_at(i)
        l_ii = low_mat.value_at(i, i)
        solution_val = (b_i - _sum) / l_ii
        solution.set_value(solution_val, i)

    return solution

def solve_upper_sys(up_matrix: Matrix, vector: Vector):
    size = vector.length
    last_index = size - 1
    solution = Vector(size)

    for i in range(last_index, -1, -1):
        _sum = 0.0

        for j in range(i + 1, size):
            u_ij = up_matrix.value_transposed_at(i, j)
            x_j = solution.value_at(j)
            _sum += u_ij * x_j

        y_i = vector.value_at(i)
        u_ii = up_matrix.value_transposed_at(i, i)
        solution_val = (y_i - _sum) / u_ii
        solution.set_value(solution_val, i)

    return solution
```

*清单 14-9：Cholesky 方法结果*

参与 Cholesky 方法求解线性方程组的三个子函数已经分别经过测试：我们可以确保它们正常工作。难道这就意味着 cholesky_solve 函数本身没有 bug 吗？不一定。将这些经过充分测试的函数组合在一起时，我们可能仍会犯错。

检查 cholesky_solve 函数整体是否正常工作需要进行一次额外的测试。这是一个确保每个子函数在组合时能正常工作的重要测试，称为 *集成测试*。

#### ***测试 Cholesky：集成测试***

最后一次打开你的 *cholesky_test.py* 文件。让我们添加一个最终的测试（如 [清单 14-10](ch14.xhtml#ch14lis10) 所示）。

```py
import unittest

from eqs.cholesky import lower_matrix_decomposition, \
    solve_lower_sys, solve_upper_sys, cholesky_solve
from eqs.matrix import Matrix
from eqs.vector import Vector

class CholeskyTest(unittest.TestCase):
    sys_matrix = Matrix(4, 4).set_data([
        4, -2, 4, 2,
        -2, 10, -2, -7,
        4, -2, 8, 4,
        2, -7, 4, 7
    ])
    low_matrix = Matrix(4, 4).set_data([
        2, 0, 0, 0,
        -1, 3, 0, 0,
        2, 0, 2, 0,
        1, -2, 1, 1
    ])
    sys_vec = Vector(4).set_data([20, -16, 40, 28])
    low_solution = Vector(4).set_data([10, -2, 10, 4])
    solution = Vector(4).set_data([1, 2, 3, 4])

    def test_lower_matrix_decomposition(self):
        actual = lower_matrix_decomposition(self.sys_matrix)
        self.assertEqual(self.low_matrix, actual)

    def test_lower_system_resolution(self):
        actual = solve_lower_sys(self.low_matrix, self.sys_vec)
        self.assertEqual(self.low_solution, actual)

    def test_upper_system_resolution(self):
        actual = solve_upper_sys(
            self.low_matrix,
            self.low_solution
        )
        self.assertEqual(self.solution, actual)

    def test_solve_system(self):
        actual = cholesky_solve(self.sys_matrix, self.sys_vec)
        self.assertEqual(self.solution, actual)
```

*清单 14-10：测试 Cholesky 分解方法*

[列表 14-10](ch14.xhtml#ch14lis10)是生成的测试文件。我们加入了最后一个测试：test_solve_system。这个测试通过调用cholesky_solve来整体测试Cholesky算法。

运行文件中的所有测试。如果所有四个测试都通过了，那就说明你写的代码完全正确。你应该为自己在这一长章中跟随代码而感到自豪。恭喜你！

如果你想从命令行运行测试，请使用以下命令：

```py
$ python3 -m unittest eqs/tests/cholesky_test.py
```

### **总结**

在这一章中，我们讨论了数值方法，然后将讨论重点放在了解决线性方程组的算法上。特别地，我们分析了Cholesky分解方法。这个[*L*][*U*]分解算法适用于对称正定矩阵，且速度是其他[*L*][*U*]算法的两倍。

我们特别关注代码的可读性。为了使算法易于理解，我们将其拆分成更小的函数，每个函数都进行了单独测试。我们在编写主算法逻辑之前就开始编写测试，这种方法被称为测试驱动开发。我们加入了最后一个测试，集成了完整的线性方程组求解。

我们实现了一个强大的求解算法，并将在本书的[第五部分](part05.xhtml#part05)中使用它。
