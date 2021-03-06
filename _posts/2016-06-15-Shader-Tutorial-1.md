---
layout: post
title: "Shader Tutorial 1"
author: "장문익"
date: 2016-06-15 03:00:00
excerpt: "Shader에 대해서 차근차근 알아봅시다."
tags: [Shader, Surface Shader, Fragment Shader, Unity]
comments: true
---

# 들어가며

이 글은 shader을 처음 접하는 사람을 대상으로 작성되었습니다. [참고자료](https://unitygem.wordpress.com/shader-part-1/)을 보시면 더욱 자세한 내용을 확인할 수 있습니다. 이 글은 참고자료를 번역 및 요약한 자료임을 밝힙니다. 그리고 unity engine에서 사용할 수 있도록 작성된 내용입니다.

# shader와 material

shader는 mesh를 가지고 스크린에 렌더링합니다. shader의 다양한 property를 조절하면 렌더링되는 모델에 다양한 효과를 줄 수 있습니다. 이런 property에 대한 설정이 바로 material로 저장됩니다.

shader는 크게 둘로 나눌 수 있습니다.

* Surface shader은 대부분의 어려운 일들을 손쉽게 해주며 다양한 환경에서 좋은 효과를 발휘합니다.
* Fragment shader는 더욱 더 다양한 것들을 할 수 있게 해줍니다. 그만큼 작성하기도 어렵습니다. 하지만 Vertex lighting과 같은 저레벨 조작을 가능하게 해줍니다. Vertex lighting은 모바일 장치에서 좀더 좋은 효과 주기 위한 pass 활용에 유용하게 사용할 수 있습니다.

이번에는 Surface shader에 집중하도록 하겠습니다.

# shader를 작성하는데 참고할만한 자료들

* [Martin Kraus’s fantastic Wiki Book GLSL Programming/Unity](https://en.wikibooks.org/wiki/GLSL_Programming/Unity)
* [Unity’s Shader Reference](http://docs.unity3d.com/Manual/SL-Reference.html)
* [NVidia’s tutorial on the CG programming language](http://http.developer.nvidia.com/CgTutorial/cg_tutorial_chapter01.html)
* [CreativeTD’s video series on writing surface shaders](http://www.creativetd.com/?page_id=617)

# shader pipeline

shader는 어떤 3D geometry를 2D 스크린의 픽셀들로 변환합니다. 다행스럽게도 우리는 이 변환 과정 중 몇몇에만 개입하면 됩니다. surface shader는 다음의 과정을 거칩니다.

![ShaderPipeline](/assets/img/ShaderPipeline.png)

*만약 여러분이 '마지막 함수' 다음에 있는 '마지막 픽셀의 색상을 지정하지 않는다면, 여러분은 lighting에 영향을 줄 수 있는 normal을 전달할 수 있습니다.*

fragment shader는 유사한 과정을 거치지만, *Vertex Function*을 사용해야 하며 픽셀 부분에 사용할 정보를 다루는 많은 일이 많습니다. surface shader는 이것들을 사용하지 않습니다.

이제 여러분의 함수가 어떻게 호출되는지 살펴봅시다. shader 코드에 필요한 것들이 무엇인지 살펴봅시다.

![ShaderCodeProcess](/assets/img/ShaderProcess.png)

이제 여러분은 shader를 작성할 것입니다. property 몇 개를 사용할 것입니다. 또 하나 혹은 그 이상의 subshader를 사용할 것입니다. subshader 사용은 여러분이 구동하는 플랫폼에 의존적입니다. 여러분은 fallback shader를 정해야 합니다. fallback shader는 만약 subshader가 디바이스에서 하나도 실행되지 않으면 사용될 것입니다.

각각의 subshader는 데이터를 처리하여 내보내는 pass를 적어도 하나 이상 가집니다. 여러분은 이 pass를 서로 다른 동작을 수행하도록 사용할 수 있습니다. 만약에 Grab Pass에서는 오브젝트가 나타날 디스플레이에 이미 렌더링된 픽셀을 캡쳐할 수 있습니다. 만약에 좀더 좋은 distortion effect(일그러짐 효과)를 원한다면 편리할 것입니다. 다수의 pass를 사용하는 또 다른 이유는 바로 단계별로 다양한 효과를 만들어 내기 위해서 사용하는 *depth buffer*를 쓰거나 무시하는 것과 같은 기능들을 다루기 위해서 입니다. 

여러분이 surface shader를 작성하면, 이는 바로 subshader에 작성됩니다. 시스템은 다수의 pass 안에 여러분의 코드를 컴파일합니다. shader는 2D 스크린에 쓰는 동안 각각의 픽셀이 카메라로부터 얼마나 떨어져 있는지에 대한 정보를 유지합니다. 그 이유는 만약에 어떤 geometry의 일부분이 이미 그려진 다른 것 뒤에 존재한다면 덮어서 쓰면 안되기 때문입니다. 여러분은 shader 코드가 이 *Z buffer*에 영향을 받을지, 혹은 pass 안의 명령어를 이 버퍼에 적용할 수 있습니다. 예를 들어 명령어 'Zwrite Off'는 Z buffer를 업데이트하지 않도록 합니다.

여러분은 Z buffer를 이용하여 다른 오브젝트에 구멍을 만들 수 있습니다. 이 shader를 사용한 모델 뒤에 위치한 오브젝트는 그 어떤 실질적인 픽셀을 출력하지 않습니다(왜냐하면 Z buffer는 무언가가 이미 거기에 존재한다고 말할 것이기 때문입니다).

다음 shader 코드를 봅시다. 

![ShaderEx1](/assets/img/ShaderEx1.png)

아마 *Propertise* 부분, *Subshader*, *Fallback*을 확인할 수 있을 것입니다.

# shader 코드 이해하기

이 글의 나머지는 대부분 이 단순한 코드 블럭 내부에서 어떤 마법이 펼쳐지는지 이해하는데 집중할 것입니다. 왜냐하면 shader coding convention을 준수해야만 마법이 작동하기 때문에 여러분은 반드시 이해해야 합니다.

shader 프로그래밍을 할 때, 여러분은 올바른 이름을 사용하여야 합니다.

# property 소개

 여러분은 shader 정의 내부의 Properties {...} 부분에 property를 정의할 수 있습니다. property는 모든 sub shader들에게 공유됩니다.

 property의 정의는 다음의 형식을 따릅니다.

 *_Name("Displayed Name", type) = default value[{options}]*

 * *_Name*은 여러분의 프로그램에 참조될 이 property의 이름입니다.
 * *Displayed Name*은 material editor에 나타납니다.
 * *type*은 property의 타입입니다. 여러분은 다음 중 하나를 선택할 수 있습니다.
    * Color - value는 전체 프로그램을 위한 하나의 색상이 될 것입니다.
    * 2D - value는 2의 제곱 크기의 텍스처입니다. 이 텍스처는 모델의 UV에 기반하여 특정 픽셀이 프로그램에 의해 샘플링됩니다.
    * Rect - value는 2의 제곱 크기가 아닌 텍스처입니다.
    * Cube - value는 반사에 사용되는 3D 큐브 맵 텍스처입니다. 특정한 픽셀을 프로그램으로 샘플링할 수 있습니다.
    * Range(min, max) - value는 min과 max 사이의 부동소수점입니다.
    * Float - value는 부동소수점 숫자 중 어떤 값입니다.
    * Vector - value는 4차원 벡터입니다.
* defalut value는 property의 기본 값입니다.
    * Color - 색상은 부동소수점으로 (r, g, b, a) 각 부분을 나타내는 방식으로 표현됩니다. 예를 들면 (1,1,1,1)입니다.
    * 2D / Rect / Cube -  텍스처 타입의 기본값으로 빈 string이거나 "white", "black", "gray", "bump"로 표현됩니다.
    * Float / Range - 채택될 값입니다.
    * Vector - (x, y, z, w)로 표현되는 4차원 벡터입니다.
* *{ options }*은 단지 2D, Rect, Cube와 같은 텍스처 타입과 관련있습니다. 하지만 **반드시** {}는 명시되어야 합니다. 여러분은 공백으로 분리하여 다양한 옵션을 조합할 수 있습니다. 선택지는 다음과 같습니다.
    * TexGen *texgenmode*: 자동 텍스처 조정 생성 모드입니다. *ObjectLinear, EyeLinear, SphereMap, CubeReflect, CubeNormal* 중 하나입니다. 이것들은 OpenGL texgen 모드와 일치합니다. 만약 여러분이 정점 함수를 작성한다면 무시됩니다. 

그래서 property는 아마 다음과 같이 표현될 것입니다.

{% highlight glsl %}
//Define a color with a default value of semi-transparent red
_MainColor ("Main Color", Color) = (1,0,0,0.5)
//Define a texture with a default of white
_Texture ("Texture", 2D) = "white" {}
{% endhighlight %}

# Tags

surface shader는 하나 혹은 그 이상의 tag로 꾸며집니다. 이 tag들은 하드웨어가 shader를 언제 호출할지를 결정할 수 있도록 합니다.

에시 코드의 **Tags {"RenderType" = "Opaque"}**은 시스템이 Opaque 기하구조를 렌더링할 때 우리에게 알려줄 것을 지시합니다. Unity에서는 이런 것들을 설정할 수 있는 다양한 옵션을 제공합니다. 또 다른 예는 **"RenderType" = "Transparent"** 입니다. 이 tag는 shader에게 잠재적으로 semi-transparent 혹은 transparent 픽셀을 출력할 것이라고 알려줍니다. Unity standard shader는 또한 cutout과 fade를 제공합니다.

오브젝트가 프로젝터에 의해 영향 받지 않도록 하는 **"IgnoreProjector" = "True"**와 **"Queue" = "xxx"**와 같은 tag들도 유용합니다.

Queue tag는 RenderType이 Transparent로 설정되어 있을 때, 매우 흥미로운 효과를 보여줍니다. 기본적으로 오브젝트가 언제 렌더링될지를 알려줍니다.

* Background - 이 render queue는 다른 것들 보다 먼저 렌더링됩니다. skybox와 같은 것에 사용됩니다.
* Geometry(*default*) - 대부분의 오브젝트에 사용됩니다. Opaque geometry는 이 queue를 사용합니다.
* AlphaTest - alpha test된 geometry는 이 queue를 사용합니다. 모든 오브젝트들이 렌더링된 후에 alpha test된 오브젝트를 렌더링하는 것이 효율적이기 때문에 Geometry queue로부터 분리된 queue입니다.
* Transparent - 이 렌더링 queue는 *Geometry*와 AlphaTest를 거친 후에 렌더링됩니다. glass나 particle effect와 같은 alpha-blend된 오브젝트(즉, depth buffer에 쓰지 않는 shader)는 이 queue에 위치해야 합니다.
* Overlay - 이 render queue는 덮어 씌우는 효과를 의미합니다. lens flare와 같이 가장 마지막에 렌더링되어야 하는 것들은 여기에 위치해야 합니다.

흥미로운 점은 여러분은 이 기본적인 queue들로부터 value를 더하거나 뺄 수 있다는 것입니다. transparent 오브젝트에 중요한 효과를 줍니다. 그리고 만약에 여러분이 빌보드로 처리된 나무로 덮힌 물 표면을 만들고자 한다면 이 queue 효과적인 해결책이 될 수 있습니다.

예를 들면 **"Queue" = "Transparent-102"**는 투명한 물이 빌보드로 처리된 나무들 뒤에 위치하도록 해줍니다.

# Shader Structure

이제 shader 내부의 코드를 살펴보도록 하겠습니다.

![Shader Structure](/assets/img/ShaderStructure.png)

우리의 CG Program은 CG 언어로 작서오디었습니다. CG 언어는 C 언어와 유사합니다. 자세한 내용은 NVidia의 문서를 읽어보시길 바랍니다. 여기에서는 기초적인 내용만 다루도록 하겠습니다. 우리는 CGPROGRAM과 ENDCG를 추가하여 CG 프로그래밍 코드 구역을 구분합니다. 그래서 이 둘 사이에서 CG 메서드를 사용할 수 있습니다.

가장 중요한 것은 float와 vector(vec) 뒤에는 2와 4 사이의 수로 정의할 수 있다는 점입니다. 이로써 여러분은 2개 혹은 4개를 사용하여 float를 구성할 수 있습니다. 그리고 실질적으로 그것들을 단독으로 혹은 함께 사용할 수도 있습니다.

{% highlight glsl %}
//Define a float variable
vec2 coordinate;
//Define a color variable
float4 color;
//Multiply out a color
float3 multipliedColor = color.rgb * coordinate.x;
{% endhighlight %}

여러분은 개별의 요소를 호출할 수도 있고, 혹은 요소 묶음을 호출할 수도 있습니다. 여러분은 color, position, normal 등을 저장하기 위해서 .xyzw와 .rgba 표기법을 사용할 수 있습니다. 여러분은 float을 단일 값으로 사용할 수 있습니다. .rgba와 같은 사용법은 *swizzling*이라고 부릅니다.

여러분은 *half*와 *double* 타입도 만날 것입니다. 이 타입들은 보통의 float에 비해서 각각 반 혹은 두 배의 해상도를 가집니다. *half*는 성능 상의 이유로 자주 사용되지만 정밀도는 떨어집니다. *fixed*라는 고정 소수점의 실수도 있습니다.

여러분은 아마 색상에 사용하기 위해서 값의 범위를 0 ~ 1로 변환하고 싶을 것입니다. 이렇게 하기 위해서 여러분은 *saturate* 함수를 사용할 수 있습니다. 예를 들어 return saturate(somecolor.rgb)와 같이 *swizzled* 버전도 사용할 수 있습니다. 여러분은 vector의 길이를 알기 위해서 length 명령어를 사용할 수 있습니다. 예를 들면 float size = length(someVec4.xz)와 같습니다.

# Surface Shader로부터 정보를 출력하기

surface 함수는 각 픽셀마다 한 번만 호출됩니다. 시스템은 하나의 픽셀을 위해서 우리가 전달한 Input 구조체의 현재 값을 처리합니다. 기본적으로 Input 구조체 안의 *보간한* 값들이 모든 mesh 표면을 가로지릅니다.

*surf* 함수를 살펴봅시다.

{% highlight glsl %}
void surf (Input IN, inout SurfaceOutput o) 
{
     o.Albedo = tex2D (_MainTex, IN.uv_MainTex).rgb;
}
{% endhighlight %}

뭔가를 o.Albedo에 반환한다는 것을 알 수 있습니다. o.Albedo는 unity에서 정의하여 제공하는 SurfaceOutput 구조체 내부에 있습니다. 한 번 빠르게 살펴보도록 하겠습니다. Albedo는 픽셀의 색상입니다.

{% highlight glsl %}
struct SurfaceOutput 
{
    half3 Albedo;      //The color of the pixel
    half3 Normal;     //The normal of the pixel
    half3 Emission;   //The emissive color of the pixel
    half Specular;     //Specular power of the pixel
    half Gloss;         //Gloss intensity of the pixel
    half Alpha;         //Alpha value for the pixel
};
{% endhighlight %}

여러분은 단지 이 구조체에 값을 반환하기만 하면 됩니다. 그리고 scene 뒤에서 이 구조체를 처리하는 실제의 pass들이 만들어 질 때, Unity는 이 구조체를 처리합니다. 

# 저는 여러분에게 마법을 보여줄 것을 약속합니다

surf 함수의 가장 앞 부분을 봅시다. 우리는 Input 구조체를 정의하였습니다. 다음과 같습니다.

{% highlight glsl %}
struct Input {
    float2 uv_MainTex;
};
{% endhighlight %}

우리는 매번 surf 함수를 호출할 때마다 현재 픽셀의 MainTex 텍스처 좌표를 알려주도록 하기 위해서 구조체를 생성하기만 하면 됩니다. 만약 우리가 _OtherTexture라고 부르는 두번째 텍스처를 가지고 있다면 우리는 uv 좌표를 얻을 수 있습니다. 바로 다음을 추가하기만 하면 됩니다.

{% highlight glsl %}
struct Input {
    float2 uv_MainTex;
    float2 uv_OtherTexture;
};
{% endhighlight %}

만약에 우리가 다른 텍스처를 위한 두 개의 uv 세트를 가졌다면 둘 다 얻을 수 있습니다. 

{% highlight glsl %}
struct Input {
    float2 uv_MainTex;
    float2 uv2_OtherTexture;
};
{% endhighlight %}

Input 구조체는 보통 우리가 사용하는 모든 텍스처를 위한 uv와 uv2 좌표 모두 가지고 있습니다. 

만약 shader가 복잡하고 음영처리된(shaded) 픽셀에 대해 알 필요가 있다면, 우리는 Input 구조체에 변수를 추가하기만 함으로써 요청할 수 있습니다.

* float3 viewDir은 view direction을 포함할 것입니다. Parallax effects, rim lighting 등에 사용됩니다.
* float4 COLOR semantic은 보간된 각 정점 색상을 포함할 것입니다.
* float4 screenPos는 reflection effect에 사용되는 screen 공간 위치를 포함할 것입니다.
* float3 worldPos는 world 공간 위치를 포함할 것입니다.
* float3 worldRefl는 *만약 surface shader가 o.Normal에 쓰지 않는다면* world reflection 벡터를 포함할 것입니다. 
* float3 worldNormal은 *만약 surface shader가 o.Normal에 쓰지 않는다면* world normal 벡터를 포함할 것입니다.
* INTERNAL_DATA는 o.Normal에 쓸 때 무언가를 연산하기위해 사용하는 WorldNormalVector와 같은 함수에 사용되는 구조체입니다. 

여러분은 "COLOR semantic가 도대체 뭐에요?"라고 물을 수도 있습니다. 여러분이 normal fragment shader를 작상할 때, 여러분은 각각의 변수들이 실제로 무엇을 표현하는지 input 구조체에 알려줍니다. 그래서 만약 여러분이 미쳤다면 *float2 MyUncleFred : TEXCOORD0;*라고 말할 수도 있습니다. 그러면 MyUncleFred는 모델의 UV가 될 것입니다. surface shader에서 우리가 고민해야 하는 유일한 것은 바로 COLOR입니다. 그래서 *float4 currentColor: COLOR;* 이 픽셀의 보간된 색상이 될 것입니다. 여러분은 아마도 이것에 대해서 걱정할 필요가 없을 것입니다.

# 실제로 뭔가 하기

이제 다루지 않은 두 줄만 남았습니다.

{% highlight glsl %}
Sampler2D _MainTex;
{% endhighlight %}

우리는 Properties 부분에서 정의한 모든 property에 접근하기 위해서 반드시 CG 프로그램에 변수를 넣어주어야 합니다. 변수명은 반드시 property와 동일한 이름이어야 합니다.

![ShaderEx1](/assets/img/CGVariables.png)

uv 혹은 uv2에 상관없이 텍스처 좌표를 얻기 위해서는 Input 구조체에 반드시 동일한 이름을 사용해야 합니다.

_MainTex 변수는 main texture와 연결된 *Sampler2D*입니다. *Sampler2D*는 주어진 uv 좌표로 텍스처의 픽셀을 읽을 수 있습니다.

만약 우리가 _Color 변수를 정의하였다면 우리는 property를 반드시 다음과 같이 정의해야 합니다.

{% highlight glsl %}
float4 _Color;
{% endhighlight %}

이제는 우리 shader에서 한 줄만 남았습니다.

{% highlight glsl %}
o.Albedo = tex2d( _MainTex, IN.uv_MainTex).rgb;
{% endhighlight %}

tex2d는 시스템으로부터 얻은 uv 좌표로 _MainTex를 샘플링 합니다. 왜냐하면 텍스처는 float4(alpha를 포함하는)이고, 우리는 o.Albedo에 저장할 .rgb 값만 필요합니다. 만약 우리가 alpha도 얻기 원한다면 코드는 다음과 같을 것입니다.

{% highlight glsl %}
float4 texColor = tex2d( _MainTex, IN.uv_MainTex );
o.Albedo = texColor.rgb;
o.Alpha = texColor.a;
{% endhighlight %}

# 요약

여러분은 지금까지 많은 내용을 읽었습니다. 지금까지 읽은 내용들을 바탕으로 shader를 작성하기는 제법 어려울 수도 있습니다. 하지만 이 지식들을 바탕으로 다음 글을 읽는다면 여러분은 여러 텍스처와 normal들을 사용하여 멋진 shader를 만들어 낼 수 있을 것입니다.
