---
layout: post
title: "Shader Tutorial 3"
author: "장문익"
date: 2016-06-19 03:00:00
excerpt: "Shader Tutorial 1,2에서 배운 내용을 정리하고, Rim Lighting을 만들어 봅시다."
tags: [Shader, Rim Lighting Shader, Surface Shader, Unity]
comments: true
---

Shader Tutorial 1,2에서 다루지 못했던 부분들을 보충하고, 이를 바탕으로 간단한 Rim Lighting Shader를 만들어 봅시다.

# 들어가며

스터디 중에 가장 많이 들어왔던 질문들에 대해서 좀더 자세히 알아보도록 하겠습니다.

* SubShader: 유니티 쉐이더는 기본적으로 SubShader 목록으로 구성됩니다. 즉, 하나의 쉐이더에 다수의 SubShader가 존재할 수 있다는 의미입니다. 이렇게 다수의 SubShader가 존재하는 이뉴는 뭘까요? 그리고 다수의 SubShader는 모두 실행될까요? 사실 SubShader가 다수 존재하더라고 쉐이더 코드 내에서 실행되는 SubShader는 하나입니다. 이는 곧 여러 쉐이더 중에서 하나의 쉐이더를 선택한다는 의미이기도 합니다. 그렇다면 SubShader를 선택하는 기준도 존재할 것입니다. 유니티 엔진은 SubShader는 기본적으로 위에서부터 차례대로 shader를 훑어나가게 되어있습니다. 만약 디바이스에서 실행 가능한 SubShader를 찾게 되면, 나머지 Shader는 무시되게 됩니다. 이는 멀티플랫폼을 지향하는 유니티 엔진의 특성이라고 생각하시면 됩니다. 다양한 환경에서 그 환경에 맞는 최적의 쉐이더를 실행하기 위함입니다. 

* Fallback: "이 하드웨어에서 실행할 수 있는 하위 쉐이더가 없는 경우, 다른 쉐이더에서 시도한다."는 것을 의미합니다. 만약 작성한 SubShader를 하드웨어가 지원하지 않을 때, Fallback으로 설정해둔 쉐이더를 실행하게 됩니다. 모바일 디바이스는 매우 다양하며, 각 하드웨어마다 지원하는 쉐이더의 범위도 다를 것입니다. 이런 경우를 대비하여 최악의 경우에도 쉐이더 실행을 보장하기 위한 설정이라고 볼 수 있습니다. 그런 만큼 단순하고 기본적인 쉐이더가 설정됩니다. 유니티에서 제공하는 예제의 대부분은 Fallback "Diffuse"로 설정되어 있습니다. 가장 단순한 만큼 거의 모든 디바이스에서 동작이 보장됩니다.

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

유니티에서 제공하는 텍스처를 적용한 diffuse shader입니다. 가장 아래에 Fallback "Diffuse"가 선언될 것을 확인할 수 있을 것입니다. SubShader로 지정한 내용을 디바이스가 실행할 수 없는 경우, 앞에서 언급했듯이 가장 단순하고 기본적이라고 할 수 있는 Diffuse가 대신해서 실행됩니다. '이도저도 안되는 상황이라면 이걸 실행해라.'라고 생각해되 되겠습니다.

* Property: 유니티 에디터의 inspector 기능과 관련이 있습니다. 보통 shader를 material에 추가하여 사용하게 되는데, 이때 property로 설정한 것들은 유니티 에디터에서 직접 값을 수정할 수 있게 됩니다. 위의 쉐이더 코드에서 _MainTex ("Texture", 2D) = "white" {} 로 설정한 부분이 있습니다. 이 쉐이더를 적용한 material의 inspector를 유니티 에디터 상에서 살펴보면 다음과 같은 모습을 볼 수 있습니다.

![_MainTexture](/assets/img/_MainTextureInspector.png)

