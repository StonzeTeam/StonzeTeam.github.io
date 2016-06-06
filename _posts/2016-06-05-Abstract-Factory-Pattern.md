---
layout: post
title: "AbstractFactoryPattern"
author: "이인재"
date: 2016-06-05 03:00:00
excerpt: "객체군들의 생성에 유용한 AbstractFactoryPattern"
tags: [Design Pattern, 객체생성, Factory Pattern]
comments: true
---

특정 조건이나 상황에 따라 알맞은 객체들을 생성하는데 유용한 패턴

# 제품군 생성

제품군 : 하나의 제품을 이루는 객체들의 집합
일상생활에서도 제품군에 대한 예를 매우 많이 접할 수 있다. 자동차를 보자. 자동차의 제품군은 무엇일까? 모든 차가 그런 것은 아니겠지만 대부분의 자동차는 엔진, 기어, 브레이크, 핸들, 바퀴 등등의 공통적인 구성품들로 이루어져 있다.
이때 자동차를 이루는 공통적인 구성품들의 집합을 제품군으로 볼 수 있다.  
그런데 자동차가 엔진 기어, 브레이크, 핸들, 바퀴를 다 갖추고 있다고해도 스포츠가에 쓰이는 부품들과 티코에 쓰이는 부품들은 다를 것이다.  
스포츠카에 들어가는 제품군과 티코에 들어가는 제품군은 다르다.
![자동차를 이루는 제품군](/assets/img/1.png)  

# Abstract Factory Pattern

Abstract Factory Pattern은 이런 제품군을 생성할때 유용하게 쓰이는 패턴이다. 쉽게말해 스포츠카는 스포츠카에 맞는 제품군을 생성해주고, 티코는 티코에 맞는 제품군을 생성해주는 것이다. 이 패턴의 장점은 다음고 같다.
* 자기한테 맞는 제품군을 생성한다는 것을 보장한다.
* 자기한테 맞는 제품군의 생성을 독립적으로 할 수 있다.  

스타1을 예로들어보자. 스타에서는 첫 시작시 각 종족이 무엇이냐에 따라 본진과 일꾼이 생성된다. 이때 제품을 종족으로 보고 제품군을 본진, 일꾼으로 생각할 수 있다. 내가 저그로 게임을 하는데 해처리와 SCV가 나와버린다면 안된다... 저그를 했으면 해처리와 드론이 알맞게 생성되어야한다.  

![스타1 - 해처리에 SCV?](/assets/img/2.png)  

그렇다면 종족이 결정되었을때 각 종족에 맞는 본진과 일꾼을 어떻게 생성해야할까?  
또한 각 종족에 맞게 알맞은 본진과 일꾼을 생성한다는 것을 어떻게 보장할 수 있을까?  

![스타1 - 저그면 해처리와 드론이 나와야함](/assets/img/3.png)  

# Enum 및 Base Class 정의

Sepicies      : 각 종족을 나타내는 Enum값  
Worker        : 일꾼을 나타내는 interface class(추상클래스)  
Headquarters  : 본진을 나타내는 interface class(base class)    

{% highlight cpp %}
#include <iostream>
 
using namespace std;
 
 
enum Species
{
    Zerg ,
    Terran ,
    Protoss ,
    None ,
};
 
class Worker
{
public:
    virtual void Work( ) = 0;
};
 
class Headquarters
{
};
 
class Hatchery : public Headquarters
{
public:
    Hatchery( )
    {
        cout << "내 본진은 해처리" << endl;
    }
};
 
class CommandCenter : public Headquarters
{
public:
    CommandCenter( )
    {
        cout << "내 본진은 커맨드 센터" << endl;
    }
};
 
class Nexus : public Headquarters
{
public:
    Nexus( )
    {
        cout << "내 본진은 넥서스" << endl;
    }
};
 
class Drone : public Worker
{
public:
    Drone( )
    {
        cout << " 난 드론 " << endl;
    }
 
    void Work( ) override
    {
        cout << "집게질 열심히해서 미네랄 캐자" << endl;
    }
};
 
