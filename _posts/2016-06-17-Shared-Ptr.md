---
layout: post
title: "SmartPointer - shared_ptr"
author: "이인재"
date: 2016-06-17 03:00:00
excerpt: "소유권 공유자원의 관리 -  shared_ptr"
tags: [C++, Pointer, SmartPointer, shared_ptr]
comments: true
---

shared_ptr : 소유권 독점 자원의 관리에 유용한 smart pointer

# shared_ptr

앞서 스터디했던 unique_ptr이 독점 자원 관리를 하는 스마트 포인터 였다면 shared_ptr은 공유 자원을 관리하는 스마트 포인터이다. 
스마트 포인터이기 때문에 앞서 unique_ptr 자료에서 설명했던 것처럼 스마트 포인터의 장점들을 모두 갖고 있다. 
shared_ptr은 추가적으로 "자원 공유" + "해제 시점 예측 가능" 의 기능을 갖추고 있다. 
shared_ptr의 사용법은 다음과 같다.

{% highlight cpp %}
shared_ptr<T>  변수명(new T()); //T 생성하고자 하는 object 타입
{% endhighlight %}

다음 예는 WidgletClass를 shared_ptr이 관리하도록 생성한 코드이다.

{% highlight cpp %}
class WidgetClass
{
public:
                WidgetClass()
                {
                                cout << "객체 생성" << endl;
                }

                ~WidgetClass()
                {
                                cout << "객체 해제" << endl;
                }
};

int main()
{
                shared_ptr<WidgetClass> pWidgetClass(new WidgetClass);
}
{% endhighlight %}

# 소유권 공유 자원 관리 - 가리키는 객체(메모리)에 대한 "공유" 가능

shared_ptr은 소유권을 공유하는 객체에 대한 관리를 효율적으로 할 수 있는 smart pointer 이다. 
앞서 unique_ptr은 고유하게 독점 소유하는 객체를 관리하는 포인터였다. 
하지만 shared_ptr은 공유가 가능하며 이것은 unique_ptr과 shared_ptr의 가장 큰 차이점 중 하나이다. 
독점 소유하는 unique_ptr의 특징으로 포인터값 자체의 복사와 관련된 모든 연산은 불가능했다. 
하지만 shared_ptr은 객체를 공유할 수 있기 때문에 포인터의 복사와 관련된 연산이 가능하다.
  
예를들면 컨테이너(예 : vector, array 등)에 포인터를 저장했다고 해보자. 만약 unique_ptr을 저장해놓았다면 복사를 통해 각 원소들을 읽을 수 없다. 
따라서 unique_ptr은 복사가 발생하지 않도록 참조를 사용하면 가능하다. 
물론 쓸데없이 복사를 사용하는 예제일 수 있지만 unique_ptr과 shared_ptr의 공유/비공유의 관점에서 차이점을 보기위해 
다음과 같은 예제를 들었다.

{% highlight cpp %}

vector<unique_ptr<WidgetClass>> container;

container.emplace_back(unique_ptr<WidgetClass>(new WidgetClass));
container.emplace_back(unique_ptr<WidgetClass>(new WidgetClass));
container.emplace_back(unique_ptr<WidgetClass>(new WidgetClass));

for (auto Elem : container)  //unique_ptr을 Elem 에 복사하여 container 값을 읽으려고 하기 때문에 compile error
{
 //...
}

for (auto& Elem : container)  //unique_ptr 엘리먼트에 대해 Elem을 사용하여 참조하여 읽고있다. 이는 정상적으로 compile 
{
 //...
}

{% endhighlight %}

하지만 shared_ptr은 공유가 가능하기 때문에 복사를 해도 전혀 문제가 없다.

{% highlight cpp %}

vector<shared_ptr<WidgetClass>> container;

container.emplace_back(shared_ptr<WidgetClass>(new WidgetClass));
container.emplace_back(shared_ptr<WidgetClass>(new WidgetClass));
container.emplace_back(shared_ptr<WidgetClass>(new WidgetClass));

