

## 第二十章：20 使用 Composer 管理类和命名空间



![](img/opener.jpg)

随着你的 PHP 项目变得越来越大且复杂，你越来越有可能遇到 *命名冲突* 的问题，或者出现两个同名的类。在本章中，你将了解 *命名空间*，这是一种面向对象语言提供的解决方案，用于避免命名冲突。此外，你还将学习如何使用有用的 Composer 命令行工具，它能够自动加载类和函数声明文件，并简化与命名空间的工作。几乎每个现代的面向对象 PHP 项目都会使用 Composer，我们将在本书的其余部分中使用它。

你可能认为命名冲突不太可能发生；毕竟，直到现在，我们一直在 PHP 文件中编写与类名相同的类声明，并将这些类声明文件放置在项目的 *src* 目录中。由于 PHP 不允许在同一目录中有两个同名的文件，难道我们不会遇到两个同名的类吗？

事实上，命名冲突可能在几种情况下发生。首先，你可能会尝试声明一个与 PHP 语言的内置类同名的类，例如 Error、Directory 或 Generator。其次，你可能会在不同的目录中声明两个同名的类（例如，在 *src* 的不同子目录中）。第三，你可能会将自己的类与第三方库的类混合在一起。

### 命名空间

*命名空间* 可以看作是一个虚拟的目录层级结构，用于存放类，以避免类名冲突。类被组织在命名空间和子命名空间中，就像计算机文件被组织在目录和子目录中一样。就像你需要指定计算机文件在硬盘上的目录位置一样，使用命名空间的类也需要同时指定类名和命名空间，以唯一标识某个类。

反斜杠字符（\）用于分隔命名空间、子命名空间（如果有的话）和类名。例如，\MyNamespace\MySubNamespace\MyClass 指的是一个名为 MyClass 的类，它位于 MySubNamespace 子命名空间中，而 MySubNamespace 是更大命名空间 MyNamespace 的一部分。通过命名空间和子命名空间来标识 MyClass，可以避免与其他命名空间中名为 MyClass 的类发生冲突，例如 \YourNamespace\MyClass。根据约定，命名空间或子命名空间的首字母大写，类似于类名。命名空间或子命名空间中的其他字母也可以大写。

PHP 语言内建的类被认为是位于*根命名空间*，它只由一个反斜杠字符标识。例如，你可以写 \DateTime 或 \Exception 来明确引用 PHP 内建的 DateTime 或 Exception 类。在本书至今的章节中，我们省略了内建类名称前的反斜杠，因为我们在编写自己的类时并未使用命名空间。包含反斜杠使得我们明确表示是在引用根命名空间中的 PHP 类。

在接下来的示例中，我将使用 Mattsmithdev 命名空间。这是我为我编写的所有类所使用的命名空间，每个我工作过的项目都有一个子命名空间。你可能想为自己创建一个命名空间，例如 Supercoder 或 DublinDevelopers，并在跟随本书的章节时使用它并编写自己的类。我们还将在“将第三方库添加到项目中”一章中遇到其他命名空间，见 第 390 页。

#### 声明类的命名空间

要声明类的命名空间，使用 namespace 关键字后跟命名空间的名称。这应该是类声明文件中的第一行 PHP 代码。为了演示，我们将声明一个名为 Shirt 的类，并将其归属于 Mattsmithdev 命名空间。开始一个新项目，创建一个名为 *src/Shirt.php* 的文件，并输入 列表 20-1 中的代码。

```
<?php
namespace Mattsmithdev;

class Shirt
{
    private string $type ='t-shirt';

    public function getType(): string
    {
        return $this->type;
    }

    public function setType(string $type): void
    {
        $this->type = $type;
    }
}
```

列表 20-1：Mattsmithdev 命名空间中的 Shirt 类

