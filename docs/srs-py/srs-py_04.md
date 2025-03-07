## **4**

**处理时间戳和时区**

![image](../images/common01.jpg)

时区是复杂的。大多数人期望处理时区仅仅是从协调世界时（UTC）中加减几个小时的简单操作，从−12小时到+12小时。

然而，现实情况并非如此：时区并非逻辑上或可预测的。存在15分钟粒度的时区；有些国家每年两次更改时区；有些国家在夏季使用不同的时区（夏令时），并且开始日期不同；还有大量特殊的边缘情况。这些因素使得时区的历史既有趣又复杂，处理时区时也会带来挑战。所有这些特殊情况应当在处理时区时引起你的思考。

本章将概述为什么处理时区很棘手，并介绍如何在你的程序中最佳地处理时区。我们将讨论如何构建时间戳对象，如何以及为什么让它们具备时区感知功能，以及如何处理你可能遇到的边缘情况。

### **缺失时区的问题**

如果时间戳没有附带时区信息，它就没有任何有用的信息，因为没有时区，你无法推测你的应用程序到底指的是哪个时间点。因此，没有时区的时间戳无法进行比较；这就像在没有日期的情况下比较星期几——星期一是否在星期二之前，取决于它们属于哪个星期。因此，没有时区附带的时间戳应被视为无关紧要。

因此，你的应用程序绝不应处理没有时区的时间戳。相反，如果没有提供时区，它必须抛出错误，或者明确指出假设的默认时区——例如，选择UTC作为默认时区是常见的做法。

你还必须小心在*存储*时间戳之前进行任何形式的时区转换。假设一个用户在其本地时区创建了一个每周三上午10:00的重复事件，比如中欧时间（CET）。CET比协调世界时（UTC）快一个小时，因此，如果你将这个时间戳转换为UTC来存储，事件将被存储为每周三上午09:00。CET在夏季从UTC+01:00切换到UTC+02:00，因此，夏季期间，你的应用程序会计算出事件每周三上午11:00开始。你可以看到，这个程序很快就会变得冗余！

