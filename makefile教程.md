# 0. 程序的编译过程
  - 预处理  gcc -E xxx.c -o xxx.i 预处理编译器cpp将头文件展开，宏替换，注释去掉
  - 编译    gcc -S xxx.i -o xxx.s 编译器gcc词法分析、语法分析、语义分析等生成汇编文件
  - 汇编    gcc -c xxx.s -o xxx.o 汇编器as 将xxx.s 翻译成机器语言保存在xxx.o 中（二进制文本形式）
  - 链接    gcc xxx.o -o xxx      链接器ld 将函数库中相应的代码组合到目标文件中  
    将多个.o文件，或者.o文件和库文件链接成为可被操作系统执行的可执行程序（Linux环境下，可执行文件的格式为“ELF”格式）  
    链接器不检查函数所在的源文件，只检查所有.o文件中的定义的符号。将.o文件中使用的函数和其它.o或者库文件中的相关符号进行合并，  
    对所有文件中的符号进行重新安排（重定位），并链接系统相关文件（程序启动文件等）最终生成可执行程序。链接过程使用GNU 的“ld”工具
  - 静态库：又称为文档文件（Archive File）。它是多个.o文件的集合  
    Linux中静态库文件的后缀为“.a”。静态库中的各个成员（.o文件）没有特殊的存在格式，  
    仅仅是一个.o文件的集合。使用“ar”工具维护和管理静态库
  - 共享库：也是多个.o文件的集合，但是这些.o文件市编译器按照一种特殊的方式生成   （Linux中，共享库文件格式通常为“ELF”格式。共享库已经具备了可执行条件）  
    模块中各个成员的地址（变量引用和函数调用）都是相对地址。使用此共享库的程序在运行时，共享库被动态加载到内存并和主程序在内存中进行连接  
    多个可执行程序可共享库文件的代码段（多个程序可以共享的使用库中的某一个模块，共享代码，不共享数据）。另外共享库的成员对象可被执行（由libdl.so提供支持）
# 1. make规则
  ```
    target : prerequisites
        recipe
    target 是一个目标文件，可以是一个可执行文件或目标文件或一个标签（clean）
    prerequisites 是生成target文件所需要的依赖文件
    recipe 是生成target文件所需要执行的命令，一个规则可以有多个命令行，每一条命令占一行  
    1. 目标名以点号“.”开始的并且其后不存在斜线“/”（“./”被认为是当前目录；“../”被认为是上一级目录）  
    2. 模式规则的目标。当这两种目标所在的规则是Makefile的第一个规则时，它们并不会被作为“终极目标”
    注意：每一个命令行必须以[Tab]字符开始，[Tab]字符告诉make此行是一个命令行。make按照命令完成相应的动作
  ```
# 2. make执行过程
  ```
    1. 依次读取变量“MAKEFILES”定义的makefile文件列表
    2. 读取工作目录下的makefile文件
    3. 依次读取工作目录makefile文件中使用指示符“include”包含的文件
    4. 查找重建所有已读取的makefile文件的规则（如果存在一个目标是当前读取的某一个makefile文件，则执行此规则重建此makefile文件，完成以后从第一步开始重新执行）
    5. 初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行分支
    6. 根据“终极目标”以及其他目标的依赖关系建立依赖关系链表
    7. 执行除“终极目标”以外的所有的目标的规则（规则中如果依赖文件中任一个文件的时间戳比目标文件新，则使用规则所定义的命令重建目标文件）
    8. 执行“终极目标”所在的规则
  ```
# 3. makefile里面包含什么
    - 显式规则、隐晦规则、变量定义、文件指示和注释
  ```
    1. 显式规则：显式规则说明了如何生成一个或多个目标文件。这是由Makefile 的书写者显示的指出要生成的文件、文件的依赖文件和生成的命令。
    2. 隐晦规则：根据make的自动推导功能能自动生成目标文件
    3. 变量的定义：Makefile支持定义变量，当Makefile 执行时，变量会被展开到相应的引用位置上
    4. 文件指示：一个是在一个Makefile 中引用另一个Makefile，include <filename>   
    另一个是指根据某些情况指定Makefile 中的有效部分，就像C 语言中的预编译#if一样；还有就是定义一个多行的命令，如define  
    1. 注释：Makefile 中只有行注释，用# 字符，注意，Makefile 中的命令，必须要以Tab 键开始
  ```
