## 第二十二章：**并发与并行**

*高级监视员有她自己的格言：“给我看一个完全平稳的操作，我会告诉你那是某人掩盖错误的结果。真正的船只会摇摆。”*

— 弗兰克·赫伯特，《沙丘圣殿》

![图片](img/common.jpg)

在编程中，*并发*意味着在给定时间段内运行两个或更多任务。*并行*意味着两个或更多任务在同一时刻运行。这两个术语常常可以互换使用而不会产生负面后果，因为它们关系密切。本章介绍了这两个概念的基础知识。由于并发和并行编程是庞大而复杂的主题，全面的探讨需要一本完整的书籍。在本章末尾的“进一步阅读”部分，您可以找到相关书籍。

在本章中，您将学习如何使用 future 进行并发和并行编程。接下来，您将学习如何通过互斥量、条件变量和原子操作来安全地共享数据。然后，本章将演示如何利用执行策略加速代码，同时也可能带来潜在的风险。

### 并发编程

*并发程序*拥有多个*执行线程*（简称*线程*），这些线程是指令的序列。在大多数运行时环境中，操作系统充当调度程序，决定何时执行线程的下一条指令。每个进程可以有一个或多个线程，这些线程通常共享资源，例如内存。由于调度程序决定线程执行的时机，程序员通常无法依赖线程的执行顺序。作为交换，程序可以在同一时间段内（或者同时）执行多个任务，这通常会导致显著的加速。要观察从串行到并发版本的加速，系统需要具有并发硬件，例如多核处理器。

本节从异步任务开始，这是使程序并发的高级方法。接下来，您将学习一些基本的方法来协调这些任务，特别是在它们处理共享可变状态时。然后，您将了解一些低级功能，这些功能可用于在高层工具无法满足性能需求的独特情况下使用。

#### *异步任务*

引入并发到程序中的一种方式是创建*异步任务*。异步任务不需要立即获得结果。要启动异步任务，可以使用`std::async`函数模板，该模板位于`<future>`头文件中。

##### async

当你调用 `std::async` 时，第一个参数是启动策略 `std::launch`，它有两个值可选：`std::launch::async` 或 `std::launch::deferred`。如果你传递 `launch::async`，运行时会创建一个新线程来启动任务。如果传递 `deferred`，运行时会等到你需要任务结果时才会执行（有时这种模式被称为 *延迟求值*）。这个第一个参数是可选的，默认为 `async|deferred`，意味着具体使用哪种策略由实现决定。`std::async` 的第二个参数是一个函数对象，表示你想执行的任务。函数对象接受的参数数量和类型没有限制，且它可以返回任何类型。`std::async` 函数是一个可变参数模板，包含一个函数参数包。你传递的任何额外参数都会在异步任务启动时用于调用函数对象。此外，`std::async` 会返回一个名为 `std::future` 的对象。

以下是简化的 `async` 声明，帮助总结：

```
std::future<FuncReturnType> std::async([policy], func, Args&&... args);
```

现在你知道如何调用 `async`，让我们来看一下如何与其返回值进行交互。

##### 回到未来

`future` 是一个类模板，用于保存异步任务的结果值。它有一个模板参数，对应异步任务的返回值类型。例如，如果你传递一个返回 `string` 的函数对象，`async` 会返回一个 `future<string>`。给定一个 `future`，你可以通过三种方式与异步任务进行交互。

首先，你可以通过 `valid` 方法查询 `future` 是否有效。一个有效的 `future` 会关联一个共享状态。异步任务有共享状态，以便它们可以传递结果。任何由 `async` 返回的 `future` 在你获取异步任务的返回值之前都会是有效的，之后共享状态的生命周期结束，如 示例 19-1 所示。

```
#include <future>
#include <string>

using namespace std;

TEST_CASE("async returns valid future") {
  using namespace literals::string_literals;
  auto the_future = async([] { return "female"s; }); ➊
  REQUIRE(the_future.valid()); ➋
}
```

*示例 19-1：`async` 函数返回一个有效的 `future`。*

你启动一个异步任务，它简单地返回一个 `string` ➊。因为 `async` 总是返回一个有效的 `future`，所以 `valid` 返回 `true` ➋。

如果你默认构造一个 `future`，它没有关联共享状态，因此 `valid` 会返回 `false`，如 示例 19-2 所示。

```
TEST_CASE("future invalid by default") {
  future<bool> default_future; ➊
  REQUIRE_FALSE(default_future.valid()); ➋
}
```

*示例 19-2：默认构造的 `future` 是无效的。*

你默认构造一个 `future` ➊，然后 `valid` 返回 `false` ➋。

其次，你可以通过 `get` 方法从有效的 `future` 中获取值。如果异步任务尚未完成，调用 `get` 会阻塞当前执行的线程，直到结果可用。示例 19-3 演示了如何使用 `get` 获取返回值。

```
TEST_CASE("async returns the return value of the function object") {
  using namespace literals::string_literals;
  auto the_future = async([] { return "female"s; }); ➊
  REQUIRE(the_future.get() == "female"); ➋
}
```

*示例 19-3：`async` 函数返回一个有效的 `future`。*

你使用`async`来启动一个异步任务➊，然后在返回的`future`对象上调用`get`方法。正如预期的那样，结果是你传递给`async`的函数对象的返回值➋。

如果异步任务抛出异常，`future`将收集该异常，并在你调用`get`时抛出它，正如清单 19-4 所展示的那样。

```
TEST_CASE("get may throw ") {
  auto ghostrider = async(
                      [] { throw runtime_error{ "The pattern is full." }; }); ➊
  REQUIRE_THROWS_AS(ghostrider.get(), runtime_error); ➋
}
```

*清单 19-4：`get`方法将抛出异步任务抛出的异常。*

你将一个抛出`runtime_error`的 lambda 传递给`async`➊。当你调用`get`时，它会抛出该异常➋。

