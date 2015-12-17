---
layout: post
title: "Unity3D - 过程纹理"
date: 2014-04-14 18:58:15 +0800
comments: true
categories: 
---
# 过程纹理

过程纹理主要用于模拟自然界中常见的Marble,Stone,Wood,Cloud等纹理。大多数的过程纹理都是基于某类噪声函数（Noise Function），比如说perlin noise。在过去，由于过程纹理计算量很大，在实时绘制中很少使用。但是GPU的出现，促进了过程纹理在实时渲染中的广泛应用。

1,新建一个名为ProceduralTexture的C#文件和一个Plane,并将脚本拖到平面上。  <br>
2，设置脚本

	using UnityEngine;
	using System.Collections;
	
	public class ProceduralTexture : MonoBehaviour 
	{
		#region Public Variables
		public int widthHeight = 512;//贴图的长宽
		public Texture2D generatedTexture;//贴图数据
		#endregion
		
		#region Private Variables
		//These variables will be internal to this
		//script
		private Material currentMaterial;
		private Vector2 centerPosition;
		#endregion
		
	
		// Use this for initialization
		void Start () 
		{
			//检查模型是否有材质
			if(!currentMaterial)
			{
				currentMaterial = transform.renderer.sharedMaterial;
				if(!currentMaterial)
				{
					Debug.LogWarning("Cannot find a material on: " + transform.name);
				}
			}
			
			//生成过程贴图
			if(currentMaterial)
			{
				//Generate the prabola texture
				centerPosition = new Vector2(0.5f, 0.5f);
				generatedTexture = GenerateParabola();
				
				//Assign it to this transforms material
				currentMaterial.SetTexture("_MainTex", generatedTexture);
			}
			
			Debug.Log(Vector2.Distance(new Vector2(256,256), new Vector2(32,32))/256.0f);
		
		}
	
		//创建新的贴图
		private Texture2D GenerateParabola()
		{
			//创建一个新的2D贴图
			Texture2D proceduralTexture = new Texture2D(widthHeight, widthHeight);
			
			//获得贴图的中心点
			Vector2 centerPixelPosition = centerPosition * widthHeight;
			
			//遍历贴图上的每一个像素点 
			for(int x = 0; x < widthHeight; x++)
			{
				for(int y = 0; y < widthHeight; y++)
				{
					//获取当前像素点距离贴图中心的距离，并将其转化为[0,1]区间，方便颜色设值
					Vector2 currentPosition = new Vector2(x,y);
					float pixelDistance = Vector2.Distance(currentPosition, centerPixelPosition)/(widthHeight*0.5f);

					//Mathf.Clamp函数确保取值在0-1之间
					//值由大变小
					pixelDistance = Mathf.Abs(1-Mathf.Clamp(pixelDistance, 0f,1f));
				
					//设置颜色值
					Color pixelColor = new Color(pixelDistance, pixelDistance, pixelDistance, 1.0f);
					proceduralTexture.SetPixel(x,y,pixelColor);

				}
			}
			//将像素应用到贴图
			proceduralTexture.Apply();
			
			//return the texture to the main program.
			return proceduralTexture;
		}
	}

3,运行之后，能看到以Panel中心点为圆心的圆。颜色由中心向外依次加深,代码里描述了这个渐变贴图的创建过程。

---
# 环绕效果
下面的代码，对贴图生成函数`GenerateParabola`进行了一些有趣的修改。

 
	private Texture2D GenerateParabola()
	{
		//Create a new Texture2D
		Texture2D proceduralTexture = new Texture2D(widthHeight, widthHeight);
		
		//Get the center of the texture
		Vector2 centerPixelPosition = centerPosition * widthHeight;
		
		//loop through each pixel of the new texture and determine its 
		//distance from the center and assign a pixel value based on that.
		for(int x = 0; x < widthHeight; x++)
		{
			for(int y = 0; y < widthHeight; y++)
			{
				//获取当前像素点距离贴图中心的距离，并将其转化为[0,1]区间，方便颜色设值
				Vector2 currentPosition = new Vector2(x,y);
				float pixelDistance = Vector2.Distance(currentPosition, centerPixelPosition)/(widthHeight*0.5f);
				//Mathf.Clamp函数确保取值在0-1之间
				//颜色值由大变小
				pixelDistance = Mathf.Abs(1-Mathf.Clamp(pixelDistance, 0f,1f));
				//环绕效果
				pixelDistance = (Mathf.Sin(pixelDistance * 30.0f) * pixelDistance);
				
				//you can also do some more advanced vector calculations to achieve
				//other types of data about the model itself and its uvs and
				//pixels
				Vector2 pixelDirection = centerPixelPosition - currentPosition;
				pixelDirection.Normalize();
				float rightDirection = Vector2.Angle(pixelDirection, Vector3.right)/360;
				float leftDirection = Vector2.Angle(pixelDirection, Vector3.left)/360;
				float upDirection = Vector2.Angle(pixelDirection, Vector3.up)/360;
				
				//Invert the values and make sure we dont get any negative values 
				//or values above 1.
				
				
				//注释打开看效果1
				//Color pixelColor = new Color(Mathf.Max(0.0f, rightDirection),Mathf.Max(0.0f, leftDirection), Mathf.Max(0.0f,upDirection), 1f);
				//星云效果2
				Color pixelColor = new Color(pixelDistance, pixelDistance, pixelDistance, 1.0f);
				//注释打开看效果3
				//Color pixelColor = new Color(rightDirection, leftDirection, upDirection, 1.0f);
				proceduralTexture.SetPixel(x,y,pixelColor);
			}
		}
		//Finally force the application of the new pixels to the texture
		proceduralTexture.Apply();
		
		//return the texture to the main program.
		return proceduralTexture;
	}
