## 第四十一章。共享库基础

共享库是一种将库函数放入单一单元中，在运行时可以被多个进程共享的技术。这种技术可以节省磁盘空间和内存。本章介绍共享库的基础知识。下一章将介绍共享库的多个高级特性。

## 目标库

构建程序的一种方法是简单地将每个源文件编译成相应的目标文件，然后将所有这些目标文件链接在一起，生成可执行程序，如下所示：

```
$ `cc -g -c prog.c mod1.c mod2.c mod3.c`
$ `cc -g -o prog_nolib prog.o mod1.o mod2.o mod3.o`
```

### 注意

链接实际上是由独立的链接器程序*ld*执行的。当我们使用*cc*（或*gcc*）命令链接程序时，编译器会在后台调用*ld*。在 Linux 上，链接器应始终通过*gcc*间接调用，因为*gcc*确保*ld*以正确的选项调用，并将程序链接到正确的库文件。

然而，在许多情况下，我们可能有多个程序使用的源文件。作为节省工作量的第一步，我们可以只编译这些源文件一次，然后根据需要将它们链接到不同的可执行文件中。虽然这种技术节省了编译时间，但它仍然存在一个缺点，即我们必须在链接阶段指定所有的目标文件。此外，我们的目录可能会因为大量的目标文件而变得杂乱无章。

为了绕过这些问题，我们可以将一组目标文件组合成一个单一单元，称为*目标库*。目标库有两种类型：*静态*和*共享*。共享库是现代的目标库类型，相比静态库，它提供了多种优势，我们将在 41.3 节中描述。

#### 顺便说一下：在编译程序时包含调试器信息

在上面显示的*cc*命令中，我们使用了*-g*选项以在编译后的程序中包含调试信息。通常，始终创建允许调试的程序和库是个好主意。（在早期，调试信息有时被省略，这样生成的可执行文件占用更少的磁盘和内存，但如今磁盘和内存都便宜了。）

此外，在一些架构上，如 x86-32，* -fomit-frame-pointer*选项不应指定，因为这会使调试变得不可能。（在某些架构上，如 x86-64，该选项默认启用，因为它不会妨碍调试。）出于同样的原因，不应使用*strip(1)*去除可执行文件和库中的调试信息。

## 静态库

在开始讨论共享库之前，我们首先简要描述静态库，以便清楚地了解共享库的差异和优势。

静态库，也称为*档案*，是 UNIX 系统上提供的第一种库类型。它们提供以下好处：

+   我们可以将一组常用的目标文件放入一个库文件中，然后使用该库文件构建多个可执行文件，而无需在构建每个应用程序时重新编译原始源文件。

+   链接命令变得更简单。我们不再在链接命令行中列出一长串目标文件，而只需指定静态库的名称。链接器知道如何搜索静态库并提取可执行文件所需的目标。

#### 创建和维护静态库

实际上，静态库只是一个文件，包含所有添加到其中的目标文件的副本。归档还记录了每个组成目标文件的各种属性，包括文件权限、用户和组的数字 ID，以及最后的修改时间。按照惯例，静态库的名称形式为`lib`*name*`.a`。

静态库的创建和维护使用*ar(1)*命令，该命令的一般形式如下：

```
$ `ar` ``*`options archive object-file`*``...
```

*options*参数由一系列字母组成，其中一个是*操作码*，其他的是影响操作执行方式的*修饰符*。一些常用的操作码如下所示：

+   *r*（替换）：将目标文件插入到归档中，替换任何同名的目标文件。这是创建和更新归档的标准方法。因此，我们可能会使用以下命令构建归档：

    ```
    $ `cc -g -c mod1.c mod2.c mod3.c`
    $ `ar r libdemo.a mod1.o mod2.o mod3.o`
    $ `rm mod1.o mod2.o mod3.o`
    ```

    如上所示，构建完库后，如果需要，我们可以删除原始目标文件，因为它们不再需要。

+   *t*（目录表）：显示归档文件的目录表。默认情况下，这将列出归档中的所有目标文件的名称。通过添加*v*（详细）修饰符，我们还可以看到归档中记录的每个目标文件的其他属性，如以下示例所示：

    ```
    $ `ar tv libdemo.a`
    rw-r--r-- 1000/100 1001016 Nov 15 12:26 2009 mod1.o
    rw-r--r-- 1000/100 406668 Nov 15 12:21 2009 mod2.o
    rw-r--r-- 1000/100  46672 Nov 15 12:21 2009 mod3.o
    ```

    我们看到的每个目标文件的附加属性，从左到右依次是：添加到归档时的权限、用户 ID 和组 ID、文件大小，以及最后修改日期和时间。

+   *d*（删除）：从归档中删除指定的模块，如以下示例所示：

    ```
    $ `ar d libdemo.a mod3.o`
    ```

#### 使用静态库

我们可以通过两种方式将程序与静态库链接。第一种方式是在链接命令中指定静态库，如下所示：

```
$ `cc -g -c prog.c`
$ `cc -g -o prog prog.o libdemo.a`
```

或者，我们可以将库放置在链接器搜索的标准目录之一（例如`/usr/lib`）中，然后使用*-l*选项指定库的名称（即库文件名，不包括`lib`前缀和`.a`后缀）：

```
$ `cc -g -o prog prog.o -ldemo`
```

如果库位于链接器通常不搜索的目录中，我们可以使用*-L*选项指定链接器搜索这个额外的目录：

```
$ `cc -g -o prog prog.o -L```*`mylibdir`*`` `-ldemo`

```

Although a static library may contain many object modules, the linker includes only those modules that the program requires.

Having linked the program, we can run it in the usual way:

```

$ `./prog`

调用 mod1-x1

调用 mod2-x2

```

## Overview of Shared Libraries

When a program is built by linking against a static library (or, for that matter, without using a library at all), the resulting executable file includes copies of all of the object files that were linked into the program. Thus, when several different executables use the same object modules, each executable has its own copy of the object modules. This redundancy of code has several disadvantages:

*   Disk space is wasted storing multiple copies of the same object modules. Such wastage can be considerable.

*   If several different programs using the same modules are running at the same time, then each holds separate copies of the object modules in virtual memory, thus increasing the overall virtual memory demands on the system.

*   If a change is required (perhaps a security or bug fix) to an object module in a static library, then all executables using that module must be relinked in order to incorporate the change. This disadvantage is further compounded by the fact that the system administrator needs to be aware of which applications were linked against the library.

