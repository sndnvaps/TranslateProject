[#]: collector: (lujun9972)
[#]: translator: (cycoe)
[#]: reviewer: ( )
[#]: publisher: ( )
[#]: url: ( )
[#]: subject: (Debugging in Emacs: The Grand Unified Debugger)
[#]: via: (https://opensourceforu.com/2019/09/debugging-in-emacs-the-grand-unified-debugger/)
[#]: author: (Vineeth Kartha https://opensourceforu.com/author/vineeth-kartha/)

Emacs 调试秘籍之——GUD 调试器
======

[![][1]][2]

_本文简短地对 Emacs 的调试工具 GUD（译注：全称 Grand Unified Debugger，鉴于其缩写形式更为人熟知，以下全文将使用缩写替代此全称）的特性进行了探索。_

如果你是一个 C 或 C++ 开发者，你很可能已经使用过 GDB（GNU 调试器），毫无疑问，它是现今最强大、最无可匹敌的调试器之一。它唯一的缺点就是它基于命令行，虽然仍能提供许多强大的功能，但有时也会具有一些局限性。这也就是为什么聪明的人们开始追求整合了编辑器和调试器的<ruby>图形化<rt>GUI</rt>集成<rt>Integrated</rt>开发<rt>Development</rt>环境<rt>Environment</rt></ruby>。仍有些开发者坚信使用鼠标会降低工作效率，在 GUI 上用鼠标点～点～点～是来自恶魔的诱惑。

因为 Emacs 是现今最酷的文本编辑器之一，我将为你展示如何在不碰鼠标且不离开 Emacs 的情况下，实现写代码、编译代码、调试代码的过程。

![图 1: Emacs 迷你缓冲区中的编译命令][3]

![图 2: 编译状态][4]

GUD 是 Emacs 下的一个<ruby>模式<rt>mode</rt></ruby>，用于在 Emacs 中运行 GDB。它向 GDB 提供了 Emacs 的所有特性，使用户无需离开编辑器就可以对代码进行调试。

**使用 GUD 的前期准备**
如果你正在使用一个 Linux 机器，很可能你已经安装了 GDB 和 gcc，接下来就是要确保已经安装了 Emacs。以下的内容我将假设读者熟悉 GDB 并且至少用它做过基本的调试。如果你未曾接触过 GDB，你可以做个快速入门，这些资料在网上随处可得。

对于那些 Emacs 新手，我将向你介绍一些基本术语。纵览整篇文章，你将看到诸如 _C-c M-x_ 等快捷键。此处 _C_ 代表 _Ctrl_ 键，_M_ 代表 _Alt_ 键。_C-c_ 代表 _Ctrl_ 键和 _c_ 键被同时按下。如果你看到 _C-c c_，它代表同时按下 _Ctrl_ 键和 _c_ 键，紧接着按下 _c_ 键。在 Emacs 中，编辑文本的主要区域被称为<ruby>主<rt>main</rt>缓冲区<rt>buffer</rt></ruby>，而在 Emacs 窗口下方用于输入命令的区域被称为<ruby>迷你<rt>mini</rt>缓冲区<rt>buffer</rt></ruby>。

启动 Emacs，并按下 _C-x C-f_ 来创建一个新文件。Emacs 将提示你输入一个文件名，此处让我们将文件命名为 `buggyFactorial.cpp`。一旦文件打开，输入如下代码：

```
#include<iostream>
#include <assert.h>

int factorial(int num) {
  int product = 1;
  while(num--) {
    product *= num;
  }
  return product;
}
int main() {
  int result = factorial(5);
  assert(result == 120);
}
```

使用 _C-x C-s_ 快捷键保存文件。文件保存完毕，是时候进行编译了。按下 _M-x_，在弹出的<ruby>提示符<rt>prompt</rt></ruby>后输入 _compile_ 并点击回车。然后在提示符后，将内容替换为 _g++ -g buggyFactorial.cpp_ 并再次点击回车。

这将在 Emacs 中开启另一个缓冲区，显示编译的状态。如果你的代码输入没有错误，你将预期得到如图 2 所示的缓冲区。

要想隐藏编译缓冲区，首先确保你的光标在编译缓冲区中（你可以不使用鼠标，而是通过 _C-x o_ 快捷键将光标从一个缓冲区移动到另一个），然后按下 _C-x 0_。下一步就是运行代码，并观察是否运行良好。按下 _M-!_ 快捷键并在迷你缓冲区的提示符后面输入 _./a.out_。

你可以看到迷你缓冲区中显示断言失败。很明显代码中有错误，因为 5 的阶乘是 120。那么让我们现在开始调试吧。

![图 3: 代码在迷你缓冲区中的输出][5]

![图 4: Emacs 中的 GDB 缓冲区][6]

**使用 GUD 调式代码**
现在，我们的代码已经编译完成，是时候看看到底哪里出错了。按下 _M-x_ 快捷键并在提示符后输入 _gdb_。在接下来的提示符后，输入 _gdb -i=mi a.out_。如果一切顺利，GDB 会在 Emacs 缓冲区中启动，你会看到如图 4 所示的窗口。
在 _gdb_ 提示符后，输入 _break main_ 来设置断点，并输入 _r_ 来运行程序。程序会开始运行并停在 _main()_ 函数处。

一旦 GDB 到达了 _main_ 处设置的断点，就会弹出一个新的缓冲区显示你正在调试的代码。注意左侧的红点，正是你设置断点的位置，同时会有一个小的标志提示你当前代码运行到了哪一行。当前，该标志就在断点处（如图 5）。

![图 5: GDB 与代码显示在两个分离的窗口][7]

![图 6: 在 Emacs 中使用独立帧显示局部变量][8]

为了调试 _factorial_ 函数，我们需要单步运行。想要达到此目的，你可以使用 _gdb_ 提示符和 GDB 命令 _step_，或者使用 Emacs 快捷键 _C-c C-s_。还有其它一些快捷键，但我更喜欢 GDB 命令。因此我将在本文的后续部分使用它们。

单步运行时让我们注意一下局部变量中的阶乘值。参考图 6 来设置在 Emacs 帧中显示局部变量值。

在 GDB 提示符中进行单步运行并观察局部变量值的变化。在循环的第一次迭代中，我们发现了一个问题。此处乘法的结果应该是 5 而不是 4。

本文到这里也差不多结束了，读者可以自行探索发现 GUD 模式这片新大陆。GDB 中的所有命令都可以在 GUD 模式中运行。我将此代码的修复留给读者作为一个练习。看看你在调试的过程中，可以做哪一些定制化，来使你的工作流更加简单和高效。

--------------------------------------------------------------------------------

via: https://opensourceforu.com/2019/09/debugging-in-emacs-the-grand-unified-debugger/

作者：[Vineeth Kartha][a]
选题：[lujun9972][b]
译者：[cycoe](https://github.com/cycoe)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]: https://opensourceforu.com/author/vineeth-kartha/
[b]: https://github.com/lujun9972
[1]: https://i0.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Screenshot-from-2019-09-25-15-39-46.png?resize=696%2C440&ssl=1 (Screenshot from 2019-09-25 15-39-46)
[2]: https://i0.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Screenshot-from-2019-09-25-15-39-46.png?fit=800%2C506&ssl=1
[3]: https://i1.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_1.png?resize=350%2C228&ssl=1
[4]: https://i2.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_2.png?resize=350%2C228&ssl=1
[5]: https://i0.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_3.png?resize=350%2C228&ssl=1
[6]: https://i0.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_4.png?resize=350%2C227&ssl=1
[7]: https://i1.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_5.png?resize=350%2C200&ssl=1
[8]: https://i1.wp.com/opensourceforu.com/wp-content/uploads/2019/09/Figure_6.png?resize=350%2C286&ssl=1