# 4. 伪目标
  ```
    .PHONY : clean
    clean :
      rm *.o temp
  ```   
    clean就是一个伪目标，需要显示的执行，make clean
    - 如果你的Makefile 需要一口气生成若干个可执行文件，并且，所有的目标文件都写在一个Makefile 中，那么你可以使用“伪目标”这个特性  
  ```
    all : prog1 prog2 prog3
    .PHONY : all
    prog1 : prog1.o utils.o
      cc -o prog1 prog1.o utils.o
    prog2 : prog2.o
      cc -o prog2 prog2.o
    prog3 : prog3.o sort.o utils.o
      cc -o prog3 prog3.o sort.o utils.o
  ```
    - “all”是一个伪目标，伪目标只是一个标签不会生成文件，所以不会有“all”文件产生  
    于是，其它三个目标的规则总是会被执行。也就达到了我们一口气生成多个目标文件的目的
    .PHONY : all 声明了“all”这个目标为“伪目标”，不写也可以由make的隐晦规则自动推导
# 5. 显示命令
    - 用@ 字符在命令行前，这个命令将不会被make 显示出来
  ```
    @echo 正在编译XXX 模块......
    当make 执行时，会输出“正在编译XXX 模块⋯⋯”字串
  ```
    - 可以在Makefile 的命令行前加一个减号- （在Tab 键之后），标记为不管命令出不出错都认为是成功的
# 6. 嵌套执行make
  ```
    subsystem:
        $(MAKE) -C subdir
    进入“subdir”目录，然后执行make 命令

    SUBDIRS = foo bar baz
    .PHONY: subdirs $(SUBDIRS)
    subdirs: $(SUBDIRS)
    $(SUBDIRS):
      $(MAKE) -C $@
    foo: baz
    上边的实现中有一个没有命令行的规则“foo: baz”，此规则用来限制子目录的make顺序。  
    它的作用是限制同步目录“foo”和“baz”的make过程（在处理“foo”目录之前，需要等待“baz”目录处理完成）  
    提醒大家：在书写一个并行执行make的Makefile时，目录的处理顺序是需要特别注意的
  ```