既然你已经理解了处理时区的常见问题，让我们深入了解我们最喜欢的编程语言。Python 提供了一个名为 datetime.datetime 的时间戳对象，可以存储精确到微秒的日期和时间。datetime.datetime 对象可以是时区*感知*的，这意味着它包含时区信息；也可以是时区*无感知*的，这意味着它不包含时区信息。不幸的是，datetime API 默认返回一个时区无感知的对象，正如你将在[清单 4-1](ch04.xhtml#ch4list1)中看到的那样。让我们看看如何构建一个默认的时间戳对象，然后如何纠正它以便使用时区。  

### **构建默认的 datetime 对象**  

要构建一个包含当前日期和时间的 datetime 对象，你可以使用 datetime.datetime.utcnow() 函数。此函数检索当前 UTC 时区的日期和时间，如[清单 4-1](ch04.xhtml#ch4list1)所示。要使用机器所在区域的时区的日期和时间构建相同的对象，你可以使用 datetime.datetime.now() 方法。[清单 4-1](ch04.xhtml#ch4list1) 同时获取 UTC 和我所在区域时区的时间和日期。  

>>> import datetime  

>>> datetime.datetime.utcnow()  

➊ datetime.datetime(2018, 6, 15, 13, 24, 48, 27631) >>> datetime.datetime.utcnow().tzinfo 是 None  

➋ 正确  

*清单 4-1：使用 datetime 获取一天中的时间*  

我们导入 datetime 库并定义一个使用 UTC 时区的 datetime 对象。这将返回一个 UTC 时间戳，其中的值分别是年份、月份、日期、小时、分钟、秒和微秒 ➊，如清单所示。我们可以通过检查 tzinfo 对象来判断该对象是否具有时区信息，在这里我们看到它没有时区信息 ➋。  

然后，我们使用 datetime.datetime.now() 方法创建 datetime 对象，以检索机器所在区域的默认时区的当前日期和时间：  

>>> datetime.datetime.now()  

➌ datetime.datetime(2018, 6, 15, 15, 24, 52, 276161)  

这个时间戳同样没有返回时区信息，我们可以通过缺少 tzinfo 字段 ➌ 来判断——如果时区信息存在，它会出现在输出的末尾，像 tzinfo=<UTC> 这样的形式。

datetime API 默认始终返回无感知的 datetime 对象，由于输出中无法看出时区信息，这些对象几乎没有用处。  

Flask 框架的创建者 Armin Ronacher 建议，应用程序应该始终假设 Python 中的无感知 datetime 对象是 UTC。然而，正如我们刚才看到的，这对于 datetime.datetime.now() 返回的对象并不适用。当你构建 datetime 对象时，我强烈建议你始终确保它们是时区感知的。这可以确保你能够直接比较对象，并检查它们是否返回了你需要的正确信息。让我们看看如何使用 tzinfo 对象创建时区感知的时间戳。  

**附加：从日期构建一个 datetime 对象**

你还可以通过传递你想要的日期各个组成部分的值，构建一个特定日期的 datetime 对象，如[示例 4-2](ch04.xhtml#ch4list2)所示。

>>> import datetime

>>> datetime.datetime(2018, 6, 19, 19, 54, 49)

datetime.datetime(2018, 6, 19, 19, 54, 49)

*示例 4-2：构建你自己的时间戳对象*

### **使用 dateutil 进行时区感知的时间戳**

已经有许多现有时区的数据库，由 IANA（互联网编号分配局）等中央机构维护，并随所有主要操作系统一起发布。因此，Python 开发者依赖于 dateutil 项目来获取 tzinfo 类，而不是在每个 Python 项目中手动创建自己的时区类并复制这些类。dateutil 项目提供了 Python 模块 tz，它使时区信息可以直接获取，且不需要太多的工作：tz 模块可以访问操作系统的时区信息，也可以分发和嵌入时区数据库，使其能直接从 Python 中访问。

你可以使用 pip 命令安装 dateutil，命令为 `pip install python-dateutil`。dateutil API 允许你根据时区名称获取 tzinfo 对象，如下所示：

>>> 来自 dateutil 的 tz

>>> tz.gettz("Europe/Paris")

tzfile('/usr/share/zoneinfo/Europe/Paris')

>>> tz.gettz("GMT+1")

tzstr('GMT+1')

dateutil.tz.gettz() 方法返回一个实现 tzinfo 接口的对象。此方法接受多种字符串格式作为参数，例如基于位置的时区（例如，“Europe/Paris”）或相对于 GMT 的时区。dateutil 时区对象可以直接用作 tzinfo 类，如[示例 4-3](ch04.xhtml#ch4list3)中所示。

>>> import datetime

>>> 来自 dateutil 的 tz

>>> now = datetime.datetime.now()

>>> 当前时间

datetime.datetime(2018, 10, 16, 19, 40, 18, 279100)

>>> tz = tz.gettz("Europe/Paris")

>>> now.replace(tzinfo=tz)

datetime.datetime(2018, 10, 16, 19, 40, 18, 279100, tzinfo=tzfile('/usr/share/zoneinfo/Europe/

巴黎'))

*示例 4-3：将 dateutil 对象作为 tzinfo 类使用*

只要你知道所需时区的名称，就可以获得一个与目标时区匹配的 tzinfo 对象。dateutil 模块可以访问操作系统管理的时区，如果由于某种原因无法获取该信息，它会回退到自己嵌入的时区列表。如果你需要访问这个嵌入的列表，可以通过 datetutil.zoneinfo 模块来实现：

>>> 来自 dateutil.zoneinfo 的 get_zonefile_instance

>>> zones = list(get_zonefile_instance().zones)

>>> sorted(zones)[:5] ['Africa/Abidjan', 'Africa/Accra', 'Africa/Addis_Ababa', 'Africa/Algiers', 'Africa/Asmara']

>>> len(zones)

592

在某些情况下，你的程序不知道自己运行在哪个时区，所以你需要自行确定。datetutil.tz.gettz() 函数如果不传入任何参数，将返回你计算机的本地时区，如 [示例 4-4](ch04.xhtml#ch4list4) 所示。

>>> from dateutil import tz

>>> import datetime

>>> now = datetime.datetime.now()

>>> localzone = tz.gettz()

>>> localzone

tzfile('/etc/localtime')

>>> localzone.tzname(datetime.datetime(2018, 10, 19))

'CEST'

>>> localzone.tzname(datetime.datetime(2018, 11, 19))

'CET'

*示例 4-4：获取你的本地时区*

如你所见，我们分别将两个日期传递给 localzone.tzname(datetime.datetime())，dateutil 能告诉我们一个日期在中欧夏令时（CEST），另一个在中欧标准时间（CET，非夏令时）。如果你传入当前日期，它会返回你所在的时区。

你可以在 tzinfo 类中使用来自 dateutil 库的对象，而无需在应用程序中自己实现它们。这使得将不带时区感知的日期时间对象转换为带时区感知的日期时间对象变得容易。

**实现你自己的时区类**

Python 中有一个类允许你自己实现时区类：datetime.tzinfo 类是一个抽象类，为实现表示时区的类提供了基础。如果你想实现一个类来表示时区，你需要使用这个类作为父类，并实现三个不同的方法：

+   utcoffset(dt)，必须返回时区相对于 UTC 的偏移量（以分钟为单位），东区为正值

+   dst(dt)，必须返回时区的夏令时调整（以分钟为单位，东区为正值）

+   tzname(dt)，必须返回时区名称的字符串

这三个方法会嵌入一个 tzinfo 对象，使你可以将任何时区感知的日期时间转换为另一个时区。

然而，如前所述，由于时区数据库的存在，自己实现这些时区类是不可行的。

### **序列化时区感知的日期时间对象**

你经常需要将一个日期时间对象从一个地方传输到另一个地方，而这些地方可能不是 Python 本地的。如今的典型情况是使用 HTTP REST API，API 必须将日期时间对象序列化并返回给客户端。Python 的原生方法 isoformat 可以用于将日期时间对象序列化成非 Python 本地格式，如 [示例 4-5](ch04.xhtml#ch4list5) 所示。

>>> import datetime

>>> from dateutil import tz

➊ >>> def utcnow():

return datetime.datetime.now(tz=tz.tzutc())

>>> utcnow()

➋ datetime.datetime(2018, 6, 15, 14, 45, 19, 182703, tzinfo=tzutc())

➌ >>> utcnow().isoformat()

'2018-06-15T14:45:21.982600+00:00'

*示例 4-5：序列化时区感知的日期时间对象*

我们定义了一个名为utcnow的新函数，并明确告诉它返回一个带有UTC时区的对象➊。如你所见，返回的对象现在包含时区信息➋。然后我们使用ISO格式对字符串进行格式化➌，确保时间戳也包含一些时区信息（如+00:00部分）。

你可以看到，我使用了isoformat()方法来格式化输出。我建议你始终使用ISO 8601格式来格式化你的日期时间输入和输出字符串，使用方法datetime.datetime.isoformat()，以返回一种包含时区信息的可读格式的时间戳。

你的ISO 8601格式字符串可以被转换为原生的datetime.datetime对象。iso8601模块仅提供一个函数parse_date，它完成了字符串解析并确定时间戳和时区值的所有繁重工作。iso8601模块不是Python的内置模块，所以你需要使用pip install iso8601来安装它。[清单4-6](ch04.xhtml#ch4list6)展示了如何使用ISO 8601解析时间戳。

>>> import iso8601

>>> import datetime

>>> from dateutil import tz

>>> now = datetime.datetime.utcnow()

>>> now.isoformat()

'2018-06-19T09:42:00.764337'

➊ >>> parsed = iso8601.parse_date(now.isoformat())

>>> parsed

datetime.datetime(2018, 6, 19, 9, 42, 0, 764337, tzinfo=<iso8601.Utc>)

>>> parsed == now.replace(tzinfo=tz.tzutc())

True

*清单4-6：使用iso8601模块解析ISO 8601格式的时间戳*

在[清单4-6](ch04.xhtml#ch4list6)中，iso8601模块被用来从字符串构建一个datetime对象。通过对包含ISO 8601格式时间戳的字符串调用iso8601.parse_date➊，该库能够返回一个datetime对象。由于该字符串没有包含任何时区信息，iso8601模块假定时区为UTC。如果时区包含正确的时区信息，iso8601模块会正确返回。

使用时区感知的datetime对象，并将ISO 8601作为其字符串表示的格式，是解决大多数时区问题的完美方案，确保不会出错，并在你的应用程序与外部世界之间建立良好的互操作性。

### **解决歧义时间**

有些情况下，一天中的时间可能是模糊的；例如在夏令时转换期间，某个“墙钟”时间可能一天内出现两次。dateutil库为我们提供了is_ambiguous方法，用以区分这样的时间戳。为了演示这个方法的作用，我们将在[清单4-7](ch04.xhtml#ch4list7)中创建一个模糊的时间戳。

>>> import dateutil.tz

>>> localtz = dateutil.tz.gettz("Europe/Paris")

>>> confusing = datetime.datetime(2017, 10, 29, 2, 30)

>>> localtz.is_ambiguous(confusing)

True

*清单4-7：一个在夏令时转换期间发生的模糊时间戳*

2017年10月30日夜间，巴黎从夏令时切换到冬令时。城市在凌晨3:00切换，时间会倒退到凌晨2:00。如果我们尝试使用该日期的2:30的时间戳，那么这个对象无法确定它是夏令时切换之后还是之前。

然而，可以通过使用 `fold` 属性来指定时间戳位于折叠的哪一侧，这个属性是 Python 3.6 中由 PEP 495（本地时间消歧义—*[https://www.python.org/dev/peps/pep-0495/](https://www.python.org/dev/peps/pep-0495/)*）加入到 `datetime` 对象中的。这个属性指示 `datetime` 位于折叠的哪一侧，如 [清单 4-8](ch04.xhtml#ch4list8) 所示。

>>> import dateutil.tz

>>> import datetime

>>> localtz = dateutil.tz.gettz("Europe/Paris")

>>> utc = dateutil.tz.tzutc()

>>> confusing = datetime.datetime(2017, 10, 29, 2, 30, tzinfo=localtz)

>>> confusing.replace(fold=0).astimezone(utc)

datetime.datetime(2017, 10, 29, 0, 30, tzinfo=tzutc())

>>> confusing.replace(fold=1).astimezone(utc)

datetime.datetime(2017, 10, 29, 1, 30, tzinfo=tzutc())

*清单 4-8：消除模糊的时间戳*

你只需要在非常罕见的情况下使用这个方法，因为模糊的时间戳只会出现在一个小的时间窗口内。坚持使用 UTC 是一个很好的解决方法，既能保持生活简单，又能避免遇到时区问题。然而，知道 `fold` 属性的存在以及 `dateutil` 能在这种情况下提供帮助也是很有益的。

### **总结**

在本章中，我们已经看到在时间戳中携带时区信息是多么重要。内建的 `datetime` 模块在这方面并不完整，但 `dateutil` 模块是一个很好的补充：它让我们能够获得与 `tzinfo` 兼容的对象，这些对象随时可以使用。`dateutil` 模块还帮助我们解决了诸如夏令时模糊性等微妙问题。

ISO 8601 标准格式是序列化和反序列化时间戳的绝佳选择，因为它在 Python 中已经现成可用，并且与任何其他编程语言兼容。
