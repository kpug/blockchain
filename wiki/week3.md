# Week3 Mechanics of Bitcoin

이 강의에서는 비트코인의 메커니즘에 대해서 자세히 살펴본다. 데이터 구조와 스크립트에 대해서 알아볼 것이다.

## Bitcoin Transactions

트랜젝션은 화폐에 기초가 될 블록을 만드는 기본요소이다.

장부는 기본적으로 추가만 가능하여, 시간이 갈 수록 더 많이 쌓이기만 한다.

비트코인의 트랜젝션은 개략적으로 이런 모습이다. 고유 ID과 입력, 출력으로 구성된다.

```script
[1] Input: 0
    Outputs: 25.0 -> Alice

[2] Inputs: 1[0]
    Outputs: 17.0 -> Bob, 8.0 -> Alice

[3] Inputs: 2[0]
    Outpus: 8.0 -> Carol, 7.0 -> Bob

[4] Inputs: 2[1]
    Outpus: 6.0 -> David, 2.0 -> Alice
```

고유 ID 2의 트랜젝션을 보면 Alice가 소유했던 코인을 다시 스스로에게 보내는 것을 볼 수 있는데 이를 Change address라 한다.

입력에 이전 출력이 명시되어 있기 때문에 전체 트랜젝션을 보지 않고도 더 쉽게 데이터를 검증 할 수 있다.

위에 트랜젝션을 합쳐 아래와 같이 공동지불 할 수도 있다.

```
[1] Inputs: ...
    Outputs : 17.0 -> Bob, 8.0 -> Alice

[2] Inputs: 1[1]
    Outputs : 6.0 -> Carol, 2.0 -> Bob

[3] Inputs: 2[0], 2[1]
    Outpus: 8.0 -> David
```

실제 비트코인의 구조를 상상해 보면 아래와 같다.

```json
{
    // Metadata
    "hash": "5a42590...b8b6b",
    "ver": 1,
    "vin_sz": 2,
    "vout_sz": 1,
    "lock_time": 0,
    "size": 404,

    // Inputs
    "in": [
        {
            "prev_out": {
                "hash" : "3be4...80260",
                "n": 0
            },
            "scriptSig": "304440....3f3a4ce81",
        }
    ],

    // Outputs
    "out": [
        {
            "value": "10.12287097",
            "scriptPubKey": "OP_DUP OP_HASH160 69e...3d42e OP_EQUALVERIFY OP_CHECKSIG",
        },
    ]
}
```

## Bitcoin Scripts

각 트랜젝션에 출력은 단순 공개키가 아니고, 실제로는 스크립트 이다.

출력 주소는 실제 스크립트이며, 아래와 같이 구성된다. 이 예제에서는 4개의 명령이 있다.

```script
OP_DUP
OP_HASH160
69e02e18...
OP_EQUALVERIFY OP_CHECKSIG
```

입력 주소 또한 스크립트이다. 그래서 간단하게 입력과 출력을 붙여서 하나의 스크립트를 만들 수 있다. 각각을 scriptSig, scriptPubKey라고 부른다.

```
// scriptSig
30440220...
0467d2c9...

// scriptPubKey
OP_DUP
OP_HASH160
69e92e18...
OP_EQUALVERIFY OP_CHECKSIG
```

트랜젝션이 유효하려면 두 스크립트를 함께 복사하여 실행했을 때 에러 없이 수행되면 유효한 것으로 본다.

비트코인 스크립트 언어는 Forth라는 언어에서 영감을 얻었고, 아주 간단한 스택 기반 언어이다. 세련된 암호화관련 명령은 없어서, 해시 함수를 계산하거나 서명을 계산하고, 검증하는 특수 목적의 명령이 있다. 또한 루프가 없다.

실제 비트코인 스크립트를 살펴보면 아래와 같다.

```script
<sig>
<pubKey>
OP_DUP
OP_HASH160
<pubKeyHash?>
OP_EQUALVERIFY
OP_CHECKSIG
```

OP_CHECKSIG 명령은 최종적으로 스택에 남아있는 `<pubKey><sig>`를 검증하고, 스택을 비우게 된다. 그러면 스크립트는 모든 명령을 완료하게 되고 트렌젝션이 유효한 것으로 판정이 끝난다.

265 opcodes (15 disabled, 75 reserved)