for (auto Elem : container) //share_ptr은 공유가 가능한 스마트포인터이기 때문에 정상적으로 compile
{
                //...
}

{% endhighlight %}

# 해제 시점 예측 가능

스마트 포인터는 메모리 관리를 효율적으로 도와주는 Wrapper 클래스이다. 
이 중 특히 메모리릭이 발생하지 않도록 소멸시 메모리 해제를 알아서 처리해준다. 
이러한 기능은 C#이나 자바에서는 GC(Garbage Collection)을 통해 제공한다. 
그렇다면 Garbage Collection과 차이점은 무엇일까.  
가장 큰 차이점은 "언제 메모리가 해제 및 반환" 되느냐는 것이다. 
객체가 삭제되는 시점을 프로그래머가 명시적으로 알고있음에도 불구하고 메모리가 GC의 관리를 받는다면, 
해당 메모리는 GC가 메모리를 해제할 때까지 기다려야한다. 
또한 GC가 메모리를 검사하여 해제시킬 메모리를 찾고, 해당 메모리를 해제 하는 타이밍과 점유 시간등에 대해서도 예측할 수 없다. 
이로인해  GC가 해제할 메모리 검사 및 메모리 해제시 전체 프로그램이 일시적으로 멈출 수 있다.  
반면 shared_ptr은  내부적으로 reference counting 방식을 사용하여 reference count 가 0이 되는 순간 
메모리가 해제됨을 명시적으로 알 수 있고, 개발자 입장에서 필요한 경우 의도적으로 메모리를 해제할수도 있다. 

{% highlight cpp %}

//WidgetClass 생성
shared_ptr<WidgetClass> pWidgetClass1(new WidgetClass);
//...

//필요에 의해 pWidgetClass1을 다른 곳에서도 참조
shared_ptr<WidgetClass> pWidgetClass2(pWidgetClass1);
//...

//쓸 것 다써서 이제 WidgetClass 해제
pWidgetClass1.reset();
pWidgetClass2.reset(); //reference count == 0 인순간 메모리 해제

{% endhighlight %}

# Referce Counting

shared_ptr은 어떻게 자기가 파괴되는 시점을 알 수 있을까?  
그 방법은 Reference Counting이다. shared_ptr은 shared_ptr이 관리하고있는 객체(메모리)를 참조하는 참조자들의 유효 갯수를 저장해놓는다. 
즉 해당 객체를 참조하는 shared_ptr의 갯수 라고 할 수 있다. 
shared_ptr의 생성자는 참조 카운트를 증가(아닌 경우도 있는 일반적으로는 증가 시킨다라고 알고있자), 소멸자는 감소시킨다. 
정리하면 객체를 가리키던 마지막 shared_ptr마저 해당 객체를 가리키지 않게되었을때 참조 카운트는 0이되고 
shared_ptr은 해당 객체를 메모리에서 해제한다.  

{% highlight cpp %}

shared_ptr<WidgetClass> sp1(new WidgetClass); //sp1 은 새로운 WidgetClass 객체를 가리킨다.
shared_ptr<WidgetClass> sp2(new WidgetClass); //sp2 는 새로운 WidgetClass 객체를 가리킨다.

shared_ptr<WidgetClass> sp3 = sp1;                    //sp3은 sp1이 가리키는 WidgetClass를 소유하게된다.
sp2 = sp1;                                            //sp2또한 sp1이 가리키는 WidgetCLass를 소유하게된다.

{% endhighlight %}

위의 코드를 통해 객체의 생성과 소멸이 어떻게 나타나는지 그림으로 알아보자.
참고로 smart_pointer를 raw_pointer로의 변환하는 것은 이전에도 말했지만 불가능하다. 이는 다시 raw_pointer의 문제점을 그대로 유발할 수 있기 때문이다.

![shared_ptr_1](/assets/img/shared_ptr_1.png)

------------------------------------------------------------------------

![shared_ptr_2](/assets/img/shared_ptr_2.png)

------------------------------------------------------------------------