第三，你可以使用`std::wait_for`或`std::wait_until`来检查异步任务是否已完成。选择哪个取决于你想传递的`chrono`对象的类型。如果你有一个`duration`对象，你将使用`wait_for`；如果你有一个`time_point`对象，你将使用`wait_until`。两者都返回一个`std::future_status`，它有三种可能的值：

+   `future_status::deferred`表示异步任务将被懒惰评估，因此一旦调用`get`，任务就会执行。

+   `future_status::ready`表示任务已完成，结果已经准备好。

+   `future_status::timeout`表示任务尚未准备好。

如果任务在指定的等待时间之前完成，`async`会提前返回。

清单 19-5 展示了如何使用`wait_for`检查异步任务的状态。

```
TEST_CASE("wait_for indicates whether a task is ready") {
  using namespace literals::chrono_literals;
  auto sleepy = async(launch::async, [] { this_thread::sleep_for(100ms); }); ➊
  const auto not_ready_yet = sleepy.wait_for(25ms); ➋
  REQUIRE(not_ready_yet == future_status::timeout); ➌
  const auto totally_ready = sleepy.wait_for(100ms); ➍
  REQUIRE(totally_ready == future_status::ready); ➎
}
```

*清单 19-5：使用`wait_for`检查异步任务的状态*

首先，你使用`async`启动一个异步任务，该任务仅等待最多 100 毫秒后再返回➊。接下来，你调用`wait_for`并设置等待时间为 25 毫秒➋。由于任务仍在睡眠中（25 < 100），`wait_for`返回`future_status::timeout` ➌。你再次调用`wait_for`并等待最多 100 毫秒 ➍。因为第二次`wait_for`会在`async`任务完成后结束，所以最终的`wait_for`会返回`future_status::ready` ➎。

**注意**

*从技术上讲，清单 19-5 中的断言并不保证总是会通过。页面 389 中介绍的“等待”引入了`this_thread::sleep_for`，它并不精确。操作环境负责调度线程，可能会在指定的时间后再调度睡眠中的线程。*

##### 异步任务示例

清单 19-6 包含了`factorize`函数，它用于查找一个整数的所有因数。

**注意**

*清单 19-6 中的因式分解算法效率非常低，但对于本示例足够用了。要了解高效的整数因式分解算法，请参考 Dixon 算法、连分式因式分解算法或二次筛法。*

```
#include <set>

template <typename T>
std::multiset<T> factorize(T x) {
  std::multiset<T> result{ 1 }; ➊
  for(T candidate{ 2 }; candidate <= x; candidate++) { ➋
    if (x % candidate == 0) { ➌
      result.insert(candidate); ➍
      x /= candidate; ➎
      candidate = 1; ➏
    }
  }
  return result;
}
```

*清单 19-6：一个非常简单的整数因式分解算法*

该算法接受一个单一的参数`x`，并通过初始化一个包含 1 的`set`开始 ➊。接下来，它从 2 迭代到`x` ➋，检查`candidate`是否与之取模后结果为 0 ➌。若是，则`candidate`是一个因子，并将其添加到因子`set`中 ➍。你将`x`除以刚刚发现的因子 ➎，然后通过将`candidate`重置为 1 重新开始搜索 ➏。

由于整数分解是一个难题（并且因为 Listing 19-6 效率低下），调用`factorize`可能需要相较于本书中大多数函数更长的时间。这使得它成为异步任务的一个理想候选。`factor_task`函数在 Listing 19-7 中使用了第十二章中 Listing 12-25 中的`Stopwatch`来封装`factorize`，并返回一个格式化良好的消息。

```
#include <set>
#include <chrono>
#include <sstream>
#include <string>

using namespace std;

struct Stopwatch {
--snip--
};

template <typename T>
set<T> factorize(T x) {
--snip--
}

string factor_task(unsigned long x) { ➊
  chrono::nanoseconds elapsed_ns;
  set<unsigned long long> factors;
  {
    Stopwatch stopwatch{ elapsed_ns }; ➋
    factors = factorize(x); ➌
  }
  const auto elapsed_ms =
             chrono::duration_cast<chrono::milliseconds>(elapsed_ns).count(); ➍
  stringstream ss;
  ss << elapsed_ms << " ms: Factoring " << x << " ( "; ➎
  for(auto factor : factors) ss << factor << " "; ➏
  ss << ")\n";
  return ss.str(); ➐
}
```

*Listing 19-7: 一个包装`factorize`调用并返回格式化消息的`factor_task`函数*

和`factorize`类似，`factor_task`也接受一个单一的参数`x`进行分解 ➊。（为了简化，`factor_task`接受一个`unsigned long`类型的参数，而不是模板参数）。接下来，你在一个嵌套作用域中初始化一个`Stopwatch` ➋，然后调用`factorize`来分解`x` ➌。结果是，`elapsed_ns`包含了`factorize`执行时经过的纳秒数，而`factors`则包含了`x`的所有因子。

接下来，你通过首先将`elapsed_ns`转换为毫秒数 ➍，构建一个格式化良好的字符串。你将这些信息写入名为`ss`的`stringstream`对象 ➎，然后写入`x`的因子 ➏。最后，返回生成的`string` ➐。

Listing 19-8 使用`factor_task`分解六个不同的数字，并记录总的程序运行时间。

```
#include <set>
#include <array>
#include <vector>
#include <iostream>
#include <limits>
#include <chrono>
#include <sstream>
#include <string>

using namespace std;

struct Stopwatch {
--snip--
};

template <typename T>
set<T> factorize(T x) {
--snip--
}

string factor_task(unsigned long long x) {
--snip--
}

array<unsigned long long, 6> numbers{ ➊
        9'699'690,
        179'426'549,
        1'000'000'007,
        4'294'967'291,
        4'294'967'296,
        1'307'674'368'000
};

int main() {
  chrono::nanoseconds elapsed_ns;
  {
    Stopwatch stopwatch{ elapsed_ns }; ➋
    for(auto number : numbers) ➌
      cout << factor_task(number); ➍
  }
  const auto elapsed_ms =
             chrono::duration_cast<chrono::milliseconds>(elapsed_ns).count(); ➎
  cout << elapsed_ms << "ms: total program time\n"; ➏
}
-----------------------------------------------------------------------
0 ms: Factoring 9699690 ( 1 2 3 5 7 11 13 17 19 )
1274 ms: Factoring 179426549 ( 1 179426549 )
6804 ms: Factoring 1000000007 ( 1 1000000007 )
29035 ms: Factoring 4294967291 ( 1 4294967291 )
0 ms: Factoring 4294967296 ( 1 2 )
0 ms: Factoring 1307674368000 ( 1 2 3 5 7 11 13 )
37115ms: total program time
```

