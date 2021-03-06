---
layout: post
title:    ZKP 기술보고서 - 4. NIZK의 예 [Schnorr protocol with Fiat-Shamir heuristic]
date:   2020-12-16 00:00:01 +09:00
author: "장재혁"
categories: zkp
tag: [zkp]
youtubeId :
slideWebId :
---


> 작성자: 장재혁, [GIST 블록체인 인터넷 경제 연구센터 (센터장 이흥노)](https://infonet.gist.ac.kr/?page_id=6711)
>
> This work was created through a joint research with Onther Co., LTD., and supported by a grant-in-aid of Institute of Information & Communications Technology Planning & Evaluation (IITP), Republic of Korea.
>
> 이 글은 정보통신기획평가원(IITP)의 지원을 받아 (주)온더와의 공동연구를 통해 만들어진 결과물이다.

앞선 Schnorr 프로토콜의 예에서 대화형 구조와 도전적 문제 (challenge)의 중요성을 살펴보았다. 도전적 문제가 없으면 Schnorr 프로토콜이 건실성을 만족하지 못하였다. 그리고 도전적 문제는 증명자가 아닌 검증자가 대화의 형태로 제출하였다: 증명자가 먼저 무작위 값을 제출 한 후 검증자가 이어서 도전적 문제를 제출하는 것이었다. 이렇게하지 않으면 거짓 증명자가 도전적 문제를 자신에게 유리하게 조작할 수 있게 되어 거짓 증명이 쉬워지기 때문이었다. 즉, 도전적 문제의 공정성이 훼손되면 증명 프로토콜의 건실성이 위협받는다.

NIZK의 한 예인 Fiat-Shamir heuristic은 "random oracle"이라는 랜덤 프로그램의 존재를 가정한다. 증명자가 random oracle로부터 무작위 값을 수여받아 검증자의 도전적 문제를 대체한다. Random oracle은 매번 다른 무작위 값을 생성하여야 하며, 생성된 무작위 값이 random oracle에 의한 값이라는 것을 누구나 확인 할 수 있어야 한다. 다시 말해, 증명자가 도전적 문제를 정직한 과정을 거쳐 생성하였다는 것을 누구나 검증할 수 있어야 한다.

Random oracle로써 hash 함수가 사용 될 수도 있다[^1]. Hash 함수는 입력값이 변함에 따라 출력값이 무작위처럼 변하는 특성이 있으며, 입력값을 알면 누구나 같은 출력값을 재생산 해볼 수 있다. 즉, 도전적 문제가 공정하게 생성된 것이 맞는지 누구나 검증 할 수 있다.

그러나 모든 NIZK가 random oracle의 존재를 가정하는 것은 아니다. 이후에 함께 살펴볼 최신 증명 프로토콜인 Groth16와 같은 프로토콜은 random oracle에 의존하지 않는 대신 암호화를 활용하여 도전적 문제가 공정함을 보장한다.

본론으로 돌아와, Fiat-Shamir heuristic에서는 도전적 문제를 검증자가 아닌 random oracle이 선택한다. Schnorr 프로토콜에 Fiat-Shamir heuristic을 적용하면 다음과 같다. 증명자는 공개 된 값 $$y$$, $$g$$, $$p$$에 대해 $$y\equiv g^x\bmod p$$를 만족하는 비공개 값 $$x$$를 알고 있음을 검증자에게 증명하는 상황이다.

| 프로토콜 3. Fiat-Shamir heuristic이 적용된 Schnorr 프로토콜  |
| ------------------------------------------------------------ |
| 1. 증명자는 하나의 (비공개) 무작위 값 $$v$$ $$\left( 0\le v<p-1 \right)$$를 골라서 $$t=g^v\bmod p$$를 계산 하여 검증자에게 보낸다.<br/>2. 증명자는 random oracle로부터 $$c$$ $$\left( 0\le c<p-1 \right)$$를 받아 검증자에게 보낸다.<br/>3. 증명자는 $$r=v-cx\bmod \left( p-1 \right)$$를 계산하여 검증자에게 보낸다.<br/>4. 검증자는 $$c$$가 random oracle에게서 받은 값임을 확인하고, $$t\equiv g^ry^c\bmod p$$임을 확인한다. |

이전 글의 원본 Schnorr 프로토콜인 프로토콜 1과 Fiat-Shamir heuristic이 적용된 프로토콜 3을 비교해 보면, 증명자와 검증자 간의 대화(conversation)과정이 사라진 대신, 증명자가 검증자에게 최종 제출할 증거가 추가되었다. 구체적으로, 프로토콜 1에서는 $$r$$과 $$t$$ 두 개의 값을 검증자에게 제출하였으나, 프로토콜 3에서는 $$r$$과 $$t$$, $$c$$ 세 개의 값을 제출한다.

결론적으로, 대화과정을 제거한 대신 증거의 길이가 길어지는 trade-off현상이 발생하였다. 일반적으로 NIZK는 증명해야 할 명제가 복잡해질수록 증거의 길이가 더욱 길어질 수 있다. 증거의 길이가 길어지면 다양한 분야로의 응용이 제한이 될 수 있다. 이러한 이유로 최근의 영지식 증명 연구자들은 증거의 길이를 효율적인 수준으로 줄이는 zk-SNARK를 연구하고 있다.

[^1]: Wikipedia의 Fiat-Shamir heuristic 페이지에서는 random oracle로써 hash함수를 사용한 예를 보여준다. 그러나 이 경우 거짓 증명자가 도전적 문제를 자신에게 유리하게 조작 할 수 없는지에 관한 검증이 필요하다.
