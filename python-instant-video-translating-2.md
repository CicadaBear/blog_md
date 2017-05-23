目前掌握的资料，技术还谈不上，FFmpeg是一个功能强大的多媒体命令行工具，是命令行工具，FFmpeg底层用的是库是Libav。我期望的功能中FFmpeg已经具备了，输入输出流可以很多种，不一定是文件，pipe，tcp，udp等。但是FFmpeg的字幕输入不知道能不能动态的，通过流获取，而不是一下读完一个字幕文件。还没找到文档资源证明可行，如果不可行，就要通过libav,或者openvc来动态添加字幕在把流输出出去，还有获取微软云的翻译需要时间，how，缓冲实现，10s，20s，目前看到的 ffmpeg -i (input) 字幕文件也是通过-i输入进来，FFmpeg输入输出协议没说是视频输入还是字幕输入。FFmpeg先到这，python的FFmpeg的库都是操作命令行工具的。还发现了一个库，Python livestream也是一个命令行工具，但也开放了操作流的基本接口，可以用来在python代码中从网络获取各种视频流。

接下来，我会详细阅读FFmpeg和livestream的相关文档，并基本翻译出来。

# FFmpeg文档
From [ffmpeg.org](https://ffmpeg.org/ffmpeg.html)

## 1 Synopsis 大纲
```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```
## 2 Description 描述    

**ffmpeg** is a very fast video and audio converter that can also grab from a live audio/video source. It can also convert between arbitrary sample rates and resize video on the fly with a high quality polyphase filter.   

**ffmpeg** 是一个非常快的视频和音频的转换器，它还可以用来从直播的视频音频源中获取资源。它还可以转换任意的取样率并且调整视频大小on the fly 用一个高质量的多项过滤器。   

**ffmpeg** reads from an arbitrary number of input "files" (which can be regular files, pipes, network streams, grabbing devices, etc.), specified by the -i option, and writes to an arbitrary number of output "files", which are specified by a plain output url. Anything found on the command line which cannot be interpreted as an option is considered to be an output url.  

**ffmpeg** 可以从任意数量的的输入“文件”中读，（文件不一定是指文件，还可以是pipes,network stream,摄像头设备 等等）， 通过 -i 选项指定输入源，并且可以写入到任意数量的输出“文件”，通过指定一个输出url。 任何命令行中的东西不能被解释的选项都将被考虑为输出url。  

Each input or output url can, in principle, contain any number of streams of different types (video/audio/subtitle/attachment/data). The allowed number and/or types of streams may be limited by the container format. Selecting which streams from which inputs will go into which output is either done automatically or with the -map option (see the Stream selection chapter).  

任何的输入输出url可以包含任意数量的不同种类的流，（视频/音频/字幕/附件/数据）在原则上是这样的。这允许流的数量，类型可能会被container format容器格式所限制。选择哪个流从哪进从哪出也是可以自动完成的，或者通过 -map 选项（看 流选择 的章节）。  

To refer to input files in options, you must use their indices (0-based). E.g. the first input file is 0, the second is 1, etc. Similarly, streams within a file are referred to by their indices. E.g. 2:3 refers to the fourth stream in the third input file. Also see the Stream specifiers chapter.   

为了在选项中用上多个输出文件，你必须使用它们的索引（从0开始）。比如，第一个输入的文件用0，第二个用1。相似的，一个文件里的多个流可以被使用通过它们的索引。比如，2:3 提及的是第三个输入文件中的第四个流。还可以看流 说明符（区分符）的章节。  

As a general rule, options are applied to the next specified file. Therefore, order is important, and you can have the same option on the command line multiple times. Each occurrence is then applied to the next input or output file. Exceptions from this rule are the global options (e.g. verbosity level), which should be specified first.  

一个通用的规则，选项是被用在下一个指定的文件上。因此，顺序非常重要，并且你可以使用相同的选项很多次在一条命令中。每次出现是被应用到下一个输入或输出文件上。这个规则的例外情况是全局选项。（比如，日志的啰嗦级别），应该被首先指定。

Do not mix input and output files – first specify all input files, then all output files. Also do not mix options which belong to different files. All options apply ONLY to the next input or output file and are reset between files.  

不要把输入输出文件混在一起。首先，指定所有的输入文件，然后是所有的输出文件。也不要把属于不同文件的选项混在一起。所有的选项只应用到下一个输入输出文件，并且选择放在文件之间。  

* To set the video bitrate of the output file to 64 kbit/s: 
```
ffmpeg -i input.avi -b:v 64k -bufsize 64k output.avi
```

* To force the frame rate of the output file to 24 fps:  
```
ffmpeg -i input.avi -r 24 output.avi
```

* To force the frame rate of the input file (valid for raw formats only) to 1 fps and the frame rate of the output file to 24 fps:  
```
ffmpeg -r 1 -i input.m2v -r 24 output.avi
```

## Detailed description 详细描述

The transcoding process in ffmpeg for each output can be described by the following diagram:  

ffmpeg对每一个输出的转码处理可以被描述为以下图表：  

```
 _______              ______________
|       |            |              |
| input |  demuxer   | encoded data |   decoder
| file  | ---------> | packets      | -----+
|_______|            |______________|      |
                                           v
                                       _________
                                      |         |
                                      | decoded |
                                      | frames  |
                                      |_________|
 ________             ______________       |
|        |           |              |      |
| output | <-------- | encoded data | <----+
| file   |   muxer   | packets      |   encoder
|________|           |______________|

```
**ffmpeg** calls the libavformat library (containing demuxers) to read input files and get packets containing encoded data from them. When there are multiple input files, ffmpeg tries to keep them synchronized by tracking lowest timestamp on any active input stream.  

**ffmpeg** 调用libavformat库（包含demuxers（信号分离器）），来读取输入的文件并且从中获取包含编码的数据包。当这里有多个输入文件的时候，ffmpeg就会尝试让它们保持同步通过跟踪时间最最短的输入流。  

Encoded packets are then passed to the decoder (unless streamcopy is selected for the stream, see further for a description). The decoder produces uncompressed frames (raw video/PCM audio/...) which can be processed further by filtering (see next section). After filtering, the frames are passed to the encoder, which encodes them and outputs encoded packets. Finally those are passed to the muxer, which writes the encoded packets to the output file.  

编码的数据包然后被传入解码器（除非是这个流的流复制选项被选择了，参阅进一步说明）。解码器产生的是没有压缩过的的frames帧（生的video/PCM audio/...） 这些可以被进一步处理通过过滤器（见下一节）。过滤完成后，这些帧被传入编码器，编码器将它们编码并且输出编码后的数据包。最后，这些数据包被传入到muxer（合成器），muxer将编码后的数据包写入到输出文件中。  

## 3.1 Filtering 过滤

Before encoding, **ffmpeg** can process raw audio and video frames using filters from the libavfilter library. Several chained filters form a filter graph. **ffmpeg** distinguishes between two types of filtergraphs: simple and complex.

编码之前，**ffmpeg** 可以处理生的音频和视频帧，通过使用过滤器，过滤器在libavfilter包中。 几个链式的过滤器形成一个过滤器图。**ffmpeg** 分成了两种类型的过滤器图，简单类型和复杂类型。  

### 3.1.1 Simple filtergraphs 简单过滤器图

Simple filtergraphs are those that have exactly one input and output, both of the same type. In the above diagram they can be represented by simply inserting an additional step between decoding and encoding:  

简单过滤器图就是那些有着明确的一个输入一个输出的， 输入输出都是同一种类型。在之前的图表中它们可以被表示，通过简单的插入一个附加的一步在解码和编码之间：  

```
 _________                        ______________
|         |                      |              |
| decoded |                      | encoded data |
| frames  |\                   _ | packets      |
|_________| \                  /||______________|
             \   __________   /
  simple     _\||          | /  encoder
  filtergraph   | filtered |/
                | frames   |
                |__________|

```

Simple filtergraphs are configured with the per-stream -filter option (with -vf and -af aliases for video and audio respectively). A simple filtergraph for video can look for example like this:   

简单过滤器图被配置通过在流之前使用 -filter 选项 （-vf 和 -af 别名分别代表video和audio）。一个简单过滤器图可以被看做如下示例：  

```
 _______        _____________        _______        ________
|       |      |             |      |       |      |        |
| input | ---> | deinterlace | ---> | scale | ---> | output |
|_______|      |_____________|      |_______|      |________|
```

Note that some filters change frame properties but not frame contents. E.g. the fps filter in the example above changes number of frames, but does not touch the frame contents. Another example is the setpts filter, which only sets timestamps and otherwise passes the frames unchanged.  

注意，一些过滤器修改frame帧的属性但是没有修改frame帧的内容。比如，fps过滤器在这个例子中修改了帧的数量，但是没有触碰帧的内容。另一个例子是，setpts过滤器，只设置时间戳，并且经过的帧是没有改变的。  

### 3.1.2 Complex filtergraphs 复杂过滤器图

Complex filtergraphs are those which cannot be described as simply a linear processing chain applied to one stream. This is the case, for example, when the graph has more than one input and/or output, or when output stream type is different from input. They can be represented with the following diagram:  

复杂过滤器图是不能被描述为一个直线的处理链应用到一个流上。这是个例子，例如，当这个图有不止一个输入输出的时候，或者输入输出流类型不同的时候。它们可以被表示为如下图表：  

```
 _________
|         |
| input 0 |\                    __________
|_________| \                  |          |
             \   _________    /| output 0 |
              \ |         |  / |__________|
 _________     \| complex | /
|         |     |         |/
| input 1 |---->| filter  |\
|_________|     |         | \   __________
               /| graph   |  \ |          |
              / |         |   \| output 1 |
 _________   /  |_________|    |__________|
|         | /
| input 2 |/
|_________|

```

Complex filtergraphs are configured with the -filter_complex option. Note that this option is global, since a complex filtergraph, by its nature, cannot be unambiguously associated with a single stream or file.  

复杂过滤器图被配置通过 -filter_complex 选项。注意，这个选项是全局的，既然是一个复杂过滤器图，按照其性质，不可能仅仅跟一个流 文件关联。  

The -lavfi option is equivalent to -filter_complex.  

-lavfi 选项跟 -filter_complex 等价。  

A trivial example of a complex filtergraph is the overlay filter, which has two video inputs and one video output, containing one video overlaid on top of the other. Its audio counterpart is the amix filter.  

一个不重要的复杂过滤器图例子是overlay覆盖物过滤器，这个过滤器有两个视频输入一个视频输出，包含一个视频覆盖在另一个上边。它的音频配对物，是一个混合过滤器。  

### 3.2 Stream copy 流复制

Stream copy is a mode selected by supplying the **copy** parameter to the -codec option. It makes **ffmpeg** omit the decoding and encoding step for the specified stream, so it does only demuxing and muxing. It is useful for changing the container format or modifying container-level metadata. The diagram above will, in this case, simplify to this:  

流复制是一个模式，通过应用一个 **copy** 参数在 -codec 选项上来选中这个模式。它使**ffmpeg**省略解码编码的步骤对一个指定的流，所以它只负责demuxing和muxing。它对于改变container format或者修改container-level metadata是有用的。在这个情况下，上边的图可以简化为如下：  

```
 _______              ______________            ________
|       |            |              |          |        |
| input |  demuxer   | encoded data |  muxer   | output |
| file  | ---------> | packets      | -------> | file   |
|_______|            |______________|          |________|
```

Since there is no decoding or encoding, it is very fast and there is no quality loss. However, it might not work in some cases because of many factors. Applying filters is obviously also impossible, since filters work on uncompressed data.  

既然这里没有解码编码，所以它会非常快，并且没有质量损失。然而，它有可能在一些情况下不工作，由于many factors。应用过滤器很明显也不可能了，因为过滤器工作在不压缩的数据上。  

## 4 Stream selection 流选择

By default, **ffmpeg** includes only one stream of each type (video, audio, subtitle) present in the input files and adds them to each output file. It picks the "best" of each based upon the following criteria: for video, it is the stream with the highest resolution, for audio, it is the stream with the most channels, for subtitles, it is the first subtitle stream. In the case where several streams of the same type rate equally, the stream with the lowest index is chosen.  

默认情况下，**ffmpeg**从所有输入文件中每种类型只包含一个流（video，audio，subtitle），并且把它们添加到每一个输出文件中。**ffmpeg**选择每一个流里边最好的，基于以下几个标准：视频，选择最高分辨率的，音频，选择最多声道的，字幕，选择第一个字幕流。在这个情况下，几个相同类型流的评分相同，选择索引最小的。  

You can disable some of those defaults by using the **-vn/-an/-sn/-dn** options. For full manual control, use the **-map** option, which disables the defaults just described.   

你可以禁用这里的一些默认通过使用 **-vn/-an/-sn/-dn** 这些选项。完全手动控制，使用 **-map** ，会自动禁用刚才的默认。

## 5 Options 选项

All the numerical options, if not specified otherwise, accept a string representing a number as input, which may be followed by one of the SI unit prefixes, for example: ’K’, ’M’, or ’G’.  

所有的数字选项，如果没有另外说明，接受一个字符串代表数字作为输入，这些数字输入后边可能会跟着一个单位的首写字母，例如，’K’, ’M’, or ’G’.  

If ’i’ is appended to the SI unit prefix, the complete prefix will be interpreted as a unit prefix for binary multiples, which are based on powers of 1024 instead of powers of 1000. Appending ’B’ to the SI unit prefix multiplies the value by 8. This allows using, for example: ’KB’, ’MiB’, ’G’ and ’B’ as number suffixes.   

如果‘i’被添加在了单位首写字母（比如，‘K’）后边，这个完整的组合将会被解释为一个二进制倍数的单位前置，变成了是1024的幂，而不是1000的幂。添加'B'，将会在数值上乘8。这些可以使用，例如， ’KB’, ’MiB’, ’G’ and ’B’ ，作为数字后缀。  


### 5.1 Stream specifiers 流标识符









***

Reference
[can-ffmpeg-be-used-as-a-library-instead-of-a-standalone-program](https://stackoverflow.com/questions/2401764/can-ffmpeg-be-used-as-a-library-instead-of-a-standalone-program)