*Listing 19-8: 一个使用`factor_task`来分解六个不同数字的程序*

你构建了一个包含六个不同大小和素数性质的`numbers`数组 ➊。接下来，你初始化一个`Stopwatch` ➋，遍历`numbers`中的每个元素 ➌，并调用`factor_task`进行分解 ➍。然后，你计算程序的运行时间（以毫秒为单位） ➎，并打印出来 ➏。

输出结果显示，某些数字，如 9,699,690、4,294,967,296 和 1,307,674,368,000，几乎可以立即分解，因为它们包含较小的因子。然而，素数需要相当长的时间。请注意，由于程序是单线程的，整个程序的运行时间大致等于分解每个数字所花费时间的总和。

如果将每个`factor_task`视为异步任务会怎样？Listing 19-9 演示了如何使用`async`实现这一点。

```
#include <set>
#include <vector>
#include <array>
#include <iostream>
#include <limits>
#include <chrono>
#include <future>
#include <sstream>
#include <string>

using namespace std;

struct Stopwatch {
--snip--
};

template <typename T>
set<T> factorize(T x) {
--snip--
}

string factor_task(unsigned long long x) {
--snip--
}

array<unsigned long long, 6> numbers{
--snip--
};

int main() {
  chrono::nanoseconds elapsed_ns;
  {
    Stopwatch stopwatch{ elapsed_ns }; ➊
    vector<future<string>> factor_tasks; ➋
    for(auto number : numbers) ➌
      factor_tasks.emplace_back(async(launch::async, factor_task, number)); ➍
    for(auto& task : factor_tasks) ➎
      cout << task.get(); ➏
  }
  const auto elapsed_ms =
             chrono::duration_cast<chrono::milliseconds>(elapsed_ns).count(); ➐
  cout << elapsed_ms << " ms: total program time\n"; ➑
}
-----------------------------------------------------------------------
0 ms: Factoring 9699690 ( 1 2 3 5 7 11 13 17 19 )
1252 ms: Factoring 179426549 ( 1 179426549 )
6816 ms: Factoring 1000000007 ( 1 1000000007 )
28988 ms: Factoring 4294967291 ( 1 4294967291 )
0 ms: Factoring 4294967296 ( 1 2 )
0 ms: Factoring 1307674368000 ( 1 2 3 5 7 11 13 )
28989 ms: total program time
```

*Listing 19-9: 一个使用`factor_task`异步地分解六个不同数字的程序*

如在示例 19-8 中所示，你初始化一个`Stopwatch`来记录程序执行的时长 ➊。接下来，你初始化一个名为`factor_tasks`的`vector`，它包含`future<string>`类型的对象 ➋。你遍历`numbers` ➌，调用`async`并使用`launch::async`策略，指定`factor_task`为函数对象，并传递`number`作为任务的参数。你对每个生成的`future`调用`emplace_back`，将其加入到`factor_tasks` ➍。现在，`async`已经启动了每个任务，你遍历`factor_tasks`中的每个元素 ➎，调用`get`来获取每个`task`的结果，并将其写入`cout` ➏。一旦从所有的`future`中收到了值，你就能计算出执行所有任务所用的毫秒数 ➐，并将其写入`cout` ➑。

由于并发性，示例 19-9 的总程序时间大约等于最大任务执行时间（28,988 毫秒），而不是任务执行时间的总和，如在示例 19-8 中所示（37,115 毫秒）。

**注意**

*示例 19-8 和示例 19-9 中的时间会因每次运行而有所不同。*

#### *共享与协调*

使用异步任务进行并发编程是简单的，只要任务不需要同步，并且不涉及共享可变数据。例如，考虑一个简单的情境，其中两个线程访问同一个整数。一个线程会递增这个整数，而另一个线程会递减它。为了修改变量，每个线程必须读取变量的当前值，进行加法或减法操作，然后将变量写回内存。如果没有同步机制，这两个线程将以未定义的交错顺序执行这些操作。这种情况有时被称为*竞争条件*，因为结果取决于哪个线程先执行。示例 19-10 展示了这种情况有多么灾难性。

```
#include <future>
#include <iostream>

using namespace std;

void goat_rodeo() {
  const size_t iterations{ 1'000'000 };
  int tin_cans_available{}; ➊

  auto eat_cans = async(launch::async, [&] { ➋
    for(size_t i{}; i<iterations; i++)
      tin_cans_available--; ➌
  });
  auto deposit_cans = async(launch::async, [&] { ➍
    for(size_t i{}; i<iterations; i++)
      tin_cans_available++; ➎
  });
  eat_cans.get(); ➏
  deposit_cans.get(); ➐
  cout << "Tin cans: " << tin_cans_available << "\n"; ➑
}
int main() {
  goat_rodeo();
  goat_rodeo();
  goat_rodeo();
}
-----------------------------------------------------------------------
Tin cans: -609780
Tin cans: 185380
Tin cans: 993137
```

*示例 19-10：展示了未同步、可变共享数据访问可能带来的灾难性后果*

**注意**

*由于程序存在未定义行为，在运行示例 19-10 时，你将获得不同的结果。*

