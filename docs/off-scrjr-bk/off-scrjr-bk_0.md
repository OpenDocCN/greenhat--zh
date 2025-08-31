## 前言

本书的历史可以追溯到 50 年前，1960 年代，西摩·帕普特提出了一个大胆的全新愿景，展示了计算机如何进入儿童的生活。当时，计算机的价格仍然高达数万美元，甚至更多。第一台个人计算机要再过十年才能进入市场。但西摩预见到，计算机最终将变得人人可及，甚至儿童也能使用，并且他为计算如何改变儿童的学习和玩耍方式奠定了理论基础。

虽然其他研究人员认为计算机有一天可能被用来向孩子们提供信息或向他们提问，但西摩（Seymour）却有着截然不同的愿景。他认为孩子们应该掌控计算机，利用它们进行实验、探索和自我表达。西摩和他在麻省理工学院的同事们专门为孩子们开发了编程语言 Logo，让孩子们能够编程创作自己的图画、故事和游戏。在他 1980 年的著作《*心智风暴：儿童、计算机与强大的思想*》中，西摩主张，孩子们应该编程，而不是计算机来编程。

本书的两位作者（玛丽娜和米奇）在 MIT 攻读研究生期间与西摩密切合作，并深受他的思想影响。我们两人都将自己的职业生涯奉献给延续西摩的工作，为年轻人提供与新技术一起设计、创造和发明的机会。

米奇的终身幼儿园研究小组在 MIT 媒体实验室与乐高集团密切合作，开发了 MINDSTORMS 和 WeDo 机器人套件，并共同创办了为低收入社区青少年提供课外学习的计算机俱乐部网络。最近，该小组还开发了 Scratch 编程语言和在线社区，全球数百万 8 岁及以上的年轻人正在使用这一工具。

玛丽娜（Marina）在塔夫茨大学艾略特·皮尔森儿童研究与人类发展系的开发技术（DevTech）研究小组，专注于学前教育，开发适用于学前班、幼儿园和初等教育阶段学生的技术与活动。该小组开发了最终促成 KIBO 机器人套件的理念和原型，使得 4 到 7 岁的孩子可以通过拼接木块序列来编程机器人项目。通过 KIBO，孩子们可以在没有屏幕或键盘的情况下学习编程。2013 年，玛丽娜共同创立了 KinderLab Robotics，使 KIBO 广泛可用。玛丽娜及 DevTech 研究小组的工作理念在玛丽娜的书籍《*从积木到机器人：在学前班课堂中利用技术学习*》（教师学院出版社，2007 年）和《*为积极青少年发展设计数字体验：从游戏区到操场*》（牛津大学出版社，2012 年）中有所阐述。

