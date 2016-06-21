---
layout: post
title: "Shader Tutorial 2"
author: "장문익"
date: 2016-06-19 03:00:00
excerpt: "Shader Tutorial 1을 바탕으로 기본적인 shader를 만들어 봅시다."
tags: [Shader, Bumped Shader, Snow Shader, Unity]
comments: true
---

# 들어가며

Tutorial 1을 바탕으로 이제 유용한 shader를 만들어 보도록 하겠습니다. 가장 기본적인 "Snow" shader를 만들도록 하겠습니다.

![ShaderSampleImage](/assets/img/Rocks.jpg)

위의 이미지는 Asset Store에서 무료로 사용할 수 있는 울퉁불퉁한(bumped) 바위에 눈을 쌓이게 만든 예시입니다.

# Shader를 만들 계획을 세워봅시다

우리가 만들고자하는 것은 제법 단순합니다. 우리는 다음과 같이 표현할 수 있습니다.

* *Snow Level*이 증가하면 우리는 픽셀을 바꿀 것입니다. 픽셀은 material의 텍스처보다는 *Snow Color*로 바꿀 것입니다.
* *Snow Level*이 증가하면 우리는 모델을 좀더 크게 변화시킬 것입니다. 커지는 방향은 눈이 내리는 방향을 향할 것입니다.

# 1단계 - Bumped Diffues Shader

{% highlight glsl%}
Shader "Custom/SnowShader" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        //New normal map texture
        _Bump ("Bump", 2D) = "bump" {}
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
 
        CGPROGRAM
        #pragma surface surf Lambert
 
        sampler2D _MainTex;
        //Must add a sample with the same name
        sampler2D _Bump;
 
        struct Input {
            float2 uv_MainTex;
            //Get the uv coordinates for the bump map
            float2 uv_Bump;
        };
 
        void surf (Input IN, inout SurfaceOutput o) {
             half4 c = tex2D (_MainTex, IN.uv_MainTex);
 
             //Extract the normal map information from the texture
             o.Normal = UnpackNormal(tex2D(_Bump, IN.uv_Bump);
 
             o.Albedo = c.rgb;
             o.Alpha = c.a;
        }
        ENDCG
    } 
    FallBack "Diffuse"
}
{% endhighlight %}

이것은 Unity가 자동으로 생성해주는 Bump 효과를 추가하는 shader입니다.

* 기본 "bump"(빈 normal map)를 사용한 2D 이미지인 *_Bump*라는 property를 정의합니다.
* sampler2D를 **같은 이름**으로 만듭니다(Shader Tutorial 1을 참고하시면 됩니다).
* Bump(**다시 동일한 이름으로**)에 사용할 uv 좌표를 받도록 *Input* 내부에 목록을 만듭니다.  
* *UnpackNormal* 함수를 호출하는 코드를 추가합니다. 이 함수는 normal map 텍스처를 가지고, normal로 그 결과를 변환합니다. 우리는 *tex2D*와 *_Bump* 변수, *Input* 구조체 내부의 uv 좌표를 사용하여 텍스처의 픽셀에 normal을 전달합니다.

이것으로 우리는 평범한 bumped shader를 만들었습니다.

# 2단계 - 눈 추가하기

이번 단계에서 우리는 픽셀의 normal이 눈이 내려오는 방향과 동일하게 만들어 주어야 합니다.

이를 위해서 *내적*을 사용할 것입니다. 두 단위 벡터 사이의 *내적*은 그 벡터들 사이 각도의 코사인 값과 같습니다. CG는 *내적*을 계산해주는 *dot* 함수를 제공합니다. *내적*의 결과는 두 벡터가 같은 방향을 가리키면 1, 반대 방향을 가리키면 -1이 됩니다. 그래서 우리는 구체적인 각도를 알 필요가 없습니다. 단지 픽셀의 normal과 눈이 내리는 방향 사이의 *내적* 결과만 있으면 됩니다. 

