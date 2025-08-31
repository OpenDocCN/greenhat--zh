# 第十三章：公开你的作品

![](img/chapterart.png)

## 草图 100：在网页上的处理

Processing 草图通常可以在浏览器中执行，几乎不需要修改，就能创建动态和交互式的网页对象。实现这一功能的系统是 *Processing.js*；它将 Processing 草图转换成 JavaScript 代码后再运行，并将结果显示在 HTML5 画布中。

在网页上运行草图有四个步骤：

1.  下载 Processing.js。这意味着你需要访问像 [`processingjs.org/download/`](https://processingjs.org/download/) 这样的网站，下载 *processing.js* 和 *processing.min.js* 文件。

1.  创建 Processing 草图。我们将使用草图 91，即极光模拟，作为示例。这个草图将命名为 *sketch100.pde*。

1.  创建一个网页，在其中嵌入草图。该网页必须在页面的头部加载 *processing.min.js* 作为脚本 2：

    ```
    <script src="processing.min.js"></script>
    ```

1.  创建一个画布，指定 *sketch100.pde* 作为数据处理源 3：

    ```
    <canvas data-processing-sources="sketch100.pde"> </canvas>
    ```

这只有在网页服务器上才能正常工作，因此你需要将所有文件上传到服务器，并从互联网上显示页面，或者在你的电脑上安装一个服务器。

所有三个文件——HTML 源文件、草图和*processing.min.js*——应该放在网页服务器的同一个目录下。当页面加载时，草图应当运行并在画布中显示结果。

根据草图的不同，可能会有其他问题。首先，如果草图使用了图像，这些图像必须被预加载，以便在草图运行时能够获取它们的大小和其他属性。`preload` 指令必须出现在草图开头的注释中。例如，在这个例子中，使用了 `trees.gif` 和 `stars.jpg` 文件 1：

```
/* @pjs preload="trees.gif, stars.jpg"; */
```

接下来，如果草图使用了整数，要小心。Processing 代码会被转换成 JavaScript，而 JavaScript 并没有整数类型。整数将变成浮点数。任何依赖于整数运算的程序（例如 5/2 = 2）将无法正常工作。

任何需要 Java 库的程序也无法工作。Minim 是一个 Java 库，视频类也是如此。这些库有 JavaScript 版本，但使用它们将需要学习 JavaScript 的工作原理，以及如何从 Processing 访问 JavaScript，反之亦然。

网页的 HTML 代码位于下一页草图代码之后。
