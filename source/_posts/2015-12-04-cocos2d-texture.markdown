---
layout: post
title: "cocos2d - 纹理的选择 "
date: 2015-12-04 12:15:05 +0800
comments: true
categories: 
---

普通的2D纹理就是一张图片,在cocos2d里,纹理的使用主要分为两步:

- 一,从本地加载纹理图片,并将纹理数据读取到内存。
- 二,将内存里的纹理数据以copy的形式上传到GPU内存。

<font color='#bd260d'>**前者主要涉及到的问题是图片尺寸以及加载后的内存占用。**</font>例如,我们要从本地加载了一张1024\*1024的png图片(假设是RGBA8888格式),其大小约为1024\*1024\*4 = 4MB。这意味着,如果这张图片打包进安装包就会使整个程序增加4MB,如果在运行时加载到游戏里就相当于多了至少4MB内存。
	
对于一个有责任心(小肚鸡肠)的开发者来说,程序无缘无故增加了4MB内存,无疑是令人耿耿于怀的。能不能把尺寸或者运行时内存减少一点呢?答案是可以的。

要解决纹理的内存问题,首先,我们应当了解一下纹理图片为何会占了这么多内存。对于,我们常见的图片,无论是png还是jpg图片,他们都是非压缩纹理。非压缩纹理的意思是,这些图片的RGBA四个通道的像素数据在文件里都是按照一定顺序规则排列的。类似于RGBA,RGBA,....,RGBA这样的排列,不同的非压缩格式图片的区别在于,有一些只使用了rgb三个通道值,有一些使用的通道精度是小于2^8。

cocos2d支持十余种格式的非压缩纹理,上面提到的RGBA8888就是其中一种,也是最常见的一种(一般来说,没有特殊设置图片的像素格式就是RGBA8888)。并且图片的扩展名和它的像素格式没什么直接关系,一个png图片其像素格式可能是RGBA8888、RGBA4444也或者是其他格式。

说到这里,那么RGBA8888纹理有什么特别的地方呢？首先,该纹理格式上的每个像素点包括了R、G、B以及Alpha四个通道值,并且每个通道值是精度2^8,图片数据是以二进制存储的,我们知道2^8的bit值等于1byte,所以RGBA8888格式的纹理每个像素通道大小是1byte,所以该格式的纹理每个像素大小就是4*1byte = 4byte。

以此推之,RGBA4444像素格式就是通道精度为 2^4,包含RGBA四个通道的纹理。其大小为4\*0.5byte = 2byte。由此可见,RGBA4444纹理的大小是RGBA8888的一半。也就是说,如果我们有张RGBA8888的纹理图片,如果能把它转化为RGBA4444,其尺寸立刻就减小了二分之一。多么诱人,但是我们也注意到RGBA4444相较于RGBA8888,其精度只有2^4=16。这种精度的损失对一些颜色渐变的纹理来说,影响比较明显。但是无论精度区间是[0,16]还是[0,256],这些像素都会被映射到OpenGL支持的[0,1]这个颜色区间。

所以,我们应当根据实际情况选取合适的像素格式来减少图片尺寸和运行时的内存。<font color='#bd260d'>**通常来说,对于非压缩纹理 rgb565 和 rgba5551(RGB5A1) 的效果和内存占用相对会好一些。**</font>

cocos2d支持的非压缩纹理像素格式:

	    enum class PixelFormat
	    {
	        //! auto detect the type
	        AUTO,
	        //! 32-bit texture: BGRA8888
	        BGRA8888,
	        //! 32-bit texture: RGBA8888
	        RGBA8888,
	        //! 24-bit texture: RGBA888
	        RGB888,
	        //! 16-bit texture without Alpha channel
	        RGB565,
	        //! 8-bit textures used as masks
	        A8,
	        //! 8-bit intensity texture
	        I8,
	        //! 16-bit textures used as masks
	        AI88,
	        //! 16-bit textures: RGBA4444
	        RGBA4444,
	        //! 16-bit textures: RGB5A1
			RGB5A1,
			
	        //省略了压缩纹理
	        //...
	     }

#加载时的设置

<font color='#bd260d'>**不同的非压缩纹理时,在加载时需要注意格式问题。**</font>

cocos2d对纹理的默认处理方式是PixelFormat::NONE,这个设置在游戏运行时会被转化为默认值为"RGBA8888"。但是,如果要加载的纹理图片的格式RGBA4444,这时引擎就会通过转换函数将RGBA4444转为RGBA8888。

这种转换不是我们想要的,我们想直接加载并绘制RGBA4444这种格式的纹理图片。因为RGBA4444是OpenGL可以直接读取并渲染的格式,我们不需要再将其转化为其他格式。(转换不但会耗费一定的CPU,也会增加运行时内存(rgba4444->rgba888内存增加一倍))

所以,在加载时如果遇到非RGBA8888的情况,需要进行代码声明:

	//在这里,我们高呼,我们加载的格式是RGBA5551,不用帮我们转换
	[Texture2D setDefaultAlphaPixelFormat:kTexture2DPixelFormat_RGB5A1];
    Sprite *s1 = [Sprite spriteWithFile:@"rgba5551.png"];
    [self addChild:s1];
    
    //在RGBA5551加载完后,我们告诉引擎你可以去按照你默认的格式(RGBA8888)去处理其他纹理了
    [Texture2D setDefaultAlphaPixelFormat:kTexture2DPixelFormat_RGBA8888];
    Sprite *s2 = [Sprite spriteWithFile:@"rgba8888.png"];
    [self addChild:s2];