Shared libraries were designed to address these shortcomings. The key idea of a shared library is that a single copy of the object modules is shared by all programs requiring the modules. The object modules are not copied into the linked executable; instead, a single copy of the library is loaded into memory at run time, when the first program requiring modules from the shared library is started. When other programs using the same shared library are later executed, they use the copy of the library that is already loaded into memory. The use of shared libraries means that executable programs require less space on disk and (when running) in virtual memory.

### Note

Although the code of a shared library is shared among multiple processes, its variables are not. Each process that uses the library has its own copies of the global and static variables that are defined within the library.

Shared libraries provide the following further advantages:

*   Because overall program size is smaller, in some cases, programs can be loaded into memory and started more quickly. This point holds true only for large shared libraries that are already in use by another program. The first program to load a shared library will actually take longer to start, since the shared library must be found and loaded into memory.

*   Since object modules are not copied into the executable files, but instead maintained centrally in the shared library, it is possible (subject to limitations described in Compatible Versus Incompatible Libraries) to make changes to the object modules without requiring programs to be relinked in order to see the changes. Such changes can be carried out even while running programs are using an existing version of the shared library.

The principal costs of this added functionality are the following:

*   Shared libraries are more complex than static libraries, both at the conceptual level, and at the practical level of creating shared libraries and building the programs that use them.

*   Shared libraries must be compiled to use position-independent code (described in Position-Independent Code), which has a performance overhead on most architectures because it requires the use of an extra register ([Hubicka, 2003]).

*   *Symbol relocation* must be performed at run time. During symbol relocation, references to each symbol (a variable or function) in a shared library need to be modified to correspond to the actual run-time location at which the symbol is placed in virtual memory. Because of this relocation process, a program using a shared library may take a little more time to execute than its statically linked equivalent.

### Note

One further use of shared libraries is as a building block in the *Java Native Interface* (JNI), which allows Java code to directly access features of the underlying operating system by calling C functions within a shared library. For further information, see [Liang, 1999] and [Rochkind, 2004].

## Creating and Using Shared Libraries—A First Pass

To begin understanding how shared libraries operate, we look at the minimum sequence of steps required to build and use a shared library. For the moment, we’ll ignore the convention that is normally used to name shared library files. This convention, described in Shared Library Versions and Naming Conventions, allows programs to automatically load the most up-to-date version of the libraries they require, and also allows multiple incompatible versions (so-called *major versions*) of a library to coexist peacefully.

In this chapter, we concern ourselves only with Executable and Linking Format (ELF) shared libraries, since ELF is the format employed for executables and shared libraries in modern versions of Linux, as well as in many other UNIX implementations.

### Note

ELF supersedes the older *a.out* and *COFF* formats.

### Creating a Shared Library

In order to build a shared version of the static library we created earlier, we perform the following steps:

```

$ `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

$ `gcc -g -shared -o libfoo.so mod1.o mod2.o mod3.o`

```

The first of these commands creates the three object modules that are to be put into the library. (We explain the *cc -fPIC* option in the next section.) The *cc -shared* command creates a shared library containing the three object modules.

By convention, shared libraries have the prefix `lib` and the suffix `.so` (for *shared object*).

In our examples, we use the *gcc* command, rather than the equivalent *cc* command, to emphasize that the command-line options we are using to create shared libraries are compiler-dependent. Using a different C compiler on another UNIX implementation will probably require different options.

Note that it is possible to compile the source files and create the shared library in a single command:

```

$ `gcc -g -fPIC -Wall mod1.c mod2.c mod3.c -shared -o libfoo.so`

```

However, to clearly distinguish the compilation and library building steps, we’ll write the two as separate commands in the examples shown in this chapter.

Unlike static libraries, it is not possible to add or remove individual object modules from a previously built shared library. As with normal executables, the object files within a shared library no longer maintain distinct identities.

### Position-Independent Code

The *cc -fPIC* option specifies that the compiler should generate *position-independent code*. This changes the way that the compiler generates code for operations such as accessing global, static, and external variables; accessing string constants; and taking the addresses of functions. These changes allow the code to be located at any virtual address at run time. This is necessary for shared libraries, since there is no way of knowing at link time where the shared library code will be located in memory. (The run-time memory location of a shared library depends on various factors, such as the amount of memory already taken up by the program that is loading the library and which other shared libraries the program has already loaded.)

On Linux/x86-32, it is possible to create a shared library using modules compiled without the *-fPIC* option. However, doing so loses some of the benefits of shared libraries, since pages of program text containing position-dependent memory references are not shared across processes. On some architectures, it is impossible to build shared libraries without the *-fPIC* option.

In order to determine whether an existing object file has been compiled with the *-fPIC* option, we can check for the presence of the name `_GLOBAL_OFFSET_TABLE_` in the object file’s symbol table, using either of the following commands:

```

$ `nm mod1.o | grep _GLOBAL_OFFSET_TABLE_`

$ `readelf -s mod1.o | grep _GLOBAL_OFFSET_TABLE_`

```

Conversely, if either of the following equivalent commands yields any output, then the specified shared library includes at least one object module that was not compiled with *-fPIC*:

```

$ `objdump --all-headers libfoo.so | grep TEXTREL`

$ `readelf -d libfoo.so | grep TEXTREL`

```

The string `TEXTREL` indicates the presence of an object module whose text segment contains a reference that requires run-time relocation.

We say more about the *nm*, *readelf*, and *objdump* commands in Section 41.5.

### Using a Shared Library

In order to use a shared library, two steps must occur that are not required for programs that use static libraries:

*   Since the executable file no longer contains copies of the object files that it requires, it must have some mechanism for identifying the shared library that it needs at run time. This is done by embedding the name of the shared library inside the executable during the link phase. (In ELF parlance, the library dependency is recorded in a `DT_NEEDED` tag in the executable.) The list of all of a program’s shared library dependencies is referred to as its *dynamic dependency list*.

*   At run time, there must be some mechanism for resolving the embedded library name—that is, for finding the shared library file corresponding to the name specified in the executable file—and then loading the library into memory, if it is not already present.

Embedding the name of the library inside the executable happens automatically when we link our program with a shared library:

```

$ `gcc -g -Wall -o prog prog.c libfoo.so`

```

If we now attempt to run our program, we receive the following error message:

```

$ `./prog`

./prog：加载共享库时出错：libfoo.so：无法打开

打开共享对象文件：没有此文件或目录

