---
layout: post
title: "Unity surface shader"
description: "Unity surface shader"
image: ""
date: 2018-03-13 23:10:15
categories: Shader
tags: [Shader,CG,Unity]
---
<!-- more -->
* Table of Contents
{:toc}

# 什么是Surface Shader

&nbsp; &nbsp; &nbsp; &nbsp;**Surface shader**是unity引擎特有的一种shader，可以直接表示物体表面的光照模型。在Unlit shader中，必须自己编写顶点和片段着色器，而且对于多个光源的情况，还是编写多个renderpass，使用`ForwardAdd`和`ForwardBase`来区分。而在Surface Shader中，顶点着色器是可选的，而片元着色器则被一个功能相近的表面着色器取代，同时，不需要在手工处理多个光源的情况。光照模型可以使用Unity内置的光照模型，当然也可以编写自己的。

# 默认的表面着色器

&nbsp; &nbsp; &nbsp; &nbsp;`Create ➤ Shader ➤
Standard Surface Shader`可以创建一个表面着色器，里面的代码如下。

```c
Shader "Custom/SurfaceShader" {
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
	}
	
	SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
        CGPROGRAM
        // Physically based Standard lighting model, and enable shadows on all light types
        #pragma surface surf Standard fullforwardshadows
        
        // Use shader model 3.0 target, to get nicer looking lighting
        #pragma target 3.0
        
        sampler2D _MainTex;
        
        struct Input {
            float2 uv_MainTex;
		};
		
        half _Glossiness;
        half _Metallic;
        fixed4 _Color;
        
        // Add instancing support for this shader. You need to check 'Enable Instancing' on
        materials that use the shader.
        // See https://docs.unity3d.com/Manual/GPUInstancing.html for more information about
        instancing.
        // #pragma instancing_options assumeuniformscaling
        UNITY_INSTANCING_CBUFFER_START(Props)
            // put more per-instance properties here
        UNITY_INSTANCING_CBUFFER_END
        
        void surf (Input IN, inout SurfaceOutputStandard o) {
            // Albedo comes from a texture tinted by color
            fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
            // Metallic and smoothness come from slider variables
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
            o.Alpha = c.a;
		}
		ENDCG 
	}
    FallBack "Diffuse"
}
```

&nbsp; &nbsp; &nbsp; &nbsp;接下来解释该程序。

## Pragmas

&nbsp; &nbsp; &nbsp; &nbsp;表示表面着色器的Pragmas语法如下

```c
#pragma surface <surfaceFunc> <lightModel> [vertex:<vertFunc>] [<shadowType>]
```

&nbsp; &nbsp; &nbsp; &nbsp;宏中必须指定表面着色函数，还有使用的光照模型，指定顶点着色函数，还有阴影类型都是可选的，比如如下都是正确的。

```c
#pragma surface surf Standard fullforwardshadows
#pragma surface surf Lambert vertex:vert
```

> Standard，Lambert，Blinn-phong都是Unity内置的光照模型

## SurfaceOutputStandard

&nbsp; &nbsp; &nbsp; &nbsp;`SurfaceOutputStandard`对象在`UnityPBSLighting.cginc`文件中被定义，Unity的表面着色器会自动include一系列头文件，不需要再次手工引入。

```c
struct SurfaceOutputStandard
{
	fixed3 Albedo;     // base (diffuse or specular) color
	fixed3 Normal;     // tangent space normal, if written
	half3 Emission;
	half Metallic;     // 0=non-metal, 1=metal
	half Smoothness;   // 0=rough, 1=smooth
	half Occlusion;    // occlusion (default 1)
	fixed Alpha;       // alpha for transparencies
};
```

&nbsp; &nbsp; &nbsp; &nbsp;如果熟悉PBR，那对这个表面对象的各个参数应该比较了解了吧。

## Surface Function

&nbsp; &nbsp; &nbsp; &nbsp;shader的主角来啦。

```c
void surf (Input IN, inout SurfaceOutputStandard o) {
    fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
    o.Albedo = c.rgb;
    // Metallic and smoothness come from slider variables
    o.Metallic = _Metallic;
    o.Smoothness = _Glossiness;
    o.Alpha = c.a;
}
```