示例 19-10 涉及定义一个名为`goat_rodeo`的函数，它包含一个灾难性的竞争条件，以及一个调用`goat_rodeo`三次的`main`函数。在`goat_rodeo`中，你初始化了共享数据`tin_cans_available` ➊。接下来，你启动一个名为`eat_cans`的异步任务 ➋，在该任务中，一群山羊会将共享变量`tin_cans_available`递减一百万次 ➌。然后，你启动另一个名为`deposit_cans`的异步任务 ➍，该任务会递增`tin_cans_available` ➎。启动这两个任务后，你通过调用`get`等待它们完成（顺序无关） ➏➐。任务完成后，你打印出`tin_cans_available`变量的值 ➑。

从直觉上讲，你可能会期望每个任务完成后 `tin_cans_available` 等于零。毕竟，无论你如何排序递增和递减，如果它们的次数相等，它们会相互抵消。你调用了三次 `goat_rodeo`，每次调用的结果都完全不同。

表 19-1 说明了在 清单 19-10 中，无同步访问如何导致问题。

**表 19-1：** `eat_cans` 和 `deposit_cans` 的一种可能调度

| **eat_cans** | **deposit_cans** | **cans_available** |
| --- | --- | --- |
| 读取 `cans_available` (0) |  | 0 |
|  | 读取 `cans_available` (0) ➊ | 0 |
| 计算 `cans_available+1` (1) |  | 0 |
|  | 计算 `cans_available-1` (-1) ➌ | 0 |
| 写入 `cans_available+1` (1) ➋ |  | 1 |
|  | 写入 `cans_available-1` (-1) ➍ | -1 |

表 19-1 显示了交替读取和写入如何带来灾难。在这种特殊情况下，`deposit_cans` 的读取 ➊ 在 `eat_cans` 的写入 ➋ 之前发生，因此 `deposit_cans` 计算了一个过时的结果 ➌。更糟糕的是，它在写入时覆盖了 `eat_cans` 的写入 ➍。

这个数据竞争问题的根本原因是 *对可变共享数据的无同步访问*。你可能会问，为什么每当一个线程计算 `cans_available+1` 或 `cans_available-1` 时，`cans_available` 不会立即更新？答案在于，表 19-1 中的每一行都表示某个指令执行完毕的时刻，而加法、减法、读取和写入内存的指令是分开的。由于 `cans_available` 变量是共享的，并且两个线程都在没有同步其操作的情况下写入它，因此指令在运行时会以未定义的方式交替执行（并带来灾难性后果）。在接下来的子节中，你将学习三种应对这种情况的工具：*互斥量*、*条件变量* 和原子操作。

##### 互斥量

*互斥算法*（*mutex*）是一种防止多个线程同时访问资源的机制。互斥量是 *同步原语*，支持两种操作：锁定和解锁。当一个线程需要访问共享数据时，它会锁定互斥量。根据互斥量的性质以及是否有其他线程已获得锁，锁定操作可能会被阻塞。当线程不再需要访问时，它会解锁互斥量。

`<mutex>` 头文件提供了几种互斥选项：

+   `std::mutex` 提供基本的互斥功能。

+   `std::timed_mutex` 提供了带有超时的互斥功能。

+   `std::recursive_mutex` 提供了允许同一线程递归锁定的互斥功能。

+   `std::recursive_timed_mutex` 提供了允许同一线程递归锁定的互斥功能，并且有超时功能。

`<shared_mutex>` 头文件提供了两个额外的选项：

+   `std::shared_mutex`提供共享互斥功能，这意味着多个线程可以同时拥有该互斥锁。这个选项通常用于多个读线程可以访问共享数据，而写线程需要独占访问的场景。

+   `std::shared_timed_mutex`提供共享互斥功能，并实现了带有超时的锁定机制。

**注意**

*为了简单起见，本章仅介绍互斥锁。有关其他选项的更多信息，请参见[thread.mutex]。*

`mutex`类只定义了一个单一的默认构造函数。当你需要获得互斥访问时，你可以在`mutex`对象上调用两个方法之一：`lock`或`try_lock`。如果调用`lock`，它不接受任何参数并返回`void`，调用线程会阻塞，直到`mutex`可用。如果调用`try_lock`，它也不接受任何参数并返回一个`bool`，它会立即返回。如果`try_lock`成功获得了互斥访问，它会返回`true`，并且调用线程现在拥有锁。如果`try_lock`失败，它会返回`false`，并且调用线程没有获得锁。要释放互斥锁，你只需调用`unlock`方法，它不接受任何参数并返回`void`。

列表 19-11 展示了一种基于锁的方式来解决列表 19-10 中的竞态条件。

```
#include <future>
#include <iostream>
#include <mutex>

using namespace std;

void goat_rodeo() {
  const size_t iterations{ 1'000'000 };
  int tin_cans_available{};
  mutex tin_can_mutex; ➊

  auto eat_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++) {
      tin_can_mutex.lock(); ➋
      tin_cans_available--;
      tin_can_mutex.unlock(); ➌
    }
  });
  auto deposit_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++) {
      tin_can_mutex.lock(); ➍
      tin_cans_available++;
      tin_can_mutex.unlock(); ➎
    }
  });
  eat_cans.get();
  deposit_cans.get();
  cout << "Tin cans: " << tin_cans_available << "\n";
}

int main() {
  goat_rodeo(); ➏
  goat_rodeo(); ➐
  goat_rodeo(); ➑
}
-----------------------------------------------------------------------
Tin cans: 0 ➏
Tin cans: 0 ➐
Tin cans: 0 ➑
```

*列表 19-11：使用`mutex`解决列表 19-10 中的竞态条件*

你在`goat_rodeo` ➊中添加了一个名为`tin_can_mutex`的`mutex`，它对`tin_cans_available`提供互斥访问。在每个异步任务中，线程在修改`tin_cans_available`之前会获取一个锁 ➋➍。修改完成后，线程会解锁 ➌➎。注意，每次运行结束时，`tin_cans_available`的最终数量为零 ➏➐➑，这表明你已经修复了竞态条件。

**互斥锁实现**