Property에서 "Texture"라는 이름으로 2D 텍스처를 설정할 수 있도록 해두었고, 이를 이제 Inspector를 통해서 확인할 수 있습니다. 물론 Inspector 상에서 수정도 가능합니다.
Property 설정은 쉐이더 코드의 sampler2D _MainTex와 깊은 관련이 있습니다. Property 설정은 어디까지나 Inspector 상에서 이루어지는 유니티 엔진의 고유한 기능입니다. 정확히는 Unity ShaderLab에서 제공하는 기능입니다. 이를 연결하는 과정에 대한 자세한 설명은 Shader Tutorial 1에서 확인할 수 있습니다. 여기서는 간단하게 Property와 이름을 동일하게 변수명을 정해야 한다는 정도로만 언급하고 넘어가도록 하겠습니다.

 * Input: Property로 설정한 텍스처를 쉐이더 코드에서 사용할 수 있도록 연결도 하였는데, 뜬금없이 Input 구조체가 등장해서 이상하게 생각이 될 수도 있습니다. 이 구조체는 surf 함수가 호출될 때마다 연산할 정점과 관련된 정보를 전달할 목적으로 사용한다고 생각하시면 편합니다. 위 코드에서는 Input 구조체 내부에 uv_MainTex가 선언되어 있습니다. 이는 해당 정점의 텍스처의 uv좌표를 의미합니다. surf 함수가 호출되면 정점 데이터에 포함된 uv 좌표를 연산하는데 사용하겠다는 것입니다. 실제로 surf 함수 내부에서는 tex2D함수를 통해서 Property로 설정해둔 텍스처를 uv 좌표를 통해서 rgb값을 읽고 있습니다. 정점의 색상값을 샘플링한 것입니다. 위 코드는 단순하게 텍스처를 적용하는 코드이므로 Input 구조체로 설정된 다른 변수는 더이상 없습니다. 하지만 오늘 살펴볼 Rim Lighting과 같은 경우, Input에서 제공하는 float3 viewDir를 사용해야 합니다. 우리가 해야할 일은 그저 Input 구조체 내부에 필요한 정보를 얻을 수 있는 변수를 선언하기만 하면 됩니다. Shader Tutorial 1에서 Input 구조체에 선언할 수 있는 다양한 내용에 대한 소개를 해두었으니 참고하시기 바랍니다.

# 텍스쳐를 적용한 Diffuse Shader

특별한 내용이 아닙니다. Diffuse Shader에서 텍스처만 적용하면 끝납니다. 그리고 이 쉐이더는 바로 위에 있는 코드입니다. 이에 관한 내용은 벌써 위에서 설명하였습니다. Property에서 텍스처를 설정하여 유니티 엔진 상에서 설정할 수 있도록 하고, 이 텍스처를 쉐이더 코드와 연결합니다. 그리고 surf 함수를 통해서 텍스처의 색상값을 읽어서 SurfaceOutput에 적용합니다. 이 과정은 tex2D 함수를 통해서 이뤄집니다. 이 쉐이더 코드를 바탕으로 Bumpmap을 적용하도록 해보겠습니다. Bumpmap을 적용하기 전에 비교대상으로 Diffuse Texture Shader을 적용한 결과를 보도록 하겠습니다.

![_MainTexture](/assets/img/DiffuseTexture.png)

# Bumpmap Shader

Diffuse Shader와 Bumpmap Shader는 크게 다르지 않습니다. 모델의 noraml 정보를 담고 있는 텍스처를 추가로 사용하면 끝입니다. 우선 코드를 보도록 하겠습니다.

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

추가된 내용이 많지 않습니다. Property에 _BumpMap이 추가되었고, 쉐이더 코드에 연결하기 위한 변수가 설정되었습니다. Input 구조체에서 BumpMap을 위한 변수가 선언되었습니다. 그리고 surf 함수 내부에서 _BumpMap을 사용하고 있습니다.
주의해서 살펴볼 곳은 surf 함수 내부입니다. 다른 부분은 Diffuse Shader를 설명하면서 언급한 내용으로 충분이 이해할 수 있을 것입니다. BumpMap으로 사용할 텍스처를 읽는 과정이 MainTex와는 다소 다르다는 것을 확인할 수 있습니다. UnpackNormal 함수가 추가적으로 사용되었습니다. 이 함수가 바로 Normal Mapping을 가능하게 해주는 함수입니다.

# UnpackNormal

