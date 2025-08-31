# 第三十章：C

命令行

与大多数矢量编辑器不同——但与大多数开源软件相似——Inkscape 拥有强大的命令行界面。通过它，你可以执行导出、文档修改、查询等任务，既可以通过脚本，也可以通过命令行提示符完成，而无需使用 Inkscape 的图形用户界面（GUI）。在大多数情况下，当 Inkscape 完成命令行参数中指定的任务后，它会直接退出，根本不显示 GUI。这使得它运行得更快，占用更少的资源，因为没有时间或内存浪费在创建和销毁 GUI 上。

在本附录中，我将解释并提供 Inkscape 最常用的命令行参数示例。我还介绍了通过 `--actions` 参数提供的动作语言，允许你编写复杂的文档处理场景。由于程序的命令行界面在最近的版本中经过了彻底的修订，我还指出了最重要的变化，帮助你迁移可能在 Inkscape 1.0 中无法正常运行的旧脚本。

## C.1 命令行可执行文件

在 Windows 上，命令行工具和 GUI 应用程序是两种不同类型的程序。一个可执行文件不能同时作为 GUI 应用程序和命令行应用程序运行。这就是为什么 Inkscape 在 Windows 上包含了两个可执行文件：inkscape.exe 和 inkscape.com。

+   inkscape.exe 是在需要交互式使用 Inkscape 时运行的 GUI 应用程序。如果你尝试用它来执行命令行任务，你将在控制台（命令提示符窗口）中看不到任何输出，这使得查询参数（C.5）无法使用。此外，inkscape.exe 会异步返回（立即），可能会在复杂操作（如导出）完成之前就返回。

+   inkscape.com 最适用于脚本编写和命令行任务。它在控制台中显示正确的输出并同步返回。然而，如果你运行它用于图形用户界面（GUI），你将始终看到一个无法关闭的寄生命令行窗口，除非关闭 GUI 窗口才能将其关闭。

在 Linux 或 macOS 上，只有一个名为 inkscape 的可执行文件，用于图形界面和命令行。在本附录中的示例中，我使用 `inkscape` 作为可执行文件的名称。此外，我在文件路径中使用了正斜杠（`/`），这种路径在包括 Windows 在内的所有平台上都可以正常工作。

## C.2 获取帮助

要获取与你版本的 Inkscape 兼容的命令行参数完整列表，请使用 `--help` 参数运行它：

```
**$ inkscape --help**
Usage: org.inkscape.Inkscape [OPTION…] file1 [file2 [fileN]]
Process (or open) one or more files.
Help Options: -?, --help                                 Show help options
... 
```

参数或 *选项* 的名称以 `--` 开头，例如 `--help`。其中一些可以有由 `=` 分隔的值，例如 `--export-fileid=file.pdf`。使用空格来分隔命令行参数。更详细的参考可以通过 Inkscape 的用户界面中的帮助 ▶ 命令行选项获取（这将打开一个网页浏览器并从互联网获取该页面），或通过在命令行中输入 `man inkscape`（仅限 Linux）来获取。

一个非常有用的参数（尤其是当你想报告 Bug 或请求 Inkscape 开发者添加功能时）是`--version`，它会打印你使用的 Inkscape 副本的版本号、修订号和构建日期。你还可以使用`--system-data-directory`或`--user-data-directory`查询 Inkscape；这两个参数分别会打印系统偏好设置对话框中的“Inkscape 数据”和“用户配置”标签值。

## C.3 打开文档

命令行最简单的使用方式是列出你想在 GUI 中打开的文档的文件路径，而不带任何参数名称。例如，输入以下命令将启动 Inkscape 的 GUI，并将两个 SVG 文档和一个 PDF 文档（自动导入）加载到三个 Inkscape 窗口中：

```
**$ inkscape file.svg folder/subfolder/document.svg pdfs/file.pdf**
```

