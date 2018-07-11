#vim 使用笔记
ctrl+^ 在两个buf间来回切换
vim中复制到ex模式的command line 

“.” 这个 mark 代表最后一次修改的地方，所以 `. 可以跳到最后一次修改的地方，'. 可以跳到最后一次修改的那一行。
g; 和 g, 则可以在整个 changelist 里面来回跳转，敲 :help changelist 可以看说明。

再来是zz，居中很方便。还有 ``, 可以在两个地方来回改, 也是左手位很方便的一个跳转操作.

##宏录制

vim-录制命令的使用
使用vim时无意间触碰到q键，左下角出现“recording”这个标识，这是vim的一个强大功能。
他可以录 制一个宏（Macro)，在开始记录后，会记录你所有的键盘输入，包括在insert模式下的输入、正常模式下使用的各种命令等。

具体使用：

第一步：在正常模式下（非insert模式、非visual模式）按下q键盘

第二步：选择a-z或0-9中任意一个作为缓冲器的名字，准备开始录制宏

第三步：正常的操作，此次所有的操作都会被记录在上一步中定义的缓冲器中

第四步：在非insert模式下输入q停止宏的录制

第五步：使用@ + 第二步中定义的缓冲器的名字即可。

例如想把下面的文字
line1
line-2
line3-1
l4

变成如下的文字
System.out.println(line1);
System.out.println(line1);
System.out.println(line-2);
System.out.println(line3-1);
System.out.println(L4);

观察可以发现他们的规律，在每行文字的开头添加“System.out.println(”，结尾添加“);”就变成下面的信息了。下面简单介绍一下如何使用recording来完成这样的操作。
首先把光标移动line1上，输入qt，准备开始录制，缓冲器的名字为t，
录制的动作为：shift + ^ 回到行首、按下i键进入insert模式、输入“System.out.println(”、按下esc键回到正常模式、shift + $ 回到行尾部、按下i键进入insert模式、输入“);”按下esc键回到正常模式，按下q停止录制。
然后把光标移动到下面一行的任意位置输入 @ + t 即可。

recording还可以和查询结合起来使用，例如想把一个文件中含有特定字符串的行注释，可以通过这样的宏来实现。
在正常模式下输入/search string + enter、shift + ^、i、#、esc、shift + $。

让定制的宏自动执行多次的方法是先输入一个数字，然后在输入@ + 缓冲器的名字。 例如 100@t，表示执行100次。


### Master Vim Registers with CTRL-R
<font color=red> 替换控制符号时必须用ctrl-r 了！！</font>
Vim’s registers are incredibly powerful. You use them all the time when you yank and put text or record macros, but are you using CTRL-R (in insert mode)? If you aren’t, you’re missing out on a huge efficiency boost! I will show you what CTRL-R does and how it can make you faster and give you even more uses for Vim’s registers.

From the Vim documentation:

Insert the contents of a register. Between typing CTRL-R and the second character, " will be displayed to indicate that you are expected to enter the name of a register.

In its simplest form, you can press CTRL-R followed by a register name (such as a letter, number, or symbol as seen in the output of :registers) and the contents of that register will be inserted at the cursor position, as though you had typed it. I use this all the time to insert text that I had just yanked by pressing CTRL-R ". Double quote is the name of the default register.
” and types out “ and my ax!” followed by pressing escape.

**qqA and my ax!^[q**

Note that ^[ is how Vim represents a literal escape. Great, so I have a macro recorded into register “q” and I can play it back by pressing @q as usual. But let’s say I want to make a change to this macro and I am far too lazy to re-record it. Instead of inserting “ and my ax!” at the end of the line, I want it to insert “My ax and ” at the beginning of the line.

We can use CTRL-R to get access to that macro in a buffer so that we can edit it. Just press:

**i^R^Rq**

Press “i” to enter insert mode, then press CTRL-R twice, then press “q” to insert the contents of the “q” register. What ought to come out looks like this:

**A and my ax!^[**

The reason you press CTRL-R twice is to insert that escape character code literally. If you only press CTRL-R once, the contents of the register will be inserted without any literal control characters, so you’ll lose the escape keypress, which is bad news bears.

Now that I have the contents of the macro sitting in a buffer, I can edit it normally as text. I will change it to look like this:

**IMy ax and ^[**

Now the macro presses “I” to start inserting at the beginning of the line and types out “My ax and ” then presses escape. Let’s capture that back into the “q” register. I place my cursor at the beginning of that line and press:

**"qy$**

I have just used the double quote key to tell Vim I want to use the “q” register for the next command and pressed y$ to yank to the end of the line. Now my revised macro is in the “q” register and can be played normally with @q. This little trick comes in very handy when you are working with longer macros and you make a mistake partway through or want to tweak the behavior a little bit without re-recording the whole sequence.

As a bonus, now that you know how to use CTRL-R twice in a row to output the contents of a register literally, including control characters, you can confidently save macros for later use by creating mappings. If I want to create a new mapping to run the same sequence that I recorded in this macro, I could type this:

**nnoremap <Leader>a ^R^Rq**

The nnoremap command creates a mapping for the key sequence <Leader>a to the literal contents of the “q” register without allowing nested mappings. It’s important to use nnoremap because in the future you may create a mapping for one of the key sequences used in this mapping and you don’t want that to run; you want this key sequence to be played back literally. After pressing “q” in the line above, it should look like this:

**nnoremap <Leader>a IMy ax and ^[**

Ta daa!

Expressions
In a previous post I talked about how to use CTRL-R and the expression register to easily edit complex Vim settings. The expression register is super powerful and can do a lot more than just output simple values. For example, it is possible to perform calculations and call any of the built-in functions found in :h function-list.

Things are about to get pretty advanced, so make sure you’re strapped in. This is a completely contrived example, but I am going to show you how to create a macro that uses CTRL-R to multiply the number under the cursor by two. Later on, you can think of other crazy ways to apply these concepts and leave them in the comments below for all to enjoy.

The basic procedure is this:

Begin recording a macro into register “q”
Change the word (the number) under the cursor. This places the deleted value in the default register, "", and puts you in insert mode
Use CTRL-R to insert an expression and use CTRL-R again to get the value from the default register, "", into the command.
The exact keystrokes to record the macro are these: qqciw^R=2*^R"^M^[q

Note in the macro string that ^R is CTRL-R, ^M is the return key, and ^[ is escape. Give it a try, record the macro, then type in some numbers in the buffer and replay the macro with the cursor on each number. Because we have used ciw you can place your cursor anywhere over the number and it will replace the whole number with the sum!

Now you have the power to truly master CTRL-R!

##非root用修改root权限文件
:w !sudo tee %
在编辑一些需要root权限才可以编辑的文件的时候
如果已经以非root的用户打开了 vim 而且这个用户是 sudoer
这条命令可以不用退出 vim 切换到 root 就完成对文件的写操作

#替换^@ 用
:%s/\%x00/ /g