class Scv : public Worker
{
public:
    Scv( )
    {
        cout << "난  SCV" << endl;
    }
 
    void Work( ) override
    {
        cout << "드릴 질 열심히 해서 미네랄 캐자" << endl;
    }
};
 
class Probe : public Worker
{
public:
    Probe( )
    {
        cout << "난 Probe" << endl;
    }
 
    void Work( ) override
    {
        cout << "레이져 질 열심히 해서 미네랄 캐자" << endl;
    }
};


{% endhighlight %}

# 타입 조건 체크를 통한 직접 생성

{% highlight cpp %}
Headquarters* CreateHeadquarters( Species MySpecies)
{
    switch ( MySpecies )
    {
    case Species::Zerg:
        return new Hatchery( );
        break;
 
    case Species::Terran:
        return new CommandCenter( );
        break;
 
    case Species::Protoss:
        return new Nexus( );
        break;
 
    default:
        break;
    }
 
    return nullptr;
}
 
Worker* CreateWorker( Species MySpecies )
{
    switch ( MySpecies )
    {
    case Species::Zerg:
        return new Drone( );
        break;
 
    case Species::Terran:
        return new Scv( );
        break;
 
    case Species::Protoss:
        return new Probe( );
        break;
 
    default:
        break;
    }
 
    return nullptr;
}
 
 
int main( void )
{
    Species         MySpecies       = Species::Zerg;
    Headquarters*   MyHeadquarters  = CreateHeadquarters( MySpecies );
    Worker*         MyWorker        = CreateWorker(MySpecies);
 
    getchar( );
    return 0;
}
{% endhighlight %}

직접 타입체크를 통한 코드의 장단점은 무엇일까?  
<장점>
* 매우 직관적이고 쉽다.  
  
<단점>
* 직접 생성하는 코드이므로 후에 추가, 수정, 변경, 삭제 등이 필요할때 해당 구현부분을 직접 찾아서 + 고쳐주어야한다.
  * 제4의 종족이 추가된다면 각각 본진과 일꾼을 생성해주는 코드를 찾아서 그 부분을 고쳐주어야한다. 
* 하드코딩으로 생성하기 때문에 다른곳에서 객체 생성이 필요할시 코드의 중복이 발생한다.
  * melee, free for all, team 모드 등에서 각각 해당 코드가 중복될 수 있다.   
* 위의 2가지의 단점으로 인해 실수발생 여지가 크며, 디버깅이 어렵다.
단순히 객체 생성하는 코드만 있어서 왜 이러한 단점들이 크게 문제가 될까? 라고 생각하기 쉽다. 하지만 실제 코드는 훨씬 방대할 수 있으며 객체의 생성은 매우 다양한 곳에서 빈번하게 사용되어진다는 것을 감안해야한다. 즉 코드의 전체적인 부분에서 파편적으로 객체의 생성이 빈번하게 호출되면 해당 부분을 찾는것도, 해당 부분을 찾아서 고치는 것은 더욱더 부담이 되는 일일 수 있다.  

이를 그림으로 표현하면 전혀 캡슐화 및 정보은닉화가 되어있지않은 날것의 느낌일 것이다.  
![직접 타입체크를 통한 객체 생성](/assets/img/4.png)

# 객체 생성 전담 클래스 활용

조건 체크를 통한 직접생성의 단점을 극복하려면 어떤 방법이 있을까?  
문제점이 생긴 이유는 변경될 가능성이 큰 부분이 여기저기에 퍼져있다는 것이다. OOP프로그래밍에서 캡슐화가 되어있지 않음을 의미한다. 캡슐화가 되어있지않으면 변경에 유연하지 않으며 정보은닉 또한 지켜질 수 없는 상황이 발생한다(직접 코딩된 곳을 보면 각 종족에 따라 어떤 객체들을 생성하는지 구체적인 내용들을 훤히 알 수 있다). 즉 변경될 가능성이 큰 부분을 한곳으로 모아 캡슐화하면 코드의 중복을 제거할 수 있으며, 파편화된 곳을 직접 전부 찾아 수정하는 문제점을 해결할 수 있다.   
객체지향 설계에서 변경이 잦은 부분을 따로 캡슐화하여 관리할 수 있게 하는 강력한 도구로 클래스를 종종 이용한다. 변경될 가능성이 많은 부분에 대하여 클래스 내부로 숨겨 정보은닉의 효과 또한 가져올 수 있다. 물론 변경이 발생해도 해당 클래스의 내부만 고쳐주면 되므로 변경에도 유연하게 대처할 수 있다. 코드로 살펴보자.

