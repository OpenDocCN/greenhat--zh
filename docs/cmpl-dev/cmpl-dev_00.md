

# 前言



![](img/Drop-image.jpg)

几乎所有的编程职位如今都要求至少对全栈开发有一个粗略的了解，但如果你是初学者，你可能会很难找到切入这个庞大话题的正确入口。你甚至可能不知道这个术语的含义。

简单来说，*全栈 Web 开发*通常指的是使用 JavaScript 及其构建的各种框架来创建完整的 Web 应用。这要求精通前端和后端开发的传统学科，并且能够编写中间件和各种类型的应用程序接口（API）。

最后，一名全面的全栈开发人员能够处理数据库，并具备专业技能，比如能够编写自动化测试并自行部署代码。要做到这一点，他们必须理解 HTML、CSS 和 JavaScript，以及该语言的类型化对应物 TypeScript。有关这些术语的速成课程，请参见第 xxiv 页的《全栈应用的组成部分》。

如果这听起来很多，你来对地方了。本书将向你介绍现代应用的各个组成部分，并教你如何使用一些最广泛使用的技术来构建它们。

## 谁应该阅读这本书？

本书的主要读者有两类。一类是希望通过掌握全栈开发来提升职业生涯的前端或后端工程师；另一类是对 Web 开发感兴趣的初学者。

虽然本书从零开始介绍了许多技术，但它假设读者对 HTML、CSS 和 JavaScript 有一定的基础了解，并且了解大多数 Web 应用的客户端/服务器架构。作为复习，可以参考 Sam Taylor 的*《编码工作手册》*（No Starch Press, 2020），该书教你如何使用 HTML 和 CSS 构建网站；以及 Peter Gasston 的*《CSS3 书籍》*第二版（No Starch Press, 2014），来提升你的 CSS 技能。为了熟悉 JavaScript，我推荐 Nick Morgan 的*《JavaScript 速成课程》*（No Starch Press, 2024），这是一本为初学者设计的快速 JavaScript 教程；以及 Marijn Haverbeke 的*《Eloquent JavaScript》第三版*（No Starch Press, 2018），深入探讨 JavaScript。

## 本书内容

本书分为两部分。第一部分，包括第一章到第十章，将向你介绍现代技术栈的各个组成部分。每一章重点介绍一种技术，并强调作为全栈开发人员需要掌握的知识点。练习将鼓励你从第 1 页开始编写应用代码。

**第一章: Node.js**  介绍了 Node.js 及其生态系统，使你能够在浏览器外运行 JavaScript 代码。然后，你将使用 Node.js 和 Express.js 框架创建一个简单的 JavaScript 网页服务器。

**第二章: 现代 JavaScript**  聚焦于现代 JavaScript 语法，适用于全栈开发人员，包含如何使用模块编写可维护的代码包。我们将探讨定义变量和常量的不同方式、箭头函数以及异步代码的技巧。你将使用这些知识重写你的 JavaScript 服务器。

**第三章: TypeScript**  介绍了 TypeScript，这是一种 JavaScript 的超集，并强调了现代全栈开发如何从中受益。我们讨论了 JavaScript 的不足和陷阱，以及如何通过类型推断有效利用 TypeScript 的类型系统。最后，你将通过类型注解、自定义类型和接口重构你的 JavaScript 服务器。

**第四章: React**  讨论了 React，这是最常用的用户界面组件库之一。你将看到它的组件如何简化全栈开发，并学习如何使用其 JSX 元素、虚拟 DOM 和 Hooks。然后，你将使用 React 为你的 Express.js 服务器添加一个响应式用户界面。

**第五章: Next.js**  重点介绍了 Next.js，这是一个基于 React 构建的领先 Web 应用框架。你将使用 Next.js 的基于文件的路由创建页面和自定义 API 路由，之后学习在框架内渲染页面的不同方式。最后，你将进行一个练习，将 Express.js 服务器迁移到 Next.js。

**第六章: REST 和 GraphQL APIs**  教你关于 API 的所有知识，API 是什么，以及如何在全栈 Web 开发中使用它们。我们探索了两种类型的 API：REST 和 GraphQL。最后，你将通过向你的 Next.js 全栈应用添加 Apollo GraphQL 服务器来完成本章内容。

**第七章: MongoDB 和 Mongoose**  讨论了传统关系型数据库和非关系型数据库（如 MongoDB）之间的区别。你将把 Mongoose 对象数据建模工具添加到你的技术栈中，以简化数据库操作。接着，你将把 GraphQL API 连接到你自己的 MongoDB 数据库。

**第八章: 使用 Jest 框架进行测试**  解释了自动化测试和测试驱动开发对全栈开发的重要性。我们探索了不同类型的测试、常见的测试模式以及测试双胞胎、存根、假对象和模拟的概念。最后，你将使用 Jest 框架向你的 Next.js 应用添加一些基本的快照测试。

