---
layout: post
title: "AbstractFactoryPattern"
author: "이인재"
date: 2016-06-05 03:00:00
excerpt: "객체군들의 생성에 유용한 AbstractFactoryPattern"
tags: [Design Pattern, 객체생성, Factory Pattern]
comments: true
---

앞으로 자주 공부하게 될 Design Pattern에도 중요한 원칙들이 있습니다. 이 원칙들은 각각의 Design Pattern이 추구하는 가치이기도 합니다.

# 제품군 생성

제품군 : 하나의 제품을 이루는 객체들의 집합
일상생활에서도 제품군에 대한 예를 매우 많이 접할 수 있다. 자동차를 보자. 자동차의 제품군은 무엇일까? 모든 차가 그런 것은 아니겠지만 대부분의 자동차는 엔진, 기어 브레이크, 핸들, 바퀴 등등의 공통적인 구성품들로 이루어져 있다.
이때 자동차를 이루는 공통적인 구성품들의 집합을 제품군으로 볼 수 있다.

![자동차를 이루는 제품군](/assets/img/1.jpg)

# OCP is not a panacea.

![생각이 있는 Beverage](/assets/img/decorator.png)

{% highlight csharp %}
aaa
{% endhighlight %}

참고자료: GOF 디자인 패턴, HeadFirst Design Pattern
