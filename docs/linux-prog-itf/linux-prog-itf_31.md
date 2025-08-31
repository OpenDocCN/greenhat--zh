## 第三十一章。线程：线程安全性和每线程存储

本章扩展了 POSIX 线程 API 的讨论，提供了线程安全函数和一次性初始化的描述。我们还讨论了如何使用线程特定数据或线程局部存储，使现有的函数变得线程安全，而无需更改函数的接口。

## 线程安全性（和可重入性的再讨论）

如果一个函数可以被多个线程同时安全调用，则称该函数是*线程安全的*；换句话说，如果一个函数不是线程安全的，则我们不能在一个线程执行它时从另一个线程调用它。例如，下面的函数（类似于我们在保护共享变量的访问：互斥锁中看到的代码）不是线程安全的：

```
static int glob = 0;

static void
incr(int loops)
{
    int loc, j;
    for (j = 0; j < loops; j++) {
     loc = glob;
     loc++;
     glob = loc;
   }
}
```

如果多个线程同时调用此函数，*glob*中的最终值是不可预测的。该函数说明了一个函数不是线程安全的典型原因：它使用了所有线程共享的全局或静态变量。

有多种方法可以使一个函数变得线程安全。一种方法是将互斥锁与该函数（或者可能是与所有在库中共享相同全局变量的函数）关联，在调用该函数时锁定互斥锁，并在互斥锁返回时解锁它。这种方法的优点是简单。另一方面，这意味着一次只有一个线程可以执行该函数——我们称访问该函数是*串行化的*。如果线程在执行此函数时花费了大量时间，则这种串行化会导致并发性丧失，因为程序的线程无法再并行执行。

更复杂的解决方案是将互斥锁与共享变量关联。然后，我们确定函数的哪些部分是访问共享变量的临界区，并且只在这些临界区的执行过程中获取和释放互斥锁。这允许多个线程同时执行该函数并进行并行操作，除非多个线程需要执行一个临界区。

#### 非线程安全的函数

为了促进线程应用程序的开发，SUSv3 中规定的所有函数都必须以线程安全的方式实现，除了在表 31-1 中列出的函数。（这些函数中的许多在本书中未讨论。）

除了在表 31-1 中列出的函数，SUSv3 还规定了以下内容：

+   如果传递`NULL`参数，则*ctermid()*和*tmpnam()*函数不需要是线程安全的。

+   如果最终参数（*ps*）为`NULL`，则函数*wcrtomb()*和*wcsrtombs()*不需要是线程安全的。

SUSv4 对表 31-1 中的函数列表进行了修改，如下所示：

+   由于这些函数已从标准中移除，*ecvt()*, *fcvt()*, *gcvt()*, *gethostbyname()*, 和 *gethostbyaddr()* 被移除。

+   新增了*strsignal()*和*system()*函数。*system()*函数是不可重入的，因为它必须对信号处理做出操作，这些操作会影响整个进程。

标准并不禁止实现使表 31-1 中的函数变为线程安全。然而，即使这些函数在某些实现中是线程安全的，可移植的应用程序也不能依赖于所有实现都具备线程安全。

表 31-1。SUSv3 不要求线程安全的函数

| *asctime()**basename()**catgets()**crypt()**ctime()**dbm_clearerr()**dbm_close()**dbm_delete()**dbm_error()**dbm_fetch()**dbm_firstkey()**dbm_nextkey()**dbm_open()**dbm_store()**dirname()**dlerror()**drand48()**ecvt()**encrypt()**endgrent()**endpwent()**endutxent()* | *fcvt()**ftw()**gcvt()**getc_unlocked()**getchar_unlocked()**getdate()**getenv()**getgrent()**getgrgid()**getgrnam()**gethostbyaddr()**gethostbyname()**gethostent()**getlogin()**getnetbyaddr()**getnetbyname()**getnetent()**getopt()**getprotobyname()**getprotobynumber()**getprotoent()**getpwent()* | *getpwnam()**getpwuid()**getservbyname()**getservbyport()**getservent()**getutxent()**getutxid()**getutxline()**gmtime()**hcreate()**hdestroy()**hsearch()**inet_ntoa()**l64a()**lgamma()**lgammaf()**lgammal()**localeconv()**localtime()**lrand48()**mrand48()**nftw()* | *nl_langinfo()**ptsname()**putc_unlocked()**putchar_unlocked()**putenv()**pututxline()**rand()**readdir()**setenv()**setgrent()**setkey()**setpwent()**setutxent()**strerror()**strtok()**ttyname()**unsetenv()**wcstombs()**wctomb()* |
| --- | --- | --- | --- |