你可以为打开非 SVG 文件指定一些导入选项。最常用的选项是`--pdf-page`，它可以选择从 PDF 文档中导入的页面，以及`--pdf-poppler`，它切换到 Poppler 导入方法（B.3.1）。例如，以下是如何从 PDF 书籍章节中打开第 5 页：

```
**$ inkscape --pdf-page=5 chapter.pdf** 
```

如果你有一个生成 SVG 文件并将其打印到控制台的脚本或程序，可以使用`|`字符将该文档*传送*到 Inkscape，并使用`--pipe`参数，例如：

```
**$ svg-generating-script | inkscape --pipe**
```

## C.4 导出

Inkscape 最常见的任务之一是将 SVG 文档导出为其他格式。通过命令行，Inkscape 可以将位图导出为 PNG（18.6），并将矢量图导出为 PS（PostScript）、EPS、PDF、EMF、WMF 和 XAML（附录 B）。Inkscape 根据你在命令行中提供的文件名扩展名来决定使用哪种格式；如果你想显式指定格式，可以添加`--export-type`并指定所需的类型（有效值包括`svg`、`png`、`ps`、`eps`、`pdf`、`emf`、`wmf`、`xaml`）。例如，导出为 PDF 的命令如下：

```
**$ inkscape --export-fileid=file.pdf file.svg**
```

这相当于

```
**$ inkscape --export-fileid=file --export-type=pdf file.svg**
```

这将加载 file.svg 并创建 PDF 文件；不会加载 GUI，完成导出后 Inkscape 会退出。如果你想强制覆盖导出文件，可以添加`--export-overwrite`。

你可以在`--export-type`中指定多个输入文件和多个导出类型（用逗号分隔）。例如，以下命令

```
**$ inkscape --export-type=png,pdf file.svg file2.svg**
```

会创建四个文件：file.png、file.pdf、file2.png、file2.pdf。

你还可以将文档导出为 SVG 文件——例如，你可以使用 Inkscape 将 PDF 文件转换为 SVG，或从一个 SVG 文件中提取一个元素并将其保存为独立的 SVG。如果你添加了`--export-plain-svg`，文档将以纯 SVG 格式保存，而不是 Inkscape 的 SVG 格式（1.4）。如果你添加了`--vacuum-defs`，生成的 SVG 将不会在`defs`中包含任何未使用的定义（A.4）。例如，这将把 PDF 文件中的一页转换为纯 SVG 格式：

```
**$ inkscape --export-fileid=file.svg --export-plain-svg --pdf-page=5 file.pdf**
```

[1.1]

在旧版本中，Inkscape 为每种导出格式提供了单独的参数：`--export-png`、`--export-pdf`、`--export-ps`、`--export-eps`等。它们都接收一个文件名作为值。现在，`--export-filename`已经取代了这些参数。`--export-plain-svg`依然存在，但现在是一个布尔开关，而不是文件名参数。

除了它原生支持的导出格式外，Inkscape 还包括了多个输出扩展（19.4.2），用于支持一些不太常见的格式。你也可以通过命令行使用这些扩展。`--export-extension`参数的值为扩展的唯一 ID，你可以在其.inx 文件中查找（19.4.1）。例如，Inkscape 1.1 包括一个用于导出到绘图仪使用的 HPGL 格式的扩展。在 share\inkscape\extensions\hpgl_output.inx 文件中，你可以找到它的 ID，即`org.ekips.output.hpgl_output`。现在，运行

```
**$ inkscape --export-extension=org.ekips.output.hpgl_output \ --export-fileid=file.hpgl file.svg**
```

将 file.svg 转换为 HPGL 格式，并创建一个名为 file.hpgl 的文件。

### C.4.1 导出区域

默认情况下，Inkscape 导出文档的页面（2.3），页面外的对象在导出时不可见。添加`--export-area-drawing`可以使导出覆盖文档中所有可见对象，而不管其页面大小。例如：

```
**$ inkscape --export-fileid=file.png --export-area-drawing file.svg**
```

唯一的例外是 EPS 格式，默认情况下会导出整个绘图；使用此格式时，使用`--export-area-page`可以导出整个页面。即使如此，由于 EPS 的限制，导出区域将会裁剪为页面中实际存在的对象，若它们没有延伸到页面边缘。

