//---
layout: post
title: "Shader Tutorial 3"
author: "장문익"
date: 2016-06-19 03:00:00
excerpt: "Color Blend를 사용하여 사실적인 Snow shader를 만들어 봅시다."
tags: [Shader, Blend Shader, Surface Shader, Snow Shader, Unity]
comments: true
---//

다음 경우 이 튜토리얼이 도움이 될 수 있습니다.
* surface shader의 blend colour에 대해서 이해하고 싶은 경우
* 좀 더 사실적인 snow shader를 만들고 싶은 경우

# 들어가며

Snow Shader를 좀더 완벽하게 만들어 봅시다. 눈이 내린 구역과 눈이 내리지 않은 구역 사이에 어색함이 느껴집니다. 축축한 눈이 쌓여있다는 느낌보다는 페인트가 튄 것 같은 느낌이 더 강합니다. 그래서 다음 과정은 눈과 텍스처가 동시에 랜더링 되는 곳에 여백을 두도록 합니다.

이를 위해서 surface shader 프로그램의 픽셀 설정을 수정하기만 할 것입니다. 하지만 이것만으로도 충분히 유용한 saturate 함수를 보여줄 수 있습니다.

# Shader 계획하기

눈이 얼마나 내릴지 Snow level로 저장될 때, 눈이 반투명한 곳에는 여백을 두도록 합니다. 눈이 점점 쌓일 수록 픽셀은 점점 더 불투명하게 됩니다. 전에는 단순하게 눈이 온 곳은 완전히 불투명한 흰 색이었습니다.

# Shader 구현하기

이전의 Snow Shader와 가장 다른 점은 이진 스위치를 Range 값으로 변경한다는 점입니다. 이로써 우리는 shader 로직을 다시 작성할 수 있고, if 기반의 스위치와는 다르게 수학을 사용할 수 있게 됩니다.

먼저 눈을 얼마나 blend할지를 property에 지정합니다. _Wetness라고 정하도록 하겠습니다.

{% highlight glsl%}
_Wetness ("Wetness", Range(0, 0.5)) = 0.3
{% endhighlight %}

이제 property에 해당하는 변수도 지정합니다.
{% highlight glsl%}
flaot _Wetness;
{% endhighlight %}

이제 눈과 픽셀의 normal을 내적합니다. 그리고 snow level을 lerp한 값도 적용하여 줍니다. 



    