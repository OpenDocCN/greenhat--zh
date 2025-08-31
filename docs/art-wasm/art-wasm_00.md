# 前言

![](img/chapterart.png)

欢迎来到*WebAssembly 的艺术*。本书教你如何在虚拟机级别阅读、编写和理解 WebAssembly。它将帮助你了解 WebAssembly 如何与 JavaScript、网页浏览器以及嵌入环境进行交互。到最后，你将理解 WebAssembly 是什么，它的理想使用场景，以及如何编写接近本地速度的 WebAssembly 代码。

## 谁应该阅读本书

本书适合那些希望理解何时以及为何使用 WebAssembly 的网页开发者。如果你真心想掌握 WebAssembly，你需要深入学习它。关于 WebAssembly 工具链，已有多本书籍进行讨论。本书并不专注于为 WebAssembly 编写 C/C++、Rust 或其他语言的代码；相反，它探索了 WebAssembly 的机制和能力。

本书适合那些想要了解 WebAssembly 是什么，它能做什么以及如何最佳使用它的用户。WebAssembly 可以比 JavaScript 表现得更好，且可以创建更小的下载和内存占用。但开发高性能的 WebAssembly 应用程序不仅仅是用 C++/Rust 或 AssemblyScript 等语言编写应用程序并将其编译为 WebAssembly。要构建一个执行速度是其 JavaScript 等效程序两到三倍的应用程序，你需要深入了解 WebAssembly 的工作原理。

读者应具备基本的网页技术知识，如 JavaScript、HTML 和 CSS，但不需要是这些技术的专家。在目前的 WebAssembly 形式下，如果不了解网页及其工作原理，使用 WebAssembly 并不容易。我不会解释网页的基础知识，但我也不假设读者对网页如何运作有太多了解。

## 为什么用户对 WebAssembly 感兴趣

在第一次 WebAssembly 峰会上，Ashley Williams（[@ag_dubs](http://www.twitter.com/@ag_dubs)）展示了她在 Twitter 上发起的调查结果，询问 WebAssembly 用户为什么对这项技术感兴趣。以下是结果：

+   多语言，40.1 百分

+   更小更快的代码，36.8 百分

+   沙盒化（安全），17.3 百分

然后，她询问那些对 WebAssembly 支持多种语言感兴趣的用户，为什么会这样：

+   JavaScript 无法满足我的需求，43.5 百分

+   重用现有库，40.8 百分

+   预先存在的应用分发（distribution），8.1 百分

对于那些认为 JavaScript 无法满足其需求的用户，她询问了原因：

+   性能差或不一致，42 百分

+   生态系统无法满足我的需求，17.4 百分

+   我不喜欢或理解它，31.3 百分

你可以在 YouTube 上观看她的演讲，“Why the #wasmsummit Website Isn’t Written in Wasm”，网址：[`www.youtube.com/watch?v=J5Rs9oG3FdI`](https://www.youtube.com/watch?v=J5Rs9oG3FdI)。

尽管这些调查并非科学性调查，但它们仍然提供了相当有启发性的见解。首先，如果你将第一和第三次调查中那些有兴趣使用 WebAssembly 提升应用性能的用户结合起来，总数超过了 55%。毫无疑问，使用 WebAssembly 提升代码性能是可能的。但要真正利用 WebAssembly 并非魔法；你只需要知道自己在做什么。到本书结束时，你将掌握足够的 WebAssembly 知识，以显著提升你 web 应用的性能。

## 为什么世界需要 WebAssembly

我从 1990 年代中期开始开发 web 应用程序。最初，网页不过是带有图片的文档。随着 Java 和 JavaScript 的出现，这一情况发生了变化。那时，JavaScript 是一种玩具语言，只能为网页上的按钮添加鼠标悬停效果。Java 才是真正的技术，而 Java 虚拟机（JVM）则是令人兴奋的技术。但是，Java 从未在网页平台上发挥出其全部潜力。Java 需要插件，而插件技术最终因其安全性问题和恶意软件威胁而过时。

不幸的是，Java 是一项专有技术，这阻止了它直接集成到网页浏览器中。然而，WebAssembly 不同，因为它不是由单一技术公司单方面创建的。WebAssembly 起初是由许多硬件和软件供应商（如 Google、Mozilla、Microsoft 和 Apple）合作推出的。它在每个现代浏览器中都可以直接使用，无需插件。你可以使用它通过 Node.js 编写硬件独立的软件。由于它不是专有的，任何硬件或软件平台都可以使用它，无需支付版权费或获得许可。它实现了 1990 年代的梦想——*一个二进制文件统治一切*。

## 本书内容

本书将带领你了解 WebAssembly 如何在低层次上工作，通过介绍 WebAssembly 文本格式来实现。我们将讨论许多低层次的主题，并花一些时间展示 WebAssembly 如何在 Node.js 和基于 web 的应用程序中与 JavaScript 协同工作。本书的阅读顺序是有意设计的，概念之间相互构建。书中还将有指向代码示例的引用，这些示例可以在 [`wasmbook.com`](https://wasmbook.com) 找到。

**第一章：WebAssembly 简介**

1.  我们将详细探讨 WebAssembly 是什么，它不是什麽，以及什么时候最好使用它。你将接触到 WebAssembly 文本（WAT），它让你理解 WebAssembly 如何在最低层次上运作。我们还将设置你将用来跟随本书示例的环境。

**第二章：WebAssembly 文本基础**

1.  我们将介绍 WAT 的基础知识，以及它如何与部署到 WebAssembly 的高级语言相关。你将编写你的第一个 WAT 程序，并讨论一些基础概念，如变量使用和控制流。

**第三章：函数和表**

1.  我们将讨论如何在 WebAssembly 模块中创建函数并从 JavaScript 调用它们。你将构建一个检查素数的程序来说明这些概念。我们还将探讨从表格中调用函数以及性能影响。

**第四章：低级位操作**

1.  你将学习可以用来提升 WebAssembly 模块性能的低级概念，例如数字系统、位掩码和 2 的补码。

**第五章：WebAssembly 中的字符串**

1.  WebAssembly 并没有内建的字符串数据类型，因此在本章中，你将学习字符串如何在 WebAssembly 中表示，以及如何操作它们。

**第六章：线性内存**

1.  你将了解线性内存以及 WebAssembly 模块如何使用它与 JavaScript 或其他嵌入环境共享大型数据集。我们开始创建一个物体碰撞程序，让物体随机移动并检测碰撞，之后我们将在整本书中使用它。

**第七章：Web 应用程序**

1.  你将学习如何使用 HTML、CSS、JavaScript 和 WebAssembly 创建一个简单的 Web 应用程序。

**第八章：与 Canvas 一起工作**

1.  我们将讨论如何使用 HTML canvas 和 WebAssembly 创建极速的 Web 动画。我们使用 canvas 来优化我们的物体碰撞应用程序。

**第九章：优化性能**

1.  你将学习 WebAssembly 如何在计算密集型任务中表现出色，例如碰撞检测。你将花一些时间使用 Chrome 和 Firefox 的性能分析工具以及其他优化工具来提升应用程序的性能。

**第十章：调试 WebAssembly**

1.  我们将介绍调试基础知识，例如使用警告和堆栈跟踪记录到控制台。你还将学习如何使用 Chrome 和 Firefox 中的调试工具逐步调试 WebAssembly 代码。

**第十一章：AssemblyScript**

1.  我们将讨论如何使用 WAT 来理解高级语言，通过使用它来评估 AssemblyScript，这是一种旨在高效部署到 WebAssembly 的高级语言。
