---
layout: post
title: "Shader Tutorial 4"
author: "장문익"
date: 2016-07-17 03:00:00
excerpt: "Detail Texture Shader, Cubemap Reflaction Shader 등에 대해서 알아봅시다."
tags: [Shader, Rim Lighting Shader, Surface Shader, Unity]
comments: true
---

# 들어가며

Shader Tutorial 3을 이해했다면 오늘 다룰 내용의 기본은 갖추었다고 생각하시면 됩니다. 이번에는 Detail Texutrer Shader, Detail Texutre in Screens Space Shader, Cubemap Reflaction Shader에 대해서 다뤄볼 예정입니다. 특히 Cubemap Reflacion Shader는 이름에서 알 수 있듯이 반사와 관련된 내용입니다. 앞의 둘은 Texture의 읽고 색상을 처리하는 과정이므로 이미 익숙한 내용이기도 합니다.

# Detail Texture Shader

이름부터 뭔가 세밀하게 표현해줄 것 같은 쉐이더입니다. 지금까지 우리가 텍스처를 읽고 모델에 적용한 것은 두 가지였습니다. 하나는 가장 단순한 텍스처를 적용하기만 한 Diffuse Texture Shader였고, 다른 하나는 Normal 벡터 정보를 가지고 있는 텍스처를 읽고 적용하였던 Bumpmap Shader였습니다. 이 둘의 공통점은 텍스처를 읽고 모델의 색상에 적용하였던 점입니다. 우리는 아직까지 정점의 위치를 바꾸는 형태의 쉐이더는 접하지 않았습니다. Detail Texture Shader 또한 마찬가지입니다. 단순하게 세부 묘사를 위해 마련해둔 텍스처를 읽고 이를 색상값을 계산하는 연산에 추가하기만 하면 됩니다. 그러면 시작해보도록 하겠습니다. 먼저 익숙한 Bumpmap shader를 살펴보겠습니다.