우선 Normal Mapping에 대해서 간단하게 살펴보도록 하겠습니다. Normal Map은 텍스처의 형태로 모델의 Normal를 저장하고, 이를 바탕으로 간단한 폴리곤 모델을 시각적으로 현실감있게 표현할 수 있도록 해줍니다. 자세한 내용은 검색을 통해서 알아보시기 바랍니다. 지금은 쉐이더에서 Normal Map을 적용하는 과정에 집중하도록 하겠습니다.
먼저 tex2D 함수로 텍스처로부터 RGB 채널값을 읽을 수 있습니다. 하지만 우리는 이 RGB 채널로부터 Normal Vector값을 추출해야 합니다. R은 x축, G는 y축, B는 z축에 대응됩니다. 문제는 RGB 값은 0과 1사이의 값으로 표현된다는 점입니다. Normal Vector의 x, y, z값은 양과 음의 방향성을 가진 값입니다. 그래서 우리는 RGB의 0 ~ 1 공간을 xyz의 -1 ~ 1 공간으로 변환해야 합니다. 참고로 -1 ~ 1이라는 수치가 정해진 이유는 Normal vector를 표현할 수 있는 벡터 공간이기 때문입니다. 이 과정을 편하게 해주는 함수가 바로 UnpackNormal입니다. 먼저 함수를 살펴보도록 하겠습니다.   

{% highlight glsl%}
inline fixed3 UnpackNormal(fixed4 packednormal)
{
#if defined(SHADER_API_GLES)  defined(SHADER_API_MOBILE)
    return packednormal.xyz * 2 - 1;
#else
    fixed3 normal;
    normal.xy = packednormal.wy * 2 - 1;
    normal.z = sqrt(1 - normal.x*normal.x - normal.y * normal.y);
    return normal;
#endif
}
{% endhighlight %}

함수는 매우 단순합니다. 이 함수의 기능 또한 방금 설명한 내용 그대로 입니다. 정의된 값에 의해서 처리하는 방식이 조금 다를 뿐입니다. SHADER_API_GLES와 SHADER_API_MOBILE가 정의되어 있다면 간편하게 벡터 자체를 2배하고 1을 빼주어서 처리할 수 있습니다. 그 외의 경우에는 xy 벡터로부터 z 값을 계산해내는 것입니다. 보통 Normal Map으로 사용되는 텍스처는 RG 채널만 벡터를 표현하기 위해서 사용됩니다. 왜냐하면 RG 채널로부터 B 채널을 계산해낼 수 있기 때문입니다. 이는 Normal의 크기는 1으로 정의되어 있기 때문입니다. 그러면 사용하지 않는 B 채널은 다른 곳에 이용할 수 있게 됩니다. 보통은 B 채널을 광 차폐 정도, 주변 법선 벡터와 각도 차이들은 표현하는데 사용된다고 합니다.
결국 UnpackNoraml을 사용하면 궁극적으로 RGB로부터 xyz 벡터를 추출할 수 있게 됩니다. BumpMap을 적용하기 위한 준비는 모두 끝났습니다. 결과는 다음과 같습니다.

![_MainTexture](/assets/img/Bumpmap.png)

# Rim Lighting Shader

이제 마지막으로 Rim Lighting Shader를 살펴볼 차례입니다. 글로 설명하지말고 바로 Rim Lighting이 무엇인지 사진을 통해서 보도록 하겠습니다.

![_MainTexture](/assets/img/RimLighting.jpeg)

사진을 살펴보면 뒤에서 빛이 쏘아지고 있고, 인물의 외곽선 따라서 빛이 반사되어 밝게 보입니다. 이 효과가 바로 Rim Lighting입니다. 눈으로 확인하면 금방 알 수 있습니다. 우리는 지금부터 쉐이더를 사용하여 이 효과를 흉내내도록 할 예정입니다. 거창하게 시작하지만 매우 단순한 형태로 구현하도록 하겠습니다. Rim Lighting을 표현하는 기본적인 아이디어를 맛본다고 생각하시면 됩니다.

# 가장 단순한 아이디어