从 SVG 导出时，你可以导出文档中的单个对象，使得导出的文件只覆盖该对象的边界框，其他内容不会包含在内。该对象由其`id`属性指定（A.9）：

```
**$ inkscape --export-fileid=text.pdf --export-id=text2054 file.svg**
```

对于 PNG 导出，如果其他对象与导出的对象的边界框重叠并且可见，它们也会出现在导出的文件中。对于 SVG 导出，即使其他对象位于导出页面区域之外，它们仍然会出现在文件中。要抑制其他对象并仅渲染所选对象，可以添加`--export-id-only`。对于 PDF、PS 和 EPS，这是唯一可能的模式——当指定`--export-id`时，其他对象始终会被丢弃。

对于 PNG 导出，你还可以通过指定两个角点来显式提供导出区域。例如，以下命令导出从点 0, 0 到点 200, 100（以像素为单位）区域：

```
**$ inkscape --export-fileid=area.png --export-area=0:0:200:100 file.svg**
```

同样对于 PNG 导出，无论你使用哪种方法来指定区域，你都可以通过添加`--export-area-snap`来将该区域“对齐”到像素网格——即，将其四舍五入到最近的完整坐标（以`px`为单位）。当你在默认的 96 dpi 下导出，并希望你的对象始终在导出的位图中保持清晰（无论你导出的区域如何），此选项非常有用。

### C.4.2 导出大小和分辨率

对于 PNG 导出，您可以指定导出位图的大小或分辨率（默认是 96 dpi）。例如：

```
**$ inkscape --export-fileid=file.png --export-dpi=600 file.svg****$ inkscape --export-fileid=file.png --export-width=1000 file.svg****$ inkscape --export-fileid=file.png --export-height=400 file.svg**
```

第一行将文件.svg 以 600 dpi 的分辨率导出，因此一个宽度为 3 英寸的文档页面将导出为宽度为 1800 像素的位图。其他两个示例明确设置了导出的像素大小，并计算出相应的分辨率（如果同时存在`--export-dpi`，则会覆盖它）。

如果只指定了`--export-width`或`--export-height`，Inkscape 会按相同分辨率计算另一个维度。然而，使用命令行导出时，您还可以获得一个垂直分辨率不等于水平分辨率的 PNG 文件（这是从 GUI 中无法做到的，18.6.1.2）。例如，您可以将一个 1000×1000 像素的区域导出为一个 2000×500 像素的扭曲 PNG 文件，命令为`--export-width=2000 --export-height=500`。

`--export-dpi` 参数还会影响导出为矢量格式（PS、EPS、PDF）时的处理，尤其是当 Inkscape 需要将文件的某些特性（如 PS 和 PDF 中的滤镜和网格渐变，或 PS 中的透明度）栅格化时。

### C.4.3 导出背景（仅限 PNG）

所有导出格式中没有对象的区域在导出时会显示为透明。然而，在 PNG 导出（而不是 PDF、PS 或 EPS）中，您可以指定任何背景色或透明度，以便在导出过程中应用。例如，如果您希望使用完全不透明的黑色背景，可以输入：

```
**$ inkscape --export-fileid=file.png --export-background=#000000 \** **--export-background-opacity=1.0 file.svg**
```

背景颜色的语法与 SVG 中的相同；特别是，您可以使用`#RRGGBB`格式。背景透明度（默认为完全不透明）可以是一个浮动数字，范围从 0.0 到 1.0，或者是一个整数，范围从 1 到 255；例如，`--export-background-opacity=0.5`等同于`--export-background-opacity=128`。

### C.4.4 色彩模式（仅限 PNG）[1.1]

对于 PNG 导出，您可以使用`--export-png-color-mode`选项来设置导出文件的色彩模式，该选项接受以下值之一：`Gray_1`，`Gray_2`，`Gray_4`，`Gray_8`，`Gray_16`，`RGB_8`，`RGB_16`，`GrayAlpha_8`，`GrayAlpha_16`，`RGBA_8`，或`RGBA_16`（18.6.1.5）。