{% highlight cpp %}
class GameStartFactory
{
public:
    GameStartFactory( ){}
 
    static Headquarters* CreateHeadquarters( Species MySpecies )
    {
        switch ( MySpecies )
        {
        case Species::Zerg:
            return new Hatchery( );
            break;
 
        case Species::Terran:
            return new CommandCenter( );
            break;
 
        case Species::Protoss:
            return new Nexus( );
            break;
 
        default:
            break;
        }
 
        return nullptr;
    }
 
    static Worker* CreateWorker( Species MySpecies )
    {
        switch ( MySpecies )
        {
        case Species::Zerg:
            return new Drone( );
            break;
 
        case Species::Terran:
            return new Scv( );
            break;
 
        case Species::Protoss:
            return new Probe( );
            break;
 
        default:
            break;
        }
 
        return nullptr;
    }
 
};
 
 
 
int main( void )
{
    Species         MySpecies       = Species::Zerg;
    Headquarters*   MyHeadquarters  = GameStartFactory::CreateHeadquarters( MySpecies );
    Worker*         MyWorker        = GameStartFactory::CreateWorker( MySpecies );
 
    getchar( );
    return 0;
}
{% endhighlight %}  

위와 같이 객체 생성을 전담하는 클래스를 만들어서 객체생성을 관리하면 된다. 위와 같이 객체 생성 클래스를 "Simple Factory"라고 하며 이것은 Pattern은 아니지만 간단한 객체 생성에 있어서 종종 쓰이는 기법이다. 즉 심플하게 하나의 클래스에서 객체생성을 담당하겠다라는 것이다. 같은 의미로 Static Function Library등을 만들어서 관리하기도 한다.  

<장점>
* 하드코딩된 객체 생성 코드가 여기저기 중복으로 사용되는 것을 해결
* 변경될 가능성이 많은 부분을 캡슐화하여 유지보수 및 변경에 유연하게 대처가능
* 캡슐화를 통한 정보은닉성을 얻음  

<단점>
* 여전히 타입에 따른 조건 체크의 중복이 발생(종족과 일꾼 2개에서 종족체크를 중복 으로 계속 함)
* 종족이 추가된다면 종족과 일꾼에 각각 조건 체크 부분을 추가해야한다.  

그림으로 표현하면 캡슐화와 은닉화되어서 객체를 생성하는 이정도의 느낌일 것이다.  
![Simple Factory](/assets/img/5.png)    

직접 조건 체크를 했을때의 문제점을 해결했지만, 아직까지 조건 체크의 중복에 대한 문제점은 여전히 안고있다.

# Abstract Factory 패턴을 통한 객체 생성

패턴에 대해 알아보기전에 클래스 상속의 유의미한 점에 대해서 먼저 얘기해보자. 클래스 상속이 유의미하게 되는 경우는 크게 2가지ㄹ를 생각해 볼 수 있다.  
* 상위 클래스가 하위 클래스의 공통의 데이터 및 기능을 이용.
* 상위 클래스를 통한 인터페이스 제공, 하위 클래스(자식 클래스) 에서 인터페이스에 대해 구체적인 구현.
  