**第九章：OAuth 授权**  讨论身份验证和授权，以及全栈开发人员如何通过集成第三方服务使用 OAuth 协议来处理这些任务。我们将详细讲解此授权流程及其组件。你将通过命令行运行一次完整的 OAuth 交互，深入探讨每个步骤。

**第十章：使用 Docker 进行容器化**  介绍了如何使用 Docker 部署应用程序。我们首先讲解微服务架构的概念，然后介绍 Docker 生态系统的所有相关组件：主机、Docker 守护进程、Dockerfile、镜像、容器、卷和 Docker Compose。最后，你将通过将应用程序拆分为自包含的微服务来完成这部分内容。

在第二部分中，你将运用新学到的知识构建一个 Web 应用程序，应用第一部分中介绍的概念、工具和框架。Food Finder 应用是一个位置搜索服务，允许用户通过 GitHub 帐户登录并维护一个想要访问的地点愿望清单。

**第十一章：设置 Docker 环境**  通过运用你对 Docker 和容器化的知识，创建你的 Food Finder 应用的基础。你将使用 Docker Compose 将应用开发与本地系统解耦，然后添加一个作为独立服务的 MongoDB 服务器。

**第十二章：构建中间件**  创建 Food Finder 应用的第一个中间件部分。在这里，你将连接 Mongoose 到 MongoDB 服务，并创建其架构、模型、服务和自定义类型。有了这些组件，你将能够从数据库中创建、读取、更新和删除数据。

**第十三章：构建 GraphQL API**  运用你对 GraphQL 的知识，在 Food Finder 应用中添加一个 Apollo GraphQL 服务器，然后实现一个公共的 GraphQL API。你将能够使用 Apollo 沙盒来读取和更新 MongoDB 服务器上的数据。

**第十四章：构建前端**  使用 React 组件和 Next.js 框架构建 Food Finder 应用的前端。在这一阶段，你将实现一个完整的现代全栈应用程序，通过自定义中间件从数据库读取数据并将数据呈现到应用的前端。

**第十五章：添加 OAuth**  向你的应用程序添加 OAuth 流程，让访客能够登录并维护个人地点愿望清单。你将使用*next-auth*包从 Auth.js 中添加通过 GitHub 的登录选项。

**第十六章：在 Docker 中运行自动化测试**  使用 Jest 设置自动化快照测试，并配置一个新的服务来自动运行这些测试。

然后，在附录中，你将获得关于 TypeScript 编译器选项和最常见 Jest 匹配器的详细信息。此外，你还将运用你新获得的知识，探索并理解 Next.js 的现代应用程序目录方法。

**附录 A：TypeScript 编译器选项**展示了最常见的 TypeScript 编译器（TSC）选项，以便你可以根据个人喜好自定义自己的 TypeScript 项目。

**附录 B：Next.js 应用程序目录**探索了 Next.js 在版本 13 中引入的使用*app*目录的新路由模式。你可以选择继续使用传统的页面方法（详见第五章），或者在即将到来的项目中使用现代的*app*目录。

**附录 C：常见的匹配器**展示了用于使用 Jest 和 Jest DOM 测试应用程序的最常见匹配器。

## 全栈应用程序的各个部分

在本书中，我们将讨论应用程序的各个部分。本节为你提供一个速成课程，讲解当我们使用术语*前端*、*中间件*和*后端*时的含义。

### 前端