# 7. 变量
    - 变量在声明时需要给予初值，使用时在变量名前加上$ 符号，但最好用小括号() 或是大括号{} 把变量给包括起来。如果你要使用$ 字符，要用$$ 来表示
    - 变量的基本赋值
      - 1. = 直接赋值 
      - 2. := 前面的变量不能使用后面的变量，只能使用前面已定义好了的变量
      - 3. ?= 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效
      - 4. += 给变量追加值
    - 变量替换
      - foo := a.o b.o c.o
        bar := $(foo:.o=.c)  
        这个示例中，我们先定义了一个$(foo) 变量，而第二行的意思是把$(foo) 中所有以.o 字串“结尾”全部替换成.c ，所以我们的$(bar) 的值就是“a.c b.c c.c”。
      - foo := a.o b.o c.o
        bar := $(foo:%.o=%.c)  
        这依赖于被替换字串中的有相同的模式，模式中必须包含一个% 字符，这个例子同样让$(bar) 变量的值为“a.c b.c c.c”。
    - override 指示符
      - 如果变量是make 的命令行参数设置的，那么Makefile 中对这个变量的赋值会被忽略。如果想设置这类参数的值，可以使用“override”指示符
  ```
        override <variable>; = <value>;
        override <variable>; := <value>;
        override <variable>; += <more text>;
  ```
    - 多行变量
      - 使用define 关键字设置变量的值可以有换行，这有利于定义一系列的命令  
        define 指示符后面跟的是变量的名字，而重起一行定义变量的值，定义是以endef 关键字结束,命令需要以[Tab]键开头
  ```
        define two-lines
        echo foo
        echo $(bar)
        endef
  ```
    - 隐含规则使用的变量
  ```
        AR : 函数库打包程序。默认命令是ar
        AS : 汇编语言编译程序。默认命令是as
        CC : C 语言编译程序。默认命令是cc
        CXX : C++ 语言编译程序。默认命令是g++
        CPP : C 程序的预处理器（输出是标准输出设备）。默认命令是$(CC) –E
        ARFLAGS : 函数库打包程序AR 命令的参数。默认值是rv
        ASFLAGS : 汇编语言编译器参数。（当明显地调用.s 或.S 文件时）
        CFLAGS : C 语言编译器参数
        CXXFLAGS : C++ 语言编译器参数
        CPPFLAGS : C 预处理器参数。（C 和Fortran 编译器也会用到）
        LDFLAGS : 链接器参数。（如：ld ）
  ```
    - 自动化变量
  ```
        $@ : 表示规则中的目标文件集。在模式规则中，如果有多个目标，那么，$@ 就是匹配于目标中模式定义的集合  
        $% : 仅当目标是函数库文件中，表示规则中的目标成员名。例如，如果一个目标是foo.a(bar.o)，那么，$% 就是bar.o ，$@ 就是foo.a 。如果目标不是函数库文件其值为空  
        $< : 依赖目标中的第一个目标名字。如果依赖目标是以模式（即% ）定义的，那么$< 将是符合模式的一系列的文件集。注意，其是一个一个取出来的  
        $? : 所有比目标新的依赖目标的集合。以空格分隔  
        $^ : 所有的依赖目标的集合。以空格分隔。如果在依赖目标中有多个重复的，那个这个变量会去除重复的依赖目标，只保留一份  
        $+ : 这个变量很像$^ ，也是所有依赖目标的集合。只是它不去除重复的依赖目标  
        $* : 这个变量表示目标模式中% 及其之前的部分。如果目标是dir/a.foo.b ，并且目标的模式是a.%.b ，那么，$* 的值就是dir/a.foo  
        如果目标是foo.c ，因为.c 是make 所能识别的后缀名，所以，$* 的值就是foo  
        这个特性是GNU make 的，可能不兼容于其它版本的make，所以尽量避免使用$*  
        $? : 所有比目标新的依赖目标的集合。以空格分隔
  ```
# 8. 通配符
  - *：匹配0个或者是任意个字符
  ```
    .PHONY:clean
    clean:
      rm -rf *.o test
  ```
  - ?: 匹配任意一个字符
  - []: 我们可以指定匹配的字符放在 "[]" 中

# 9. 条件判断
    - ifeq (<arg1>, <arg2>)
    - ifneq (<arg1>, <arg2>)
    - ifdef <variable-name>
    - ifndef <variable-name>
  ```
    libs_for_gcc = -lgnu
    normal_libs =
    foo: $(objects)
    ifeq ($(CC),gcc)
        $(CC) -o foo $(objects) $(libs_for_gcc)
    else
        $(CC) -o foo $(objects) $(normal_libs)
    endif
  ```
    - ifeq 的意思表示条件语句的开始，并指定一个条件表达式，表达式包含两个参数，以逗号分隔，表达式以圆括号括起。  
    else 表示条件表达式为假的情况。endif 表示一个条件语句的结束，任何一个条件表达式都应该以endif 结束
  ```
    bar =
    foo = $(bar)
    ifdef foo
        frobozz = yes
    else
        frobozz = no
    endif

    foo =
    ifdef foo
        frobozz = yes
    else
        frobozz = no
    endif
    第一个例子中，$(frobozz) 值是yes ，第二个则是no
    ifdef只是测试一个变量是否有值，不会对变量进行替换展开来判断变量的值是否为空
  ```
