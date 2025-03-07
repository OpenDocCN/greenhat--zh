## 第七章：数据集的结合

![](img/chapterart.png)

数据常常分布在多个容器中。因此，你通常需要将不同的数据集合并成一个。在前几章中你已经进行过一些数据合并，但在本章中，我们将深入探讨更多结合数据集的技术。

在某些情况下，结合数据集可能仅仅是将一个数据集添加到另一个数据集的末尾。例如，财务分析师每周可能会收到一批新的股票数据，需要将其添加到现有的股票数据集中。其他时候，你可能需要更有选择性地结合共享某个公共列的数据集，将它们合并为一个汇总数据集。例如，零售商可能希望将有关在线订单的通用数据与关于所订购商品的具体细节合并，正如你在第六章中所看到的。在任何情况下，一旦你将数据结合起来，就可以用它进行进一步的分析。例如，你可以对结合后的数据集运行一系列筛选、分组或聚合操作。

正如你在前几章中学到的那样，Python 中的数据集可以采用内建数据结构，如列表、元组和字典，或者可以使用第三方数据结构，如 NumPy 数组或 pandas DataFrame。在后者的情况下，你拥有更丰富的工具集来结合数据，因此在需要满足某些连接条件时，你有更多的选择。然而，这并不意味着你不能有效地结合内建的 Python 数据结构。本章将展示如何做到这一点，以及如何结合第三方数据结构。

## 结合内建数据结构

将 Python 内建数据结构结合的语法非常简单。在这一节中，你将看到如何使用`+`运算符结合列表或元组。接着，你将学习如何使用`**`运算符结合字典。你还将探索如何对元组列表执行连接、聚合和其他操作，基本上将它们当作数据库表来处理，每个元组代表一行数据。

### 使用 `+` 结合列表和元组

将两个或更多列表或两个或更多元组结合起来的最简单方法是使用 `+` 运算符。你只需写一个语句，将列表或元组加在一起，就像你加多个数字一样。当你需要将结构中的元素合并到一个新的结构中，而不改变元素本身时，这个方法非常有效。这个过程通常被称为 *连接*。

为了演示，我们将回到上一章中介绍的在线时尚零售商的例子。假设每天的订单信息都被收集在一个列表中，因此你会为每一天有一个列表。你可能有以下三个列表：

```py
orders_2022_02_04 = [
 (9423517, '2022-02-04', 9001),
 (4626232, '2022-02-04', 9003),
 (9423534, '2022-02-04', 9001)
]
orders_2022_02_05 = [
 (9423679, '2022-02-05', 9002),
 (4626377, '2022-02-05', 9003),
 (4626412, '2022-02-05', 9004)
]
orders_2022_02_06 = [
 (9423783, '2022-02-06', 9002),
 (4626490, '2022-02-06', 9004)
]
```

为了进一步分析，你可能需要将这些列表合并为一个单一的列表。`+` 运算符使这变得非常简单；你只需将三个列表加在一起：

```py
orders = orders_2022_02_04 + orders_2022_02_05 + orders_2022_02_06
```

合并后的`orders`列表如下所示：