在 PHP 标签的开头，我们使用 namespace Mattsmithdev 来将我们接下来要声明的类包含到 Mattsmithdev 命名空间中。我们在命名空间声明后加上两行空白行，这是 PHP 编码规范推荐的做法。然后，我们照常进行类声明。在这个例子中，Shirt 类有一个名为 type 的私有属性，默认值为 't-shirt'，并且有该属性的公共 getter 和 setter 方法。

#### 使用命名空间类

一旦类声明属于一个命名空间，你需要明确告诉 PHP 引擎这是你想要使用的类。你可以通过两种方式来做到这一点。

第一个选项是始终在引用类时包含命名空间；这被称为使用类的*完全限定名称*。例如，要创建一个新的 Shirt 对象，你可以写 new \Mattsmithdev\Shirt()。现在让我们尝试一下。将 *public/index.php* 文件添加到你的项目中，并输入 列表 20-2 中的代码。

```
<?php
require_once __DIR__ . '/../src/Shirt.php';

$shirt1 = new \Mattsmithdev\Shirt();
$shirt2 = new \Mattsmithdev\Shirt();

print "shirt 1 type = {$shirt1->getType()}";
```

列表 20-2：在 index.php 中创建 \Mattsmithdev\Shirt 类的对象

在读取类声明文件之后，我们创建了两个 Shirt 类的对象，使用该类的完全限定名称。通过命令行运行项目，使用 php public/index.php，你应该看到以下内容：

```
shirt 1 type = t-shirt
```

消息指示通过类的完全限定名称成功创建了一个 Shirt 对象。

引用特定命名空间中的类的第二种方式是，在调用类之前包含一个`use`语句。例如，`use Mattsmithdev\Shirt` 告诉 PHP 引擎，任何后续对 Shirt 类的引用都是专门指向 Mattsmithdev 命名空间中的那个类。要查看 `use` 语句如何工作，请更新你的*public/index.php* 文件以匹配 Listing 20-3。

```
<?php
require_once __DIR__ . '/../src/Shirt.php';

use Mattsmithdev\Shirt;

$shirt1 = new Shirt();
$shirt2 = new Shirt();

print "shirt 1 type = {$shirt1->getType()}";
```

Listing 20-3: 在 index.php 中使用 `use` 语句引用 Shirt 类

我们在读取类声明后包含一个 `use` 语句，以确保代码中后续的 Shirt 引用指向 Mattsmithdev\Shirt。请注意，在 `use` 语句中我们不包括命名空间前的反斜杠。这种类标识符，没有初始反斜杠，称为*限定名*，与包含初始反斜杠的*完全限定名*相对。然后我们可以简单地使用 `new Shirt()` 来创建两个 Shirt 对象，因为得益于 `use` 语句，PHP 引擎知道我们正在引用哪个类。重新运行 index 脚本，你应该会看到输出没有变化。我们仍然成功地创建了一些 Shirt 对象。

如果你需要在同一段代码中区分两个同名但属于不同命名空间的类，你可以通过它们的完全限定名来引用它们（例如，\Mattsmithdev\Shirt 和 \OtherNamespace\Shirt），或者为其中一个类提供 `use` 语句，并对另一个类进行限定。

#### 在类声明中引用命名空间

假设你正在为一个命名空间类编写类声明文件（而不是像*index.php* 这样的通用脚本），并且你想引用来自其他命名空间的类。如果你没有写 `use` 语句，你必须使用另一个类的完全限定名，从反斜杠开始。例如，如果你正在为一个在 Mattsmithdev 命名空间中声明的类编写代码，并且你想引用 PHP 内置的 DateTime 类，你必须写成 `\DateTime` 来表明它属于根命名空间。同样，如果你想引用一个第三方类，你需要写一个反斜杠，然后是第三方命名空间，再一个反斜杠，最后是类名，例如 `\MathPHP\Algebra`。

