---
layout: post
title: "Layer 2 scaling(2)"
date: 2021-06-07
img: circuit.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [etherum, layer2]
---

비탈릭 부테린은 "[롤업 중심의 로드맵][rollup-centric]"이라는 글에서 오랜 인내(?)가 필요한 이더리움 PoS 전환 과정에서 L2 솔루션이 중요한 역할을 담당해야 한다는 주장을 했습니다. L1에서 성능 문제가 확실히 개선되기까지는 아직 갈 길이 먼데, 최근 다양한 L2 솔루션들, "옵티미스틱 롤업", "Fuel", "Arbitrum", 
"Loopring", "zkSync" 등의 롤업 기반 솔루션들이 큰 진전을 보이고 있으므로 이들 솔루션들이 원활하게 동작할 수 있도록 이더리움의 개발 로드맵의 우선 순위를 조정해야 한다는 취지의 의견을 내놓은 것입니다.

>In a further twist of irony, eth2’s usability as a data availability layer for rollups comes in phase 1, long before eth2 becomes usable for 
“traditional” layer-1 applications. These facts taken together lead to a particular conclusion: the Ethereum ecosystem is likely to 
be all-in on rollups (plus some plasma and channels) as a scaling strategy for the near and mid-term future.

즉 이더리움 2.0이 L2 트랜잭션 데이터의 "data availability layer"의 용도로 사용되면 결국 L2 롤업 솔루션들이 이더리움과 마치 
하나처럼 움직이게 될 것이라는 의미가 되겠습니다. 

앞서 플라즈마가 가진 문제는 L2에서 벌어지는 트랜잭션에 대한 "Data availability"가 확보되지 않을 수 있다는 것이었는데, 롤업은 그것을 해결하면서 처리 성능을 높일 수 있다는 것입니다. 비탈릭 부테린은 심지어 롤업이 정착되면 Phase 2에서 샤딩이 구현되어 성능 개선이 되더라도 사람들이 관심을 끌지 못할 것이라는 말도 하고 있습니다.

>It seems very plausible to me that when phase 2 finally comes, essentially no one will care about it.

그렇다면 "롤업"이 무엇이길래 이렇게 많은 지지와 관심을 받는 것일까요?  

롤업은 한마디로 말하면 트랜잭션 실행은 L2에서 수행하지만 
데이터를 L1에 기록하므로써 L1이 가진 높은 신뢰성을 이용하자는 것입니다. 물론 단순히 트랜잭션 데이터 그대로 저장하는 것은 아니고, 데이터를 압축하고 암호학적으로 
증명 가능한 방법으로 저장하는 것입니다.