```py
[
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

如你所见，三个原始列表中的元素现在都出现在一个单一列表中，顺序由你写合并语句时的顺序决定。在这个特定的例子中，合并的列表元素都是元组。然而，`+`运算符同样适用于合并元素类型为任何类型的列表。因此，你可以轻松地合并包含整数、字符串、字典或其他任何内容的列表。

你可以使用相同的`+`语法来合并多个元组。然而，如果你尝试使用`+`合并字典，你将遇到`unsupported operand type(s)`错误。以下部分将解释合并字典的正确语法。

### 使用**合并字典

`**`运算符将字典拆开，或称为解包，将其拆解为单个的键值对。要将两个字典合并为一个，你可以使用`**`解包两个字典，并将结果存储在一个新字典中。这即使在其中一个或两个字典具有层级结构的情况下也能工作。在我们的零售商示例中，考虑以下包含订单相关额外字段的字典：

```py
extra_fields_9423517 = {
  'ShippingInstructions' : { 'name'   : 'John Silver',
                             'Phone' :  [{ 'type' : 'Office', 'number' : '809-123-9309' },
                                         { 'type' : 'Mobile', 'number' : '417-123-4567' }
                                        ]}
}
```

得益于有意义的键名，字典的嵌套结构变得清晰。事实上，通过键而不是位置访问数据的能力，使得在处理层级数据结构时，字典比列表更为优越。

现在假设你有另一个字典，其中包含相同订单的其他字段：

```py
order_9423517 = {'OrderNo':9423517, 'Date':'2022-02-04', 'Empno':9001}
```

你的任务是将这些字典合并成一个单一的字典，包含原始字典中的所有键值对。你可以像下面这样使用`**`运算符：

```py
order_9423517 = {**order_9423517, **extra_fields_9423517}
```

你将要连接的字典放在花括号内，每个字典名前面加上`**`。`**`运算符解包这两个字典为它们的键值对，然后花括号将它们重新包装成一个单一的字典。现在，`order_9423517`看起来像这样：

```py
{
 'OrderNo': 9423517,
 'Date': '2022-02-04',
 'Empno': 9001,
 'ShippingInstructions': {'name': 'John Silver',
                          'Phone': [{'type': 'Office', 'number': '809-123-9309'},
                                    {'type': 'Mobile', 'number': '417-123-4567'}
                                   ]}
}
```

如你所见，原始字典中的所有元素都已存在，并且它们的层级结构得到了保留。

### 将两个结构中的对应行合并

你已经知道如何将多个列表合并成一个列表，而不改变这些列表的元素。实际上，你还经常需要将两个或多个共享公共列的数据结构合并为一个结构，将这些数据结构中的对应行合并为单一行。如果你的数据结构是 pandas DataFrame，你可以使用像`join()`和`merge()`这样的方法，正如你在前几章中看到的那样。然而，如果你的数据结构是包含“行”的元组列表，那么这些方法并不可用。相反，你需要遍历这些列表，并逐行地合并。

为了说明，我们将结合本章中“通过+合并列表和元组”部分中创建的`orders`列表与第六章“数据聚合”中介绍的`details`列表。提醒一下，以下是该列表的样子：

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
```

两个列表都包含元组，元组的第一个元素是订单号。目标是找到具有匹配订单号的元组，将它们合并为一个元组，并将所有元组存储在一个列表中。实现方法如下：

```py
❶ orders_details = []
❷ for o in orders:
  for d in details:
  ❸ if d[0] == o[0]:
      orders_details.append(o + ❹ d[1:])
```

首先，创建一个空列表来接收合并后的元组 ❶。然后，使用一对嵌套循环来遍历`details`列表 ❷，并通过`if`语句 ❸来合并只有匹配订单号的元组。为了避免在合并并追加到`orders_details`列表的元组中重复出现订单号，您可以对每个`details`元组使用切片 ❹，选择它的所有字段，除了第一个包含冗余订单号的字段。

看着这段代码，您可能会想，它是否可以更优雅地实现为一行代码。确实，借助列表推导式，您可以如下实现相同的结果：

```py
orders_details =  [[o for o in orders if d[0] == o[0]][0] + d[1:] for d in details]
```

在外部列表推导式中，您遍历`details`列表中的元组。在内部列表推导式中，您找到`orders`列表中与当前`details`元组的订单号匹配的元组。由于`details`中的订单行应该只有一个与`orders`中的元组匹配，内部列表推导式应该生成一个包含单个元素的列表（代表一个订单的元组）。因此，您通过`[0]`运算符取出内部列表推导式的第一个元素，然后使用`+`运算符将该订单的元组与`details`中对应的元组连接起来，省略冗余的订单号部分`[1:]`。

无论是通过列表推导式创建的`orders_details`，还是借助两个`for`循环创建的，如前所示，生成的列表将如下所示：