![shared_ptr_3](/assets/img/shared_ptr_3.png)

# 참조 카운트를 증가시키지 않는 경우 - "이동"

shared_ptr의 생성자에서 참조 카운트를 증가시키지 않는 예외는 무엇일까? 바로 "move 생성자"이다. 
move는 복사가 아니라 이동이므로 sp1에서 sp2로 소유권을 이동하면 sp1은 더이상 아무 객체도 가리키게 되지 않고, 
sp2는 sp1이 가리키던 객체를 가리키게된다. 즉 복사가 아니므로 참조 카운트를 증가시키지 않는다. 

{% highlight cpp %}

shared_ptr<WidgetClass> sp1(new WidgetClass);
shared_ptr<WidgetClass> sp2 = move(sp1);

{% endhighlight %}

![shared_ptr_4](/assets/img/shared_ptr_4.png)

---------------------------------------------------------------------------

![shared_ptr_5](/assets/img/shared_ptr_5.png)

# shared_ptr의 특징

* shared_ptr의 크기는 raw pointer의 2배이다. 이는 내부적으로 실제 객체를 가리키고있는 raw pointer뿐만 아니라 자원의 참조 횟수를 가리키는 raw pointer가 더 필요하기 때문이다.
* 참조 횟수를 담을 메모리를 반드시 동적으로 할당해야 한다. 참조 카운트는 shared_ptr이 가리키는 객체와 연관되어있지만, 역으로 객체는 shared_ptr과 참조 카운트의 존재에 대해선 알지 못한다. 
* std::make_shared를 이용하여 shared_ptr을 생성하면 동적 할당의 비용을 피할 수 있다. 그러나 항상 std::make_shared를 사용할 수 있는 것은 아니다. (추후 스터디에서 make_shared는 따로 알아본다)
* 참조 횟수의 증가와 감소는 반드시 원자적이어야 한다. 멀티 스레드인 경우 동시에 해당 리소스에 접근하는 경우가 있기 때문이다. 다음과 같은 상황을 가정해보자. 
  * 스레드 A에서 리소스 R을 다 사용하고 reset을 하여 참조 카운트를 0으로 만들었고 객체의 해제가 실행되고 있다.
  * 스레드 B가 위에서 스레드 A가 R에 대한 메모리를 해제하는 도중에 R에 대한 소유권을 얻어서 참조카운트를 1로 만들었다
  * 위와 같은 상황은 멀티스레드인 경우 예상치 못한 일이 발생하기때문에 항상 원자적으로 참조 카운트를 관리해야 한다(shared_ptr의 소유권에 대한것은 원자적으로 접근해야함).

# 커스텀 삭제자를 지원한다

unique_ptr과 같이 shared_ptr은 delete를 기본으로 하되, 커스텀 삭제자를 지원한다. 
하지만 삭제자를 지원하는 방식은 unique_ptr과 다르다. 
unique_ptr에서는 삭제자의 형식이 포인터 형식의 일부지만, shared_ptr은 삭제자의 형식이 포인터 형식의 일부가 아니다. 
따라서 shared_ptr의 설계가 좀 더 유연하다고 볼 수 있다. 

{% highlight cpp %}

auto CustomDel = [](WidgetClass* pWidgetClass)
{
    cout << "커스텀 삭제자" << endl;
    delete pWidgetClass;
};
                
//unique_ptr은 unique_ptr의 형식으로 커스텀 삭제자의 형식이 있어야 한다.
unique_ptr<WidgetClass, decltype(CustomDel)> pUnique_pWidgetClass(new WidgetClass, CustomDel);

//shared_ptr은 shared_ptr의 형식에 커스텀 삭제자의 형식이 필요없다.
shared_ptr<WidgetClass> sp_WidgetClass(new WidgetClass, CustomDel);

{% endhighlight %}

또한 custom deleter의 형식이 달라지면 unique_ptr은 다른 타입이 되기 때문에 포인터의 이동연산을 할 수 없다. 
하지만 shared_ptr은 custom deleter에 따라 포인터의 타입이 바뀌는 것이 아니기 때문에 
다른 custom deleter을 사용한 shared_ptr끼리 복사/대입이 가능하다.

