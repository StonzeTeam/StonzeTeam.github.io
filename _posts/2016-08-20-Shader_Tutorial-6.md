---
layout: post
title: "Shader Tutorial 6"
author: "장문익"
date: 2016-08-21 03:00:00
excerpt: "특정 영역을 랜더링하지 않는 쉐이더를 작성해봅시다"
tags: [Shader, Clip, Texture Scrolling, Surface Shader, Unity]
comments: true
---

# 들어가며 

이번에는 쉐이더를 활용하여 특정 범위는 랜더링되지 않도록 하는 효과를 만들어 보도록 하겠습니다. 이 쉐이더를 좀더 발전시키면 물체의 원하는 부분을 뚫어버리는 효과를 만들 수 있을 것 같다는 생각이 듭니다. 벽을 뚫어버리고 벽 뒤에 대상을 보이도록 할 수도 있을 것 같습니다. 이번에는 쉐이더를 이용해서 원형의 구명을 뚫어보도록 하겠습니다. 그리고 Tutorial 5에서 사용해보았던 ShaderLab internal value 중 하나인 _SinTime을 이용해서 이 원형의 구멍을 커졌다 작아졌다 하도록 해보겠습니다.

# clip

우리가 만들고자 하는 쉐이더의 핵심이 되는 함수입니다. 엔비디아 홈페이지의 있는 개발자 공간에서 이 함수를 살펴보면 간단한 설명을 확인할 수 있습니다. ([clip](http://http.developer.nvidia.com/Cg/clip.html)) 이 함수는 조건에 따라 픽셀 결과값을 제거합니다. 조건은 주어진 벡터, 스칼라의 어떤 구성요소라도 음수인 경우를 뜻합니다. 이를 좀더 살펴보면 다음과 같다고 합니다.

{% highlight cg%}
void clip(float4 x)
{
	if (any(x < 0))
		discard;
}
{% endhighlight %}

간단합니다. 앞에서 짧게 설명한 내용 그대로 입니다. 인자로 전달된 것들의 어떤 요소라도 0보다 작으면 버립니다. 우리는 이 조건을 잘 활용해서 어떤 경우에 랜더링에서 제외할지만 결정하면 됩니다.

# _SinTime

우리가 원하는 효과는 원형의 구멍을 뚫는 것이고, 이 구멍은 커졌다 작아졌다 해야합니다. 직관적으로 떠오르는 것은 sin 값일 것입니다. 친절하게도 ShaderLab internal value에는 이미 이런 값을 제공하고 있습니다. 시간의 흐름에 따라서 sin값을 계산해줍니다. 시간이 흐르는 동안 이 값은 -1과 1사이를 sin함수 값을 계산하여 줍니다. 이렇게 계산된 값이 바로 _SinTime입니다. sin이 있는 만큼 cos 또한 존재합니다. 필요한 경우에 맞게 선택하면 되겠습니다. 일정한 주기로 제어하고 싶은 무언가가 있다면 이 값을 활용하면 된다는 사실만 기억하면 좋을 것 같습니다. 사용하는 방법은 Tutorial 5에서 사용한 _Time과 동일합니다. 구체적은 사항은 메뉴얼을 참고하시기 바랍니다. ([_SinTime](http://docs.unity3d.com/462/Documentation/Manual/SL-BuiltinValues.html))

# 아이디어 적용하기

이제 clip과  _SinTime을 활용하여 '들어가며'에서 계획하였던 것을 만들어보도록 하겠습니다. 기본적인 아이디어는 다음과 같습니다. 평범하게 스크롤링되는 오브젝트가 있습니다. 이 오브젝트 앞에는 또다른 오브젝트가 있고, 이 오브젝트에 우리는 구멍을 뚫도록 할 것입니다. 구멍은 우리가 지정한 반지름에 따르도록 합니다. 그리고 _SinTime을 사용하여 따라 반지름이 커졌다 작아졌다 하도록 적용합니다. 이 때, _SinTime 값은 음수가 될 수 있습니다. 음수인 경우에 clip 함수를 사용하게 되면 해당 픽셀은 랜더링되지 않습니다. 결국 시간에 따라 계산된 sin값이 존재하고, 이를 활용하여 시간에 따라 게산된 반지름을 얻을 수 있습니다. 샘플링되는 픽셀의 위치가 반지름 내부이면 clip되도록 처리해줍니다.

먼저, Property로 반지름을 받도록 합니다.

{% highlight glsl%}
Properties {
		_MainTex("texture", 2D) = "white" {}
		_Offset("Radius", Range(0, 1)) = 0.7	// 반지름
	}
{% endhighlight %}

물론 Properties를 Shader 코드와 바인딩하는 것 또한 필수적으로 요구됩니다. 

이제 shader에 사용할 surf() 함수만 작성하면 됩니다. 적용할 내용은 단순합니다.

* 시간에 따라 반지름을 커졌다 작아졌다 하게 만듭니다(_SinTime을 이용).
* 반지름 내부에 있는 샘플링 지점은 clip되도록 처리합니다.

{% highlight glsl%}
void surf (Input IN, inout SurfaceOutput o) {
	o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb; // 텍스처로부터 색상을 얻습니다.

	// _SinTime는 float4이므로 x,y,z,a 혹은 [0]~[4]와 같은 방식으로 요소에 접근할 수 있습니다.
	float absT = abs(_SinTime.a) * _Offset;

	// 노멀맵의 중심지점(0.5, 0.5)를 기준으로 현재 지점의 거리를 게산합니다.
	float targetD = (IN.uv_MainTex.x - 0.5) * (IN.uv_MainTex.x - 0.5) + (IN.uv_MainTex.y - 0.5) * (IN.uv_MainTex.y - 0.5);

	clip (targetD - absT * absT);	// 샘플링 지점의 거리가 우리가 지정한 값보다 짧은 경우, 즉 내부에 있으면 음수가 되어 자동으로 clip 됩니다.
}
{% endhighlight %}

surf 함수를 살펴보도록 하겠습니다. 주석으로도 충분하긴 하지만 좀더 살펴보도록 하겠습니다. Input으로 전달되는 정점 정보에서 tex2D 함수로 색상값을 읽어옵니다. 기본적인 텍스쳐 적용입니다. 그 다음은 _SinTime을 사용합니다. 유니티 메뉴얼을 찾아보면 _SinTime은 float4 자료형이라는 것을 알 수 있습니다. 유니티에서는 float4와 같은 자료형은 x, y, z, a 혹은 [0]과 같은 인덱스 방식으로 접근할 수 있도록 되어있습니다. 이와 관련된 내용은 Tutorial 1에 언급되어 있습니다. 이 예시에서 _SinTime.a를 사용하여 t를 사용합니다. 참고로 _SinTime.x는 t/8이기 때문에 _SinTime.a에 비해서 변화량이 작습니다. 시간에 따라서 주기적으로 변화하는 반지름은 만들었습니다.

다음은 샘플링하는 정점이 오브젝트 중점으로부터 얼마나 떨어져있는지 계산합니다. 

마지막으로 이 값을 비교합니다. Tutorial 4에서도 언급한 적이 있습니다. Shader에서 if와 같은 분기는 성능에 나쁜 영향을 미칩니다. 정적으로 정해진 계산 과정으로 대상 정점 정보들을 처리하는 것이 가장 유리합니다. 하지만 if와 같은 분기를 사용하게 되면 연산 과정을 다시 적용되어야 합니다. 이는 CPU와 GPU의 처리 과정이 다르기 때문입니다. shader에 사용되는 함수들은 대부분 이런 점을 고려되어 만들어졌습니다. clip 함수도 마찬가지 입니다. 양수, 음수에 상관없이 동일한 연산으로 처리되기 때문에 분기가 발생되지 않도록 작성되어 있는 코드입니다. 만약 이를 우리가 직접적으로 제어한다면 양수인 경우, 픽셀값을 적용하고, 음수일 경우 픽셀값을 적용하지 않는 분기를 작성해야만 합니다. 이는 성능에 좋지 않은 영향을 미치게 됩니다([쉐이더 코드 내부에서 사용된 if else 구문이 성능에 안 좋은 영향을 미치는 이유](http://answers.unity3d.com/questions/442688/shader-if-else-performance.html)).

결국, 샘플링되는 지점이 텍스처 좌표계를 기준으로 거리가 반지름보다 작다면 clip되게 되고, 우리는 구멍이 뚫린 결과를 확인할 수 있게 됩니다. 

# 완선된 오브젝트 구멍 뚫는 쉐이더 코드

{% highlight glsl%}
Shader "Custom/SliceViaWorldSpace" {
	Properties {
		_MainTex("texture", 2D) = "white" {}
		_Offset("Radius", Range(0, 1)) = 0.5
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		Cull Off
		
		CGPROGRAM
		#pragma surface surf Lambert
		#pragma target 3.0

		struct Input {
			float2 uv_MainTex;
			float3 worldPos;
		};

		sampler2D _MainTex;
		float _Offset; 

		void surf (Input IN, inout SurfaceOutput o) {
			o.Albedo = tex2D(_MainTex,IN.uv_MainTex).rgb; // 텍스처로부터 색상을 얻습니다.

			float absT = abs(_SinTime.a) * _Offset;	// _SinTime는 float4이므로 x,y,z,a 혹은 [0]~[4]와 같은 방식으로 요소에 접근할 수 있습니다. 
			// 노멀맵의 중심지점(0.5, 0.5)를 기준으로 현재 지점의 거리를 게산합니다.
			float targetD = (IN.uv_MainTex.x - 0.5) * (IN.uv_MainTex.x - 0.5) + (IN.uv_MainTex.y - 0.5) * (IN.uv_MainTex.y - 0.5);

			clip (targetD - absT * absT);	// 샘플링 지점의 거리가 우리가 지정한 값보다 짧은 경우, 즉 내부에 있으면 음수가 되어 자동으로 clip 됩니다.
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

# 마무리하며

지금까지 살펴본 내용은 다음과 같습니다.

* Shader Built-in Values -  _SinTime
* clip 함수의 기능
* 쉐이더 코드 내에서 분기문이 성능에 미치는 영향

특정 픽셀에 대한 연산을 랜더링에 제외하는 것이 가능하다는 것을 알 수 있었습니다. 그리고 제외되면 당연히 z 버퍼를 기준으로 다음 픽셀이 랜더링되게 된다는 사실도 알 수 있었습니다.