```

This brings us to the second required step: *dynamic linking*, which is the task of resolving the embedded library name at run time. This task is performed by the *dynamic linker* (also called the *dynamic linking loader* or the *run-time linker*). The dynamic linker is itself a shared library, named `/lib/ld-linux.so.2`, which is employed by every ELF executable that uses shared libraries.

### Note

The pathname `/lib/ld-linux.so.2` is normally a symbolic link pointing to the dynamic linker executable file. This file has the name `ld-`*version*`.so`, where *version* is the *glibc* version installed on the system—for example, `ld-2.11.so`. The pathname of the dynamic linker differs on some architectures. For example, on IA-64, the dynamic linker symbolic link is named `/lib/ld-linux-ia64.so.2`.

The dynamic linker examines the list of shared libraries required by a program and uses a set of predefined rules in order to find the library files in the file system. Some of these rules specify a set of standard directories in which shared libraries normally reside. For example, many shared libraries reside in `/lib` and `/usr/lib`. The error message above occurs because our library resides in the current working directory, which is not part of the standard list searched by the dynamic linker.

### Note

Some architectures (e.g., zSeries, PowerPC64, and x86-64) support execution of both 32-bit and 64-bit programs. On such systems, the 32-bit libraries reside in `*/lib` subdirectories, and the 64-bit libraries reside in `*/lib64` subdirectories.

#### The `LD_LIBRARY_PATH` environment variable

One way of informing the dynamic linker that a shared library resides in a nonstandard directory is to specify that directory as part of a colon-separated list of directories in the `LD_LIBRARY_PATH` environment variable. (Semicolons can also be used to separate the directories, in which case the list must be quoted to prevent the shell from interpreting the semicolons.) If `LD_LIBRARY_PATH` is defined, then the dynamic linker searches for the shared library in the directories it lists before looking in the standard library directories. (Later, we’ll see that a production application should never rely on `LD_LIBRARY_PATH`, but for now, this variable provides us with a simple way of getting started with shared libraries.) Thus, we can run our program with the following command:

```

$ `LD_LIBRARY_PATH=. ./prog`

调用了 mod1-x1

调用了 mod2-x2

```

The (*bash*, Korn, and Bourne) shell syntax used in the above command creates an environment variable definition within the process executing *prog*. This definition tells the dynamic linker to search for shared libraries in `.`, the current working directory.

### Note

An empty directory specification in the `LD_LIBRARY_PATH` list (e.g., the middle specification in *dirx::diry*) is equivalent to `.`, the current working directory (but note that setting `LD_LIBRARY_PATH` to an empty string does not achieve the same result). We avoid this usage (SUSv3 discourages the corresponding usage in the `PATH` environment variable).

#### Static linking and dynamic linking contrasted

Commonly, the term *linking* is used to describe the use of the linker, *ld*, to combine one or more compiled object files into a single executable file. Sometimes, the term *static* linking is used to distinguish this step from *dynamic* linking, the run-time loading of the shared libraries used by an executable. (Static linking is sometimes also referred to as *link editing*, and a static linker such as *ld* is sometimes referred to as a link editor.) Every program—including those that use shared libraries—goes through a static-linking phase. At run time, a program that employs shared libraries additionally undergoes dynamic linking.

### The Shared Library Soname

In the example presented so far, the name that was embedded in the executable and sought by the dynamic linker at run time was the actual name of the shared library file. This is referred to as the library’s *real name*. However, it is possible—in fact, usual—to create a shared library with a kind of alias, called a *soname* (the `DT_SONAME` tag in ELF parlance).

If a shared library has a soname, then, during static linking, the soname is embedded in the executable file instead of the real name, and subsequently used by the dynamic linker when searching for the library at run time. The purpose of the soname is to provide a level of indirection that permits an executable to use, at run time, a version of the shared library that is different from (but compatible with) the library against which it was linked.

In Shared Library Versions and Naming Conventions, we’ll look at the conventions used for the shared library real name and soname. For now, we show a simplified example to demonstrate the principles.

The first step in using a soname is to specify it when the shared library is created:

```

$ `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

$ `gcc -g -shared -Wl,-soname,libbar.so -o libfoo.so mod1.o mod2.o mod3.o`

```

The *-Wl,-soname,libbar.so* option is an instruction to the linker to mark the shared library `libfoo.so` with the soname `libbar.so`.

If we want to determine the soname of an existing shared library, we can use either of the following commands:

```

$ `objdump -p libfoo.so | grep SONAME`

SONAME      libbar.so

$ `readelf -d libfoo.so | grep SONAME`

0x0000000e (SONAME)      库的 soname：[libbar.so]

```

Having created a shared library with a soname, we then create the executable as usual:

```

$ `gcc -g -Wall -o prog prog.c libfoo.so`

```

However, this time, the linker detects that the library `libfoo.so` contains the soname `libbar.so` and embeds the latter name inside the executable.

Now when we attempt to run the program, this is what we see:

```

$ `LD_LIBRARY_PATH=. ./prog`

prog：加载共享库时出错：libbar.so：无法打开

共享对象文件：没有此文件或目录

```

The problem here is that the dynamic linker can’t find anything named `libbar.so`. When using a soname, one further step is required: we must create a symbolic link from the soname to the real name of the library. This symbolic link must be created in one of the directories searched by the dynamic linker. Thus, we could run our program as follows:

```

$ `ln -s libfoo.so libbar.so`

        *在当前目录中创建 soname 符号链接*

$ `LD_LIBRARY_PATH=. ./prog`

调用了 mod1-x1

调用了 mod2-x2

```

Figure 41-1 shows the compilation and linking steps involved in producing a shared library with an embedded soname, linking a program against that shared library, and creating the soname symbolic link needed to run the program.

![Creating a shared library and linking a program against it](img/41-1_SHLIBS-A-shlib-create.png.jpg)Figure 41-1. Creating a shared library and linking a program against it

Figure 41-2 shows the steps that occur when the program created in Figure 41-1 is loaded into memory in preparation for execution.

### Note