前端是网站或 Web 应用程序的面向用户部分。它运行在客户端，通常是一个 Web 浏览器。你可以将其视为 Web 应用程序的“前台”。例如，在 [*https://<wbr>www<wbr>.google<wbr>.com*](https://www.google.com)上，前端是一个带有简单搜索栏的页面，当然，前端开发可能比这更复杂；看看谷歌的搜索结果页面或你最近访问的最后一个网站的界面。

前端开发者专注于用户参与、体验和界面。他们依赖 HTML 来创建网站界面的元素，CSS 用于样式，JavaScript 用于用户交互，以及 Next.js 等框架来将所有内容结合在一起。

### 中间件

中间件连接应用程序的前端和后端，并执行所有任务，例如与第三方服务的集成、数据的传输和更新。你可以将其看作是公司楼层上的员工。

作为全栈开发者，我们经常为*路由*应用程序编写中间件，这意味着为特定 URL 提供正确的数据，处理数据库连接并执行授权。例如，在 [*https://<wbr>www<wbr>.google<wbr>.com*](https://www.google.com)上，中间件会向服务器请求登录页面的 HTML。然后，另一部分中间件会检查用户是否已登录，如果已登录，应该显示哪些个人数据。与此同时，第三部分中间件会整合这些数据流中的信息，然后以正确的 HTML 响应服务器的请求。

一个全栈应用程序的中间件的一个重要部分是它的 *API 层*，该层公开了应用程序的 API。通常，API 是用来连接两台机器的代码。通常，API 让前端代码（或第三方）访问应用程序的后端。由 JavaScript 驱动的开发依赖于两种主要的架构框架来创建 API：REST 和 GraphQL，二者在第六章中有详细介绍。

你可以使用任何编程语言来编写中间件。大多数全栈开发者使用现代 JavaScript 或 TypeScript，但他们也可以选择使用 PHP、Ruby 或 Go。

### 后端

后端是 Web 应用程序中看不见的部分。在一个由 JavaScript 驱动的应用程序中，后端运行在服务器上，通常是 Express.js，尽管其他人可能使用 Apache 或 NGINX。你可以把它看作是 Web 应用程序的“后台”部分。

更具体地说，后端处理涉及应用程序数据的任何操作。它对存储在数据库中的值执行创建、读取、更新和删除（CRUD）操作，并通过中间件的 API 层返回用户请求的数据集。对于[*https://<wbr>www<wbr>.google<wbr>.com*](https://www.google.com)，后端是用来搜索数据库中你在前端输入的关键词的代码，这些关键词通过中间件传递给后端。中间件将这些搜索结果与其他相关信息结合起来。然后，用户将在前端呈现的搜索结果页面中看到这些内容。

后端开发可以使用任何编程语言进行。全栈开发者通常使用现代 JavaScript 或 TypeScript。其他选择包括 PHP、Ruby、Elixir、Python、Java 以及像 Symfony、Ruby on Rails、Phoenix 和 Django 这样的框架。

## JavaScript 和全栈开发的简史

所有开发者都应该理解他们所使用工具的背景。在我们开始开发之前，让我们先了解一点历史。

全栈开发者职位是与 JavaScript 一同发展的，JavaScript 最初只不过是一个在用户浏览器中运行的脚本语言。开发者使用它来为网站添加元素，如手风琴、弹出菜单和覆盖层，这些元素会根据用户的行为立即响应，而无需向应用程序的服务器发出请求。

直到 2000 年代末，大多数 JavaScript 库的设计都是为了提供一致的接口，以处理供应商特定的特殊情况。通常，JavaScript 引擎的速度较慢，特别是在与 HTML 交互、更新或修改时。因此，JavaScript 曾被视为一个有些怪异的前端脚本语言，并且不被后端开发者所看好。

几个项目曾试图普及 JavaScript 在后端的应用，但直到 2009 年 Node.js 发布之前，这些尝试都没有取得显著进展。Node.js（在第一章中讨论）是一个用于开发后端的 JavaScript 工具。随后，Node.js 的包管理器 npm 构建了全栈 JavaScript 开发所需的生态系统。

这个生态系统包括了一系列用于处理数据库、构建用户界面和编写服务器端代码的 JavaScript 库（我们将在本书中探讨其中的许多）。这些新工具使得开发人员可以在客户端和服务器端可靠地使用 JavaScript。特别重要的是，谷歌于 2010 年发布了 Angular 框架，Meta（当时被称为 Facebook）于 2013 年发布了 React。互联网巨头们致力于构建 JavaScript 工具，使得全栈 Web 开发成为一个备受追捧的职位。

## 设置

在本书中，您将编写代码并运行命令行工具。您可以使用任何开发环境，但以下是一些指导建议。

目前最常见的代码编辑器是 Visual Studio Code，您可以从[*https://<wbr>code<wbr>.visualstudio<wbr>.com*](https://code.visualstudio.com)下载。它是微软的开源编辑器，适用于 Windows、macOS 和 Linux，且免费。此外，您可以通过大量第三方插件扩展和配置它，并根据个人喜好调整外观。不过，如果您习惯使用其他编辑器，比如 Vim 或 Emacs，您也可以继续使用。本书并不要求使用特定的工具。

根据您的操作系统，默认的命令行程序可能是*命令提示符*（Windows）或*终端*（macOS 和 Linux）。这些程序在执行诸如创建、修改和列出目录内容等任务时，使用略有不同的语法。本书展示的是 Linux 和 macOS 版本的命令。如果您使用的是 Windows，您需要根据操作系统调整命令。例如，Windows 使用dir来列出当前目录中的文件和文件夹，而不是 Linux 中的<code>ls</code>。微软的官方命令行参考文档列出了所有可用的命令，您可以在这里查看：[ *https://<wbr>learn<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/windows<wbr>-server<wbr>/administration<wbr>/windows<wbr>-commands<wbr>/windows<wbr>-commands#command<wbr>-line<wbr>-reference<wbr>-a<wbr>-z*](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands#command-line-reference-a-z)。

本书中与操作系统相关的最显著区别是多行 cURL 命令中用于换行的转义字符。这个转义字符在 macOS 中是\，而在 Windows 中是^。我们将在第六章中指出这些区别，当我们首次使用 cURL 时。

你可以从[*https://<wbr>www<wbr>.usemodernfullstack<wbr>.dev<wbr>/downloads*](https://www.usemodernfullstack.dev/downloads)下载本书第一部分的代码清单以及 Food Finder 应用程序的完整源代码。
