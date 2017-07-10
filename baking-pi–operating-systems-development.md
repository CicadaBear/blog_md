# Baking Pi – Operating Systems Development  

From [Baking Pi – Operating Systems Development](https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/)

最近发现了个非常棒的资源，网上很多人推荐的，有中国的，有老外，剑桥大学的一个计算机基础课程，关于树莓派的，用树莓派开发自己的操作系统，非常棒，全程用汇编，本次相关文章会有十多篇，这个课程可能比较老了，但是不过时。  

### Warning 

This course has not yet been updated to work with the Raspberry Pi models B+ and A+. Some elements may not work, in particular the first few lessons about the LED. It has also not been updated for Raspberry Pi v2.  

警告，这个课程使用的树莓派数Model A,没有为Model B+和A+更新。 一些课程可能在其他型号上不能工作，特别是开始的几节跟LED有关的课程。 也没有为树莓派2代更新，注意。  

Welcome to Baking Pi: Operating Systems Development! Course by [Alex Chadwick](awc32@cam.ac.uk).   

欢迎来到Baking Pi:操作系统开发课程，by [Alex Chadwick](awc32@cam.ac.uk). 

You can now help contribute to this tutorial on GitHub.
This website is here to guide you through the process of developing very basic operating systems on the [Raspberry Pi](http://www.raspberrypi.org/)! This website is aimed at people aged 16 and upwards, although younger readers may still find some of it accessible, particularly with assistance. More lessons may be added to this course in time.   

你可以帮助贡献这篇教程在GitHub上。这个网站为引导你通过这个在树莓派上开发非常基础的操作系统的过程。这个网站面向16岁及以上的学习者，即使是更小的读者可能也可以发现这里的一部分可以读懂，特别是在有帮助的情况下。 更多的课会按时加入到这个课程里来。   

This course takes you through the basics of operating systems development in assembly code. I have tried not to assume any prior knowledge of operating systems development or assembly code. It may be helpful to have some programming experience, but the course should be accessible without. The [Raspberry Pi forums](http://www.raspberrypi.org/phpBB3/viewforum.php?f=72) are full of friendly people ready to help you out if you run into trouble. This course is divided into a series of 'lessons' designed to be taken in order as below. Each 'lesson' includes some theory, and also a practical exercise, complete with a full answer.   

这个课程带你通过操作系统开发的基础用汇编代码。 我已经尝试假设你没有任何先前的知识关于操作系统和汇编的。 如果你有编程经验可能会有帮助，但是课程是不要求的。 树莓派的论坛到处都是友好的准备去帮助别人解决困难的人。 这个课程被分成很多节课，被设计按以下的顺序来进行。 每一节课包含一些理论，并且还有一个实际的练习，包括完整的答案。     

Rather than leading the reader through the full details of creating an Operating System, these tutorials focus on achieving a few common tasks separately. Hopefully, by the end, the reader should know enough about Operating Systems that they could try to put together everything they've learned and make one. Although the lessons are generally focused on creating very specific things, there is plenty of room to play with what you learn. Perhaps, after reading the lesson on functions, you imagine a better style of assembly code. Perhaps after the lessons on graphics you imagine a 3D operating system. Since this is an Operating Systems course, you will have the power to design things how you like. If you have an idea, try it! Computer Science is still a young subject, and so there is plenty left to discover!   

这些教程不是带领着读者通过创建一个操作系统所有的细节，而是关注于分开实现几个通常的任务。 希望到最后，读者应该掌握做够的知识关于操作系统，可以尝试把学到的知识放在一起并且做出一个。 虽然这些课程一般关注于创建非常特殊的东西，这里还是有很大的空间来玩学到的东西。 或许，在读完函数的课程后，你想象一个更好的汇编代码风格。 或许，在读完图形相关的课程之后，你想象一个3D的操作系统。 既然这是一个操作系统的课程，你将拥有设计你喜欢的东西的能力。 如果你有一个想法，尝试一下。计算机科学仍然是一个非常年轻的学科，这里还有很多留着去探索的。  

### **Contents**  

* [1 Requirements](#Requirements)
 * [1.1 Hardware](#Hardware)
 * [1.2 Software](#Software)
* [2 Lessons](#Lessons)

## <p id="Requirements"> 1 Requirements </p>  

### **<p id="Hardware"> 1.1 Hardware </p>**  

In order to complete this course you will need a Raspberry Pi with an SD card and power supply. It is helpful, but not necessary, for your Raspberry Pi to be able to be connected to a screen and keyboard.  

为了完成这个课程，你将需要一个树莓派，SD卡，电源。 树莓派连接上键盘显示器会很有帮助，但是不是必须。  

In addition to the Raspberry Pi used to test and run your operating system code, you also need a seperate computer running Linux, Microsoft Windows or Mac OS X capable of writing to the type of SD card used by your Raspberry Pi. This other computer is your development and support system.  

还有，树莓派被用来测试和运行你的操作系统代码，你还需要一台电脑运行着Linux,Windows或者Mac,有读写SD卡能力的电脑。这台电脑是你的开发，和支持系统。  

### **<p id="Software"> 1.2 Software </p>**  

In terms of software, you require a GNU compiler toolchain that targets ARMv6 processors. You will install or build a set of tools, called a cross-compiler, on your development system. This cross-compiler converts your source code files into Raspberry Pi-compatible executable files which are placed on the SD card. The SD card is then transferred to the Raspberry Pi where the executable can be tested.   

在软件方面，你需要一个GNU编译器工具链，目标是ARMv6处理器的。你将会安装或者编译一系列的工具，叫做交叉编译器，在你的开发系统上。 这个交叉编译器把你的源码转化成树莓派兼容的可执行文件，把编译出的文件放在SD卡上。 这个SD卡被插在树莓派上测试。  

You can find instruction for getting the toolchain on the [Downloads Page](https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/downloads.html), along with model answers for all of the exercises.   

你可以在下载页上，找到获取工具链的说明，还有所有练习的标准答案。  

## <p id="Lessons">2 Lessons </p>

<table>					<caption>						Table 2.1 - Lessons</caption>
					<thead>						<tr>							<th>							</th>
							<th>								Name
							</th>
							<th>								Description
							</th>
						</tr>
					</thead>
					<tbody>						<tr>							<td>								0
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/introduction.html">Introduction</a> 介绍
							</td>
							<td>								This introductory lesson does not contain a practical element, but exists to explain
								the basic concepts of what is an operating system, what is assembly code, and other
								important basics. If you just want to get straight into practicals, it should be
								safe to skip this lesson.  
							这个介绍课程不包含任何实际的部分，但是它的存在是为解释一些基本概念，比如什么是操作系统，什么是汇编代码，还有其他一些重要的基础。 如果你想直接进入实际的，可以跳过这一节。
							</td>
						</tr>
						<tr>							<th colspan="3">								OK LED Series (Beginner) 板载LED(ok/act)系列（入门级）
							</th>
						</tr>
						<tr>							<td>								1
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/ok01.html">OK01</a>
							</td>
							<td>								The OK01 lesson contains an explanation about how to get started and teaches how
								to enable the 'OK' or 'ACT' LED on the Raspberry Pi board near the RCA and USB ports.
							OK01课程包含了怎样开始，怎样使'OK'or'ACT'LED亮起来。   
							</td>
						</tr>
						<tr>							<td>								2
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/ok02.html">OK02</a>
							</td>
							<td>								The OK02 lesson builds on OK01, by causing the 'OK' or 'ACT' LED to turn on and off repeatedly.  
							'OK02' 课程是在OK01的基础上建立的，通过是OK LED开关开关，循环往复。  
							</td>
						</tr>
						<tr>							<td>								3
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/ok03.html">OK03</a>
							</td>
							<td>								The OK03 lesson builds on OK02 by teaching how to use functions in assembly to make
								more reusable and rereadable code.   
							OK03课程在OK02的基础上建立的，教了，怎样在汇编语言中使用方法函数，使代码更可重用，更可读。  
							</td>
						</tr>
						<tr>							<td>								4
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/ok04.html">OK04</a>
							</td>
							<td>								The OK04 lesson builds on OK03 by teaching how to use the timer to flash the 'OK' or 'ACT'
								LED at precise intervals.   
							OK04课程建立在OK03的基础上，教了，怎样使用定时器来使LED闪烁，以一个精确的时间间隔。
							</td>
						</tr>
						<tr>							<td>								5
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/ok05.html">OK05</a>
							</td>
							<td>								The OK05 lesson builds on OK04 using it to flash the SOS morse code pattern (...---...).  
							OK05课程建立在OK04的基础上，使用它来闪烁SOS莫尔斯码。
							</td>
						</tr>
						<tr>							<th colspan="3">								Screen Series (Advanced) 屏幕系列（高级）
							</th>
						</tr>
						<tr>							<td>								6
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/screen01.html">Screen01</a>
							</td>
							<td>								The Screen01 lesson teaches some basic theory about graphics, and then applies it
								to display a gradient pattern to the screen or TV.   
							Screen01 课程教了一些基础的理论关于图形，并且应用它来显示有倾斜的图形在屏幕或者电视上。
							</td>
						</tr>
						<tr>							<td>								7
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/screen02.html">Screen02</a>
							</td>
							<td>								The Screen02 lesson builds on Screen01, by teaching how to draw lines and also
								a small feature on generating pseudo random numbers.  
							Screen02课程建立在Screen01的基础上，教了，怎样划线，还有一个小特色，生成假的随机数字。  
							</td>
						</tr>
						<tr>							<td>								8
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/screen03.html">Screen03</a>
							</td>
							<td>								The Screen03 lesson builds on Screen02 by teaching how to draw text to the screen,
								and introduces the concept of the kernel command line.  
							Screen03课程建立在Screen02的基础上，教了，怎样在屏幕上划字，并且介绍内核命令行概念。  
							</td>
						</tr>
						<tr>							<td>								9
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/screen04.html">Screen04</a>
							</td>
							<td>								The Screen04 lesson builds on Screen03 by teaching how to manipulate text to display
								computed values on the screen.  
							Screen04课程建立在Screen03的基础上，教了，怎样操作字符来把计算的数据显示在屏幕上。  
							</td>
						</tr>
						<tr>							<th colspan="3">								Input Series (Advanced) 输入系列（高级）
							</th>
						</tr>
						<tr>							<td>								10
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/input01.html">Input01</a>
							</td>
							<td>								The Input01 lesson teaches some theory about drivers, and linking programs, as well
								as keyboards. It is then applied to print out input characters to the screen. 
							Input01课程教了一些关于驱动的理论，还有连接程序，还有键盘。然后它被用来把输入的字符在屏幕上打印出来。
							</td>
						</tr>
						<tr>							<td>								11
							</td>
							<td>								<a href="https://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/os/input02.html">Input02</a>
							</td>
							<td>								The Input02 lesson builds on Input01 by teaching how to make a command line interface
								for an Operating System.  
							Input02课程建立在Input01的基础上，教了，怎样做一个操作系统的命令行界面。
							</td>
						</tr>
					</tbody>
				</table>