在实践中，互斥锁有多种实现方式。最简单的互斥锁可能是*自旋锁*，其中线程会执行一个循环，直到锁被释放。这种锁通常可以最小化一个线程释放锁与另一个线程获取锁之间的时间。但它在计算上是昂贵的，因为 CPU 会花费大量时间检查锁是否可用，而其他线程本可以进行有生产力的工作。通常，互斥锁需要原子指令，如`compare-and-swap`、`fetch-and-add`或`test-and-set`，这样它们就能在一个操作中检查并获取锁。

现代操作系统，如 Windows，提供了比自旋锁更高效的替代方案。例如，基于*异步过程调用*的互斥锁允许线程在等待互斥锁时进入*等待状态*。一旦互斥锁变得可用，操作系统会唤醒等待的线程，并将互斥锁的所有权交给该线程。这使得其他线程可以在 CPU 上做有生产力的工作，而不是被自旋锁占用。

一般来说，除非互斥锁成为程序的瓶颈，否则你不需要关心操作系统如何实现互斥锁的细节。

如果你认为处理`mutex`锁定是 RAII 对象的完美任务，你是对的。假设你忘记调用`unlock`释放一个互斥锁，比如因为它抛出了异常。当下一个线程来尝试通过`lock`获取这个互斥锁时，你的程序会停滞不前。正因如此，标准库提供了用于处理互斥锁的 RAII 类，位于`<mutex>`头文件中。在那里，你会找到几个类模板，它们都接受互斥锁作为构造函数参数，并且有一个与互斥锁类型对应的模板参数：

+   `std::lock_guard`是一个不可复制、不可移动的 RAII 封装器，它在构造函数中接受一个互斥锁对象，并调用`lock`。然后在析构函数中调用`unlock`。

+   `std::scoped_lock`是一个避免死锁的 RAII 封装器，用于多个互斥锁。

+   `std::unique_lock`实现了一个可移动的互斥锁所有权封装器。

+   `std::shared_lock`实现了一个可移动的共享互斥锁所有权封装器。

为简洁起见，本节重点讨论`lock_guard`。清单 19-12 展示了如何重构清单 19-11，以使用`lock_guard`代替手动操作`mutex`。

```
#include <future>
#include <iostream>
#include <mutex>

using namespace std;

void goat_rodeo() {
  const size_t iterations{ 1'000'000 };
  int tin_cans_available{};
  mutex tin_can_mutex;
  auto eat_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++) {
      lock_guard<mutex> guard{ tin_can_mutex }; ➊
      tin_cans_available--;
    }
  });
  auto deposit_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++) {
      lock_guard<mutex> guard{ tin_can_mutex }; ➋
      tin_cans_available++;
    }
  });
  eat_cans.get();
  deposit_cans.get();
  cout << "Tin cans: " << tin_cans_available << "\n";
}

int main() {
  goat_rodeo();
  goat_rodeo();
  goat_rodeo();
}
-----------------------------------------------------------------------
Tin cans: 0
Tin cans: 0
Tin cans: 0
```

*清单 19-12：重构清单 19-11 以使用`lock_guard`*

与其使用`lock`和`unlock`来管理互斥，你可以在需要同步的每个作用域开始时构造一个`lock_guard` ➊➋。由于你的互斥机制是`mutex`，你需要将其指定为`lock_guard`模板参数。清单 19-11 和清单 19-12 在运行时行为上是等价的，包括程序执行所需的时间。RAII 对象不会引入比手动释放和获取锁更高的运行时成本。

不幸的是，互斥锁涉及运行时成本。你可能也注意到，执行清单 19-11 和 19-12 的时间明显比执行清单 19-10 要长。原因是获取和释放锁是相对昂贵的操作。在清单 19-11 和清单 19-12 中，`tin_can_mutex`被获取然后释放了两百万次。相较于增减一个整数，获取或释放锁花费的时间要多得多，因此使用互斥锁来同步异步任务是次优的。在某些情况下，你可以通过使用原子操作采取可能更高效的方法。

**注意**

*有关异步任务和未来值的更多信息，请参阅[futures.async]。*

##### 原子操作

单词*atomic*来自希腊语*átomos*，意为“不可分割”。当一个操作在不可分割的单元中发生时，这个操作就是原子的。另一个线程无法观察到操作进行到一半的状态。当你在清单 19-10 中引入锁来生成清单 19-11 时，你使得增量和减量操作变得原子化，因为异步任务无法再交错地读取和写入`tin_cans_available`。正如你在运行这个基于锁的解决方案时体验到的那样，这种方法非常慢，因为获取锁是非常昂贵的。

另一种方法是使用`std::atomic`类模板，该模板在`<atomic>`头文件中提供了常用于*无锁并发编程*的原语。无锁并发编程解决了数据竞争问题，而不涉及锁。在许多现代架构中，CPU 支持原子指令。使用原子操作，你可能能够通过依赖原子硬件指令来避免锁。

本章不会详细讨论`std::atomic`或如何设计自己的无锁解决方案，因为这非常难以正确实现，最好留给专家。不过，在简单的情况下，比如在清单 19-10 中，你可以使用`std::atomic`来确保增量或减量操作无法被拆分。这样可以巧妙地解决数据竞争问题。

`std::atomic`模板为所有基本类型提供了特化，如表 19-2 所示。

**表 19-2：** `std::atomic`模板对基本类型的特化

| **模板特化** | **别名** |
| --- | --- |
| `std::atomic<bool>` | `std::atomic_bool` |
| `std::atomic<char>` | `std::atomic_char` |
| `std::atomic<unsigned char>` | `std::atomic_uchar` |
| `std::atomic<short>` | `std::atomic_short` |
| `std::atomic<unsigned short>` | `std::atomic_ushort` |
| `std::atomic<int>` | `std::atomic_int` |
| `std::atomic<unsigned int>` | `std::atomic_uint` |
| `std::atomic<long>` | `std::atomic_long` |
| `std::atomic<unsigned long>` | `std::atomic_ulong` |
| `std::atomic<long long>` | `std::atomic_llong` |
| `std::atomic<unsigned long long>` | `std::atomic_ullong` |
| `std::atomic<char16_t>` | `std::atomic_char16_t` |
| `std::atomic<char32_t>` | `std::atomic_char32_t` |
| `std::atomic<wchar_t>` | `std::atomic_wchar_t` |

