## **处理时间戳和时区**

![image](img/common01.jpg)

时区很复杂。大多数人认为处理时区仅仅是从协调世界时（UTC）中加减几个小时，从−12 小时到+12 小时。

然而，现实情况并非如此：时区并不符合逻辑，也不可预测。有些时区的粒度是 15 分钟；一些国家每年会调整时区两次；还有一些国家在夏季使用一种特殊时区，叫做夏令时，且夏令时的开始日期各不相同；此外，还有很多特殊情况和边缘案例。这些使得时区的历史既有趣又复杂，处理起来充满挑战。所有这些特殊情况应该在处理时区时引起你的关注。

本章将概述为什么处理时区很棘手，以及如何在程序中最好地处理它们。我们将讨论如何构建时间戳对象，如何以及为什么使它们具备时区感知能力，以及如何处理可能遇到的边缘案例。

### **缺失时区的问题**

如果一个时间戳没有附带时区信息，它就没有任何有用的信息，因为没有时区，你无法推断出应用程序实际上所指的时间点。因此，缺少时区的时间戳不能进行比较；这就像没有日期的星期几比较——是否周一在周二之前，取决于它们属于哪个星期。没有附带时区的时间戳应该视为无关紧要。

因此，你的应用程序绝不应该处理没有时区的时间戳。如果没有提供时区，它必须抛出错误，或者明确假设默认时区——例如，选择 UTC 作为默认时区是一种常见做法。

你还必须小心在*存储*时间戳之前进行任何时区转换。假设某个用户在他们的本地时区（比如中欧时间 CET）每周三上午 10:00 创建一个重复事件。CET 比 UTC 快一个小时，如果你将该时间戳转换为 UTC 并存储，那么事件将被存储为每周三上午 09:00。CET 时区在夏季从 UTC+01:00 切换到 UTC+02:00，因此在夏季，应用程序会计算出每周三事件的开始时间为上午 11:00 CET。你可以看到，这个程序很快就变得冗余了！

现在你理解了处理时区的一般问题，让我们深入了解我们最喜欢的编程语言。Python 提供了一个名为`datetime.datetime`的时间戳对象，可以精确到微秒存储日期和时间。`datetime.datetime`对象可以是时区*感知*的，在这种情况下它会嵌入时区信息，或者是时区*无感知*的，在这种情况下它没有时区信息。不幸的是，`datetime` API 默认返回一个无感知时区的对象，正如你将在清单 4-1 中看到的那样。接下来我们来看看如何构建一个默认的时间戳对象，然后如何修正它，使其使用时区信息。

### **构建默认的 datetime 对象**

要构建一个带有当前日期和时间值的`datetime`对象，你可以使用`datetime.datetime.utcnow()`函数。这个函数会获取当前 UTC 时区的日期和时间，如清单 4-1 所示。要使用机器所在地区时区的日期和时间来构建相同的对象，你可以使用`datetime.datetime.now()`方法。清单 4-1 检索了 UTC 和我所在地区的时区的日期和时间。

```py
>>> import datetime
>>> datetime.datetime.utcnow()
  ➊ datetime.datetime(2018, 6, 15, 13, 24, 48, 27631) >>> datetime.datetime.utcnow().tzinfo is None
  ➋ True
```

*清单 4-1：使用 datetime 获取一天中的时间*

我们导入`datetime`库，并将`datetime`对象定义为使用 UTC 时区。这将返回一个 UTC 时间戳，其值依次为年份、月份、日期、小时、分钟、秒和微秒 ➊，如列表所示。我们可以通过检查`tzinfo`对象来检查这个对象是否包含时区信息，结果显示它没有 ➋。

然后，我们使用`datetime.datetime.now()`方法创建`datetime`对象，以检索当前机器区域的默认时区的日期和时间：

```py
>>> datetime.datetime.now()
  ➌ datetime.datetime(2018, 6, 15, 15, 24, 52, 276161)
```

这个时间戳同样没有返回任何时区信息，正如我们从缺少`tzinfo`字段可以看出 ➌—如果时区信息存在，它会以类似`tzinfo=<UTC>`的形式出现在输出的末尾。

`datetime` API 默认总是返回无时区感知的`datetime`对象，而且由于输出中无法判断时区信息，这些对象几乎没有什么用处。

Flask 框架的创始人 Armin Ronacher 建议，应用程序应始终假设 Python 中的`datetime`对象默认是 UTC 时区。然而，正如我们刚刚看到的，这对于由`datetime.datetime.now()`返回的对象并不适用。当你构建`datetime`对象时，我强烈建议你始终确保它们是时区感知的。这样可以确保你能够直接比较对象，并检查它们是否正确返回了所需的时区信息。接下来我们来看如何使用`tzinfo`对象创建时区感知的时间戳。

**附加内容：从日期构建 datetime 对象**

你还可以通过传递你想要的日期的各个组件的值来构建你自己的`datetime`对象，如列表 4-2 所示。

