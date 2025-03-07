# 第五章：D

使用 Git 进行版本控制

![](img/chapterart.png)

版本控制软件允许你在项目处于工作状态时拍摄快照。当你对项目进行更改时——例如，添加一个新功能——如果项目当前的状态无法正常工作，你可以回到先前的工作状态。

使用版本控制软件使你可以自由地进行改进和犯错，而不必担心破坏项目。这个功能在大型项目中尤为重要，但在较小的项目中也同样有帮助，甚至当你只是在处理单个文件中的程序时。

在本附录中，你将学习如何安装 Git，并在当前项目中使用它进行版本控制。*Git* 是目前最流行的版本控制软件。它的许多高级工具帮助团队在大型项目中协作，但其最基本的功能也非常适合单独开发者使用。Git 通过跟踪项目中每个文件的更改来实现版本控制；如果你犯了错误，只需返回到先前保存的状态。

## 安装 Git

Git 可在所有操作系统上运行，但每个系统的安装方法有所不同。以下各节提供了针对每个操作系统的具体安装说明。

Git 在一些系统中默认包含，并且常常与其他你可能已经安装的软件包一起捆绑。安装 Git 之前，先检查一下系统中是否已经安装了它。打开一个新的终端窗口并运行命令 `git --version`。如果输出显示了特定的版本号，则表示 Git 已经安装。如果看到提示你安装或更新 Git 的信息，请按照屏幕上的指示操作。