表 19-3 列出了`std::atomic`的一些支持操作。`std::atomic`模板没有拷贝构造函数。

**表 19-3：** `std::atomic`的支持操作

| **操作** | **描述** |
| --- | --- |
| a`{}`a`{ 123 }` | 默认构造函数。将值初始化为 123。 |
| a`.is_lock_free()` | 如果 a 是无锁的，则返回 true。（取决于 CPU。） |
| a`.store(123)` | 将值 123 存储到 a 中。 |
| a`.load()`a`() | 返回存储的值。 |
| a`.exchange(123)` | 将当前值替换为 123，并返回旧值。这是一个“读-修改-写”操作。 |
| a`.compare_exchange_weak(10, 20)`a`.compare_exchange_strong(10, 20)` | 如果当前值为 10，则替换为 20。若值被替换，返回 true。有关弱交换与强交换的详情，请参见`[atomic]`。 |

**注意**

*<cstdint>中的类型也提供了特化操作。详细信息请参见[atomics.syn]。*

对于数值类型，特化操作提供了附加的操作，如表 19-4 所列。

**表 19-4：** `std::atomic` a 的数值特化支持的操作

| **操作** | **描述** |
| --- | --- |
| a`.fetch_add(123)`a`+=123` | 用当前值加上参数的结果替换当前值。返回修改前的值。这是一个“读-修改-写”操作。 |
| a`.fetch_sub(123)`a`-=123` | 用当前值减去参数的结果替换当前值。返回修改前的值。这是一个“读-修改-写”操作。 |
| a`.fetch_and(123)`a`&=123` | 用当前值与参数进行按位与运算的结果替换当前值。返回修改前的值。这是一个“读-修改-写”操作。 |
| a`.fetch_or(123)`a`&#124;=123` | 用当前值与参数进行按位或运算的结果替换当前值。返回修改前的值。这是一个“读-修改-写”操作。 |
| a`.fetch_xor(123)`a`^=123` | 用当前值与参数进行按位异或运算的结果替换当前值。返回修改前的值。这是一个“读-修改-写”操作。 |
| a`++`a`--` | 增加或减少 a 的值。 |

因为清单 19-12 是一个适合无锁解决方案的典型例子，你可以将`tin_cans_available`的类型替换为`atomic_int`，并移除`mutex`。这样可以防止像表 19-1 所示的竞争条件。清单 19-13 实现了这一重构。

```
#include <future>
#include <iostream>
#include <atomic>

using namespace std;

void goat_rodeo() {
  const size_t iterations{ 1'000'000 };
  atomic_int➊ tin_cans_available{};
  auto eat_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++)
      tin_cans_available--; ➋
  });
  auto deposit_cans = async(launch::async, [&] {
    for(size_t i{}; i<iterations; i++)
      tin_cans_available++; ➌
  });
  eat_cans.get();
  deposit_cans.get();
  cout << "Tin cans: " << tin_cans_available << "\n";
}

int main() {
  goat_rodeo();
  goat_rodeo();
  goat_rodeo();
}
-----------------------------------------------------------------------
Tin cans: 0
Tin cans: 0
Tin cans: 0
```

*清单 19-13：使用`atomic_int`而非`mutex`解决竞争条件*

你将`int`替换为`atomic_int` ➊并移除`mutex`。因为递减 ➋ 和递增 ➌ 运算符是原子的，竞争条件仍然得到解决。

**注意**

*有关原子操作的更多信息，请参见[atomics]。*

你可能还注意到，从清单 19-12 到 19-13，性能有了显著的提升。一般来说，使用原子操作将比获取互斥锁更快。

**警告**

*除非你有一个非常简单的并发访问问题，比如本节中的例子，否则你真的不应该尝试自己实现无锁解决方案。请参考 Boost Lockfree 库，获取高质量、经过彻底测试的无锁容器。像往常一样，你必须决定基于锁的实现还是无锁实现更为优化。*

##### 条件变量

*条件变量*是一种同步原语，它会阻塞一个或多个线程，直到被通知。另一个线程可以通知条件变量。通知后，条件变量可以解除阻塞一个或多个线程，使它们能够继续执行。一个非常流行的条件变量模式涉及一个线程执行以下操作：

1.  获取一些与等待线程共享的互斥锁。

1.  修改共享状态。

1.  通知条件变量。

1.  释放互斥锁。

任何在条件变量上等待的线程会执行以下操作：

1.  获取互斥锁。

1.  在条件变量上等待（这会释放互斥锁）。

1.  当另一个线程通知条件变量时，当前线程醒来并可以执行一些工作（这会自动重新获取互斥锁）。

1.  释放互斥锁。

由于现代操作系统的复杂性，有时线程会无故醒来。因此，重要的是在等待的线程醒来时，验证条件变量确实已被通知。

标准库在`<condition_variable>`头文件中提供了`std::condition_variable`，它支持多种操作，包括表 19-5 中的操作。`condition_variable`仅支持默认构造，并且拷贝构造函数被删除。

**表 19-5：** `std::condition_variable` cv 支持的操作

| **操作** | **描述** |
| --- | --- |
| cv`.notify_one()` | 如果有线程在等待条件变量，此操作会通知其中一个线程。 |
| cv`.notify_all()` | 如果有线程在等待条件变量，此操作会通知所有线程。 |
| cv`.wait(`lock`, [`pred`])` | 在通知者拥有的互斥锁上获取锁，唤醒时返回。如果提供了`pred`，则确定通知是否是虚假通知（返回`false`）还是有效通知（返回`true`）。 |
| cv`.wait_for(`lock`, [`durn`], [`pred`])` | 与 cv`.wait`相同，除了`wait_for`只等待`durn`。如果超时发生且未提供`pred`，返回`std::cv_status::timeout`；否则，返回`std::cv_status::no_timeout`。 |
| cv`.wait_until(`lock`, [`time`], [`pred`])` | 与`wait_for`相同，只是使用`std::chrono::time_point`而不是`std::chrono::duration`。 |