#### 可重入和不可重入函数

尽管使用临界区来实现线程安全比使用每个函数的互斥锁要好，但由于锁定和解锁互斥锁存在一定的开销，仍然有些低效。*可重入函数*在不使用互斥锁的情况下实现线程安全。它通过避免使用全局和静态变量来实现这一点。任何必须返回给调用者的信息，或在多次调用函数之间维护的信息，都会存储在调用者分配的缓冲区中。（我们第一次遇到可重入性是在讨论在可重入与异步信号安全函数中对全局变量的处理时。）然而，并非所有函数都能做到可重入。通常的原因如下：

+   从本质上讲，一些函数必须访问全局数据结构。*malloc*库中的函数就提供了一个很好的例子。这些函数维护着一个全局的空闲块链表。*malloc*库的函数通过使用互斥锁（mutex）来确保线程安全。

+   一些函数（在线程出现之前定义）其接口本质上是不可重入的，因为它们返回指向由函数静态分配的存储区的指针，或者它们使用静态存储在连续的调用之间保持信息。在表 31-1 中的大多数函数都属于这一类。例如，*asctime()*函数（转换破碎时间和可打印形式之间的转换）返回一个指向包含日期时间字符串的静态分配缓冲区的指针。

对于一些具有不可重入接口的函数，SUSv3 指定了以后缀*_r*结尾的可重入等效函数。这些函数要求调用者分配一个缓冲区，然后将其地址传递给函数，用于返回结果。这样，调用线程可以使用局部（栈）变量作为函数结果缓冲区。为此，SUSv3 指定了*asctime_r()*、*ctime_r()*、*getgrgid_r()*、*getgrnam_r()*、*getlogin_r()*、*getpwnam_r()*、*getpwuid_r()*、*gmtime_r()*、*localtime_r()*、*rand_r()*、*readdir_r()*、*strerror_r()*、*strtok_r()*和*ttyname_r()*。

### 注意

一些实现还提供了其他传统非重入函数的额外重入版本。例如，*glibc*提供了*crypt_r()*、*gethostbyname_r()*、*getservbyname_r()*、*getutent_r()*、*getutid_r()*、*getutline_r()*和*ptsname_r()*。然而，便携式应用程序不能依赖这些函数在其他实现中存在。在某些情况下，SUSv3 没有指定这些重入等效函数，因为存在一些替代传统函数的既优越又重入的替代方案。例如，*getaddrinfo()*是*gethostbyname()*和*getservbyname()*的现代重入替代方案。

## 一次性初始化

有时，线程应用程序需要确保某些初始化操作只执行一次，无论创建多少个线程。例如，一个互斥锁可能需要使用*pthread_mutex_init()*进行带有特殊属性的初始化，而该初始化必须只进行一次。如果我们是从主程序创建线程，那么通常很容易实现——我们在创建任何依赖初始化的线程之前完成初始化。然而，在库函数中，这是不可能的，因为调用程序可能在第一次调用库函数之前就创建了线程。因此，库函数需要一种方法，在它第一次被任何线程调用时执行初始化。

库函数可以使用*pthread_once()*函数执行一次性初始化。

```
#include <pthread.h>

int `pthread_once`(pthread_once_t **once_control*, void (**init*)(void));
```

### 注意

成功时返回 0，出错时返回正数错误代码。

*pthread_once()*函数使用参数*once_control*的状态来确保由*init*指向的调用者定义的函数仅执行一次，无论*pthread_once()*调用被调用多少次，或者是从多少个不同的线程调用。

*init*函数在没有任何参数的情况下被调用，因此具有以下形式：

```
void
init(void)
{
    /* Function body */
}
```

*once_control*参数是一个指向变量的指针，该变量必须使用值`PTHREAD_ONCE_INIT`静态初始化：

```
pthread_once_t once_var = PTHREAD_ONCE_INIT;
```

第一次调用指定指向特定*pthread_once_t*变量的指针的*pthread_once()*时，将修改*once_control*指向的变量的值，以确保后续对*pthread_once()*的调用不会再次调用*init*。

