

# 附录 C 语言标准第五版（C23）



*与 Aaron Ballman 合作*

![](img/opener.jpg)

最新的（第五版）C 语言标准（ISO/IEC 9899:2024）被称为 C23。C23 保持了*C 的精神*，同时增加了新特性和功能，以提高语言的安全性、可靠性和功能性。

## 属性

[[attributes]] 语法被加入到 C23 中，用于指定各种源构造（如类型、对象、标识符或块）的附加信息（Ballman 2019）。在 C23 之前，类似的功能是以实现定义（非便携）方式提供的：

```
__declspec(deprecated)
__attribute__((warn_unused_result))
int func(const char *str)__attribute__((nonnull(1)));
```

从 C23 开始，可以按如下方式指定属性：

```
[[deprecated, nodiscard]]
int func(
  const char *str [[gnu::nonnull]]
);
```

类似于 C++，语法位置决定了分配方式。属性包括 deprecated、fallthrough、maybe_unused、nodiscard、unsequenced 和 reproducible。属性语法支持标准属性和厂商特定的属性。__has_c_attribute 条件包含运算符可用于功能测试。

## 关键字

C 语言经常因其丑陋的关键字而受到嘲笑。C 语言通常使用以下划线字符（_）开头、后跟大写字母的保留标识符来定义新的关键字。

C23 引入了这些关键字的更自然拼写方式（Gustedt 2022）。在表 A-1 中，左侧展示了使用这种约定的 C11 关键字，而右侧则展示了 C23 引入的更自然拼写。

表 A-1： 关键字拼写

| 值 | 类型 |
| --- | --- |
| _Bool | bool |
| _Static_assert | static_assert |
| _Thread_local | thread_local |
| _Alignof | alignof |
| _Alignas | alignas |

另一个更新是引入了 nullptr 常量。老旧的 NULL 宏具有指针类型或可能是整数类型。它会隐式转换为任何标量类型，因此在类型安全性上并不特别强。nullptr 常量的类型是 nullptr_t，并且仅会隐式转换为指针类型、void 或 bool。

## 整数常量表达式

整数常量表达式不是一种可移植的构造；厂商可以扩展它们。例如，func 中的 array 可能是，也可能不是一个可变长度数组（VLA）：

```
void func() {
  static const int size = 12;
  int array[size]; // might be a VLA
}
```

C23 添加了 constexpr 变量（它意味着 const 限定符），当你确实需要某些东西作为常量时（Gilding 和 Gustedt 2022a）：

```
void func() {
  static constexpr int Size = 12;
  int Array[Size]; // never a VLA
}
```

C23 目前不支持 constexpr 函数，仅支持对象。结构成员不能是 constexpr。

## 枚举类型

C 枚举类型在 C17 中看起来正常，但有一些奇怪的行为。例如，底层整数类型是实现定义的，可以是有符号整数类型或无符号整数类型。C23 现在允许程序员为枚举指定底层类型（Meneide 和 Pygott 2022）：

```
enum E : unsigned short {
  Valid = 0, // has type unsigned short
  NotValid = 0x1FFFF // error, too big
};

// can forward declare with fixed type
enum F : int;
```

你还可以声明比 int 更大的枚举常量：

```
// has underlying type unsigned long
enum G {
  BiggerThanInt = 0xFFFF'FFFF'0000L,
};
```

## 类型推导

C23 增强了使用类型推导的单一对象定义的 auto 类型说明符（Gilding 和 Gustedt 2022b）。这基本上和 C++ 中的想法一样，但 auto 不能出现在函数签名中：

```
const auto l = 0L; // l is const long
auto huh = "test"; // huh is char *, not char[5] or const char *
void func();
auto f = func; // f is void (*)()
auto x = (struct S){  // x is struct S
  1, 2, 3.0
};
#define swap(a, b) \
  do {auto t = (a); (a) = (b); (b) = t;} \
  while (0)
```

## typeof 运算符

C23 添加了对 typeof 和 typeof_unqual 运算符的支持。这些类似于 C++ 中的 decltype，用于根据另一种类型或表达式的类型来指定类型。typeof 运算符保留限定符，而 typeof_unqual 会去除限定符，包括 _Atomic。

### K&R C 函数

K&R C 允许声明没有原型的函数：

```
int f();
int f(a, b) int a, b; {return 0;}
```

K&R C 函数在 35 年前已被弃用，并最终将从标准中移除。所有函数现在都有原型。空参数列表曾意味着“接受任意数量的参数”，现在意味着“接受零个参数”，与 C++ 一致。通过变参函数签名可以模拟“接受零个或多个参数”的情况：int f(...);，这现在是可能的，因为 va_start 不再要求在 ... 前传递参数。

### 预处理器

C23 新增了一些功能来改进预处理。#elifdef 指令是 #ifdef 的补充，还包括 #elifndef 形式。#warning 指令是 #error 的补充，但不会停止翻译过程。__has_include 操作符用于检测头文件的存在，__has_c_attribute 操作符用于检测标准或供应商属性的存在。

#embed 指令通过预处理器将外部数据直接嵌入源代码中：

```
unsigned char buffer[] = {
#embed "/dev/urandom" limit(32) // embeds 32 chars from /dev/urandom
};
struct FileObject {
  unsigned int MagicNumber;
  unsigned _BitInt(8) RGBA[4];
  struct Point {
    unsigned int x, y;
  } UpperLeft, LowerRight;
} Obj = {
#if __has_embed(SomeFile.bin) == __STDC_EMBED_FOUND__
// embeds contents of file as struct
// initialization elements
#embed "SomeFile.bin"
#endif
};
```

