---
layout: post
title: "Shader Tutorial 5"
author: "장문익"
date: 2016-08-14 03:00:00
excerpt: "UV 좌표를 이용한 텍스처 스크로링"
tags: [Shader, Texture Scrolling, Surface Shader, Unity]
comments: true
---

# 들어가며

지금까지는 정적으로 표현되는 쉐이더에 대해서 알아보았습니다. 이번에는 단순하지만 사용하기 좋은 텍스처 스크롤링에 대해서 알아보려고 합니다. 기능은 매우 단순합니다. 텍스처가 상하좌우로 움직이도록 만들 것입니다.

# 텍스처 맵핑과 스크롤링에 대한 기본적인 아이디어

기본적인 아이디어를 먼저 떠올려봅시다. 텍스처를 스크콜링하기 위해서는 당연히 텍스처가 적용되어야 합니다. 텍스처 적용은 Shader Tutorial 2, 3을 통해서 반복적으로 살펴보았으므로 어렵지 않을 것입니다. 텍스처를 적용할 때 가장 중요한 것은 대상이 되는 텍스처와 텍스처의 컬러값을 읽을 때 참조할 uv 좌표입니다. uv 좌표에 따라 읽어진 컬러값이 우리가 지정한 대상에 적용됩니다. 만약 uv 좌표를 변경하면 어떤 일이 생길지 생각해보도록 하겠습니다. 다음은 이미지는 우리가 사용할 테스트 텍스처입니다.

![Test Texture](/assets/img/test_texture.png)

uv 좌표가 적용되는 결과를 좀더 직관적으로 파악하기 위해서 이미지에 표시를 해두었습니다. 우리는 이 텍스처를 Quad에 적용할 예정입니다. Plane에 적용하면 결과가 다르게 나올 수도 있으므로 Quad를 기준으로 알아보도록 하겠습니다. 텍스처 맵핑의 기본적인 이해는 다음 이미지로 대신하도록 하겠습니다. 

![TextureMapping](/assets/img/TextureMapping.jpg)

이미 알고 있듯이 uv 좌표는 0에서 1 사이의 구간으로 이루어진 텍스처 좌표계입니다. uv 좌표를 통해서 텍스처의 컬러값을 읽게 되고, 이 컬러값을 오브젝트에 지정된 위치에 적용하는 것이 바로 텍스처 맵핑입니다.

다시 Test Texture을 보도록 하겠습니다. 만약 uv 좌표 (0,0) 대신 (0.75,0.25)을 입력하면 어떤 결과가 나올까요? (0,0)으로 컬러값을 읽었다면 녹색에 해당하는 컬러값이 읽혀졌을 것입나다. 하지만 이를 대신하여 (0.75,0.25)로 컬러값을 읽으므로 파란색에 해당하는 컬러값이 읽어지게 됩니다. 이것이 텍스처를 스크롤링하는 가장 기본적인 아이디어가 됩니다. 우리는 지금까지 uv 좌표을 수정한적이 없기 때문에 텍스처는 고정된 것처럼 랜더링되었던 것입니다. 하지만 uv 좌표를 매 프레임 변경하게되면 어떻게 될까요? 예를 들어, uv 좌표 (0,0)을 대신하여 매 프레임 u,v값이 (0.1,0.1)만큼 증가한다고 생각해보시기 바랍니다. 원래는 녹색이 랜더링되어야 하는 픽셀이 (0.1,0.1)만큼 증가하는 컬러값으로 변경될 것입니다. 3프레임 후에는 녹색이 아닌 노란색 컬러값이 적용될 것입니다. 스크린 상의 하나의 픽셀 지점에 색상값이 변화되었습니다. 이를 텍스처 전체로 확장하여 생각하도록 하겠습니다. 결국 텍스처가 매 프레임 (-0.1,-0.1)만큼 이동하는 것과 동일한 결과를 가져올 것입니다. 우리는 랜더링하고자 하는 대상의 위치는 고정 시켜둔 상태에서 텍스처를 맵핑하는 uv 좌표를 수정하여 마치 텍스처가 움직이는 것과 같은 효과를 얻을 수 있게 되었습니다.

# Texture Scrolling Shader

 이제 위에서 살펴보았던 기본적인 아이디어를 쉐이더 코드로 옮기는 일만 남았습니다. 먼저 기본이 되는 쉐이더를 보도록 하겠습니다.