*pthread_once()*的一种常见用法是与线程特定数据一起使用，我们接下来将描述这一点。

### 注意

*pthread_once()*存在的主要原因是，在早期版本的 Pthreads 中，无法静态初始化互斥锁。相反，需要使用*pthread_mutex_init()*（[Butenhof, 1996]）。考虑到后续增加了静态分配的互斥锁，库函数现在可以使用静态分配的互斥锁和静态布尔变量来执行一次性初始化。尽管如此，*pthread_once()* 仍然作为一种方便的方式保留下来。

## 线程特定数据

使函数线程安全的最有效方法是使其可重入。所有新的库函数应该以这种方式实现。然而，对于现有的不可重入库函数（可能是在线程使用普及之前设计的），这种方法通常需要改变函数的接口，这意味着必须修改所有使用该函数的程序。

线程特定数据是一种使现有函数在不改变其接口的情况下变得线程安全的技术。使用线程特定数据的函数可能比可重入函数效率稍低，但可以让我们保持调用该函数的程序不变。

线程特定数据允许函数为每个调用该函数的线程维护一个变量的单独副本，如图 31-1 所示。线程特定数据是持久的；每个线程的变量在该线程调用函数之间持续存在。这允许函数在调用之间保持每线程的信息，并允许函数将不同的结果缓冲区（如果需要）传递给每个调用线程。

![线程特定数据（TSD）为函数提供每线程存储](img/31-1_THREADS-B1-TSD-scale90.png.jpg)图 31-1. 线程特定数据（TSD）为函数提供每线程存储

### 从库函数的角度看线程特定数据

为了理解线程特定数据 API 的使用，我们需要从使用线程特定数据的库函数的角度来考虑问题：

+   函数必须为每个调用该函数的线程分配一个单独的存储块。这个存储块需要在该线程第一次调用函数时分配。

+   在同一线程的后续调用中，函数需要能够获取第一次该线程调用函数时分配的存储块的地址。函数不能在自动变量中保持对该块的指针，因为自动变量在函数返回时消失；也不能将指针存储在静态变量中，因为每个静态变量在进程中只有一个实例。Pthreads API 提供了处理此任务的函数。

+   不同的（即独立的）函数可能需要各自的线程特定数据。每个函数都需要一种方法来标识其线程特定数据（一个键），与其他函数使用的线程特定数据区分开来。

+   该函数无法直接控制线程终止时发生的事情。当线程终止时，它可能正在执行函数外部的代码。然而，必须有某种机制（析构函数）来确保在线程终止时自动释放为该线程分配的存储块。如果没有做到这一点，随着线程不断创建、调用函数然后终止，可能会发生内存泄漏。

### 线程特定数据 API 概述

库函数使用线程特定数据的一般步骤如下：

1.  该函数创建了一个*键*，这是区分该函数使用的线程特定数据项与其他函数使用的线程特定数据项的手段。键是通过调用 *pthread_key_create()* 函数创建的。创建键只需在第一个线程调用该函数时进行一次。为此，使用了 *pthread_once()*。创建键不会分配任何线程特定数据的块。

1.  调用 *pthread_key_create()* 还有第二个目的：它允许调用者指定程序员定义的析构函数的地址，用于释放为此键分配的每个存储块（参见下一步）。当一个具有线程特定数据的线程终止时，Pthreads API 会自动调用析构函数，并传递一个指向该线程数据块的指针。

1.  该函数为每个调用它的线程分配一个线程特定数据块。使用 *malloc()*（或类似函数）完成此分配。每个线程第一次调用该函数时，仅为该线程进行一次分配。

1.  为了保存在前一步分配的存储的指针，该函数使用了两个 Pthreads 函数：*pthread_setspecific()* 和 *pthread_getspecific()*。调用 *pthread_setspecific()* 是向 Pthreads 实现发出的请求，表示“保存这个指针，并记录它与特定键（本函数的键）和特定线程（调用线程）相关联”。调用 *pthread_getspecific()* 执行互补任务，返回之前与给定键和调用线程相关联的指针。如果没有指针与特定键和线程相关联，则 *pthread_getspecific()* 返回 `NULL`。这就是一个函数如何判断它是首次由该线程调用，因此必须为该线程分配存储块的方式。

### 线程特定数据 API 的详细信息