需要注意的是,在Sprite里,除了initWithTexture以外,其他全部创建方法会将纹理自动加载到内存并向GPU内存上传纹理数据。所以,实际开发中如果图像资源中同时包含多种纹理格式,一定要小心处理。

OpenGL通过glTexImage2D上传非压缩纹理,虽然GPU拥有高效的图像处理速度,但是从本地内存向显卡内存传递图片数据仍然是CPU和GPU共同处理的过程,资源的绘制效率也会受资源数据的大小限制(传输中的宽带问题,数据越大传输时间越长),所以预先上传纹理数据会对纹理的绘制效率有很大的提高,可以参见Texture2DCache对纹理缓存的管理。

下面的代码是Cocos里向 GPU 内存上传非压缩纹理数据的源码:

	glTexImage2D(GL_TEXTURE_2D, i, info.internalFormat, 
				 (GLsizei)width, (GLsizei)height, 0, info.format, info.type, data);
	
	
其中,第三个参数的可选值为:

	GL_RGB,GL_RGBA,GL_ALPHA,GL_LUMINANCE,GL_LUMINANCE_ALPHA 
	
倒数第二个参数的格式可选值为:

	GL_UNSIGNED_BYTE,GL_UNSIGNED_SHORT_5_6_5,GL_UNSIGNED_SHORT_4_4_4_4,GL_UNSIGNED_SHORT_5_5_5_1
	
是不是有点熟悉,这两个参数的组合其实就是我们上面提到的某一个像素格式,比如RGBA4444对应的就是GL_RGBA和GL_UNSIGNED_SHORT_4_4_4_4。

#压缩纹理

除了非压缩纹理外,cocos2d也支持压缩纹理,压缩纹理是通过一些优化算法将像素数据进行压缩,这些压缩纹理可以在相应的GPU直接解析(比如PVR格式的文件就可以直接 PowerVR图形芯片 解析)。

在OpenGL里,压缩纹理通过 glCompressedTexImage2D 接口将纹理数据从本地上传到GPU内存:

	glCompressedTexImage2D(GL_TEXTURE_2D, 	i, 	info.internalFormat,
						   (GLsizei)width, 	(GLsizei)height, 	0, 	datalen, data);


Cocos里支持的压缩纹理格式:
	
	enum class PixelFormat
    {
    	...
    
		//! 4-bit PVRTC-compressed texture: PVRTC4
        PVRTC4,
        //! 4-bit PVRTC-compressed texture: PVRTC4 (has alpha channel)
        PVRTC4A,
        //! 2-bit PVRTC-compressed texture: PVRTC2
        PVRTC2,
        //! 2-bit PVRTC-compressed texture: PVRTC2 (has alpha channel)
        PVRTC2A,
        //! ETC-compressed texture: ETC
        ETC,
        //! S3TC-compressed texture: S3TC_Dxt1
        S3TC_DXT1,
        //! S3TC-compressed texture: S3TC_Dxt3
        S3TC_DXT3,
        //! S3TC-compressed texture: S3TC_Dxt5
        S3TC_DXT5,
        //! ATITC-compressed texture: ATC_RGB
        ATC_RGB,
        //! ATITC-compressed texture: ATC_EXPLICIT_ALPHA
        ATC_EXPLICIT_ALPHA,
        //! ATITC-compresed texture: ATC_INTERPOLATED_ALPHA
        ATC_INTERPOLATED_ALPHA,
        //! Default texture format: AUTO
        DEFAULT = AUTO,
        
        NONE = -1
    };


#纹理选择

- <font color='#bd260d'>**在 iOS 设备上最好使用PVR格式的压缩格式**</font>,因为iOS设备采用的是 PowerVR 图形芯片,PVR格式图片在PowerVR图形芯片中效率极高，占用显存也小。[其中PVRTC4格式的PVR有损压缩包pvr.ccz,尺寸是原来的1/8](http://blog.chukong-inc.com/index.php/2013/02/04/%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96cocos2d-x%E6%B8%B8%E6%88%8F%E7%9A%84%E5%86%85%E5%AD%98/)

- <font color='#bd260d'>**在 Android 设备上使用双层 ETC1 的压缩纹理**</font>(极少数的GPU对ETC格式支持有问题。)。

#TexturePacker

TexturePacker是一款很出众的纹理压缩工具,上述的纹理格式都可以通过TexturePacker工具导出,下图是以导出一个PVRTC4格式的pvr.ccz文件的截图:

![](/images/2015/12/texture_packer.png)

#参见

- [如何优化Cocos2d-X游戏的内存](http://blog.chukong-inc.com/index.php/2013/02/04/%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96cocos2d-x%E6%B8%B8%E6%88%8F%E7%9A%84%E5%86%85%E5%AD%98/)
- [pvr与png的内存占用](http://blog.csdn.net/kaitiren/article/details/8054856)
- [ImageAlpha Mac](http://pngmini.com/)
- <https://www.khronos.org/opengles/sdk/docs/man/xhtml/glTexImage2D.xml>
- <http://www.learn-cocos2d.com/2011/11/depth-ios-cocos2d-performance-analysis-test-project/#image-formats>
- <http://matrixcn.tumblr.com/post/75239316805/%E6%89%B9%E9%87%8F%E8%BD%AC%E6%8D%A2png%E5%9B%BE%E7%89%87%E5%88%B0rgba4444>