후자의 경우 다형성을 이용하여 OCP의 원칙에 맞는 구현을 할 수 있다. OCP에 대해서는 팀원 장문익님의 OCP와 decorator패턴에서 확인할 수 있다. 즉 상위 클래스의 인터페이스는 변경하지 않고, 하위 클래스에서 구체적인 구현에 대해 얼마든지 변경할 수 있다. 즉 구체적인 구현에 따라 상위 클래스의 인터페이스는 영향을 받지 않기때문에 수정에는 닫혀있고 확장에는 열려있는 구조가 된다.  
Abstract Factory 패턴은 이런 클래스 상속의 장점을 활용하는 패턴이다. 각 제품군을 생성하는 클래스가 존재하고, 이 클래스들은 새로운 제품군 생성에 대해 Open된 상태를 유지하기위해 상속구조를 이용한다. 이때 상속받는 상위 클래스는 Abstract Class(추상클래스로 객체생성 인터페이스 제공)가 된다.  
* 저그로 게임스타트시 해처리 + 드론을 생성해주는 클래스
* 테란으로 게임 스타트시 커맨드센터 + SCV를 생성해주는 클래스
* 프로토스로 게임 스타트시 넥서스 + 프로브 를 생성해주는 클래스
* 위와 3가지 제품군 생성 클래스는 상위 추상 클래스를 상속받아 추상 클래스의 객체생성 인터페이스를 구현
  
명시적으로 코드에서 확인해보도록 하자.
* AbstractGameStartFactory  : 각 종족 제품군의 객체 생성 interaface를 제공하는 추상 클래스  
* ZergGameStartFactory      : 저그 제품군 객체 생성 구현 클래스  
* TerranGameStartFactory    : 테란 제품군 객체 생성 구현 클래스  
* ProtossGameStartFactory   : 프로토스 제품군 객체 생성 구현 클래스    

{% highlight cpp %}
#include <iostream>

using namespace std;


enum Species
{
	Zerg ,
	Terran ,
	Protoss ,
	None ,
};


class Worker
{
public:
	virtual void Work( ) = 0;
};

class Headquarters
{
};

class Hatchery : public Headquarters
{
public:
	Hatchery( )
	{
		cout << "내 본진은 해처리" << endl;
	}
};

class CommandCenter : public Headquarters
{
public:
	CommandCenter( )
	{
		cout << "내 본진은 커맨드 센터" << endl;
	}
};

class Nexus : public Headquarters
{
public:
	Nexus( )
	{
		cout << "내 본진은 넥서스" << endl;
	}
};

class Drone : public Worker
{
public:
	Drone( )
	{
		cout << " 난 드론 " << endl;
	}

	void Work( ) override
	{
		cout << "집게질 열심히해서 미네랄 캐자" << endl;
	}
};

class Scv : public Worker
{
public:
	Scv( )
	{
		cout << "난  SCV" << endl;
	}

	void Work( ) override
	{
		cout << "드릴 질 열심히 해서 미네랄 캐자" << endl;
	}
};

class Probe : public Worker
{
public:
	Probe( )
	{
		cout << "난 Probe" << endl;
	}

	void Work( ) override
	{
		cout << "레이져 질 열심히 해서 미네랄 캐자" << endl;
	}
};

class AbstractGameStartFactory
{
public:
	virtual Headquarters* CreateHeadquarters( ) = 0;
	virtual Worker* CreateWorker( ) = 0;
};

class ZergGameStartFactory : public AbstractGameStartFactory
{
public:
	ZergGameStartFactory( ) {}

	Headquarters* CreateHeadquarters( ) override
	{
		return new Hatchery( );
	}

	Worker* CreateWorker( ) override
	{
		return new Drone( );
	}
};

class TerranGameStartFactory : public AbstractGameStartFactory
{
public:
	TerranGameStartFactory( ) {}

	Headquarters* CreateHeadquarters( ) override
	{
		return new CommandCenter( );
	}

	Worker* CreateWorker( ) override
	{
		return new Scv( );
	}
};

class ProtossGameStartFactory : public AbstractGameStartFactory
{
public:
	ProtossGameStartFactory( ) {}

	Headquarters* CreateHeadquarters( ) override
	{
		return new Nexus( );
	}

	Worker* CreateWorker( ) override
	{
		return new Probe( );
	}
};