如果没有初始反斜杠，PHP 会假设你引用的是当前命名空间中的类或子命名空间。例如，在 Mattsmithdev 命名空间中的一个类中，如果引用 `DateTime()` 没有加初始反斜杠，PHP 会认为是指 `Mattsmithdev\DateTime`，即 Mattsmithdev 命名空间中的 `DateTime` 类。同样，引用 `MathPHP\Algebra` 如果没有加初始反斜杠，PHP 会认为是指 `Mattsmithdev\MathPHP\Algebra`，即 `MathPHP` 被认为是 Mattsmithdev 的子命名空间，`Algebra` 被认为是该子命名空间中的类。写上以反斜杠开头的完全限定命名空间，可以确保 PHP 引擎正确理解你引用的类的命名空间。

另一方面，如果你*正在*引用当前命名空间中的类或子命名空间，则不应在类或子命名空间前加反斜杠。例如，如果你在 Mattsmithdev 命名空间中工作，`Shirt()` 被理解为指的是 Mattsmithdev 命名空间中的 Shirt 类，而 `SubNamespace\Example` 被理解为指的是 Mattsmithdev\SubNamespace\Example 类。

如果你只使用来自另一个命名空间的类一次，直接写该类的完全限定名（包括初始反斜杠）可能更合适。然而，如果你需要多次引用该类，最好在类声明的开始写一个 `use` 语句，这样更高效。在这种情况下，不需要加上初始反斜杠。随着你阅读和编写更多的 PHP 代码，你会经常看到在类声明的开头会有很多 `use` 语句，当代码使用来自其他命名空间的类时，正如 列表 20-4 中所示。

```
<?php

namespace App\Controller;

use App\Entity\ChessGame;
use App\Entity\Comment;
use App\Form\ChessGameType;
use App\Repository\ChessGameRepository;
use App\Repository\CommonRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\HttpFoundation\Session\SessionInterface;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

/**
* @Route("/chessgame")
*/
class ChessGameController extends AbstractController
{
    private $session;

    public function __construct(SessionInterface $session)
    {
--snip--
```

列表 20-4：包含多个 `use` 语句的类声明

这段代码片段是我在一个 PHP Symfony 网页框架的国际象棋项目中的一个类声明开始部分。它包含了多达 11 个 `use` 语句，引用了来自多个命名空间和子命名空间的类。`use` 语句帮助我们理清这些类的来源，但如果要同时处理这么多类仍然让你觉得有些不知所措，不用担心：接下来我们会讨论一个用于管理项目中所有类的工具。

### Composer

Composer 是一个命令行工具，用于支持面向对象的 PHP 编程。它帮助加载类和函数声明文件（包括你自己的文件以及第三方库的文件），并简化了使用不同命名空间中类的工作。它是一个重要且易于使用的工具，适用于专业的 Web 应用程序项目。在本节中，你将设置 Composer，并学习如何使用它来创建命令行别名、自动加载类声明文件以及管理项目的第三方依赖项。

> 注意