To find out which shared libraries a process is currently using, we can list the contents of the corresponding Linux-specific `/proc/`*PID/*`maps` file (Location of Shared Memory in Virtual Memory).

![Execution of a program that loads a shared library](img/41-2_SHLIBS-A-shlib-load.png.jpg)Figure 41-2. Execution of a program that loads a shared library

## Useful Tools for Working with Shared Libraries

In this section, we briefly describe a few tools that are useful for analyzing shared libraries, executable files, and compiled object (`.o`) files.

#### The *ldd* command

The *ldd(1)* (list dynamic dependencies) command displays the shared libraries that a program (or a shared library) requires to run. Here’s an example:

```

$ `ldd prog`

        libdemo.so.1 => /usr/lib/libdemo.so.1 (0x40019000)

        libc.so.6 => /lib/tls/libc.so.6 (0x4017b000)

        /lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)

```

The *ldd* command resolves each library reference (employing the same search conventions as the dynamic linker) and displays the results in the following form:

```

*library-name* => *resolves-to-path*

```

For most ELF executables, *ldd* will list entries for at least `ld-linux.so.2`, the dynamic linker, and `libc.so.6`, the standard C library.

### Note

The name of the C library is different on some architectures. For example, this library is named `libc.so.6.1` on IA-64 and Alpha.

#### The *objdump* and *readelf* commands

The *objdump* command can be used to obtain various information—including disassembled binary machine code—from an executable file, compiled object, or shared library. It can also be used to display information from the headers of the various ELF sections of these files; in this usage, it resembles *readelf*, which displays similar information, but in a different format. Sources of further information about *objdump* and *readelf* are listed at the end of this chapter.

#### The *nm* command

The *nm* command lists the set of symbols defined within an object library or executable program. One use of this command is to find out which of several libraries defines a symbol. For example, to find out which library defines the *crypt()* function, we could do the following:

```

$ `nm -A /usr/lib/lib*.so 2> /dev/null | grep ' crypt$'`

/usr/lib/libcrypt.so:00007080 W crypt

```

The *-A* option to *nm* specifies that the library name should be listed at the start of each line displaying a symbol. This is necessary because, by default, *nm* lists the library name once, and then, on subsequent lines, all of the symbols it contains, which isn’t useful for the kind of filtering shown in the above example. In addition, we discard standard error output in order to hide error messages about files in formats unrecognized by *nm*. From the above output, we can see that *crypt()* is defined in the *libcrypt* library.

## Shared Library Versions and Naming Conventions

Let’s consider what is entailed by shared library versioning. Typically, successive versions of a shared library are compatible with one another, meaning that the functions in each module present the same calling interface and are semantically equivalent (i.e., they achieve identical results). Such differing but compatible versions are referred to as *minor versions* of a shared library. Occasionally, however, it is necessary to create a new *major version* of a library—one that is incompatible with a previous version. (In Compatible Versus Incompatible Libraries, we’ll see more precisely what may cause such incompatibilities.) At the same time, it must still be possible to continue running programs that require the older version of the library.

To deal with these versioning requirements, a standard naming convention is employed for shared library real names and sonames.

#### Real names, sonames, and linker names

Each incompatible version of a shared library is distinguished by a unique *major version identifier*, which forms part of its real name. By convention, the major version identifier takes the form of a number that is sequentially incremented with each incompatible release of the library. In addition to the major version identifier, the real name also includes a *minor version identifier*, which distinguishes compatible minor versions within the library major version. The real name employs the format convention `lib`*name*`.so.`*major-id*`.`*minor-id*.

Like the major version identifier, the minor version identifier can be any string, but, by convention, it is either a number, or two numbers separated by a dot, with the first number identifying the minor version, and the second number indicating a patch level or revision number within the minor version. Some examples of real names of shared libraries are the following:

```

libdemo.so.1.0.1

libdemo.so.1.0.2              *次要版本，与版本 1.0.1 兼容*

libdemo.so.2.0.0              *新的主版本，与版本 1 不兼容*

libreadline.so.5.0

```

The soname of the shared library includes the same major version identifier as its corresponding real library name, but excludes the minor version identifier. Thus, the soname has the form `lib`*name*`.so.`*major-id*.

Usually, the soname is created as a relative symbolic link in the directory that contains the real name. The following are some examples of sonames, along with the real names to which they might be symbolically linked:

```

libdemo.so.1        -> libdemo.so.1.0.2

libdemo.so.2        -> libdemo.so.2.0.0

libreadline.so.5    -> libreadline.so.5.0

```

For a particular major version of a shared library, there may be several library files distinguished by different minor version identifiers. Normally, the soname corresponding to each major library version points to the most recent minor version within the major version (as shown in the above examples for `libdemo.so`). This setup allows for the correct versioning semantics during the run-time operation of shared libraries. Because the static-linking phase embeds a copy of the (minor version-independent) soname in the executable, and the soname symbolic link may subsequently be modified to point to a newer (minor) version of the shared library, it is possible to ensure that an executable loads the most up-to-date minor version of the library at run time. Furthermore, since different major versions of a library have different sonames, they can happily coexist and be accessed by the programs that require them.

In addition to the real name and soname, a third name is usually defined for each shared library: the *linker name*, which is used when linking an executable against the shared library. The linker name is a symbolic link containing just the library name without the major or minor version identifiers, and thus has the form `lib`*name*.`so`. The linker name allows us to construct version-independent link commands that automatically operate with the correct (i.e., most up-to-date) version of the shared library.

Typically, the linker name is created in the same directory as the file to which it refers. It can be linked either to the real name or to the soname of the most recent major version of the library. Usually, a link to the soname is preferable, so that changes to the soname are automatically reflected in the linker name. (In Installing Shared Libraries, we’ll see that the *ldconfig* program automates the task of keeping sonames up to date, and thus implicitly maintains linker names if we use the convention just described.)

### Note

If we want to link a program against an older major version of a shared library, we can’t use the linker name. Instead, as part of the link command, we would need to indicate the required (major) version by specifying a particular real name or soname.

The following are some examples of linker names:

```

libdemo.so           -> libdemo.so.2

libreadline.so       -> libreadline.so.5

```

Table 41-1 summarizes information about the shared library real name, soname, and linker name, and Figure 41-3 portrays the relationship between these names.

Table 41-1. Summary of shared library names

| Name | Format | Description |
| --- | --- | --- |
| real name | `lib` *name* `.so.` *maj* `.` *min* | File holding library code; one instance per major-plus-minor version of the library. |
| soname | `lib` *name* `.so.` *maj* | One instance per major version of library; embedded in executable at link time; used at run time to find library via a symbolic link with same name that points to corresponding (most up-to-date) real name. |
| linker name | `lib` *name* `.so` | Symbolic link to latest real name or (more usually) latest soname; single instance; allows construction of version-independent link commands. |

![Conventional arrangement of shared library names](img/41-3_SHLIBS-A-shlib-names-scale90.png.jpg)Figure 41-3. Conventional arrangement of shared library names

#### Creating a shared library using standard conventions

Putting all of the above information together, we now show how to build our demonstration library following the standard conventions. First, we create the object files:

```

$ `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

```

Then we create the shared library with the real name `libdemo.so.1.0.1` and the soname `libdemo.so.1`.

```

$ `gcc -g -shared -Wl,-soname,libdemo.so.1 -o libdemo.so.1.0.1 \`

`mod1.o mod2.o mod3.o`

```

Next, we create appropriate symbolic links for the soname and linker name:

```

$ `ln -s libdemo.so.1.0.1 libdemo.so.1`

$ `ln -s libdemo.so.1 libdemo.so`

```

We can employ *ls* to verify the setup (with *awk* used to select the fields of interest):

```

$ `ls -l libdemo.so* | awk '{print $1, $9, $10, $11}'`

lrwxrwxrwx libdemo.so -> libdemo.so.1

lrwxrwxrwx libdemo.so.1 -> libdemo.so.1.0.1

-rwxr-xr-x libdemo.so.1.0.1

```

Then we can build our executable using the linker name (note that the link command makes no mention of version numbers), and run the program as usual:

```

$ `gcc -g -Wall -o prog prog.c -L. -ldemo`

$ `LD_LIBRARY_PATH=. ./prog`

调用了 mod1-x1

调用了 mod2-x2

```

## Installing Shared Libraries

In the examples up to now, we created a shared library in a user-private directory, and then used the `LD_LIBRARY_PATH` environment variable to ensure that the dynamic linker searched that directory. Both privileged and unprivileged users may use this technique. However, this technique should not be employed in production applications. More usually, a shared library and its associated symbolic links are installed in one of a number of standard library directories, in particular, one of the following:

*   `/usr/lib`, the directory in which most standard libraries are installed;

*   `/lib`, the directory into which libraries required during system startup should be installed (since, during system startup, `/usr/lib` may not be mounted yet);

*   `/usr/local/lib`, the directory into which nonstandard or experimental libraries should be installed (placing libraries in this directory is also useful if `/usr/lib` is a network mount shared among multiple systems and we want to install a library just for use on this system); or

*   one of the directories listed in `/etc/ld.so.conf` (described shortly).

In most cases, copying a file into one of these directories requires superuser privilege.

After installation, the symbolic links for the soname and linker name must be created, usually as relative symbolic links in the same directory as the library file. Thus, to install our demonstration library in `/usr/lib` (whose permissions only allow updates by *root*), we would do the following:

```

$ `su`

密码：

# `mv libdemo.so.1.0.1 /usr/lib`

# `cd /usr/lib`

# `ln -s libdemo.so.1.0.1 libdemo.so.1`

# `ln -s libdemo.so.1 libdemo.so`

```

The last two lines in this shell session create the soname and linker name symbolic links.

#### *ldconfig*

The *ldconfig(8)* program addresses two potential problems with shared libraries:

*   Shared libraries can reside in a variety of directories. If the dynamic linker needed to search all of these directories in order to find a library, then loading libraries could be very slow.

*   As new versions of libraries are installed or old versions are removed, the soname symbolic links may become out of date.

The *ldconfig* program solves these problems by performing two tasks:

1.  It searches a standard set of directories and creates or updates a cache file, `/etc/ld.so.cache`, to contain a list of the (latest minor versions of each of the) major library versions in all of these directories. The dynamic linker in turn uses this cache file when resolving library names at run time. To build the cache, *ldconfig* searches the directories specified in the file `/etc/ld.so.conf` and then `/lib` and `/usr/lib`. The `/etc/ld.so.conf` file consists of a list of directory pathnames (these should be specified as absolute pathnames), separated by newlines, spaces, tabs, commas, or colons. In some distributions, the directory `/usr/local/lib` is included in this list. (If not, we may need to add it manually.)

    ### Note

    The command *ldconfig -p* displays the current contents of `/etc/ld.so.cache`.

2.  It examines the latest minor version (i.e., the version with the highest minor number) of each major version of each library to find the embedded soname and then creates (or updates) relative symbolic links for each soname in the same directory.

In order to correctly perform these actions, *ldconfig* expects libraries to be named according to the conventions described earlier (i.e., library real names include major and minor identifiers that increase appropriately from one library version to the next).

By default, *ldconfig* performs both of the above actions. Command-line options can be used to selectively inhibit either action: the *-N* option prevents rebuilding of the cache, and the *-X* option inhibits the creation of the soname symbolic links. In addition, the *-v* (*verbose*) option causes *ldconfig* to display output describing its actions.

We should run *ldconfig* whenever a new library is installed, an existing library is updated or removed, or the list of directories in `/etc/ld.so.conf` is changed.

As an example of the operation of *ldconfig*, suppose we wanted to install two different major versions of a library. We would do this as follows:

```

$ `su`

密码：

# `mv libdemo.so.1.0.1 libdemo.so.2.0.0 /usr/lib`

# `ldconfig -v | grep libdemo`

        libdemo.so.1 -> libdemo.so.1.0.1（已更改）

        libdemo.so.2 -> libdemo.so.2.0.0（已更改）

```

Above, we filter the output of *ldconfig*, so that we see just the information relating to libraries named `libdemo`.

Next, we list the files named `libdemo` in `/usr/lib` to verify the setup of the soname symbolic links:

```

# `cd /usr/lib`

# `ls -l libdemo* | awk '{print $1, $9, $10, $11}'`

lrwxrwxrwx libdemo.so.1 -> libdemo.so.1.0.1

-rwxr-xr-x libdemo.so.1.0.1

lrwxrwxrwx libdemo.so.2 -> libdemo.so.2.0.0

-rwxr-xr-x libdemo.so.2.0.0

```

We must still create the symbolic link for the linker name, as shown in the next command:

```

# `ln -s libdemo.so.2 libdemo.so`

```

However, if we install a new 2.*x* minor version of our library, then, since the linker name points to the latest soname, *ldconfig* has the effect of also keeping the linker name up to date, as the following example shows:

```

# `mv libdemo.so.2.0.1 /usr/lib`

# `ldconfig -v | grep libdemo`

        libdemo.so.1 -> libdemo.so.1.0.1

        libdemo.so.2 -> libdemo.so.2.0.1（已更改）

```

If we are building and using a private library (i.e., one that is not installed in one of the standard directories), we can have *ldconfig* create the soname symbolic link for us by using the *-n* option. This specifies that *ldconfig* should process only libraries in the directories on the command line and should not update the cache file. In the following example, we use *ldconfig* to process libraries in the current working directory:

```

$ `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

$ `gcc -g -shared -Wl,-soname,libdemo.so.1 -o libdemo.so.1.0.1 \`

`mod1.o mod2.o mod3.o`

$ `/sbin/ldconfig -nv .`

.:

        libdemo.so.1 -> libdemo.so.1.0.1

$ `ls -l libdemo.so* | awk '{print $1, $9, $10, $11}'`

lrwxrwxrwx libdemo.so.1 -> libdemo.so.1.0.1

-rwxr-xr-x libdemo.so.1.0.1

```

In the above example, we specified the full pathname when running *ldconfig*, because we were using an unprivileged account whose `PATH` environment variable did not include the `/sbin` directory.

## Compatible Versus Incompatible Libraries

Over time, we may need to make changes to the code of a shared library. Such changes result in a new version of the library that is either *compatible* with previous version(s), meaning that we need to change only the minor version identifier of the library’s real name, or *incompatible*, meaning that we must define a new major version of the library.

A change to a library is compatible with an existing library version if *all* of the following conditions hold true:

*   The semantics of each public function and variable in the library remain unchanged. In other words, each function keeps the same argument list, and continues to produce its specified effect on global variables and returned arguments, and returns the same result value. Thus, changes that result in an improvement in performance or fix a bug (resulting in closer conformance to specified behavior) can be regarded as compatible changes.

*   No function or variable in the library’s public API is removed. It is, however, compatible to add new functions and variables to the public API.

*   Structures allocated within and returned by each function remain unchanged. Similarly, public structures exported by the library remain unchanged. One exception to this rule is that, under certain circumstances, new items may be added to the end of an existing structure, though even this may be subject to pitfalls if, for example, the calling program tries to allocate arrays of this structure type. Library designers sometimes circumvent this limitation by defining exported structures to be larger than is required in the initial release of the library, with some extra padding fields being “reserved for future use.”

If none of these conditions is violated, then the new library name can be updated by adjusting the minor version of the existing name. Otherwise, a new major version of the library should be created.

## Upgrading Shared Libraries

One of the advantages of shared libraries is that a new major or minor version of a library can be installed even while running programs are using an existing version. All that we need to do is create the new library version, install it in the appropriate directory, and update the soname and linker name symbolic links as required (or, more usually, have *ldconfig* do the job for us). To produce a new minor version (i.e., a compatible upgrade) of the shared library `/usr/lib/libdemo.1.0.1`, we would do the following:

```

$ `su`

密码:

# `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

# `gcc -g -shared -Wl,-soname,libdemo.so.1 -o libdemo.so.1.0.2 \`

`mod1.o mod2.o mod3.o`

# `mv libdemo.so.1.0.2 /usr/lib`

# `ldconfig -v | grep libdemo`

        libdemo.so.1 -> libdemo.so.1.0.2 (已更改)

```

Assuming the linker name was already correctly set up (i.e., to point to the library soname), we would not need to modify it.

Already running programs will continue to use the previous minor version of the shared library. Only when they are terminated and restarted will they too use the new minor version of the shared library.

If we subsequently wanted to create a new major version (2.0.0) of the shared library, we would do the following:

```

# `gcc -g -c -fPIC -Wall mod1.c mod2.c mod3.c`

# `gcc -g -shared -Wl,-soname,libdemo.so.2 -o libdemo.so.2.0.0 \`

`mod1.o mod2.o mod3.o`

# `mv libdemo.so.2.0.0 /usr/lib`

# `ldconfig -v | grep libdemo`

        libdemo.so.1 -> libdemo.so.1.0.2

        libdemo.so.2 -> libdemo.so.2.0.0 (已更改)

# `cd /usr/lib`

# `ln -sf libdemo.so.2 libdemo.so`

```

As can be seen in the above output, *ldconfig* automatically creates a soname symbolic link for the new major version. However, as the last command shows, we must manually update the linker name symbolic link.

## Specifying Library Search Directories in an Object File

We have already seen two ways of informing the dynamic linker of the location of shared libraries: using the `LD_LIBRARY_PATH` environment variable and installing a shared library into one of the standard library directories (`/lib`, `/usr/lib`, or one of the directories listed in `/etc/ld.so.conf`).

There is a third way: during the static editing phase, we can insert into the executable a list of directories that should be searched at run time for shared libraries. This is useful if we have libraries that reside in fixed locations that are not among the standard locations searched by the dynamic linker. To do this, we employ the *-rpath* linker option when creating an executable:

```

$ `gcc -g -Wall -Wl,-rpath,/home/mtk/pdir -o prog prog.c libdemo.so`

```

The above command copies the string */home/mtk/pdir* into the run-time library path (*rpath*) list of the executable *prog*, so, that when the program is run, the dynamic linker will also search this directory when resolving shared library references.