{% highlight glsl%}
Shader "Example/Diffuse Texture" {
    Properties {
      _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader {
      Tags { "RenderType" = "Opaque" }
      CGPROGRAM
      #pragma surface surf Lambert
      struct Input {
          float2 uv_MainTex;
      };
      sampler2D _MainTex;
      void surf (Input IN, inout SurfaceOutput o) {
          o.Albedo = tex2D (_MainTex, IN.uv_MainTex).rgb;
      }
      ENDCG
    } 
    Fallback "Diffuse"
}
{% endhighlight %}

텍스처를 읽어서 적용하는 가장 기본적인 쉐이더 코드입니다. 우리는 이 코드를 기본적인 아이디어를 적용할 수 있도록 수정할 것입니다. 스크롤하는 속도를 x축 방향과 y축 방향 각각 적용하기 위한 property를 설정하겠습니다.  

{% highlight glsl%}
Properties {
	_MainTex ("Texture", 2D) = "white" {}
	_ScrollSpeedX ("X Scroll Speed", Float) = 0.2
	_ScrollSpeedY ("Y Scroll Speed", Float) = 0.2
}
{% endhighlight %}

이제 property를 Unity Shaderlab과 바인딩하도록 하기 위해 변수를 설정하도록 합니다.

{% highlight glsl%}
sampler2D _MainTex;
fixed _ScrollSpeedX;
fixed _ScrollSpeedY;
{% endhighlight %}

다음은 텍스처를 스크롤하는 기능을 surf() 함수에 추가할 것입니다. 매 프레임 지날때마다 uv 좌표값이 증가하기만 하면 됩니다. 이 내용이 적용되는 부분은 당연히 tex2D() 함수입니다. 이 함수의 두 번째 인자가 바로 uv 좌표를 전달하는데 사용되기 때문입니다. 그래서 우리는 이 두 번째 인자를 매 프레임마다 값을 증가시켜서 전달해주면 됩니다.

{% highlight glsl%}
void surf (Input IN, inout SurfaceOutput o) {
	fixed2 scrolledUV = In.uv_MainTex;	// 원래의 uv 좌표
	scrolledUV += fixed2(_ScrollSpeedX, _ScrollSpeedY);	// 지정한 속도만큼 uv 좌표를 변경

	o.Albedo = tex2D (_MainTex, scrolledUV).rgb;	// 원래의 uv 좌표되신 수정된 uv 좌표 적용
}
{% endhighlight %}

모두가 예상한 코드가 만들어 졌습니다. 쉐이더의 이름까지 적용한 전체 코드를 보도록 하겠습니다.

{% highlight glsl%}
Shader "Custom/ScrollTexture" {
	Properties {
		_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_ScrollXSpeed("X Scroll Speed", Float) = 0.01
		_ScrollYSpeed("Y Scroll Speed", Float) = 0.01

	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		#pragma surface surf Lambert

		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0

		sampler2D _MainTex;
		fixed4 _MainTint;
		fixed _ScrollXSpeed;
		fixed _ScrollYSpeed;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			fixed2 scrolledUV = IN.uv_MainTex;
			scrolledUV += fixed2(_ScrollXSpeed, _ScrollYSpeed);

			// Albedo comes from a texture tinted by color
			fixed4 c = tex2D (_MainTex, scrolledUV);
			o.Albedo = c.rgb * _MainTint;
			o.Alpha = c.a;
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

위의 쉐이더의 코드를 실행하면 어떤 결과가 나올지 예상해보시기 바랍니다. _ScrollXSpeed, _ScrollYSpeed만큼 uv 좌표가 매 프레임 증가하기 때문에 마치 텍스처가 7시 방향 근처로 스크롤링 될 것입니다. 실제로 실행시켜 보시기 바랍니다. 하지만 실행 결과는 우리가 예상한 것과는 다른 것을 알 수 있습니다. 텍스처가 매 프레임 스크롤링 되지 않고, property에 지정한 만큼만 이동하고 정지된 상태를 유지한다는 것을 확인할 수 있습니다. 분명히 scrolledUV 값을 증가했음에도 불구하고 텍스처는 스크롤되지 않습니다. 왜 이런 일이 발생하는지 생각해보도록 하겠습니다.

# Unity에서 오브젝트를 랜더링하는 과정

![Shader Theory](/assets/img/shader-theory.png)

위의 이미지에서 확인할 수 있듯이 유니티에서 3D 모델은 정점으로 이루어져 있습니다. 그리고 각각의 정점은 *normal*, 색상, uv 정보 등으로 이루어져 있습니다. 이런 정보로 이루어진 3D 모델이 유니티 환경에서 저절로 랜더링되는 것은 아닙니다. 모델은 *material* 없이는 랜더링되지 않습니다. materail은 *shader*와 *property*를 가집니다. 각각의 material이 동일한 shader를 공유할 수도 있습니다. material을 shader를 사용해서 모델을 랜더링하게 됩니다. 우리가 지금 알아보고 있는 *surface shader*는 이런 쉐이더 간략하게 하여 개발자들이 사용하기 편하게 만든 것입니다. 다음 3D 모델이 surface shader에 어떻게 정보를 전달하고 랜더링되는 지에 대한 간략한 설명을 보여줍니다.

![Surface Shader](/assets/img/Surface-shader.png)

3D 모델은 먼저 기하정보를 수정할 수 있는 함수를 전달합니다. 이 함수는 정점을 수정할 수 있습니다. 그리고 수정된 정보는 "겉모습"을 정의하는 함수인 surf() 전달하고, 이를 *Input*을 통해 연산에 필요한 정보를 전달합니다. 그리고 연산 결과는 *SurfaceOutput*으로 다음 과정으로 전달되어 최종적으로 *Lighting* 연산을 거치게 됩니다. 최종적인 결과는 각각 픽셀에 그려질 RGBA로 표현되는 색생값입니다.

우리가 작성한 surface shader는 3D 모델로부터 전달되는 정점 정보를 처리하는 함수의 모임으로 볼 수 있습니다. 대량으로 전달되는 정점 정보를 지정된 쉐이더 코드에서 정의한대로 GPU의 강력한 연산처리를 병렬적으로 실행하여 연산합니다. 각각의 정점 정보는 다른 정점 정보와는 완전히 무관하고 독립적으로 처리됩니다. 그리고 한 번에 전달되는 정점 정보들은 쉐이더에 의해 동일한 연산 과정을 거치게 됩니다. 그렇다면 다음 프레임에 쉐이더는 어떻게 적용될까요? 고민할 것 없습니다. 예상하셨들이 이전 프레임과 동일한 연산을 거치게 됩니다. 쉐이더에서 지정한 연산을 그대로 계산할 것입니다. 이 시점에서 바로 위에서 작성한 surf() 함수를 다시 살펴보도록 합시다.

{% highlight glsl%}
void surf (Input IN, inout SurfaceOutput o) {
	fixed2 scrolledUV = IN.uv_MainTex;
	scrolledUV += fixed2(_ScrollXSpeed, _ScrollYSpeed);

	// Albedo comes from a texture tinted by color
	fixed4 c = tex2D (_MainTex, scrolledUV);
	o.Albedo = c.rgb * _MainTint;
	o.Alpha = c.a;
}
{% endhighlight %}

매 프레임에 랜더링하고자 하는 3D 모델을 구성하는 정점 정보는 이 함수에 의해 연산됩니다. 여기서 주의할 점은 property로 정의한 항목과 이를 *ShaderLab*에 바인딩하기 위한 변수는 모두 클래스의 멤버변수처럼 사용해서는 안 된다는 것입니다. 쉐이더 코드의 내용들은 정점 정보를 연산하기 위해 정의된 고정적인 함수입니다. 쉐이더 코드 내에서 정의된 데이터 공간을 활용하게 클래스의 멤버변수처럼 데이터를 수정하는 식의 접근은 옳지 못한 방법입니다. 사실 위의 surf() 함수는 로컬 변수를 활용하였기 때문에 이와 같은 문제가 발생하였지만, Property를 이용해서 변수화하여도 결과는 마찬가지입니다. 중요한 것은 바로 Input으로 전달되는 정보들은 모두 독립적이라는 점이고, Input 데이터를 갱신하거나 수정하는 방식이 아니라 처리된 정보를 전달하기만 한다는 점입니다. 그렇다면 우리가 원하는 시간이 지날수록 증가되는 uv 좌표를 어떻게 표현할 수 있을까요?

# ShaderLab built-in values

앞에서 언급하였듯이 쉐이더 코드에서 클래스의 멤버 변수처럼 사용할 수 있는 변수를 지정하는 것은 바람직한 접근 방식이 아닙니다. 하지만 우리의 목표는 누적해서 증가하는 변수가 있다면 쉽게 해결할 수 있습니다. 그래서 ShaderLab에서는 이럴 때 사용하기 좋은 기능을 제공합니다. 바로 *ShaderLab built-in values*입니다. 이는 쉐이더 코드 내에서 제공하는 유용한 변수들입니다. 이 값들의 자세한 항목들은 검색을 통해서 쉽게 찾아볼 수 있을 것입니다. 이번에 우리가 사용할 값은 _Time입니다. 이 값은 쉐이더 내부에서 무언가를 에니메이션 적용할 때 유용한 값이라고 문서에 나와있습니다. 내용은 간단합니다. ShaderLab에서 제공하는 현재 시간에 해당하는 값입니다. 시스템에서 제공하는 시간 정보라고 이해햐셔도 무방할 것 같습니다. 여하튼 이 _Time은 시간이 지날수록, 프레임이 반복될 수록 값이 커집니다. 우리가 사용하기 좋은 값을 찾았습니다. 스크롤 속도와 _Time을 사용하면 누적해서 증가하는 값을 사용할 수 있을 것 같습니다. 그리고 이 값을 uv 값으로 전달하면 텍스처는 스크롤될 것입니다. 여기서 주의할 점은 유니티 엔진에서 이미지를 *Import*할 때, *wrap mode*를 *Clamp*가 아닌 *Repeat*로 설정해야합니다. 그래야 0에서 1 사이를 초과한 uv 좌표를 정상적으로 적용해줍니다. Clamp로 설정되면 1을 초과한 uv 값은 1로 처리됩니다. 그럼 이제 _Time을 적용한 스크롤 코드를 보도록 하겠습니다.

{% highlight glsl%}
void surf (Input IN, inout SurfaceOutput o) {
	fixed2 scrolledUV = IN.uv_MainTex;

	fixed scrolledUVX =  _ScrollXSpeed * _Time;
	fixed scrolledUVY = _ScrollYSpeed * _Time;

	scrolledUV += fixed2(scrolledUVX, scrolledUVY);

	// Albedo comes from a texture tinted by color
	fixed4 c = tex2D (_MainTex, scrolledUV);
	o.Albedo = c.rgb * _MainTint;
	o.Alpha = c.a;
}
{% endhighlight %}

크게 변경된 내용은 아닙니다. 매 프레임마다 증가하는 _Time을 적용하여 텍스처를 읽을 uv 좌표값을 증가시켜주는 것이 전부입니다. uv 좌표 값이 증가하므로 프레임이 반복될수록 텍스처는 왼쪽으로 스크롤될 것입니다.

# 완성된 Scroll Texture Shader

{% highlight glsl%}
Shader "Custom/ScrollTexture" {
	Properties {
		_MainTint ("Diffuse Tint", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_ScrollXSpeed("X Scroll Speed", Float) = 0.5
		_ScrollYSpeed("Y Scroll Speed", Float) = 0.5

	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200
		
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		#pragma surface surf Lambert

		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0

		sampler2D _MainTex;
		fixed4 _MainTint;
		fixed _ScrollXSpeed;
		fixed _ScrollYSpeed;

		struct Input {
			float2 uv_MainTex;
		};

		void surf (Input IN, inout SurfaceOutput o) {
			fixed2 scrolledUV = IN.uv_MainTex;

			fixed scrolledUVX =  _ScrollXSpeed * _Time;
			fixed scrolledUVY = _ScrollYSpeed * _Time;

			scrolledUV += fixed2(scrolledUVX, scrolledUVY);

			// Albedo comes from a texture tinted by color
			fixed4 c = tex2D (_MainTex, scrolledUV);
			o.Albedo = c.rgb * _MainTint;
			o.Alpha = c.a;
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

# 마무리하며

지금까지 살펴본 내용은 다음과 같습니다.

* 텍스처 맵핑에 대한 기본적인 개념
* 유니티 엔젠에서 3D 모델을 랜더링 하는 과정에 대한 간략한 소개
* ShaderLab built-in values
* uv 좌표를 수정하여 구현한 Scroll Texture Shader

스크롤 기능에 대한 아이디어는 다양한 곳에서 적용하여 사용할 수 있습니다. 배경에 적용할 수 있고, 흘러가는 물을 표현할 때도 요긴하게 사용할 수 있습니다. 상황에 맞게 변형하여 사용하도록 합시다. 
