---
layout: post
title: "SmartPointer - unique_ptr"
author: "이인재"
date: 2016-06-13 03:00:00
excerpt: "소유권 독점 자원의 관리에 유용한 smart pointer"
tags: [C++, Pointer, SmartPointer, unique_ptr]
comments: true
---

unique_ptr : 소유권 독점 자원의 관리에 유용한 smart pointer

# Raw Pointer
new를 통해 생성한 객체를 일반 포인터로 관리할 때 이 포인터를 Raw Pointer라고 한다. 
다음의 예제는 class WidgetClass를 new로 동적할당하여 일반포인터 WidgetClass* pWidgetClass로 객체를 관리하는 코드이다.

{% highlight cpp %}
WidgetClass* pWidgetClass = new WidgetClass;
{% endhighlight %}

# Smart Pointer
new를 통해 생성된 객체의 메모리 관리를 할 수 있도록 Raw Pointer를 감싼 Wrapper 객체(class)이다. 
다음의 예제는 class WidgetClass를 생성하여 Smart Pointer 들인 unique pointer, shared pointer, weak pointer로 해당 객체생성한 코드이다.

{% highlight cpp %}
// unique pointer
unique_ptr<WidgetClass> pUnique_pWidgetClass(new WidgetClass);

// shared popinter
shared_ptr<WidgetClass> pShared_WidgetClass(new WidgetClass);

// weak pointer
weak_ptr<WidgetClass> pWeak_WidgetClass(pShared_WidgetClass);
{% endhighlight %}
그렇다면 Raw Pointer보다 Smart Pointer를 사용을 지향해야 하는 이유에 무엇인지 알아보도록하자.

# Raw Pointer의 문제점
* 선언만 봐서는 하나의 객체를 가리키는지 배열을 가리키는지 구분할 수 없다.  이는 직접 delete시에 단일 객체 해제 : delete, 배열 객체 해제 : delete[]와도 연관이 있다.
{% highlight cpp %}
int* a  = nullptr;
a       = new int;
{% endhighlight %}
위의 코드와 아래의 코드에서 int* a 라는 포인터 변수의 선언만으로는 단일 객체인지 배열을 가리키는지 알 수 없다.(직접 관리 필요)
{% highlight cpp %}
int* a  = nullptr;
a       = new int[100];
{% endhighlight %}
* 선언만 봐서는 포인터가 가리키는 객체가 존재하는 지에 대한 여부를 알 수 없다.
{% highlight cpp %}
int* pNum  = new int;
*pNum = 10;
delete pNum;
cout << *pNum << endl;
{% endhighlight %}
위 코드에서 이미 pNum이 가리키는 객체는 메모리에서 해제되었는데 알 길이 없어서 미정의 동작을 유발한다.

* 구체적으로 어떻게 해제해야 하는지 알 수 없다. 직접 delete를 해주어야하는지, 객체를 생성/삭제를 해주는 클래스에 위임을 해야하는 것인지 등등 객체 관리에 대해 알 수 없다. 알더라도 늘 객체의 메모리 해제시에 신경을 써주어야한다는 번거로움이 있다. 이는 실수를 유발할 수도 있고, 실수를 하게된다면 메모리릭 또는 중복 삭제 등등의 문제가 생길 수 있다.
* 객체의 해제가 코드의 모든 경로에서 정확히 한 번 일어난다는 것을 보장할 수 없다.

Raw Pointer로 동적할당/해제를 할때에는 항상 메모리릭과 잘못된 메모리 주소 참조로 인한 미정의 사태에 심혈을 기울여야 한다.

# Smart Pointer
Smart Pointer에는 다음 4종류가 있다.

* auto_ptr
* unique_ptr
* shared_ptr
* weak_ptr

Smart Pointer의 공통점으로는 동적할당한 객체에 대한 관리를 한다는 것이다. 특히 해제에 대한 관리로 인해 메모리 릭이 생기지 않도록 설계되어 있다.

참고 :  이중 auto_ptr은 엄밀히 말하면 smart pointer가 아니다. auto_ptr은 unique_ptr이 나오기 전부터 쓰여졌던 것이기 때문이다. 하지만 문제점과 사용성에 제한이 있어서 현재 c++ 표준 위원회에서는 잠정적 폐기 결론을 내렸다. 그리고 auto_ptr이 가지고 있던 기능은 unique_ptr로 모든것을 더 효율적으로 할 수 있다. auto_ptr이 가진 문제점들(예 : auto_ptr은 STL 컨테이너의 요소로 사용 못함)을 해결한것이 unique_ptr이기 때문에 auto_ptr은 사용하지 않는다.

