---
layout: post
title:    ZKP 기술보고서 - 7. zk-SNARK의 예 [Groth16 프로토콜 (3)]
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

보안 목적의 ECC기반 암호화(cryptography)와 pairing
--------------------------------------------------

QAP가 사용 된 증명 프로토콜의 *완전성*은 ***정리 3*** (이전 글 참고)에 의해 보장된다. 증명자가 써킷을 완벽히 복원 할 수 있음은 곧 증명자가 함수 를 정직하게 계산하였음을 의미하기 때문이다. 그러나 QAP가 증명프 로토콜의 *건실성*까지 보장하지는 못한다. 거짓 증명자가 QAP의 계수들을 모르더라도, ***정리 3***의 조건을 만족시키는 다항식들을 특수한 조합을 사용하여 만들어 낼 수도 있기 때문이다. 이 뿐만 아니라 *영지식성* 또한 보장받지 못한다. 가 노출되면 정직한 증명자의 QAP 계수가 검증자 혹은 제 3자에게 탈취당할 수 있다.

QAP를 사용하는 영지식증명 프로토콜의 *건실성*과 *기밀성* 보장을 위하여 ECC와 pairing을 적용할 수 있다. ECC는 비공개 정보를 암호화하여 숨긴다. ECC의 암호화는 복호가 매우 어렵다. Pairing은 비공개 값(정보)가 ECC에 의해 암호화된 상태에서도 더하기 및 곱하기 연산의 결과를 검증 가능하도록 해준다. 구체적으로 어떤 비공개 정보 $$x$$와 $$y$$가 있고 공개정보 $$z=xy$$일 때, ECC로 암호화한 결과를 각각 $$ECC\left( x \right)$$와 $$ECC\left( y \right)$$, $$ECC\left( z \right)$$라 하면, pairing의 역할은 $$ECC\left( xy \right)=ECC\left( z \right)$$가 맞는지의 확인을 $$ECC\left( x \right)$$와 $$ECC\left( y \right)$$의 복호 없이 수행할 수 있게 해주는 것이다[^1]. 따라서 증명자는 $$p\left( x \right)$$와 $$h\left( x \right)$$를 암호화 한 후 증거로써 제출하고, 검증자는 pairing을 활용하여 ***정리 3***의 조건 $$p\left( x \right)=t\left( x \right)h\left( x \right)$$이 성립하는지를 복호 없이 확인 할 수 있다.

## Elliptic curve?

ECC는 타원곡선(elliptic curve)이라 불리는 곡선상의 점(point)들을 사용하여 정보를 감춘다. 타원곡선은 그 정의가 다양한데, 한가지의 예로 $$S:y^2=x^3+ax+b$$의 형태로 정의되는 곡선을 들 수 있다. 여기서 $$a$$와 $$b$$는 상수이며 정수이다. 그림 8은 다양한 $$a$$와 $$b$$에 따른 곡선 $$S$$를 그린 결과이다. 타원곡선 $$S$$상에서 정의된 점이란 $$\left( x,y \right)$$의 좌표 형태로 표현되며, $$x$$와 $$y$$를 타원곡선 방정식에 대입하였을 때 등호가 성립되는 점이다. ECC에서는 타원곡선상의 두 점간의 "점 덧셈" 연산을 정의하여 사용한다. 이 점 덧셈의 기호는 우리가 일반적으로 다루는 숫자덧셈의 기호와 "+"로 같지만, 그 정의는 완전히 다르다. 두 실수의 덧셈, 1+2=3 이라는 정의가 점 덧셈에서는 사용되지 않는다. 구체적으로, 타원곡선상의 두 점 $$A$$와 $$B$$의 덧셈 $$A+B$$는 다음과 같은 순서로 수행된다[^2]:

1.  $$A$$와 $$B$$가 서로 다를 경우, 두 점을 잇는 직선 $$l$$을 그린다.

2.  $$A$$와 $$B$$가 서로 같을 경우, 그 점에서 타원곡선 $$S$$와 접하는
    접선 $$l$$을 그린다.

3.  선 $$l$$과 타원곡선 $$S$$가 교차하는 새로운 점을 찾는다.

4.  만약 교차하는 새로운 점이 없다면, 덧셈의 결과는 항등원인 $$O$$이다.

5.  만약 교차하는 새로운 점이 있다면, 그 점의 $$x$$축 대칭점이 덧셈의
    결과인 $$A+B=C$$이다.