```py
>>> import datetime
>>> datetime.datetime(2018, 6, 19, 19, 54, 49)
datetime.datetime(2018, 6, 19, 19, 54, 49)
```

*列表 4-2：构建你自己的`timestamp`对象*

### **使用 dateutil 实现时区感知的时间戳**

目前已经有很多现有的时区数据库，由 IANA（互联网号码分配局）等中央机构维护，并且这些数据库已经包含在所有主要操作系统中。因此，Python 开发者不需要创建自己的时区类并在每个 Python 项目中手动复制它们，而是依赖`dateutil`项目来获取`tzinfo`类。`dateutil`项目提供了 Python 模块`tz`，它直接提供时区信息，无需太多操作：`tz`模块可以访问操作系统的时区信息，并且可以携带和嵌入时区数据库，使得这些信息可以直接在 Python 中访问。

你可以通过使用`pip`命令`pip install python-dateutil`来安装`dateutil`。`dateutil` API 允许你基于时区名称获取一个`tzinfo`对象，示例如下：

```py
>>> from dateutil import tz
>>> tz.gettz("Europe/Paris")
tzfile('/usr/share/zoneinfo/Europe/Paris')
>>> tz.gettz("GMT+1")
tzstr('GMT+1')
```

`dateutil.tz.gettz()`方法返回一个实现`tzinfo`接口的对象。这个方法接受多种字符串格式作为参数，比如基于位置的时区（例如，“Europe/Paris”）或相对于 GMT 的时区。`dateutil`时区对象可以直接作为`tzinfo`类使用，如列表 4-3 所示。

```py
>>> import datetime
>>> from dateutil import tz
>>> now = datetime.datetime.now()
>>> now
datetime.datetime(2018, 10, 16, 19, 40, 18, 279100)
>>> tz = tz.gettz("Europe/Paris")
>>> now.replace(tzinfo=tz)
datetime.datetime(2018, 10, 16, 19, 40, 18, 279100, tzinfo=tzfile('/usr/share/zoneinfo/Europe/
Paris'))
```

*列表 4-3：使用 dateutil 对象作为 tzinfo 类*

只要你知道所需时区的名称，就可以获得与目标时区匹配的`tzinfo`对象。`dateutil`模块可以访问操作系统管理的时区，如果由于某种原因这些信息不可用，它将回退到其嵌入的时区列表。如果你需要访问这个嵌入的列表，可以通过`datetutil.zoneinfo`模块进行访问：

```py
>>> from dateutil.zoneinfo import get_zonefile_instance
>>> zones = list(get_zonefile_instance().zones)
>>> sorted(zones)[:5] ['Africa/Abidjan', 'Africa/Accra', 'Africa/Addis_Ababa', 'Africa/Algiers', 'Africa/Asmara']
>>> len(zones)
592
```

在某些情况下，你的程序并不知道自己在哪个时区运行，因此你需要自己确定它。`datetutil.tz.gettz()`函数如果不传入参数，将返回你计算机的本地时区，如列表 4-4 所示。

```py
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
```

*列表 4-4：获取本地时区*

如你所见，我们分别将两个日期传递给`localzone.tzname(datetime.datetime())`，`dateutil`能够告诉我们一个是在中欧夏令时（CEST），另一个是在中欧时间（非夏令时）。如果你传入当前日期，你将得到你所在的当前时区。

你可以在`tzinfo`类中使用来自`dateutil`库的对象，而无需在应用程序中自己实现这些类。这使得将不具备时区信息的`datetime`对象转换为具备时区信息的`datetime`对象变得容易。

**实现你自己的时区类**

Python 中有一个类允许你自己实现时区类：`datetime.tzinfo` 类是一个抽象类，提供了实现表示时区的类的基础。如果你想实现一个表示时区的类，你需要使用它作为父类，并实现三个不同的方法：

+   `utcoffset(dt)`，必须返回该时区相对于 UTC 的偏移量，以分钟为单位，表示从 UTC 向东的偏移。

+   `dst(dt)`，必须返回该时区的夏令时调整，以 UTC 为基准，表示向东的分钟数。

+   `tzname(dt)`，必须返回时区名称的字符串形式。

这三种方法将嵌入一个 `tzinfo 对象`，允许你将任何带有时区信息的`datetime`转换为另一个时区。

然而，如前所述，由于存在时区数据库，自己实现这些时区类是不实际的。

### **序列化带时区信息的 datetime 对象**

你通常需要将一个`datetime`对象从一个地方传输到另一个地方，而这些地方可能不是 Python 原生的。现在的典型情况是通过 HTTP REST API，它必须将`datetime`对象序列化并返回给客户端。Python 原生方法 `isoformat` 可以用来将`datetime`对象序列化到非 Python 原生的地方，如 示例 4-5 所示。