# unique_ptr 은 가리키는 객체에 대해 "독점 + 소유" 한다.
unique_ptr은 소유한 객체 또는 배열에 대한 포인터를 저장한다. 해당 객체/배열은 다른 unique_ptr로 소유권을 이전하지 않는한 독점적으로 소유하게 된다. 그리고 객체/배열의 소멸 시점은 unique_ptr이 소멸될 때 같이 소멸된다. 즉 unique_ptr은 리소스를 고유하게 관리한다. 하나의 객체를 둘이상의 unique_ptr로 가리킬 수 없다는 뜻이다. 따라서 포인터의 복사는 불가능하며 이동만이 가능하다. 만약 복사가 가능하다면 중복으로 해제할 경우 미정의 동작에 빠지게 될 수 있기 때문이다.
{% highlight cpp %}
unique_ptr<WidgetClass> pUnique_pWidgetClass(new WidgetClass);
unique_ptr<WidgetClass> pUnique_WidgetClass2 = pUnique_pWidgetClass;
{% endhighlight %}

{% highlight cpp %}
unique_ptr<WidgetClass> pUnique_pWidgetClass(new WidgetClass);
unique_ptr<WidgetClass> pUnique_WidgetClass2;
pUnique_WidgetClass2 = pUnique_pWidgetClass;
{% endhighlight %}
위의 두 코드는 컴파일 에러가 난다. unique_ptr을 복사/대입하려고 하기 때문이다. 복사한다는 것은 소유권을 2개이상의 포인터가 가지고 있다는 얘기인데 이는 unique_ptr의 독점한다는 의도와 맞지 않기 때문이다.
따라서 포인터의 소유권을 옮기는 것만 가능하다. 이것은 std::move 를 통해서 간단하게 실행할 수 있다. std::move는 이동시맨틱을 지원하는 함수이다. (이동 시맨틱은 추후 따로 알아보도록 한다)
{% highlight cpp %}
unique_ptr<WidgetClass> pUnique_pWidgetClass(new WidgetClass);
unique_ptr<WidgetClass> pUnique_WidgetClass2 = std::move(pUnique_pWidgetClass);

//pUnique_pWidgetClass가 가리키던 객체는 pUnique_WidgetClass2가 가리키게 되며,
//기존의 pUnique_pWidgetClass는 Empty가 된다.
{% endhighlight %}

위의 내용을 그림으로 나타내면 다음과 같다.
![unique_ptr_move1](/assets/img/unique_ptr_1.png)

---

![unique_ptr_move2](/assets/img/unique_ptr_2.png)

# raw pointer로의 복사/대입 연산 불가능 
raw pointer에 unique_ptr을 move시킬 수 없다. smart pointer에서 raw pointer로의 형변환이 가능하다는 것은 다시 raw pointer의 문제점을 안고 가겠다는 의미다. 따라서 애초에 불가능하게 설계되어 있다.
{% highlight cpp %}
unique_ptr<WidgetClass, decltype(CustomDel)> pUnique_pWidgetClass(new WidgetClass, CustomDel);

WidgetClass* rawPointer = pUnique_pWidgetClass;
{% endhighlight %}
위의 코드는 컴파일 에러를 뱉는다.

# unique_ptr의 크기는 raw pointer 크기와 같다.
raw pointer를 통해서 충분한 성능이 나온다면, unique_ptr로 대체한 포인터에 대해서도 똑같은 성능을 유지할 수 있다. 그 이유는 unique_ptr은 아래에서 설명할 커스텀 삭제자가 없다면 raw pointer와 크기가 같으며 raw pointer와 대부분 같은 연산 및 명령들을 사용하기 때문이다.

# 커스텀 삭제자를 활용할 수 있다.
unique_ptr은 커스텀 삭제자를 활용할 수 있다. 이는 객체의 메모리 해제는 결정되었고, 해제 전에 커스텀하게 실행시켜야할 기능을 함수객체의 형태로 지정하는 것이다. 일반적으로는 Lambda를 활용하여 커스텀 삭제자를 지정하면 유용하게 쓸 수 있다.
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
    auto CustomDel = [](WidgetClass* pWidgetClass)
    {
              cout << "커스텀 삭제자" << endl;
              delete pWidgetClass;
    };
    unique_ptr<WidgetClass, decltype(CustomDel)> pUnique_pWidgetClass(new WidgetClass, CustomDel);

    getchar();
    return 0;
}
{% endhighlight %}

실행결과는 다음과 같다.
![unique_ptr_3](/assets/img/unique_ptr_3.png)

위의 코드는 UniquePtr 생성시 커스텀 삭제자인 lambda함수를 같이 등록한다. 객체 해제는 CustomDel이라는 커스텀 삭제자를 통해서 하고 있는 모습이다.
커스텀 삭제자를 사용할때 상태없는 함수객체(갈무리 없는 람다)를 사용한다면 unique_ptr의 크기 변화는 없다. 하지만 커스텀 삭제자가 상태를 가진 함수객체라면 해당 함수 객체에 저장된 상태의 크기만큼 unique_ptr의 크기도 증가한다.