{% highlight cpp %}

auto CustomDel = [](WidgetClass* pWidgetClass)
{
                cout << "커스텀 삭제자1" << endl;
                delete pWidgetClass;
};

auto CustomDel2 = [](WidgetClass* pWidgetClass)
{
                cout << "커스텀 삭제자2" << endl;
                delete pWidgetClass;
};

//unique_ptr은 Custom Deleter의 형식이 다르면 다른 타입이기 때문에 move할 수 없다.
unique_ptr<WidgetClass, decltype(CustomDel)> up1(new WidgetClass, CustomDel);
unique_ptr<WidgetClass, decltype(CustomDel)> up2 = move(up1); //컴파일 에러

//shared_ptr은 shared_ptr의 형식에 커스텀 삭제자의 형식에 따라 타입이 달라지지 않기때문에 자유롭게 변환가능하다.
shared_ptr<WidgetClass> sp1(new WidgetClass, CustomDel);
shared_ptr<WidgetClass> sp2(new WidgetClass, CustomDel2);
sp2 = sp1; //문제없이 컴파일 잘됨

{% endhighlight %}

또 다른 unique_ptr과의 차이점은 커스텀 삭제자를 지정해도 shared_ptr 객체의 크기는 변하지 않는다는 점이다. 
즉, 삭제자가 어떻든 shared_ptr은 항상 포인터 2개 크기이다. 
물론 실제로 메모리를 더 사용할 수 있다. 단지 그 메모리는 shared_ptr의 것이 아니다. 
그래서 shared_ptr의 크기는 추가적으로 증가하지 않는다. 
그러나 힙에서 추가적으로 할당되며, 그 메모리는 shared_ptr의 커스텀 할당자 지원을 활용하는 경우 그 할당자가 관리하는 메모리가 사용된다.

# Control Block

그렇다면 reference count에 대한 정보는 어디에 있을까? 
사실 shared_ptr은 reference count를 비롯한 raw pointer가 하지 못하는 기능들을 하기위한 자료구조를 갖고 있는데 
이 자료구조를 Control Block이라고 한다. 위에서 설명한 커스텀 삭제자 또한 이 control blok에 복사되어 저장된다. 
또한 커스텀 할당자를 별도로 만들었다면 그 복사본 또한 이 control block에 저장된다. 
더불어 후에 설명할 weak_ptr (약포인터) 의 참조 횟수도 이 control block에 저장이된다. 

![shared_ptr_6](/assets/img/shared_ptr_6.png)

객체의 control block(이하 제어 블록) 은 그 객체를 가리키는 최초의 std::shared_ptr가 생성될 때 설정된다. 
일반적으로 어떤 객체에 대한 shared_ptr을 생성하는 코드에서 
그 객체를 가리키는 다른 shared_ptr이 이미 존재하는지(즉 제어블록이 이미 존재하는가)에 대해서 알아내는 것은 불가하지만 
제어 블록의 생성 여부에 관해서는 다음과 같은 규칙들을 적용시켜보고, 유추할 수 있다.

* make_shared(추후 공부할 것) 는 항상 제어 블록을 생성한다. 이 함수는 shared_ptr이 가리킬 객체를 새로 생성한다. make_shared 가 호출되는 시점에서 그 객체에 대한 제어 블록이 이미 존재할 가능성은 전혀 없다. 
* unique_ptr로부터 shared_ptr객체를 생성하면 제어블록이 생성된다. unique_ptr은 독점 소유 이므로 제어블록이 전혀 생성될 필요가없다. 또한 unique_ptr은 커스텀 삭제자가 아니라면 raw_pointer와 크기가 같으므로 제어블록을 가리키는 pointer 또한 없다. 하지만 이를 shared_ptr객체로 관리한다고 한다면 당연히 제어블록이 필요하므로 생성하게 된다. 덧붙여 shared_ptr로 unique_ptr이 가리키던 객체를 관리하게 하면 unique_ptr은 소유권을 이전했으므로 empty가된다.
* raw_pointer로 shared_ptr 생성자를 호출하면 제어 블록이 생성된다. 이미 제어 블록이 있는 객체로부터 shared_ptr을 생성하고 싶다면 raw_pointer가 아니라 shared_ptr 혹은 추후에 스터디할 weak_ptr를 생성자의 인수로 지정하면 된다. shared_ptr이나 weak_ptr을 생성자의 인수로 받는 shared_ptr은 새 제어 블록을 만들지 않는다. 이미 인수로 전달된 포인터들이 제어블록을 가리키고 있을 것이기 때문이다.