If necessary, the *-rpath* option can be specified multiple times; all of the directories are concatenated into a single ordered *rpath* list placed in the executable file. Alternatively, multiple directories can be specified as a colon-separated list within a single *-rpath* option. At run time, the dynamic linker searches the directories in the order they were specified in the *-rpath* option(s).

### Note

An alternative to the *-rpath* option is the `LD_RUN_PATH` environment variable. This variable can be assigned a string containing a series of colon-separated directories that are to be used as the *rpath* list when building the executable file. `LD_RUN_PATH` is employed only if the *-rpath* option is not specified when building the executable.

#### Using the *-rpath* linker option when building a shared library

The *-rpath* linker option can also be useful when building a shared library. Suppose we have one shared library, `libx1.so`, that depends on another, `libx2.so`, as shown in Figure 41-4. Suppose also that these libraries reside in the nonstandard directories `d1` and `d2`, respectively. We now go through the steps required to build these libraries and the program that uses them.

![A shared library that depends on another shared library](img/41-4_SHLIBS-A-shlib-interdep.png.jpg)Figure 41-4. A shared library that depends on another shared library

First, we build `libx2.so`, in the directory `pdir/d2`. (To keep the example simple, we dispense with library version numbering and explicit sonames.)

```

$ `cd /home/mtk/pdir/d2`

$ `gcc -g -c -fPIC -Wall modx2.c`

$ `gcc -g -shared -o libx2.so modx2.o`

```

