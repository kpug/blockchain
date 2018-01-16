# Week1 Introduction to Crypto and Cryptocurrencies

## Welcome

이 강의는 11주차로 되어있다.

이번 강의에서는 화화폐에 대해 이야기 하는데 필요한 암호화 기본 요소(cryptographic primitives)에 대해 소개할 것이다.

암호화 해시(cryptographic hashes)와 전자 서명(digital signatures)에 대해 이야기 하고, 암호화폐를 만드는 몇가지 방법에 대해서 이야기 할 것이다.

## Crptographic Hash Functions

암호화 해시 함수는 수학적 함수이다. 해시 함수는 다음 세가지 속성을 가진다.

* 아무 문자열을 입력으로 가진다.
* 고정된 크기의 출력이 있다. (; 우리는 256 bits를 사용할 것이다.)
* 효율적으로 계산할 수 있다.

암호화 속성의 해시 함수는 복잡한 개념이지만 우린 아래 세가지 개념만 살펴볼 것이다.

* collision-free
* hiding
* puzzle-friendly

### Collision-free (충돌 회피)

어떤 값 x, y에 대해 x != y 라도 H(x) == H(y)일 수 있기 때문에, x와 y의 값을 찾는 것은 불가능하다.

그러나 충돌이 없다고 이야기할 수는 없다. 가능한 모든 입력이 256 bits로 사상되는 것이기 때문에 입력이 무한대로 커지면 동일한 값으로 사상(map)될 수 있다. 그러나 어느누구도 충돌하는 값을 찾아낼 수는 없다.

보통의 사람이 보통의 컴퓨터로 충돌을 찾는 것은 불가능에 가깝다. 2^130의 임의의 수를 입력으로 할 때 99.8%의 확률로 충돌을 발견할 수 있다. 그러나 임의의 수가 천문학적으로 커진다면, 충돌을 발견할 확률은 극히 드물어진다.

그렇다면, H(x)가 H(y)라면 안전하게 x = y 라고 추측할 수 있다.

### Hiding

H(x)의 값이 있을 때, x는 찾을 수 없다. 입력이 단 2개라면 ("heads", "tails"), 순서대로 Hash 값을 구해보고 비교해보면 쉽게 찾을 수 있다. 그러나 input이 무엇인지 알 수 없는 상황에서는 x를 찾을 수 없다.

r이 높은 최소 엔트로피를 갖는 확률 분포에서 선택되면, H (r | x)가 주어졌을 때 x를 발견하는 것은 불가능하다.

* Commitment

"어떤 값을 봉투에 담은 뒤", "나중에 봉투를 연다고" 가정해보자.

Commitment API는 담는 것 (com, key) := commit(msg) 과 여는 것 match := verify(com, key, msg)가 될 것이다.