如果你没有看到屏幕上的任何提示，并且使用的是 Windows 或 macOS，可以从 [`git-scm.com`](https://git-scm.com) 下载一个安装程序。如果你是使用 apt 兼容系统的 Linux 用户，可以通过命令 `sudo apt install git` 安装 Git。

### 配置 Git

Git 跟踪谁对项目进行了更改，即使只有一个人在项目中工作。为此，Git 需要知道你的用户名和电子邮件地址。你必须提供一个用户名，但可以随意编造一个假的电子邮件地址：

```py
$ **git config --global user.name "**`username`**"**
$ **git config --global user.email "**`username@example.com`**"**
```

如果你忘记了这一步，Git 会在你第一次提交时提示你提供这些信息。

最好为每个项目设置主分支的默认名称。一个好的名称是 `main`：

```py
$ **git config --global init.defaultBranch main**
```

这个配置意味着，你使用 Git 管理的每个新项目将以一个名为 *main* 的单一分支提交开始。

## 创建一个项目

让我们创建一个项目来进行操作。在你的系统上创建一个名为 *git_practice* 的文件夹。在该文件夹内，创建一个简单的 Python 程序：

**hello_git.py**

```py
print("Hello Git world!")
```

我们将使用这个程序来探索 Git 的基本功能。

## 忽略文件

扩展名为*.pyc*的文件是从*.py*文件自动生成的，所以我们不需要 Git 来跟踪它们。这些文件被存储在一个名为*__pycache__*的目录中。为了告诉 Git 忽略这个目录，我们创建一个名为*.gitignore*的特殊文件——文件名以点号开头且没有扩展名，并将以下行添加到文件中：

**.gitignore**

```py
__pycache__/
```

这个文件告诉 Git 忽略任何位于*__pycache__*目录中的文件。使用*.gitignore*文件可以保持项目整洁，更容易进行操作。

你可能需要修改文件浏览器的设置，以便显示隐藏文件（文件名以点号开头）。在 Windows 资源管理器中，勾选“查看”菜单中的**隐藏的项目**框。在 macOS 中，按⌘-SHIFT-.（点号）。在 Linux 中，查找名为“显示隐藏文件”的设置。

## 初始化仓库

现在你有一个包含 Python 文件和*.gitignore*文件的目录，你可以初始化一个 Git 仓库。打开终端，进入*git_practice*文件夹，并运行以下命令：

```py
git_practice$ **git init**
Initialized empty Git repository in git_practice/.git/
git_practice$
```

输出显示 Git 已经在*git_practice*中初始化了一个空的仓库。一个*仓库*是 Git 正在积极跟踪的程序文件集合。Git 用来管理仓库的所有文件都位于一个隐藏的目录*.git*中，你不需要直接与这个目录交互。只要不要删除这个目录，否则你将丢失项目的历史记录。

## 检查状态

在做任何操作之前，让我们先查看一下项目的状态：

```py
git_practice$ **git status**
❶ On branch main
No commits yet

❷ Untracked files:
  (use "git add <file>..." to include in what will be committed)
      .gitignore
      hello_git.py

❸ nothing added to commit but untracked files present (use "git add" to track)
git_practice$
```

在 Git 中，一个*分支*是你正在工作的项目版本；在这里你可以看到我们在一个名为`main`的分支上❶。每次检查项目的状态时，它应该显示你在`main`分支上。接着你可以看到我们即将进行初始提交。一个*提交*是项目在某一特定时刻的快照。

Git 告知我们项目中有未被跟踪的文件❷，因为我们还没有告诉它要跟踪哪些文件。接着我们被告知，当前提交没有任何内容，但存在一些未跟踪的文件，我们可能想要将它们添加到仓库中❸。

## 添加文件到仓库

让我们将这两个文件添加到仓库中，再次检查状态：

```py
❶ git_practice$ **git add .**
❷ git_practice$ **git status**
On branch main
No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
❸       new file:   .gitignore
      new file:   hello_git.py

git_practice$
```

命令`git add .`将所有项目中尚未被跟踪的文件添加到仓库中❶，只要它们没有被列在*.gitignore*文件中。它不会提交这些文件；只是告诉 Git 开始关注它们。当我们现在检查项目的状态时，可以看到 Git 已经识别出一些需要提交的更改❷。标签*新文件*表示这些文件是新添加到仓库中的❸。

## 提交更改

让我们进行第一次提交：

```py
❶ git_practice$ **git commit -m "Started project."**
❷ [main (root-commit) cea13dd] Started project.
❸ 2 files changed, 5 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 hello_git.py
❹ git_practice$ **git status**
On branch main
nothing to commit, working tree clean
git_practice$
```

我们使用命令`git commit -m "``message``"`❶来对项目进行快照。`-m`标志告诉 Git 记录后续的消息（`Started` project.）到项目日志中。输出显示我们在`main`分支上❷，并且有两个文件发生了变化❸。

当我们现在检查状态时，可以看到我们位于 `main` 分支，并且工作树是干净的 ❹。这是你每次提交项目工作状态时应该看到的信息。如果你看到不同的信息，仔细阅读它；很可能是你在提交前忘记添加某个文件。

## 检查日志

Git 会记录所有对项目所做的提交日志。让我们查看日志：

```py
git_practice$ **git log**
commit cea13ddc51b885d05a410201a54faf20e0d2e246 (HEAD -> main)
Author: eric <eric@example.com>
Date:   Mon Jun 6 19:37:26 2022 -0800

    Started project.
git_practice$
```

每次你进行提交时，Git 会生成一个独特的 40 字符的参考 ID。它记录谁进行了提交、何时进行的提交以及记录的消息。你并不总是需要所有这些信息，因此 Git 提供了一个选项来打印更简洁的日志条目：

```py
git_practice$ **git log --pretty=oneline**
cea13ddc51b885d05a410201a54faf20e0d2e246 (HEAD -> main) Started project.
git_practice$
```

`--pretty=oneline` 标志提供了两条最重要的信息：提交的参考 ID 和为该提交记录的消息。

## 第二次提交

要看到版本控制的真正强大之处，我们需要对项目进行更改并提交该更改。在这里，我们将只是再给 *hello_git.py* 添加一行：

**hello_git.py**

```py
print("Hello Git world!")
print("Hello everyone.")
```

当我们检查项目的状态时，我们会看到 Git 已经注意到文件发生了变化：

```py
git_practice$ **git status**
❶ On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)

❷  modified:   hello_git.py

❸ no changes added to commit (use "git add" and/or "git commit -a")
git_practice$
```

我们看到当前工作的分支 ❶、被修改的文件名 ❷，以及没有提交的更改 ❸。让我们提交这个更改，然后再次检查状态：

```py
❶ git_practice$ **git commit -am "Extended greeting."**
[main 945fa13] Extended greeting.
 1 file changed, 1 insertion(+), 1 deletion(-)
❷ git_practice$ **git status**
On branch main
nothing to commit, working tree clean
❸ git_practice$ **git log --pretty=oneline**
945fa13af128a266d0114eebb7a3276f7d58ecd2 (HEAD -> main) Extended greeting.
cea13ddc51b885d05a410201a54faf20e0d2e246 Started project.
git_practice$
```

我们进行一次新的提交，在使用 `git commit` 命令时传入 `-am` 标志 ❶。`-a` 标志告诉 Git 将所有修改过的文件添加到当前的提交中。（如果你在两次提交之间创建了新文件，请重新发出 `git add .` 命令，将新文件包含进来。）`-m` 标志告诉 Git 在日志中记录这个提交的消息。

当我们检查项目的状态时，我们看到我们再次拥有一个干净的工作树 ❷。最后，我们在日志中看到这两个提交 ❸。

## 放弃更改

现在让我们来看一下如何放弃更改并回到上一个工作状态。首先，给 *hello_git.py* 添加一行：

**hello_git.py**

```py
print("Hello Git world!")
print("Hello everyone.")

print("Oh no, I broke the project!")
```

保存并运行这个文件。

我们检查状态时发现 Git 已经注意到这个更改：

```py
git_practice$ **git status**
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)

❶     modified:   hello_git.py

no changes added to commit (use "git add" and/or "git commit -a")
git_practice$
```

Git 发现我们修改了 *hello_git.py* ❶，如果需要，我们可以提交这个更改。但是这次，我们不会提交这个更改，而是回到上次提交的状态，那个时候我们知道项目是正常工作的。我们不会对 *hello_git.py* 做任何操作：既不删除这行，也不使用文本编辑器的撤销功能。相反，在你的终端会话中输入以下命令：

```py
git_practice$ **git restore .**
git_practice$ **git status**
On branch main
nothing to commit, working tree clean
git_practice$
```

命令 `git restore` `filename` 允许你放弃自上次提交以来某个特定文件的所有更改。命令 `git restore .` 放弃自上次提交以来在所有文件中做的所有更改；此操作将项目恢复到最后一次提交的状态。

当你回到文本编辑器时，你会看到 *hello_git.py* 已经恢复成了这个样子：

```py
print("Hello Git world!")
print("Hello everyone.")
```

尽管在这个简单项目中回到之前的状态看起来似乎很不起眼，但如果我们在一个有数十个修改文件的大项目中工作，自上次提交以来所有变动的文件都会被恢复。这个功能非常有用：你可以在实现新特性时进行尽可能多的更改，如果它们不起作用，你可以丢弃这些更改而不影响项目。你不需要记住那些更改并手动撤销，Git 会为你做这一切。

## 检出之前的提交

你可以通过使用`checkout`命令并利用参考 ID 的前六个字符，重新访问日志中的任何提交。在检出并查看一个较早的提交后，你可以返回到最新的提交，或者放弃最近的工作，从较早的提交开始继续开发：

```py
git_practice$ **git log --pretty=oneline**
945fa13af128a266d0114eebb7a3276f7d58ecd2 (HEAD -> main) Extended greeting.
cea13ddc51b885d05a410201a54faf20e0d2e246 Started project.
git_practice$ **git checkout cea13d**
Note: switching to 'cea13d'.

❶ You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

❷ Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at cea13d Started project.
git_practice$
```

当你检出一个之前的提交时，你将离开主分支，进入 Git 所说的*detached HEAD*状态❶。*HEAD*是项目当前已提交的状态；你之所以*detached*，是因为你已经离开了一个命名的分支（此处为`main`）。

要返回到`main`分支，你可以按照建议❷撤销之前的操作：

```py
git_practice$ **git switch -**
Previous HEAD position was cea13d Started project.
Switched to branch 'main'
git_practice$
```

此命令将你带回`main`分支。除非你想使用 Git 的更高级功能，否则在检出一个之前的提交后，最好不要对你的项目进行任何更改。不过，如果你是唯一一个在项目上工作的人，并且想要放弃所有最近的提交，回到之前的状态，你可以将项目重置到一个之前的提交。工作在`main`分支时，输入以下命令：

```py
❶ git_practice$ **git status**
On branch main
nothing to commit, working directory clean
❷ git_practice$ **git log --pretty=oneline**
945fa13af128a266d0114eebb7a3276f7d58ecd2 (HEAD -> main) Extended greeting.
cea13ddc51b885d05a410201a54faf20e0d2e246 Started project.
❸ git_practice$ **git reset --hard cea13d**
HEAD is now at cea13dd Started project.
❹ git_practice$ **git status**
On branch main
nothing to commit, working directory clean
❺ git_practice$ **git log --pretty=oneline**
cea13ddc51b885d05a410201a54faf20e0d2e246 (HEAD -> main) Started project.
git_practice$
```

我们首先检查状态，确保我们在`main`分支❶。当我们查看日志时，看到两个提交❷。然后，我们使用`git reset --hard`命令，并输入想要永久回退的提交的参考 ID 前六个字符❸。我们再次检查状态，看到我们处于`main`分支且没有需要提交的内容❹。当我们再次查看日志时，发现我们已经回到想要重新开始的提交❺。

## 删除仓库

有时候你可能会弄乱仓库的历史，甚至不知道如何恢复。如果发生这种情况，首先可以考虑按照附录 C 中讨论的方法寻求帮助。如果你无法修复它，而且是在做一个独立项目，你可以继续使用文件，但通过删除*.git*目录来摆脱项目的历史。这不会影响任何文件的当前状态，但会删除所有提交，因此你将无法检出项目的其他状态。

要执行此操作，你可以打开文件浏览器并删除*.git*仓库，或者通过命令行删除它。之后，你需要从一个新的仓库开始，重新开始跟踪更改。下面是在终端会话中执行此操作的过程：

```py
❶ git_practice$ **git status**
On branch main
nothing to commit, working directory clean
❷ git_practice$ **rm -rf .git/**
❸ git_practice$ **git status**
fatal: Not a git repository (or any of the parent directories): .git
❹ git_practice$ **git init**
Initialized empty Git repository in git_practice/.git/
❺ git_practice$ **git status**
On branch main
No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
      .gitignore
      hello_git.py

nothing added to commit but untracked files present (use "git add" to track)
❻ git_practice$ **git add .**
git_practice$ **git commit -m "Starting over."**
[main (root-commit) 14ed9db] Starting over.
 2 files changed, 5 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 hello_git.py
❼ git_practice$ **git status**
On branch main
nothing to commit, working tree clean
git_practice$
```

我们首先检查状态，发现工作目录是干净的 ❶。然后我们使用命令`rm -rf .git/`删除*.git*目录（在 Windows 上使用`del .git`） ❷。删除*.git*文件夹后检查状态，系统提示这不是一个 Git 仓库 ❸。Git 用来跟踪仓库的所有信息都存储在*.git*文件夹中，因此删除它会删除整个仓库。

接着，我们可以使用`git init`来初始化一个新的仓库 ❹。检查状态显示我们回到了初始阶段，等待第一次提交 ❺。我们添加文件并进行第一次提交 ❻。现在检查状态时显示我们处于新的`main`分支，且没有需要提交的内容 ❼。

使用版本控制需要一些练习，但一旦开始使用，你就再也不想没有它了。
