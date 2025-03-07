## 第六章：聚合数据

![](img/chapterart.png)

为了从数据中获得最大的决策价值，您通常需要生成数据的聚合。*聚合*是一个收集数据的过程，以便它可以以汇总形式呈现，通过小计、总计、平均数或其他统计数据分组。本章探讨了 pandas 内置的聚合技术，并讨论了如何使用它们来分析您的数据。

聚合是快速获取大型数据集概览的有效方式，它使您能够回答有关数据值的问题。例如，大型零售企业可能希望根据品牌确定产品的表现，或者查看不同地区的销售总额。网站所有者可能希望根据访问者数量确定网站上最具吸引力的资源。气候学家可能需要根据每年阳光明媚的天数，确定一个地区最阳光明媚的地方。

聚合可以通过将特定的数据值收集在一起并以总结的形式呈现，来回答像这样的问提。由于聚合是基于相关数据集群呈现信息，因此它首先意味着按一个或多个属性对数据进行分组。在大型零售商的情况下，这可能意味着按品牌分组数据，或者按地区和日期同时分组。

在接下来的示例中，您将看到如何在使用 pandas DataFrame 时，通过对每组行应用*聚合函数*，实现数据分组。聚合函数基于整个行组返回一个单一的结果行，从而为每个组形成一个单一的聚合总结行。

## 聚合数据

为了了解聚合如何工作，我们将创建一组包含在线户外时尚零售商销售数据的示例 DataFrame。数据将包括订单号和日期等值；每个订单中所购商品的详细信息，如价格和数量；完成每个订单的员工；以及公司履行订单的仓库位置。在实际应用中，这些数据很可能存储在数据库中，您可以从 Python 代码中访问，如第五章所述。为了简化，我们将从元组列表中将数据加载到 DataFrame 中。您可以从本书的 GitHub 仓库下载这些元组列表。

让我们从一些示例订单开始。名为`orders`的列表包含多个元组，每个元组代表一个订单。每个元组有三个字段：订单号、日期和完成订单的员工 ID，分别如下：

```py
orders = [
 (9423517, '2022-02-04', 9001),
 (4626232, '2022-02-04', 9003),
 (9423534, '2022-02-04', 9001),
 (9423679, '2022-02-05', 9002),
 (4626377, '2022-02-05', 9003),
 (4626412, '2022-02-05', 9004),
 (9423783, '2022-02-06', 9002),
 (4626490, '2022-02-06', 9004)
]
```

导入 pandas 并按如下方式将列表加载到 DataFrame 中：

```py
import pandas as pd
df_orders = pd.DataFrame(orders, columns =['OrderNo', 'Date', 'Empno'])
```

订单详情（也称为*订单行*）通常存储在另一个数据容器中。在本例中，我们有一个名为`details`的元组列表，您将把它加载到另一个 DataFrame 中。每个元组代表订单中的一行，包含订单号、商品名称、品牌、价格和数量等字段：

```py
details = [
 (9423517, 'Jeans', 'Rip Curl', 87.0, 1),
 (9423517, 'Jacket', 'The North Face', 112.0, 1),
 (4626232, 'Socks', 'Vans', 15.0, 1),
 (4626232, 'Jeans', 'Quiksilver', 82.0, 1),
 (9423534, 'Socks', 'DC', 10.0, 2),
 (9423534, 'Socks', 'Quiksilver', 12.0, 2),
 (9423679, 'T-shirt', 'Patagonia', 35.0, 1),
 (4626377, 'Hoody', 'Animal', 44.0, 1),
 (4626377, 'Cargo Shorts', 'Animal', 38.0, 1),
 (4626412, 'Shirt', 'Volcom', 78.0, 1),
 (9423783, 'Boxer Shorts', 'Superdry', 30.0, 2),
 (9423783, 'Shorts', 'Globe', 26.0, 1),
 (4626490, 'Cargo Shorts', 'Billabong', 54.0, 1),
 (4626490, 'Sweater', 'Dickies', 56.0, 1)
]
# converting the list into a DataFrame
df_details = pd.DataFrame(details, columns =['OrderNo', 'Item', 'Brand', 'Price', 'Quantity'])
```

我们将在第三个 DataFrame 中存储公司员工的信息。通过另一个名为 `emps` 的元组列表创建该 DataFrame，其中包含员工编号、姓名和位置：

