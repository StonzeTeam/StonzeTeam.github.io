---
layout: post
title: "고루틴은 어떻게 동작하는가?"
author: "김려은, 이선협"
date: 2016-05-30 03:00:00
excerpt: "http://blog.nindalf.com/how-goroutines-work/를 번역했습니다."
tags: []
comments: true
---

이번 스터디에서는 Goroutine의 동작 방법에 대해서 조사했습니다. 좀 더 많은 자료가 있지만 자세한 내용은 정리해서 올릴 예정입니다.

출처: http://blog.nindalf.com/how-goroutines-work/

# 서론

만약 당신이 Go언어를 배우는 것이 처음이거나 "동시성은 병렬 구조가 아니다"라는 문장이 당신에게 의미가 없다면, Rob Pike의 그 주제에 대한 [뛰어난 논의](http://www.youtube.com/watch?v=cN_DpYBzKso)를 확인해라. 이것은 30분 정도 소요될 것이고, 나는 그것을 보는 것이 30분을 유용하게 소비하는 거라는 걸 보증한다.

차이점을 요약하자면, 사람들이 동시성이라는 단어를 들었을 때 그들은 흔히 관련된 단어로 병렬 구조를 떠올린다. 하지만 둘은 서로 아주 분명히 **다른 개념**이다. 프로그래밍에서, 동시성은 독립적으로 과정들을 수행하는 구성 요소이다. 반면에, 병렬 구조는 (가능한 한 관련된) 계산의 동시 실행이다. 동시성은 많은 것들을 한꺼번에 다룬다. 병렬 구조는 한 번에 여러 가지 일을 한다.

우리는 Go에서 동시성 프로그래밍을 사용할 수 있게 해준다. 그것은 Gorountine과 그들 간의 중요한 소통 능력을 제공한다. 나는 전자에 초점을 맞출 것이다.

# Gorountine과 Thread의 차이점

Go는 Gorountine을 사용하고 자바와 같은 언어는 Thread를 사용한다. 둘 간의 차이점은 무엇인가? 우리는 3가지 요소를 볼 필요가 있다 – 메모리 소비, Setup과 Teardown, Context Switching time.

## 메모리 소비

Gorountine의 창출은 많은 메모리를 필요로 하지 않는다 - 오직 **2kB**의 스택 공간만. 그들은 필요한 만큼의 할당하고 자유로운 Heap storage로 인해 성장한다. 반대로 Thread는 하나의 Thread의 메모리와 다른 Thread의 메모리 간의 경비 역할을 하는 Guard page라고 불리는 메모리 지역과 함께 **1Mb(500배 더)**로 시작한다. 

따라서 수신 요청을 처리하는 서버는 문제 없이 요청 한 건 당 하나의 Goroutine을 만들지만, 요청 한 건 당 하나의 Thread는 결과적으로 무서운 OutOfMemoryError로 이끌 것이다. 이것은 자바에 한정되어 있는 것이 아니다 - 동시성의 검출부로서 OS Thread를 사용하는 어느 언어든 이 문제에 대면할 것이다.

## Setup과 Teardown 비용

Threads는 거대한 설치(setup)과 철거(teardown) 비용을 가진다. 왜냐하면 이것은 OS로부터 Resoruce들을 요청해야 하고 작업이 끝나면 Resource들을 돌려줘야 하기 때문이다. 이 문제의 제2의 해결책(workaround)는 Thread의 Pool을 유지하는 것이다. 대조적으로, Gorountine들은 런타임에서 만들어지고 파괴되고 그러한 작업들은 매우 저렴하다. 언어는 Goroutine의 메뉴얼 관리를 지원하지 않는다.

## Switiching costs

Thread가 Blocking될 때, 다른 Thread가 그 자리를 스케쥴화해야 한다. Thread들은 우선적으로 스케쥴화되고, Thread가 바뀔 동안, 스케쥴러는 모든 Register들을 save/restore해야 한다, 즉, 16개의 범용 레지스터, PC(Program Counter), SP(Stack Pointer), segment 레지스터, 16개의 XMM 레지스터, FP coprocessor state, 16개의 AVX 레지스터, 모든 MSR들 등등. 이것은 thread들 간의 급격한 전환이 있을 때 매우 중요하다.

Gorountine은 협조적으로 스케쥴화되고 교체가 일어날 때, 오직 3개의 레지스터만이 save/restore되기 위해 필요하다 - Program Counter, Stack Pointer 그리고 DX. 비용은 훨씬 덜 든다.

일찍이 논의했던 바와 같이, gorountine의 갯수는 일반적으로 훨씬 높지만, 2가지 이유에 의해 교체 시간에 차이점을 낳지 않는다. 오직 런어블(runnable) goroutine들만이 고려되고, Blocking된 것들을 고려되지 않는다. 또한, 현대의 스케쥴러들은 O(1) 복잡하지 않다, 즉 교체 시간이 선택(threads 혹은 gorountines)의 수에 의해 영향을 받지 않는다는 의미이다.

# 어떻게 Gorountine들이 실행되는가

일찍이 언급했던 바와 같이, runtime은 창조부터 teardown의 스케쥴링까지 내내 goroutine을 관리한다. runtime은 모든 goroutine들이 다중화되어 있는 조금의 thread들에 할당된다. 어느 시점에서, 각각의 thread는 하나의 goroutine을 실행할 것이다. 만약 그 goroutine이 Blocking되었다면, 그 thread는 자기 대신에 실행할 다른 goroutine으로 교체할 것이다.

goroutine들이 협조적으로 스케쥴화되기 때문에, 계속해서 고리모양을 만드는(loop하는) goroutine은 동일한 thread에서 다른 goroutine들을 굶겨 죽일 수 있다. Go 1.2에서, 이 문제는 function을 입력할 때 Go 스케쥴러가 가끔 작동함으로써 다소 완화되고, 따라서 non-inlined function 명령을 포함하는 루프(loop)는 prempt(촉발하다prompt를 잘못 쓴 건가. 이런 단어는 사전 찾아봐도 없는데)된다.

# Goroutines Blocking

Goroutine들은 저렴하고 Blocking을 위해 다중화된 thread들을 야기하지 않을 것이다. 만약 그들이 이것들에 의해 Blocking된다면
*	네트워크 입력(network input)
*	sleeping
*	채널 작동(channel operations) 혹은
*	동기화 패키지의 초기 단계에서 Blocking하는 것. 

수많은 goroutine들이 만들어짐에도 불구하고, 만약 대부분의 goroutine들이 이들 중 하나에 Blocking당한다면 runtime이 대신에 또다른 goroutine을 스케쥴화하기 때문에 이는 시스템 resource들의 낭비가 아니다.

간단히 말해서, goroutine들은 threads보다 가벼운 관념이다. Go 프로그래머는 threads들을 다루지 않고, 유사하게 OS는 goroutines의 존재를 알지 못한다. OS의 관점에서, Go 프로그램은 이벤트 기반 C 프로그램과 같이 행동할 것이다.

# Threads and processors

비록 당신이 runtime이 만들 thread들의 수를 직접적으로 조정할 수 없더라도, 프로그램에 의해 이용되는 프로세서 코어들의 수를 조정하는 것은 가능하다. 이것은 runtime.GOMAXPROCS(n)의 요청과 함께 variable GOMAXPROCS를 설정하는 것으로 이루어진다. 코어들의 개수를 증가하는 것은 아마 당신의 프로그램의 성능을 증가시키는 것에 필수적이지는 않을 것이다. 그것의 디자인에 의존하여.
profiling tool들은 당신의 프로그램을 위한 이상적인 코어의 개수들을 찾는 데 사용될 수 있다.

# 결론

다른 언어들과 함께, 하나의 goroutine보다 더 많은 goroutine들에 의해 공유된 resource들에의 동시적인 접근을 방지하는 것은 중요하다. channel들을 사용해서 goroutine들 간에 데이터를 전송하는 것이 가장 좋다. 즉, 메모리 공유에 의해 소통하지 마라; 대신에, 소통에 의해 메모리를 공유해라.

마지막으로, 나는 강력히 당신이 C. A. R. Hoare가 쓴 Communicating Sequential Processes를 보기를 추천한다. 이 사람은 정말 천재다. (1978년 출판된)이 책에서 그는 어떻게 프로세서의 단일한 핵심 성능(core performance)이 마침내 고원(plateau)이 되고 chipmaker가 대신에 코어들의 수를 증가시킬 것이라는 걸 예측했다. 이것을 이용한 그의 제안은 Go의 디자인에 깊은 영향을 주었다.