현재 롤업에는 ZK롤업(ZK-rollups)과 옵티미스틱 롤업(Optimistic rollups) 2가지가 있습니다. 두 방식의 가장 큰 차이점은 
ZK롤업은 "validity proof"이고 옵티미스틱 롤업은 "fraud proof" 라는 것입니다. 이더리움 개발자 사이트에 있는 롤업의 [설명](https://ethereum.org/en/developers/docs/scaling/layer-2-rollups/)을 인용하면:

>Zero knowledge rollups: runs computation off-chain and submits a validity proof to the chain
Optimistic rollups: assumes transactions are valid by default and only runs computation, via a fraud proof, in the event of a challenge


### ZK Rollup

트랜잭션 데이터에 대한 신뢰성을 어떻게 담보할 것인가는 L2 솔루션에서 공통적인 문제입니다. 그렇다고 L2에서 일어나는 트랜잭션을 
항상 감시하는 것은 비효율적인 일입니다. zkSync는 ZK롤업을 실제 이더리움 메인넷에 적용한 좋은 사례입니다. zkSync를 개발한 Matter Labs의 [설명](https://medium.com/matter-labs/optimistic-vs-zk-rollup-deep-dive-ea141e71e075)을 인용해 보겠습니다.

>In a ZK-Rollup, operator(s) must generate a succinct Zero-Knowledge Proof (SNARK) for every state transition, 
which is verified by the Rollup contract on the mainchain. This SNARK proves that there exists a series of transactions, 
correctly signed by owners, which update the account balances in the correct way, and which lead from the old Merkle root to the new one. 
It is thus impossible for the operators to commit an invalid or manipulated state.   

L2의 오퍼레이터는 트랜잭션을 "영지식증명(Zero Knowlegde Proof)"에 기반한 "SNARK(Succinct Non-interactive Argument of Knowledge)"이라는 증명(proof)으로 만들어서 L1의 롤업 컨트랙트의 검증을 받습니다. 
그런데 이 "SNARK"은 트랜잭션들의 서명과 잔액의 증감이 올바르다는 것을 암호학적으로 증명하는 것이므로 오퍼레이터가 임의로 조작할 수 없습니다.
이렇게 트랜잭션의 유효성이 검증(validity proof)되기 때문에 최종 상태변경은 즉시 L1에 반영될 수 있습니다. 소위 말하는 트랜잭션에 대해 이의를 제기하는 시간을 주는 "challenge period"가 필요하지 않은 것입니다.

"영지식증명"이라는 것이 익명성을 보장해주는 암호기술이라고 알려져 있지만 롤업에서는 익명성보다는 L2에서 발생한 많은 트랜잭션을 전부 공개하지 않고도 트랜잭션들이 모두 참이라는 것을 증명하는 것입니다. 결국 트랜잭션 데이터를 L1에 유지하면서도 암호학에 기반한 검증을 "온체인"에서 수행할 수 있다는 것이 ZK롤업의 가장 큰 장점이라고 할 수 있습니다. L2의 트랜잭션의 유효성을 L1에서 합의할 수 있게 되는 것입니다.

ZK롤업에서 영지식증명의 요소는 다음과 같이 매칭될 수 있습니다.

1. Secret:  공개하지 않는 것(트랜잭션 데이터)
2. Prover: SNARK를 만들어서 L1에 제출(Relayers)
3. Verifier: L1의 롤업 스마트 컨트랙트

하지만 영지식증명에 기반한 "validity proof"를 검증하는데 드는 가스비용이 다소 크다는 단점이 있습니다(비탈릭 부테릭 [글][guide_l2]에 의하면 약 50만 가스가 소비). 또 아직까지는 이더나 토큰 전송같은 송금 트랜잭션만이 가능합니다.

### 영지식증명(Zero Knowlegde Proof)  

도대체 ZK롤업에서 사용하는 "영지식증명"이라는 "마법"은 무엇일까요? 두 개의 비유적 설명을 살펴보겠습니다. 
먼저 영지식증명을 설명할 때 자주 나오는 예는 "알리바바의 동굴"입니다. 다음과 같은 상황을 제시합니다.

1. Verifier와 Prover는 동굴 밖에 있습니다. 
2. Prover는 자신이 동굴 안에 있는 잠겨진 문을 열 수 있는 키가 있다고 주장합니다.
3. Verifier는 함께 들어가서 문을 열어보자고 합니다. 또는 키를 자신에게 주면 직접 들어가서 확인하겠다고 합니다.
4. Prover는 자신의 키를 노출시키지 않고 Verifier에게 키를 가지고 있음을 증명하려고 합니다.

영지식증명은 키를 전혀 노출시키지 않고서도 이것이 가능함을 증명할 수 있는 방법입니다.

![fig01]({{site.baseurl}}/assets/img/cave.PNG)

1. Verifier는 동굴 밖에서 기다리고 Prover가 동굴 안으로 먼저 들어갑니다. 동굴 밖에서는 동굴 안이 전혀 보이지 않습니다.
2. Prover는 A 또는 B 중 하나의 경로를 선택할 수 있습니다. A로 들어갔다고 가정해보겠습니다.
3. 이제 Verifier가 동굴 안으로 들어갑니다. 하지만 Prover는 이미 사라진 상태이기 때문에 보이지 않습니다.
4. Verifier는 Prover에게 B로 나오라고 소리칩니다.
5. Prover는 가지고 있는 키로 문을 열고 B로 나옵니다.
6. Verifier는 처음에 Prover가 B 방향으로 들어가서 그냥 되돌아 나온 것일 수도 있다는 합리적인 의심을 할 수 있습니다.
7. Verifier와 Prover는 다시 첫번째 단계로 되돌아서 같은 일을 반복합니다. 

매번 어느 쪽으로 나오라고 할지 알 수 없지만 Prover는 키를 가지고 있으므로 상관없을 겁니다. 만약에 Prover가 거짓말을 하고 있다면? 20번을 반복했을 때 
Prover가 키를 가지고 있지 않음에도 Verifier의 요구를 모두 충족시킬 확률은 어느 정도 일까요? A와 B 확률 50% 이므로 백만분의 1 정도의 
확률(1/2 * 1/2 * ...)을 가집니다. 즉 Prover가 키를 모를 확률이 백만분의 1이라는 것입니다.
Verifier는 충분히 많은 반복을 시행한 후에 드디어 Prover가 키를 가지고 있음을 확신하게 됩니다.

또 한가지 설명은 존스홉킨스 컴퓨터학 교수 매튜 그린이 쓴 [글][zkp]입니다.


[zkp]: https://blog.cryptographyengineering.com/2014/11/27/zero-knowledge-proofs-illustrated-primer/
[rollup-centric]: https://ethereum-magicians.org/t/a-rollup-centric-ethereum-roadmap/4698
[guide_l2]: https://vitalik.ca/general/2021/01/05/rollup.html