例如，您可以重构清单 19-12，使得*放置罐子*任务在*吃罐子*任务之前完成，使用条件变量，正如清单 19-14 所示。

```
#include <future>
#include <iostream>
#include <mutex>
#include <condition_variable>

using namespace std;

void goat_rodeo() {
  mutex m; ➊
  condition_variable cv; ➋
  const size_t iterations{ 1'000'000 };
  int tin_cans_available{};
  auto eat_cans = async(launch::async, [&] {
    unique_lock<mutex> lock{ m }; ➌
    cv.wait(lock, [&] { return tin_cans_available == 1'000'000; }); ➍
    for(size_t i{}; i<iterations; i++)
      tin_cans_available--;
  });

  auto deposit_cans = async(launch::async, [&] {
    scoped_lock<mutex> lock{ m }; ➎
    for(size_t i{}; i<iterations; i++)
      tin_cans_available++;
    cv.notify_all(); ➏
  });
  eat_cans.get();
  deposit_cans.get();
  cout << "Tin cans: " << tin_cans_available << "\n";
}

int main() {
  goat_rodeo();
  goat_rodeo();
  goat_rodeo();
}
-----------------------------------------------------------------------
Tin cans: 0
Tin cans: 0
Tin cans: 0
```

*清单 19-14：使用条件变量确保所有罐子在被吃之前都被放置*

你声明一个 `mutex` ➊ 和一个 `condition_variable` ➋，你将用它们来协调异步任务。在 *吃罐头* 任务中，你获取一个 `unique_lock` 对 `mutex` 的锁，并将其与一个谓词一起传递给 `wait`，该谓词如果有罐头可用时返回 `true` ➌。该方法将释放 `mutex`，然后阻塞直到满足两个条件：`condition_variable` 唤醒该线程，并且有一百万个罐头可用 ➍（记住，你必须检查所有罐头是否可用，因为可能会发生虚假唤醒）。在 *存罐头* 任务中，你获取对 `mutex` ➎ 的锁，存入罐头，然后通知所有在 `condition_variable` 上阻塞的线程 ➏。

请注意，与之前的所有方法不同，`tin_cans_available` 不可能为负，因为存罐头和吃罐头的顺序是有保障的。

**注意**

*有关条件变量的更多信息，请参考 *[thread.condition]*。*

#### *低级并发设施*

标准库的 `<thread>` 库包含用于并发编程的低级设施。例如，`std::thread` 类模拟了操作系统线程。然而，最好不要直接使用 `thread`，而是通过更高级的抽象设计并发，比如任务。如果你需要低级线程访问，[thread] 提供了更多信息。

但是，`<thread>` 库确实包含了几个有用的函数，用于操作当前线程：

+   `std::this_thread::yield` 函数不接受任何参数，并返回 `void`。`yield` 的确切行为取决于环境，但通常它提供一个提示，操作系统应该给其他线程一个运行的机会。这在例如，当某个资源的锁竞争很激烈时非常有用，你希望帮助所有线程获得访问的机会。

+   `std::this_thread::get_id` 函数不接受任何参数，并返回一个类型为 `std::thread::id` 的对象，这是一个轻量级线程，支持比较操作符和 `operator<<`。通常，它被用作关联容器中的键。

+   `std::this_thread::sleep_for` 函数接受一个 `std::chrono::duration` 参数，阻塞当前线程的执行，直到至少经过指定的时长，并返回 `void`。

+   `std::this_thread::sleep_until` 接受一个 `std::chrono::time_point`，并返回 `void`。它完全类似于 `sleep_for`，只不过它会阻塞线程直到至少达到指定的 `time_point`。

当你需要这些功能时，它们是不可或缺的。否则，你真的不应该需要与 `<thread>` 头文件交互。

### 并行算法

第十八章介绍了 stdlib 的算法，其中许多算法接受一个可选的第一个参数，称为其执行策略，由一个`std::execution`值进行编码。在支持的环境中，有三个可能的值：`seq`、`par`和`par_unseq`。后两个选项表示你希望并行执行算法。

#### *示例：并行排序*

示例 19-15 展示了如何通过将单一参数从`seq`改为`par`，大幅影响程序运行时间，方法是对十亿个数字进行两种方式的排序。

```
#include <algorithm>
#include <vector>
#include <numeric>
#include <random>
#include <chrono>
#include <iostream>
#include <execution>

using namespace std;

// From Listing 12-25:
struct Stopwatch {
--snip--
};

vector<long> make_random_vector() { ➊
  vector<long> numbers(1'000'000'000);
  iota(numbers.begin(), numbers.end(), 0);
  mt19937_64 urng{ 121216 };
  shuffle(numbers.begin(), numbers.end(), urng);
  return numbers;
}

int main() {
  cout << "Constructing random vectors...";
  auto numbers_a = make_random_vector(); ➋
  auto numbers_b{ numbers_a }; ➌
  chrono::nanoseconds time_to_sort;
  cout << " " << numbers_a.size() << " elements.\n";
  cout << "Sorting with execution::seq...";
  {
    Stopwatch stopwatch{ time_to_sort };
    sort(execution::seq, numbers_a.begin(), numbers_a.end()); ➍
  }
  cout << " took " << time_to_sort.count() / 1.0E9 << " sec.\n";

  cout << "Sorting with execution::par...";
  {
    Stopwatch stopwatch{ time_to_sort };
    sort(execution::par, numbers_b.begin(), numbers_b.end()); ➎
  }
  cout << " took " << time_to_sort.count() / 1.0E9 << " sec.\n";
}
-----------------------------------------------------------------------
Constructing random vectors... 1000000000 elements.
Sorting with execution::seq... took 150.489 sec.
Sorting with execution::par... took 17.7305 sec.
```

