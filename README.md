# 推荐的C语言风格和编码规则

本文档描述了Tilen MAJERLE在其项目和库中使用的C代码风格。

## 目录

- [推荐的C语言风格和编码规则](#推荐的c语言风格和编码规则)
  - [目录](#目录)
  - [最重要的规则](#最重要的规则)
  - [与VSCode集成](#与vscode集成)
  - [使用的约定](#使用的约定)
  - [通用规则](#通用规则)
  - [注释](#注释)
  - [函数](#函数)
  - [变量](#变量)
  - [结构体、枚举、类型定义](#结构体枚举类型定义)
  - [复合语句](#复合语句)
    - [Switch语句](#switch语句)
  - [宏和预处理器指令](#宏和预处理器指令)
  - [文档](#文档)
  - [头文件/源文件](#头文件源文件)
  - [Clang格式集成](#clang格式集成)
  - [艺术风格配置](#艺术风格配置)
  - [Eclipse格式化器](#eclipse格式化器)

## 最重要的规则

让我们从[GNOME开发者](https://developer.gnome.org/documentation/guidelines/programming/coding-style.html)网站的引言开始。

> 编写代码时最重要的规则是：*检查周围的代码并尝试模仿它*。
>
> 作为维护者，收到与周围代码明显不同编码风格的补丁是令人沮丧的。这是不尊重的行为，就像有人穿着泥泞的鞋子踏进一间一尘不染的房子。
>
> 所以，无论本文档推荐什么，如果已经有编写好的代码而你正在对其进行补丁，即使这不是你最喜欢的风格，也要保持其当前风格的一致性。

## 与VSCode集成

VScode预装了`clang-format`工具（LLVM包的一部分），它旨在帮助开发人员在代码开发过程中使用自动格式化工具。

因此，它允许用户在文件更改（和保存）时格式化代码。
当文件保存时，vscode将尝试调用clang-format并格式化代码。要使用的规则在`.clang-format`文件中。如果clang-format在当前文件的路径中找不到规则，它将一直向上到根目录，直到找到一个。如果仍然没有可用的，则使用默认规则。

此存储库包含始终最新的`.clang-format`文件，其规则与所解释的规则匹配。
您可以将文件夹放在项目的根目录或甚至软件开发项目的根目录中 -> 为所有项目使用一个文件！

需要启用一些配置：
![VSCode配置](images/vscode-settings.png) 

## 使用的约定

本文档中的关键词*必须*（MUST）、*不得*（MUST NOT）、*要求*（REQUIRED）、*应当*（SHALL）、*不应当*（SHALL NOT）、*应该*（SHOULD）、*不应该*（SHOULD NOT）、*推荐*（RECOMMENDED）、*不推荐*（NOT RECOMMENDED）、*可以*（MAY）和*可选*（OPTIONAL）应按照BCP 14 [RFC2119] [RFC8174]中描述的方式解释。

## 通用规则

这里列出了最明显和重要的通用规则。在继续其他章节之前，请仔细检查它们。

- `clang-format` 应该与附在此存储库的格式化文件一起使用（最低版本`15.x`）
- 使用`C11`标准
- 不要使用制表符，使用空格代替
- 每个缩进级别使用`4`个空格
- 关键词和开括号之间使用`1`个空格
```c
/* OK */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)

/* Wrong */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

- 函数名和开括号之间不要使用空格
```c
int32_t a = sum(4, 3);              /* OK */
int32_t a = sum (4, 3);             /* Wrong */
```

- 永远不要在变量/函数/宏/类型前使用`__`或`_`前缀。这是为C语言本身保留的
    - 对于严格的模块私有（静态）函数，推荐使用`prv_`名称前缀
    - 对于库内部函数，推荐使用`libname_int_`或`libnamei_`前缀，这些函数不应该被用户应用程序使用，但必须在不同的库内部模块中使用
- 变量/函数/类型只使用小写字符，可选地使用下划线`_`字符
- 开花括号总是与关键词（`for`、`while`、`do`、`switch`、`if`等）在同一行
```c
size_t i;
for (i = 0; i < 5; ++i) {           /* OK */
}
for (i = 0; i < 5; ++i){            /* Wrong */
}
for (i = 0; i < 5; ++i)             /* Wrong */
{
}
```

- 比较和赋值运算符前后使用单个空格
```c
int32_t a;
a = 3 + 4;              /* OK */
for (a = 0; a < 5; ++a) /* OK */
a=3+4;                  /* Wrong */
a = 3+4;                /* Wrong */
for (a=0;a<5;++a)       /* Wrong */
```

- 每个逗号后使用单个空格
```c
func_name(5, 4);        /* OK */
func_name(4,3);         /* Wrong */
```

- 不要将`全局`变量初始化为任何默认值（或`NULL`），在专用的`init`函数中实现它（如果需要）。
```c
static int32_t a;       /* Wrong */
static int32_t b = 4;   /* Wrong */
static int32_t a = 0;   /* Wrong */
```
> 在嵌入式系统中，RAM内存分布在系统的不同内存位置是很常见的。
> 当用户声明自定义RAM段时，处理所有情况很快就会变得棘手。
> 启动脚本负责设置默认值（.data和.bss），而其他自定义段可能不会填充默认值，这导致带有初始值的变量不会有任何效果。
>
> 为了独立于这种问题，为每个模块创建init函数并使用它来为所有变量设置默认值，如下所示：

```c
static int32_t a;       /* OK */
static int32_t b = 4;   /* Wrong - 如果链接器脚本和启动文件没有正确处理，这个值可能不会设置为零 */

void
my_module_init(void) {
    a = 0;
    b = 4;
}
```

- 在同一行中声明所有相同类型的局部变量
```c
void
my_func(void) {
    /* 1 */
    char a;             /* OK */
    
    /* 2 */
    char a, b;          /* OK */
    
    /* 3 */
    char a;
    char b;             /* Wrong, char类型的变量已经存在 */
}
```

- 按顺序声明局部变量
    1. 自定义结构和枚举
    2. 整数类型，更宽的无符号类型优先
    3. 单/双精度浮点数
```c
int
my_func(void) {
    /* 1 */
    my_struct_t my;     /* 首先是自定义结构 */
    my_struct_ptr_t* p; /* 指针也是 */

    /* 2 */
    uint32_t a;
    int32_t b;
    uint16_t c;
    int16_t g;
    char h;
    /* ... */

    /* 3 */
    double d;
    float f;
}
```

- 总是在块的开始，第一个可执行语句之前声明局部变量
- 在结构（或其子级）初始化的最后一个元素中总是添加尾逗号（这有助于clang-format正确格式化结构）。除非结构非常简单和简短
```c
typedef struct {
    int a, b;
} str_t;

str_t s = {
    .a = 1,
    .b = 2,   /* 这里的逗号 */
}

/* "复杂"结构的示例，有或缺少几个尾逗号，在clang-format运行格式化后 */
static const my_struct_t my_var_1 = {
    .type = TYPE1,
    .type_data =
        {
            .type1 =
                {
                    .par1 = 0,
                    .par2 = 1, /* 这里的尾逗号 */
                }, /* 这里的尾逗号 */
        },  /* 这里的尾逗号 */
};

static const my_struct_t my_var_2 = {.type = TYPE2,
                                     .type_data = {
                                         .type2 =
                                             {
                                                 .par1 = 0,
                                                 .par2 = 1,
                                             },
                                     }};    /* 这里缺少逗号 */
static const my_struct_t my_var_3 = {.type = TYPE3,
                                     .type_data = {.type3 = {
                                                       .par1 = 0,
                                                       .par2 = 1,
                                                   }}}; /* 这里缺少2个逗号 */

/* 没有尾逗号 - 仅适用于小型和简单的结构 */
static const my_struct_t my_var_4 = {.type = TYPE4, .type_data = {.type4 = {.par1 = 0, .par2 = 1}}};
```

- 在`for`循环中声明计数器变量
```c
/* OK */
for (size_t i = 0; i < 10; ++i)

/* OK，如果你以后需要计数器变量 */
size_t i;
for (i = 0; i < 10; ++i) {
    if (...) {
        break;
    }
}
if (i == 10) {

}

/* Wrong */
size_t i;
for (i = 0; i < 10; ++i) ...
```

- 避免在声明中使用函数调用进行变量赋值，除了单个变量
```c
void
a(void) {
    /* 声明变量时避免函数调用 */
    int32_t a, b = sum(1, 2);

    /* 使用这个 */
    int32_t a, b;
    b = sum(1, 2);

    /* 这样是可以的 */
    uint8_t a = 3, b = 4;
}
```

- 除了`char`、`float`或`double`，总是使用在`stdint.h`库中声明的类型，例如`uint8_t`表示`无符号8位`等。
- 不要使用`stdbool.h`库。分别使用`1`或`0`表示`true`或`false`
```c
/* OK */
uint8_t status;
status = 0;

/* Wrong */
#include <stdbool.h>
bool status = true;
```

- 永远不要与`true`比较，例如`if (check_func() == 1)`，使用`if (check_func()) { ... }`
- 总是将指针与`NULL`值比较
```c
void* ptr;

/* ... */

/* OK，与NULL比较 */
if (ptr == NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

- 总是使用*前置递增（和递减）*而不是*后置递增（和递减）*
```c
int32_t a = 0;
...

a++;            /* Wrong */
++a;            /* OK */

for (size_t j = 0; j < 10; ++j) {}  /* OK */
```

- 总是对长度或大小变量使用`size_t`
- 如果函数不应该修改`指针`指向的内存，总是对指针使用`const`
- 如果函数参数或变量不应该被修改，总是使用`const`
```c

/* 当d可以被修改时，d指向的数据不能被修改 */
void
my_func(const void* d) {

}

/* 当d和d指向的数据都不能被修改时 */
void
my_func(const void* const d) {

}

/* 不是必需的，但建议 */
void
my_func(const size_t len) {

}

/* 当d不应该在函数内部被修改时，只有d指向的数据可以被修改 */
void
my_func(void* const d) {

}
```

- 当函数可能接受任何类型的指针时，总是使用`void *`，不要使用`uint8_t *`
    - 函数必须在实现中处理适当的类型转换
```c
/*
 * 为了发送数据，函数不应该修改`data`变量指向的内存
 * 因此`const`关键词很重要
 *
 * 为了发送通用数据（或将它们写入文件）
 * 任何类型都可以作为数据传递，
 * 因此使用`void *`
 */
/* OK示例 */
void
send_data(const void* data, size_t len) { /* OK */
    /* 不要转换`void *`或`const void *` */
    const uint8_t* d = data;/* 函数处理内部使用的适当类型 */
}

void
send_data(const void* data, int len) {    /* Wrong，不要使用int */
}
```

- 总是对`sizeof`运算符使用括号
- 永远不要使用*变长数组*（VLA）。使用标准C`malloc`和`free`函数的动态内存分配，或者如果库/项目提供自定义内存分配，使用其实现
    - 查看[LwMEM](https://github.com/MaJerle/lwmem)，自定义内存管理库
```c
/* OK */
#include <stdlib.h>
void
my_func(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * n); /* OK，分配内存 */
    arr = malloc(sizeof *arr * n);  /* Wrong，sizeof运算符缺少括号 */
    if (arr == NULL) {
        /* FAIL，没有内存 */
    }

    free(arr);  /* 使用后释放内存 */
}

/* Wrong */
void
my_func(size_t size) {
    int32_t arr[size];  /* Wrong，不要使用VLA */
}
```

- 总是将变量与零比较，除非它被视为`布尔`类型
- 永远不要将`布尔处理`变量与零或一比较。使用NOT（`!`）代替
```c
size_t length = 5;  /* 计数器变量 */
uint8_t is_ok = 0;  /* 布尔处理变量 */
if (length)         /* Wrong，length不被视为布尔值 */
if (length > 0)     /* OK，length被视为包含多个值的计数器变量，不仅仅是0或1 */
if (length == 0)    /* OK，length被视为包含多个值的计数器变量，不仅仅是0或1 */

if (is_ok)          /* OK，变量被视为布尔值 */
if (!is_ok)         /* OK，-||- */
if (is_ok == 1)     /* Wrong，永远不要将布尔变量与1比较！ */
if (is_ok == 0)     /* Wrong，使用!进行负检查 */
```

- 总是使用`/* comment */`进行注释，即使是*单行*注释
- 在头文件中总是包括`C++`的`extern`关键词检查
- 每个函数必须包括*doxygen启用*的注释，即使函数是`static`
- 对函数、变量、注释使用英文名称/文本
- 变量使用*小写*字符
- 如果变量包含多个名称，使用*下划线*，例如`force_redraw`。不要使用`forceRedraw`
- 永远不要转换返回`void *`的函数，例如`uint8_t* ptr = (uint8_t *)func_returning_void_ptr();`，因为`void *`可以安全地提升为任何其他指针类型
    - 使用`uint8_t* ptr = func_returning_void_ptr();`代替
- 对于C标准库包含文件总是使用`<`和`>`，例如`#include <stdlib.h>`
- 对于自定义库总是使用`""`，例如`#include "my_library.h"`
- 当转换为指针类型时，总是将星号对齐到类型，例如`uint8_t* t = (uint8_t*)var_width_diff_type`
- 总是尊重项目或库中已经使用的代码风格

## 注释

- 不允许以`//`开头的注释。总是使用`/* comment */`，即使是单行注释
```c
//This is comment (wrong)
/* This is comment (ok) */
```

- 对于多行注释，每行使用`空格+星号`
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/**
 * Wrong, use double-asterisk only for doxygen documentation
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

- 注释时使用`12`个缩进（`12 * 4`个空格）偏移。如果语句大于`12`个缩进，使注释`4-空格`对齐（下面的示例）到下一个可用缩进
```c
void
my_func(void) {
    char a, b;

    a = call_func_returning_char_a(a);          /* 这是从行开始有12*4个空格缩进的注释 */
    b = call_func_returning_char_a_but_func_name_is_very_long(a);   /* 这是注释，对齐到4-空格缩进 */
}
```

## 函数

- 每个可能从其模块外部访问的函数，必须包括函数*原型*（或*声明*）
- 函数名必须是小写，可选地用下划线`_`字符分隔
```c
/* OK */
void my_func(void);
void myfunc(void);

/* Wrong */
void MYFunc(void);
void myFunc();
```

- 当函数返回指针时，将星号对齐到返回类型
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t a, int32_t b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```
- 对齐所有函数原型（具有相同/相似功能）以提高可读性
```c
/* OK，函数名对齐 */
void        set(int32_t a);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t a);
const char * get(void);
```

- 函数实现必须在单独的行中包括返回类型和可选的其他关键词
```c
/* OK */
int32_t
foo(void) {
    return 0;
}

/* OK */
static const char*
get_string(void) {
    return "Hello world!\r\n";
}

/* Wrong */
int32_t foo(void) {
    return 0;
}
```

## 变量

- 使变量名全部小写，可选使用下划线`_`字符
```c
/* OK */
int32_t a;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t A;
int32_t myVar;
int32_t MYVar;
```

- 按`类型`将局部变量分组在一起
```c
void
foo(void) {
    int32_t a, b;   /* OK */
    char a;
    char b;         /* Wrong，char类型已经存在 */
}
```

- 不要在第一个可执行语句后声明变量
```c
void
foo(void) {
    int32_t a;
    a = bar();
    int32_t b;      /* Wrong，已经有可执行语句 */
}
```

- 你可以在下一个缩进级别内声明新变量
```c
int32_t a, b;
a = foo();
if (a) {
    int32_t c, d;   /* OK，c和d在if语句作用域内 */
    c = foo();
    int32_t e;      /* Wrong，块内已经有可执行语句 */
}
```

- 声明指针变量时将星号对齐到类型
```c
/* OK */
char* a;

/* Wrong */
char *a;
char * a;
```

- 当声明多个指针变量时，你可以将它们声明为星号对齐到变量名
```c
/* OK */
char *p, *n;
```

## 结构体、枚举、类型定义

- 结构或枚举名必须是小写，单词之间可选使用下划线`_`字符
- 结构或枚举可能包含`typedef`关键词
- 所有结构成员必须是小写
- 所有枚举成员应该是大写
- 结构/枚举必须遵循doxygen文档语法

当声明结构时，它可以使用`3`种不同选项之一：

1. 当结构仅用*名称*声明时，它*不得*在其名称后包含`_t`后缀。
```c
struct struct_name {
    char* a;
    char b;
};
```
2. 当结构仅用*typedef*声明时，它*必须*在其名称后包含`_t`后缀。
```c
typedef struct {
    char* a;
    char b;
} struct_name_t;
```
3. 当结构用*名称和typedef*声明时，它*不得*为基本名称包含`_t`，并且*必须*为typedef部分在其名称后包含`_t`后缀。
```c
typedef struct struct_name {    /* 没有_t */
    char* a;
    char b;
    char c;
} struct_name_t;    /* _t */
```

错误声明的示例及其建议的更正
```c
/* a和b必须分为2行 */
/* 带有typedef的结构名必须包含_t后缀 */
typedef struct {
    int32_t a, b;
} a;

/* 更正版本 */
typedef struct {
    int32_t a;
    int32_t b;
} a_t;

/* 错误的名称，不得包含_t后缀 */
struct name_t {
    int32_t a;
    int32_t b;
};

/* 错误的参数，必须全部大写 */
typedef enum {
    MY_ENUM_TESTA,
    my_enum_testb,
} my_enum_t;
```

- 在声明时初始化结构时，使用`C99`初始化风格
```c
/* OK */
a_t a = {
    .a = 4,
    .b = 5,
};

/* Wrong */
a_t a = {1, 2};
```

- 当为函数句柄引入新的typedef时，使用`_fn`后缀
```c
/* 函数接受2个参数并返回uint8_t */
/* typedef的名称有`_fn`后缀 */
typedef uint8_t (*my_func_typedef_fn)(uint8_t p1, const char* p2);
```

## 复合语句

- 每个复合语句必须包括开花括号和闭花括号，即使它只包含`1`个嵌套语句
- 每个复合语句必须包括单个缩进；当嵌套语句时，为每个嵌套包括`1`个缩进大小
```c
/* OK */
if (c) {
    do_a();
} else {
    do_b();
}

/* Wrong */
if (c)
    do_a();
else
    do_b();

/* Wrong */
if (c) do_a();
else do_b();
```

- 在`if`或`if-else-if`语句的情况下，`else`必须与第一个语句的闭括号在同一行
```c
/* OK */
if (a) {

} else if (b) {

} else {

}

/* Wrong */
if (a) {

}
else {

}

/* Wrong */
if (a) {

}
else
{

}
```

- 在`do-while`语句的情况下，`while`部分必须与`do`部分的闭括号在同一行
```c
/* OK */
do {
    int32_t a;
    a = do_a();
    do_b(a);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

- 每个开括号都需要缩进
```c
if (a) {
    do_a();
} else {
    do_b();
    if (c) {
        do_c();
    }
}
```

- 复合语句必须包括花括号，即使在单个语句的情况下。下面的示例显示了不良做法
```c
if (a) do_b();
else do_c();

if (a) do_a(); else do_b();
```

- 空的`while`、`do-while`或`for`循环必须包括括号
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

- 如果`while`（或`for`、`do-while`等）是空的（在嵌入式编程中可能是这种情况），使用空的单行括号
```c
/* 在嵌入式硬件单元中等待位被设置 */
volatile uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* 等待位13准备好 */
while (*addr & (1 << 13)) {}        /* OK，空循环在花括号内不包含空格 */
while (*addr & (1 << 13)) { }       /* Wrong */
while (*addr & (1 << 13)) {         /* Wrong */

}
while (*addr & (1 << 13));          /* Wrong，缺少花括号。可能导致编译器警告或意外错误 */
```
- 总是按此顺序优先使用循环：`for`、`do-while`、`while`
- 如果可能，避免在循环块内递增变量，见示例

```c
/* 不推荐 */
int32_t a = 0;
while (a < 10) {
    .
    ..
    ...
    ++a;
}

/* 更好 */
for (size_t a = 0; a < 10; ++a) {

}

/* 更好，如果递增可能不在每个周期中发生 */
for (size_t a = 0; a < 10; ) {
    if (...) {
        ++a;
    }
}
```

- 内联`if`语句可能仅用于赋值或函数调用操作
```c
/* OK */
int a = condition ? if_yes : if_no; /* 赋值 */
func_call(condition ? if_yes : if_no); /* 函数调用 */
switch (condition ? if_yes : if_no) {...}   /* OK */

/* Wrong，这代码不好维护 */
condition ? call_to_function_a() : call_to_function_b();

/* 重构以获得更好的程序流程 */
if (condition) {
    call_to_function_a();
} else {
    call_to_function_b();
}
```

### Switch语句

- 为每个`case`语句添加*单个缩进*
- 在每个`case`或`default`语句中为`break`语句使用额外的*单个缩进*
```c
/* OK，每个case都有单个缩进 */
/* OK，每个break都有额外的缩进 */
switch (check()) {
    case 0:
        do_a();
        break;
    case 1:
        do_b();
        break;
    default:
        break;
}

/* Wrong，case缩进缺失 */
switch (check()) {
case 0:
    do_a();
    break;
case 1:
    do_b();
    break;
default:
    break;
}

/* Wrong */
switch (check()) {
    case 0:
        do_a();
    break;      /* Wrong，break必须有缩进，因为它在case下 */
    case 1:
    do_b();     /* Wrong，case下缺少缩进 */
    break;
    default:
        break;
}
```

- 总是包括`default`语句
```c
/* OK */
switch (var) {
    case 0:
        do_job();
        break;
    default:
        break;
}

/* Wrong，缺少default */
switch (var) {
    case 0:
        do_job();
        break;
}
```

- 如果需要局部变量，使用花括号并将`break`语句放在里面。
    - 将开花括号与`case`语句放在同一行
```c
switch (a) {
    /* OK */
    case 0: {
        int32_t a, b;
        char c;
        a = 5;
        /* ... */
        break;
    }

    /* Wrong */
    case 1:
    {
        int32_t a;
        break;
    }

    /* Wrong，break应该在里面 */
    case 2: {
        int32_t a;
    }
    break;
}
```

## 宏和预处理器指令

- 总是使用宏而不是字面常数，特别是数字
- 所有宏必须完全大写，可选使用下划线`_`字符，除非它们明确标记为函数，可能在将来用常规函数语法替换
```c
/* OK */
#define SQUARE(x)         ((x) * (x))

/* Wrong */
#define square(x)           ((x) * (x))
```

- 总是用括号保护输入参数
```c
/* OK */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))

/* Wrong */
#define MIN(x, y)           x < y ? x : y
```

- 总是用括号保护最终宏求值
```c
/* Wrong */
#define MIN(x, y)           (x) < (y) ? (x) : (y)
#define SUM(x, y)           (x) + (y)

/* 想象使用错误的SUM实现这个方程的结果 */
int32_t x = 5 * SUM(3, 4);  /* 期望结果是5 * 7 = 35 */
int32_t x = 5 * (3) + (4);  /* 它被求值为这个，最终结果= 19，这不是我们期望的 */

/* 正确的实现 */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))
#define SUM(x, y)           ((x) + (y))
```

- 当宏使用多个语句时，使用`do {} while (0)`语句保护这些语句
```c
typedef struct {
    int32_t px, py;
} point_t;
point_t p;                  /* 定义新点 */

/* 错误的实现 */

/* 定义宏来设置点 */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)    /* 2个语句。最后一个不应该实现分号 */

SET_POINT(&p, 3, 4);        /* 将点设置为位置3, 4。这求值为... */
(&p)->px = (3); (&p)->py = (4); /* ... 这个。在这个例子中这不是问题。 */

/* 考虑这个丑陋的代码，但是它按C标准是有效的（不推荐） */
if (a)                      /* 如果a为真 */
    if (b)                  /* 如果b为真 */
        SET_POINT(&p, 3, 4);/* 将点设置为x = 3, y = 4 */
    else
        SET_POINT(&p, 5, 6);/* 将点设置为x = 5, y = 6 */

/* 求值为下面的代码。你看到问题了吗？ */
if (a)
    if (b)
        (&p)->px = (3); (&p)->py = (4);
    else
        (&p)->px = (5); (&p)->py = (6);

/* 或者如果我们稍微重写它 */
if (a)
    if (b)
        (&p)->px = (3);
        (&p)->py = (4);
    else
        (&p)->px = (5);
        (&p)->py = (6);

/*
 * 问自己一个问题：`else`关键词属于哪个`if`语句？
 *
 * 基于代码的第一部分，答案是直接的。当我们检查`b`条件时，属于内部`if`语句
 * 实际答案：编译错误，因为`else`不属于任何地方
 */

/* 更好和正确的宏实现 */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)    /* 2个语句。while循环后没有分号 */
/* 或者更好 */
#define SET_POINT(p, x, y)  do {    \   /* 反斜杠表示语句在新行中继续 */
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)                             /* 2个语句。while循环后没有分号 */

/* 现在原始代码求值为 */
if (a)
    if (b)
        do { (&p)->px = (3); (&p)->py = (4); } while (0);
    else
        do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* `if`或`else`的每个部分只包含`1`个内部语句（do-while），因此这是有效的求值 */

/* 为了使代码完美，对每个if-ifelse-else语句使用括号 */
if (a) {                    /* 如果a为真 */
    if (b) {                /* 如果b为真 */
        SET_POINT(&p, 3, 4);/* 将点设置为x = 3, y = 4 */
    } else {
        SET_POINT(&p, 5, 6);/* 将点设置为x = 5, y = 6 */
    }
}
```

- 避免使用`#ifdef`或`#ifndef`。使用`defined()`或`!defined()`代替
```c
#ifdef XYZ
/* do something */
#endif /* XYZ */
```

- 总是记录`if/elif/else/endif`语句
```c
/* OK */
#if defined(XYZ)
/* 如果定义了XYZ就执行 */
#else /* defined(XYZ) */
/* 如果没有定义XYZ就执行 */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
/* 如果定义了XYZ就执行 */
#else
/* 如果没有定义XYZ就执行 */
#endif
```

- 不要在`#if`语句内缩进子语句
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* 当定义ABC时执行 */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* 当没有定义XYZ时执行 */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
    #if defined(ABC)
        /* 当定义ABC时执行 */
    #endif /* defined(ABC) */
#else /* defined(XYZ) */
    /* 当没有定义XYZ时执行 */
#endif /* !defined(XYZ) */
```

## 文档

文档化的代码允许doxygen解析并生成html/pdf/latex输出，因此在项目的早期阶段正确地做这件事是非常重要的。

- 对`变量`、`函数`和`结构/枚举`使用doxygen启用的文档风格
- 总是对doxygen使用`\`，不要使用`@`
- 总是从行开始使用`5x4`个空格（`5`个制表符）偏移文本
```c
/**
 * \brief           保存指向链表中第一个条目的指针
 *                  此文本的开头距离行开头5个制表符（20个空格）
 */
static
type_t* list;
```

- 每个结构/枚举成员必须包括文档
- 将不同结构成员之间的注释开始对齐到同一列
```c
/**
 * \brief           这是点结构
 * \note            此结构用于计算所有点
 *                      相关的内容
 */
typedef struct {
    int32_t x;                                  /*!< 点X坐标 */
    int32_t y;                                  /*!< 点Y坐标 */
    int32_t size;                               /*!< 点大小。
                                                    由于注释非常大，
                                                    你可以转到下一行 */
} point_t;

/**
 * \brief           点颜色枚举
 */
typedef enum {
    COLOR_RED,                                  /*!< 红色 */
    COLOR_GREEN,                                /*!< 绿色 */
    COLOR_BLUE,                                 /*!< 蓝色 */
} point_color_t;
```

- 函数的文档必须写在函数实现中（通常是源文件）
- 函数必须包括`brief`和所有参数文档
- 每个参数必须注明它是*输入*和*输出*的`in`或`out`
- 如果函数返回某些内容，函数必须包括`return`参数。这不适用于`void`函数
- 函数可以包括其他doxygen关键词，如`note`或`warning`
- 在参数名和其描述之间使用冒号`:`
```c
/**
 * \brief           将`2`个数相加
 * \param[in]       a: 第一个数
 * \param[in]       b: 第二个数
 * \return          输入值的和
 */
int32_t
sum(int32_t a, int32_t b) {
    return a + b;
}

/**
 * \brief           将`2`个数相加并将其写入指针
 * \note            此函数不返回值，而是将其存储到指针中
 * \param[in]       a: 第一个数
 * \param[in]       b: 第二个数
 * \param[out]      result: 用于保存结果的输出变量
 */
void
void_sum(int32_t a, int32_t b, int32_t* result) {
    *result = a + b;
}
```

- 如果函数返回枚举的成员，使用`ref`关键词指定哪一个
```c
/**
 * \brief           我的枚举
 */
typedef enum {
    MY_ERR,                                     /*!< 错误值 */
    MY_OK                                       /*!< OK值 */
} my_enum_t;

/**
 * \brief           检查某个值
 * \return          成功时返回\ref MY_OK，否则返回\ref my_enum_t的成员
 */
my_enum_t
check_value(void) {
    return MY_OK;
}
```

- 对常数或数字使用符号（\`NULL\` => `NULL`）
```c
/**
 * \brief           从输入数组获取数据
 * \param[in]       in: 输入数据
 * \return          成功时返回指向输出数据的指针，否则返回`NULL`
 */
const void *
get_data(const void* in) {
    return in;
}
```

- 宏的文档必须包括`hideinitializer` doxygen命令
```c
/**
 * \brief           获取`x`和`y`之间的最小值
 * \param[in]       x: 第一个值
 * \param[in]       y: 第二个值
 * \return          `x`和`y`之间的最小值
 * \hideinitializer
 */
#define MIN(x, y)       ((x) < (y) ? (x) : (y))
```

## 头文件/源文件

- 在文件末尾留一个空行
- 每个文件必须包括doxygen对`file`和`brief`描述的注释，后跟空行（当使用doxygen时）
```c
/**
 * \file            template.h
 * \brief           模板包含文件
 */
                    /* 这里是空行 */
```

- 每个文件（*头*文件或*源*文件）必须包括许可证（开头注释包括单个星号，因为这必须被doxygen忽略）
- 使用与项目/库已经使用的相同许可证
```c
/**
 * \file            template.h
 * \brief           模板包含文件
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
```

- 头文件必须包括保护`#ifndef`
- 头文件必须包括`C++`检查
- 在`C++`检查外包括外部头文件
- 首先包括STL C文件的外部头文件，然后是应用程序自定义文件
- 头文件必须只包括正确编译所需的每个其他头文件，但不能更多（.c应该包括剩余的，如果需要）
- 头文件必须只暴露模块公共变量/类型/函数
- 在头文件中对全局模块变量使用`extern`，稍后在源文件中定义它们
```
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* 这是头文件中的全局变量声明 */

#endif

/* file.c ... */
int32_t my_variable;        /* 实际上在源文件中定义 */
```
- 永远不要在另一个`.c`文件中包含`.c`文件
- `.c`文件应该首先包含相应的`.h`文件，然后是其他文件，除非明确必要
- 不要在头文件中包含模块私有声明

- 头文件示例（为了示例而省略许可证）
```c
/* 许可证在这里 */
#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* 包含头文件 */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/* 文件内容在这里 */

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */
```

## Clang格式集成

存储库带有始终最新的`.clang-format`文件，这是`clang-format`工具的输入配置。它可以与大多数最新技术IDE无缝集成，包括VSCode。然后在每次文件保存时即时进行格式化。

https://code.visualstudio.com/docs/cpp/cpp-ide#_code-formatting

## 艺术风格配置

[AStyle](http://astyle.sourceforge.net/)是一个很棒的软件，可以基于输入配置帮助格式化代码。

此存储库包含`astyle-code-format.cfg`文件，可以与`AStyle`软件一起使用。

```
astyle --options="astyle-code-format.cfg" "input_path/*.c,*.h" "input_path2/*.c,*.h"
```

> 艺术风格配置已过时，不再更新

## Eclipse格式化器

存储库包含`eclipse-ext-kr-format.xml`文件，可以与基于eclipse的工具链一起使用来设置格式化器选项。

它基于K&R格式化器，并修改以遵守上述规则。
您可以在eclipse设置中导入它，`Preferences -> LANGUAGE -> Code Style -> Formatter`标签。

```