* 산술적
* if/then
* 로직/데이터 처리
* 암호화!
    * 해시
    * 서명 검증
    * 다중-서명 검증

OP_CHECKMULTISIG은 단일 서명 확인 보다 훨씬 강력한 명령어이다. n 공개키와 매개변수 t이 있으면, 이 명령은 검증을 수행한다.

그러나 여기엔 버그가 있다. ㅋㅋㅋㅋ 실제 추가적인 데이터 값을 스택에서 꺼내고 무시해버린다.

99.9%는 단일 서명 확인이고, 0.01%는 다중서명 검증, 0.01%는 Pay-to-Script-Hash이며, 나머지는 오류이다.

proof-of-burn은 절대 유효화될 수 없는 스크립트 이다. 이게 있으면 그 동전들이 파괴되어 더이상 사용될 수 없음을 나타낸다. 명령어로는 OP_RETURN을 사용하며, 스택에서 이를 만나면 프로그램이 종료(crash)된다. 

proof-of-burn을 하는 이유는 2가지가 있다.

// TBD

Pay-to-Script-Hash

// TBD

## Applications of Bitcoin Script

// 보충 필요

비트코인 스크립트에 이점에 대해서 살펴보자.

1. 조건부 날인 증서 트랜젝션(Escrow transactions)
2. Green addresses
3. Efficient micro-payments

## Bitcoin Blocks

여러가지 이유로 트랜젝션들을 하나로 묶어서 블록으로 생선한다. 첫 번째, 마이너가 작업하기 편한 하나의 단위를 생성하기 위해서이다. 두 번째, 블록의 해시 체인이 길이가 더 짧아져서 블록 체인 데이터 구조를 검증하기가 쉽다.

블록은 두 가지로 구성된다. 블록의 해시 체인과 모든 트랜젝션의 트리인 머클 트리이다.

블록의 해시 체인 부분은 또 블록 헤더와 이전 블록을 가리키는 해시 포인터로 구분된다.

```json
{
    // black header
    "hash": "000000000000000001aad2...",
    "ver": 2,
    "prev_block": "000000000000000003043....",
    "time": 1391279636,
    "bits": 419558700,
    "nonce": 459459841,

    // transaction data
    "mrkl_root": "89776....",
    "n_tx": 354,
    "size": 181520,
    "tx": [
        ...
    ],
    "mrkl_tree": [
        "6bd5eb25...",
        ...
        "89776cdb..."
    ]
}
```

블록 헤더는 주로 마이닝에 관련된 정보가 있다. 헤어에서 트렌젝션에 관련된 정보는 머클 루트 매개변수 하나이다.

머클 트리에 특별한 트랜젝션 하나가 있다. 바로 코인베이스 트랜젝션으로 새로운 코인과 비트코인이 생성되는 곳이다. 코인베이스 트랜젝션은 코인 보상과 트랜젝션 비용으로 이루어져 있다.

## The Bitcoin Network

// 보충 필요

3시간 이상 응답이 없으면 네트워크에서 제거됨

* P2P 가입
* 트랜잭션 전파(transaction propagation)
* Race condition

네트워크가 얼마나 큰지는 측정이 불가능하다.

완전히 검증된 노드는 영구적으로 연결되어 있고 모든 블록 체인을 저장하고 있으며, 모든 노드와 트랜젝션에 대해 듣고 전달한다.

UTXO(Unspent Transaction Output)

Thin clients나 Simple Payment Verification clients도 있다. 비트코인 네트워크의 대다수의 노드이며, 모든 블록체인을 가지지 않고, 검증 하기 위해 필요한 특정 트랜젝션만 저장한다.

## Limitations & Improvements

### 비트코인엔 하드코딩된 요소들이 많다.

* 블록하나의 평균 생성시간: 10분
* 블록의 크기: 1M
* 블록의 서명 명령: 20, 000
* 비트코인당 100M 사토시
* 최대 비트코인 수: 21M
* 비트코인 채굴 보상: 50, 25, 12.5 ...

### 비트코인의 처리량 한계

* 1M bytes/block
* > 250 bytes/transaction
* 7 transactions/sec

### 암호화 한계

* 오직 하나의 서명 알고리즘 (ECDSA/P256)
* 하드 코딩된 해시 함수

### Hard forking

// TBD

### Soft forking

// TBD