```py
emps = [
 (9001, 'Jeff Russell', 'LA'),
 (9002, 'Jane Boorman', 'San Francisco'),
 (9003, 'Tom Heints', 'NYC'),
 (9004, 'Maya Silver', 'Philadelphia')
]

df_emps = pd.DataFrame(emps, columns =['Empno', 'Empname', 'Location'])
```

最后，我们将每个仓库的城市和地区存储在一个名为 `locations` 的元组列表中，这将被存储在第四个 DataFrame 中：

```py
locations = [
 ('LA', 'West'),
 ('San Francisco', 'West'),
 ('NYC', 'East'),
 ('Philadelphia', 'East')
]

df_locations = pd.DataFrame(locations, columns =['Location', 'Region'])
```

现在你已经将数据加载到 DataFrame 中，你可以以多种方式进行汇总，从而回答关于业务状况的各种问题。例如，你可能想查看不同地区的销售表现，并按日期生成小计。为此，你首先需要将相关数据合并到一个单一的 DataFrame 中。然后，你可以对数据进行分组，并应用聚合函数。

## 合并 DataFrame

你经常需要从多个不同的容器中收集数据，才能获得完成所需聚合所需的所有信息。我们的示例也不例外。即使是表示订单的数据，也分布在两个不同的 DataFrame 中：`df_orders` 和 `df_details`。你的目标是生成按地区和日期汇总的销售总额。你需要合并哪些 DataFrame？每个 DataFrame 中应该包含哪些列？

由于你需要汇总销售额，因此必须包含 `df_details` 中的 `Price` 和 `Quantity` 列。同时，还需要包含 `df_orders` 中的 `Date` 列和 `df_locations` 中的 `Region` 列。这意味着你需要连接以下 DataFrame：`df_orders`、`df_details` 和 `df_locations`。

`df_orders` 和 `df_details` DataFrame 可以通过 pandas 的 `merge()` 方法直接连接，代码如下：

```py
df_sales = df_orders.merge(df_details)
```

你可以根据 `OrderNo` 列连接 DataFrame。无需明确指定这一点，因为 `OrderNo` 在两个 DataFrame 中都存在，因此默认选择它。新合并的 DataFrame 现在包含每个订单行的一条记录，如同 `df_details` 中一样，但加入了来自 `df_orders` 的相应记录的信息。要查看合并后的 DataFrame 中的记录，你可以直接打印 DataFrame：

```py
print(df_sales)
```

`df_sales` 的内容将如下所示：

```py
 OrderNo        Date  Empno          Item           Brand  Price  Quantity
0   9423517  2022-02-04   9001         Jeans        Rip Curl   87.0         1
1   9423517  2022-02-04   9001        Jacket  The North Face  112.0         1
2   4626232  2022-02-04   9003         Socks            Vans   15.0         1
3   4626232  2022-02-04   9003         Jeans      Quiksilver   82.0         1
4   9423534  2022-02-04   9001         Socks              DC   10.0         2
5   9423534  2022-02-04   9001         Socks      Quiksilver   12.0         2
6   9423679  2022-02-05   9002       T-shirt       Patagonia   35.0         1
7   4626377  2022-02-05   9003         Hoody          Animal   44.0         1
8   4626377  2022-02-05   9003  Cargo Shorts          Animal   38.0         1
9   4626412  2022-02-05   9004         Shirt          Volcom   78.0         1
10  9423783  2022-02-06   9002  Boxer Shorts        Superdry   30.0         2
11  9423783  2022-02-06   9002        Shorts           Globe   26.0         1
12  4626490  2022-02-06   9004  Cargo Shorts       Billabong   54.0         1
13  4626490  2022-02-06   9004       Sweater         Dickies   56.0         1
```

如你在 `Quantity` 列中所见，一条订单中可能包含多个相同的商品。因此，你需要将 `Price` 和 `Quantity` 字段的值相乘，以计算订单行的总额。你可以将这一乘法结果存储在 DataFrame 的新字段中，如下所示：

```py
df_sales['Total'] = df_sales['Price'] * df_sales['Quantity']
```