### 整数类型与表示

从 C23 开始，二进制补码是唯一允许的整数表示方式（Bastien 和 Gustedt 2019）。有符号整数溢出仍然是未定义行为。int8_t、int16_t、int32_t 和 int64_t 类型现在可以在所有平台上便捷使用。[u]intmax_t 类型不再是最大类型，仅要求表示 long long 值，而非扩展或位精确的整数值。

C23 还引入了位精度整数类型（Blower 等，2020）。这些是有符号和无符号类型，允许你指定位宽。这些整数不会进行整数提升，因此它们保持你请求的大小。位宽包括符号位，因此 _BitInt(2) 是最小的有符号位精度整数。BITINT_MAXWIDTH 指定了位精度整数的最大宽度。它必须至少为 ULLONG_WIDTH，但可以大得多（Clang 支持大于 2M 位）。

在 C17 中，添加两个半字需要一些位操作：

```
unsigned int add(
  unsigned int L, unsigned int R)
{
  unsigned int LC = L & 0xF;
  unsigned int RC = R & 0xF;
  unsigned int Res = LC + RC;
  return Res & 0xF;
}
```

使用 _BitInt 会简单得多：

```
unsigned _BitInt(4) add(
  unsigned _BitInt(4) L,
  unsigned _BitInt(4) R)
{
  return L + R;
}
```

C23 还新增了二进制文字。整数文字 0b00101010101、0x155、341 和 0525 表示相同的值。现在，你还可以使用数字分隔符以提高可读性，例如：0b0000'1111'0000'1100、0xF'0C、3'852 和 07'414。

C23 最终检查了整数运算，能够检测加法、减法和乘法运算中的溢出和回绕（Svoboda 2021）：

```
#include <stdckdint.h> // new header

bool ckd_add(Type1 *Result, Type2 L, Type3 R);
bool ckd_sub(Type1 *Result, Type2 L, Type3 R);
bool ckd_mul(Type1 *Result, Type2 L, Type3 R);
```

除法不受支持，并且它只适用于除普通的 char、bool 或位精度整数以外的整数类型。Type1、Type2 和 Type3 可以是不同的类型。如果运算的数学结果可以用 Type1 表示，这些函数将返回 false；否则，它们将返回 true。这些函数简化了遵循 CERT C 编码标准和 MISRA C 指南，但编写操作时仍然很笨重。

## unreachable 函数宏

unreachable 函数宏在 <stddef.h> 中提供。它展开为一个无返回值的表达式；在执行过程中到达该表达式是未定义行为。这使得你可以向优化器提供有关无法到达的流程控制的提示（Gustedt 2021）。

就像你告诉优化器假设的任何内容一样，使用时要小心，因为即使你错误，优化器也会相信你。以下是一个典型的例子，展示了如何在实践中使用 unreachable：

```
#include <stdlib.h>
enum Color {Red, Green, Blue};
int func(enum Color C) {
  switch (C) {
    case Red: return do_red();
    case Green: return do_green();
    case Blue: return do_blue();
  }
  unreachable(); // unhandled value
}
```

## 位与字节工具  

C23 在<stdbit.h>头文件中引入了一组位和字节工具（Meneide 2023）。这些包括以下函数：  

+   计算位模式中 1 或 0 的数量

+   计算领先或尾随的 1 或 0 的数量  

+   查找第一个领先或尾随的 1 或 0  

+   测试是否设置了单个比特位  

+   确定表示一个值所需的最小比特数  

+   根据一个值确定下一个最小或最大二的幂  

例如，以下代码可以用于统计值中连续 0 的位数，从最高有效位开始：  

```
#include <stdbit.h>
void func(uint32_t V) {
  int N = stdc_leading_zeros(V);
  // use the leading zero count N
}
```

在 C23 之前，这个操作要复杂得多：  

```
void func(uint32_t V) {
  int N = 32;
  unsigned R;
  R = V >> 16;
  if (R != 0) {N --= 16; V = R;}
  R = V >> 8;
  if (R != 0) {N --= 8; V = R;}
  R = V >> 4;
  if (R != 0) {N --= 4; V = R;}
  R = V >> 2;
  if (R != 0) {N --= 2; V = R;}
  R = V >> 1;
  if (R != 0) N -= 2;
  else        N -= V;
  // use the leading zero count N
}
```

## IEEE 浮动点支持  

C23 通过集成 TS 18661-1、2 和 3（ISO/IEC TS 18661-1 2014，ISO/IEC TS 18661-2 2015，ISO/IEC TS 18661-3 2015）更新了 IEEE 浮动点支持。附录 F 现在与 IEEE 浮动点运算标准（IEEE 754-2019）保持一致。附录 F 还适用于十进制浮动点：_Decimal32、_Decimal64和_Decimal128。但是，十进制运算不能与二进制、复数或虚数浮动点混合使用。附录 H（之前是与语言无关的算术附录）支持交换、扩展浮动类型和非算术交换格式。它允许使用 binary16、图形处理单元（GPU）数据、二进制或十进制表示。

数学库更改支持对<math.h>操作于_DecimalN、_FloatN和_FloatNx类型的支持。增加了对指数、对数、幂运算以及基于π的三角函数的特殊变体；对最小值/最大值、总排序和数值属性测试的改进函数；以及支持在浮动点值与整数或字符串之间的转换进行精细控制的函数。

已新增memset_explicit函数，适用于你真的需要清除内存的情况。它与memset相同，但优化器无法删除对它的调用。strdup和strndup函数已从 POSIX 中采纳。
