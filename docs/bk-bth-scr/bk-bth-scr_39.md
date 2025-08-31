

## 第三十六章：B 数组和哈希表对象



![](img/chapter.jpg)

在第二十九章中，我展示了如何构建、访问和更新数组以及哈希表。这些数据结构虽然与 bat 文件不常相关，但它们是第三十三章中讨论的同样不典型的现实世界批处理对象的极好应用。

在这个附录中，我将通过为每种数据结构提供一个有良好注释的对象 bat 文件来将这些概念结合起来。

## 数组对象

这里是数组对象 *oArray.bat* 的完整内容：

```
:: ****** Array Object ****** 
:: Parm 1 – Name of Array
:: Parm 2 - Name of Method: 
::       AddElemAt - Insert One Element at an Index
::               Parm 3 - Index of Element being Added
::               Parm 4 - Value of Element being Added
::       AddElem - Add One Element to the Array 
::               Parm 3 - Element being Added
::       GetElem - Get the Element at an Index
::               Parm 3 - Index of Element
::               Parm 4 - Returned Variable Name
::       GetFirst - Get the First Element in the Array
::               Parm 3 - Returned Variable Name
::       GetNext - Get the Next Element in the Array; call after
::                      :GetElem or :GetFirst or gets first element
::               Parm 3 - Returned Variable Name
::       GetSize - Get the Number of Elements in the Array
::               Parm 3 - Returned Variable Name
::       RemoveElemAt - Remove One Element from the Array
::               Parm 3 - Index of Element being Removed
::       Clear - Empty the Array of all its Elements
::       IndexOf - Get the Index of a Specific Value
::                      or return -1 if Not Found
::               Parm 3 - Value of Search Element
::               Parm 4 - Returned Variable Name
::       Contains - Get a Boolean Indicating if a Value is 
::                        Anywhere in the Array
::               Parm 3 - Value of Search Element
::               Parm 4 - Returned Boolean Name
::       Clone - Create a Copy of the Array
::               Parm 3 - Name of New Array
:: Global Variables:
::   <arrayName>Size = Size or Length of the Array
::   <arrayName>Index = Index or Pointer to the Next Element
::   <arrayName>[n] = Nth Element of the Array

  cmd /C exit 0
  call :%~2 "%~1" "%~3" "%~4" || (
     > C:\Batch\Log.txt echo ** ERROR - Invalid Method Name "%~2"
     exit
  )
  goto :eof

 :AddElemAt
  call :GetSize %~1 size
  if %~2 gtr %size% (
     echo ** Invalid Index "%~2" greater than Array Size "%size%"
     goto :eof
  )
  set /A startIndex = !%size! - 1
  for /L %%i in (%startIndex%, -1, %~2) do (
     set /A nextIndex = %%i + 1
     for /F %%n in ("!nextIndex!") do (
       set %~1[%%n]=!%~1[%%i]!
  )  )
  set %~1[%~2]=%~3
  set /A %~1Size += 1
  goto :eof

 :AddElem
  call :GetSize "%~1" size
  set %~1[!size!]=%~2
  set /A %~1Size += 1
  goto :eof

 :GetElem
  set %~3=!%~1[%~2]!
  set /A %~1Index = %~2 + 1
  goto :eof

 :GetFirst
  set %~2=!%~1[0]!
  set %~1Index=1
  goto :eof

 :GetNext
  if not defined %~1Index  set %~1Index=0
  call :GetSize "%~1" size
  set targIndex=!%~1Index!
  if %targIndex% geq %size% (
      set %~2=No More Elements
  ) else (
      set %~2=!%~1[%targIndex%]!
      set /A %~1Index += 1
  )
  goto :eof

 :GetSize
  if not defined %~1Size  set %~1Size=0
  set %~2=!%~1Size!
  goto :eof

 :RemoveElemAt
  call :GetSize "%~1" size
  if %~2 geq %size% (
     echo ** Nothing to do, Index "%~2" greater than Array Size "%size%"
     goto :eof
  )
  set /A %~1Size -= 1
  for /L %%i in (%~2, 1, !%~1Size!) do (
      set /A nextIndex = %%i + 1
      for /F %%n in ("!nextIndex!") do (
        set %~1[%%i]=!%~1[%%n]!
  )  )
  set %~1[!nextIndex!]=&
  goto :eof

 :Clear
  for /F "usebackq delims==" %%a in (`set %~1`) do (
      set %%a=&rem
  )
  set %~1Size=0
  goto :eof

 :IndexOf
  set %~3=-1
  set /A sizeLess1 = %~1Size - 1
  for /L %%i in (0, 1, %sizeLess1%) do (
      if "%~2" equ "!%~1[%%i]!" (
         set %~3=%%i
  )  )
  goto :eof

 :Contains
  call :IndexOf "%~1" "%~2" indexOf
  if %indexOf% equ -1 (
     set %~3=false==x
  ) else (
     set %~3=true==true
  )
  goto :eof

 :Clone
  call :Clear "%~2"
  for /F "usebackq tokens=1,2 delims==" %%p in (`set %~1`) do (
     set oldArrayItem=%%p
     set !oldArrayItem:%~1=%~2!=%%q
  )
  goto :eof 
```