단위 벡터는 크기가 1인 벡터입니다. 그래서 x, y, z 각각 제곱의 합의 제곱근은 항상 1입니다. (1, 1, 1)을 단위 벡터로 착각해서는 안 됩니다. 두 벡터 사이의 각을 알아내기 위해서는 단위 벡터보다 크기가 큰 벡터는 scale을 통해서 단위 벡터로 크기를 변환해야 합니다.

우리는 shader에 사용할 properties를 정의하여야 합니다.

{% highlight glsl %}
Properties {
    _MainTex ("Base (RGB)", 2D) = "white" {}
    _Bump ("Bump", 2D) = "bump" {}
    _Snow ("Snow Level", Range(0,1) ) = 0
    _SnowColor ("Snow Color", Color) = (1.0,1.0,1.0,1.0)
    _SnowDirection ("Snow Direction", Vector) = (0,1,0)
    _SnowDepth ("Snow Depth", Range(0,0.3)) = 0.1
}
{% endhighlight %}

Properties에 추가된 것은 다음과 같습니다.

* *_Snow* 변수는 바위를 덮는 눈의 양입니다. 크기는 항상 0 ~ 1 사이입니다.
* *_SnowColor*는 흰색입니다.
* *_SnowDirection*은 눈이 내리는 방향입니다. 곧게 아래로 내리기 때문에 누적 벡터는 곧게 위를 향합니다. 
* *_SnowDepth_는 눈의 양입니다. 우리는 3단계에서 정점들을 수정하는데 이 변수를 사용할 것입니다. 크기는  0 ~ 0.3 사이입니다.

Shader Tutorial 1에서 언급한 것처럼 Properties에 맞게 변수의 이름도 잘 정해야합니다.

{% highlight glsl %}
sampler2D _MainTex;
sampler2D _Bump;
float _Snow;
float4 _SnowColor;
float4 _SnowDirection;
float _SnowDepth;
{% endhighlight %}

우리가 어떻게 모든 것들을, 텍스처 sampler를 제외하고, 다른 크기의 float처럼 다룰 수 있는지 알아두시오. Cg 부분은 CGPROGRAM과 ENDCG 사이입니다. Cg는 Unity shader 시스템인 ShaderLab을 사용하는 다른 shader 부분과는 독립적입니다. Properties 부분에 정의된 property들은 ShaderLab에 속하고, property들은 Cg와 연결되어야 합니다. 이것이 바로 property들의 이름을 똑같이 설정해야 하는 이유입니다. ShaderLab 컴파일러는 이름을 통해서 ShaderLab과 Cg을 연결합니다.

다음에 우리는 Input을 shader에 업데이트해야 합니다. normal map 텍스처는 픽셀의 normal을 수정합니다. 하지만 우리가 원하는 연출은 world 공간의 normal로 변환된 결과가 필요합니다. 그리고 우리는 변환된 결과를 눈이 내리는 방향과 비교할 수 있습니다.

이 과정을 처리하기 위해서 문서를 좀 읽어봅시다. 기본적으로 우리 shader에서 o.Normal를 쓰기를 원하기 때문에 우리는 Unity가 제공하는 INTERNAL_DATA을 얻어야 합니다. 그리고 그런 정보가 필요한 shader 프로그램 내에서 WorldNormalVector 함수를 호출하여야 합니다. 이런 것들을 Input 구조체 내에 넣어야 합니다.

{% highlight glsl %}
struct Input {
    float2 uv_MainTex;
    float2 uv_Bump;
    float3 worldNormal;
    INTERNAL_DATA
};
{% endhighlight %}

이제 드디어 shader 프로그램을 작성할 준비가 되었습니다.

{% highlight glsl %}   
void surf (Input IN, inout SurfaceOutput o) { 
    //Normal color of a pixel
    half4 c = tex2D (_MainTex, IN.uv_MainTex);
 
    //Get the normal from the bump map
    o.Normal = UnpackNormal (tex2D (_Bump, IN.uv_Bump));
 
    //Get the dot product of the real normal vector and our snow direction
    //and compare it to the snow level
    if(dot(WorldNormalVector(IN, o.Normal), _SnowDirection.xyz) > lerp(1,-1,_Snow))
    //If this should be snow pass on the snow color
        o.Albedo = _SnowColor.rgb;
    else
        o.Albedo = c.rgb;
    o.Alpha = 1;
}
{% endhighlight %}