在 2010 年，Marina 提出了让我们的两个团队合作开发一个面向年轻孩子的编程语言的建议，延续 MIT 在 Scratch 上的工作，并借鉴塔夫茨大学在儿童早期学习方面的经验，从而诞生了 ScratchJr 的想法。我们与 Playful Invention Company (PICO) 的 Paula Bontá 和 Brian Silverman 携手合作，他们在为儿童设计和开发编程语言方面拥有强大的专业知识（并且曾与 Seymour Papert 紧密合作过）。ScratchJr 是一个真正的团队合作成果，得到了塔夫茨大学、MIT、PICO 及其他地方许多人的贡献。我们鼓励你访问 ScratchJr 网站（ *[`www.scratchjr.org/`](http://www.scratchjr.org/)* ）以查看完整的贡献者名单。

我们非常高兴地看到来自世界各地成千上万的孩子、家长和教师对 ScratchJr 的回应，但我们也意识到，仍需要更多更好的支持材料来帮助人们充分发挥 ScratchJr 的潜力。我们写这本书是为了支持在家庭和学校中使用 ScratchJr。我们希望你能觉得这本书有用，也期待收到你的反馈和建议。

我们要感谢参与本书研究、写作和制作的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感激我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。在整个写书和出版的过程中，他们提供了无价的帮助和建议。

如果没有来自国家科学基金会（资助编号：DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 将不可能实现。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

享受吧！

Marina 和 Mitch

这本书的历史可以追溯到 50 年前，1960 年代，Seymour Papert 提出了一个大胆的新视野，探索计算机如何进入孩子们的生活。当时，计算机仍然昂贵，价格动辄上万美元，甚至更多。第一台个人计算机要再过十年才能商业化推出。但 Seymour 预见到，计算机最终会变得对每个人——甚至是孩子们——都能接触到，他为计算机如何改变孩子们的学习和游戏方式奠定了智力基础。

尽管其他研究者设想计算机可能有一天会用来向儿童传递信息或向儿童提问，Seymour 却有着完全不同的愿景。他认为，儿童应该掌控计算机，利用它们进行实验、探索和自我表达。Seymour 和 MIT 的同事们专为儿童开发了编程语言 Logo，使儿童能够编程自己的图片、故事和游戏。在他 1980 年的书籍*《思维风暴：儿童、计算机与强大思想》*中，Seymour 主张，儿童应当编程计算机，而不是相反。

我们两位写这本书的人（Marina 和 Mitch）在研究生时期曾与 Seymour 紧密合作，我们的思想深受他的影响。我们两人都将职业生涯献给了延续 Seymour 的工作，为年轻人提供使用新技术进行设计、创造和发明的机会。

MIT 媒体实验室的 Mitch 终身幼儿园研究小组与乐高集团紧密合作，开发了 MINDSTORMS 和 WeDo 机器人套件，并共同创立了为低收入社区青少年提供课后学习的计算机俱乐部网络。最近，该小组开发了 Scratch 编程语言和在线社区，全球数百万年轻人（8 岁及以上）都在使用它。

Marina 在塔夫茨大学 Eliot-Pearson 儿童研究与人类发展系的开发技术（DevTech）研究小组，专注于幼儿学习，开发适用于学前、小学和早期小学阶段的学生的技术和活动。该小组开发了 KIBO 机器人套件的理念和原型，KIBO 可以让 4 至 7 岁的儿童通过组合木块序列来编程机器人项目。使用 KIBO，儿童无需屏幕或键盘就能学习编程。2013 年，Marina 共同创办了 KinderLab Robotics，以使 KIBO 得到广泛应用。Marina 及 DevTech 研究小组工作背后的理念，已在 Marina 的书籍*《从积木到机器人：在早期儿童课堂中学习技术》*（教师学院出版社，2007 年）和*《为积极青少年发展设计数字体验：从游戏场到操场》*（牛津大学出版社，2012 年）中有所描述。

2010 年，Marina 提议我们两组合作，开发一种适合小孩子的编程语言，扩展麻省理工学院在 Scratch 方面的工作，并借鉴塔夫茨大学在早期儿童教育方面的经验，ScratchJr 的构思因此诞生。我们与 Playful Invention Company（PICO）的 Paula Bontá和 Brian Silverman 合作，他们在为儿童设计和开发编程语言方面有着丰富的专业经验（他们也曾与 Seymour Papert 有过紧密合作）。ScratchJr 是一次真正的团队合作，得到了塔夫茨大学、麻省理工学院、PICO 以及其他地方许多人的贡献。我们鼓励您访问 ScratchJr 网站（*[`www.scratchjr.org/`](http://www.scratchjr.org/)*），查看完整的贡献者名单。

我们非常高兴看到来自世界各地成千上万的儿童、家长和教师对 ScratchJr 的反馈，但我们也认识到，需要更多、更好的支持材料，帮助人们充分发挥 ScratchJr 的潜力。我们写这本书是为了支持在家庭和学校中使用 ScratchJr。我们希望您能觉得这本书有用，期待收到您的反馈和建议。

我们要感谢在本书的研究、写作和制作过程中给予帮助的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。在整个写作和出版过程中，他们提供了宝贵的帮助和建议。

如果没有来自美国国家科学基金会（资助编号 DRL-1118664）和 Scratch 基金会的慷慨财政支持，ScratchJr 是不可能实现的。如果您喜欢这本书和 ScratchJr，我们希望您能考虑向 Scratch 基金会捐款（*[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)*），以支持 ScratchJr 软件和教育材料的未来发展。

祝您愉快！

Marina 和 Mitch

虽然其他研究人员设想过，计算机有一天可能用于向儿童传递信息或向儿童提问，但 Seymour 的看法截然不同。他认为，儿童应该控制计算机，利用它们进行实验、探索和表达自己。Seymour 和他在麻省理工学院的同事们专门为儿童开发了编程语言 Logo，使孩子们可以编写自己的图片、故事和游戏。在他 1980 年出版的《*Mindstorms: Children, Computers, and Powerful Ideas*》一书中，Seymour 认为，孩子们应该是编程计算机的人，而不是相反。

本书的两位作者（Marina 和 Mitch）在我们还是麻省理工学院的研究生时，与 Seymour 密切合作，并深受他的思想影响。我们俩都将自己的职业生涯献给了扩展 Seymour 的工作，为年轻人提供利用新技术进行设计、创作和发明的机会。

MIT 媒体实验室的 Mitch 终身幼儿园研究小组与 LEGO 集团紧密合作，开发了 MINDSTORMS 和 WeDo 机器人套件，并共同创办了计算机俱乐部网络，为低收入社区的青少年提供课后学习中心。更近期，该小组开发了 Scratch 编程语言和在线社区，全球数百万年轻人（8 岁及以上）使用它。

Marina 在塔夫茨大学 Eliot-Pearson 儿童学习与人类发展系的开发技术（DevTech）研究小组专注于早期儿童学习，开发适用于学前、幼儿园和早期小学学生的技术和活动。该小组开发了导致 KIBO 机器人套件的理念和原型，KIBO 使得年轻儿童（4 至 7 岁）可以通过组合木块序列来编程机器人项目。使用 KIBO，孩子们可以在没有屏幕或键盘的情况下学习编程。2013 年，Marina 共同创办了 KinderLab Robotics，以便广泛推广 KIBO。Marina 和 DevTech 研究小组的工作理念描述在 Marina 的两本书中：《*Blocks to Robots: Learning with Technology in the Early Childhood Classroom*》（Teachers College Press，2007 年）和《*Designing Digital Experiences for Positive Youth Development: From Playpen to Playground*》（Oxford University Press，2012 年）。

2010 年，Marina 建议我们两个团队合作开发一款面向儿童的编程语言，延续 MIT 在 Scratch 上的工作，并结合塔夫茨大学在幼儿教育方面的经验，从而诞生了 ScratchJr 的构想。我们与 Playful Invention Company (PICO)的 Paula Bontá和 Brian Silverman 合作，他们在儿童编程语言设计和开发方面拥有丰富的经验（并且曾与 Seymour Papert 密切合作）。ScratchJr 是一个真正的团队合作成果，得到了塔夫茨大学、MIT、PICO 及其他地方许多人的贡献。我们鼓励你访问 ScratchJr 网站（ *[`www.scratchjr.org/`](http://www.scratchjr.org/)* ）查看完整的贡献者名单。

我们对来自全球成千上万的儿童、家长和教师对 ScratchJr 的反馈感到兴奋，但我们也意识到，需要更多更好的支持材料来帮助人们充分发挥 ScratchJr 的潜力。我们写这本书是为了支持 ScratchJr 在家庭和学校中的使用。我们希望你能觉得这本书有用，并期待听到你的反馈和建议。

我们要感谢 ScratchJr 团队的成员，他们在本书的研究、写作和制作过程中提供了帮助，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还要特别感谢我们的出版商 No Starch Press 的 Tyler Ortman 和 Serena Yang。他们在书籍编写和出版过程中提供了无价的帮助和建议。

ScratchJr 的开发离不开来自国家科学基金会（资助号 DRL-1118664）和 Scratch 基金会的慷慨资助。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（*[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)*），以支持 ScratchJr 软件和教育材料的未来发展。

祝你玩得开心！

Marina 和 Mitch

我们两位撰写本书（Marina 和 Mitch）曾在麻省理工学院读研究生时与 Seymour 紧密合作，深受他思想的影响。我们俩都将自己的职业生涯奉献给了延续 Seymour 的工作，为年轻人提供通过新技术进行设计、创造和发明的机会。

Mitch 在麻省理工学院媒体实验室的终身幼儿园研究小组与 LEGO 集团密切合作，参与了 MINDSTORMS 和 WeDo 机器人套件的开发，并共同创办了为低收入社区青少年提供课后学习机会的计算机俱乐部网络。最近，该小组还开发了 Scratch 编程语言和在线社区，全球有数百万 8 岁及以上的青少年在使用这款工具。

Marina 在塔夫茨大学埃利奥特-皮尔森儿童学习与人类发展系领导的开发技术（DevTech）研究小组，专注于早期儿童学习，为学前班、幼儿园和早期小学生开发技术和活动。该小组开发了 KIBO 机器人套件的构思和原型，KIBO 使得年轻儿童（4-7 岁）可以通过拼接木块的顺序来编程机器人项目。使用 KIBO，孩子们无需屏幕或键盘就能学习编程。2013 年，Marina 联合创办了 KinderLab Robotics 公司，旨在让 KIBO 得到广泛应用。Marina 和 DevTech 研究小组的工作理念详细描述在 Marina 的两本书中：《Blocks to Robots: Learning with Technology in the Early Childhood Classroom》（教师学院出版社，2007 年）和《Designing Digital Experiences for Positive Youth Development: From Playpen to Playground》（牛津大学出版社，2012 年）。

2010 年，Marina 建议我们的两组团队合作，开发一种适合小孩子的编程语言，扩展麻省理工学院的 Scratch 项目，并借鉴塔夫茨大学在早期儿童学习方面的经验，从而诞生了 ScratchJr 的构思。我们与 Playful Invention Company (PICO) 的 Paula Bontá 和 Brian Silverman 合作，他们在为儿童设计和开发编程语言方面拥有丰富的专业知识（并且也曾与 Seymour Papert 紧密合作）。ScratchJr 是一个真正的团队合作成果，得到了塔夫茨大学、麻省理工学院、PICO 以及其他地方许多人的贡献。我们鼓励你访问 ScratchJr 网站（*[`www.scratchjr.org/`](http://www.scratchjr.org/)*）以查看完整的贡献者名单。

我们对全球成千上万的孩子、家长和教师对 ScratchJr 的反响感到非常兴奋，但我们也意识到需要更多、更好的支持材料，帮助人们充分发挥 ScratchJr 的潜力。我们写这本书是为了支持在家庭和学校中使用 ScratchJr。我们希望你能觉得这本书有用，期待收到你的反馈和建议。

我们要感谢 ScratchJr 团队的成员，他们在本书的研究、写作和制作过程中提供了帮助，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。他们在整个写作和出版过程中提供了宝贵的帮助和建议。

如果没有来自国家科学基金会（资助号：DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

祝阅读愉快！

Marina 和 Mitch

麻省理工学院媒体实验室的 Mitch 终身幼儿园研究小组与乐高集团密切合作，开发了 MINDSTORMS 和 WeDo 机器人套件，并共同创办了面向低收入社区青少年的课后学习中心——计算机俱乐部网络。最近，该小组开发了 Scratch 编程语言和在线社区，全球有数百万年轻人（8 岁及以上）正在使用该平台。

Marina 在塔夫茨大学 Eliot-Pearson 儿童研究与人类发展系的开发技术（DevTech）研究小组，专注于早期儿童学习，开发适用于学前班、幼儿园和初级小学学生的技术和活动。该小组开发了 KIBO 机器人套件的理念和原型，KIBO 让 4 至 7 岁的孩子通过拼接木块序列来编程机器人项目。使用 KIBO，孩子们在没有屏幕或键盘的情况下学习编程。2013 年，Marina 共同创办了 KinderLab Robotics，使 KIBO 能够广泛使用。Marina 和 DevTech 研究小组的工作理念详细描述在 Marina 的两本书中：《Blocks to Robots: Learning with Technology in the Early Childhood Classroom》（教师学院出版社，2007 年）和《Designing Digital Experiences for Positive Youth Development: From Playpen to Playground》（牛津大学出版社，2012 年）。

2010 年，Marina 建议我们两个团队合作，开发一种面向儿童的编程语言，扩展 MIT 在 Scratch 上的工作，并借鉴塔夫茨大学在幼儿教育方面的经验，从而诞生了 ScratchJr 的构想。我们与 Playful Invention Company (PICO)的 Paula Bontá和 Brian Silverman 合作，他们在儿童编程语言的设计和开发方面拥有深厚的专业知识（并且也曾与 Seymour Papert 紧密合作）。ScratchJr 是一个真正的团队合作成果，得到了塔夫茨大学、MIT、PICO 以及其他地方许多人的贡献。我们鼓励你访问 ScratchJr 网站（ *[`www.scratchjr.org/`](http://www.scratchjr.org/)* ）查看完整的贡献者名单。

我们对全球成千上万的孩子、家长和老师对 ScratchJr 的回应感到非常激动，但我们也认识到，需要更多、更好的支持材料来帮助人们充分利用 ScratchJr。我们写这本书是为了支持在家庭和学校中使用 ScratchJr。我们希望你觉得这本书有用，并期待听到你的反馈和建议。

我们要感谢参与本书研究、写作和制作的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感激出版商 No Starch Press 的 Tyler Ortman 和 Serena Yang。在整个写作和出版过程中，他们提供了宝贵的帮助和建议。

如果没有国家科学基金会（资助编号 DRL-1118664）和 Scratch 基金会的大力支持，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来开发。

祝你愉快！

Marina 和 Mitch

Marina 在塔夫茨大学 Eliot-Pearson 儿童研究与人类发展系的早期技术（DevTech）研究小组，专注于幼儿学习，开发适用于学前班、幼儿园和小学早期学生的技术和活动。该小组开发了思想和原型，促成了 KIBO 机器人套件的诞生，KIBO 使得年轻孩子（年龄 4-7 岁）能够通过拼接木块序列来编程机器人项目。使用 KIBO，孩子们可以在没有屏幕或键盘的情况下学习编程。2013 年，Marina 共同创办了 KinderLab Robotics，使 KIBO 能够广泛使用。Marina 及其 DevTech 研究小组的工作理念，已在 Marina 的书籍《Blocks to Robots: Learning with Technology in the Early Childhood Classroom》（教师学院出版社，2007 年）和《Designing Digital Experiences for Positive Youth Development: From Playpen to Playground》（牛津大学出版社，2012 年）中进行了描述。

2010 年，Marina 建议我们的两个小组合作，开发一种适合小孩子的编程语言，扩展麻省理工学院在 Scratch 上的工作，同时结合塔夫茨大学在早期儿童学习方面的经验，于是 ScratchJr 的构想诞生了。我们与 Playful Invention Company（PICO）的 Paula Bontá和 Brian Silverman 合作，他们在为儿童设计和开发编程语言方面拥有丰富的专业知识（同时也与 Seymour Papert 密切合作过）。ScratchJr 是一个真正的团队合作项目，得到了塔夫茨大学、麻省理工学院、PICO 及其他地方许多人贡献的力量。我们鼓励你访问 ScratchJr 官网（*[`www.scratchjr.org/`](http://www.scratchjr.org/)*），了解完整的贡献者名单。

我们对来自世界各地数千名孩子、家长和教师对 ScratchJr 的反响感到非常兴奋，但我们也认识到，为了帮助人们充分利用 ScratchJr，仍然需要更多、更好的支持材料。我们写这本书就是为了支持在家庭和学校中使用 ScratchJr。我们希望你觉得这本书有用，期待收到你的反馈和建议。

我们要感谢在研究、写作和制作这本书过程中帮助我们的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感谢我们的出版商 No Starch Press 的 Tyler Ortman 和 Serena Yang。在整个写作和出版过程中，他们提供了宝贵的帮助和建议。

ScratchJr 的成功离不开国家科学基金会（资助号：DRL-1118664）和 Scratch 基金会的慷慨资助。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（*[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)*），以支持 ScratchJr 软件和教育材料的未来发展。

祝你享受阅读！

Marina 和 Mitch

2010 年，Marina 建议我们的两个团队合作开发一个适合幼儿的编程语言，延续麻省理工学院（MIT）在 Scratch 上的研究成果，并结合塔夫茨大学在儿童早期教育方面的经验，从而诞生了 ScratchJr 的构想。我们与 Playful Invention Company（PICO）的 Paula Bontá 和 Brian Silverman 合作，他们在为儿童设计和开发编程语言方面拥有丰富的专业知识（同时也曾与 Seymour Papert 紧密合作过）。ScratchJr 是一次真正的团队合作，得到了塔夫茨大学、MIT、PICO 等许多人的贡献。我们鼓励你访问 ScratchJr 网站（ *[`www.scratchjr.org/`](http://www.scratchjr.org/)* ）查看完整的贡献者名单。

我们对来自全球成千上万的孩子、家长和老师对 ScratchJr 的反馈感到非常激动，但我们也意识到，为了帮助大家充分发挥 ScratchJr 的潜力，仍然需要更多、更好的支持材料。我们写这本书是为了支持 ScratchJr 在家庭和学校中的使用。希望这本书对你有帮助，我们期待听到你的反馈和建议。

我们要感谢参与本书研究、写作和制作的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还要特别感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。在写作和出版本书的过程中，他们提供了宝贵的帮助和建议。

如果没有来自美国国家科学基金会（资助编号 DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持未来 ScratchJr 软件和教育材料的发展。

祝你享受！

Marina 和 Mitch

我们对来自全球成千上万的孩子、家长和老师对 ScratchJr 的反馈感到非常激动，但我们也意识到，为了帮助大家充分发挥 ScratchJr 的潜力，仍然需要更多、更好的支持材料。我们写这本书是为了支持 ScratchJr 在家庭和学校中的使用。希望这本书对你有帮助，我们期待听到你的反馈和建议。

我们要感谢参与本书研究、写作和制作的 ScratchJr 团队成员，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还要特别感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。在写作和出版本书的过程中，他们提供了宝贵的帮助和建议。

没有来自国家科学基金会（拨款号 DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

享受吧！

Marina 和 Mitch

我们要感谢 ScratchJr 团队的成员，他们在本书的研究、写作和制作过程中提供了帮助，特别是 Claire Caine、Amanda Strawhacker、Mollie Elkin、Dylan Portelance、Amanda Sullivan 和 Alex Puganali。

我们还非常感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。他们在写作和出版过程中的帮助和建议是无价的。

没有来自国家科学基金会（拨款号 DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

享受吧！

Marina 和 Mitch

我们还非常感谢我们出版社 No Starch Press 的 Tyler Ortman 和 Serena Yang。他们在写作和出版过程中的帮助和建议是无价的。

没有来自国家科学基金会（拨款号 DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

享受吧！

Marina 和 Mitch

没有来自国家科学基金会（拨款号 DRL-1118664）和 Scratch 基金会的慷慨资助，ScratchJr 是不可能实现的。如果你喜欢这本书和 ScratchJr，我们希望你能考虑向 Scratch 基金会捐款（ *[`www.scratchfoundation.org/`](http://www.scratchfoundation.org/)* ），以支持 ScratchJr 软件和教育材料的未来发展。

享受吧！

Marina 和 Mitch

享受吧！

Marina 和 Mitch

Marina 和 Mitch
