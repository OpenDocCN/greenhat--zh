# 第十四章：不安全的反序列化

![](img/chapterart.png)

*不安全的反序列化*漏洞发生在应用程序在没有适当预防措施的情况下反序列化程序对象时。攻击者可以操控序列化对象，从而改变程序的行为。

不安全的反序列化漏洞一直让我着迷。它们很难发现和利用，因为它们的表现形式取决于所使用的编程语言和库，因而看起来各不相同。这些漏洞还需要深厚的技术理解和创造力才能加以利用。尽管它们可能很难被发现，但值得付出努力。无数的研究报告描述了研究人员如何利用这些漏洞实现 RCE（远程代码执行），并攻破 Google、Facebook 等公司的关键资产。

在本章中，我将讲解什么是不安全的反序列化，PHP 和 Java 应用程序中如何出现不安全的反序列化漏洞，以及如何利用这些漏洞。

## 机制

*序列化*是将编程语言中的某些数据转换为一种格式的过程，以便将其保存到数据库或通过网络传输。*反序列化*指的是相反的过程，即程序从文件或网络读取序列化的对象，并将其转换回对象。

这非常有用，因为在编程语言中，一些对象在通过网络传输或存储到数据库时很容易发生损坏。序列化和反序列化允许编程语言在不同的计算环境中重建相同的程序对象。许多编程语言都支持对象的序列化和反序列化，包括 Java、PHP、Python 和 Ruby。

开发人员常常信任用户提供的序列化数据，因为这些数据对用户来说难以读取或根本无法读取。这种信任假设正是攻击者可以利用的漏洞。*不安全的反序列化*是一种漏洞类型，发生在攻击者能够操控序列化对象，从而导致程序出现意外后果时。这可能导致身份验证绕过，甚至是 RCE。例如，如果一个应用程序从用户那里获取一个序列化的对象，并利用其中的数据来确定谁已经登录，恶意用户可能会篡改该对象，从而冒充其他人登录。如果应用程序使用了不安全的反序列化操作，恶意用户甚至可能将代码片段嵌入对象中，并在反序列化过程中执行这些代码。

理解不安全的反序列化的最佳方法是学习不同编程语言如何实现序列化和反序列化。由于这些过程在每种语言中都有不同的表现，我们将探讨这种漏洞在 PHP 和 Java 中的表现形式。在继续之前，如果你想测试本章中的示例代码，你需要安装 PHP 和 Java。