*示例 19-15：使用`std::sort`和`std::execution::seq`与`std::execution::par`对十亿个数字进行排序。（结果来自一台 Windows 10 x64 机器，配有两颗 Intel Xeon E5-2620 v3 处理器。）*

`make_random_vector`函数 ➊ 生成一个包含十亿个唯一数字的`vector`。你创建两个副本，`numbers_a` ➋ 和 `numbers_b` ➌。你分别对每个`vector`进行排序。在第一种情况下，你使用顺序执行策略进行排序 ➍，`Stopwatch`显示操作花费了大约两分半钟（约 150 秒）。在第二种情况下，你使用并行执行策略进行排序 ➎。相比之下，`Stopwatch`显示操作只花费了大约 18 秒。顺序执行耗时大约是并行执行的 8.5 倍。

#### *并行算法并非魔法*

不幸的是，并行算法并非魔法。尽管它们在简单情况中表现得非常出色，例如在示例 19-15 中的`sort`，但在使用时仍需小心。每当算法产生超出目标序列的副作用时，你就必须深入思考竞态条件。一个警示信号是任何向算法传递函数对象的算法。如果函数对象有共享的可变状态，执行的线程将共享访问，可能会发生竞态条件。例如，考虑示例 19-16 中的并行`transform`调用。

```
#include <algorithm>
#include <vector>
#include <iostream>
#include <numeric>
#include <execution>

int main() {
  std::vector<long> numbers{ 1'000'000 }, squares{ 1'000'000 }; ➊
  std::iota(numbers.begin(), numbers.end(), 0); ➋
  size_t n_transformed{}; ➌
  std::transform(std::execution::par, numbers.begin(), numbers.end(), ➍
                 squares.begin(), [&n_transformed] (const auto x) {
                  ++n_transformed; ➎
                  return x * x; ➏
                });
  std::cout << "n_transformed: " << n_transformed << std::endl; ➐
}
-----------------------------------------------------------------------
n_transformed: 187215 ➐
```

*示例 19-16：由于非原子访问`n_transformed`而导致的竞态条件程序*

你首先初始化两个`vector`对象，`numbers`和`squares`，它们包含一百万个元素 ➊。接着，你使用`iota`填充其中一个 ➋，并将变量`n_transformed`初始化为`0` ➌。然后，你使用并行执行策略调用`transform`，将`numbers`作为目标序列，`squares`作为结果序列，并传入一个简单的 lambda ➍。这个 lambda 会递增`n_transformed` ➎，并返回参数`x`的平方 ➏。由于多个线程会执行这个 lambda，必须对`n_transformed`的访问进行同步 ➐。

上一节介绍了两种解决此问题的方法：锁和原子操作。在这种情况下，最好的方法可能就是直接使用`std::atomic_size_t`来替代`size_t`。

### 总结

本章对并发性和并行性进行了非常高层次的概述。此外，你还学习了如何启动异步任务，这使你能够轻松地将多线程编程概念引入你的代码中。尽管将并行和并发概念引入程序中可以显著提升性能，但你必须小心避免引入竞态条件，这些竞态条件可能导致未定义的行为。你还学习了几种同步访问可变共享状态的机制：互斥锁、条件变量和原子操作。

**练习**

**19-1.** 编写你自己的基于自旋锁的互斥锁，命名为 `SpinLock`。暴露 `lock`、`try_lock` 和 `unlock` 方法。你的类应该删除拷贝构造函数。尝试使用 `std::lock_guard<SpinLock>` 来管理你的类的实例。

**19-2.** 阅读著名的双重检查锁定模式（DCLP）及其不应该使用的原因。（参见 Scott Meyers 和 Andrei Alexandrescu 在“进一步阅读”部分提到的文章。）然后了解如何使用 `std::call_once` 确保可调用对象只被调用一次，详见 [thread.once.callonce]。

**19-3.** 创建一个线程安全的队列类。该类必须暴露一个类似于 `std::queue` 的接口（见 [queue.defn]）。内部使用 `std::queue` 来存储元素。使用 `std::mutex` 来同步访问这个内部的 `std::queue`。

**19-4.** 向你的线程安全队列添加 `wait_and_pop` 方法和一个 `std::condition_variable` 成员。当用户调用 `wait_and_pop` 且队列包含元素时，它应该弹出队列中的元素并返回。如果队列为空，线程应该阻塞，直到有元素可用，然后继续弹出元素。

**19-5.** （可选）阅读 Boost Coroutine2 文档，特别是“概述”、“介绍”和“动机”部分。

**进一步阅读**

+   “C++ 与双重检查锁定的危险：第一部分”由 Scott Meyers 和 Andrei Alexandrescu 编写（[*http://www.drdobbs.com/cpp/c-and-the-perils-of-double-checked-locki/184405726/*](http://www.drdobbs.com/cpp/c-and-the-perils-of-double-checked-locki/184405726/))

+   *ISO 国际标准 ISO/IEC (2017) — C++ 编程语言*（国际标准化组织；瑞士日内瓦； *[`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/)*）

+   *C++ 并发实战*，第二版，作者：Anthony Williams（Manning，2018）

+   “有效的并发性：了解何时使用活动对象而非互斥锁”，由 Herb Sutter 编写（[*https://herbsutter.com/2010/09/24/effective-concurrency-know-when-to-use-an-active-object-instead-of-a-mutex/*](https://herbsutter.com/2010/09/24/effective-concurrency-know-when-to-use-an-active-object-instead-of-a-mutex/))

+   *Effective Modern C++: 42 种改进你使用 C++ 11 和 C++ 14 的具体方法*，由 Scott Meyers 编写（O'Reilly Media，2014）

+   彼得·L·蒙哥马利的《现代整数分解算法综述》。《CWI 季刊》7.4（1994 年）：337–365。