사진으로부터 힌트를 얻어보도록 하겠습니다. 가장 먼저 눈에 띄는 모습은 인물의 외곽선이 두드러진다는 점입니다. 내부는 어둡게 표현되고 있습니다. 이부분에 주목합니다. 외곽이라는 것은 정점의 Normal Vector와 관찰자의 위치를 향하는 View Direction Vector가 이루는 각도의 크기가 90도 라는 것을 의미합니다. 각도는 벡터의 내적을 통해서 확인할 수 있습니다. 즉, Normal Vector와 View Direction Vector의 내적이 0이 되는 정점이 바로 외곽선을 이루는 정점이 됩니다. 물체의 외곽을 확인할 수 있는 방법은 찾았습니다. 이를 적용하는 쉐이더 코드를 작성하도록 하겠습니다.

{% highlight glsl%}
Shader "Custom/RimLighting1" {
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
			float3 viewDir;	// 관찰자의 위치를 향하는 방향 벡타입니다. Normal Vector와 내적하기 위해서 추가합니다.
		};

		void surf (Input IN, inout SurfaceOutput o) {
			o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));    // 텍스처로부터 값을 읽은 후에 UnpackNormal
			half rim = 1.0 - saturate(dot(IN.viewDir, o.Normal));   // 내적 결과를 처리합니다.
			o.Emission = rim * (1, 1, 1);
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