这会在 DataFrame 中添加一个 `Total` 列，除了原有的七列之外。现在，你可以选择删除不需要的列，以便生成按地区和日期汇总的销售额。在这个阶段，你只需要保留 `Date`、`Total` 和 `Empno` 列。前两列显然对你的计算至关重要。稍后我们将讨论 `Empno` 的必要性。

要将 DataFrame 筛选到必要的列，传递一个列名列表给 DataFrame 的 `[]` 操作符，如下所示：

```py
df_sales = df_sales[['Date','Empno','Total']]
```

现在，你需要将刚刚创建的 `df_sales` DataFrame 与 `df_regions` DataFrame 连接起来。然而，由于它们没有共同的列，不能直接连接。因此，你必须通过 `df_emps` DataFrame 来连接它们，因为 `df_emps` 和 `df_sales` 共享一个列，`df_emps` 和 `df_regions` 也共享一个列。具体来说，`df_sales` 和 `df_emps` 可以通过 `Empno` 列连接，这也是我们在 `df_sales` 中保留该列的原因，而 `df_emps` 和 `df_locations` 可以通过 `Location` 列连接。使用 `merge()` 方法实现这些连接：

```py
df_sales_emps = df_sales.merge(df_emps)
df_result = df_sales_emps.merge(df_locations)
```

打印出的 `df_result` DataFrame 如下所示：

```py
 Date  Empno  Total       Empname       Location Region
0   2022-02-04   9001   87.0  Jeff Russell             LA   West
1   2022-02-04   9001  112.0  Jeff Russell             LA   West
2   2022-02-04   9001   20.0  Jeff Russell             LA   West
3   2022-02-04   9001   24.0  Jeff Russell             LA   West
4   2022-02-04   9003   15.0    Tom Heints            NYC   East
5   2022-02-04   9003   82.0    Tom Heints            NYC   East
6   2022-02-05   9003   44.0    Tom Heints            NYC   East
7   2022-02-05   9003   38.0    Tom Heints            NYC   East
8   2022-02-05   9002   35.0  Jane Boorman  San Francisco   West
9   2022-02-06   9002   60.0  Jane Boorman  San Francisco   West
10  2022-02-06   9002   26.0  Jane Boorman  San Francisco   West
11  2022-02-05   9004   78.0   Maya Silver   Philadelphia   East
12  2022-02-06   9004   54.0   Maya Silver   Philadelphia   East
13  2022-02-06   9004   56.0   Maya Silver   Philadelphia   East
```

再次，你可能想要移除不必要的列，只保留实际需要的列。这次你可以去掉 `Empno`、`Empname` 和 `Location` 列，保留 `Date`、`Region` 和 `Total`：

```py
df_result = df_result[['Date','Region','Total']]
```

现在 `df_result` 的内容如下所示：

```py
 Date Region  Total
0   2022-02-04   West   87.0
1   2022-02-04   West  112.0
2   2022-02-04   West   20.0
3   2022-02-04   West   24.0
4   2022-02-04   East   15.0
5   2022-02-04   East   82.0
6   2022-02-05   East   44.0
7   2022-02-05   East   38.0
8   2022-02-05   West   35.0
9   2022-02-06   West   60.0
10  2022-02-06   West   26.0
11  2022-02-05   East   78.0
12  2022-02-06   East   54.0
13  2022-02-06   East   56.0
```

在不包含不必要列的情况下，`df_result` DataFrame 现在已经理想地格式化，用于按地区和日期聚合销售数据。

## 数据分组与聚合

要对数据进行聚合计算，必须先将其排序为相关的组。pandas 的 `groupby()` 函数会将 DataFrame 的数据拆分为具有相同列值的子集。对于我们的例子，可以使用 `groupby()` 按日期和地区对 `df_result` DataFrame 进行分组。然后，可以对每个组应用 pandas 的 `sum()` 聚合函数。你可以在一行代码中同时完成这两个操作：

```py
df_date_region = df_result.groupby(['Date','Region']).sum()
```

第一次分组是基于 `Date` 列。然后，在每个日期内，按照 `Region` 分组。`groupby()` 函数返回一个对象，然后你可以对其应用 `sum()` 聚合函数。这个函数会对数值型列的值进行求和。在这个特定的例子中，`sum()` 仅应用于 `Total` 列，因为这是 DataFrame 中唯一的数值型列。（如果 DataFrame 中有其他数值型列，聚合函数也会应用到这些列。）最终，你将得到如下的 DataFrame：

