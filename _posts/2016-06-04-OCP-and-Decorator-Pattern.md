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

# Decorator Pattern

Decorator Pattern의 정의는 다음과 같습니다. Decorator Pattern에서는 객체에 추가적인 요건을 '동적'으로 첨가합니다. Decorator는 서브 클래스를 만드는 것을 통해서 기능을 유연하게 확장할 수 있는 방법을 제공합니다. 

![alt text](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e9/Decorator_UML_class_diagram.svg/960px-Decorator_UML_class_diagram.svg.png)

작성 중 ....