```py
[
 (9423517, '2022-02-04', 9001, 'Jeans', 'Rip Curl', 87.0, 1),
 (9423517, '2022-02-04', 9001, 'Jacket', 'The North Face', 112.0, 1),
 (4626232, '2022-02-04', 9003, 'Socks', 'Vans', 15.0, 1),
 (4626232, '2022-02-04', 9003, 'Jeans', 'Quiksilver', 82.0, 1),
 (9423534, '2022-02-04', 9001, 'Socks', 'DC', 10.0, 2),
 (9423534, '2022-02-04', 9001, 'Socks', 'Quiksilver', 12.0, 2),
 (9423679, '2022-02-05', 9002, 'T-shirt', 'Patagonia', 35.0, 1),
 (4626377, '2022-02-05', 9003, 'Hoody', 'Animal', 44.0, 1),
 (4626377, '2022-02-05', 9003, 'Cargo Shorts', 'Animal', 38.0, 1),
 (4626412, '2022-02-05', 9004, 'Shirt', 'Volcom', 78.0, 1),
 (9423783, '2022-02-06', 9002, 'Boxer Shorts', 'Superdry', 30.0, 2),
 (9423783, '2022-02-06', 9002, 'Shorts', 'Globe', 26.0, 1),
 (4626490, '2022-02-06', 9004, 'Cargo Shorts', 'Billabong', 54.0, 1),
 (4626490, '2022-02-06', 9004, 'Sweater', 'Dickies', 56.0, 1)
]
```

该列表包含`details`列表中的所有元组，每个元组还包含来自`orders`列表中对应元组的附加信息。

### 实现不同类型的列表连接

您在前一部分执行的操作是一个标准的“一对多”连接：`details`中的每个订单行都有一个在`orders`中匹配的订单，而`orders`中的每个订单有一个或多个在`details`中的订单行。然而，在实际操作中，连接的两个数据集中的某些行可能没有在另一个数据集中找到匹配的行。为了应对这种情况，您必须能够执行类似于我们在第三章中讨论的各种数据库风格的连接操作：左连接、右连接、内连接和外连接。

举个例子，`details`列表可能包含一些在`orders`列表中找不到的订单行。发生这种情况的一种方式是你对`orders`进行某个日期范围的过滤；由于`details`中没有日期字段，因此你无法相应地过滤`details`列表。为了模拟这种情况，向`details`列表中添加一行，引用一个不在`orders`中的订单：

```py
details.append((4626592, 'Shorts', 'Protest', 48.0, 1))
```

如果你现在像之前一样尝试生成`orders_details`列表：

```py
orders_details = [[o for o in orders if d[0] == o][0] + d[1:] for d in details]
```

你会得到以下错误：

```py
IndexError: list index out of range
```

问题出现在你到达`details`列表中的不匹配订单号时，并试图检索对应内部列表推导中的第一个元素。由于订单号不在`orders`列表中，因此不存在这样的元素。解决此问题的一种方法是在外部列表推导中的`for d in details`循环内添加`if`语句，检查`details`行中的订单号是否可以在`orders`的任何行中找到，如下所示：

```py
orders_details = [[o for o in orders if d[0] in o][0] + d[1:] for d in details
                ❶ if d[0] in [o[0] for o in orders]]
```

你通过排除任何没有匹配行的`details`行来解决问题，并在`for d in details`循环后的`if`语句中实现检查❶。因此，下面显示的列表推导会导致一个内连接。

但是，如果你想在结果`orders_details`列表中包含所有的`details`行呢？你可能想这样做，例如，方便你总结所有订单的总额，而不仅仅是当前`orders`列表中的订单总额（该列表假设已经按日期进行过过滤）。你可能会总结当前`orders`列表中*存在*的订单总额，并比较这些总和。