{% highlight glsl%}
Shader "Custom/Bumpmap" {
	Properties {
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_BumpMap ("Bumpmap", 2D) = "Bump" {}    //  Normal을 저장하고 있는 텍스처
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		
		CGPROGRAM
		#pragma surface surf Lambert

		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0

		// Properties와 동일한 변수명으로 설정합니다.
		sampler2D _MainTex;
		sampler2D _BumpMap;

		// Vertex를 연산할 때 필요한 데이터를 정합니다.
		// surf 함수로 호출될 때마다 해당 정점의 관련 데이터가 연산에 적용됩니다.
		struct Input {
			float2 uv_MainTex;	// uv는 해당 텍스처의 텍스처 좌표를 의미합니다.
			float2 uv_BumpMap;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));    // 텍스처로부터 값을 읽은 후에 UnpackNormal
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

![Bumpmap](/assets/img/Bumpmap.png)

복습 겸 위 쉐이더 코드의 내용을 간단히 살펴보겠습니다. 먼저 Properties에서 두 개의 텍스처를 선언하였습니다. 이를 쉐이더 코드에 연결하기 위한 변수도 SubShader 내부에서 확인할 수 있습니다. 텍스처를 읽기 위한 정보를 전달하기 위한 Input 구조체도 보입니다. 그리고 텍스처를 읽어야 하기 때문에 uv 좌표를 전달하기 위한 float2 변수도 2개 선언되어 있습니다. 기본적인 준비는 여기까지 입니다. 그리고 surf 함수 내부에서는 메인텍스처를 tex2D 함수를 사용하여 색상값을 읽어 Albedo에 적용합니다. Bumpmap도 같은 방식으로 읽습니다. 다만 Shader Tutorial2에서 설명한 UnpackNormal 함수를 사용한다는 점도 확인하시기 바랍니다. 결국 Normal 벡터도 적용하였습니다. 그 결과는 SurfaceOutput 구조체에 저장되고 물체가 랜더링될 때 이 구조체가 참조됩니다. 끝입니다.

이제 본격적으로 Deatil Texture Shader로 넘어가보도록 하겠습니다. 간단하게 생각하도록 하겠습니다. Texture을 사용하여야 합니다. 그렇다면 Properties에 하나 추가하도록 하겠습니다. 그리고 이를 쉐이더 코드와 연결시키기 위한 변수도 하나 선언합니다. surf 함수에서 tex2D로 이를 읽기만 하면 끝입니다. 참 쉽네요. 완성된 코드는 다음과 같습니다.

{% highlight glsl%}
Shader "Custom/DetailTexture" {
	Properties {
      _MainTex ("Texture", 2D) = "white" {}	// 메인 텍스처
      _BumpMap ("Bumpmap", 2D) = "bump" {}	// 범프맵 텍스처
      _Detail ("Detail", 2D) = "gray" {}	// 세부 표현을 위한 텍스처
      _DetailIntensity("Detail Color Intensity", Range(0, 5)) = 2
    }
    SubShader {
      Tags { "RenderType" = "Opaque" }

      CGPROGRAM

      #pragma surface surf Lambert

      struct Input {
          float2 uv_MainTex;
          float2 uv_BumpMap;
          float2 uv_Detail;
      };

      sampler2D _MainTex;
      sampler2D _BumpMap;
      sampler2D _Detail;
      float _DetailIntensity;

      void surf (Input IN, inout SurfaceOutput o) {
          o.Albedo = tex2D (_MainTex, IN.uv_MainTex).rgb;
          o.Albedo *= tex2D (_Detail, IN.uv_Detail).rgb * _DetailIntensity;	// detailTexture을 읽어서 곱 연산
          o.Normal = UnpackNormal (tex2D (_BumpMap, IN.uv_BumpMap));
      }
      ENDCG
    } 
    Fallback "Diffuse"
}
{% endhighlight %}

![DetailTexture](/assets/img/DetailTexture.png)

_DetailIntensity를 수정하면 물체가 점점 밝아지고 결국 흰색으로 보일 것입니다. 이는 당연한 결과입니다. Detail Texture를 읽은 결과를 Albedo에 곱 연산하였기 때문입니다. 만약 Detail Texture의 색상값이 (0,0,0)이라면 Albedo 또한 (0,0,0)이 됩니다. 그러므로 이 쉐이더 코드는 완전하지는 완전하다고는 볼 수 없습니다. 그리고 디테일 텍스처를 예시처럼 검은색으로 만들지도 않기도 할 것입니다. 대게 Detail Texture는 피부와 물체의 재질을 표현하기 위해서 사용됩니다. 자세한 내용을 검색을 통해서 쉽게 확인할 수 있을 것입니다. 다음은 Detail Texture Shader를 스크린 공간으로 적용시켜보겠습니다. 스크린 공간에 적용시키면 체크 무늬가 구를 감싸는 것이 아니라 스크린을 기준으로 평면으로 적용됩니다. 

# Detail Texture Shader in Sreen Space 

이 쉐이더 코드는 텍스처를 물체의 uv로부터 적용하는 것이 아니라 스크린 공간의 uv로부터 적용한다는 점입니다. 좀더 자세하게 설명해보도록 하겠습니다. 일반적인 텍스처 적용 방식은 정점과 함께 저장된 uv 좌표를 가지고 텍스처로부터 RGBA와 같은 정보를 읽습니다. 그러면 정점에 해당하는 텍스처의 색상 정보가 적절하게 쉐이딩되어 랜더링됩니다. 스크린 공간에 텍스처를 적용하는 것은 이와는 조금 다릅니다. 정점 정보의 uv를 무시하고, 스크린 공간으로부터 uv를 추출한 후에 텍스처에 적용하여 색상값을 읽어와야 합니다. 다시 말해 모델 정점 정보로부터 uv를 읽으면 일반적인 텍스처 맵핑이되고, 이를 무시하고 정점의 스크린 좌표를 uv로 맵핑한 후, 이를 텍스처 맵핑에 사용하면 스크린 공간에 텍스처 맵핑을 한 결과를 얻을 수 있습니다. 코드를 보면서 자세한 설명을 하도록 하겠습니다.

{% highlight glsl%}
Shader "Custom/DetailTextureInScreenSpace" {
	Properties {
      _MainTex ("Texture", 2D) = "white" {}
      _Detail ("Detail", 2D) = "gray" {}
	  _DetailScaleX("Detail Scale X", Range(0, 5)) = 1	// 스크린 공간으로 맵핑할 때, 텍스처의 x 스케일값입니다. 증가하면 맵핑 결과가 가로로 늘어납니다.
	  _DetailScaleY("Detail Scale Y", Range(0, 5)) = 1	// 스크린 공간으로 맵핑할 때, 텍스처의 y 스케일값입니다. 증가하면 맵핑 결과가 세로로 늘어납니다.
    }
    SubShader {
      Tags { "RenderType" = "Opaque" }
      CGPROGRAM
      #pragma surface surf Lambert

      struct Input {
          float2 uv_MainTex;
          float4 screenPos;
      };

      sampler2D _MainTex;
      sampler2D _Detail;
	  float _DetailScaleX;
	  float _DetailScaleY;

      void surf (Input IN, inout SurfaceOutput o) {
          o.Albedo = tex2D (_MainTex, IN.uv_MainTex).rgb;
          float2 screenUV = IN.screenPos.xy / IN.screenPos.w;	// 스크린 해상도를 기본으로 uv를 계산합니다. w값은 가로 해상도입니다.
          screenUV *= float2(_DetailScaleX, _DetailScaleY);		// 유니티 에디터에서 설정한 Detail Scale을 적용합니다.
          o.Albedo *= tex2D (_Detail, screenUV).rgb * 2;
      }
      ENDCG
    } 
    Fallback "Diffuse"
}
{% endhighlight %} 

추가된 내용을 먼저 설명하겠습니다. Properties에 Detail Texture을 스케일링할 수 있도록 선언하였습니다. 만약 둘 다 0이라면 uv 좌표는 (0,0)으로 고정될 것입니다. 우리가 사용하는 Detail Texture의 (0,0)에 해당하는 색상값은 흰색입니다. 결국 Detail Texture는 모두 흰색으로만 적용될 것입니다. 우선은 이해하기 쉽도록 이 스케일 값이 (1,1)으로 가정하도록 하겠습니다. 그러면 물체가 스크린에 랜더링될 때의 좌표를 스크린의 가로 크기로 나누워 uv 좌표를 만들어 낼 수 있습니다. 만약 스크린의 해상도가 (16,9)라면 IN.screenPos.w는 16이 됩니다. 그리고 이 uv값으로 텍스처를 맵핑하면 결과적으로 3d 물체가 아니라 2d 스크린에 텍스처가 적용되는 결과를 얻을 수 있습니다. 결과는 다음과 같습니다. 물체에 텍스처가 적용되기는 하지만 구체에 입혀지는 것이 아니라 물체가 랜더링되는 스크린 공간에 텍스처가 입혀지게 됩니다. 물체 정점의 uv가 아니라 스크린으로부터 계산한 uv를 대신해서 적용했다는 것을 이해하시면 됩니다.

![DetailTextureInScreenSpace](/assets/img/DetailTextureInScreenSpace.png)

# Cubemap Reflection Shader

처음으로 반사가 등장하였습니다. 반사라는 용어를 사용하지만 사실은 맵핑의 한 종류입니다. 큐브로 구성된 텍스처를 물체에 적용하는 것입니다. 먼저 기본적인 Cubemap Reflection의 개념을 살펴보도록 하겠습니다.

![CubemapReflection](/asstes/img/cubemapReflection.jpg)
[출처](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Reflecting_Surfaces)

큐브맵은 스카이박스로 생각하시면 됩니다. 우리의 목표는 물체에 이 스카이박스의 이미지를 텍스처 맵핑하는 것입니다. 텍스처 맵핑이 목표이므로 필요한 것은 2가지입니다. 텍스처와 uv 좌표입니다. Detail Texture Shader에서는 정점 정보에 포함된 uv 좌표를 사용하였습니다. Detail Texture Shader in Screen Space에서는 정점이 랜더링될 스크린 공간으로부터 uv 좌표를 얻었습니다. 이번에는 조금 복잡한 과정을 거쳐서 uv 좌표를 얻어와야 합니다. 카메라 레이가 물체와 만나는 지점이 있습니다. 이 지점의 Normal 벡터를 통해서 반사 벡터를 구할 수 있습니다. 그리고 이 지점에서 반사 벡터 방향으로 레이를 만듭니다. 이 레이는 스카이박스와 만나게 됩니다. 이 만드는 지점의 색상값이 바로 물체의 맵핑될 텍스처의 색상값이 됩니다. 이 과정을 설명한 것이 위의 그림입니다. 그림으로 살펴보면 그렇게 복잡하지 않습니다. 그러나 내부적으로 본다면 카메라 레이와 물체가 만나는 지점을 검출해야 하고, 그 지점의 Normal 벡터를 통해 반사 벡터를 구해야 합니다. 이 지점으로부터 또 다시 반사 벡터 방향으로 만든 레이와 큐브맨이 만나는 지점을 검출해야 합니다. 상당히 복잡한 과정입니다. 하지만 이런 과정들은 쉐이더 코드로 작성할 필요는 없습니다. CG에서 제공하는 유용한 함수들을 사용할 수 있기 때문입니다. 이 복잡한 내용들은 매우 간단한 쉐이더 코드로 표현될 수 있습니다. 

가장 먼저 Cubemap을 준비해야 합니다. 큐브맵을 만드는 것은 어렵지 않습니다. 유니티 에디터에 큐브맵으로 사용할 텍스처를 추가하고, Inspector에서 Cubemap으로 설정해주면 자동으로 생성됩니다. 이렇게 생성된 Cubemap을 Propertise에 설정할 _CubeMap에 등록하기만 하면 됩니다. 코드를 살펴보도록 합시다.

{% highlight glsl%}
Shader "Custom/CubemapReflection" {
	Properties {
      _MainTex ("Texture", 2D) = "white" {}
      _Cube ("Cubemap", CUBE) = "" {}   // 사용할 큐브맵을 설정합니다.
	  _Blend ("Blend Offset", Range(0,1)) = 0.5 // 메인 텍스처와 큐브맵의 어느 정도 비율로 blend할지 설정합니다.
    }
    SubShader {
      Tags { "RenderType" = "Opaque" }

      CGPROGRAM

      #pragma surface surf Lambert

      struct Input {
          float2 uv_MainTex;
          float3 worldRefl; // 해당 정점에 카메라 레이가 반사되는 방향을 나타내는 벡터입니다.
      };

      sampler2D _MainTex;
      samplerCUBE _Cube;
	  float _Blend;

      void surf (Input IN, inout SurfaceOutput o) {
          // _Blend를 기준으로 메인 텍스처와 큐브맵의 색상값을 blend 처리하기 때문에 색상값이 1을 넘지 않습니다. Alpha Blend와 유사합니다.
          o.Albedo = tex2D (_MainTex, IN.uv_MainTex).rgb * _Blend;
		  o.Albedo += texCUBE (_Cube, IN.worldRefl).rgb * (1 - _Blend);
      }
      ENDCG
    } 
    Fallback "Diffuse"
}
{% endhighlight %} 

설명한 내용은 복잡했지만 쉐이더 코드는 매우 단순하다는 것을 알 수 있습니다. 이상하다 싶을 정도로 단순합니다. 이런 간결한 코드로 Cubemap Reflection을 적용할 수 이유는 Unity ShaderLab에서 제공하는 property 중에 CUBU가 존재하고, Input에 이미 float3 worldRefl이 존재하기 때문입니다. worldRefl은 카메라 레이가 해당 정점에서 Normal 벡터를 기준으로 반사되는 벡터를 World 공간으로 나타낸 벡터입니다. 이 벡터를 texCUBE 함수에 Cubemap과 함께 전달하면 우리가 찾고자 하는 픽셀의 색상값이 반환됩니다. 허무할 정도로 간단하게 처리되었습니다. 지금까지 두 개 이상의 텍스처로부터 색상값을 읽어 올때의 문제점은 색상값이 1을 넘는 경우에 대한 처리였습니다. 두 텍스처를 곱 연산 처리하기 때문에 1을 초과하는 값이 나와서 물체가 전체적으로 흰색이 되기도 했고, 색상값이 1보다 작은 경우는 제곱 연산을 하면 어두워지는 문제가 있었습니다. 이를 해결하기 위해서 적당한 수치를 설정해주어야 했습니다. 하지만 위의 쉐이더 코드를 사용하면 색상값이 1을 넘지 않게 됩니다. 그리고 기준이 되는 수치를 조절해서 메인 텍스처가 더 잘 보이게 혹은 반대로 큐브맵이 더 잘 보이게 등의 처리도 가능합니다.




