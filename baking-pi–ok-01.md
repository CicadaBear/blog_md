# Lesson 1 OK01  

The OK01 lesson contains an explanation about how to get started and teaches how to enable the 'OK' or 'ACT' LED on the Raspberry Pi board near the RCA and USB ports. This light was originally labelled OK but has been renamed to ACT on the revision 2 Raspberry Pi boards.  

OK01课程包含了解释怎样开始并且教了怎样使板载OK LED亮起来。 这个LED灯最开始被标注为OK,之后的树莓派版本换了这个LED的名字为ACT。  

## Contents  

* [1 Getting Started](#Getting-Started)
* [2 The Beginning](#The-Beginning)
* [3 The First Line](#The-First-Line)
* [4 Enabling Output](#Enabling-Output)
* [5 A Sign Of Life](#A-Sign-Of-Life)
* [6 Happily Ever After](#Happily-Ever-After)
* [7 Pi Time](#Pi-Time)

## <p id="Getting-Started"> 1 Getting Started </p>  

I am assuming at this point that you have already visited the Downloads page, and got the necessary GNU Toolchain. Also on the downloads page is a file called OS Template. Please download this and extract its contents to a new directory.  

我假设现在你已经看过了下载页，并且获取了必须的GNU工具链。 还有，下载页还有一个文件叫“OS Template”。 把它下载下来，并且解压到一个新的文件夹。  

## <p id="The-Beginning"> 2 The Beginning </p>  

Now that you have extracted the template, create a new file in the 'source' directory called 'main.s'. This file will contain the code for this operating system. To be explicit, the folder structure should look like:

现在你已经解压了模板文件，在source文件夹创建一个新的文件叫‘main.s’。 这个文件将会包含操作系统的代码。 为了更明了，这个文件夹的结构应该像这样：  

```
build/
   (empty)  
source/
   main.s
kernel.ld
LICENSE
Makefile
```
#### 提示  

The '.s' file extension is commonly used for all forms of assembly code, it is up to us to remember this is ARMv6.  

后缀名为.s的文件通常情况下被用来作为任何形式的汇编代码的源文件，对于我们来说要记住这是ARMv6。  

Open 'main.s' in a text editor so that we can begin typing assembly code. The Raspberry Pi uses a variety of assembly code called ARMv6, so that is what we'll need to write in.  

用文件编辑器打开main.s, 我们就可以开始敲汇编代码了。 树莓派使用一些汇编代码称为ARMv6, 这些代码就是我们需要写的。  

Copy in these first commands.

把这些指令复制进去。  

```
.section .init
.globl _start
_start:
```  

As it happens, none of these actually do anything on the Raspberry Pi, these are all instructions to the assembler. The assembler is the program that will translate between assembly code that we understand, and binary machine code that the Raspberry Pi understands. In Assembly Code, each line is a new command. The first line here tells the Assembler[1] where to put our code. The template I provided causes the code in the section called .init to be put at the start of the output. This is important, as we want to make sure we can control which code runs first. If we don't do this, the code in the alphabetically first file name will run first! The .section command simply tells the assembler which section to put the code in, from this point until the next .section or the end of the file.  

复制这几行代码时，这几行代码实际上并没有在树莓派上做任何事，这几行指令是对于汇编器的。 汇编器是一种把我们能看懂的汇编代码转化为树莓派可以识别的二进制机器码的程序。 在汇编代码中，每一行是一个新的指令。 这里的第一行是告诉汇编器把代码放在哪。 提供的模板使得.init section的代码会被放在输出的开始位置。 这很重要，当我们想要确认我们可以控制哪段代码首先运行。 如果我们不这么做（设置section .init），代码将会按照字母表的顺序，运行第一个代码文件。 这里的.section 指令简单的告诉了汇编器把这段代码放在哪个section，从这一点到下一个.section或者文件结尾。  

The next two lines are there to stop a warning message and aren't all that important.[2]  

这里下边的两行是为了不显示警告信息，没有第一行重要。  

## <p id="The-First-Line"> 3 The First Line </p>  

Now we're actually going to code something. In assembly code, the computer simply goes through the code, doing each instruction in order, unless told otherwise. Each instruction starts on a new line.  

现在，我们真要编两行代码了。 在汇编代码中，计算机简单地运行代码，按顺序运行每一行，除非被告知。 每一个指令开始于新的一行。  

Copy the following instruction.  

把下边的指令复制一下。  

```
ldr r0,=0x20200000
```
That is our first command. It tells the processor to store the number 0x20200000 into the register r0. I shall need to answer two questions here, what is a register, and how is 0x20200000 a number?   

这是我们的第一行指令。 它告诉处理器来把数值 0x20200000 保存到 寄存器 r0中。 我应该需要在这回答两个问题，什么是寄存器，还有，0x20200000为什么是一个数字？  

A register is a tiny piece of memory in the processor, which is where the processor stores the numbers it is working on right now. There are quite a few of these, many of which have a special meaning, which we will come to later. Importantly there are 13 (named r0,r1,r2,...,r9,r10,r11,r12) which are called General Purpose, and you can use them for whatever calculations you need to do. Since it's the first, I've used r0 in this example, but I could very well have used any of the others. As long as you're consistent, it doesn't matter.  

寄存器是处理器里边一个很小的一片内存，处理器把数字存在寄存器里边，现在它是工作的。 这里还有好几个寄存器，其中几个寄存器有特殊的意义，我们稍后再讲。 重要的是，这里有13个一般用途寄存器，你可以用它来做任何你需要的计算。 既然这是第一个，我们使用r0在这个例子中，但是我们还可以用其他寄存器。 As long as you're consistent, it doesn't matter.  

0x20200000 is indeed a number. However it is written in Hexadecimal notation. To learn more about hexadecimal expand the box below:  

0x20200000确实是一个数，但是是以16进制写成的。 详细了解16进制看原文。  

So our first command is to put the number 2020000016 into r0. That doesn't sound like it would be much use, but it is. In computers, there are an awful lot of chunks of memory and devices. In order to access them all, we give each one an address. Much like a postal address or a web address this is just a means of identifying the location of the device or chunks of memory we want. Addresses in computers are just numbers, and so the number 2020000016 happens to be the address of the GPIO controller. This is just a design decision taken by the manufacturers, they could have used any other address (providing it didn't conflict with anything else). I know this address only because I looked it up in a manual[3], there is no particular system to the addresses (other than that they are all large round numbers in hexadecimal).  

所以，我们第一条指令是把数字 2020000016 放入到r0寄存器。 这听起来好像不太常用，但是这非常常用。 在计算机中，这里有特别多的内存块，和设备。 为了访问它们，我们给每一个一个地址。 很像邮编或者网址，这里的地址只是为了标识我们想要使用的设备或者内存的位置。 地址在计算机中只是数字，并且这个数字2020000016恰好是GPIO控制器的地址。 这仅仅是制造商的决定，他们可以用任何地址，只要跟别的地址不冲突就可以。 我知道这个地址，因为我查了芯片手册，关于地址没有什么特别的系统，除了它们是一大推16进制的数字。  


## <p id="Enabling-Output"> 4 Enabling Output </p>  

Having read the manual, I know we're going to need to send two messages to the GPIO controller. We need to talk its language, but if we do, it will obligingly do what we want and turn on the OK LED. Fortunately, it is such a simple chip, that it only needs a few numbers in order to understand what to do.   

读过手册之后，我知道我们需要给GPIO controller发两个信息。 我们需要用它的语言，如果我们做了，它将会做我们想要的，点亮OK LED。 幸运的是，它是个非常简单的芯片，它只需要几个数字来理解做什么。  

```
mov r1,#1
lsl r1,#18
str r1,[r0,#4]
```

These commands enable output to the 16th GPIO pin. First we get a necessary value in r1, then send it to the GPIO controller. Since the first two instructions are just trying to get a value into r1, we could use another ldr command as before, but it will be useful to us later to be able to set any given GPIO pin, so it is better to deduce the value from a formula than write it straight in. The OK LED is wired to the 16th GPIO pin, and so we need to send a command to enable the 16th pin.   

这几个指令是打开第16个GPIO引脚的输出模式。 首先我们在r1寄存器上存储一个必须的值，然后把它发给GPIO控制器。 既然开始的两条指令仅仅是把一个值放入到r1，我们可以使用另外一条指令ldr，像之前一样，但是我们用的这两条指令将会非常有用在之后的代码中，设置任何给定的GPIO引脚，所以现在用一个公式推导出值比直接写入更好。 OK LED和GPIO 16相连，所以我们需要发送一个指令开始16号引脚。  

The value in r1 is needed to enable the LED pin. The first line puts the number 110 into r1. The mov command is faster than the ldr command, because it does not involve a memory interaction, whereas ldr loads the value we want to put into the register from memory. However, mov can only be used to load certain values[4]. In ARM assembly code, almost every instruction begins with a three letter code. This is called the mnemonic, and is supposed to hint at what the operation does. mov is short for move and ldr is short for load register. mov moves the second argument #1 into the first r1. In general, # must be used to denote numbers, but we have already seen a counterexample to this.   

r1存储的值是开启LED引脚所需要的。 第一行是把十进制的1存入r1。 mov指令比ldr更快，因为它不涉及内存交互，反之，ldr要从内存加载一个我们想要放入寄存器的值。 然而，mov指令只能被用来加载一些特定的数，（一个范围的数），最后的注释中会有一个比较详细的说明。 在ARM汇编代码中，几乎每一条指令都是以三个字母开始。这叫做助记的，并且提示了什么操作在运行。 mov是move的缩写，ldr是load register的缩写。 mov移动第二个参数#1到第一个r1寄存器。 一般情况下，#必须被用来表示数字，但是我们已经看到了一个反例了（没注意）。

The second instruction is lsl or logical shift left. This means shift the binary representation for the first argument left by the second argument. In this case this will shift the binary representation of 1[10] (which is 1[2]) left by 18 places (making it 1000000000000000000[2]=262144[10]).  

第二个指令是lsl或者叫logical shift left逻辑左移。 这里的意思是，将第一个参数左移第二参数的位数。 在这个例子中，将10进制1的二进制表达1，左移18位是，1000000000000000000[2] 262144[10]。  

If you are unfamiliar with binary, expand the box below:

查看二进制的详细解释，看原文。  

Once again, I only know that we need this value from reading the manual[3]. The manual says that there is a set of 24 bytes in the GPIO controller, which determine the settings of the GPIO pin. The first 4 relate to the first 10 GPIO pins, the second 4 relate to the next 10 and so on. There are 54 GPIO pins, so we need 6 sets of 4 bytes, which is 24 bytes in total. Within each 4 byte section, every 3 bits relates to a particular GPIO pin. Since we want the 16th GPIO pin, we need the second set of 4 bytes because we're dealing with pins 10-19, and we need the 6th set of 3 bits, which is where the number 18 (6×3) comes from in the code above.  

再说一次，我是从芯片手册上查到我们所需要的额值的，芯片手册在最下边的附录中。 手册上说，GPIO控制器中有24个字节，这些字节决定了GPIO的引脚。 前4个字节跟前10个GPIO引脚有关，第二个4个字节跟下一个10个GPIO引脚有关，依次下去。 这里有54个GPIO引脚，所以我们需要6个 4字节，所以总共是24个字节。 在每4个字节中，每3个位（比特）关系一个GPIO引脚。 既然我们想要第16个引脚，我们需要第二个4字节段，因为我们正在处理10-19，每3位关系到一个引脚，（6*3）18，这就是上边左移18位的来源。  

Finally the str 'store register' command stores the value in the first argument, r1 into the address computed from the expression afterwards. The expression can be a register, in this case r0, which we know to be the GPIO controller address, and another value to add to it, in this case #4. This means we add 4 to the GPIO controller address and write the value in r1 to that location. This happens to be the location of the second set of 4 bytes that I mentioned before, and so we send our first message to the GPIO controller, telling it to ready the 16th GPIO pin for output.  

最后，这个str 'store register' 指令，把第一个参数 r1寄存器的值存储到第二个参数也就是表达式计算出的结果地址中。 这个表达式可以是个寄存器，在这个例子中 r0，我们知道里边存的是GPIO控制器的地址，并且另外一个值加上了它，这个例子中是数字4。 这里的意思是，我们在GPIO控制器的地址上加了4，并且把r1存的值放到那个地址。 这里恰好是第二个4字节之前提到过的，我们把第一条信息发送给了GPIO控制器，告诉它第16号引脚开启输出模式。  

## <p id="A-Sign-Of-Life"> 5 A Sign Of Life </p>  

Now that the LED is ready to turn on, we need to actually turn it on. This means sending a message to the GPIO controller to turn pin 16 off. Yes, turn it off. The chip manufacturers decided it made more sense[5] to have the LED turn on when the GPIO pin is off. Hardware engineers often seem to take these sorts of decisions, seemingly just to keep OS Developers on their toes. Consider yourself warned.   

现在LED准备开启了，我们开启它。 这里需要给GPIO控制器发送一条信息来把pin 16关闭（置为低电压状态）。是的，关闭它。 芯片制造商决定了当这个GPIO引脚为关闭的时候，LED会被点亮。 硬件工程师似乎经常做这些决定，好像是为了系统开发者保持警惕。 注意一下。  

```
mov r1,#1
lsl r1,#16
str r1,[r0,#40]
``` 
Hopefully you should recognise all of the above commands, if not their values. The first puts a 1 into r1 as before. The second shifts the binary representation of this 1 left by 16 places. Since we want to turn pin 16 off, we need to have a 1 in the 16th bit of this next message (other values would work for other pins). Finally we write it out to the address which is 40[10] added to the GPIO controller address, which happens to be the address to write to turn a pin off (28 would turn the pin on).  

希望是，你应该认识所有这些指令，如果不知道它们的确切含义。 第一条指令把1放入r1寄存器，像之前一样。 第二条指令，将1的二进制表达逻辑左移16位。 既然我们想要把pin 16关闭，我们需要一个第16位为1的二进制信息，其他的值对其他引脚pin起作用。 最后我们将这个值写入到一个地址，40[10]（十进制 40）加上GPIO控制器的地址，这个地址恰好是关掉一个引脚的地址（28是打开这个引脚的地址）。  


## <p id="Happily-Ever-After"> 6 Happily Ever After </p>   