```py
 Total
Date       Region       
2022-02-04 East     97.0
           West    243.0
2022-02-05 East    160.0
           West     35.0
2022-02-06 East    110.0
           West     86.0
```

`Date` 和 `Region` 是新 DataFrame 的索引列。它们一起构成了一个*层次化索引*，也称为*多级索引*，或者简称为*MultiIndex*。

MultiIndex 使得在 DataFrame 的 2D 结构中，可以通过使用多列来唯一标识每一行，从而处理具有任意维度的数据。在我们的例子中，`df_date_region` DataFrame 可以视为一个 3D 数据集，三个轴分别对应日期、地区和汇总值（每个轴代表对应的维度），如 表 6-1 所示。

表 6-1：`df_date_region` DataFrame 的三个维度

| **轴** | **坐标** |
| --- | --- |
| 日期 | 2022-02-04, 2022-02-05, 2022-02-06 |
| 地区 | 西部, 东部 |
| 聚合 | 总计 |

我们的数据框的 MultiIndex 使我们能够编写查询，遍历数据框的维度，按日期、地区或两者的总和进行访问。我们将能够唯一标识数据框中的每一行，并访问不同数据组内的选定聚合值。

### 按 MultiIndex 查看特定的汇总信息

在数据框中查看特定类别的信息是常见的需求。例如，在您刚创建的 `df_date_region` 数据框中，您可能只需要获取特定日期的聚合销售数据，或者同时获取某个地区和某个日期的销售数据。您可以使用数据框的索引（或 MultiIndex）来查找所需的聚合数据。

为了更好地理解如何使用 MultiIndex，查看每个 MultiIndex 值在 Python 中的表示方式会有所帮助。您可以使用 `df_date_region` 数据框的 `index` 属性来实现这一点：

```py
print(df_date_region.index)
```

`index` 属性返回数据框的所有索引值或行标签，无论该数据框是使用简单索引还是 MultiIndex。以下是 `df_date_region` 的 MultiIndex 值：

```py
MultiIndex([('2022-02-04', 'East'),
            ('2022-02-04', 'West'),
            ('2022-02-05', 'East'),
            ('2022-02-05', 'West'),
            ('2022-02-06', 'East'),
            ('2022-02-06', 'West')],
           names=['Date', 'Region'])
```

如您所见，每个 MultiIndex 值都是一个元组，您可以使用它来访问 `Total` 字段中的相应销售数据。考虑到这一点，以下是如何访问特定日期和地区的总销售数据：

```py
df_date_region❶ [df_date_region.index.isin(❷ [('2022-02-05', 'West')])]
```

您将表示所需 MultiIndex 的元组放入 `[]` 运算符 ❷ 中，并将其传递给 pandas 的 `index.isin()` 方法。该方法要求传递的参数必须是可迭代对象（如列表或元组）、Series、DataFrame 或字典，这就是为什么您需要将所需的 MultiIndex 放在方括号中的原因。该方法返回一个布尔数组，指示数据框中每个索引值是否与您指定的索引值匹配：如果匹配则为 `True`，否则为 `False`。在这个特定示例中，`isin()` 方法生成了数组 `[False, False, False, True, False, False]`，表示第四个索引值是匹配的。

然后，您将布尔数组传递给 `df_date_region` 数据框中的 `[]` 运算符 ❶，从而选择相应的销售数据，如下所示：

```py
 Total
Date       Region       
2022-02-05 West     35.0
```

您不仅限于从数据框中检索单行数据。您可以将多个索引值传递给 `index.isin()`，以获取一组对应的销售数据，如下所示：

```py
df_date_region[df_date_region.index.isin([('2022-02-05', 'East'), ('2022-02-05', 'West')])]
```

这将从 `df_date_region` 中检索以下两行数据：

```py
 Total
Date       Region       
2022-02-05 East    160.0
           West     35.0
```

尽管此特定示例使用了两个相邻的索引，实际上您可以按任意顺序将任何索引传递给 `index.isin()`，如下所示：

```py
df_date_region[df_date_region.index.isin([('2022-02-06', 'East'),
               ('2022-02-04', 'East'), ('2022-02-05', 'West')])]
```