本节提供了前面提到的每个函数的详细信息，并通过描述其典型实现方式，阐明线程特定数据的操作。下一节将展示如何使用线程特定数据编写线程安全的标准 C 库函数 *strerror()* 的实现。

调用 *pthread_key_create()* 创建一个新的线程特定数据键，该键通过指向 *key* 的缓冲区返回给调用者。

```
#include <pthread.h>

int `pthread_key_create`(pthread_key_t **key*, void (**destructor*)(void *));
```

### 注意

成功时返回 0，失败时返回正的错误号。

因为返回的键是由进程中的所有线程使用的，*key* 应该指向一个全局变量。

*destructor* 参数指向一个程序员定义的函数，形式如下：

```
void
dest(void *value)
{
    /* Release storage pointed to by 'value' */
}
```

当一个线程终止时，如果该线程与 *key* 关联的值为非 `NULL`，Pthreads API 会自动调用析构函数并将该值作为其参数传递。传递的值通常是指向该线程的线程特定数据块的指针。如果不需要析构函数，则可以将 *destructor* 指定为 `NULL`。

### 注意

如果一个线程有多个线程特定数据块，则析构函数调用的顺序是未指定的。析构函数应设计为相互独立运行。

查看线程特定数据的实现有助于我们理解它是如何使用的。一个典型的实现（NPTL 是典型的）涉及以下数组：

+   一个全局（即进程范围内）数组，用于存储线程特定数据键的信息；

+   一组每线程数组，每个数组包含指向为特定线程分配的所有线程特定数据块的指针（即，这个数组包含通过调用 *pthread_setspecific()* 存储的指针）。

在这个实现中，*pthread_key_t* 由 *pthread_key_create()* 返回的值仅仅是全局数组的一个索引，我们将其标记为 *pthread_keys*，其形式如图 31-2 所示。该数组的每个元素都是一个包含两个字段的结构体。第一个字段表示该数组元素是否正在使用（即，是否已经通过先前调用 *pthread_key_create()* 分配）。第二个字段用于存储指向该键的线程特定数据块的析构函数的指针（即，它是 *pthread_key_create()* 的 *destructor* 参数的副本）。

![线程特定数据键的实现](img/31-2_THREADS-B1-TSD-key-scale90.png.jpg)图 31-2. 线程特定数据键的实现

*pthread_setspecific()* 函数请求 Pthreads API 将 *value* 的副本保存在一个数据结构中，该数据结构将其与调用线程和 *key*（由先前调用 *pthread_key_create()* 返回的键）关联。*pthread_getspecific()* 函数执行相反的操作，返回先前为该线程与给定 *key* 关联的值。

```
#include <pthread.h>

int `pthread_setspecific`(pthread_key_t *key*, const void **value*);
```

### 注意

成功时返回 0，失败时返回正的错误号。

```
void `*pthread_getspecific`(pthread_key_t *key*);
```

### 注意

返回指针，如果没有与 *key* 关联的线程特定数据，则返回 `NULL`。

给*pthread_setspecific()*的*value*参数通常是指向调用者之前分配的内存块的指针。该指针将在线程终止时作为该*key*的析构函数的参数传递。

### 注意

*value*参数不需要是指向内存块的指针。它可以是某个标量值，可以通过类型转换（cast）赋值给*void **。在这种情况下，之前调用的*pthread_key_create()*会将*destructor*指定为`NULL`。

图 31-3 展示了用于存储*value*的数据结构的典型实现。在这个图中，我们假设*pthread_keys[1]*被分配给一个名为*myfunc()*的函数。对于每个线程，Pthreads API 维护一个指向线程特定数据块的指针数组。每个线程特定数组的元素与图 31-2 中显示的全局*pthread_keys*数组的元素一一对应。*pthread_setspecific()*函数为调用线程设置数组中与*key*对应的元素。

![用于实现线程特定数据（TSD）指针的数据结构](img/31-3_THREADS-B1-TSD-specific-scale90.png.jpg)图 31-3. 用于实现线程特定数据（TSD）指针的数据结构

当一个线程首次创建时，所有其线程特定数据指针都会被初始化为`NULL`。这意味着当我们的库函数首次被线程调用时，它必须先使用*pthread_getspecific()*来检查该线程是否已经有与*key*关联的值。如果没有，函数会分配一个内存块，并使用*pthread_setspecific()*保存指向该块的指针。在下一节中，我们展示了线程安全的*strerror()*实现的一个示例。