你可以按照 PHP 手册页面上的说明来安装 PHP（[`www.php.net/manual/en/install.php`](https://www.php.net/manual/en/install.php)）。然后，你可以通过命令行运行 `php` `YOUR_PHP_SCRIPT.php` 来运行 PHP 脚本。或者，你可以使用像 ExtendsClass 这样的在线 PHP 测试工具（[`extendsclass.com/php.html`](https://extendsclass.com/php.html)）来测试示例脚本。搜索 *online PHP tester* 以获取更多选项。请注意，并非所有在线 PHP 测试工具都支持序列化和反序列化，因此请确保选择一个支持这些功能的工具。

大多数计算机应该已经安装了 Java。如果你在命令行中运行 `java -version` 并看到返回的 Java 版本号，则无需再次安装 Java。否则，你可以在 [`java.com/en/download/help/download_options.html`](https://java.com/en/download/help/download_options.html) 查找安装 Java 的说明。你还可以使用在线 Java 编译器来测试代码；Tutorials Point 提供了一个在线编译器，地址为 [`www.tutorialspoint.com/compile_java_online.php`](https://www.tutorialspoint.com/compile_java_online.php)。

### PHP

尽管在现实中大多数反序列化漏洞是由 Java 中的不安全反序列化引起的，但我发现 PHP 的反序列化漏洞也非常常见。在我的一个研究项目中，我研究了 HackerOne 上公开披露的反序列化漏洞，发现所有公开披露的反序列化漏洞中，有一半是由 PHP 中的不安全反序列化引起的。我还发现，大多数反序列化漏洞被归类为高影响或严重影响漏洞；令人难以置信的是，大多数漏洞可以用来在目标服务器上执行任意代码。

当 PHP 中发生不安全的反序列化漏洞时，我们有时称之为 *PHP 对象注入漏洞*。要理解 PHP 对象注入漏洞，首先需要了解 PHP 如何序列化和反序列化对象。

当应用程序需要存储一个 PHP 对象或通过网络传输它时，它会调用 PHP 函数 `serialize()` 来打包该对象。当应用程序需要使用这些数据时，它会调用 `unserialize()` 来解包并获取底层对象。

例如，这段代码片段将会序列化名为 `user` 的对象：

```
<?php1 class User{ public $username; public $status; }2 $user = new User;3 $user->username = 'vickie';4 $user->status = 'not admin';5 echo serialize($user);
?>
```

这段 PHP 代码声明了一个名为 `User` 的类。每个 `User` 对象将包含一个 `$username` 和一个 `$status` 属性 1。然后，它创建了一个名为 `$user` 的新 `User` 对象 2。接着，它将 `$user` 的 `$username` 属性设置为 `'vickie'` 3，并将其 `$status` 属性设置为 `'not admin'` 4。然后，它序列化 `$user` 对象，并打印出表示序列化对象的字符串 5。

将这段代码保存为名为 *serialize_test.php* 的文件，并使用命令 `php serialize_test.php` 运行它。你应该能够得到表示 `user` 对象的序列化字符串：

```
O:4:"User":2:{s:8:"username";s:6:"vickie";s:6:"status";s:9:"not admin";}
```

让我们分解一下这个序列化字符串。PHP 序列化字符串的基本结构是 `数据类型``:``数据`。关于数据类型，`b`表示布尔值，`i`表示整数，`d`表示浮动数，`s`表示字符串，`a`表示数组，`O`表示某个类的对象实例。其中一些类型后面跟着关于数据的附加信息，如下所述：

```
b:`THE_BOOLEAN`;
i:`THE_INTEGER`;
d:`THE_FLOAT`;
s:`LENGTH_OF_STRING`:"`ACTUAL_STRING`";
a:`NUMBER_OF_ELEMENTS`:{`ELEMENTS`}
O:`LENGTH_OF_NAME`:"`CLASS_NAME`":`NUMBER_OF_PROPERTIES`:{`PROPERTIES`}
```

以这个参考为指南，我们可以看到我们的序列化字符串表示一个 `User` 类的对象。它有两个属性。第一个属性名为 `username`，值为 `vickie`。第二个属性名为 `status`，值为 `not admin`。这些名字和值都是字符串。

当你准备再次操作对象时，可以使用 `unserialize()` 来反序列化字符串：

```
<?php1 class User{ public $username; public $status; } $user = new User; $user->username = 'vickie'; $user->status = 'not admin'; $serialized_string = serialize($user);2 $unserialized_data = unserialize($serialized_string);3 var_dump($unserialized_data); var_dump($unserialized_data["status"]);
?>
```

这段代码的前几行创建了一个用户对象，将其序列化，并将序列化字符串存储到一个名为 `$serialized_string` 1 的变量中。然后，它反序列化该字符串并将恢复的对象存储到变量 `$unserialized_data` 2 中。`var_dump()` PHP 函数显示变量的值。最后两行显示了反序列化后的对象 `$unserialized_data` 及其状态属性 3。

大多数面向对象的编程语言都有类似的接口来序列化和反序列化程序对象，但它们的序列化对象格式不同。一些编程语言还允许开发人员序列化成其他标准格式，如 JSON 和 YAML。

#### 控制变量值

你可能已经注意到这里有些不对劲。如果序列化的对象没有加密或签名，任何人都可以创建一个`User`对象吗？答案是可以！这是一个常见的、不安全的反序列化危害应用程序的方式。

利用 PHP 对象注入漏洞的一种可能方式是通过操作对象中的变量。一些应用程序只是传递一个序列化对象作为认证方法，而没有对其进行加密或签名，认为仅凭序列化就能阻止用户篡改值。如果是这种情况，你可以修改序列化字符串中编码的值：

```
<?php class User{ public $username; public $status; } $user = new User; $user->username = 'vickie';1 $user->status = 'admin'; echo serialize($user);
?>
```

在我们之前创建的`User`对象的这个示例中，你可以通过修改你的 PHP 脚本 1，将 `status` 改为 `admin`。然后，你可以在代理中拦截传出的请求，并将新的对象替换掉旧的对象，看看应用程序是否授予你管理员权限。

你还可以直接修改你的序列化字符串：

```
O:4:"User":2:{s:8:"username";s:6:"vickie";s:6:"status";s:9:"**admin**";}
```

如果你直接篡改序列化字符串，记得也要更改字符串的长度标记，因为你的 `status` 字符串的长度已经发生变化：

```
O:4:"User":2:{s:8:"username";s:6:"vickie";s:6:"status";**s:5**:"admin";}
```

#### unserialize() 内部机制

为了理解 `unserialize()` 如何导致 RCE（远程代码执行），让我们来看看 PHP 是如何创建和销毁对象的。

*PHP 魔术方法*是具有特殊属性的 PHP 方法名称。如果序列化对象的类实现了任何具有魔术名称的方法，这些方法将具有魔术属性，例如在执行的某些时刻自动运行，或者在满足某些条件时执行。这些魔术方法中有两个是`__wakeup()`和`__destruct()`。

`__wakeup()`方法在实例化时使用，当程序在内存中创建类的实例时，这就是`unserialize()`的作用；它接受序列化字符串，指定该对象的类和属性，并使用这些数据创建原始序列化对象的副本。然后，它会搜索`__wakeup()`方法并执行其中的代码。`__wakeup()`方法通常用于重建对象可能具有的任何资源，重新建立序列化过程中丢失的数据库连接，并执行其他重新初始化任务。在 PHP 对象注入攻击中，它通常很有用，因为它提供了一个方便的入口点，进入服务器的数据库或程序中的其他功能。

然后程序在对象上进行操作，并使用它执行其他操作。当没有对反序列化对象的引用时，程序会调用`__destruct()`函数来清理该对象。这个方法通常包含在利用中的有用代码。例如，如果`__destruct()`方法包含删除并清理与对象相关的文件的代码，攻击者可能通过控制传入这些函数的输入，破坏文件系统的完整性。

#### 实现 RCE

当你控制一个传入`unserialize()`的序列化对象时，你就能控制创建对象的属性。你也可能能够控制传入自动执行的方法（如`__wakeup()`或`__destruct()`）的值。如果你能做到这一点，就有可能实现 RCE。

例如，考虑以下漏洞代码示例，来自[`www.owasp.org/index.php/PHP_Object_Injection`](https://www.owasp.org/index.php/PHP_Object_Injection)：

```
1 class Example2 { private $hook; function __construct(){ // some PHP code... } function __wakeup(){ 2 if (isset($this->hook)) eval($this->hook); } } // some PHP code...3 $user_data = unserialize($_COOKIE['data']);
```

代码声明了一个名为`Example2`的类。它有一个`$hook`属性和两个方法：`__construct()`和`__wakeup()`。1. 如果`$hook`不为空，`__wakeup()`函数会将存储在`$hook`中的字符串作为 PHP 代码执行。2. PHP `eval()`函数接受一个字符串并将其内容作为 PHP 代码运行。然后，程序在用户提供的名为`data`的 cookie 上运行`unserialize()`。

在这里，你可以实现 RCE，因为代码将用户提供的对象传递给`unserialize()`，并且有一个对象类`Example2`，它具有一个魔术方法，该方法在实例化对象时会自动对用户提供的输入运行`eval()`。

要利用这个 RCE，你需要将你的`data` cookie 设置为一个序列化的`Example2`对象，并将`hook`属性设置为你想执行的任何 PHP 代码。你可以通过以下代码片段生成序列化对象：

```
class Example2
{ private $hook = "phpinfo();";
}
print 1 urlencode(serialize(new Example2));
```

在打印对象之前，我们需要对其进行 URL 编码 1，因为我们将通过 cookie 注入该对象。将该代码生成的字符串传递给`data` cookie 会导致服务器执行 PHP 代码`phpinfo();`，该代码输出服务器上 PHP 配置的信息。phpinfo()函数通常用作概念验证函数，在漏洞报告中运行，以证明 PHP 命令注入成功。以下是此攻击在目标服务器上发生的详细过程：

1.  序列化后的`Example2`对象作为`data` cookie 传递到程序中。

1.  程序对`data` cookie 调用`unserialize()`方法。

1.  因为`data` cookie 是一个序列化的`Example2`对象，`unserialize()`会实例化一个新的`Example2`对象。

1.  `unserialize()`函数看到`Example2`类中实现了`__wakeup()`，于是调用了`__wakeup()`方法。

1.  `__wakeup()`函数查找对象的`$hook`属性，如果它不是`NULL`，则执行`eval($hook)`。

1.  `$hook`属性不是`NULL`，因为它被设置为`phpinfo();`，因此`eval("phpinfo();")`会被执行。

1.  你通过执行放置在`data` cookie 中的任意 PHP 代码，成功实现了远程代码执行（RCE）。

#### 使用其他魔术方法

到目前为止，我们提到了魔术方法`__wakeup()`和`__destruct()`。实际上，当试图利用`unserialize()`漏洞时，有四个魔术方法特别有用：`__wakeup()`、`__destruct()`、`__toString()`和`__call()`。

与`__wakeup()`和`__destruct()`不同，这两个方法只要对象被创建就会执行，而`__toString()`方法仅在对象被当作字符串处理时才会被调用。它允许类决定当其中一个对象被当作字符串时该如何反应。例如，它可以决定在对象传递给`echo()`或`print()`函数时显示什么内容。你将在“使用 POP 链”一节（第 238 页）看到如何在反序列化攻击中使用此方法的例子。

当调用一个未定义的方法时，程序会调用`__call()`方法。例如，调用`$object->undefined($args)`将变成`$object->__call('undefined', $args)`。同样，这个魔术方法的可利用性差异很大，取决于它是如何实现的。有时，攻击者可以在应用程序代码中存在错误或允许用户定义要调用的方法名时利用该魔术方法。

通常，你会发现这四个魔术方法在利用中最为有用，但还有许多其他方法存在。如果这里提到的方法不可利用，可能值得查看该类中其他魔术方法的实现，看看是否能从那里发起攻击。阅读更多关于 PHP 魔术方法的信息，参考[`www.php.net/manual/en/language.oop5.magic.php`](https://www.php.net/manual/en/language.oop5.magic.php)。

#### 使用 POP 链

到目前为止，你已经知道，当攻击者控制传递给 `unserialize()` 的序列化对象时，他们可以控制创建的对象的属性。这使他们有机会通过选择传递给魔术方法（如 `__wakeup()`）的值来劫持应用程序的流程。

这个漏洞有时能工作。但这个方法有个问题：如果类声明的魔术方法在利用时没有任何有用的代码怎么办？例如，有时用于对象注入的类只有几个方法，而且这些方法中没有任何代码注入的机会。那么不安全的反序列化就变得没用了，漏洞就失效了，对吧？

即使在这种情况下，我们还有另一种实现 RCE（远程代码执行）的方法：POP 链。*面向属性编程（POP）链* 是一种漏洞类型，其名称来源于攻击者控制反序列化对象的所有属性这一事实。POP 链通过将多个代码片段（称为 *gadgets*）串联起来，来实现攻击者的最终目标。这些 *gadgets* 是从代码库中借用的代码片段。POP 链将魔术方法作为其初始 *gadget*，攻击者可以利用这些方法来调用其他 *gadgets*。

如果这看起来很抽象，请参考以下示例应用程序代码，摘自 [`owasp.org/www-community/vulnerabilities/PHP_Object_Injection`](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)：

```
class Example
{1 private $obj; function __construct() { // some PHP code... } function __wakeup() { 2 if (isset($this->obj)) return $this->obj->evaluate(); }
}
class CodeSnippet
{3 private $code; 4 function evaluate() { eval($this->code); }
}
// some PHP code...5 $user_data = unserialize($_POST['data']);
// some PHP code...
```

在这个应用程序中，代码定义了两个类：`Example` 和 `CodeSnippet`。`Example` 类有一个名为 `obj` 的属性 1，当一个 `Example` 对象被反序列化时，它的 `__wakeup()` 函数会被调用，而该函数会调用 `obj` 的 `evaluate()` 方法 2。

`CodeSnippet` 类有一个名为 `code` 的属性，包含要执行的代码字符串 3，并且有一个 `evaluate()` 方法 4，该方法在 `code` 字符串上调用 `eval()`。

在代码的另一部分，程序接受来自用户的 POST 参数 `data` 并对其调用 `unserialize()` 5。

由于最后一行包含一个不安全的反序列化漏洞，攻击者可以使用以下代码生成一个序列化对象：

```
class CodeSnippet
{ private $code = "phpinfo();";
}
class Example
{ private $obj; function __construct() { $this->obj = new CodeSnippet; }
}
print urlencode(serialize(new Example));
```

这个代码片段定义了一个名为 `CodeSnippet` 的类，并将它的 `code` 属性设置为 `phpinfo();`。然后定义了一个名为 `Example` 的类，并在实例化时将其 `obj` 属性设置为一个新的 `CodeSnippet` 实例。最后，创建一个 `Example` 实例，进行序列化，并对序列化字符串进行 URL 编码。攻击者可以将生成的字符串传递给 POST 参数 `data`。

请注意，攻击者的序列化对象使用了应用程序源代码中其他地方找到的类和属性名称。因此，当程序接收到伪造的 `data` 字符串时，它将执行以下操作。

首先，它会反序列化对象并创建一个`Example`实例。然后，由于`Example`实现了`__wakeup()`方法，程序会调用`__wakeup()`并看到`obj`属性被设置为一个`CodeSnippet`实例。最后，它会调用`obj`的`evaluate()`方法，该方法执行`eval("phpinfo();")`，因为攻击者将`code`属性设置为`phpinfo()`。攻击者能够执行他们选择的任何 PHP 代码。

POP 链通过将应用程序代码库中找到的代码链接和重用来实现 RCE。让我们来看一个如何使用 POP 链来实现 SQL 注入的例子。这个例子也来自[`owasp.org/www-community/vulnerabilities/PHP_Object_Injection`](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)。

假设应用程序在代码的某个地方定义了一个名为`Example3`的类，并从 POST 参数`data`中反序列化未经清理的用户输入：

```
class Example3
{ protected $obj; function __construct() { // some PHP code... }1 function __toString() { if (isset($this->obj)) return $this->obj->getValue(); }
}
// some PHP code...
$user_data = unserialize($_POST['data']);
// some PHP code...
```

请注意，`Example3`实现了`__toString()`魔术方法 1。在这种情况下，当`Example3`实例被当作字符串处理时，它将返回在其`$obj`属性上运行的`getValue()`方法的结果。

假设在应用程序的某个地方，代码定义了一个名为`SQL_Row_Value`的类。它有一个名为`getValue()`的方法，该方法执行 SQL 查询。SQL 查询从`SQL_Row_Value`实例的`$_table`属性中获取输入：

```
class SQL_Row_Value
{ private $_table; // some PHP code... function getValue($id) { $sql = "SELECT * FROM {$this->_table} WHERE id = " . (int)$id; $result = mysql_query($sql, $DBFactory::getConnection()); $row = mysql_fetch_assoc($result);
return $row['value']; }
}
```

攻击者可以通过控制`Example3`中的`$obj`来实现 SQL 注入。以下代码将创建一个`Example3`实例，将`$obj`设置为一个`SQL_Row_Value`实例，并将`$_table`设置为字符串`"SQL Injection"`：

```
class SQL_Row_Value
{ private $_table = "SQL Injection";
}
class Example3
{ protected $obj; function __construct() { $this->obj = new SQL_Row_Value; }
}
print urlencode(serialize(new Example3));
```

因此，每当攻击者的`Example3`实例被当作字符串处理时，它的`$obj`的`get_Value()`方法就会被执行。这意味着`SQL_Row_Value`的`get_Value()`方法将会执行，并且`$_table`字符串被设置为`"SQL Injection"`。

攻击者已经实现了有限的 SQL 注入，因为他们可以控制传递到 SQL 查询中的字符串`SELECT * FROM {$this->_table} WHERE id = " . (int)$id;`。

POP 链类似于*基于返回的编程（**ROP)**攻击，这是一种在二进制利用中使用的有趣技术。你可以在维基百科上阅读更多相关内容，链接：[`en.wikipedia.org/wiki/Return-oriented_programming`](https://en.wikipedia.org/wiki/Return-oriented_programming)*。

### Java

现在你了解了 PHP 中不安全反序列化的工作原理，让我们来探索另一种易受此类漏洞影响的编程语言：Java。Java 应用程序容易受到不安全反序列化漏洞的影响，因为许多 Java 应用程序处理序列化的对象。为了了解如何利用 Java 中的反序列化漏洞，我们来看看 Java 中序列化和反序列化是如何工作的。

要使 Java 对象可序列化，它们的类必须实现`java.io.Serializable`接口。这些类还实现了特殊方法`writeObject()`和`readObject()`，分别处理该类对象的序列化和反序列化。让我们看一个例子。将以下代码存储在名为*SerializeTest.java*的文件中：

```
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.Serializable;
import java.io.IOException;1 class User implements Serializable{2 public String username;
}
public class SerializeTest{ public static void main(String args[]) throws Exception{ 3 User newUser = new User(); 4 newUser.username = "vickie"; FileOutputStream fos = new FileOutputStream("object.ser"); ObjectOutputStream os = new ObjectOutputStream(fos); 5 os.writeObject(newUser); os.close(); FileInputStream is = new FileInputStream("object.ser"); ObjectInputStream ois = new ObjectInputStream(is); 6 User storedUser = (User)ois.readObject(); System.out.println(storedUser.username); ois.close(); }
}
```

然后，在存储文件的目录中，运行以下命令。这些命令将编译程序并执行代码：

```
$ javac SerializeTest.java
$ java SerializeTest
```

你应该看到字符串`vickie`作为输出打印出来。我们来详细分析一下这个程序。首先，我们定义一个名为`User`的类，它实现了`Serializable`接口 1。只有实现了`Serializable`接口的类才能被序列化和反序列化。`User`类有一个`username`属性，用于存储用户的用户名 2。

然后，我们创建一个新的`User`对象 3，并将其用户名设置为字符串`"vickie"` 4。我们将`newUser`的序列化版本写入并存储到文件*object.ser* 5 中。最后，我们从文件中读取对象，反序列化它，并打印出用户的用户名 6。

要利用 Java 应用程序中的不安全反序列化漏洞，我们首先必须找到一个入口点，通过该入口点插入恶意的序列化对象。在 Java 应用程序中，序列化对象通常用于在 HTTP 头、参数或 Cookies 中传输数据。

Java 序列化对象不像 PHP 序列化字符串那样易于阅读。它们通常包含不可打印的字符。但它们确实有几个标识符，可以帮助你识别它们并找到潜在的攻击入口：

+   在十六进制中以`AC ED 00 05`开头，或在 Base64 中以`rO0`开头。 (你可能会在 HTTP 请求中看到这些，作为 Cookies 或参数。)

+   HTTP 消息的`Content-Type`头被设置为`application/x-java-serialized-object`。

由于 Java 序列化对象包含很多特殊字符，因此在传输之前常常会对其进行编码，因此还需要注意这些标识符的不同编码版本。

在发现一个用户提供的序列化对象后，你可以尝试的第一件事是通过篡改对象中存储的信息来操控程序逻辑。例如，如果 Java 对象作为访问控制的 Cookie 使用，你可以尝试更改其中的用户名、角色名和其他身份标识符，将其重新序列化后再返回给应用程序。你还可以尝试篡改对象中任何类型的值，例如文件路径、文件说明符或控制流值，看看是否能够改变程序的执行流程。

有时，当代码没有限制应用程序允许反序列化的类时，它可以反序列化任何它可以访问的序列化类。这意味着攻击者可以创建任何类的对象。潜在攻击者可以通过构造合适的类对象，导致任意命令的执行，从而实现远程代码执行（RCE）。

#### 实现 RCE

从 Java 反序列化漏洞到远程代码执行（RCE）的路径可能会非常复杂。为了获得代码执行权限，你通常需要使用一系列工具链来达成最终的代码执行方法。这与使用 PHP 中的 POP 链利用反序列化漏洞的方式类似，因此我们在这里不会重新描述整个过程。在 Java 应用程序中，你会在应用程序加载的库中找到这些工具链。通过使用应用程序范围内的工具链，创建方法调用链，最终实现远程代码执行。

查找并串联工具链以构建利用载荷可能需要消耗大量时间。你还受限于应用程序中可用的类，这可能限制你的利用方式。为了节省时间，尝试通过使用流行库中的工具链来创建利用链，例如 Apache Commons-Collections、Spring 框架、Apache Groovy 和 Apache Commons FileUpload。你会在网上找到很多此类资源。

#### 使用 Ysoserial 自动化利用

Ysoserial ([`github.com/frohoff/ysoserial/`](https://github.com/frohoff/ysoserial/)) 是一个工具，可以用来生成利用 Java 不安全反序列化漏洞的有效载荷，节省大量时间，避免你自己开发工具链。

Ysoserial 使用在常见 Java 库中发现的工具链来构造利用对象。使用 Ysoserial，你可以通过一个命令创建使用指定库中的工具链的恶意 Java 序列化对象：

```
$ java -jar ysoserial.jar `gadget_chain command_to_execute`
```

例如，要创建一个利用 Commons-Collections 库中的工具链在目标主机上打开计算器的有效载荷，可以执行此命令：

```
$ java -jar ysoserial.jar CommonsCollections1 calc.exe
```

Ysoserial 生成的工具链都能让你在系统上执行命令。该程序接受你指定的命令，并生成一个序列化对象来执行该命令。

有时，使用哪个库来构建你的工具链似乎很明显，但通常这是一个试错过程，因为你必须发现目标应用程序实现了哪些易受攻击的库。这时，良好的侦察工作将会帮助你。

你可以在 GitHub 上找到更多关于利用 Java 反序列化的资源，链接为 [`github.com/GrrrDog/Java-Deserialization-Cheat-Sheet/`](https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet/)。

## 防御措施

防御反序列化漏洞是非常困难的。保护应用程序免受这些漏洞攻击的最佳方法根据所使用的编程语言、库和序列化格式的不同而有很大差异。没有一种通用的解决方案。

你应该确保不会反序列化任何未经适当检查的用户输入污染的数据。如果必须反序列化，使用允许列表来限制反序列化仅限于少数允许的类。

你还可以使用简单的数据类型，如字符串和数组，而不是在传输时需要被序列化的对象。此外，为了防止序列化的 Cookie 被篡改，你可以在服务器上跟踪会话状态，而不是依赖用户输入的会话信息。最后，你应当关注补丁更新，确保你的依赖项是最新的，以避免通过第三方代码引入反序列化漏洞。

一些开发人员尝试通过识别常见的易受攻击的类并将其从应用程序中移除来缓解反序列化漏洞。这确实限制了攻击者在“工具链”中可以使用的有效工具。然而，这并不是一种可靠的防护方式。限制工具可以是一个很好的防御层，但黑客是富有创意的，他们总能在其他库中找到更多的工具，并通过创新的方式实现相同的结果。重要的是要解决这个漏洞的根本原因：应用程序不安全地反序列化用户数据。

OWASP 反序列化备忘单是一个学习如何防止特定技术中的反序列化缺陷的优秀资源：[`cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html`](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)。

## 寻找不安全的反序列化

进行源代码审查是检测反序列化漏洞最可靠的方法。从本章的示例中可以看到，寻找不安全的反序列化漏洞的最快方法是通过在源代码中搜索反序列化函数，并检查是否有用户输入被不加限制地传递到这些函数中。例如，在 PHP 应用程序中，查找 `unserialize()`，在 Java 应用程序中，查找 `readObject()`。在 Python 和 Ruby 应用程序中，分别查找 `pickle.loads()` 和 `Marshall.load()` 函数。

但许多漏洞奖励猎人已经能够在没有检查任何代码的情况下发现反序列化漏洞。以下是一些可以在没有源代码访问权限的情况下用来寻找不安全反序列化的策略。

首先，密切关注传递到应用程序中的大块数据。例如，base64 字符串 `Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6InZpY2tpZSI7czo2OiJzdGF0dXMiO3M6OToibm90IGFkbWluIjt9` 是 PHP 序列化字符串 `O:4:"User":2:{s:8:"username";s:6:"vickie";s:6:"status";s:9:"not admin";}` 的 base64 编码版本。

这是一个序列化的 Python 对象的 base64 表示形式，类名为 `Person`，其中 `name` 属性的值为 `vickie`：`gASVLgAAAAAAAACMCF9fbWFpbl9flIwGUGVyc29ulJOUKYGUfZSMBG5hbWWUjAZWaWNraWWUc2Iu`。

这些大数据块可能是表示对象注入机会的序列化对象。如果数据被编码了，尝试解码它。大多数传入 Web 应用程序的编码数据是用 base64 编码的。例如，如前所述，Java 序列化对象通常以十六进制字符`AC ED 00 05`或 base64 编码的`rO0`开头。还要注意 HTTP 请求或响应的`Content-Type`头。例如，设置为`application/x-java-serialized-object`的`Content-Type`表示应用程序通过 Java 序列化对象传递信息。

另外，你可以通过寻找容易受到反序列化缺陷影响的特性开始。查找可能需要反序列化用户提供的对象的功能，比如数据库输入、身份验证令牌和 HTML 表单参数。

一旦你找到了用户提供的序列化对象，你需要确定它是哪种类型的序列化对象。它是 PHP 对象、Python 对象、Ruby 对象还是 Java 对象？阅读每种编程语言的文档，熟悉其序列化对象的结构。

最后，尝试使用我提到的技术篡改对象。如果应用程序将序列化对象用作身份验证机制，尝试篡改字段，看是否能够以其他用户身份登录。你还可以尝试通过小工具链实现远程代码执行或 SQL 注入。

## 升级攻击

本章已经描述了不安全的反序列化漏洞如何通常导致远程代码执行，进而赋予攻击者广泛的能力来影响应用程序。因此，反序列化漏洞是宝贵且具有影响力的漏洞。即使无法实现远程代码执行（RCE），你也许可以绕过身份验证，或以其他方式干扰应用程序的逻辑流程。

然而，当不安全的反序列化依赖于一个晦涩的入口点，或者需要一定级别的应用权限才能利用，或者如果该漏洞函数对未认证用户不可用时，不安全的反序列化影响可能会受到限制。

在提升反序列化缺陷时，要考虑赏金计划的范围和规则。反序列化漏洞可能是危险的，因此在尝试操作程序逻辑或执行任意代码时，确保不要对目标应用程序造成损害。阅读第十八章，获取有关如何为远程代码执行（RCE）创建安全 PoC 的建议。

## 找到你的第一个不安全的反序列化漏洞！

现在是时候深入并找到你的第一个不安全的反序列化漏洞了。按照我们讲解的步骤去寻找：

1.  如果你能够访问应用程序的源代码，搜索源代码中接受用户输入的反序列化函数。

1.  如果你无法访问源代码，可以查找传入应用程序的大块数据。这些数据可能表示被编码的序列化对象。

1.  或者，寻找可能需要反序列化由用户提供的对象的功能，例如数据库输入、身份验证令牌和 HTML 表单参数。

1.  如果序列化的对象包含有关用户身份的信息，尝试篡改找到的序列化对象，看看能否实现身份验证绕过。

1.  查看是否可以将该漏洞升级为 SQL 注入或远程代码执行。务必小心，不要对目标应用程序或服务器造成损害。

1.  撰写你的第一个不安全反序列化报告！