```py
   >>> import datetime
   >>> from dateutil import tz
➊ >>> def utcnow():
       return datetime.datetime.now(tz=tz.tzutc())
   >>> utcnow()
➋ datetime.datetime(2018, 6, 15, 14, 45, 19, 182703, tzinfo=tzutc())
➌ >>> utcnow().isoformat()
   '2018-06-15T14:45:21.982600+00:00'
```

*示例 4-5：序列化带时区信息的 datetime 对象*

我们定义了一个新的函数 `utcnow` 并明确告诉它返回一个带有 UTC 时区的对象 ➊。如你所见，现在返回的对象包含时区信息 ➋。然后我们使用 ISO 格式 ➌ 格式化字符串，确保时间戳也包含一些时区信息（`+00:00` 部分）。

你可以看到我使用了 `isoformat()` 方法来格式化输出。我建议你始终使用 ISO 8601 格式来格式化你的`datetime`输入和输出字符串，使用方法 `datetime.datetime.isoformat()`，以返回带有时区信息的可读格式的时间戳。

你的 ISO 8601 格式字符串可以转换为原生的 `datetime.datetime` 对象。`iso8601` 模块只提供一个函数，`parse_date`，它完成了解析字符串并确定时间戳和时区值的所有繁重工作。`iso8601` 模块不是 Python 的内置模块，所以你需要使用 `pip install iso8601` 来安装它。示例 4-6 展示了如何使用 ISO 8601 解析时间戳。

```py
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
```

*示例 4-6：使用 iso8601 模块解析 ISO 8601 格式的时间戳*

在列表 4-6 中，使用`iso8601`模块从字符串构造`datetime`对象。通过对包含 ISO 8601 格式时间戳的字符串 ➊ 调用`iso8601.parse_date`，该库能够返回一个`datetime`对象。由于该字符串不包含任何时区信息，`iso8601`模块假设时区为 UTC。如果字符串包含正确的时区信息，`iso8601`模块会正确返回结果。

使用时区感知的`datetime`对象并将 ISO 8601 作为其字符串表示格式，是解决大多数时区问题的完美方案，确保不犯错误，并在你的应用程序与外部世界之间建立良好的互操作性。

### **解决模糊时间**

在某些情况下，时间可能是模糊的；例如在夏令时转换期间，同一“钟表”时间一天出现两次。`dateutil`库提供了`is_ambiguous`方法来区分此类时间戳。为了展示这一点，我们将在列表 4-7 中创建一个模糊的时间戳。

```py
>>> import dateutil.tz
>>> localtz = dateutil.tz.gettz("Europe/Paris")
>>> confusing = datetime.datetime(2017, 10, 29, 2, 30)
>>> localtz.is_ambiguous(confusing)
True
```

*列表 4-7：一个令人困惑的时间戳，发生在夏令时交替期间*

2017 年 10 月 30 日晚上，巴黎从夏令时切换到冬令时。切换发生在凌晨 3:00，此时时间回拨至 2:00 AM。如果我们尝试在该日期使用 2:30 的时间戳，这个对象无法确定它是在夏令时更改前还是后。

然而，通过使用`fold`属性，可以指定时间戳位于折叠的哪一侧，该属性通过 PEP 495（本地时间消歧义—*[`www.python.org/dev/peps/pep-0495/`](https://www.python.org/dev/peps/pep-0495/)*）在 Python 3.6 中被添加到`datetime`对象中。此属性指示 datetime 位于折叠的哪一侧，如列表 4-8 所示。

```py
>>> import dateutil.tz
>>> import datetime
>>> localtz = dateutil.tz.gettz("Europe/Paris")
>>> utc = dateutil.tz.tzutc()
>>> confusing = datetime.datetime(2017, 10, 29, 2, 30, tzinfo=localtz)
>>> confusing.replace(fold=0).astime zone(utc)
datetime.datetime(2017, 10, 29, 0, 30, tzinfo=tzutc())
>>> confusing.replace(fold=1).astime zone(utc)
datetime.datetime(2017, 10, 29, 1, 30, tzinfo=tzutc())
```

*列表 4-8：消除模糊的时间戳*

你只需要在非常少见的情况下使用此方法，因为模糊时间戳只会出现在一个小的时间窗口内。坚持使用 UTC 是一个很好的解决方法，可以保持生活简单，避免时区问题。然而，了解`fold`属性的存在以及`dateutil`在这种情况下能够提供帮助，还是很有用的。

### **总结**

在本章中，我们已经看到携带时区信息在时间戳中是多么重要。内置的`datetime`模块在这方面并不完备，但`dateutil`模块是一个很好的补充：它允许我们获取与`tzinfo`兼容的对象，这些对象已准备好使用。`dateutil`模块还帮助我们解决了如夏令时模糊性等细微问题。

ISO 8601 标准格式是序列化和反序列化时间戳的绝佳选择，因为它在 Python 中可以轻松使用，并与任何其他编程语言兼容。