### 使用线程特定数据（Thread-Specific Data）API

在我们第一次描述标准的*strerror()*函数时，参考了系统调用和库函数错误处理一章，我们提到该函数可能返回一个指向静态分配字符串的指针作为其函数结果。这意味着*strerror()*可能不是线程安全的。在接下来的几页中，我们将探讨一个非线程安全的*strerror()*实现，并展示如何使用线程特定数据使该函数线程安全。

### 注意

在许多 UNIX 实现中，包括 Linux，标准 C 库提供的*strerror()*函数是线程安全的。然而，我们还是以*strerror()*为例，因为 SUSv3 并未要求该函数是线程安全的，其实现为线程特定数据的使用提供了一个简单的示例。

示例 31-1*实现")展示了一个简单的非线程安全的*strerror()*实现。该函数利用了*glibc*定义的一对全局变量：*_sys_errlist*是一个指向字符串的指针数组，这些字符串对应于*errno*中的错误号（例如，*_sys_errlist[EINVAL]*指向字符串*Invalid operation*），而*_sys_nerr*指定了*_sys_errlist*中的元素数量。

示例 31-1. 一个非线程安全的*strerror()*实现

```
`threads/strerror.c`

#define _GNU_SOURCE                 /* Get '_sys_nerr' and '_sys_errlist'
                                       declarations from <stdio.h> */

#include <stdio.h>
#include <string.h>           /* Get declaration of strerror() */

#define MAX_ERROR_LEN 256            /* Maximum length of string
                                        returned by strerror() */

static char buf[MAX_ERROR_LEN];     /* Statically allocated return buffer */

char *
strerror(int err)
{
    if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {
        snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);
    } else {
        strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);
        buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */
    }

    return buf;

}

      `threads/strerror.c`
```

我们可以使用示例 31-2")中的程序来演示示例 31-1*实现")中*strerror()*实现不是线程安全的后果。该程序从两个不同的线程调用*strerror()*，但仅在两个线程都调用过*strerror()*后才显示返回值。即使每个线程为*strerror()*指定了不同的值（`EINVAL`和`EPERM`）作为参数，当我们使用示例 31-1*实现")中展示的*strerror()*版本编译并链接此程序时，显示的结果如下：

```
$ `./strerror_test`
Main thread has called strerror()
Other thread about to call strerror()
Other thread: str (0x804a7c0) = Operation not permitted
Main thread:  str (0x804a7c0) = Operation not permitted
```

两个线程都显示了对应于`EPERM`的*errno*字符串，因为第二个线程（在*threadFunc*中）调用*strerror()*时覆盖了主线程中调用*strerror()*时写入的缓冲区。检查输出时，可以看到两个线程中的局部变量*str*指向相同的内存地址。

示例 31-2. 从两个不同线程调用*strerror()*

```
`threads/strerror_test.c`
#include <stdio.h>
#include <string.h>                 /* Get declaration of strerror() */
#include <pthread.h>
#include "tlpi_hdr.h"

static void *
threadFunc(void *arg)
{
    char *str;

    printf("Other thread about to call strerror()\n");
    str = strerror(EPERM);
    printf("Other thread: str (%p) = %s\n", str, str);

    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t t;
    int s;
    char *str;

    str = strerror(EINVAL);
    printf("Main thread has called strerror()\n");

    s = pthread_create(&t, NULL, threadFunc, NULL);
    if (s != 0)
        errExitEN(s, "pthread_create");

    s = pthread_join(t, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");

    printf("Main thread:  str (%p) = %s\n", str, str);

    exit(EXIT_SUCCESS);
}

      `threads/strerror_test.c`
```

示例 31-3*实现")展示了一个重新实现的*strerror()*，该实现使用线程特定数据来确保线程安全。

修订版的*strerror()*执行的第一步是调用*pthread_once()* ![](img/U004.png)，以确保此函数的首次调用（来自任何线程）会调用*createKey()* ![](img/U002.png)。*createKey()*函数调用*pthread_key_create()*来分配一个线程特定的数据键，该键存储在全局变量*strerrorKey* ![](img/U003.png)中。调用*pthread_key_create()*时，还会记录析构函数的地址 ![](img/U001.png)，该析构函数将用于释放与此键对应的线程特定缓冲区。