Diffuse Texture Shader에서 추가된 부분을 살펴보겠습니다. 먼저 Input 구조체에 viewDir가 선언된 것을 확인할 수 있습니다. viewDir은 주석에서 언급한 대로 관찰자 위치를 향하는 벡터입니다. 보통은 카메라를 향하는 벡터로 생각하시면 됩니다. 이 벡터와 BumpMap으로부터 얻어낸 Normal을 내적합니다. 그러면 -1 ~ 1 사이의 결과가 나올 것입니다. Backface Culling이 적용되면 랜더링 대상이 되는 정점의 Normal과 viewDir의 내적은 0 ~ 1 사이 값이 될 것입니다. 지금은 단순하게 -1 ~ 1 사이라고 생각하도록 하겠습니다. 이전에도 설명한 적이 있는 saturate 함수가 나왔습니다. 이 함수의 기능은 매우 단순하면서도 중요합니다. 전달되는 값이 0보다 작을 경우는 0, 1보다 클 경우는 1, 그 사이 값이면 그대로 반환됩니다. 이 함수를 사용한 이유는 음수를 제외하기 위함입니다. 일반적인 코드라면 if를 통해서 분기 처리할 수도 있습니다. 만약 내적 결과가 0보다 작으면 0으로 처리하도록 할 수도 있습니다. 하지만 쉐이더 코드에서는 좋은 방법이 아닙니다. 이는 GPU의 데이터 처리 방법과 관련이 있습니다. GPU는 기본적으로 파이프라인 구조입니다. 대량의 데이터를 정해진 명령들로 처리하는데 GPU는 최적화 되어 있습니다. 이를 SIMD(single instruction multiple data)라고 부릅니다. 분기가 발생하게 되면 GPU 파이프라인은 새롭게 만들어지게 됩니다. 그러므로 쉐이더 코드에서는 분기보다는 모든 경우에 동일한 파이프라인으로 제어할 수 있도록 코드를 작성합니다. saturate 함수도 분기를 피할 수 있는 좋은 수단입니다. 경우에 따라 분기시키기 보다는 데이터 자체를 분기처리된 것처럼 수정할 수 있기 때문입니다. 앞으로도 자주 만날 수 있을 것입니다.
다시 Rim Lighting으로 돌아갑시다. saturate 함수를 통해서 0 ~ 1 사이 값을 얻었습니다. 그리고 그 결과를 1에서 빼줍니다. Rim Lighting 효과는 외곽에 더 강하게 나타나야 하기 때문입니다. 정점마다 계산된 rim 값을 컬러에 가중치로 적용하도록 합니다. Emission을 볼 수 있습니다. Emission은 표면에서 방출되는 빛의 색상과 강도를 제어합니다. 마치 물체 자체가 발광하는 것 같은 효과를 줄 수 있습니다. 이에 대한 자세한 내용은 ![Emission](https://docs.unity3d.com/kr/current/Manual/StandardShaderMaterialParameterEmission.html)를 통해서 확인할 수 있습니다. 결과적으로 rim 값이 큰 외곽부분일 수록 흰색에 가까운 효과가 강하게 나타납니다. 위 코드를 실행시키는 다음과 같은 결과를 얻을 수 있습니다.

![RimLighting1](/assets/img/RimLighting1.png)

# 좀더 완성되 높은 아이디어

여러분의 예상과는 다른 결과일 것입니다. 외곽 부분만 멋있게 빛나는 효과가 연출될 것이라고 기대했지만 결과는 예상과는 다르게 전체적으로 흰색으로 덮혀있습니다. 하지만 외곽으로 갈수록 효과는 강해진다는 것은 확인할 수 있습니다. 안쪽은 효과를 약하게 만들고, 바깥쪽은 효과를 강하게 만든다면 원하던 결과에 가까워질 것 같습니다. 이왕이면 외곽에 매우 근접한 부분만 빛나면 더욱 좋을 것 같습니다. 한번에 원하는 효과를 코드로 얻어내기는 힘들 수 있습니다. 수치를 수정하면서 원하는 결과를 찾아야 합니다. 이럴때 Property는 매우 유용하게 사용할 수 있습니다. 유니티 에디터에서 쉐이더과 관련된 데이터를 조절할 수 있고 그 결과를 바로 확인할 수 있기 때문입니다. 적절하게 Property로 추가하도록 하겠습니다. Rim Lighting의 색상을 추가해보겠습니다. 마지막으로 Rim Lighting의 강도 또한 설정할 수 있도록 하겠습니다. 이 강도는 얼마나 외곽에 집중되어 나타날지를 정합니다. 다음은 완성된 코드입니다.

{% highlight glsl%}
Shader "Custom/RimLighting2" {
	Properties {
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_BumpMap ("Bumpmap", 2D) = "Bump" {}    //  Normal을 저장하고 있는 텍스처
		_RimColor ("Rim Color", Color) = (0.1, 0.3, 0.2)	// 유니티 에디터에서 직접 수정할 수 있도록 합니다.
		_RimPower ("Rim Power", Range(0.0, 10.0)) = 5
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
		float4 _RimColor;
      	float _RimPower;

		// Vertex를 연산할 때 필요한 데이터를 정합니다.
		// surf 함수로 호출될 때마다 해당 정점의 관련 데이터가 연산에 적용됩니다.
		struct Input {
			float2 uv_MainTex;	// uv는 해당 텍스처의 텍스처 좌표를 의미합니다.
			float2 uv_BumpMap;
			float3 viewDir;	// 관찰자의 위치를 향하는 방향 벡타입니다. Normal Vector와 내적하기 위해서 추가합니다.
		};

		void surf (Input IN, inout SurfaceOutput o) {
			o.Albedo = tex2D(_MainTex, IN.uv_MainTex).rgb;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));    // 텍스처로부터 값을 읽은 후에 UnpackNormal
			half rim = 1.0 - saturate(dot(IN.viewDir, o.Normal));   // 내적 결과를 처리합니다.
			o.Emission = _RimColor.rgb * pow (rim, _RimPower);	// rim 값을 _RimPower만큼 제곱하여 Rim Lighting이 적용되는 범위를 제어합니다.
		}
		ENDCG
	}
	FallBack "Diffuse"
}
{% endhighlight %}

rim이 0 ~ 1 사이 값이므로 _RimPower값이 커질 수록 1을 제외한 값은 감소하게 됩니다. 결국 우리가 원하는 대로 외곽에 집중되어 Rim Lighting 효과를 줄 수 있습니다. 위의 쉐이더 코드를 실행하면 다음과 같은 결과를 얻을 수 있습니다.

![RimLighting2](/assets/img/RimLighting2.png)

# 마무리하며

지금까지 살펴본 내용은 다음과 같습니다.

* 유니티 쉐이더를 구성하는 요소들: Properties, SubShader, Input, FallBack, surf 함수
* Diffuse Shader
* Diffuse Texture Shader
* Bumpmap Shader, Normal Mapping, saturate 함수, 쉐이더 코드 내의 분기가 미치는 영향
* Rim Lighting Shader

이 내용들을 차례대로 살을 붙여나가는 방식으로 살펴보았습니다. 위의 예제 코드는 유니티 메뉴얼에서 참고하였습니다.