보안 특징은 hiding과 binding이다.
Hiding: com이 주어져도 msg를 찾는 것은 불가능하다.
Binding: msg != msg를 찾는 것은 불가능하다. verify(commit(msg), msg') == true

```
commit(msg) := (H(key | msg), key)
verify(com, key, msg) := (H(key | msg) == com)

Hiding: H(key| msg)가 주어져도 msg를 찾는 것은 불가능하다.
Binding: msg != msg'를 찾는 것은 불가능하다. H(key| msg) == H(key| msg')
```

### Puzzle-friendly

모든 가능한 출력 값 y를 위해,
k가 높은 최소 엔트로피를 갖는 확률 분포에서 선택되었을 때, H(k | x) = y에서 x를 찾는 것은 어렵다.

이를 퍼즐 검색으로 만들 수 있다.

puzzle Id가 주어졌을 때 Y를 목표로 정답 x를 찾는 것이다.

```
H(id | x) ∈ Y
```

Puzzle-friendly라는 것은 임의의 값 x로 시도하는 것보다 해결책이 없는 것이 더 낫다는 것을 넌지시 나타낸다.

이와 관련하여 비트코인 마이닝에 대해서 이야기 할 때 살펴볼 것이다.

### SHA-256

비트코인에서도 사용하고 있는 아주 유용한 해시 함수이다.

더 자세한 내용은 [Wikipedia](https://ko.wikipedia.org/wiki/SHA)에서 확인해보도록 하자.


## Hash Pointer and Data Structures

해시 포인터는 데이터 구조의 일종이다. 해시 포인터는 정보가 저장되어 있는 곳을 가르키며, 정보의 암호화된 해시 값을 저장한다. 해시 포인터가 있다면 우리는 정보를 가져올 수 있고, 변형되지 않았는지 검증할 수 있다.

```
H( ) -> (data)
```

핵심 아이디어는 링크드 리스트나 바이너리 트리 같은 데이터 구조의 링크를 해시 포인터로 만드는 것이다.

블록 체인의 사용 사례는 부당변경의 흔적을 찾아내는 것이다. 누군가 중간에 데이터를 임의로 변경했다면, 해시는 다른 값으로 변경될 것이기 때문에 값 영역이 틀어지게 된다.

해시 포인터를 기억하는 것만으로도 처음부터 모든 목록의 전체 해시에 대한 것을 기억하고 있다는 것이다. 

### 머클 트리

해시 포인터로 만든 유용한 바이너리 트리 데이터 구조인 머클 트리가 있다. 머클 트리는 리프를 가르키는 링크가 해시 포인터로 되어있으므로 데이터를 변조하기가 더욱 어렵다. 하위의 데이터 변조는 상위 해시 포인터를 변조가 필요하기 때문이다.

다른 블록 체인과 다르게 머클 트리의 장점은 데이터 검증을 위한 비용이 상대적으로 저렴하다는 것이다. 어떤 데이터를 검증하는데 O(log n) 시간이 소요된다.

해시 포인터는 사이클이 없는 모든 포인터 기반 데이터 구조에 사용할 수 있다.

## Digital Signatures

두 번째 암호화 기본요소는 전자서명이다. 전자 서명은 문서에 서명하는 것과 같아야 한다. 
내가 서명하고 다른 누군가가 증명할 수 있다. 서명은 특별한 문서에 종속적이며, 잘리고 덧 붙여진 다른 문서엔 할 수 없다.

### 전자 서명 API

sk가 비밀키이고, pk가 공개 검증용 키라고 한다면 아래와 같다.

```
// 키 생성
(sk, pk) := generateKeys(keysize)

// 서명 생성
sig := sign(sk, message)

// 서명 검증
isValid := verify(pk, message, sig)
```

유효한 서명이 검증된다는 것은 `verify(pk, message, sign(sk, message)) == true` 이다.

비트코인은 표준 ECDSA(Elliptic Curve Digital Signature Algorithm) 사용한다.

## Public Keys as Identities

유용한 트릭 중 하나는 공개키(public key)를 신원 증명(identity)으로 사용하는 것이다. 비밀키와 대응되는 공개키가 있다면 비밀키로 메시지를 서명하고, 공개키를 가진 사용자에게 보낸다. 그러면 공개키를 가진 사용자는 해당 메시지를 검증할 수 있다. 

이를 이용하면 분산된 신원 증명 관리(Decentralized identity management)가 가능하다. 누구든 언제든지 자유롭게 새로운 신원 증명을 생성할 수 있기 때문이다.

이것이 비트코인이 실제로 신원을 확인하는 방법이며, 이러한 신원 증명을 비트코인 전문용어로 주소라고 한다.

This is the way Bitcoin in fact does identity. These identities are called addresses in Bitcoint jargon.

## A Simple Cryptocurrency

암호화화폐 대해 이야기 위한 배경지식으로 암호화를 알아야 한다. 이제 암호화 화폐에 대해서 본격적으로 살펴보자.

구피코인에 대해서 살펴볼 것이다. 몇가지 같은 규칙이 있다.

첫 번째 규칙은 구피는 새로운 코인을 만들 수 있고, 새로 코인을 만들면 그의 소유가 된다. 코인에는 uniqueCoinID가 있고, 아무나 증명할 수 있는 전자 서명이 있다.

두 번째 규칙은 코인을 소유한 사람은 누군가에게 건낼 수 있고, 소비할 수 있으며, 소비하면서 서명을 하여 전한다. 구피에게서 엘리스로 건내진 코인은 엘리스가 직접 검증해볼 수 있다. 검증이 끝나면 엘리스는 코인의 소유주가 된다. 이제 엘리스가 서명한 코인을 다시 밥에게 건낸다.

위와 같은 구조라면 우리는 간단한 체인을 따라가서 모든 서명을 확인함으로써 동전의 유효성을 확인할 수 있다.

문제를 발생시켜보자. 앨리스가 이를 위조해서 2개의 해시 포인터를 만들고 척에게도 지불했다고 하는 것이다. 척은 엘리스로 부터 온 코인을 검증해보고 유효하므로 코인의 소유주가 되었다고 생각할 수 있다.

이를 이중 지불 문제(double-spending attack)라 한다. 이중 지불 문제는 암호화화폐가 해결한 주요한 문제 중 하나이다.

구피 코인은 이를 해결할 수 없고, 그래서 안전하지 않다. 구피코인은 암호화화폐로 만들기 위해 이를 해결하는 방법을 살펴보자.

구피 코인을 향상시킨 스크루지 코인을 만들자. 주요 아디이어는 모든 트랜잭션 이력을 가진 데이터 구조를 발행하는 것이다. 이 블록체인은 스크루지가 서명한다. 아무나 스쿠르지가 진짜 이 해시 포인터에 서명했는지 검증할 수 있다. 나중에는 비트코인 처럼 여러 트랜잭션을 하나의 블록으로 만들 수도 있다.

이렇게 하면 이중 지불 문제를 발견할 수 있다. 이전과 같은 상황에서 척이 이력을 살펴 볼 수 있기 때문이다. 실제로 엘리스가 이미 밥에게 코인을 주었다는 것을 누구나 살펴볼 수 있다.

스크루지에는 두 종류의 트랜잭션이 있다.

* 코인 생성(CreateCoins): 코인을 생성할 때는 값이 얼만지 누가 받게 되는지를 명시한다.
* 코인 지불(PayCoins): 코인을 소비하고 없애는 것이다. 코인생성에 추가하여 서명이 있다.

코인지불이 유효하다는 것은 아래 4가지 조건으로 만족된다.

* consumed coins valid
* not already consumed
* totla value out = total value in
* signed by owners of all consumed coins

이렇게 지불이 되고 나면 스크루지에 의해 서명이되고 이력에 추가된다. 코인은 불변(immutable)이며, 절대 변하지 않는다.

그러나 이것의 큰 문제는 스크루지다. 스크루지가 못된 행동을 하거나 비행을 저지르면 시스템은 더이상 동작하지 않는다. 이 문제를 우리는 중앙집권(Centralization)이라 한다.

 That is, can we get rid of that centralized Scrooge figure?

 Can we have a Cryptocurrency that operates like ScroogeCoin in many ways, but
 doesn't have any central trusted authority?

 In order to do that, we're going to need to figure out how to provide the services
 that Scrooge provides, but do it in a decentralized way,
 a way in which no particular party is particularly trusted.

 That means we're gonna need to figure out how everyone can
 agree upon a single published block chain
 that is the agreed upon history which transactions have happened.

 We need to figure out how people can agree which transactions are valid and
 which transactions have actually occurred.

 And we need to figure out how we can assign IDs to things
 in a decentralized way.