这个 bat 文件应该经常被调用，所以我使用了::（两个冒号）来进行备注，而不是使用 rem 命令，这样可以减少解释器写入到标准输出的内容。

每次调用这个对象至少传递两个参数：数组的名称和正在调用的方法或操作；根据方法的不同，可能需要传递一个或两个额外的参数。你可以将元素添加到数组的末尾或指定的索引位置；你可以获取第一个元素、下一个元素，或者获取特定索引位置的元素。该对象有删除指定索引元素、获取数组大小以及清空数组的方法。你可以获取某个特定值第一次出现的索引，或者通过布尔值检查该值是否存在于数组中。你甚至可以克隆或复制数组。

我不会逐一讲解每个方法，而是让注释来说明。请注意，我已经为每个方法附上了简短的描述，并列出了它们所需的参数。

然而，这里有一些有趣的代码值得一提。你会在这个列表中看到大量延迟扩展的例子。实际上，一些方法使用嵌套的 for 命令完全是为了延迟扩展；每个 for 命令都将外部 for 命令中分配的变量转换成可以用百分号符号（第二十章）解析的变量。此外，`:IndexOf` 和 `:Contains` 方法执行相似的功能。为了避免重复工作，后者调用了前者，将结果转化为布尔值。同样，多个方法通过调用 `:GetSize` 来获取数组的大小。`:Clone` 方法将一个数组相关的所有变量赋值给另一个数组，利用文本替换的特性以及数组元素仅仅是普通变量的事实。

你可以从另一个 bat 文件调用该对象来执行所有这些功能。这里有一个小示例：

```
call C:\Batch\oArray.bat friends AddElem Walter
call C:\Batch\oArray.bat friends AddElem Donny
call C:\Batch\oArray.bat friends AddElemAt 1 Maude
call C:\Batch\oArray.bat friends RemoveElemAt 0
call C:\Batch\oArray.bat friends GetFirst oneFriend
call C:\Batch\oArray.bat friends GetNext anotherFriend 
```

这段代码将 `oneFriend` 和 `anotherFriend` 分别赋值为 Maude 和 Donny。

为了可读性，这个对象的错误处理和传入参数的验证非常简单，但这些相对较少的代码行已经准备好可以创建、修改和访问任意数量的数组。

## 哈希表对象

这里是哈希表对象 *oHashTable.bat* 的完整内容：