*strerror()*函数随后调用*pthread_getspecific()* ![](img/U005.png)来检索与*strerrorKey*对应的该线程唯一缓冲区的地址。如果*pthread_getspecific()*返回`NULL`，则说明该线程第一次调用*strerror()*，于是该函数使用*malloc()* ![](img/U006.png)分配一个新的缓冲区，并通过*pthread_setspecific()* ![](img/U007.png)保存缓冲区的地址。如果*pthread_getspecific()*返回非`NULL`值，则该指针指向一个已存在的缓冲区，该缓冲区是在该线程之前调用*strerror()*时分配的。

该*strerror()*实现的其余部分与我们之前展示的实现类似，不同之处在于*buf*是一个线程特定数据缓冲区的地址，而不是一个静态变量。

示例 31-3. 使用线程特定数据的线程安全*strerror()*实现

```
`threads/strerror_tsd.c`
    #define _GNU_SOURCE             /* Get '_sys_nerr' and '_sys_errlist'
                                   declarations from <stdio.h> */
    #include <stdio.h>
    #include <string.h>             /* Get declaration of strerror() */
    #include <pthread.h>
    #include "tlpi_hdr.h"

    static pthread_once_t once = PTHREAD_ONCE_INIT;
    static pthread_key_t strerrorKey;

    #define MAX_ERROR_LEN 256       /* Maximum length of string in per-thread
                                 buffer returned by strerror() */

    static void                     /* Free thread-specific data buffer */
  destructor(void *buf)
    {
        free(buf);
    }

    static void                     /* One-time key creation function */
  createKey(void)
    {
        int s;

        /* Allocate a unique thread-specific data key and save the address
           of the destructor for thread-specific data buffers */

    s = pthread_key_create(&strerrorKey, destructor);
        if (s != 0)
            errExitEN(s, "pthread_key_create");
    }
        char *
    strerror(int err)
    {
        int s;
        char *buf;

        /* Make first caller allocate key for thread-specific data */

    s = pthread_once(&once, createKey);
      if (s != 0)
            errExitEN(s, "pthread_once");

    buf = pthread_getspecific(strerrorKey);
      if (buf == NULL) {          /* If first call from this thread, allocate
                                       buffer for thread, and save its location */
        buf = malloc(MAX_ERROR_LEN);
          if (buf == NULL)
              errExit("malloc");

     s = pthread_setspecific(strerrorKey, buf);
       if (s != 0)
          errExitEN(s, "pthread_setspecific");
        }

        if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {
            snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);
        } else {
            strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);
            buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */
        }

        return buf;
    }

        `threads/strerror_tsd.c`
```

如果我们将测试程序（示例 31-2")）与新的*strerror()*版本（示例 31-3 实现")）编译并链接成可执行文件`strerror_test_tsd`，那么运行该程序时，我们会看到以下结果：

```
$ `./strerror_test_tsd`
Main thread has called strerror()
Other thread about to call strerror()
Other thread: str (0x804b158) = Operation not permitted
Main thread:  str (0x804b008) = Invalid argument
```

从这个输出中，我们看到新的*strerror()*版本是线程安全的。我们还看到，在两个线程中，局部变量*str*指向的地址是不同的。

### 线程特定数据实现限制

正如我们描述的线程特定数据通常是如何实现的那样，某些实现可能需要对它所支持的线程特定数据键的数量加以限制。SUSv3 要求实现至少支持 128 个（`_POSIX_THREAD_KEYS_MAX`）个键。应用程序可以通过定义`PTHREAD_KEYS_MAX`（定义在`<limits.h>`中）或者通过调用*sysconf(_SC_THREAD_KEYS_MAX)*来确定实现实际支持多少个键。Linux 支持最多 1024 个键。

即使是 128 个键，对于大多数应用程序来说也应该绰绰有余。这是因为每个库函数通常只会使用少量的键——通常只有一个。如果一个函数需要多个线程特定数据值，通常可以将它们放在一个只有一个关联线程特定数据键的结构中。

## 线程局部存储

与线程特定数据类似，线程局部存储提供了每个线程持久的存储空间。这个特性虽然不是标准的，但许多其他 UNIX 实现（例如 Solaris 和 FreeBSD）以相同或类似的形式提供了这个功能。

线程局部存储的主要优势是它比线程特定数据更容易使用。为了创建一个线程局部变量，我们只需在全局或静态变量的声明中加入`__thread`修饰符：

