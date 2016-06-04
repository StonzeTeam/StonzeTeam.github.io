---
layout: post
title: "OCP와 Decorator Pattern"
author: "장문익"
date: 2016-06-04 03:00:00
excerpt: "Design Pattern의 중요한 원칙들 중의 하나인 OCP를 알아봅시다."
tags: [Design Pattern, OCP, Open-Closed Principle, Decorator Pattern]
comments: true
---

앞으로 자주 공부하게 될 Design Pattern에도 중요한 원칙들이 있습니다. 이 원칙들은 각각의 Design Pattern이 추구하는 가치이기도 합니다.

# OCP

OCP는 Open-Closed Principle의 줄임말입니다. 클래스는 확장에 대해서는 열려있어야 하지만(Open), 코드 변경에 대해서는 닫혀 있어야(Closed) 한다는 의미 입니다. 즉, 기존 코드는 건드리지 않은 채로 확장을 통해서 새로운 기능을 추가할 수 있도록 하는 것입니다. 이것이 가능해진다면 우리는 기능을 유연하게 추가할 수 있으면서도 코드의 안정성 또한 유지할 수 있을 것입니다.

# OCP is not a panacea.

확장에 대해 서는 열려있고(Open) 코드 변경에 대해서는 닫혀 있다(Closed)는 말을 다시 생각해봅시다. 기존 코드를 변경하지 않으면서 어떻게 기능을 확장하라는 말인지 의아할 수 있습니다. 모든 경우에 OCP를 적용하기에는 많은 노력과 시간이 필요합니다. 아주 단순한 기능을 만들기 위해서 불필요한 노력을 할 필요는 없습니다. 모든 것을 확장의 유연한 객체지향 디자인을 하기에는 현실적인 시간이 부족한 경우가 많습니다. 그리고 OCP를 지키다 보면 새로운 단계의 추상화가 발생하여 코드를 복잡하게 만들기도 합니다. 하지만 기능을 확장하기 위해서 기존 코드 수정이 불가피한 일이 자주 발생한다면 OCP를 준수하기 위해 고민을 시작해야 할 것입니다. 생산성이 현저하게 떨어질 수 있기 때문입니다. 

# 문제 상황

우리가 카페을 운영한다고 가정합시다. 다양한 음료가 있을 것이고, 각 음료마다 만드는 방법도 다를 것이고, 가격 또한 다를 것입니다. 샷을 추가 하거나 휘핑크림을 올리면 가격을 추가될 것입니다. 이런 상황을 관리할 수 있는 프로그램을 만들어야 합니다. 음료를 클래스로 만들어 봅시다.

![alt text](https://www.dropbox.com/s/8jzejwn6g6yn0k7/960px-Decorator_UML_class_diagram.svg.png?dl=0)

그림에서 볼 수 있듯이 각각의 음료마다 추가되는 재료가 다르기 때문에 가격을 계산하는 Cost() 함수는 각각의 음료가 override해야 합니다. 얼핏 봐도 좋지 않습니다. 개선하도록 합니다. 필요한 재료를 변수로 둡니다. 그리고 Beverage 클래스는 해당 재료가 음료에 사용되었는지의 여부에 따라서 Cost() 함수에서 가격에 반영할지 하지 않을지를 결정합니다. 이제 Beverage를 상속받는 음료들은 사용되는 재료가 있는지 없는지만 flag로 관리해주면 됩니다. 위의 다이어그램보다는 확실히 간략해졌습니다. 하지만 문제는 여전합니다.

* 재료가 많아지면 Beverage에 그 재료의 Has...(), Set...() 함수들을 추가해야 합니다.
* 그에 따라 Cost() 함수도 변경되어야 합니다.
* 어떤 음료는 특정재료를 사용하지 않음에도 불구하고 모든 음료는 그 재료에 해당하는 Has...(), Set...()를 상속받을 수 밖에 없습니다.
* 만약에 특정재료를 두 개 추가하는 상황에는 어떻게 해야할까요?

위의 문제점을 현재 Beverage로 해결하려면 Beverage에 모든 기능이 추가되어야 합니다. 말 그대로 Beverage는 Super Class가 됩니다. 이제 OCP를 다시 살펴봅시다. 방금 제사한 문제점들이 해결되면 OCP가 추구하던 목표에 도달한 것입니다. 기존 코드는 건드리지 않고(Beverage는 수정하지 않고), 새로운 기능을 추가(재료를 자유롭게 추가하고, 계산도 알아서 되는)하도록 변경합시다. 이런 상황에서 Decorator Pattern은 유용하게 사용됩니다.

# Decorator Pattern

Decorator Pattern의 정의는 다음과 같습니다. Decorator Pattern에서는 객체에 추가적인 요건을 '동적'으로 첨가합니다. Decorator는 서브 클래스를 만드는 것을 통해서 기능을 유연하게 확장할 수 있는 방법을 제공합니다. 

![alt text](https://www.dropbox.com/s/8jzejwn6g6yn0k7/960px-Decorator_UML_class_diagram.svg.png?dl=0)

Decorator는 Component를 상속받습니다.
Decorator는 Component를 참조할 변수가 있습니다.
Decorator의 생성자에서 Component를 참조할 변수를 초기화할 수 있도록 전달해줍니다.
Decorator 클래스는 자신의 method를 실행하고 Component를 참조하는 변수를 통해 다른 Decrator에 접근하여 method를 실행할 수 있습니다.
ConcreteDecorator 클래스는 기능이 변경되어야 할 method를 override합니다.




작성 중 ....
