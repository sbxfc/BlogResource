---
layout: post
title: "iOS - 音频格式的选择"
date: 2014-02-14 11:53:47 +0800
comments: true
categories: 
---

iOS系统支持大部分音频格式的编码(录音)和解码(播放)

#首选格式

<font color='#bd260d'>**1,对于未压缩的（最高品质）的音频，封装在CAF文件中的16位、低位字节、线性PCM音频数据。**</font>可以用Command-line tool来将音频文件转化为上述格式:

	/usr/bin/afconvert -f caff -d LEI16 {INPUT} {OUTPUT}

<font color='#bd260d'>**2,对于压缩音频，当每次只需播放一个声音，或者当不需要和iPod同时播放音频时，适合使用AAC格式的CAF或m4a文件**</font>

<font color='#bd260d'>**3,如果需要在同时播放多路声音时减少内存开销，请使用IMA4 (IMA/ADPCM)压缩格式，这样可以减少文件尺寸，同时在解压缩过程中对CPU的影响又最小。和线性PCM数据一样，请将IMA4数据封装在CAF文件中。**</font>


#解码(播放)支持:

音频格式 | 硬件解码 | 软件解码
:------------ | :------------- | :------------
	AAC (MPEG-4 Advanced Audio Coding) | Yes | Yes, starting in iOS 3.0
	ALAC (Apple Lossless) | Yes  | Yes, starting in iOS 3.0
	HE-AAC (MPEG-4 High Efficiency AAC) | Yes  | -
	iLBC (internet Low Bitrate Codec, another format for speech)    | -| Yes
	IMA4 (IMA/ADPCM) | -  | Yes
Linear PCM (uncompressed, linear pulse-code modulation) | -  | Yes
	MP3 (MPEG-1 audio layer 3) | Yes  | Yes, starting in iOS 3.0
	µ-law and a-law | -  | Yes

#编码(播放)支持:

音频格式 | 硬件编码 | 软件编码
:------------ | :------------- | :------------
AAC (MPEG-4 Advanced Audio Coding) | Yes, starting in iOS 3.1 for iPhone 3GS and iPod touch (2nd generation) Yes, starting in iOS 3.2 for iPad | Yes, starting in iOS 4.0 for iPhone 3GS and iPod touch (2nd generation)
ALAC (Apple Lossless) | -  | Yes
iLBC (internet Low Bitrate Codec, another format for speech) | -| Yes
IMA4 (IMA/ADPCM) | -  | Yes
Linear PCM (uncompressed, linear pulse-code modulation) | -  | Yes
MP3 (MPEG-1 audio layer 3) | -  | Yes
µ-law and a-law | -  | Yes

#AVAudioPlayer

在iOS程序里,Apple建议使用此类播放本地音频文件。在使用AVAudioPlayer时要引入<font color='#bd260d'>**AVFoundation.framework**</font>框架。

AudioManager.h

	#import <AVFoundation/AVFoundation.h>
	@interface AudioManager : NSObject <AVAudioPlayerDelegate>
	@end
	
AudioManager.m

	- (void) play
	{
            if (player != nil)
            {
            	 //访问isPlaying属性,首先要验证player指针是否为空,空指针会出错 EXC_BAD_ACCESS
                if (player.isPlaying == YES)
                    [player stop];
            }
            
            // 设置音乐文件路径
            NSString *soundFilePath = [[NSBundle mainBundle] pathForResource:audio ofType: @"caf"];
            
            // 判断是否可以访问这个文件
            if ([[NSFileManager defaultManager] fileExistsAtPath:soundFilePath])
            {
                NSError* error;
                // 设置 player
                player = [[AVAudioPlayer alloc] initWithContentsOfURL:
                          [NSURL fileURLWithPath:soundFilePath] error:&error];
                
                // 调节音量 (范围从0到1)
                player.volume = 0.4f;
                
                // 准备buffer，减少播放延时的时间
                [player prepareToPlay];
                
                // 设置播放次数，0为播放一次，负数为循环播放
                [player setNumberOfLoops:0];
                
                //设置代理
                [player setDelegate: self];

                [player play];
            }
	}
	
	    //暂停
	- (void)pause
	{
		[player pause];
	}
	
	//停止
	- (void)stop
	{
		player.currentTime = 0;  //当前播放时间设置为0
		[player stop];
	}

	//播放完成后的委托函数
	- (void) audioPlayerDidFinishPlaying: (AVAudioPlayer *) player successfully: (BOOL) completed {
    	if (completed == YES) {
        	NSLog(@"播放完毕");
    	}
	}

除可以监听完成播放之外,还支持以下几个委托方法：
 
- *audioPlayerDecodeErrorDidOccur 解码错误* 
- *audioPlayerBeginInteruption 处理中断*  
- *audioPlayerEndInteruption 处理中断结束*  


相关属性:

<https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVAudioPlayerClassReference/Chapters/Reference.html#//apple_ref/doc/uid/TP40008067>


#AudioServicesPlaySystemSound

除 AVAudioPlayer 外,AudioServicesPlaySystemSound也可以用于播放音频文件。但AudioServicesPlaySystemSound常被用于播放一段简单的音效或调用设备的震动，其中音效时长要在30s以内，不支持对音效基本的控制，但可以设置一个回调函数，来控制对音效的清理及其他操作。

使用时,需要引入框架 <font color='#bd260d'>**AudioToolbox.framework**</font> 


一,震动:

	AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);

二,播放声音:

	- (void) playAudio
	{
    	SystemSoundID soundID;
    	NSString *url = [[NSBundle mainBundle] pathForResource: @"alert" ofType: @"caf"];
    	AudioServicesCreateSystemSoundID((__bridge CFURLRef)[NSURL fileURLWithPath:url], &soundID);
    	AudioServicesAddSystemSoundCompletion(soundID, NULL, NULL, SoundFinished,(__bridge_retained void *)self);
    	AudioServicesPlaySystemSound(soundID);
	}

	//音效播放完毕后调用
	static void SoundFinished(SystemSoundID soundID,void *myself)
	{
		//AudioServicesDisposeSystemSoundID(soundID);  

    	MyClass *theClass = (__bridge MyClass *)myself;
    	CFRelease(myself);
    	[theClass playAudio];//循环播放
	}

Note:Sounds played with System Sound Services are not subject to configuration using your audio session.

#参见 

Using Audio:<br>
<https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/MultimediaPG/UsingAudio/UsingAudio.html>