Next, we build `libx1.so`, in the directory `pdir/d1`. Since `libx1.so` depends on `libx2.so`, which is not in a standard directory, we specify the latter’s run-time location with the *-rpath* linker option. This could be different from the link-time location of the library (specified by the *-L* option), although in this case the two locations are the same.

```

$ `cd /home/mtk/pdir/d1`

$ `gcc -g -c -Wall -fPIC modx1.c`

$ `gcc -g -shared -o libx1.so modx1.o -Wl,-rpath,/home/mtk/pdir/d2 \`

            `-L/home/mtk/pdir/d2 -lx2`

```

Finally, we build the main program, in the `pdir` directory. Since the main program makes use of `libx1.so`, and this library resides in a nonstandard directory, we again employ the *-rpath* linker option:

```

$ `cd /home/mtk/pdir`

$ `gcc -g -Wall -o prog prog.c -Wl,-rpath,/home/mtk/pdir/d1 \`

`-L/home/mtk/pdir/d1 -lx1`

```

Note that we did not need to mention `libx2.so` when linking the main program. Since the linker is capable of analyzing the *rpath* list in `libx1.so`, it can find `libx2.so`, and thus is able to satisfy the requirement that all symbols can be resolved at static link time.

We can use the following commands to examine `prog` and `libx1.so` in order to see the contents of their *rpath* lists:

```

$ `objdump -p prog | grep PATH`

RPATH       /home/mtk/pdir/d1         libx1.so *将在运行时从这里查找*

$ `objdump -p d1/libx1.so | grep PATH`

RPATH       /home/mtk/pdir/d2         libx2.so *将在运行时从这里查找*

```

### Note

We can also view the *rpath* lists by grepping the output of the *readelf --dynamic* (or, equivalently, *readelf -d*) command.

We can use the *ldd* command to show the complete set of dynamic dependencies of `prog`:

```

$ `ldd prog`

        libx1.so => /home/mtk/pdir/d1/libx1.so (0x40017000)

        libc.so.6 => /lib/tls/libc.so.6 (0x40024000)

        libx2.so => /home/mtk/pdir/d2/libx2.so (0x4014c000)

        /lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)

```

#### The ELF `DT_RPATH` and `DT_RUNPATH` entries

In the original ELF specification, only one type of *rpath* list could be embedded in an executable or shared library. This corresponded to the `DT_RPATH` tag in an ELF file. Later ELF specifications deprecated `DT_RPATH`, and introduced a new tag, `DT_RUNPATH`, for representing *rpath* lists. The difference between these two types of *rpath* lists is their relative precedence with respect to the `LD_LIBRARY_PATH` environment variable when the dynamic linker searches for shared libraries at run time: `DT_RPATH` has higher precedence, while `DT_RUNPATH` has lower precedence (refer to Finding Shared Libraries at Run Time).

By default, the linker creates the *rpath* list as a `DT_RPATH` tag. To have the linker instead create the *rpath* list as a `DT_RUNPATH` entry, we must additionally employ the *—enable-new-dtags (enable new dynamic tags)* linker option. If we rebuild our program using this option, and inspect the resulting executable file with *objdump*, we see the following:

```

$ `gcc -g -Wall -o prog prog.c -Wl,--enable-new-dtags \`

        `-Wl,-rpath,/home/mtk/pdir/d1 -L/home/mtk/pdir/d1 -lx1`

$ `objdump -p prog | grep PATH`

RPATH       /home/mtk/pdir/d1

RUNPATH     /home/mtk/pdir/d1

```

As can be seen, the executable contains both `DT_RPATH` and `DT_RUNPATH` tags. The linker duplicates the *rpath* list in this way for the benefit of older dynamic linkers that may not understand the `DT_RUNPATH` tag. (Support for `DT_RUNPATH` was added in version 2.2 of *glibc*.) Dynamic linkers that understand the `DT_RUNPATH` tag ignore the `DT_RPATH` tag (see Finding Shared Libraries at Run Time).

#### Using `$ORIGIN` in *rpath*

Suppose that we want to distribute an application that uses some of its own shared libraries, but we don’t want to require the user to install the libraries in one of the standard directories. Instead, we would like to allow the user to unpack the application under an arbitrary directory of their choice and then immediately be able to run the application. The problem is that the application has no way of determining where its shared libraries are located, unless it requests the user to set `LD_LIBRARY_PATH` or we require the user to run some sort of installation script that identifies the required directories. Neither of these approaches is desirable.

To get around this problem, the dynamic linker is built to understand a special string, `$ORIGIN` (or, equivalently, `${ORIGIN}`), in an *rpath* specification. The dynamic linker interprets this string to mean “the directory containing the application.” This means that we can, for example, build an application with the following command:

```

$ `gcc -Wl,-rpath,'$ORIGIN'/lib` ...

```

This presumes that at run time the application’s shared libraries will reside in the subdirectory `lib` under the directory that contains the application executable. We can then provide the user with a simple installation package that contains the application and associated libraries, and the user can install the package in any location and then run the application (i.e., a so-called “turn-key application”).

## Finding Shared Libraries at Run Time

When resolving library dependencies, the dynamic linker first inspects each dependency string to see if it contains a slash (`/`), which can occur if we specified an explicit library pathname when linking the executable. If a slash is found, then the dependency string is interpreted as a pathname (either absolute or relative), and the library is loaded using that pathname. Otherwise, the dynamic linker searches for the shared library using the following rules:

1.  If the executable has any directories listed in its `DT_RPATH` run-time library path list (*rpath*) and the executable does *not* contain a `DT_RUNPATH` list, then these directories are searched (in the order that they were supplied when linking the program).

2.  If the `LD_LIBRARY_PATH` environment variable is defined, then each of the colon-separated directories listed in its value is searched in turn. If the executable is a set-user-ID or set-group-ID program, then `LD_LIBRARY_PATH` is ignored. This is a security measure to prevent users from tricking the dynamic linker into loading a private version of a library with the same name as a library required by the executable.

3.  If the executable has any directories listed in its `DT_RUNPATH` run-time library path list, then these directories are searched (in the order that they were supplied when linking the program).

4.  The file `/etc/ld.so.cache` is checked to see if it contains an entry for the library.

5.  The directories `/lib` and `/usr/lib` are searched (in that order).

## Run-Time Symbol Resolution

Suppose that a global symbol (i.e., a function or variable) is defined in multiple locations, such as in an executable and in a shared library, or in multiple shared libraries. How is a reference to that symbol resolved?

For example, suppose that we have a main program and a shared library, both of which define a global function, *xyz()*, and another function within the shared library calls *xyz()*, as shown in Figure 41-5.

![Resolving a global symbol reference](img/41-5_SHLIBS-A-symbol-res.png.jpg)Figure 41-5. Resolving a global symbol reference

When we build the shared library and the executable program, and then run the program, this is what we see:

```

$ `gcc -g -c -fPIC -Wall -c foo.c`

$ `gcc -g -shared -o libfoo.so foo.o`

$ `gcc -g -o prog prog.c libfoo.so`

$`LD_LIBRARY_PATH=. ./prog`

main-xyz

```