# 10. 函数
    - subst
      - $(subst <from>,<to>,<text>)
      - 名称：字符串替换函数  
        功能：把字串<text> 中的<from> 字符串替换成<to>  
        返回：函数返回被替换过后的字符串  
        $(subst ee,EE,feet on the street)  
        把feet on the street 中的ee 替换成EE ，返回结果是fEEt on the strEEt
    - patsubst
      - $(patsubst <pattern>,<replacement>,<text>)  
        名称：模式字符串替换函数  
        功能：查找<text> 中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式<pattern> ，如果匹配的话，则以<replacement> 替换  
        这里，<pattern> 可以包括通配符% ，表示任意长度的字串。如果<replacement> 中也包含% ，  
        那么，<replacement> 中的这个% 将是<pattern> 中的那个% 所代表的字串。（可以用\ 来转义，以\% 来表示真实含义的% 字符）  
        返回：函数返回被替换过后的字符串  
        $(patsubst %.c,%.o,x.c.c bar.c)  
        把字串x.c.c bar.c 符合模式%.c 的单词替换成%.o ，返回结果是x.c.o bar.o
    - strip
      - $(strip <string>)  
        名称：去空格函数  
        功能：去掉<string> 字串中开头和结尾的空字符  
        返回：返回被去掉空格的字符串值  
        $(strip a b c )  
        把字串a b c `` 去到开头和结尾的空格，结果是``a b c
    - findstring
      - $(findstring <find>,<in>)
      - 名称：查找字符串函数  
        功能：在字串<in> 中查找<find> 字串  
        返回：如果找到，那么返回<find> ，否则返回空字符串  
        $(findstring a,a b c)  
        $(findstring a,b c)  
        第一个函数返回a 字符串，第二个返回空字符串
    - filter
      - $(filter <pattern...>,<text>)  
        名称：过滤函数  
        功能：以<pattern> 模式过滤<text> 字符串中的单词，保留符合模式<pattern> 的单词。可以有多个模式  
        返回：返回符合模式<pattern> 的字串  
        sources := foo.c bar.c baz.s ugh.h  
        foo: $(sources)  
        cc $(filter %.c %.s,$(sources)) -o foo  
        $(filter %.c %.s,$(sources)) 返回的值是foo.c bar.c baz.s
    - filter-out
      - $(filter-out <pattern...>,<text>)  
        名称：反过滤函数  
        功能：以<pattern> 模式过滤<text> 字符串中的单词，去除符合模式<pattern> 的单词。可以有多个模式  
        返回：返回不符合模式<pattern> 的字串  
        objects=main1.o foo.o main2.o bar.o  
        mains=main1.o main2.o  
        $(filter-out $(mains),$(objects)) 返回值是foo.o bar.o
    - sort  
      - $(sort <list>)  
        名称：排序函数  
        功能：给字符串<list> 中的单词排序（升序）  
        返回：返回排序后的字符串  
        示例：$(sort foo bar lose) 返回bar foo lose   
        备注：sort 函数会去掉<list> 中相同的单词
    - word  
      - $(word <n>,<text>)  
        名称：取单词函数  
        功能：取字符串<text> 中第<n> 个单词。（从一开始）  
        返回：返回字符串<text> 中第<n> 个单词。如果<n> 比<text> 中的单词数要大，那么返回空字符串  
        示例：$(word 2, foo bar baz) 返回值是bar
    - wordlist  
      - $(wordlist <ss>,<e>,<text>)  
        名称：取单词串函数
        功能：从字符串<text> 中取从<ss> 开始到<e> 的单词串。<ss> 和<e> 是一个数字  
        返回：返回字符串<text> 中从<ss> 到<e> 的单词字串。如果<ss> 比<text> 中的单词数要大，那么返回空字符串。如果<e> 大于<text> 的单词数，那么返回从<ss> 开始，到<text> 结束的单词串  
        示例：$(wordlist 2, 3, foo bar baz) 返回值是bar baz
    - words  
      - $(words <text>)     
        名程：单词个数统计函数      
        功能：统计<text> 中字符串中的单词个数       
        返回：返回<text> 中的单词数     
        示例：$(words, foo bar baz) 返回值是3       
        备注：如果我们要取<text> 中最后的一个单词，我们可以这样：$(word $(words <text>),<text>)
    - firstword
      - $(firstword <text>)  
        名称：首单词函数——firstword  
        功能：取字符串<text> 中的第一个单词  
        返回：返回字符串<text> 的第一个单词  
        示例：$(firstword foo bar) 返回值是foo  
        备注：这个函数可以用word 函数来实现：$(word 1,<text>)  
        make 使用VPATH 变量来指定“依赖文件”的搜索路径。于是，我们可以利用这个搜索路径来指定编译器对头文件的搜索路径参数CFLAGS ，如：  
        override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
        如果我们的$(VPATH) 值是src:../headers ，那么$(patsubst %,-I%,$(subst :, ,$(VPATH)))  
        将返回-Isrc -I../headers ，这正是cc 或gcc 搜索头文件路径的参数
    - dir  
      - $(dir <names...>)  
        名称：取目录函数——dir  
        功能：从文件名序列<names> 中取出目录部分。目录部分是指最后一个反斜杠（/ ）之前的部分,如果没有反斜杠，那么返回./   
        返回：返回文件名序列<names> 的目录部分  
        示例：$(dir src/foo.c hacks) 返回值是src/ ./
    - notdir
      - $(notdir <names...>)  
        名称：取文件函数——notdir  
        功能：从文件名序列<names> 中取出非目录部分。非目录部分是指最後一个反斜杠（/ ）之后的部分  
        返回：返回文件名序列<names> 的非目录部分  
        示例: $(notdir src/foo.c hacks) 返回值是foo.c hacks
    - suffix
      - $(suffix <names...>)  
        名称：取後缀函数——suffix  
        功能：从文件名序列<names> 中取出各个文件名的后缀  
        返回：返回文件名序列<names> 的后缀序列，如果文件没有后缀，则返回空字串  
        示例：$(suffix src/foo.c src-1.0/bar.c hacks) 返回值是.c .c
    - basename
      - $(basename <names...>)  
        名称：取前缀函数——basename      
        功能：从文件名序列<names> 中取出各个文件名的前缀部分        
        返回：返回文件名序列<names> 的前缀序列，如果文件没有前缀，则返回空字串     示例：$(basename src/foo.c src-1.0/bar.c hacks) 返回值是src/foo src-1.0/bar hacks
    - addsuffix
      - $(addsuffix <suffix>,<names...>)  
        名称：加后缀函数——addsuffix     
        功能：把后缀<suffix> 加到<names> 中的每个单词后面       
        返回：返回加过后缀的文件名序列      
        示例：$(addsuffix .c,foo bar) 返回值是foo.c bar.c
    - addprefix  
      - $(addprefix <prefix>,<names...>)  
        名称：加前缀函数——addprefix     
        功能：把前缀<prefix> 加到<names> 中的每个单词后面  
        返回：返回加过前缀的文件名序列  
        示例：$(addprefix src/,foo bar) 返回值是src/foo src/bar
    - join
      - $(join <list1>,<list2>)  
        名称：连接函数——join  
        功能：把<list2> 中的单词对应地加到<list1> 的单词后面。如果<list1> 的单词个数要比<list2> 的多，  
        那么，<list1> 中的多出来的单词将保持原样。如果<list2> 的单词个数要比<list1> 多，  
        那么，<list2> 多出来的单词将被复制到<list1> 中 返回：返回连接过后的字符串示例：$(join aaa bbb , 111 222 333) 返回值是aaa111 bbb222 333
    - wildcard
      - $(wildcard PATTERN)  
        名称：获取匹配模式文件名函数  
        功能：列出当前目录下所有符合模式的PATTERN格式的文件名  
        返回值：空格分隔的存在当前目录下的所有符合模式PATTERN的文件名  
        OBJ=$(wildcard *.c  *.h)  
        all:  
          @echo $(OBJ)  
        执行 make 命令，我们可以得到当前函数下所有的 ".c " 和  ".h"  结尾的文件
    - foreach 函数  
        foreach 函数和别的函数非常的不一样。因为这个函数是用来做循环用的，Makefile 中的foreach 函数几乎是仿照于Unix 标准Shell（/bin/sh）中的for 语句，或是C-Shell（/bin/csh）中的foreach 语句而构建的  
        它的语法是：  
        $(foreach <var>,<list>,<text>)  
        这个函数的意思是，把参数<list> 中的单词逐一取出放到参数<var> 所指定的变量中，然后再执行<text> 所包含的表达式  
        每一次<text> 会返回一个字符串，循环过程中，<text> 的所返回的每个  
        字符串会以空格分隔，最后当整个循环结束时，<text> 所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach 函数的返回值  
        所以，<var> 最好是一个变量名，<list> 可以是一个表达式，而<text> 中一般会使用<var> 这个参数来依次枚举<list> 中的单词。举个例子：  
        names := a b c d  
        files := $(foreach n,$(names),$(n).o)  
        上面的例子中，$(name) 中的单词会被挨个取出，并存到变量n 中，$(n).o 每次根据$(n) 计算出一个值，这些值以空格分隔，  
        最后作为foreach 函数的返回，所以，$(files) 的值是a.o b.o c.o d.o  
        注意，foreach 中的<var> 参数是一个临时的局部变量，foreach 函数执行完后，参数<var> 的变量将不在作用，其作用域只在foreach 函数当中  
    - if 函数  
        if 函数很像GNU 的make 所支持的条件语句——ifeq,if 函数的语法是：  
        $(if <condition>,<then-part>)或是  
        $(if <condition>,<then-part>,<else-part>)  
        可见，if 函数可以包含“else”部分，或是不含。即if 函数的参数可以是两个，也可以是三个  
        <condition> 参数是if 的表达式，如果其返回的为非空字符串，那么这个表达式就相当于返回真，于是，<then-part> 会被计算，否则<else-part> 会被计算  
        而if 函数的返回值是，如果<condition> 为真（非空字符串），那个<then-part> 会是整个函数的返回值，  
        如果<condition> 为假（空字符串），那么<else-part> 会是整个函数的返回值，此时如果<else-part> 没有被定义，  
        那么，整个函数返回空字串,所以，<then-part> 和<else-part> 只会有一个被计算  
    - origin 函数  
        origin 函数不像其它的函数，他并不操作变量的值，他只是告诉你你的这个变量是哪里来的？其语法是：  
        $(origin <variable>)  
        注意，<variable> 是变量的名字，不应该是引用。所以你最好不要在<variable> 中使用$ 字符  
        Origin函数会以其返回值来告诉你这个变量的“出生情况”，下面，是origin 函数的返回值:  
        undefined 如果<variable> 从来没有定义过，origin 函数返回这个值 undefined default 如果<variable> 是一个默认的定义，比如“CC”这个变量  
        environment 如果<variable> 是一个环境变量，并且当Makefile 被执行时，-e 参数没有被打开  
        file 如果<variable> 这个变量被定义在Makefile 中  
        command line 如果<variable> 这个变量是被命令行定义的  
        override 如果<variable> 是被override 指示符重新定义的  
        automatic 如果<variable> 是一个命令运行中的自动化变量  
        在我们的Makefile 中，我们可以这样写：  
        ifdef bletch  
            ifeq "$(origin bletch)" "environment"  
                bletch = barf, gag, etc.
            endif
        endif
    - shell 函数  
        shell 函数也不像其它的函数。顾名思义，它的参数应该就是操作系统Shell 的命令。它和反引号“‘”是相同的功能。这就是说，shell 函数把执行操作系统命令后的输出作为函数返回  
        于是，我们可以用操作系统命令以及字符串处理命令awk，sed 等等命令来生成一个变量,如：  
        contents := $(shell cat foo)  
        files := $(shell echo *.c)