&nbsp; &nbsp; &nbsp; &nbsp;在表面着色函数中，并没有使用SurfaceOutputStandard对象的全部参数，只对其中常用的四个参数进行了赋值。

## Lighting Model

&nbsp; &nbsp; &nbsp; &nbsp;如果想理解内置shader，就一定少不了光照模型，不过Unity Standard lighting function非常复杂，我使用更简单更好理解的`Lambert`光照模型进行讲解，该函数可以在`Lighting.cginc`找到。

```c
inline fixed4 LightingLambert (SurfaceOutput s, UnityGI gi)
{
    fixed4 c;
    UnityLight light = gi.light;
    fixed diff = max (0, dot (s.Normal, light.dir));
    c.rgb = s.Albedo * light.color * diff;
    c.a = s.Alpha;
    #ifdef UNITY_LIGHT_FUNCTION_APPLY_INDIRECT
        c.rgb += s.Albedo * gi.indirect.diffuse;
	#endif
	return c; 
}
```

&nbsp; &nbsp; &nbsp; &nbsp; `UnityGI`是一个unity内置的数据结构，用来计算照射到物体表面的光线(比如直接光和各种间接光)，使用全局光照系统(global illumination system)。GI相比直接使用一个简单的环境光常量(Ambient)能更好的表现真实的物理环境。

# 使用另一个光照模型

&nbsp; &nbsp; &nbsp; &nbsp;修改这个shader，使用BlinnPhong光照模型。

```c
Shader "Custom/SurfaceShaderBlinnPhong" {
    Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_SpecColor ("Specular Material Color", Color) = (1,1,1,1) 
		_Shininess ("Shininess", Range (0.03, 1)) = 0.078125
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		CGPROGRAM
		#pragma surface surf BlinnPhong fullforwardshadows 
		#pragma target 3.0
		
        sampler2D _MainTex;
        float _Shininess;
        
        struct Input {
            float2 uv_MainTex;
		};
		
        fixed4 _Color;
        UNITY_INSTANCING_CBUFFER_START(Props)
            // put more per-instance properties here
        UNITY_INSTANCING_CBUFFER_END
        
		void surf (Input IN, inout SurfaceOutput o) {
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color; 
			o.Albedo = c.rgb;
			o.Specular = _Shininess;
			o.Gloss = c.a;
			o.Alpha = 1.0f;
		}
		ENDCG 
	}
    FallBack "Diffuse"
}
```

&nbsp; &nbsp; &nbsp; &nbsp;这个代码有人可能会问，`_SpecColor`用在哪里了，其实用在BlinnPhong的光照模型中了。

```c
inline fixed4 UnityPhongLight (SurfaceOutput s, half3 viewDir, UnityLight light)
{
    half3 h = normalize (light.dir + viewDir);
    
    fixed diff = max (0, dot (s.Normal, light.dir));
    
	float nh = max (0, dot (s.Normal, h));
	float spec = pow (nh, s.Specular*128.0) * s.Gloss;
	
	fixed4 c;
	c.rgb = s.Albedo * light.color * diff + light.color * _SpecColor.rgb * spec; 
	c.a = s.Alpha;
	return c;
}
```

# 编写自己的Lighting Model

## 函数声明

&nbsp; &nbsp; &nbsp; &nbsp;编写自己的光照模型函数，有两种形式，一种纯漫反射模型(和视角方向无关)，另一种和视角有关。

```c
half4 Lighting<Name> (SurfaceOutput s, UnityGI gi);
half4 Lighting<Name> (SurfaceOutput s, half3 viewDir, UnityGI gi);
```

&nbsp; &nbsp; &nbsp; &nbsp;当然这是在前向渲染的情况下，在使用延迟渲染的情况下，还有额外的两个函数：

```c
half4 Lighting<Name>_Deferred (SurfaceOutput s, UnityGI gi, out half4 outDiffuseOcclusion, out half4 outSpecSmoothness, out half4 outNormal);
half4 Lighting<Name>_PrePass (SurfaceOutput s, half4 light);
```