```
static __thread buf[MAX_ERROR_LEN];
```

每个线程都有自己的一份由此修饰符声明的变量副本。线程局部存储中的变量会持续存在，直到线程终止，此时存储空间会被自动释放。

注意以下关于线程局部变量声明和使用的要点：

+   如果在变量声明中指定了`static`或`extern`关键字，`__thread`关键字必须紧跟其后。

+   线程局部变量的声明可以像普通的全局或静态变量声明一样，包含一个初始化器。

+   C 的地址运算符（`&`）可以用来获取线程局部变量的地址。

线程局部存储需要内核（Linux 2.6 提供）、Pthreads 实现（NPTL 提供）和 C 编译器（在 x86-32 上的 *gcc* 3.3 及以后版本提供）的支持。

示例 31-4 线程安全实现")展示了一个使用线程局部存储的*strerror()*线程安全实现。如果我们将测试程序（示例 31-2")）与这个版本的*strerror()*一起编译并链接，生成可执行文件 `strerror_test_tls`，那么在运行程序时，我们会看到以下结果：

```
$ `./strerror_test_tls`
Main thread has called strerror()
Other thread about to call strerror()
Other thread: str (0x40376ab0) = Operation not permitted
Main thread:  str (0x40175080) = Invalid argument
```

示例 31-4. 使用线程局部存储的*strerror()*线程安全实现

```
`threads/strerror_tls.c`
#define _GNU_SOURCE                 /* Get '_sys_nerr' and '_sys_errlist'
                                       declarations from <stdio.h> */
#include <stdio.h>
#include <string.h>           /* Get declaration of strerror() */
#include <pthread.h>

#define MAX_ERROR_LEN 256           /* Maximum length of string in per-thread
                                       buffer returned by strerror() */

static __thread char buf[MAX_ERROR_LEN];
                                    /* Thread-local return buffer */

char *
strerror(int err)
{
    if (err < 0 || err >= _sys_nerr || _sys_errlist[err] == NULL) {
        snprintf(buf, MAX_ERROR_LEN, "Unknown error %d", err);
    } else {
        strncpy(buf, _sys_errlist[err], MAX_ERROR_LEN - 1);
        buf[MAX_ERROR_LEN - 1] = '\0';          /* Ensure null termination */
    }

    return buf;
}
     `threads/strerror_tls.c`
```

## 总结

如果一个函数可以在多个线程同时安全调用，则该函数被称为线程安全的。函数不是线程安全的通常原因是它使用了全局或静态变量。使一个非线程安全的函数在多线程应用中安全的方式之一是通过互斥锁保护对该函数的所有调用。然而，这种方法的缺点是它减少了并发性，因为在任何时刻，只有一个线程可以进入该函数。一种允许更高并发的方法是，仅在操作共享变量的函数部分（关键区段）周围添加互斥锁。

互斥锁可以用来使大多数函数变为线程安全，但它们带来性能损失，因为锁定和解锁互斥锁是有代价的。通过避免使用全局和静态变量，一个可重入函数可以在不使用互斥锁的情况下实现线程安全。

在 SUSv3 中规定的大多数函数都要求是线程安全的。SUSv3 还列出了少数几个不要求线程安全的函数。通常，这些是使用静态存储来向调用者返回信息或在连续调用之间保持信息的函数。根据定义，这些函数不可重入，且不能使用互斥锁来使其线程安全。我们考虑了两种大致等价的编码技术——线程特定数据和线程局部存储——它们可以用来使一个不安全的函数变得线程安全，而不需要改变其接口。这两种技术都允许函数为每个线程分配持久的存储。

#### 进一步信息

请参考总结中列出的进一步信息来源。

## 练习

1.  实现一个函数，*one_time_init(control, init)*，它执行与*pthread_once()*等效的操作。*control*参数应是指向一个静态分配的结构体的指针，该结构体包含一个布尔变量和一个互斥锁。布尔变量表示函数*init*是否已经被调用，互斥锁控制对该变量的访问。为了简化实现，您可以忽略诸如*init()*第一次从线程调用时失败或被取消的可能性（即不需要设计一种方案，使得如果发生此类事件，下一个调用*one_time_init()*的线程会重新尝试调用*init()*）。

1.  使用线程特定数据编写线程安全版本的*dirname()*和*basename()*（解析路径名字符串：*dirname()*和*basename()* and basename()")）。