### C.4.5 导出提示（仅限 PNG）

每次通过 GUI 导出单个选中的对象为 PNG 时（18.6），导出文件名和分辨率会记录在导出提示属性中，并添加到相应的元素。如果您随后保存文档并带有这些提示，您也可以通过命令行导出为 PNG。例如，如果您写：

```
**$ inkscape --export-id=text2035 --export-use-hints file.svg**
```

Inkscape 只会将 ID 为`text2035`的对象以相同的分辨率导出到同一个文件中，该分辨率是在从 GUI 最近一次导出时设置的。请注意，`--export-filename` 参数没有，因为文件名是从导出提示中获取的。

### C.4.6 矢量导出选项

导出为 PDF、PS 和 EPS 时，您可以设置以下选项：

+   `--export-dpi`（C.4.2）设置栅格化对象（如滤镜和网格渐变）的分辨率。

+   `--export-ignore-filters`抑制经过滤的对象的栅格化，使其在导出时不应用过滤器。

+   `--export-ps-level`指定使用的 PostScript 格式的级别（版本）；它可以是 2 或 3（默认为 3）。

+   `--export-pdf-version`指定使用的 PDF 版本；它可以是 1.4 或 1.5（默认为 1.5）。

+   `--export-text-to-path`将所有文本对象转换为路径（这在导出到 SVG 时也可用）。

例如，以下操作会在导出时将所有文本对象转换为路径：

```
**$ inkscape --export-fileid=file.pdf --export-text-to-path file.svg**
```

生成的矢量文件既不需要也不嵌入任何字体。

## C.5 查询

由于 SVG 是基于文本的格式，生成或编辑 SVG 文档时常常会使用简单的脚本。在这样的脚本中，你可能需要指定某些 SVG 对象的边界框——例如，检查从数据库插入到 SVG 中的文本是否适合提供的空间，或者为特定对象创建一个背景矩形或框架。然而，通常情况下，计算 SVG 中对象的边界框是极其复杂的——你需要重新实现大量 Inkscape 的代码，以考虑可能影响对象边界框的所有因素。即使你在编写 Inkscape 扩展（第十九章），Inkscape 附带的 `inkex` Python 库也可以计算路径的边界框，但不能计算文本的边界框。

幸运的是，Inkscape 本身提供了帮助。例如：

```
**$ inkscape --query-width --query-id=text1256 file.svg**
45.2916
```

这会要求 Inkscape 提供具有 `id="text1256"` 的对象的宽度（单位为像素）。Inkscape 加载文档，找到该对象，将其宽度打印到控制台，然后退出。

类似地，你可以使用 `--query-height`、`--query-x` 和 `--query-y` 参数来查找对象边界框的尺寸和坐标。`--query-id` 参数可以包含以逗号分隔的 `id` 列表。

这样的 Inkscape 调用速度相对较快，因为它们不加载 GUI，也不渲染文档。然而，如果你需要多个对象的多个边界框值，使用这些参数可能会导致延迟。在这种情况下，最好使用`--query-all`参数，它会返回文档中所有对象的所有边界框值：

```
**$ inkscape --query-all file.svg**
svg2,-55.11053,-29.90404,328.3131,608.6359
layer1,-55.11053,-29.90404,328.3131,608.6359
image2372,-8.917463,349.8089,282.12,212.6382
text2317,-39.85518,454.3014,20.40604,13.32647
tspan2319,-32.58402,454.3014,12.79618,4.989286
tspan2408,-39.85518,462.4921,20.40604,5.135838
path2406,-16.43702,386.617,6.34172,154.7896
text2410,-46.11609,376.8753,34.34841,5.135838
tspan2414,-46.11609,376.8753,34.34841,5.135838
text2418,-55.11053,365.9197,43.02429,5.135838
```

