---
title: Bitstream or PCM？
date:   2017-09-27
categories: TV
tags:  [tv,pcm,coax,dolby]
---

电视上同轴输出的做法。  

<!-- more -->

### 背景

USB通道下播放声音格式为AAC的视频文件，同轴输出设置为Auto，功放没有声音，设置成PCM，有声音。


### 提问

Auto/PCM的做法是怎样的？Auto的做法这里其实就是Bitstream，PCM就是PCM。  
Bitstream与PCM差别在哪？如何取舍？为什么一个有声音，一个没有？


### 讨论精选

#### 一
Basically,  
Bitstream - will send the raw digital audio without processing out of the DVD player so your receiver can process  
PCM - the player will convert everything to and output as 2 channel PCM  
So if you want your receiver to process DD and DTS multichannel tracks then you must set the player to Bitstream  
This PCM/Bitstream option doesn't effect the players analogue outputs. This is the reason why your analogue outs sounded better than your digital out when its set to PCM.

#### 二
If you select PCM the bluray player will do the decoding and send the sound to the amp. Bitstream will send it in raw format and leave the amp to decode it. PCM would be your safest bet so all audio formats work, but maybe select bitstream and see what happens...

#### 三
[链接1](https://www.lifewire.com/what-is-pcm-1846928) [链接2](https://www.lifewire.com/blu-ray-audio-bitstream-vs-pcm-1846396)  
The PCM Option  
If you set the Blu-ray Disc player to output audio as PCM, the player will perform the audio decoding of all Dolby/Dolby TrueHD and DTS/DTS-HD Master Audio - related soundtracks internally and send the decoded audio signal in uncompressed form to your home theater receiver. As a result, your home theater receiver will not have to perform any additional audio decoding before the audio is sent through the amplifier section and the speakers. With this option, the home theater receiver will display the term "PCM" on its front panel display.

The Bitstream Option  
If you select Bitstream as the HDMI audio output setting for your Blu-ray player, the player will bypass its own internal Dolby and DTS audio decoders and send the undecoded signal to your HDMI-connected home theater receiver.

With this setting, the home theater receiver will do all the audio decoding of the incoming signal. As a result, in this case, the receiver will display Dolby, Dolby TrueHD, DTS, DTS-HD Master Audio, Dolby Atmos, DTS:X, etc...on its front panel display depending on which type of bitstream signal is being decoded.


NOTE: The Dolby Atmos and DTS:X surround sound formats are only available from a Blu-ray Disc player via the Bitstream setting option. There are no Blu-ray Disc players that can decode these formats internally and to PCM and pass that on to a home theater receiver.

#### 四
what is the difference between connecting a dvd player to amp, having the output mode of dvd player set to bitstream and pcm? how difference these setting will influence the output of sound?

**Answer 1:**  
Bitstream is digital output to the amp, it means either optical or coaxial output should be connected to the receiver. The digital to analog conversion will be done by the receiver.

PCM is analog output. Analog output should be connected to the receiver. For DTS etc, your player should have a dts decoder and the 5 channel output should be connected to the receiver's external decoder inputs.

**Answer 2:**  
Doors, you information is only partially correct. PCM is not analog, but digital. Let me explain. 

There are lots of confusion between PCM/LPCM and bitstream. This confusion is there because the two refer to completely different things - one is a encoding and storage methodology, while the other is just a transport mechanism. 

Pulse Code Modulation or PCM is a digital form of representing analog signals. PCM has two step process - one is called Modulation and other is called Demodulation. In Modulation, an analog signal is sampled at regular intervals and quantisized. For each sample, an available value is chosen using an advanced algorithm. This creates a fully discreet digital signal that can be easily stored and processed. In demodulation, the modulation process is reversed and a high frequency analog signal is created. This is them sent though a filter to remove, what we in audio call, jitter. Modulation is what we know as Analog-to-Digital conversion, and demodulation is what we call Digital-to-Analog conversion. Both your audio CD as well as your DVD store digital data that have been created using PCM. 

For a long time, because of the small spaces available for storage (CD, DVD, etc) as well narrow bandwidths available for data transmission, digital data has been stored in lossy compressed form for both audio and video. Such compression always have some loss of data. 

Using optical or co-axial connections, these compressed digital data is streamed across from one point to another. Since a bit is the most basic form of digital data, this way of transmission is called bitstreaming. Digital data is streamed using synchronous or asynchronous modes. In computer for example, TCP uses asynchronous mode for data transportation. 

Most data streams are sent as packets or frames of data and contain the following information:

* header
* error check
* audio or video data
* ancillary data

The header of each packet contains general information such as the CODEC, sampling frequency, number of channels, CRC protection, etc. On the receiving side, the data is validated for accuracy, and once validated, the actual data is processed as needed.

Over the last few years two things have happened. Storage space has increased, and new transmission methodology have been discovered that have a much higher bandwidth. HDMI 1.3, for example, can carry data at a bandwidth of 340 MHz which equals to 10.2 giga bits per second. In addition HDMI also allows multiplexing of multi data streams over a single physical link. A 192 kHz sampling frequency equates to just 6.144 gigabits per second of transmission speeds. 

Now suddenly you could store data with lossless compression, and also transmit multiple channels of data from one place to another at very high speed. 

LPCM is a term that is loosely used for both encoding and storage, and of decoding and transmission of lossless video/audio data. LPCM sampling resolutions can go up to 24 bits per sample, while PCM's resolution is a max of 16 bits. LPCM is generally used in conjunction with WAV files in computers (also FLAC, AIFF etc), and with Blu-Ray, TrueHD, DTS-HD 

PCM and bitstream is used in conjunction with traditional formats such as 2 channel stereo, Dolby Digital, DTS, etc. 

The question is which one should I use? 

If you are using coaxial or optical digital connection, you must use bitstream or what some players call RAW. Many DVD players will have PCM set as default. PCM will not send Dolby Digital or DTS as multi channel sound, but as Stereo PCM through these connections,

HD Audio such as TrueHD, DTS HD etc are stored in compressed form and cannot be transmitted as such. A high end DVD player will, thus, extract such sound from the disc, decode it, and mix it into muti channel PCM. This is then transmitted through HDMI 1.1 or higher connections. You have to ensure that your receiver not only has an HDMI input but should also have the ability to handle the multichannel PCM signal. 

Cheers  

#### 五
[拓展1](https://www.cnblogs.com/lihaiping/p/LPCM.html) [拓展2](https://blog.chinaunix.net/uid-9688646-id-1998399.html)  


### 最后

讨论部分，网友的回答已经非常明了。  

- 同轴选择Bitstream输出，则TV不做decode，直接把raw data给到功放，功放直接处理raw，比如Dolby的话，按功放面板上info，切到Audio decoder就会看到Dolby；同轴输出选择PCM输出，则TV给到功放的是已经经过处理后得到的PCM，功放info显示的是PCM。我的功放型号是雅马哈RX377。
- 对于好一点的功放，选Bitstream，听说音效听起来很好，否则保险起见，选PCM，因为大多数的功放都能解PCM。
- 同轴输出设置为Auto时没声音，是因为功放不支持AAC吗？查看[功放说明书](https://www.yamaha.com.cn/images/products/audio-visual/aduio-family/av-receiver/RX-V377/RX-V377.pdf)，没看懂支持不支持AAC。但是用功放直接播放USB里的AAC音乐文件，Audio Decoder显示为AAC，应该能说明AAC是支持的。
- 播放AAC音源没有声音的问题，原厂处理如下：[what]when audio = aac, spdif select AUTO, AV receiver doesn't output，[why]AV receiver don't support AAC raw data，[how]when audio = aac, spdif select AUTO, force SPDIF output PCM。我认为这个处理方法还有待商榷，因为AV receiver看起来是支持AAC的，所以当同轴输出设置为Bitstream时功放无声音的问题，软件层面应该还有更合理的改善的地方，而不是简单地“force SPDIF output PCM”。出于对比方案的做法参考和出货的紧迫时间，就这么处理了！
- 还有一个问题，功放插上U盘，播放U盘里面的音乐的时候，功放info可正确输出音乐的音频格式（试过了PCM、AAC、WMA），但是视频却不能（这里用TV多媒体通道播放视频，同轴输出到功放，把同轴输出设置为Auto时，info看到的东西除了PCM就是Dolby Digital，我用一个音频格式为wma的视频（这个视频是用格式工厂转码得到的，但是用MediaInfo对比这个视频的音频参数和之前测试的wma音乐的音频参数，文件格式/编码设置ID/编码设置ID/信息都一致）试了，info显示的是PCM），这是功放本身的做法问题吗？