위의 내용을 토대로 하나의 생 포인터로 여러개의 shared_ptr을 생성하면 해당 객체에 여러 개의 제어 블록이 만들어지므로 미정의 행동을 야기하게된다. 
제어 블록이 여러개라는 것은 참조 횟수를 관리하는 곳이 여러개라는 뜻이며, 참조 횟수를 관리하는 곳이 여러개 라는 것은 해당 객체가 여러 번 파괴된다는 뜻이다. 
당연히 객체의 삭제를 중복으로 하게되면 미정의 사태에 빠진다.

{% highlight cpp %}

//raw pointer를 통해 WidgetClass 생성
auto pW = new WidgetClass;

//pW에 대한 제어 블록이 생성됨
shared_ptr<WidgetClass> spW1(pW);

//pw에 대해서 새로운 제어 블록이 생성됨
shared_ptr<WidgetClass> spW2(pW);

{% endhighlight %}

![shared_ptr_7](/assets/img/shared_ptr_7.png)

--------------------------------------------------------------------

![shared_ptr_8](/assets/img/shared_ptr_8.png)

--------------------------------------------------------------------

![shared_ptr_9](/assets/img/shared_ptr_9.png)

--------------------------------------------------------------------

![shared_ptr_10](/assets/img/shared_ptr_10.png)

--------------------------------------------------------------------

![shared_ptr_11](/assets/img/shared_ptr_11.png)

애초에 동적으로 할당된 객체를 가리키는 raw_pointer를 만들고 이를 다시 shared_ptr이 관리하도록 하는 것 자체가 잘못되었다. 
애초에 shared_ptr를 사용하여 동적할당된 객체를 관리해야한다.  
혹은 shared_ptr의 생성자를 호출할때 raw_pointer를 넘겨주는 짓은 하지 말아야 한다. 
또한 shared_ptr의 생성자를 직접 호출하는 것이 아니라 make_shared(추후 알아본다)를 사용하는 방법도 있다. 
(단 make_shared를 사용하면 커스텀 삭제자를 사용하지 못한다는 점은 있다.)   
이래나 저래나 동적 할당을 할때 애초에 shared_ptr로 해주면 자연스럽게 문제를 해결할 수 있다.

{% highlight cpp %}

//spW1에 대한 제어 블록이 생성됨
shared_ptr<WidgetClass> spW1(new WidgetClass);

//spW2는 spW1을 초기화인수로 사용하여 복사 생성자를 호출한다.
//즉, spW2의 인수로 넘긴 spW1이 shared_ptr이 므로 새로운 제어블록을 생성하지 않고,
//같은 제어블록을 사용한다.
shared_ptr<WidgetClass> spW2(spW1);

{% endhighlight %}

# this 포인터와 shared_ptr

먼저 이 코드를 살펴보자. 이 코드는 처리된 WidgetClass들을 관리하는 vector가 전역 변수로 선언되어있다. 
그리고 처리가 끝난 위젯들을 vector에 추가하여 관리하고자 하는 코드다
( 예시를 위한 코드이니, 전역 변수로 vector를 선언한 것에 대해서는 비적절하다고 판단할 수 있다)

{% highlight cpp  linenos %}

class WidgetClass;
vector<shared_ptr<WidgetClass>> ProcessedWidgets;

class WidgetClass
{
public:
       WidgetClass()
       {
              cout << "객체 생성" << endl;
       }