检索到的行集合将如下所示：

```py
 Total
Date       Region       
2022-02-04 East     97.0
2022-02-05 West     35.0
2022-02-06 East    110.0
```

请注意，检索到的记录顺序与数据框中的记录顺序匹配，而不是您指定索引的顺序。

### 切片聚合值的范围

就像你可以使用切片从列表中获取一系列值一样，你也可以使用它从 DataFrame 中提取一系列聚合值。你可以在`df_date_region` DataFrame 中通过提供两个元组来实现，这两个元组指定了切片范围的开始和结束位置的 MultiIndex 键。以下示例获取了所有地区在`2022-02-04`到`2022-02-05`日期范围内的聚合值范围。你只需将开始和结束的 MultiIndex 键放在方括号内，用冒号分隔：

```py
df_date_region[('2022-02-04', 'East'):('2022-02-05', 'West')]
```

结果，你将得到以下几行的 DataFrame：

```py
 Total
Date       Region       
2022-02-04 East     97.0
           West    243.0
2022-02-05 East    160.0
           West     35.0
```

由于在这个特定示例中，你正在获取指定日期范围内所有地区的销售数据，你可以省略地区名称，仅传递日期：

```py
df_date_region['2022-02-04':'2022-02-05']
```

这将给你与前一个示例完全相同的结果。

### 在聚合级别中进行切片

你可能希望在不同层次的层级索引中对聚合结果进行切片。在我们的示例中，最高的聚合级别是`Date`级别，在该级别下我们有`Region`级别。假设你需要获取特定日期范围内的销售数据，并选择`Region`级别的所有内容。你可以通过结合使用 Python 的`slice()`函数和 DataFrame 的`loc`属性来实现这一点，如下所示：

```py
df_date_region.loc[(slice('2022-02-05', '2022-02-06'), slice(None)), :]
```

这里你使用了两次`slice()`。第一次，`slice()`定义了`Date`（最高聚合级别）的切片范围，生成了指定开始和结束日期的`slice`对象。第二次调用`slice()`时，你针对的是`Region`级别（下一个较低级别）。通过指定`None`，你选择了`Region`级别的所有内容。在`loc`属性的`[]`操作符中，你还包括了一个逗号，后跟一个冒号（`:`）。这种语法表示你正在使用行标签而不是列标签。

结果集如下：

```py
 Total
Date       Region       
2022-02-05 East    160.0
 West     35.0
2022-02-06 East    110.0
           West     86.0
```

在下一个示例中，你将`slice(None)`替换为`slice('East')`，从而将检索的销售数据减少到仅包含`East`的行，这些行是在指定日期范围内选取的：

```py
df_date_region.loc[(slice('2022-02-05', '2022-02-06'), slice('East')), :]
```

这将检索以下几行：

```py
 Total
Date       Region       
2022-02-05 East    160.0
2022-02-06 East    110.0
```

你可以为`Region`级别指定一个范围，而不是单个值，就像你为`Date`级别指定一个范围一样。然而，在这个特定示例中，该范围只能从`'East'`开始，并以`'West'`结束，表示为`slice('East','West')`。由于这是最大可能的范围，调用`slice('East','West')`将等同于调用`slice(None)`。

### 添加总计

在汇总销售数据时，你可能最终需要计算总计，或是所有其他销售值总和，并将其添加到 DataFrame 中。在我们的示例中，由于所有总计都在同一个 DataFrame `df_date_region` 中，你可以使用 pandas 的`sum()`方法来计算所有地区和所有日期的销售总额。该方法会计算指定轴上的值的总和，如下所示：

```py
ps = df_date_region.sum(axis = 0)
print(ps)
```

在这里，`sum()`返回一个 pandas Series，它对`df_date_region` DataFrame 中的`Total`列进行求和。记住，在调用`sum()`时无需指定`Total`列，因为它会自动应用于任何数值型数据。`ps` Series 的内容如下所示：

```py
Total    731.0
dtype: float64
```

要将新创建的 Series 附加到`df_date_region` DataFrame，首先需要为其命名。这个名称将作为 DataFrame 中总计行的索引。由于`df_date_region` DataFrame 中的索引键是元组，因此你也需要为 Series 命名时使用元组：

```py
ps.name=('All','All')
```