int main( void )
{
	Species							MySpecies		= Species::Zerg;
	AbstractGameStartFactory*		GameFactory		= nullptr;
	Headquarters*					MyHeadquarters	= nullptr;
	Worker*							MyWorker		= nullptr;

	switch(MySpecies)
	{
	case Species::Zerg:
		GameFactory		= new ZergGameStartFactory( );
		break;
	case Species::Terran:
		GameFactory = new TerranGameStartFactory( );
		break;
	case Species::Protoss:
		GameFactory = new ProtossGameStartFactory( );
		break;
	default:
		break;
	}

	MyHeadquarters	= GameFactory->CreateHeadquarters( );
	MyWorker		= GameFactory->CreateWorker( );

	getchar( );
	return 0;
}
{% endhighlight %}  

위와 같이 매우 직관적으로 객체생성을 할 수 있다. 각 종족은 각 종족에 맞는 본진과 일꾼을 그냥 생성만 해주면 된다.  
<장점>
* 직접 조건 활용이나 객체 전담 클래스에서 필요했던 타입에 따른 조건체크가 필요없음
* 자기 자신이 무엇을 생성할지에만 집중하면된다. 기존의 직접 조건 활용이나 객체전담 클래스에서의 코드 수정 및 변경이 불필요하다.
* 조건에 맞는 Factory Class가 있다면 조건 체크는 할필요도 없다.
* 기존의 코드의 수정없이 독립적으로 객체 생성에 대한 수정 및 추가가 가능하다.  

<단점>
* 제 4의 종족이 추가되려면 Factory Class 자체가 늘어난다.
* 제 5, 6, 7 ...종족이 늘어날수록 Factory Class 자체의 갯수도 늘어난다.
* Abstrac Factory 자체의 인터페이스가 변경되면 각 구현 Factory Class에서 변경된 인터페이스의 추가 및 수정을 해주어야한다.  
  * 예를들어 게임 시작기 테란은 짐레이너, 저그는 캐리건, 프로토스는 테사다 를 영웅으로 준다고 룰이 바뀐다면?

Abstract Factory Pattern을 그림으로 나타내면 각각에 맞는 객체 생성 공장을 가지고 있고, 이 공장은 캡슐화되어있음을 의미한다.  
![Abstract Factory Pattern](/assets/img/6.png)  

Abstract Pattern은 객체 생성에 매우 유리한 클래스이나 단점 또한 가지고 있다. 따라서 단점을 잘 파악한뒤 직접 조건 체크를 사용할지, Simple Factory 기법을 사용할지, Abstract Factory Pattern을 사용할지 결정해야 한다.  
물론 위와같은 단점들을 극복하기위한 기법들도 존재한다.  
<단점 극복 방안>
* 객체 생성을위한 클래스는 인스턴스가 여러개일 필요가없다. 따라서 Singleton 패턴을 결합하여 고유한 클래스로 만들어 주면 좋다.
* Factory Class 자체의 갯수가 늘어나는 단점은 Prototype pattern을 결합하여 해결가능하다. 하지만 Prototype pattern을 사용하면 Factory Class에서 생성하는 모든 인스턴스를 가지고 있어야 하므로 메모리 부담이 있을 수 있다. 또한 인스턴스를 제대로 정의하지 않으면 제품군에 맞는 객체들의 생성을 보장할 수 없다.
* Factory Class의 인터페이스 변경시 모든 Factory Class의 변경은 다형성과 Prototype pattern을 사용하여 해결할 수 있다. 이에 대한 자세한 사항은 Prototype pattern에 대해 공부후 추후 알아보도록 한다.  

# 정리

* 특정 조건이나 환경에 따른 객체군들의 생성에 유리한 패턴
* 여러 조건이나 환경에 따라 알맞은 객체군들의 생성을 보장
* 구체적인 Interface를 사용하여 객체군들을 생성하므로 캡슐화 및 정보은닉이 가능함
* 상황에 따라 SimpleFactory 패턴이 유용한 경우도 있다. 오버 엔지니어링이 되지 않도록 적절한 상황판단과 설계 결정이 필요하다.
  
참고자료: GOF 디자인 패턴, HeadFirst Design Pattern, Google 신