       ~WidgetClass()
       {
              cout << "객체 해제" << endl;
       }

       

       void ProcessWidget()
       {
              //...

              //shared_ptr의 생성자로 this를 넘긴다. 즉 제어블록이 하나 생긴다.
              ProcessedWidgets.emplace_back(this);
       }
};



int main()
{
       //pW에 대한 제어 블록이 생성됨
       shared_ptr<WidgetClass> spW1( new WidgetClass );

       spW1->ProcessWidget();
       //.....
       
       //처리된 Widget들을 비워줌
       ProcessedWidgets.clear();

       getchar();
       return 0;
}

{% endhighlight %}

이 코드의 문제점은 무엇일까? raw_pointer를 넘겨 줬을때 문제가 될 수 있었던 부분을 똑같이 안고있다.

* spW1이 새로운 WidgetClass (이하 A 객체 라고 하겠다)를 관리한다.
* ProcessWidget이라는 method에서 this pointer를 shared_ptr의 생성자 인자로 넘기기 때문에 제어블록이 생성된다. 즉 A객체에 대한 제어 블록이 추가로 생성되는 것이다.
* ProcessedWidget.clear()를 통해 처리된 Widget들을 모두 삭제한다.
* 이때 A객체 또한 Clear가 되면서 참조 카운트가 0이 되어 삭제된다.
* main이 끝나고, spW1또한 참조카운트가 0이 되어 이미 삭제된 A객체를 또 삭제하려고 한다.

즉, 중복 삭제의 문제를 고스란히 안고잇는 것이다.  
shared_ptr로 관리하는 클래스를 작성할 때, 해당 클래스의 this 포인터로부터 shared_ptr을 안전하게 생성하려면 
enable_shared_from_this를 상속하게 만들어야한다. 위의 코드에서는 Widget Class를 다음과 같이 만들면 된다.

{% highlight cpp  %}

class WidgetClass : public enable_shared_from_this<WidgetClass>
{
//...
};

{% endhighlight %}

enable_shared_from_this는 현재 객체를 가리키는 shared_ptr를 생성하되 제어 블록을 복제하지는 않는 멤버 함수 하나를 정의한다. 
내부적으로 확인해보면 weak_ptr로 shared_ptr의 생성한다. 즉 참조 횟수 증가와 제어 블록 복제를 하지 않는다.
그 멤버 함수는 shared_from_this이다. this 포인터와 같은 객체를 가리키는 shared_ptr이 필요할때 이 멤버를 사용한다. 
즉 위의 this 포인터를 shared_ptr의 인자로 넘겨주는 ProcessWidget() 메서드를 다음과 같이 바꿔주면 문제를 해결할 수 있다.

{% highlight cpp  %}

void ProcessWidget()
{
     //...
     ProcessedWidgets.emplace_back(shared_from_this());
}

{% endhighlight %}

내부적으로 shared_from_this는 현재 객체에 대한 제어 블록을 조회하고, 그 제어 블록을 지칭하는 새 std::shared_ptr을 생성한다.
(제어 블록의 재생성이 아니다). 이러한 설계에 있어서 주의해야 하는 점이 있다. 
shared_from_this()를 호출하기 위해서는 이미 이 객체를 가리키는 shared_ptr이 존재 해야만한다. 
그런 shared_ptr가 존재하지 않는다면(즉, 이 객체에 대한 제어블록이 없다는 말과 동일) 미정의 행동을 유발한다. 
다음의 코드는 WidgetClass를 raw_pointer가 관리하도록 바꾼 소스코드다. 이 코드는 미정의 행동을 유발한다.

{% highlight cpp  linenos  %}

class WidgetClass;
vector<shared_ptr<WidgetClass>> ProcessedWidgets;

class WidgetClass : public enable_shared_from_this<WidgetClass>
{
public:
       WidgetClass()
       {
              cout << "객체 생성" << endl;
       }

       ~WidgetClass()
       {
              cout << "객체 해제" << endl;
       }