From the last line of output, we can see that the definition of *xyz()* in the main program overrides (interposes) the one in the shared library.

Although this may at first appear surprising, there is a good historical reason why things are done this way. The first shared library implementations were designed so that the default semantics for symbol resolution exactly mirrored those of applications linked against static equivalents of the same libraries. This means that the following semantics apply:

*   A definition of a global symbol in the main program overrides a definition in a library.

*   If a global symbol is defined in multiple libraries, then a reference to that symbol is bound to the first definition found by scanning libraries in the left-to-right order in which they were listed on the static link command line.

Although these semantics make the transition from static to shared libraries relatively straightforward, they can cause some problems. The most significant problem is that these semantics conflict with the model of a shared library as implementing a self-contained subsystem. By default, a shared library can’t guarantee that a reference to one of its own global symbols will actually be bound to the library’s definition of that symbol. Consequently, the properties of a shared library can change when it is aggregated into a larger unit. This can lead to applications breaking in unexpected ways, and also makes it difficult to perform divide-and-conquer debugging (i.e., trying to reproduce a problem using fewer or different shared libraries).

In the above scenario, if we wanted to ensure that the invocation of *xyz()* in the shared library actually called the version of the function defined within the library, then we could use the *-Bsymbolic* linker option when building the shared library:

```

$ `gcc -g -c -fPIC -Wall -c foo.c`

$ `gcc -g -shared -Wl,-Bsymbolic -o libfoo.so foo.o`

$ `gcc -g -o prog prog.c libfoo.so`

$`LD_LIBRARY_PATH=. ./prog`

foo-xyz

```

*-Bsymbolic* 链接器选项指定应优先将共享库中的全局符号引用绑定到该库中的定义（如果存在）。(请注意，无论是否使用此选项，从主程序调用 *xyz()* 始终会调用主程序中定义的 *xyz()* 版本。)

## 使用静态库代替共享库

尽管通常更倾向于使用共享库，但在某些情况下，静态库可能更为合适。特别是，静态链接的应用程序包含其运行时所需的所有代码这一事实可能是有利的。例如，如果用户无法或不希望在要使用程序的系统上安装共享库，或者如果程序需要在共享库不可用的环境（例如 *chroot* 监狱）中运行，那么静态链接非常有用。此外，即使是兼容的共享库升级，也可能无意中引入一个错误，导致应用程序崩溃。通过静态链接应用程序，我们可以确保它不受系统上共享库变化的影响，并且拥有运行所需的所有代码（虽然会增加程序大小，并导致磁盘和内存要求增加）。

默认情况下，当链接器可以选择相同名称的共享库和静态库时（例如，我们使用 *-Lsomedir -ldemo* 链接，而 `libdemo.so` 和 `libdemo.a` 都存在），将使用共享库版本。要强制使用静态库版本，可以执行以下操作之一：

+   在 *gcc* 命令行中指定静态库的路径名（包括 `.a` 扩展名）。

+   在 *gcc* 中指定 *-static* 选项。

+   使用 *gcc* 选项 *-Wl,-Bstatic* 和 *-Wl,-Bdynamic* 明确切换链接器在静态库和共享库之间的选择。这些选项可以与 *-l* 选项在 *gcc* 命令行中交替使用。链接器按指定顺序处理这些选项。

## 总结

对象库是编译后的目标模块的集合，可以被与库链接的程序使用。像其他 UNIX 实现一样，Linux 提供了两种类型的对象库：静态库，这是早期 UNIX 系统唯一可用的库类型，以及现代的共享库。

由于共享库相较于静态库提供了多个优点，因此它们是当代 UNIX 系统中使用的主要库类型。共享库的优点主要来自于这样的事实：当程序与库链接时，程序所需的目标模块副本不会包含在最终的可执行文件中。相反，链接器仅在可执行文件中包含关于运行时所需共享库的信息。当文件执行时，动态链接器使用这些信息加载所需的共享库。在运行时，所有使用相同共享库的程序共享内存中的单个库副本。由于共享库不会被复制到可执行文件中，并且所有程序在运行时都使用共享库的单一内存副本，因此共享库减少了系统所需的磁盘空间和内存。

共享库的 soname 提供了一种在运行时解析共享库引用的间接方式。如果一个共享库有 soname，那么在静态链接器生成的可执行文件中，记录的将是这个名字，而不是库的真实名称。版本控制方案是，给共享库分配一个真实名称，格式为`lib`*name.so.major-id.minor-id*，而 soname 的格式为`lib`*name.so.major-id*，这种方式可以创建自动使用共享库最新小版本的程序（无需重新链接程序），同时也允许创建新的、不兼容的大版本共享库。

为了在运行时找到共享库，动态链接器遵循一套标准的搜索规则，其中包括搜索一组目录（例如，/`lib` 和 /`usr/lib`），大多数共享库通常安装在这些目录中。

#### 更多信息

与静态库和共享库相关的各种信息可以在*ar(1)*、*gcc(1)*、*ld(1)*、*ldconfig(8)*、*ld.so(8)*、*dlopen(3)*、*objdump(1)*的手册页中找到，此外，还可以在*ld*和*readelf*的*info*文档中找到相关信息。[Drepper, 2004 (b)] 详细介绍了在 Linux 上编写共享库的许多细节。更多有用的信息也可以在 David Wheeler 的*程序库 HOWTO*中找到，该文档可以在 LDP 网站上查阅，网址为[`www.tldp.org/`](http://www.tldp.org/)。GNU 共享库方案与 Solaris 中实现的方案有很多相似之处，因此阅读 Sun 的*链接器和库指南*（可以在[`docs.sun.com/`](http://docs.sun.com/)找到）以获取更多信息和示例是值得的。[Levine, 2000] 提供了静态链接器和动态链接器操作的介绍。

关于 GNU *Libtool*的信息，这是一种屏蔽程序员与构建共享库实现相关细节的工具，可以在线查阅，网址为[`www.gnu.org/software/libtool`](http://www.gnu.org/software/libtool)，并且在[Vaughan et al., 2000]中有介绍。

该文档《*可执行与链接格式*》由*工具接口标准*委员会提供，详细介绍了 ELF。该文档可以在线查阅，网址为[`refspecs.freestandards.org/elf/elf.pdf`](http://refspecs.freestandards.org/elf/elf.pdf)。[Lu, 1995] 也提供了大量关于 ELF 的有用细节。

## 练习

1.  尝试在编译程序时使用与不使用*-static*选项，查看一个动态链接 C 库的可执行文件与一个静态链接 C 库的可执行文件之间的大小差异。