---



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
       auto CustomDel = [](WidgetClass* pWidgetClass)
       {
                       cout << "커스텀 삭제자" << endl;
                       delete pWidgetClass;
       };

       unique_ptr<WidgetClass, decltype(CustomDel)> pUnique_pWidgetClass(new WidgetClass, CustomDel);

       cout << "갈무리 없는 lambda custom deleter 의 size =  " << sizeof(unique_ptr<WidgetClass, decltype(CustomDel)>) << endl;
       getchar();
       return 0;
}
{% endhighlight %}
위 코드처럼 갈무리 없는 람다를 사용하면 unique_ptr의 크기는 증가하지 않는다.
![unique_ptr_5](/assets/img/unique_ptr_5.png)

--



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
     int temp1 = 1; //상태를 갖는 임시 변수1
     int temp2 = 2; //상태를 갖는 임시 변수2
     auto CustomDel = [&](WidgetClass* pWidgetClass)
     {
            cout << temp1 << temp2 << endl;
            cout << "커스텀 삭제자" << endl;
            delete pWidgetClass;
     };

     unique_ptr<WidgetClass, decltype(CustomDel)> pUnique_pWidgetClass(new WidgetClass, CustomDel);

     cout << "Lambda custom deleter 의 size =  " << sizeof(unique_ptr<WidgetClass, decltype(CustomDel)>) << endl;
     getchar();
     return 0;
} 
{% endhighlight %}
위 코드를 실행하여 unique ptr의 크기를 살펴보면 다음과 같이 temp1,temp2를 갈무리한 만큼의 크기가 더해진 것을 알 수 있다.

![unique_ptr_6](/assets/img/unique_ptr_6.png)

---



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

class Function
{
public:
        Function() {};
        ~Function() {};

        void operator() (WidgetClass* pWidgetClass)
        {
             cout << "난 함수 객체" << endl;
             delete pWidgetClass;
        }
        int Temp = -1; //상태값을 테스트하기위한 임시변수
};

int main()
{
        Function FunctionObject;

        unique_ptr<WidgetClass, decltype(FunctionObject)> pUnique_pWidgetClass(new WidgetClass, FunctionObject);

        cout << "함수 객체 custom deleter 의 size =  " << sizeof(unique_ptr<WidgetClass, decltype(FunctionObject)>) << endl;
        getchar();
        return 0;
}
{% endhighlight %}
위코드를 실행해보면 함수 객체의 Temp 변수크기만큼 unique_ptr의 크기가 증가한 것을 알 수 있다.
![unique_ptr_4](/assets/img/unique_ptr_4.png)

---

커스텀 삭제자로 함수객체를 이용했다. 사이즈를 확인하면 class Function에 있는 Temp 변수의 크기만큼 unique_ptr크기도 커진것을 알 수 있다.

이처럼 상태를 가진 함수 객체, 갈무리 있는 lambda를 커스텀 삭제자로 사용할 시 unique_ptr 크기도 매우 커질 수 있으므로 주의해아한다.

# shared_ptr로의 자유로운 변환 
c++11에서 독점 소유권을 표현하는 unique_ptr은 shared_ptr로의 변환 또한 쉽고 효율적이다.
{% highlight cpp %}
unique_ptr<WidgetClass> pUnique_pWidgetClass(new WidgetClass);

shared_ptr<WidgetClass> spUnique_WidgetClass = move(pUnique_pWidgetClass);
{% endhighlight %}
위의 코드처럼 생성된 객체를 독점적으로 소유하려 하는지, 아니면 소유권을 공유하고자 하는지에 따라서 얼마든지 변환할 수 있다.
unique_ptr을 사용함으로써 프로그램의 모든 경로에서 메모리 해제가 정확히 한 번만 일어남을 보장한다. 또한 더이상 메모리 해제에 대해 raw pointer에서 만큼의 신경을 쓰지 않아도 된다.

# 정리
* (커스텀삭제자가 없다면) raw pointer와 똑같은 크기로 똑같은 성능과 똑같은 기능을 할 수 있으면서, 메모리 관리는 알아서 해준다.
* std::unique_ptr은 독점적으로 소유하여 객체를 관리하는 smart pointer
* 기본적인 메모리 해제 외에 커스텀 삭제자를 도입할 수 있다
* std::unique_ptr을 std::shared_ptr로 손쉽게 변환할 수 있다.

 참고자료: effective c++, MSDN : https://msdn.microsoft.com/ko-kr/library/hh279676.aspx