       void ProcessWidget()
       {
             //...

             // this가 가리키는 객체를 관리하는 shared_ptr이 존재하지 않는다(제어 블록이 존재하지않음)
             // 따라서 shared_from_this는 미정의 행동을 유발한다.
             ProcessedWidgets.emplace_back(shared_from_this());
       }
};



int main()
{
       //raw pointer로 클래스를 관리한다.
       auto pW1 = new WidgetClass;
       pW1->ProcessWidget();
       //.....

       //처리된 Widget들을 비워줌.
       ProcessedWidgets.clear();

       getchar();
       return 0;
}

{% endhighlight %}

# shared_ptr 특징 정리

shared_ptr은 자신의 기능을 위해서 제어 블록이라는 것을 갖는다. 
제어 블록의 크기는 shared_ptr이 추가로 필요한 정보들에 의해 가변적이다. 
보통은 몇 워드 크기이지만 커스텀 삭제자나 할당자 때문에 그보다 더 커질수 있다. 
그리고 shared_ptr의 역참조 비용은 raw_pointer의 역참조와 다르지 않다. 
참조 횟수에 대한 변경의 명령은 원자적 연산 한두개가 더 필요하지만 각 연산은 기계어 하나의 명령에 대응된다. 
만약에 shared_ptr의 사용, 즉 제어 블록에 대한 크기 및 연산 비용등에 대해 부담이 있는 프로그램일 경우 
소유권 공유가 꼭 필요한지 생각해야한다. 그렇지 않다면 unique_ptr로도 충분할 가능성이 있기 때문이다. 
앞서 스터디 자료에서 언급했지만 unique_ptr은 생포인터와 크기도, 연산비용도 거의 같다. 
설령 후에 공유가 필요하다면 shared_ptr로의 업그레이드도 손쉽게 가능하기 때문에 부담없이 사용할 수 있다. 
하지만 그 역은 반대다. shared_ptr로 관리를 하다가 공유가 필요없다는 것을 알게되었어도 그것을 unique_ptr로 변환할 수 없다.

{% highlight cpp  %}

//이미 shared_ptr로의 관리를 받는 객체에 대해서는 unique_ptr로의 관리 변환이 불가능하다.
shared_ptr<WidgetClass> spW1(new WidgetClass);
unique_ptr<WidgetClass> upW1(spW1);         //unique_ptr의 복사생성 불가능
unique_ptr<WidgetClass> upW1 = move(spW1);  //unique_ptr로의 이동연산도 불가능

{% endhighlight %}

또한 shared_ptr은 배열 관리를 하지 못한다. 
shared_ptr은 단일 객체를 가리키는 포인터를 염두에 두고 설계되었기 때문이다. 즉 shared_ptr<T[]> 같은 것은 사용할 수 없다. 
커스텀 삭제자를 이용하여 배열을 관리하는 shared_ptr을 만드려고 하지말아야한다. 
관리적인 측면이나, []에 대한 operator가 없기 때문이다. 
따라서 std::array 이나 std::vector 등등을 사용하여 이 컨테이너들이 shared_ptr을 관리하도록 하는것이 좋다.

# 정리

* shared_ptr은 공유할 수 있는 자원에 대한 관리를 하는 smart pointer
* shared_ptr 객체는 그 크기가 unique_ptr의 두 배이며 제어 블록에 관련되어 추가적은 크기가 있을 수 있다. 또한 참조 횟수는 원자적 연산을 요구한다.
* 커스텀 삭제자를 지원하며 삭제자의 형식은 shared_ptr의 타입에 영향을 미치지 않는다.
* 제어 블록의 복사에 주의해야하며 이러한 복사를 방지하기위해 raw_pointer로 shared_ptr을 생성할 때에는 주의해야 한다.

참고자료: effective c++, http://stackoverflow.com/questions/4663385/garbage-collection-vs-shared-pointers, https://ko.wikipedia.org/wiki/%EC%93%B0%EB%A0%88%EA%B8%B0_%EC%88%98%EC%A7%91_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)