元组中的第一个`'All'`与索引键的`Date`组件相关，而第二个`'All'`与`Region`索引键组件相关。现在，你可以将 Series 附加到 DataFrame：

```py
df_date_region_total = df_date_region.append(ps)
```

如果你打印出新创建的 DataFrame，它的内容将如下所示：

```py
 Total
Date       Region       
2022-02-04 East     97.0
           West    243.0
2022-02-05 East    160.0
           West     35.0
2022-02-06 East    110.0
           West     86.0
All        All     731.0
```

你可以通过其索引访问总计行，就像访问 DataFrame 中的任何其他行一样。在这里，你将表示该行索引的元组传递给`index.isin()`方法，如前所述：

```py
df_date_region_total[df_date_region_total.index.isin([('All', 'All')])]
```

这将为你提供总计行：

```py
 Total
Date Region       
All  All     731.0
```

### 添加小计

除了计算总计，你可能还希望为 DataFrame 中的每个日期添加小计，使得结果 DataFrame 如下所示：

```py
 Total
Date       Region       
2022-02-04 East     97.0
           West    243.0
           All     340.0
2022-02-05 East    160.0
           West     35.0
           All     195.0
2022-02-06 East    110.0
           West     86.0
           All     196.0
All        All     731.0
```

生成这个 DataFrame 需要几个步骤。首先，你按索引的`Date`级别对 DataFrame 进行分组。然后你遍历生成的 GroupBy 对象，访问每个日期及其对应的行集合（称为*子框架*），该集合包含该日期的区域和总计信息。接着，你选择并附加每个子框架到一个空的 DataFrame，并附加一个对应的小计行。步骤如下：

```py
❶ df_totals = pd.DataFrame()
for date, date_df in ❷ df_date_region.groupby(level=0):
❸ df_totals = df_totals.append(date_df)
❹ ps = date_df.sum(axis = 0)
❺ ps.name=(date,'All')
❻ df_totals = df_totals.append(ps)
```

你首先创建一个空的 DataFrame，`df_totals`，用来接收最终的数据 ❶。然后你创建一个 GroupBy 对象 ❷，将`df_date_region` DataFrame 按其索引的顶层层次（即`Date`列）进行分组，并进入一个`for`循环，遍历 GroupBy 对象。在每次迭代中，你将得到一个日期及其对应的子框架。你将子框架附加到`df_totals` DataFrame ❸，然后创建一个包含子框架行总和的 Series 作为小计行 ❹。接着，你用对应的日期和`'All'`为 Series 命名，表示所有区域 ❺，然后将 Series 附加到`df_totals` DataFrame ❻。

最后，你将总计行附加到 DataFrame，如下所示：

```py
df_totals = df_totals.append(df_date_region_total.loc[('All','All')])
```

结果，你将得到一个包含每个日期的销售总和以及所有日期总和的 DataFrame。

## 选择某组中的所有行

除了帮助聚合，`groupby()`函数还可以帮助你选择属于某个特定组的所有行。为此，`groupby()`返回的对象提供了`get_group()`方法。操作方法如下：

```py
group = df_result.groupby(['Date','Region'])
group.get_group(('2022-02-04','West'))
```

你将 `df_result` DataFrame 按照 `Date` 和 `Region` 分组，将列名作为列表传递给 `groupby()`，就像你之前做的那样。然后，你在结果的 GroupBy 对象上调用 `get_group()` 方法，传递包含所需索引的元组。这将返回以下 DataFrame：

```py
 Date Region  Total
0  2022-02-04   West   87.0
1  2022-02-04   West  112.0
2  2022-02-04   West   20.0
3  2022-02-04   West   24.0
```

如你所见，结果集并不是一个聚合结果。相反，它包含了与指定日期和地区相关的所有订单行。

## 总结

你在本章中学到，聚合是收集数据并以总结格式表达它的过程。通常，这个过程涉及将数据分成多个组，然后为每个组计算总结。本章中的示例展示了如何聚合包含在 pandas DataFrame 中的数据，使用 DataFrame 方法和属性，如 `merge()`、`groupby()`、`sum()`、`index` 和 `loc`。你学习了如何利用 DataFrame 的层次化索引（MultiIndex），以建模在聚合数据中的多级关系。你还学会了如何使用 MultiIndex 查看和切片聚合数据。
