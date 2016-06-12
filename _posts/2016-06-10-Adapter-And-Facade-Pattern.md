---
layout: post
title: "Adapter Pattern"
author: "장문익"
date: 2016-06-10 03:00:00
excerpt: "다른 두 인터페이스 간에 발생하는 호환성 문제를 해결해봅시다."
tags: [Design Pattern, Adapter]
comments: true
---

# 들어가며

프로그래밍에서 interface는 다형성과 연관된 중요한 요소입니다. 동일한 interface로부터 구현된 클래스들은 그 종류에 상관없이 동일한 코드 제어를 거칠 수 있습니다. 물론 결과는 각 객체마다 다릅니다. interface는 이런 장점으로 자주 사용됩니다. 반면 다른 interface 간의 호환은 매우 약합니다. interface는 자신을 상속하여 구현되는 클래스에는 일종의 규격이 될 수 있지만, 그렇지 않는 클래스에는 전혀 그렇지 않습니다. interface 단위로 구분되는 규격이기 때문입니다. 만약 두 interface를 동일하게 사용해야만 하는 상황에서는 어떻게 해야할까요? 두 interface를 동일한 코드 흐름으로 제어할 수는 없을까요? 오늘은 이에 대한 내용을 다뤄보도록 하겠습니다. 

# Adapter의 사전적인 정의

Adapter의 사전적인 정의는 다음과 같습니다.
"Adapter는 호환되지 않는 두 가지의 인터페이스를 함께 사용할 수 있도록 도와줍니다."
정의 자체는 다소 딱딱하게 느껴질 수도 있습니다. 하지만 Adapter는 다른 실생활에서 우리가 비교적 자주 접하는 용어입니다. 구글에서 'Adapter'라고 검색하면 다음 이미지가 가장 먼저 나타납니다.

![Real Adapter](/assets/img/Adapter.jpeg)

요즘은 덜 사용하긴 하지만 과거 110V에서 220V로 변하던 시기에 많은 가전 제품의 전원코드는 220V 기본으로 사용하지만 110V 전원소켓에도 사용할 수 있도록 이런 Adapter를 함께 지급하였습니다. 우리가 알아볼 Adapter도 동일한 기능을 수행합니다. 정의 그대로 서로 호환되지 않는 두 가지 interface를 호환가능하도록 도와주는 것입니다.

# Adapter Pattern의 정의

Adapter Pattern은 한 클래스의 인터페이스를 다른 측에서 사용할 수 있는 다른 인터페이스로 변환합니다. Adapter를 이용하면 인터페이스 호환성 문제때문에 같이 쓸 수 없는 클래스들을 사용할 수 있습니다.

# 문제 상황

지금 우리나라는 220v를 표준으로 채택하고 있습니다. 어느날 인터넷 쇼핑몰을 통해서 전자제품을 구매하였습니다. 이제 이 제품을 사용하기 위해서 포장을 뜯고 전원을 연결하려는 순간 당황스러운 일이 벌어집니다. 제품의 전원 코드가 110v였습니다. 순간 당황하였지만 우리는 해결책을 알고 있습니다. 220v Adapter를 붙여주면 되기 때문입니다. 전압과 상관없이 제품이 사용할 수 있도록 만들어져있기 때문에 가능한 일입니다.
프로그래밍에서도 이와 같은 상황을 접할 수 있습니다. 이미 만들어져 있는 라이브러리의 interface는 고정되어 있습니다. 하지만 우리가 사용하고 있는 객체들은 이 interface와는 다른 interface를 상속받아 구현되어 있습니다. 이런 경우 우리는 adapter pattern을 사용할 수 있습니다. 서로 다른 interface를 동일하게 제어하기 위해서 특정 interface를 다른 interface로 변환해주는 것입니다. 여기서 중요한 점은 동일하게 제어하기 위해서 두 interface 중 하나가 그 기준이 되어야 된다는 것입니다. 물론 기준이 되는 interface는 라이브러리가 요구하는 interface가 됩니다. 전원 adapter를 예로 생각하면 220v가 바로 기준이 되는 interface입니다. 

# Adapter Pattern의 구조

![ObjectAdapter](/assets/img/ObjectAdapter.jpg)

구조는 매우 단순합니다. 구조에 대해서 앞의 문제 상황을 적용해서 간략하게 설명을 하겠습니다.

* client는 전원 소켓입니다. 220v 전원 코드에 잘 맞도록 구성된 interface입니다.
* target은 220v 전원 코드를 나타내는 interface입니다.
* adaptee는 110v 전원 코드를 나타내는 interface입니다.
* adapter는 adpatee를 target으로 변환해주는 역할을 하는 클래스입니다.