> 不过这次就不用了

## 编写Surface Function

&nbsp; &nbsp; &nbsp; &nbsp;表面着色器就很简单了

```c
void surf (Input IN, inout SurfaceOutput o) {
    fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
    o.Albedo = c.rgb;
    o.Alpha = 1.0f;
}
```

## 编写自定义的光照函数

&nbsp; &nbsp; &nbsp; &nbsp;Unity没有内建的Phong模型，只有BlinnPhong模型，所以我就实现一下Phong光照了。

&nbsp; &nbsp; &nbsp; &nbsp;首先实现光照模型函数：

```c
inline fixed4 LightingPhong (SurfaceOutput s, half3 viewDir, UnityGI gi)
{
    UnityLight light = gi.light;
    
    float nl = max(0.0f, dot(s.Normal, light.dir));
    float3 diffuseTerm = nl * s.Albedo.rgb * light.color;
    float3 reflectionDirection = reflect(-light.dir, s.Normal);
    float3 specularDot = max(0.0, dot(viewDir, reflectionDirection)); //no more ambient
    float3 specular = pow(specularDot, _Shininess);
    float3 specularTerm = specular * _SpecColor.rgb * light.color.rgb;
    
    float3 finalColor = diffuseTerm.rgb + specularTerm;
    
    fixed4 c;
    c.rgb = finalColor;
    c.a = s.Alpha;
    
    #ifdef UNITY_LIGHT_FUNCTION_APPLY_INDIRECT
        c.rgb += s.Albedo * gi.indirect.diffuse;
	#endif
	
	return c; 
}
```

&nbsp; &nbsp; &nbsp; &nbsp;但是除此之外，还需要提供一个额外的GI函数，用来计算GI。和BlinnPhong模型相同即可：

```c
inline void LightingPhong_GI (SurfaceOutput s, UnityGIInput data, inout UnityGI gi)
{
    gi = UnityGlobalIllumination (data, 1.0, s.Normal);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;然后修改pragma来使用Phong模型

```c
#pragma surface surf Phong fullforwardshadows
```

## 完整代码

&nbsp; &nbsp; &nbsp; &nbsp;完整代码如下：

```c
Shader "Custom/SurfaceShaderCustomPhong" {
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _SpecColor ("Specular Material Color", Color) = (1,1,1,1) 
        _Shininess ("Shininess", Range (0.03, 128)) = 0.078125
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
        CGPROGRAM
        #pragma surface surf Phong fullforwardshadows 
        #pragma target 3.0
        
        sampler2D _MainTex;
        float _Shininess;
        fixed4 _Color;
        
        struct Input {
            float2 uv_MainTex;
        };

        UNITY_INSTANCING_CBUFFER_START(Props)
        UNITY_INSTANCING_CBUFFER_END

        inline void LightingPhong_GI (SurfaceOutput s, UnityGIInput data, inout UnityGI gi)
        {
            gi = UnityGlobalIllumination (data, 1.0, s.Normal);
        }

        inline fixed4 LightingPhong (SurfaceOutput s, half3 viewDir, UnityGI gi)
        {
            UnityLight light = gi.light;

            float nl = max(0.0f, dot(s.Normal, light.dir));
            float3 diffuseTerm = nl * s.Albedo.rgb * light.color;

            float3 reflectionDirection = reflect(-light.dir, s.Normal);
            float3 specularDot = max(0.0, dot(viewDir, reflectionDirection)); 
            float3 specular = pow(specularDot, _Shininess);
            float3 specularTerm = specular * _SpecColor.rgb * light.color.rgb;

            float3 finalColor = diffuseTerm.rgb + specularTerm;

            fixed4 c;
            c.rgb = finalColor;
            c.a = s.Alpha;

            #ifdef UNITY_LIGHT_FUNCTION_APPLY_INDIRECT
                c.rgb += s.Albedo * gi.indirect.diffuse;
            #endif
            return c;
        }
        
        void surf (Input IN, inout SurfaceOutput o) {
            fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
            o.Alpha = 1.0f;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```