```py`` What you want to implement in this case is a right join, assuming the `orders` list is on the left side of the relationship and the `details` list is on the right. Recall that a right join returns all rows from the right dataset and only the matched rows from the left dataset. Update the previous list comprehension as follows:    ``` orders_details_right = [[o for o in orders if d[0] in o][0] + d[1:] if d[0] in [o[0] for o                         in orders] ❶ else (d[0], None, None) + d[1:] for d in details] ```py    Here, you add an `else` clause ❶ to the `if` clause assigned to the `for d in details` loop. This `else` clause works for any `details` row that doesn’t have a matching row in `orders`. It creates a new tuple containing the order number plus two `None` entries to take the place of the missing `orders` fields, and it concatenates that tuple with the row from `details`, yielding a row with the same structure as all the others. So, the generated dataset will include the `details` row that doesn’t have a matched row in `orders` in addition to all the matching rows:    ``` [  `--snip--`  (4626490, '2022-02-06', 9004, 'Sweater', 'Dickies', 56.0, 1),  (4626592, None, None, 'Shorts', 'Protest', 48.0, 1) ] ```py    Now that you have the `orders_details_right` list (the right join of the `orders` and `details` lists), you can total up all the orders and compare the result with the total of just those orders included in the `orders` list. You add up the total of all the orders with Python’s built-in `sum()` function:    ``` sum(pr*qt for _, _, _, _, _, pr, qt in orders_details_right) ```py    The `for` loop passed as a parameter to `sum()` is somewhat similar to the loop used in a list comprehension in that it allows you to take only the necessary elements with each iteration of the loop. In this particular example, all you need to find with each iteration is `pr*qt`, the multiplication of the Price and Quantity values from the tuple at hand. Since you’re not actually interested in the other values of each tuple, you use `_` placeholders for them in the clause after the `for` keyword.    If you’ve followed the steps presented in this chapter so far, the preceding call will return:    ``` 779.0 ```py    You can calculate the totals of only those orders that are in the `orders` list with a modified version of the `sum()` call:    ``` sum(pr*qt for _, dt, _, _, _, pr, qt in orders_details_right ❶ if dt != None) ```py    Here you add an `if` clause to the loop to filter out orders that weren’t in the `orders` list ❶. You ignore rows where the Date (`dt`) field contains `None`, indicating that the row’s order information wasn’t retrieved from `orders`. The generated sum will be:    ``` 731.0 ```py    ## Concatenating NumPy Arrays    Unlike with lists, you can’t use the `+` operator to concatenate NumPy arrays. This is because, as discussed in Chapter 3, NumPy reserves the `+` operator for performing element-wise addition operations on multiple arrays. To concatenate two NumPy arrays, you instead use the `numpy.concatenate()` function.    To demonstrate, we’ll use the `base_salary` array from “Creating a NumPy Array” in Chapter 3, which was created as follows (here, we’ll call it `base_salary1`):    ``` import numpy as np jeff_salary = [2700,3000,3000] nick_salary = [2600,2800,2800] tom_salary = [2300,2500,2500] base_salary1 = np.array([jeff_salary, nick_salary, tom_salary]) ```py    Recall that each row in the array contains three months’ worth of base salary data for a particular employee. Now suppose you have salary information for two more employees in another array, `base_salary2`:    ``` maya_salary = [2200,2400,2400] john_salary = [2500,2700,2700] base_salary2 = np.array([maya_salary, john_salary]) ```py    You want to store the salary information for all five employees in the same array. To do so, you concatenate `base_salary1` and `base_salary2` using `numpy.concatenate()`, as follows:    ``` base_salary = np.concatenate((base_salary1, base_salary2), axis=0) ```py    The first parameter is a tuple containing the arrays to be concatenated. The second parameter, `axis`, is critical: it specifies whether the arrays should be concatenated horizontally or vertically, or in other words whether the second array will be added as new rows or new columns. The first axis (axis 0) runs vertically. Thus, `axis=0` instructs the `concatenate()` function to append the rows of `base_salary2` beneath those of `base_salary1`. The resulting array will look as follows:    ``` [[2700 3000 3000]  [2600 2800 2800]  [2300 2500 2500],  [2200 2400 2400],  [2500 2700 2700]] ```py    Now imagine that the salary information for the next month has come in. You might put these new figures in another NumPy array, as follows:    ``` new_month_salary = np.array([[3000],[2900],[2500],[2500],[2700]]) ```py    If you print the array, you’ll see the following output:    ``` [[3000]  [2900]  [2500]  [2500]  [2700]] ```py    You need to add this `new_month_salary` array to the `base_salary` array as an extra column. Assuming the order of employees is the same in both arrays, you can use `concatenate()` for this as follows:    ``` base_salary = np.concatenate((base_salary, new_month_salary), axis=1) ```py    Since axis 1 runs horizontally across the columns, `axis=1` instructs the `concatenate()` function to append the `new_month_salary` array as a column to the right of the `base_salary` array’s columns. Now `base_salary` will look like this:    ``` [[2700 3000 3000 3000]  [2600 2800 2800 2900]  [2300 2500 2500 2500]  [2200 2400 2400 2500]  [2500 2700 2700 2700]] ```py    ## Combining pandas Data Structures    In Chapter 3, we covered some basic techniques for combining pandas data structures. You saw examples of how Series objects can be combined into a DataFrame and how two DataFrames can be joined on their indexes. You also learned about the different types of joins you can create when merging two DataFrames into a single one, by passing the `how` parameter in to the pandas `join()` or `merge()` method. In this section, you’ll see more examples of how to use this parameter to create nondefault DataFrame joins, such as a right join. Before that, however, you’ll learn to concatenate two DataFrames along a particular axis.    ### Concatenating DataFrames    Like with NumPy arrays, you might need to concatenate two DataFrames along a particular axis, appending either the rows or the columns of one DataFrame to the other. The examples in this section show how to do this with the pandas `concat()` function. Before proceeding to the examples, you’ll need to create two DataFrames to be concatenated.    Using the `jeff_salary`, `nick_salary`, and `tom_salary` lists from earlier in this chapter, you can create a DataFrame using a dictionary, like so:    ``` import pandas as pd salary_df1 = pd.DataFrame(     {'jeff': jeff_salary,      'nick': nick_salary,      'tom': tom_salary     })  ```py    Each list becomes a value in the dictionary, which in turn becomes a column in the new DataFrame. The dictionary’s keys, which are the corresponding employee names, become the column labels. Each row of the DataFrame contains all the salary data for a single month. By default the rows are indexed numerically, but it would be more meaningful to index them by month. You can update the indices as follows:    ``` salary_df1.index = ['June', 'July', 'August'] ```py    The `salary_df1` DataFrame will now look like this:    ```  jeff  nick   tom June    2700  2600  2300 July    3000  2800  2500 August  3000  2800  2500 ```py    You may find it more convenient to view the salary data of an employee as a row rather than a column. You make this change with the DataFrame’s `T` property, which is a shorthand for the `DataFrame.``transpose()` method:    ``` salary_df1 = salary_df1.T ```py    This statement *transposes* the DataFrame, turning its columns into rows and vice versa. The DataFrame is now indexed by employee name and looks as follows:    ```  June  July  August jeff  2700  3000    3000 nick  2600  2800    2800 tom   2300  2500    2500 ```py    Now you need to create another DataFrame with the same columns to be concatenated with `salary_df1`. Following in line with the example of concatenating NumPy arrays, here you create a DataFrame that holds salary data for two more employees:    ``` salary_df2 = pd.DataFrame(     {'maya': maya_salary,      'john': john_salary     },     index = ['June', 'July', 'August'] ).T  ```py    You create the DataFrame, set the index, and transpose the rows and columns all in a single statement. The newly created DataFrame will look as follows:    ```  June  July  August maya  2200  2400    2400 john  2500  2700    2700 ```py    Now that you’ve created both DataFrames, you’re ready to concatenate them.    #### Concatenation Along Axis 0    The pandas `concat()` function concatenates pandas objects along a certain axis. By default, this function uses axis 0, meaning the rows of the DataFrame that appears second in the argument list will be appended below the rows of the DataFrame that appears first. Thus, to concatenate the `salary_df1` and `salary_df2` DataFrames in this manner, you can call `concat()` without passing the `axis` argument explicitly. All you have to do is specify the names of the DataFrames within square brackets:    ``` salary_df = pd.concat([salary_df1, salary_df2]) ```py    This will generate the following DataFrame:    ```  June  July  August jeff  2700  3000    3000 nick  2600  2800    2800 tom   2300  2500    2500 maya  2200  2400    2400 john  2500  2700    2700 ```py    As you can see, the `maya` and `john` rows from the second DataFrame have been added beneath the rows from the first DataFrame.    #### Concatenation Along Axis 1    When concatenating along axis 1, the `concat()` function will append the columns of the second DataFrame to the right of the columns of the first one. To illustrate this, you can use `salary_df` from the preceding section as the first DataFrame. For the second DataFrame, create the following structure, which holds two more months’ worth of salary data:    ``` salary_df3 = pd.DataFrame(     {'September': [3000,2800,2500,2400,2700],      'October': [3200,3000,2700,2500,2900]     },     index = ['jeff', 'nick', 'tom', 'maya', 'john'] ) ```py    Now call `concat()`, passing in the two DataFrames and specifying `axis=1` to ensure the concatenation is horizontal:    ``` salary_df = pd.concat([salary_df, salary_df3], axis=1) ```py    The resulting DataFrame will look as follows:    ```  June  July  August  September  October jeff  2700  3000    3000       3000     3200 nick  2600  2800    2800       2800     3000 tom   2300  2500    2500       2500     2700 maya  2200  2400    2400       2400     2500 john  2500  2700    2700       2700     2900 ```py    The salary data from the second DataFrame appears as new columns to the right of the salary data from the first DataFrame.    #### Removing Columns/Rows from a DataFrame    After combining DataFrames, you may need to remove some unnecessary rows or columns. Let’s say, for example, you want to remove the `September` and `October` columns from the `salary_df` DataFrame. You can do this with the `DataFrame.``drop()` method, as follows:    ``` salary_df = salary_df.drop(['September', 'October'], axis=1) ```py    The first argument takes the names of the columns or rows to be deleted from the DataFrame. Then you use the `axis` argument to specify whether they’re rows or columns. In this example you’re deleting columns, because `axis` is set to `1`.    With `drop()`, you aren’t limited to only deleting the last columns/rows of a DataFrame. You can pass in an arbitrary list of columns or rows to be deleted, as shown here:    ``` salary_df = salary_df.drop(['nick', 'maya'], axis=0) ```py    After performing the two previous operations, `salary_df` will appear as follows:    ```  June  July  August jeff  2700  3000    3000 tom   2300  2500    2500 john  2500  2700    2700 ```py    You’ve removed the columns for September and October and the rows for Nick and Maya.    #### Concatenating DataFrames with a Hierarchical Index    So far you’ve seen examples of concatenating DataFrames with simple indexes. Now let’s consider how to concatenate DataFrames with a MultiIndex. The following example uses the `df_date_region` DataFrame introduced in “Grouping and Aggregating the Data” in Chapter 6. The DataFrame was created as a result of several successive operations and looked as follows:    ```  Total Date       Region        2022-02-04 East     97.0            West    243.0 2022-02-05 East    160.0            West     35.0 2022-02-06 East    110.0            West     86.0 ```py    To re-create this DataFrame, you don’t have to follow the steps from Chapter 6. Instead, execute the following statement:    ``` df_date_region1 = pd.DataFrame(  [         ('2022-02-04', 'East', 97.0),   ('2022-02-04', 'West', 243.0),   ('2022-02-05', 'East', 160.0),   ('2022-02-05', 'West', 35.0),   ('2022-02-06', 'East', 110.0),   ('2022-02-06', 'West', 86.0)  ],  columns =['Date', 'Region', 'Total']).set_index(['Date','Region']) ```py    Now you’ll need another DataFrame that’s also indexed by `Date` and `Region`. Create it like this:    ``` df_date_region2 = pd.DataFrame(  [         ('2022-02-04', 'South', 114.0),   ('2022-02-05', 'South', 325.0),   ('2022-02-06', 'South', 212.0)  ],  columns =['Date', 'Region', 'Total']).set_index(['Date','Region']) ```py    The second DataFrame features the same three dates as the first but has data for a new region, `South`. The challenge when concatenating these two DataFrames is to keep the result sorted by date rather than simply appending the second DataFrame beneath the first one. Here’s how you can do this:    ``` df_date_region = pd.concat([df_date_region1,             df_date_region2]).sort_index(level=['Date','Region']) ```py    You start with a `concat()` call that looks the same as it would if it were concatenating DataFrames with single-column indexes. You identify the DataFrames to be combined, and since you omit the `axis` parameter, they’ll be concatenated vertically by default. To sort the rows in the resulting DataFrame by date and region, you then have to call the `sort_index()` method. As a result, you’ll get the following DataFrame:    ```  Total Date       Region        2022-02-04 East     97.0            South   114.0            West    243.0 2022-02-05 East    160.0            South   325.0            West     35.0 2022-02-06 East    110.0            South   212.0            West     86.0 ```py    As you can see, the rows from the second DataFrame have been integrated among the rows from the first DataFrame, maintaining the top-level grouping by date.    ### Joining Two DataFrames    When you join two DataFrames, you combine each row from one dataset with the matching row(s) from the other, rather than simply appending one DataFrame’s rows or columns beneath or beside the other’s. To review the basics of joining DataFrames, refer back to “Combining DataFrames” in Chapter 3, which covers the different types of joins you can implement. In this section, you’ll go beyond what you learned in Chapter 3 by implementing a right join and a join based on a many-to-many relationship.    #### Implementing a Right Join    A right join takes all the rows from a second DataFrame and combines them with any matching rows from a first DataFrame. As you’ll see, this type of join comes with the possibility that some rows in the resulting DataFrame will have undefined fields, which can lead to unexpected challenges.    To demonstrate, you’ll perform a right join of the `df_orders` and `df_details` DataFrames created from the `orders` and `details` lists introduced in Chapter 6, which you used in the examples in the opening section of this chapter. Create the DataFrames from these lists as follows:    ``` import pandas as pd df_orders = pd.DataFrame(orders, columns =['OrderNo', 'Date', 'Empno']) df_details = pd.DataFrame(details, columns =['OrderNo', 'Item', 'Brand',                                              'Price', 'Quantity']) ```py    Recall that each row in the original `details` list has a matching row in the `orders` list. Therefore, the same is true for the `df_details` and `df_orders` DataFrames. To properly illustrate a right join, you need to add one or more new rows to `df_details` that don’t have matches in `df_orders`. You can add a row using the `DataFrame.``append()` method, which takes the row being appended either as a dictionary or a Series.    If you followed along with the example in “Implementing Different Types of Joins for Lists” earlier in this chapter, you’ve already added the following row to the `details` list, and therefore it should already appear in the `df_details` DataFrame. In that case, you can ignore the following append operation. Otherwise, append this row to `df_details` as a dictionary. Note that the value of the new row’s `OrderNo` field can’t be found among the values of the `OrderNo` column in the `df_orders` DataFrame:    ``` df_details = df_details.append(   {'OrderNo': 4626592,    'Item': 'Shorts',    'Brand': 'Protest',    'Price': 48.0,    'Quantity': 1   }, ❶ ignore_index = True ) ```py    You must set the `ignore_index` parameter to `True` ❶, or you won’t be able to append a dictionary to a DataFrame. Setting this parameter to `True` also resets the DataFrame’s index, maintaining continuous index values (`0, 1, ...`) for the rows.    Next, you join the `df_orders` and `df_details` DataFrames using the `merge()` method. As discussed in Chapter 3, `merge()` provides a convenient way of joining two DataFrames with a common column:    ``` df_orders_details_right = df_orders.merge(df_details, ❶ how='right',                   ❷ left_on='OrderNo', right_on='OrderNo') ```py    You use the `how` parameter to specify the type of join—in this example, a right join ❶. With the `left_on` and `right_on` parameters, you specify the columns to join on from the `df_orders` and `df_details` DataFrames, respectively ❷.    The resulting DataFrame will look as follows:    ```  OrderNo        Date   Empno          Item           Brand  Price  Quantity 0   9423517  2022-02-04  9001.0         Jeans        Rip Curl   87.0         1 1   9423517  2022-02-04  9001.0        Jacket  The North Face  112.0         1 2   4626232  2022-02-04  9003.0         Socks            Vans   15.0         1 3   4626232  2022-02-04  9003.0         Jeans      Quiksilver   82.0         1 4   9423534  2022-02-04  9001.0         Socks              DC   10.0         2 5   9423534  2022-02-04  9001.0         Socks      Quiksilver   12.0         2 6   9423679  2022-02-05  9002.0       T-shirt       Patagonia   35.0         1 7   4626377  2022-02-05  9003