이제 모든 마법이 펼쳐지는 *if*문에서 대해서 해부해보도록 하겠습니다.

* 우리는 두 벡터의 *내적* 결과를 얻을 것입니다. 하나의 벡터는 눈이 내리는 방향이고, 다른 하나는 픽셀의 normal에 사용될 벡터입니다. 이 픽셀의 normal은 이 점과 bump map의 world normal 조합입니다. 우리는 ㅜnormal을 Input 구조체의 INTERNAL_DATA와 bump map의 픽셀 normal을 사용해서 전달할 수 있는 *WorldNormalVecto*를 호출하여 필요한 normal을 얻을 수 있습니다. 이 *내적* 결과는 당연하지만 -1 ~ 1 사이의 입니다.
* 우리는 내적 결과를 lerp를 이용해서 비교합니다. 만약 Snow 수치가 0이면 비교 결과 1을 반환합니다. 만약 Snow 수치가 1이면 비교 결과 -1을 반환합니다(그렇게 되면 전체 바위가 눈으로 덮히게 됩니다). Snow 수치가 0 ~ 0.5 사이라면 결과는 이상하지 않을 것입니다. 적당한 수치가 설정이 되면 실제로 눈이 내리는 방향을 바라보는 표면만 눈으로 덮힐 것입니다.
* 만약 내적의 결과가 snow 수치를 lerp한 값도가 크면 snow color를 사용하고, 아니라면 텍스처 color를 사용합니다.

{% highlight glsl %}
Shader "Custom/SnowShader" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _Bump ("Bump", 2D) = "bump" {}
        _Snow ("Snow Level", Range(0,1) ) = 0
        _SnowColor ("Snow Color", Color) = (1.0,1.0,1.0,1.0)
        _SnowDirection ("Snow Direction", Vector) = (0,1,0)
        _SnowDepth ("Snow Depth", Range(0,3)) = 0.1
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
 
        CGPROGRAM
        #pragma surface surf Lambert
 
        sampler2D _MainTex;
        sampler2D _Bump;
        float _Snow;
        float4 _SnowColor;
        float4 _SnowDirection;
        float _SnowDepth; 
 
        struct Input {
             float2 uv_MainTex;
             float2 uv_Bump;
             INTERNAL_DATA
         };
 
         void surf (Input IN, inout SurfaceOutput o) { 
              //Normal color of a pixel
              half4 c = tex2D (_MainTex, IN.uv_MainTex);
 
              //Get the normal from the bump map
              o.Normal = UnpackNormal (tex2D (_Bump, IN.uv_Bump));
 
              //Get the dot product of the real normal vector and our snow direction
              //and compare it to the snow level
              if(dot(WorldNormalVector(IN, o.Normal), _SnowDirection.xyz)>=lerp(1,-1,_Snow))
              //If this should be snow pass on the snow color
                    o.Albedo = _SnowColor.rgb;
              else
                    o.Albedo = c.rgb;
              o.Alpha = 1;
         }
         ENDCG
    } 
    FallBack "Diffuse"
}
{% endhighlight %}

# 모델 변형하기

마지막 단계는 모델을 크게 만드는 일입니다. (완벽하지는 않지만) 눈이 내리는 방향으로 크기를 키워야 합니다.

이를 위해서 우리는 모델 정점을 수정하여야 합니다. 이는 surface shader에게 우리가 이런 기능을 수행할 함수를 작성한다는 것을 알려준다는 것을 의미합니다.

{% highlight glsl %}
#pragma surface surf Lambert vertex:vert
{% endhighlight %}

**pragma 끝부분에** 인자로 *vertex*를 추가하였습니다. 우리는 이 인자를 정점 함수 *vert*로 제공할 것입니다.