![https://upload.wikimedia.org/wikipedia/commons/thumb/d/db/EllipticCurveCatalog.svg/1024px-EllipticCurveCatalog.svg.png](/images/article_7/media/image1.png){: width="90%" height="90%"}
<!-- {: width="4.340908792650919in" height="4.192820428696413in"} -->
<br/>[그림 8 타원곡선 $$y^2=x^3+ax+b$$의 $$a$$와 $$b$$의 값에 따른 plot] <br/>각 plot의 가로축은 $$x$$축이고 세로축은 $$y$$축이며 각각의 범위는 -3부터 3까지이다. (출처: Wikipedia)

위의 좌표덧셈 연산에서 점 $$O$$와 대칭점이 언급되었다. 점 $$O$$는 point at infinity로 불리는 좌표이며, 다른 좌표들과는 달리의 $$\left( x,y \right)$$의 형태로 표현 될 수 없다[^3]. 점 $$O$$는 항등원의 역할을 한다. 다시 말해, 어떤 점 $$A$$에 대해 $$A+O=A$$이며, $$O+O=O$$이다. 어떤 점 $$A=\left( x,y \right)$$의 대칭점은 $$-A$$이며, 그 좌표는 $$\left( x,-y \right)$$이다. 점 덧셈의 정의에 의해 $$A+\left( -A \right)=O$$이다.

## Elliptic curve를 이용하는 cryptography?

타원곡선의 점 덧셈 연산은 역 추적이 어렵다. 예를 들어 어떤 점 $$A$$를 5번 더한 결과를 $$B$$, 즉 $$B=5A$$라 하면, $$B$$를 보고 이 점이 $$A$$를 몇 번 더한 결과인지 유추해내는 과정이 매우 어렵다는 의미이다. 반면 실수 덧셈의 경우, 숫자 3을 5번 더해 15를 만든 후, 그 결과 15를 보고 3을 몇 번 더하였는지 쉽게 알 수 있다. EC의 이러한 역 추적 문제를 "discrete logarithm" 문제라 부른다[^4]. Discrete logarithm 문제를 활용한 암호화가 ECC이다.

암호화를 수행하기 전에, 먼저 사용할 타원곡선 $$S$$와 그 위의 한 점 $$G$$를 약속 해 둔다. 이 때 $$G$$는 "generator"라 불리는 특수한 점이다[^5]. $$G$$를 사용하여 어떤 정수 값 $$x$$를 암호화 하는 연산을 $$\left[ x \right]_G$$라 표현하겠다. 암호화 결과를 $$X$$라 할때, 암호화 결과는 다음과 같다:
<br/><br/>

$$X=\left[ x \right]_G:=xG \qquad (7)$$

<br/><br/>

암호화 결과 $$X$$는 좌표로 표현되는 타원곡선상의 한 점이며, 점 $$G$$에 $$x$$회의 점 덧셈을 수행 한 결과이다. $$G$$를 알때, $$X$$로부터 $$x$$를 유추하는 것은 매우 어렵다.

정의 에 의해, 두 정수 $$a$$와 $$b$$에 대해 암호화 연산 $$\left[ \cdot \right]_G$$는 linear 연산이다. 즉, 다음과 같은 additivity와 scalability (homogeneity)를 갖는다:

<br/><br/>

$$\left[ a+b \right]_G=\left[ a\right]_G+\left[ b \right]_G \qquad (8)$$


$$a\left[ b \right]_G=\left[ ab \right]_G \qquad (9)$$

<br/><br/>
## Bilinear pairing

Pairing $$e\left( \cdot ,\cdot \right)$$는 입력이 2개의 타원곡선 점들이고 출력이 숫자 값인 함수이다[^6]. Pairing이 지닌 특성 중 가장 중요한 특성은 "Bilinearity"이다. Bilinear pairing의 대표적인 예로 Weil pairing, Tate pairing, Ate pairing이 있으며, 최근 동향에서는 Ate pairing 중 성능이 최적화된 optimal Ate pairing이 가장 많이 사용된다[^7].

세 점 $$X$$, $$Y$$, $$Z$$와 정수 $$c$$와 $$d$$에 대하여 bilinearity를 정리하면 다음과 같다:
<br/><br/>

$$
\begin{align}
 e\left( X+Y,Z \right)=e\left( X,Z \right)e\left( Y,Z \right) \\
 e\left( X,Y+Z \right)=e\left( X,Y \right)e\left( X,Z \right) \\
 e\left( cX,dZ \right)=e\left( X,Z \right)^cd \\
\end{align} \qquad (10)
$$

<br/><br/>

이러한 Bilinear 특성은 ECC로 암호화된 정보의 더하기와 곱셈 연산을 복호 없이 수행하는데 사용 될 수 있다. 예를 들어, 어떤 점 $$G$$와 $$H$$가 있고, 이들을 사용하여 비공개 정보 (정수 값) $$x$$와 $$y$$, $$z$$, $$w$$를 암호화 하였다고 가정하자. 비공개 정보들은 $$xy+z=w$$라는 관계를 만족한다는 것이 알려져 있다고 가정하자. 암호화된 결과를 사용하여 비공개 정보들이 이 관계를 만족하는지 확인하는 방법은 다음의 등호가 성립하는지를 확인하는 것이다:
<br/><br/>

$$e\left( xG,yH \right)e\left( zG,H \right)=e\left( wG,H\right) \qquad (11)$$

<br/><br/>
식 (6) 의 bilinearity에 의해, 식 (7) 은 다음과 같이 정리 될 수 있다:
<br/><br/>

$$e\left( xG,yH \right)e\left( zG,H \right)=e\left( wG,H \right) \qquad$$

$$\Leftrightarrow e\left( G,H \right)^xye\left( G,H\right)^z=e\left( G,H \right)^w \qquad$$

$$\Leftrightarrow e\left( G,H \right)^xy+^z=e\left( G,H\right)^w \qquad (12)$$  

$$\Leftrightarrow e\left( \left( xy+z \right)G,H \right)=e\left(wG,H \right) \qquad$$

<br/><br/>

결론적으로, 암호화 결과들인 $$xG$$, $$yH$$, $$zG$$, $$wG$$에 pairing을 적용함으로써 비공개 정보들의 복호 없이 $$xy+z=w$$라는 등식이 만족하는지 확인 할 수 있다.

편의를 위해, 두 정수 $$x,y$$와 두 점 $$G,H$$에 대해, $$xG$$와 $$yH$$의 pairing을 다음과 같이 표기하겠다:
<br/><br/>

$$e\left( xG,yH \right)=:\left[ x \right]_G\cdot\left[ y \right]_H \qquad (13)$$

<br/><br/>


그리고 다음과 같은 표기를 사용하겠다:
<br/><br/>

$$\left[ xy \right]_G\cdot H:=\left[ x\right]_G\cdot \left[ y \right]_H=e\left( xG,yH\right) \qquad (14)$$

<br/><br/>

새롭게 정의된 연산 $$\left[ \cdot \right]_G\cdot H$$의 bilinearity를 다음과 같이 표기한다:

<br/><br/>

$$e\left( xG,H \right)e\left( yG,H \right)=\left[ x\right]_G\cdot H+\left[ y \right]_G\cdot H \qquad$$

$$ \qquad \qquad =\left[ x+y \right]_G\cdot H \qquad (15)$$

$$ \qquad \qquad =e\left( \left( x+y \right)G,H \right) \qquad$$


<br/><br/>

이러한 표기의 장점은 식 (7)과 같은 표현을 선형대수적 표현으로 변환하여 직관적으로 읽기 쉽게 해주기 때문이다. 예를 들어 식 (7)은 다음과 같이 표기될 수 있다:

<br/><br/>

$$e\left( xG,yH \right)e\left( zG,H \right)=e\left( wG,H \right) \qquad$$

$$\Leftrightarrow \left[ xy+z \right]_G\cdot H=\left[ w\right]_G \cdot H \qquad (16)$$  

<br/><br/>

식 (7)의 목적이 비공개 정보들이 $$xy+z=w$$의 관계를 만족하는지 확인하는 것임을 고려할 때, 식 (18)이 그 목적을 더욱 직관적으로 알아볼 수 있게 해준다.


[^1]: Pairing에는 $$ECC\left( x \right)$$와 $$ECC\left( y \right)$$만을 사용하여 $$ECC\left( x+y \right)$$를 계산해주는 기능은 없다. 이러한 기능이 지원되는 암호화를 동형암호 (homomorphic encryption)라 부른다.
[^2]: 타원 곡선의 점 덧셈의 구체적인 정의는 Wikipedia의 elliptic curve 문서에 잘 설명되어 있다. 링크: https://en.wikipedia.org/wiki/Elliptic_curve
[^3]: 이 문서에서 다루는 타원곡선은 affine 좌표계(coordinate system)라 불리는 $$\left( x,y \right)$$좌표계에서 표현되고 있다. 그러나 타원곡선을 표현하는 다른 좌표계도 존재한다. 대표적으로 projective 좌표계가 있는데, 이는 타원곡선상의 점을 $$\left( x,y,z \right)$$로 표현한다. Projective 좌표계에서는 $$O=\left( 0,1,0 \right)$$으로 표현 할 수 있다. 참고로 좌표계란 타원곡선을 표현하는 방법일 뿐, 좌표계를 변환한다고 해서 곡선의 모양이 변하지는 않는다.
[^4]: Discrete logarithm problem이란 $$y=g^x\bmod p$$에서 $$y$$와 $$g$$, $$p$$를 알 때 $$x$$를 찾는 문제이다. 이러한 문제는 수학적으로 어려운 문제로 잘 알려져 있다. ECC에서도 같은 용어가 사용되어, elliptic curve discrete logarithm problem (ECDLP)로 불리고 있다.
[^5]: Generator에 점 덧셈 연산을 반복적으로 수행하면 타원곡선상의 모든 점들이 생성된다.
[^6]: Pairing을 두 점의 내적(inner product)으로 표현하기도 한다.
[^7]: Weil pairing의 mapping rule은 D. Boneh와 M. Franklin의 2003년 논문인 "Identity-based encryption from the weil pairing"의 Appendix에 간결하게 설명되어 있다. 이 외의 다른 Pairing들도 Weil pairing과 최종 연산 방식만 다를 뿐 원리는 모두 같다. 그러므로 서로 변환 가능하다.
