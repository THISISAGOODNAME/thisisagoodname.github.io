---
layout: post
title: "在Unity中实现brdf"
description: "在Unity中实现brdf"
image: ""
date: 2018-03-14 11:26:35
categories: Shader
tags: [Shader,CG,Unity]
---
<!-- more -->
* Table of Contents
{:toc}

# 简介

&nbsp; &nbsp; &nbsp; &nbsp;在游戏中，使用最广泛的BRDF模型，就是CookTorrance和Disney BRDF。接下来，我们也会实现CookTorrance(用来处理高光)以及Disney BRDF(用来处理漫反射)。

# CookTorrance BRDF(或者叫Microfacet BRDF)

&nbsp; &nbsp; &nbsp; &nbsp;CookTorrance框架的公式如下：

$$
f(l, v) = \frac{F(l, h)G(l,v,h)D(h)}{4(n \cdot l)(n \cdot v)}
$$

> 在实际工程中，CookTorrance已经成为了次表面理论的代名词。虽然该框架在光学上依然有不能表示光的衍射和干涉现象的问题，但已经可以提供非常不错的真实感。

1. **D**(Distribution)：分布函数，表示物体表面细微的凹凸变化，全称是NDF(Normal Distribution Function)，常见的NDF有Becnmann,Phong,GGX
2. **F**(Fresnel)：菲涅尔现象是物体表面同时发生反射和散射的现象，该函数用来求反射光强，一般使用Schlick的近似经验模型
3. **G**(Geometry)：物体的几何特性，用来描述次表面上遮挡和微小阴影