# 11. make常用命令参数
  - -C DIR，--directory=DIR	在读取 Makefile 之前，进入到目录 DIR，然后执行 make。当存在多个 "-C" 选项的时候，make 的最终工作目录是第一个目录的相对路径
  - -d	make在执行的过程中打印出所有的调试信息，使用 "-d" 选项我们可以看到 make 构造依赖关系链、重建目标过程中的所有的信息
  - -f=FILE，--file=FILE，--makefile=FILE	指定文件 "FILE" 为 make 执行的 Makefile 文件
  - -i，--ignore-errors	执行过程中忽略规则命令执行的错误
  - -I DIR，--include-dir=DIR	指定包含 Makefile 文件的搜索目录，在Makefile中出现另一个 "include" 文件时，将在 "DIR" 目录下搜索。多个 "-i" 指定目录时，搜索目录按照指定的顺序进行
  - -k，--keep-going	执行命令错误时不终止 make 的执行，make 尽最大可能执行所有的命令，直至出现知名的错误才终止
  - -n，--just-print，--dry-run	只打印执行的命令，但是不执行命令
  - -s，--silent，--quiet	取消命令执行过程中的打印
  - -w，--print-directory	在 make 进入一个子目录读取 Makefile 之前打印工作目录，这个选项可以帮助我们调试 Makefile，跟踪定位错误。使用 "-C" 选项时默认打开这个选项