이제 정점 함수 vert를 살펴봅시다.

{% highlight glsl %}
void vert (inout appdata_full v) {
    //Convert the normal to world coordinates
    float4 sn = mul(UNITY_MATRIX_IT_MV, _SnowDirection);
    if(dot(v.normal, sn.xyz) >= lerp(1,-1, (_Snow*2)/3)){
         v.vertex.xyz += (sn.xyz + v.normal) * _SnowDepth * _Snow;
    }
}
{% endhighlight %}

먼저 인자를 전달하여야 합니다. 함수에 전달되는 데이터이고, 우리는 *appdata_full*(Unity에서 제공하는) 사용합니다. *appdata_full* 두 종류의 텍스처 좌표, normal, 정점 위치, 탄젠트값을 가집니다. 여러분은 Input 데이터 구조체를 사용하여 두번째 인자로 다른 추가적인 정보도 픽셀 함수에 전달할 수 있습니다. Input 구조체에 필요한 값을 추가하기만 하면 됩니다. 우리의 경우는 그럴 필요가 없습니다.

눈이 내리는 방향은 world 공간 내에 있습니다. 하지만 우리는 모델의 좌표계로 처리해야 합니다. 그래서 눈이 내리는 방향을 모델의 공간으로 변환해주어야 합니다. Unity에서 제공하는 행렬을 사용하여 곱해주면 변환이 이루어집니다.

이제 정점의 normal을 눈이 내리는 방향에 적용하였던 계산과 동일한 과정으로 구해줍니다. 하지만 우리는 눈이 적당히 덮도록 하기 위해서 snow 수치를 2/3으로 스케일했습니다.

정정의 normal과 눈이 내리는 방향을 더한 결과를 *_SnowDepth*와 *_Snow*를 곱하여 준 후, 이 결과를 다시 정점에 더하였습니다. 이 과정을 통해서 정점은 눈이 내리는 방향을 향해 움직이게 됩니다. 그리고 snow 수치가 증가할 수록 이 효과는 커집니다.

# Source Code

이제 전체 코드를 확인해보겠습니다.

{% highlight glsl %}
Shader "Custom/SnowShader" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _Bump ("Bump", 2D) = "bump" {}
        _Snow ("Snow Level", Range(0,1) ) = 0
        _SnowColor ("Snow Color", Color) = (1.0,1.0,1.0,1.0)
        _SnowDirection ("Snow Direction", Vector) = (0,1,0)
        _SnowDepth ("Snow Depth", Range(0,0.2)) = 0.1
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200
 
        CGPROGRAM
        #pragma surface surf Lambert vertex:vert 
 
        sampler2D _MainTex;
        sampler2D _Bump;
        float _Snow;
        float4 _SnowColor;
        float4 _SnowDirection;
        float _SnowDepth; 
 
        struct Input {
             float2 uv_MainTex;
             float2 uv_Bump;
             float3 worldNormal;
             INTERNAL_DATA
        };
 
        void vert (inout appdata_full v) {
             //Convert the normal to world coordinates
             float4 sn = mul(UNITY_MATRIX_IT_MV, _SnowDirection);
 
             if(dot(v.normal, sn.xyz) >= lerp(1,-1, (_Snow*2)/3)){
                  v.vertex.xyz += (sn.xyz + v.normal) * _SnowDepth * _Snow;
             }
        }
 
        void surf (Input IN, inout SurfaceOutput o) { 
             half4 c = tex2D (_MainTex, IN.uv_MainTex);
             o.Normal = UnpackNormal (tex2D (_Bump, IN.uv_Bump));
             if(dot(WorldNormalVector(IN, o.Normal), _SnowDirection.xyz)>=lerp(1,-1,_Snow))
                 o.Albedo = _SnowColor.rgb;
             else
                 o.Albedo = c.rgb;
             o.Alpha = 1;
        }
        ENDCG
    } 
    FallBack "Diffuse"
}

{% endhighlight %}

[참고자료]https://unitygem.wordpress.com/shader-part-2/