每一行是一个以逗号分隔的对象 ID、X、Y、宽度和高度的列表。在你的脚本中解析这样的行应该很简单。

## C.6 操作

Inkscape 的命令行不仅仅限于无 GUI 的导出、转换和查询任务。你还可以使用 `--actions` 参数脚本化许多常规编辑任务，该参数允许你列出任意数量的动作，以供 Inkscape 按顺序执行。

每个操作或多或少对应于在 GUI 中编辑文档时从菜单中选择的命令。一个操作有一个 *名称* 和可选的一个或多个 *参数*。在命令行中，使用冒号（`:`）分隔名称和参数，用逗号（`,`）分隔单个操作的多个参数，用分号（`;`）分隔列表中的多个操作。你也可以在字符串中包含空格，但此时字符串必须用双引号（`"`...`"`) 括起来，并且不能包含其他引号。

要查看你的 Inkscape 版本支持的完整操作列表，可以使用 `--action-list` 启动 Inkscape。以下是该列表的顶部，显示一些与导出相关的命令行参数对应的操作：

```
**$ inkscape --action-list**
action-list         :  Print a list of actions and exit.
convert-dpi-method  :  Import DPI convert method.
export-area         :  Export area.
export-area-drawing :  Export drawing area.
export-area-page    :  Export page area.
export-area-snap    :  Export snap area to integer values.
export-background   :  Export background color.
export-background-opacity:  Export background opacity.
export-do           :  Do export.
...
```

### C.6.1 示例：更改 CSS 属性

让我们看一个实际的例子。假设你想打开 file.svg，选择所有文本对象，将它们涂成红色（`#FF0000`），并保存结果。你可以通过以下操作完成此任务：

```
**$ inkscape --actions="file-open:file.svg; select-by-element:text; \** **object-set-property:fill,#FF0000; export-filename:fileout.svg; export-do"**
```

让我们按分号将这个字符串拆分成各个部分，逐一查看每个操作：

+   `file-open` 打开其参数中给定的文件——在此例中是 `file.svg`。你也可以省略此操作，直接在 `--actions` 参数后提供 `file.svg`。

+   `select-by-element` 选择所有具有给定元素名称的对象——在此例中是 `text`。

+   `object-set-property` 在选中的对象上设置一个 CSS 属性（8.1）。两个参数分别提供属性名称（`fill`）和值（`#FF0000`）。这将把所有文本涂成红色（除非它们有自己的 `tspan` 内部定义的 `fill`，会覆盖此设置）。

+   `export-filename` 通过指定文件名（以及从文件扩展名中获取的 SVG 导出格式）来准备导出结果。目前没有 `file-save` 操作，这可能反而更好；将更改的结果保存（即导出）到一个不同的文件，而不是覆盖原文件，更为安全。

+   `export-do`（无参数）执行实际的导出操作。

这个命令执行完后，你将在当前文件夹中得到一个名为 fileout.svg 的文件，该文件与 file.svg 完全相同，只不过所有文本对象的填充颜色变成了红色。

### C.6.2 Shell 模式

Inkscape 的 Shell 模式是一个便捷的方式来探索操作。如果你使用 `--shell` 启动 Inkscape，它会进入交互模式，在该模式下你可以在 Inkscape 的提示符下输入操作及其参数（以及动词），并立即执行这些操作。例如：

```
**$ inkscape --shell**
Inkscape interactive shell mode. Type 'action-list' to list all actions. Type 'quit' to quit. Input of the form: action1:arg1; action2;arg2; verb1; verb2; ...
Only verbs that don't require a desktop may be used.**> file-open:page.svg****> select-by-element:text****> delete**
Unable to find: delete
verbs_action: Invalid verb: delete**> select-by-element:text****> object-set-property:fill,#00ff00****> export-filename:green.svg****> export-do****> quit**
```

你在交互式 Shell 模式中测试并验证有效的命令序列，可以在脚本中重复使用。

在输入操作名称或文件名时，按 Tab 键可进行自动补全。Inkscape 的 Shell 模式会记住你跨会话的命令历史。
