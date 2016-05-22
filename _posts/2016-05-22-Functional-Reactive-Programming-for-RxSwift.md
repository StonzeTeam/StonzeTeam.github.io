---
layout: post
title: "Functional Reactive Programming for RxSwift"
date: 2016-05-22
excerpt: "스톤즈 팀이 스터디한 내용을 이곳에 기록합니다"
tags: [FRP, Functional Reactive Programming, 프로그래밍 패러다임, 함수형, Swift]
comments: true
---

<iframe src="//www.slideshare.net/slideshow/embed_code/key/MJStyYD39ie7fS" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/sunhyouplee/functional-reactive-programming-with-rxswift-62123571" title="Functional Reactive Programming With RxSwift" target="_blank">Functional Reactive Programming With RxSwift</a> </strong> from <strong><a href="//www.slideshare.net/sunhyouplee" target="_blank">선협 이</a></strong> </div>

상당히 오래 우려먹은 주제이지만 스터디의 첫 주제로 FRP에 대해서 다뤘습니다.

# 요약
## Functional Reactive Programming?
함수형, Observable, Data Flow로 이루어진 **프로그래밍 패러다임**.

## 함수형
* 상태없음
* 쉬운 쓰레드 세이프 확보
* 간결하게 작성 가능
* 생산성 증가

위와 같은 장점이 있지만 장점이 곧 단점이 될 수 있다.

## Observable
옵저버 패턴과 비슷하지만 Lazy Evaluation이라는 특징을 가지고 있다.

## Data Flow
if, swift, for, while같은 문법을 사용하는 Control Flow와 다르게 Recursion, Pipe 같은 처리로 데이터가 흐르도록 하는 것.

## ReactiveX
멀티 패러다임 언어에서 FRP를 쉽게 사용하도록 도와주는 오픈소스 라이브러리.

## RxSwift
ReactiveX의 Swift 구현체

