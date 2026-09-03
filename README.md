# EthornellTools
用于 Buriko General Interpreter (BGI)/Ethornell 视觉小说引擎的工具。

**BgiDisassembler**：反汇编 .\_bp 脚本文件。

**BgiImageEncoder**：将图片编码为引擎专有的“CompressedBG”格式。（要解码现有图片，可使用例如 [GARbro](https://github.com/morkt/GARbro/)。）

如果你打算翻译 Ethornell 游戏，请参阅 [VNTextPatch](https://github.com/arcusmaximus/VNTranslationTools)。

## 脚本修补
Ethornell 有两种脚本格式，每种都有各自的虚拟机和指令集：

* 一种用于内部系统脚本（设置用户界面、执行场景等）。这些脚本在每个游戏中基本相同。虚拟机在 BGI.exe 中用 C++ 实现，脚本扩展名为 .\_bp；BgiDisassembler 所针对的正是这些脚本。
* 另一种用于场景文件，包含游戏特有的叙述、对话、选项等内容。该虚拟机在多个 scr\*.\_bp 文件中实现，场景文件没有扩展名。其文本内容可以使用 VNTextPatch 提取和修补。

### 修复半角字符渲染问题
Ethornell 游戏在显示半角字符时往往会出问题（使字符重叠）。
有趣的是，引擎本身完全能够正确显示它们；你只需要告诉它这样做。

步骤如下：

* 反汇编所有 .\_bp 脚本。
* 打开 scrmsg.\_bp 的反汇编文件，找到 `graphcall 91:88` 指令（该指令调用“ConfigureFormatInfo”系统函数），并查看其倒数第二个参数（bProportional）来自哪里。然后找到写入该地址的脚本，并将值改为 1。
  
  例如：该 graphcall 指令的第二个参数是地址 198FC+12928+6 处的字节。在反汇编脚本文件夹中搜索文本“12928”，会显示 userdata.\_bp 写入了该地址。在十六进制编辑器中打开该 .\_bp 文件，转到反汇编中显示的偏移量，并将加载值的指令从 `push 0`（`00 00`）改为 `push 1`（`00 01`）。
* 在 bitmap.\_bp/scrmsg.\_bp（角色名）、logwnd.\_bp（日志）和 scrslct.\_bp（选项画面）中找到相关的 `graphcall 92:9C` 指令（调用“RenderText”系统函数），并将其第 9 个参数（从底部数起）改为 1。

### 更改字体大小
要更改字体大小（适用于具有 V1 场景文件的游戏，即文件头为“BurikoCompiledScriptVer1.00”的文件）：

* 可选步骤：反汇编包含场景虚拟机的脚本（scrmsg.\_bp），找到 `graphcall 91:88` 指令，并确认它是由场景操作码 14C（用于设置消息窗口的格式选项）调用的。
  要在反汇编中标记场景操作码处理程序，可以取消反汇编器 Program.cs 中 AssignOpcodeHandlerNames() 函数的注释，并将正则表达式中的 `push 14FE0` 改为操作码处理程序表的正确地址。
* 找到调用该场景操作码的场景文件（例如“function”），并更改第二个参数。例如：

```
  00 00 00 00 00 00 00 00    // push 0       <- 粗体
  00 00 00 00 1C 00 00 00    // push 1C      <- 字体大小
  00 00 00 00 00 00 00 00    // push 0       <- 字体族
  4C 01 00 00                // 设置消息窗口格式
```

  请注意，该操作码可能会出现多次，并带有不同的字体大小：一种用于普通行，一种用于低语，一种用于喊叫等。如果在游戏中更改某个字体大小没有效果，请尝试另一个。
* 在 logwnd.\_bp 中找到两条 `graphcall 92:9C` 指令，并将其第 12 个参数（从底部数起）改为新的字体大小。