만약 위의 문제 상황을 Adapter Pattern을 사용하면 이렇게 해결될 것입니다. 우리가 사용하고자 하는 client는 target만을 처리합니다. 하지만 adaptee는 target과는 호환되지 않는 interface입니다. 그래서 중간에 adapter는 이 둘 사이의 호환을 담당합니다. 그러면 adaptee를 문제없이 client에 적용할 수 있습니다.

# Adapter Patter 사용

아래 코드는 c#으로 작성되었습니다.

{% highlight csharp %}
using System;

namespace DoFactory.GangOfFour.Adapter.Structural
{
  /// <summary>
  /// MainApp startup class for Structural
  /// Adapter Design Pattern.
  /// </summary>
  class MainApp
  {
    /// <summary>
    /// Entry point into console application.
    /// </summary>
    static void Main()
    {
      // Create adapter and place a request
      Target target = new Adapter();
      target.Request();
 
      // Wait for user
      Console.ReadKey();
    }
  }
 
  /// <summary>
  /// The 'Target' class
  /// </summary>
  class Target
  {
    public virtual void Request()
    {
      Console.WriteLine("Called Target Request()");
    }
  }
 
  /// <summary>
  /// The 'Adapter' class
  /// </summary>
  class Adapter : Target
  {
    private Adaptee _adaptee = new Adaptee();
 
    public override void Request()
    {
      // Possibly do some other work
      //  and then call SpecificRequest
      _adaptee.SpecificRequest();
    }
  }
 
  /// <summary>
  /// The 'Adaptee' class
  /// </summary>
  class Adaptee
  {
    public void SpecificRequest()
    {
      Console.WriteLine("Called SpecificRequest()");
    }
  }
}

{% endhighlight %}

구조는 매우 단순합니다. Adaptee 클래스를 호환시키기 위한 코드입니다. Adaptee 클래스를 호환시켜주는 Adapter 클래스를 살펴봅시다. Adapter 클래스는 기본적으로 Target 클래스를 상속하여 구현되어 있으며, Adaptee 클래스를 참조하고 있습니다. Target 클래스는 우리가 사용할 기본 interface입니다. Adapter 클래스가 없다면 우리는 직접적으로 Adaptee를 사용할 수가 없습니다.(코드에는 없지만 클라이언트가 Target를 기본적으로 사용한다고 가정합니다.) 우리가 사용할 수 있는 interface는 Target이 유일하기 때문입니다. 그래서 Adaptee를 사용하기 위해서 Adapter 클래스를 새롭게 추가하였습니다. Adatpter 클래스의 역할은 단순합니다. Adaptee를 Target처럼 사용할 수 있게 감쌉니다. Adapter로 감싸진 Adaptee는 문제없이 Target처럼 사용할 수 있습니다.

# 마무리하며

Adapter Pattern은 반복해서 언급하였듯이 호환되지 않는 인터페이스를 사용하는 클라이언트를 그대로 활용할 수 있도록 해줍니다. 그 기능은 Adapter에 의해서 이루어집니다. Adaptee가 수정되어도 클라이언트에 미치는 영향은 없습니다. Adaptee는 Adapter에 의해서 캡슐화되어 있기 때문입니다. 클라이언트는 Adater에 대해서만 알지 Adaptee에 대해서는 전혀 모릅니다. 그리고 Adapter는 객체 구성(composition)을 사용합니다. Adaptee의 모든 서브 클래스는 Adapter에 의해 감쌀 수 있으므로 호환에 전혀 문제가 없습니다. Target 인터페이스를 잘 준수한다면 이를 기반으로 만들어진 클라이언트 간에 무리없이 적용할 수 있습니다. 물론 단점도 존재합니다. 만약 Target 인터페이스로 지원해야 하는 인터페이스의 크기가 커지면 Adapter 또한 필연적으로 커질 수 밖에 없습니다. 호환성을 확보하기 위해서 어쩔 수 없습니다. 그만큼 Adapter 역시 커질 수 밖에 없고, 클라이언트는 비대해진 Adapter를 사용하게 됩니다. 호환을 위해 수정한 Adater를 사용하기 위해서 클라이언트의 코드가 수정되어야 할지도 모릅니다. 이 작업도 Target 인터페이스가 크기에 비례할 것입니다. 그래서 이런 수정을 관리하는 또 다른 클래스를 만들면 어떨까 하는 생각이 들 수도 있습니다. 이는 다음에 다루게 될 Facade Pattern과 관련된 내용입니다.

참고자료: [위키피디아](https://en.wikipedia.org/wiki/Adapter_pattern), HeadFirst Design Pattern