```
:: ****** Hash Table Object ****** 
:: Parm 1 – Name of Hash Table
:: Parm 2 - Name of Method: 
::       Clear - Empty the Hash Table of all its Keys and Values
::       Put - Put One Key-Value Pair into the Hash Table
::               Parm 3 - Key being Added
::               Parm 4 - Value being Added
::       Get - Get a Value Given a Key
::               Parm 3 - Search Key
::               Parm 4 - Returned Variable Name
::       GetSize - Get the Number of Key-Value Pairs in the Hash Table
::               Parm 3 - Returned Variable Name
::       Remove - Remove One Key and its Value from the Hash Table
::               Parm 3 - Key being Removed
::       ContainsKey - Get a Boolean Indicating if a Key is 
::                       Anywhere in the Hash Table
::               Parm 3 - Search Key
::               Parm 4 - Returned Boolean Name
::       ContainsValue - Get a Boolean Indicating if a Value is 
::                       Anywhere in the Hash Table
::               Parm 3 - Search Key
::               Parm 4 - Returned Boolean Name
::       Clone - Create a Copy of the Hash Table
::               Parm 3 - Name of New Hash Table
:: Global Variable:
::    <hashTable>Size = Size or Length of the Hash Table

 cmd /C exit 0
  call :%~2 "%~1" "%~3" "%~4" || (
     > C:\Batch\Log.txt echo ** ERROR - Invalid Method Name "%~2"
     exit
  )
  goto :eof

 :Clear
  for /F "usebackq delims==" %%a in (`set %~1`) do (
      set %%a=&rem
  )
  set %~1Size=0
  goto :eof

 :Put
  call :ContainsKey "%~1" "%~2" bool
  set %~1{%~2}=%~3
  if not %bool% (
     set /A %~1Size += 1
  )
  goto :eof

 :Get
  call :ContainsKey "%~1" "%~2" bool
  if %bool% (
     set %~3=!%~1{%~2}!
  ) else (
     set %~3=Key Does Not Exist
  )
  goto :eof

 :GetSize
  if not defined %~1Size  set %~1Size=0
  set %~2=!%~1Size!
  goto :eof

 :Remove
  call :ContainsKey "%~1" "%~2" bool
  if %bool% (
     set /A %~1Size -= 1
  )
  set %~1{%~2}=&
  goto :eof

 :ContainsKey
  if defined %~1{%~2} (
     set %~3=true==true
  ) else (
     set %~3=false==x
  )
  goto :eof

 :ContainsValue
  set %~3=false==x
  for /F "usebackq tokens=2 delims==" %%v in (`set %~1{`) do (
     if "%%v" equ "%~2" (
        set %~3=true==true
  )  )
  goto :eof

 :Clone
  call :Clear "%~2"
  for /F "usebackq tokens=1,2 delims==" %%p in (`set %~1`) do (
     set oldHashTblItem=%%p
     set !oldHashTblItem:%~1=%~2!=%%q
  )
  goto :eof 
```

此对象还接受至少两个参数：哈希表的名称和正在调用的方法或操作。您可以通过调用:Put 方法向数据结构添加键值对，并通过:Get 方法获取给定键的值。其他方法可以清空整个哈希表或仅移除一个键值对。您可以获取键值对的数量，并检索一个布尔值，显示键或值是否存在，并且像数组对象一样，还有一个克隆方法。

在.bat 文件的开头注释中描述了每个方法及其对应的参数。最有趣的方法是：ContainsValue，在执行搜索之前将布尔值预设为 false，然后查看每对的值。然而，确定哈希表中是否存在键仅需稍作判断。

这里有几行代码，演示了对对象功能进行简单测试的方法：

```
call C:\Batch\oHashTable.bat jobs Put Lincoln President
call C:\Batch\oHashTable.bat jobs Put Poe Poet
call C:\Batch\oHashTable.bat jobs Put Darwin Naturalist
call C:\Batch\oHashTable.bat jobs Get Poe aJob 
```

在这些命令完成后，aJob 变量包含值 Poet。

您可以从多个.bat 文件中调用此对象，甚至可以从单个进程构建多个哈希表。现在寻找其他实例，通过将有趣的逻辑放置在可重用对象.bat 文件中，可以使您的主代码保持简洁。