&nbsp; &nbsp; &nbsp; &nbsp;本例中，分布式函数使用GGX，菲涅尔函数使用Schlick近似，几何函数使用Schlick近似的史密斯阴影方程(Smith's shadowing function)。

## Fresnel

&nbsp; &nbsp; &nbsp; &nbsp;Schlick近似经验模型如下：

$$
F_{schlick}(v,h)=c_{spec} + (1-c_{spec})(1-(v \cdot h))^5
$$

用代码实现就是：

```c
float SchlickFresnel(float4 SpecColor, float lightDir, float3 halfVector)
{
    return SpecColor + ( 1 - SpecColor ) * pow( 1 - ( dot( lightDir, halfVector )), 5);
}
```

## Geometry

&nbsp; &nbsp; &nbsp; &nbsp;接下来实现Schlick近似的史密斯阴影方程。我们使用UE4修改过的版本，来和我们选择的NDF(GGX)兼容。

> The slides and course notes for *Real Shading in Unreal Engine 4*, presented by Brian
Karis at SIGGRAPH, 2013

&nbsp; &nbsp; &nbsp; &nbsp;首先定义一个变量$k$

$$
k=\frac{(Roughness + 1)^2}{8}
$$

用代码实现就是：

```c
float modifiedRoughness = _Roughness + 1;
float k = sqr(modifiedRoughness) / 8;
```

&nbsp; &nbsp; &nbsp; &nbsp;这个变量是要在$G_1$函数中使用的：

$$
G_1(v) = \frac{n \cdot v}{(n \cdot v)(1 - k) + k}
$$

用代码实现就是：

```c
float G1 (float k, float NdotV)
{
    return NdotV / (NdotV * (1 - k) + k);
}
```

&nbsp; &nbsp; &nbsp; &nbsp;将$G_1$同时作用于光的方向和实现方向，然后求积，就是几何函数*G*:

$$
G(l, v, h) = G_1(l)G_1(v)
$$

用代码实现就是：

```c
float g1L = G1(k, NdotL);
float g1V = G1(k, NdotV);
G = g1L * g1V;
```

# Distribution

&nbsp; &nbsp; &nbsp; &nbsp;接下来实现NDF，也就是分布函数，我们使用Unreal engine修改过的GGX函数：

$$
D(h) = \frac{\alpha^2}{\pi ((n \cdot h)^2(\alpha^2 - 1) + 1)^2}
$$

用代码实现就是：

```c
float alphaSqr = sqr(alpha);
float denominator = sqr(NdotH) * (alphaSqr - 1.0) + 1.0f;
D = alphaSqr / (PI * sqr( denominator));
```

&nbsp; &nbsp; &nbsp; &nbsp;到此为止，CookTorrance的**D**,**F**,**G**三个分量实现完毕，但是，CookTorrance只有高光没有漫反射，为了看到完整的物体，还需要补充扩散模型(diffuse)。通常使用Lambert就行了，但是还有一些更好的选择。比如我们接下来要实现的。

# Disney BRDF

## Distribution

### Diffuse

&nbsp; &nbsp; &nbsp; &nbsp;最常用的漫反射模型，就是Lambert模型了。前文介绍了Schlick近似的菲涅尔函数，下面是Schlick近似的漫反射函数公式：

$$
(1 - F(\theta_l))(1 - F(\theta_d))
$$

不过和一般的漫反射模型不同，迪士尼漫反射模型中，物体表面要发生菲涅尔现象：

$$
f_d = \frac{baseColor}{\pi} (1 + (F_{D90} - 1)(1 - cos \theta_l)^5) (1 + (F_{D90} - 1)(1 - cos \theta_v)^5)
$$

其中：

$$
F_{D90} = 0.5 + 2 \times  roughness \times cos^2\theta_d 
$$

> $\theta_x$指法向量$n$和向量$x$的点积，向量$d$就是光线$l$和视线$v$的半向量

公式看着有点乱，但是用代码实现就很清晰了：

```c
float fresnelDiffuse = 0.5 + 2 * sqr(LdotH) * roughness;
float fresnelL = 1 + (fresnelDiffuse - 1) * pow(1 - NdotL, 5);
float fresnelV = 1 + (fresnelDiffuse - 1) * pow(1 - NdotV, 5);
float3 Fd = (BaseColor / PI) * fresnelL * fresnelV
```

&nbsp; &nbsp; &nbsp; &nbsp;以上就是让物体表面因为粗糙程度的不同而有不同显示效果的经验模型了：光滑的材质稍微暗一些，通过菲涅尔阴影实现；而粗糙的材质有稍微亮一些。对于多种BRDF混合的情况，可以使用 Hanrahan-Krueger subsurface BRDF进行混合。

### Specular

&nbsp; &nbsp; &nbsp; &nbsp;接下来完成BRDF的镜面反射部分(Albedo是高光和漫反射的和)。

&nbsp; &nbsp; &nbsp; &nbsp;首先是$D$:

$$
D_{GTR} = c / (\alpha ^ 2 cos ^ 2 \theta_h + sin ^ 2 \theta_h) ^ \gamma
$$

&nbsp; &nbsp; &nbsp; &nbsp;该方程被称为Generalized-Trowbridge-Reitz(广义特罗布里奇 - 赖茨，Vray 3.4也引入了这个模型)，顾名思义，是从Trowbridge-Reitz分布来的。从最亮的区域开始亮度柔和衰减。$\alpha$是一个表示粗糙程度的参数，通常$\alpha = roughness^2$，在直觉上，这个公式下粗糙程度更像线性变化的。肉眼感知明暗程度和光的亮度并不是线性的。

&nbsp; &nbsp; &nbsp; &nbsp;Trowbridge-Reitz把镜面反射分成三部分，一个是主高光(main specular)，一部分是透明涂层(clear coat)，最后一部分是光泽(sheen)。钱两者使用GTR模型，后者使用 Fresnel Schlick 。镜面参数决定入射的镜面反射量。它重新映射涵盖了大部分的常用材质。透明涂层是一个固定值，对应聚氨酸酯的折射率。

&nbsp; &nbsp; &nbsp; &nbsp;**F**继续使用Schick近似经验模型；**G**继续使用之前略微调整的GGX模型。

# 完整代码

&nbsp; &nbsp; &nbsp; &nbsp;最终完整代码如下：

```c
Shader "Custom/CookTorranceSurface" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _ColorTint ("Color", Color) = (1,1,1,1)
        _SpecColor ("Specular Color", Color) = (1,1,1,1)
        _BumpMap ("Normal Map", 2D) = "bump" {}
        _Roughness ("Roughness", Range(0,1)) = 0.5
        _Subsurface ("Subsurface", Range(0,1)) = 0.5
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
        
        CGPROGRAM
        #pragma surface surf CookTorrance fullforwardshadows
        #pragma target 3.0

        struct Input {
            float2 uv_MainTex;
        };

        sampler2D _MainTex;
        sampler2D _BumpMap;
        float _Roughness;
        float _Subsurface;
        float4 _ColorTint;

        #define PI 3.14159265358979323846f

        UNITY_INSTANCING_CBUFFER_START(Props)
        UNITY_INSTANCING_CBUFFER_END

        struct SurfaceOutputCustom {
            float3 Albedo;
            float3 Normal;
            float3 Emission;
            float Alpha;
        };

        float sqr(float value) 
        {
            return value * value;
        }

        float SchlickFresnel(float value)
        {
            float m = clamp(1 - value, 0, 1);
            return pow(m, 5);
        }


        float G1 (float k, float x)
        {
             return x / (x * (1 - k) + k);
        }

        //Disney Diffuse 
        inline float3 DisneyDiff(float3 albedo, float NdotL, float NdotV, float LdotH, float roughness){
            float albedoLuminosity = 0.3 * albedo.r 
                                   + 0.6 * albedo.g  
                                   + 0.1 * albedo.b; // luminance approx.

            float3 albedoTint = albedoLuminosity > 0 ? 
                                albedo/albedoLuminosity : 
                                float3(1,1,1); // normalize lum. to isolate hue+sat
            
            float fresnelL = SchlickFresnel(NdotL);
            float fresnelV = SchlickFresnel(NdotV);

            float fresnelDiffuse = 0.5 + 2 * sqr(LdotH) * roughness;

            float diffuse = albedoTint 
                          * lerp(1.0, fresnelDiffuse, fresnelL) 
                          * lerp(1.0, fresnelDiffuse, fresnelV);

            float fresnelSubsurface90 = sqr(LdotH) * roughness;

            float fresnelSubsurface = lerp(1.0, fresnelSubsurface90, fresnelL) 
                                    * lerp(1.0, fresnelSubsurface90, fresnelV);

            float ss = 1.25 * (fresnelSubsurface * (1 / (NdotL + NdotV) - 0.5) + 0.5);

            return saturate(lerp(diffuse, ss, _Subsurface) * (1/PI) * albedo);
        }


        float3 FresnelSchlickFrostbite (float3 F0, float F90, float u)
        {
            return F0 + (F90 - F0) * pow (1 - u, 5) ;
        }

        inline float DisneyFrostbiteDiff(float NdotL, float NdotV
                                        , float LdotH, float roughness)
        {
            float energyBias = lerp (0, 0.5, roughness) ;
            float energyFactor = lerp (1.0, 1.0/1.51, roughness ) ;
            float Fd90 = energyBias + 2.0 * sqr(LdotH) * roughness ;
            float3 F0 = float3 (1 , 1 , 1) ;
            float lightScatter = FresnelSchlickFrostbite (F0, Fd90, NdotL).r ;
            float viewScatter = FresnelSchlickFrostbite (F0, Fd90, NdotV).r ;
            return lightScatter * viewScatter * energyFactor;
        }

        //Cook-Torrance 
        inline float3 CookTorranceSpec(float NdotL, float LdotH, float NdotH, float NdotV, float roughness, float F0){
            float alpha = sqr(roughness);
            float F, D, G;

            // D
            float alphaSqr = sqr(alpha);
            float denom = sqr(NdotH) * (alphaSqr - 1.0) + 1.0f;
            D = alphaSqr / (PI * sqr(denom));

            // F
            float LdotH5 = SchlickFresnel(LdotH);
            F = F0 + (1.0 - F0) * LdotH5;

            // G
            float r = _Roughness + 1;
            float k = sqr(r) / 8;
            float g1L = G1(k, NdotL);
            float g1V = G1(k, NdotV);
            G = g1L * g1V;
            
            float specular = NdotL * D * F * G;
            return specular;
        }

        inline void LightingCookTorrance_GI (
            SurfaceOutputCustom s,
            UnityGIInput data,
            inout UnityGI gi)
        {
            gi = UnityGlobalIllumination (data, 1.0, s.Normal);
        }

        inline float4 LightingCookTorrance (SurfaceOutputCustom s, float3 viewDir, UnityGI gi){
            UnityLight light = gi.light;

            viewDir = normalize ( viewDir );
            float3 lightDir = normalize ( light.dir );
            s.Normal = normalize( s.Normal );
            
            float3 halfV = normalize(lightDir+viewDir);
            float NdotL = saturate( dot( s.Normal, lightDir ));
            float NdotH = saturate( dot( s.Normal, halfV ));
            float NdotV = saturate( dot( s.Normal, viewDir ));
            float VdotH = saturate( dot( viewDir, halfV ));
            float LdotH = saturate( dot( lightDir, halfV ));

            float3 diff = DisneyDiff(s.Albedo, NdotL,  NdotV, LdotH, _Roughness);
            float3 spec = CookTorranceSpec(NdotL, LdotH, NdotH, NdotV, _Roughness, _SpecColor);
            float3 diff2 = (DisneyFrostbiteDiff(NdotL, NdotV, LdotH, _Roughness) * s.Albedo)/PI;
            float3 firstLayer = ( diff + spec * _SpecColor) * _LightColor0.rgb;
            float4 c = float4(firstLayer, s.Alpha);

            #ifdef UNITY_LIGHT_FUNCTION_APPLY_INDIRECT
                c.rgb += s.Albedo * gi.indirect.diffuse;
            #endif
            
            return c;
        }

        void surf (Input IN, inout SurfaceOutputCustom o) {
            float4 c = tex2D (_MainTex, IN.uv_MainTex) * _ColorTint;
            o.Albedo = c.rgb;
            o.Normal = UnpackNormal( tex2D ( _BumpMap, IN.uv_MainTex ) );
            o.Alpha = c.a;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```