*SymfonyCasts 提供了一个很棒的免费视频，介绍了 Composer 工具，观看地址是* [`symfonycasts.com/screencast/composer`](https://symfonycasts.com/screencast/composer)*。*

#### 安装和测试 Composer

对于 Windows，Composer 提供了一个简单的安装程序，可以在 *[`getcomposer.org/Composer-Setup.exe`](https://getcomposer.org/Composer-Setup.exe)* 上找到。对于 macOS，你可以使用 Homebrew 安装 Composer，如 附录 A 中所述。对于 Linux，你需要执行几个命令行语句来下载并运行 *composer.php* 脚本。你可以在 *[`getcomposer.org/download/`](https://getcomposer.org/download/)* 上找到详细信息。如果你使用 Replit 跟随本书内容，请参见 附录 C，了解如何将 Composer 集成到你的项目中。

安装 Composer 后，通过打开一个新的命令行终端应用程序并输入 composer 来测试它。这将启动 Composer 工具，显示一个漂亮的 ASCII 艺术标志、版本号以及命令行选项的列表。

#### 创建 composer.json 配置文件

要在项目中使用 Composer 命令行工具，你需要创建一个 *composer.json* 文件，用于存放 Composer 所需的所有项目信息。（关于这种文件格式的回顾，请参见下面的“JSON 文件格式”框。）例如，*composer.json* 文件记录了你自己代码的命名空间和类的位置，以及项目所依赖的第三方包。此外，你还可以在 *composer.json* 文件中声明命令行 *别名*，这些快捷方式可以避免你在命令行输入冗长的命令。我们将从声明一个简单的别名开始探索 *composer.json* 文件。

*composer.json* 文本文件必须位于 PHP 项目目录的顶层，而不是像 *src* 或 *public* 这样的子文件夹内。继续本章的项目，创建 *composer.json* 文件，将其保存在项目目录的顶层，然后输入 Listing 20-5 中的内容。该代码创建了一个名为 hello 的别名，用于代替命令 `echo Hello World`。

```
{
    "scripts": {
        "hello": "echo Hello World"
    }
}
```

Listing 20-5: 在 composer.json 文件中声明一个别名

*composer.json* 的内容始终是一个 JSON 对象，因此它总是以一对大括号开始和结束。在对象内部，我们声明了一个名为 "scripts" 的属性，其值本身是一个对象。在该子对象中，我们声明了一个名为 "hello"（即我们的别名）的属性，值为 "echo Hello, world!"（将被快捷方式别名替换的代码）。

我们现在有了一个简单但有效的 *composer.json* 文件，告诉 Composer 有一个名为 "hello" 的命令行别名。要查看是否有效，请在终端输入 composer hello。你应该看到 "Hello, world!" 作为结果：

```
$ **composer hello**
> echo Hello, world!
Hello, world!
```

在这种情况下，我们写了更多的字符来声明别名，比在命令行中完整地写出 echo 语句还要多。然而，有时候这些脚本别名是很有用的。例如，下面是我在一些项目中使用的一个别名，用来输出报告，显示 *src* 文件夹中有多少代码需要修复，以符合 PHP 编程标准（尽管由于空间原因，别名在这里显示为两行，但在文件中会是单行）：

```
"reportfixsrc":"php php-cs-fixer.phar fix --level=psr2
--dry-run --diff ./src > ./tests/fixerReport.txt"
```

这个别名让我在命令行输入 composer reportfixsrc，而不是输入一个长的 PHP 命令来运行一个带有多个参数的 PHP 归档（*.phar*）文件。

正如你很快会看到的，Composer 能做的不仅仅是跟踪命令行别名。目前，我们已经成功为我们的项目创建了 *composer.json* 文件，这是使用这个强大工具的必要第一步。

#### 创建自动加载器

*自动加载器* 是一种系统，它会在需要时自动获取类声明文件，这样你就不需要自己将它们都加载到 *index.php* 文件中。随着面向对象 PHP 项目的规模和复杂性不断增长，涉及许多命名空间中的类，自动加载器变得非常有用。如果你必须在 *index.php* 前端控制器中为每个类写 require_once 语句，不仅会非常繁琐，而且很容易漏掉一个或两个，特别是当项目不断发展时。这会导致错误，并迫使你不断返回更新需要加载的文件列表。而自动加载器会为你处理这个过程，只要类是正确命名空间并按照自动加载规则正确定位的。

Composer 工具最强大的功能之一就是其自动加载器。它符合 PSR-4，这是 PHP 推荐的自动加载规则集。根据 PSR-4，你必须指定包含每个命名空间类的基础目录。例如，你可能想声明 Mattsmithdev 命名空间中的类可以在*src*目录中找到。此外，PSR-4 规定，任何子命名空间将被认为在声明的命名空间基础目录中有相应的子目录。例如，类 Mattsmithdev\Trigonometry\Angles.php 应位于 *src/Trigonometry* 目录下，类 Mattsmithdev\Utility\Security.php 应位于 *src/Utility* 目录下，依此类推。只要子目录的名称与子命名空间相同，你就不需要告诉自动加载器去哪里找到这些子命名空间的类。

要让 Composer 的自动加载器工作，需要三个步骤：

1.   在项目的 *composer.json* 文件中声明每个命名空间的基础目录。

2.   告诉 Composer 创建或更新其自动加载器脚本。

3.   在项目的 *public/index.php* 前端控制器的开头添加一个 require_once 语句来引入自动加载器脚本。这个单一的 require_once 语句替代了为每个单独类所写的多个 require_once 语句。

我们将演示如何设置 Composer 自动加载器以加载我们的 Mattsmithdev\Shirt 类。首先，列表 20-6 展示了在 *composer.json* 文件中需要写入的内容，用于声明 Mattsmithdev 命名空间中的类可以在 *src* 目录中找到。

```
{
    "autoload": {
        "psr-4": {
            "Mattsmithdev\\": "src"
        }
    }
}
```

列表 20-6：在 composer.json 文件中设置自动加载器

我们声明一个“autoload”属性，它的值是一个对象。在这个对象中，我们声明“psr-4”属性，它的值是另一个对象。它包含一个“Mattsmithdev\\" 属性，值为 "src"。这告诉 Composer，Mattsmithdev 命名空间中的类文件位于 *src* 目录中。注意命名空间后面的两个反斜杠字符 (\\)。这是 PSR-4 的要求。

对于我们在接下来的章节中将要处理的项目，*composer.json* 文件将基本与 列表 20-6 相同。每个项目之间唯一可能的不同是“psr-4”对象中声明的实际命名空间和位置。

> 注意

*Composer 自动加载器有一些额外的细节。如果你想了解更多，请参考 Composer 文档* [`getcomposer.org/doc/04-schema.md#psr-4`](https://getcomposer.org/doc/04-schema.md#psr-4)*。*

现在我们已经在 *composer.json* 文件中声明了 Mattsmithdev 命名空间的基础目录，我们可以告诉 Composer 为我们生成类的自动加载器。在当前项目的工作目录下输入以下命令：

```
$ **composer dump-autoload**
```

此命令将在项目中创建一个名为 *vendor* 的新文件夹（如果它还不存在的话），并在该文件夹内生成或更新多个文件。这个 *vendor* 文件夹是 Composer 用来存储项目工作文件的地方。你可以查看其内容，但不应更改其中的内容。你也可以随时删除这个文件夹并让 Composer 重新生成它，所以在备份项目时可以安全地省略这个文件夹。

在 *vendor* 目录中，你应该能看到一个 *vendor/autoload.php* 文件，以及一个包含多个脚本的 *vendor/composer* 文件夹，其中包括 *autoload_psr4.php*，它编码了我们的 PSR-4 合规声明。这个文件包含返回 Mattsmithdev 命名空间类的位置（*src/*）的语句。

现在我们已经生成了自动加载器，我们可以更新 *public/index.php* 脚本，仅引用这个 *autoload.php* 文件，无论我们在项目中需要引用多少个类。只要在 *composer.json* 文件中声明了命名空间的基础目录，并且通过 composer dump-autoload 命令更新了自动加载器，那么每当我们编写命名空间类的 use 语句时，PHP 引擎将加载其声明，准备供我们的代码使用。列表 20-7 展示了如何更新 *index.php*。

```
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Mattsmithdev\Shirt;

$shirt1 = new Shirt();
$shirt2 = new Shirt();

print "shirt 1 type = {$shirt1->getType()}";
```

列表 20-7：将 Composer 生成的自动加载脚本加载到 index.php 中

我们将 `require_once` 语句更改为从 *vendor* 目录中读取并执行 Composer 生成的自动加载器脚本。当你运行项目时，输出将与之前一样，但现在我们使用 Composer 自动加载器来自动读取 *src* 文件夹中 Mattsmithdev\Shirt 类的声明，而不是手动读取它。虽然对于我们这个单类项目来说，似乎差别不大，但对于包含多个类的项目，自动加载器可以节省大量时间。

#### 将第三方库添加到项目中

Composer 另一个强大的功能是能够将第三方库添加到项目中，并在 *composer.json* 文件中维护这些依赖关系的记录。成千上万的开源库可供使用，许多由经验丰富的软件开发者保持最新；在许多情况下，几分钟的搜索就能找到一个现成的库，能够完成你所需要的全部或大部分功能。维护良好的开源项目经过了充分的测试，并进行了重构以实现最佳实践，因此，合理使用第三方库可以减少你的工作量，同时帮助保持软件项目的质量。

如果没有 Composer 的包依赖功能，你需要从网站或 Git 仓库下载第三方库的代码，将其复制到适当的位置，比如 *lib* 文件夹，并更新 *composer.json* 文件，记录这些库类的命名空间和位置。相反，你只需告诉 Composer 你需要在项目中使用某个第三方库，它将为你完成所有繁重的工作。它会自动下载代码，将文件创建并复制到 *vendor* 下的适当子目录中，更新自动加载器，并在 *composer.json* 文件中记录依赖关系（及其版本）。你只需要知道你需要的包名和供应商。

要查看这如何工作，我们将告诉 Composer 将来自供应商 markrogoyski 的 math-php 包添加到我们的章节项目中。这是一个提供许多有用数学运算的优秀包。在当前项目工作目录的命令行中，输入以下内容：

```
$ **composer require markrogoyski/math-php**
```

这个 `require` 命令触发了 Composer 执行一系列操作。首先，如果你检查项目的 *vendor* 文件夹，你应该会看到 Composer 创建了一个新的子文件夹，名称与包的供应商名称匹配（在这个例子中是 *vendor/markrogoyski*）。在里面，你会找到一个 math-php 包的文件夹，包含所有必要的代码。

请记住，供应商名称（markrogoyski）和包名称（math-php）*不是*命名空间。它们只是 Composer 用来标识和定位要添加到项目中的第三方脚本的名称。Composer 会自动确定所有开源库类的命名空间，因此 *vendor/composer* 的内容会更新，包含所有已添加到 *vendor* 文件夹中的这些类。特别地，*autoload_psr4.php* 很可能会更新为命名空间第三方类的基本目录，因为大多数开源库使用 PSR-4 自动加载标准。同时，你需要阅读包的文档，了解第三方类的命名空间，以便在代码中正确引用它们。

`require` 命令还会提示 Composer 更新 *composer.json* 文件，添加有关 markrogoyski/math-php 包的信息。如果你查看该文件，现在应该能看到类似于清单 20-8 的内容。

```
{
    "autoload": {
        "psr-4": {
            "Mattsmithdev\\": "src"
        }
    },
    "require": {
 "markrogoyski/math-php": "².10"
    }
}
```

清单 20-8：composer.json 文件中记录的 math-php 库依赖

除了我们之前写的 "autoload" 属性外，*composer.json* 中的主对象现在还有一个 "require" 属性，这是 Composer 自动生成的。它的值是一个对象，列出了项目所需的所有包。在这种情况下，有一个 "markrogoyski/math-php" 条目。它的值 "².10" 表示包的可接受版本。插入符号（^）意味着我们愿意使用具有相同主版本号的新版本（例如 2.10.1、2.11、2.2 等），但不使用版本 3.*x* 或更高版本，因为那可能会破坏向后兼容性。

现在 Composer 已经将 markrogoyski/math-php 包集成到我们的项目中，我们可以尝试使用它。具体来说，我们将使用该包的 Average 类来计算一系列数字的平均值。请更新 *public/index.php* 的内容，使用清单 20-9 中的代码。

```
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use MathPHP\Statistics\Average;

$numbers = [13, 18, 13, 14, 13, 16, 14, 21, 13];
$numbersString = implode(', ', $numbers);
$mean = Average::mean($numbers);
print "average of [$numbersString] = $mean";
```

清单 20-9：计算一组整数的平均值

我们首先通过 `use` 语句告诉 PHP 引擎，Average 是指命名空间为 MathPHP\Statistics\Average 的类。注意，这个类的命名空间与我们之前在 `require` 语句中为 Composer 使用的供应商和包名称不同。接下来，我们声明一个 `$numbers` 数组，并使用内置的 `implode()` 函数将其转换为字符串，以便用户友好的输出。然后，我们调用 Average 类中的 `mean()` 方法，将结果存储在 `$mean` 中。接着，我们打印出数字列表和计算出的平均值。

请注意，我们调用了 `mean()` 方法，而不需要实际创建 Average 类的对象。这是因为 `mean()` 是一个 *静态方法*。我们将在第二十五章中详细探讨这一面向对象的概念。

### 在哪里可以找到 PHP 库

你可能会想知道 Composer 工具是如何知道在哪里下载 markrogoyski/math-php 包文件以供我们的项目使用的。答案是 Packagist (*[`packagist.org`](https://packagist.org)*)，这是一个用于发布开源 PHP 包的网站。供应商可以在该网站上注册（例如，我在 Packagist 上的用户名是 mattsmithdev），然后发布 PHP 包，供任何人通过 Composer 安装。

在发布包时，供应商必须提供包括包的公开可下载文件的 GitHub（或其他仓库）位置。例如，markrogoyski/math-php 包的 Packagist 页面列出了一个 GitHub 地址* [`github.com/markrogoyski/math-php`](https://github.com/markrogoyski/math-php)*。这就是 Composer 去下载包文件的位置。Packagist 的每个页面还列出了您需要的确切 require 命令，以便 Composer 将该包添加到您的项目中。

### 总结

在本章中，您学习了如何通过使用命名空间清晰地区分同名类。您还学习了如何使用强大的 Composer 命令行工具支持面向对象的 PHP 编程。学习如何维护*composer.json*文件，以及如何使用 Composer 自动加载类并将第三方库集成到项目中，将为您节省无数小时的繁琐手动工作。

### 练习

1.   开始一个新项目，并为一个命令创建 Composer 脚本别名，显示消息 Hello name，将 name 替换为您的名字。然后使用 Composer 执行该命令。

提示：在*composer.json*中声明一个脚本别名，然后使用 composer 别名在命令行中运行它。

2.   开始一个新项目，并创建一个*src/Product.php*文件，声明一个 Product 类，包含私有属性$id、$description 和$price，以及每个属性的公有 getter 和 setter 方法。声明该类位于 Mattsmithdev 命名空间下。将一个*composer.json*文件添加到项目的根文件夹，声明 Mattsmithdev 命名空间中的类可以在*src*目录中找到。然后使用 Composer 生成自动加载器。

编写一个*public/index.php*文件，执行以下操作：

a.   读取并执行*vendor/autoload.php*中的 Composer 自动加载器

b.   创建一个新的 Product 对象$p1，id 为 7，描述为'hammer'，价格为 9.99

c.   使用 var_dump()输出$p1 的详细信息

运行代码时，您应该看到类似以下内容：

```
object(Mattsmithdev\Product)#4 (3) {
  ["id":"Mattsmithdev\Product":private]=>
  int(7)
  ["description":"Mattsmithdev\Product":private]=>
  string(6) "hammer"
 ["price":"Mattsmithdev\Product":private]=>
  float(9.99)
}
```

3.   访问 Packagist 网站，打开* [`packagist.org`](https://packagist.org)*并搜索 mattsmithdev/faker-small-english 包。查看文档，然后使用 Composer 为新项目要求 mattsmithdev/faker-small-english 包。编写一个*public/index.php*文件，循环 10 次，从 FakerSmallEnglish 对象中显示 10 个随机名字。