# 12. 综合实例
  - ubuntu14.04下新建一个目录make-example用于测试makefile
    ```
    zhoudq@ubuntu:~$ mkdir make-example
    zhoudq@ubuntu:~$ cd make-example/
    ```
  - 在make-example目录下创建所需要的目录，bin用于存放最终生成的目标文件，include目录存放头文件，main目录存放主程序文件，src目录存放源文件
    ```
    zhoudq@ubuntu:~/make-example$ mkdir bin include main src 
    zhoudq@ubuntu:~/make-example$ ls
    bin  include  main  src
    ```
  - 创建所需的文件，主目录下创建主Makefile，src目录下创建Makefile helloword.cc文件和hello world目录  
    hello目录下创建hello.cc Makefile文件  
    world目录下创建world.cc Makefile文件  
    main目录下创建main.cc Makefile文件  
    include目录下创建hello.h world.h helloworld.h文件
    ```
    zhoudq@ubuntu:~/make-example$ ls
    bin  include  main  Makefile  src
    zhoudq@ubuntu:~/make-example/src$ ls
    hello  helloworld.cc  Makefile  world
    zhoudq@ubuntu:~/make-example/src/hello$ ls
    hello.cc  Makefile
    zhoudq@ubuntu:~/make-example/src/world$ ls
    Makefile  world.cc
    zhoudq@ubuntu:~/make-example/main$ ls
    main.cc  Makefile
    zhoudq@ubuntu:~/make-example/include$ ls
    hello.h  helloworld.h  world.h
    ```
  - 源文件和头文件内容如下：
    ```
    hello.cc文件：
      #include <iostream>
    
      using namespace std;
    
      void hello(void)
      {
            cout << "Hello!" << endl;
      }
    world.cc文件：
      #include <iostream>
      
      using namespace std;
      
      void world(void)
      {
            cout << "World!" << endl;
      }
    helloworld.cc 文件：
      #include "hello.h"
      #include "world.h"
    
      void helloworld(void)
      {
              hello();
              world();
      }
    main.cc 文件：
      #include "helloworld.h"
    
      int main(int argc, char *argv[])
      {
              helloworld();
    
              return 0;
      }
    hello.h 文件：
      #ifndef _HELLO_H
      #define _HELLO_H

      void hello(void);
    
      #endif
    world.h 文件：
      #ifndef _WORLD_H
      #define _WORLD_H

      void world(void);
    
      #endif
    helloworld.h 文件：
      #ifndef _HELLOWORLD_H
      #define _HELLOWORLD_H
    
      void helloworld(void);
    
      #endif
    ```
  - Makefile文件内容如下：
    ```
    主Makefile内容：
      #指定根目录，子目录，头文件目录和目标文件目录变量
      export cur_dir := $(shell pwd)
      sub_dir := main src
      export inc_dir := $(cur_dir)/include
      bin_dir := $(cur_dir)/bin
      #定义中间生成的目标文件
      export target := built-in.o
      #定义编译器相关的变量
      export CROSS_COMPILE :=
      export CXX := $(CROSS_COMPILE)g++
      export AR := $(CROSS_COMPILE)ar
      export LD := $(CROSS_COMPILE)ld
      export CXXFLAGS += -I$(inc_dir) -g -Wall
      export LDFLAGS += 
      #定义最终生成文件
      bin_target := hello_world

      #定义生成规则
      .PHONY : all $(bin_target) $(sub_dir)
      all : $(bin_target)
      $(bin_target) : $(sub_dir)
        $(CXX) $(CXXFLAGS) $(LDFLAGS) $(^:=/$(target)) -o $@; \
        mv $(bin_target) $(bin_dir)

      $(sub_dir):
        $(MAKE) -C $@

      #定义清除规则
      .PHONY : clean
      clean :
        $(RM) $(bin_dir)/$(bin_target)
        for dir in $(sub_dir);do \
          $(MAKE) -C $$dir clean; \
        done
    main目录下Makefile内容：
      srcs := $(wildcard *.cc)
      objs := $(patsubst %.cc, %.o, $(srcs))

      $(target) : $(objs)
        $(LD) $(LDFLAGS) $^ -r -o $@

      %.o : %.cc
        $(CXX) $(CXXFLAGS) $< -c 
      %.d : %.cc
        $(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
        sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
        $(RM) $@.$$$$
      sinclude $(srcs:%.cc=%.d)
      .PHONY : clean
      clean :
        $(RM) $(objs) $(srcs:%.cc=%.d) $(target)
    src目录下Makefile内容：
      srcs := $(wildcard *.cc)
      objs := $(patsubst %.cc, %.o, $(srcs))
      sub_dir := hello world

      .PHONY : $(target) $(sub_dir)
      $(target) : $(sub_dir) $(objs)
        $(LD) $(LDFLAGS) $(sub_dir:=/$(target)) $(objs) -r -o $@

      $(sub_dir):
        $(MAKE) -C $@
      %.o : %.cc
        $(CXX) $(CXXFLAGS) $< -c
      %.d : %.cc
        $(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
        sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
        $(RM) $@.$$$$
      sinclude $(srcs:%.cc=%.d)
      .PHONY : clean
      clean :
        $(RM) $(target) $(objs) $(srcs:%.cc=%.d)
        for dir in $(sub_dir);do \
          $(MAKE) -C $$dir clean; \
        done
    src/hello目录下Makefile内容：
      srcs := $(wildcard *.cc)
      objs := $(patsubst %.cc, %.o, $(srcs))

      $(target) : $(objs)
        $(LD) $(LDFLAGS) $^ -r -o $@

      %.o : %.cc
        $(CXX) $(CXXFLAGS) $< -c 
      %.d : %.cc
        $(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
        sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
        $(RM) $@.$$$$
      sinclude $(srcs:%.cc=%.d)
      .PHONY : clean
      clean :
        $(RM) $(objs) $(srcs:%.cc=%.d) $(target)
    src/world目录下Makefile内容：
      srcs := $(wildcard *.cc)
      objs := $(patsubst %.cc, %.o, $(srcs))

      $(target) : $(objs)
        $(LD) $(LDFLAGS) $^ -r -o $@

      %.o : %.cc
        $(CXX) $(CXXFLAGS) $< -c 
      %.d : %.cc
        $(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
        sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
        $(RM) $@.$$$$
      sinclude $(srcs:%.cc=%.d)
      .PHONY : clean
      clean :
        $(RM) $(objs) $(srcs:%.cc=%.d) $(target)

    ```