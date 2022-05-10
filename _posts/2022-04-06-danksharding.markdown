---
layout: post
title: "Danksharding"
date: 2022-04-06
img: leaves.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [ethereum, danksharding]
---

최근에 이더리움의 레이어 2 솔루션들이 다양하게 나오면서 성능 개선에 대한 기대감도 함께 오르고 있습니다. 레이어 2의 목적은 "Scaling Ethereum without compromising on security or decentralization." 이라고 요약할 수 있겠습니다. 
거래 중개자들을 일방적으로 믿기 보다는 현재 탈중앙화된 "trustless" 이더리움에 거래 데이터와 검증 자료를 기록하여(합의를 거쳐) 레이어 2의 신뢰성을 담보하는 것입니다.

그런데 기대가 커진 만큼 실망 역시 그에 비례하는 것 같습니다. 생각처럼 레이어 2가 활성화되지 못하고 있다는 비판이 있습니다. 
높은 가스비로 인하여 거래 데이터, 즉 롤업 데이터를 합의 레이어인 이더리움에 기록하는 비용이 증가하기 때문입니다.

[레이어 2의 거래 비용][l2fee]을 보면(2022년 4월) 이더 송금의 경우 약 3-6배의 비용 절감 효과를 가지고 있습니다. 오래 전 비탈릭 부테린은 수수료가 5센트를 넘어가면 
안된다고 말한 적이 있습니다. 지금 수준과 비교하면 아직도 10배 이상 저렴해야 합니다.

이더리움의 성능 개선은 샤딩(Sharding)으로 실현됩니다. 비탈릭 부테린이 쓴 [글][blob-tx]을 살펴보겠습니다.  

>However, even these fees are too expensive for many users. The long-term solution to the long-term inadequacy of rollups by themselves has always been data sharding, which would add ~16 MB per block of dedicated data space to the chain 
that rollups could use. However, data sharding will still take a considerable amount of time to finish implementing and deploying. 

현재 레이어 2의 수수료는 일반 사용자들에게 여전히 높고, 또 장기적으로 보면 레이어 2 만으로는 충분한 성능 개선이 이루어지기 어렵기 때문에 
앞으로 도입될 데이터 샤딩의 저장 영역(블록당 16 MB)에 롤업 데이터를 저장하여 레이어 2의 활용도를 끌어올리자는 취지입니다. 

원래 이더리움 PoS의 로드맵 Phase 1 샤딩의 목적은 현재 단일 체인을 64개의 샤드 체인으로 분할해서 거래를 나누어 처리하는 것이었습니다. 비콘 체인의 검증자들로 구성된 committee를 다수의 샤드에 각각 배치하여 샤드 블록을 만들고 비콘 체인에 그 내역을 기록하는 방식입니다. 트랜잭션은 실행 레이어에서 이루어지기 때문에 샤드 체인에서는 데이터만 저장합니다. 그래서 샤드 체인을 "데이터 레이어"로 표현합니다.

각 샤드는 실행 레이어를 가지고 있게 되고 현재 이더리움도 EVM 실행 엔진이 달린 한 샤드로 볼 수 있습니다. 지금과 비교해보면 실행 레이어는 현재 레이어 2가 담당하고 (데이터 레이어에 해당하는 샤드 체인은 아직 없으므로) 이더리움 블록에 롤업 데이터(calldata)를 저장하는 구조로 생각할 수 있겠습니다.

그런데 레이어 2의 비용을 체감적으로 크게 낮출 수 없는 이유 중 하나는 롤업 데이터를 저장하는 트랜잭션은 어차피 이더리움에 전송하기 위해 가스비를 내야 하므로 
가스비가 매우 높은 상황에서는 레이어 2의 효과가 감소한다는 문제가 있습니다.

비탈릭 부테린은 두 가지 정도의 제안을 내놓은 상태입니다. [EIP-4488][eip-4488]과 [EIP-4844][eip-4844]인데 장기적인 방향과 일치하는 것은 EIP-4844입니다. 여기서는 다음과 같은 "stop-gap" 솔루션을 제안합니다.

>This EIP provides a stop-gap solution until that point by implementing the transaction format that would be used in sharding, 
but not actually sharding those transactions. Instead, the data from this transaction format is simply part of the beacon chain 
and is fully downloaded by all consensus nodes (but can be deleted after only a relatively short delay). 
Compared to full data sharding, this EIP has a reduced cap on the number of these transactions that can be included, 
corresponding to a target of 1 MB per block and a limit of 2 MB.

앞에서 언급한 것처럼 샤딩 데이터의 저장 영역을 롤업 데이터 전용 저장 공간으로 활용하자는 것입니다. 향후 샤딩의 설계와도 호환(forwards-compatible)되도록 
구현하고 또 블록 당 약 1 MB에서 최대 2 MB로 제한한다는 것입니다.

여기서 "shard blob transaction"은 트랜잭션 타입을 0x05로 지정한 통상적인 L1 트랜잭션으로 취급하여 전송된 데이터를 비콘 체인에 저장하게 됩니다. 지금 롤업 데이터는 calldata를 의미하는 것이지만 나중에 샤딩이 구현되면 롤업 데이터는 "shard blob"에 해당되는 것입니다. 

"blob"이라고 표현한 것은 샤딩 [스펙][shard-spec]에 따르면 "Data with commitments and meta-data, like a flattened bundle of L2 transactions"으로 정의합니다. "blob"이라는 용어를 사용한 것은 현재 단일 체인의 전체 데이터를 각 노드들이 가지고 있는 것과 비교하여 샤딩된(나누어진, 부분) 데이터라는 의미로, 그 자체로 검증 가능한 정보(commitment)도 함께 가지고 있는 형태로 이해할 수 있을 것 같습니다.  

- Data: A list of KZG points, to translate a byte string into  
- Blob: Data with commitments and meta-data, like a flattened bundle of L2 transactions.  
- Builder: Independent actor that builds blobs and bids for proposal slots via fee-paying blob-headers, responsible for availability.  
- Shard proposer: Validator taking bids from blob builders for shard data opportunity, co-signs with builder to propose the blob.  

EIP-4844는 새로운 샤딩 방식인 [Danksharding][danksharding]과 관련이 있습니다.

>Here is an alternative proposal: Add a new type of transaction that can contain additional sharded data as calldata. 
The block is then constructed by a single block builder (which can be different from the proposer using proposer builder separation), 
and includes normal transactions as well as transactions with sharded calldata. 
This is efficient because it means that tight integrations between rollups and L1 become possible, 
and it is expected that this “super-block-builder” strategy will emerge in practice anyway in order to maximize MEV extraction.

사실 Danksharding은 샤딩의 본래 목적에서 다소 이탈한 샤딩이라고 말할 수 있습니다. 왜냐하면 샤딩이란 각 샤드에서 트랜잭션을 나누어 (병렬)처리하는 것인데 
Danksharding은 데이터 가용성(Data Availability)만을 보장하고 실행은 오프-체인, 즉 레이어 2에서 하는 것을 전제로 하기 때문입니다. 말하자면 성능 개선의 두 가지 측면에서, 
처리 속도는 레이어 2에서 올리고 수수료는 데이터 샤딩을 통해 낮추려는 의도라고 볼 수 있겠습니다.

이 제안에서는 각 샤드에 배치된 committee에서 샤드 블록을 만드는 대신 새로운 타입의 트랜잭션, 즉 "blob transaction"을 정의하고 
블록 생성자 하나가 이 트랜잭션들을 비콘 블록에 저장하도록 하는 것입니다. blob 트랜잭션은 거래 트랜잭션이 아니라 데이터를 저장하는 트랜잭션입니다. "The Merge"가 완료되면 비콘 체인 검증자들이 기존 이더리움(EL)의 트랜잭션과 샤드 blob 트랜잭션까지 처리하게 되는 셈인데, 이렇게 되면 상당한 부하를 처리할 수 있는 
하드웨어가 필요할 수도 있습니다("super-block-builder" 라고 표현한 이유).

그래서 PBS(Proposer/Builder Separation)라는 방식을 도입하여 블록 바디를 만드는 생성자(Builder)와 블록헤더를 만들어서 최종 블록을 
네트워크에 전파하는 제안자(Proposer)를 분리합니다. 마치 채굴자처럼 실제 트랜잭션을 모으는 것은 생성자의 역할이고 제안자(현재 비콘 체인 검증자들)는 그렇게 만들어진 블록 바디들 중에 하나를 선택하는 방식입니다(생성자가 블록을 비딩하고 제안자가 선택).

이러한 "수퍼 블록" 생성자는 누구나 될 수 있기 때문에 검열의 문제가 발생할 수 있습니다(생성자는 honest minority에 의존). 이를 방지하기 위해 검열 저항 목록(crList)를 제안자가 만들어서 브로드캐스팅합니다. crList는 트랜잭션 풀에 있는 임의의 트랙잭션들입니다. 블록 생성자는 crList에 있는 트랜잭션들을 포함시켜서 블록을 만들어야 하므로 생성자가 블록에 저장되는 트랜잭션을 임의로 제외할 수 없게 됩니다.

다시 말해서 블록을 만드는 일, 즉 데이터를 처리하는 일은 다소 높은 하드웨어 성능을 요구할 수 있기 때문에 컴퓨팅 리소스가 충분한 사람들에게 일임하려는 의도를 가지고 
있는 것 같습니다. 하지만 그것을 검증하는 것은 불특정 다수(honest majority에 의존), 즉 현재 비콘 체인의 검증자들이 수행하도록 하므로써 탈중앙화를 유지할 수 있습니다.

원래 각 샤드에서 거래가 처리되면 샤드마다 수수료가 달라질 수 있는데, Danksharding에서는 blob 트랜잭션의 타입을 하나 더 추가할 뿐이므로 좀더 간소화된 수수료 처리가 
가능합니다(일반 트랜잭션과 다른 수수료를 적용하는 것도 가능). 이것을 "merged fee market"이라고 말할 수 있습니다.

>The main innovation introduced by Danksharding (see also: [1] [2] [3]) is the merged fee market: instead of there being a fixed number of 
shards that each have distinct blocks and distinct block proposers, in Danksharding there is only one proposer that chooses all transactions 
and all data that go into that slot.

>To avoid this design forcing high system requirements on validators, we introduce proposer/builder separation (PBS) (see also: [1] [2]): 
a specialized class of actors called block builders bid on the right to choose the contents of the slot, and the proposer need only 
select the valid header with the highest bid. Only the block builder needs to process the entire block (and even there, it’s possible 
to use third-party decentralized oracle protocols to implement a distributed block builder); 
all other validators and users can verify the blocks very efficiently through data availability sampling (remember: the “big” part of the block is 
just data).

Danksharding의 설계는 아직 논의가 더 필요하고 스펙이 정해지더라도 구현 과정에서 문제점들이 나타날 수 있으므로 변경 가능성이 있습니다.
그래서 일단 프로토타입처럼 일부 기능을 먼저 구현하는 방식도 생각하고 있습니다. 그것이 [proto-Danksharding][proto-dank]입니다. 

앞서 말한 것처럼 데이터를 저장하는 이유는 데이터 가용성을 제공하기 위함입니다. 거래의 결과(상태 전이)가 올바른지 검증하기 위해서는 그 과정에서 발생한 데이터를 일정 기간 저장할 필요가 있는 것입니다.  

블록체인에서 거래 데이터가 올바르게 저장되었는지 확인하는 방법 중 하나는 머클 트리를 이용하는 것입니다. 어떤 블록에 저장된 거래를 확인하려면 머클 증명과 머클 루트를 사용하여 확인할 수 있습니다. Danksharding에서는 머클 트리 대신 "KZG commitment"라는 암호학 이론을 적용합니다. 

[Danksharding 세미나][danksharding-webinar]에서 Dankrad Feist의 설명을 인용하면 KZG commitment는 polynomial commitment의 한 종류로 다음과 같은 수학적 특성을 가지고 있습니다.

- 다항식 f에 대한 commitment(commitment to polynomial) = C(f)
- Prover는 z의 proof를 계산 = π(f,z)
- Verifier는 C, π, y, z를 사용하여 f(z) = y 임을 확인 

머클 트리에서는 데이터들을 차례로 해시하여 머클 루트로 표현할 수 있는데, polynomial commitment에서는 이들 데이터가 어떤 다항식의 값이 된다고 생각합니다. 


EIP-4844에 있는 `BlobTransaction`은 현재 다음과 같이 정의되어 있습니다. 이더리움의 트랜잭션은 EIP-2718에 의해 트랜잭션 타입을 지정할 수 있으므로 트랜잭션 타입을 별도로 지정하여 일반 트랜잭션과 구분합니다. 

```
class SignedBlobTransaction(Container):
    message: BlobTransaction
    signature: ECDSASignature

class BlobTransaction(Container):
    chain_id: uint256
    nonce: uint64
    priority_fee_per_gas: uint256
    max_basefee_per_gas: uint256
    gas: uint64
    to: Union[None, Address] # Address = Bytes20
    value: uint256
    data: ByteList[MAX_CALLDATA_SIZE]
    access_list: List[AccessTuple, MAX_ACCESS_LIST_SIZE]
    blob_versioned_hashes: List[VersionedHash, MAX_VERSIONED_HASHES_LIST_SIZE]
```    

BlobTransaction은 일반 트랜잭션처럼 data 필드에 calldata를 운반합니다. MAX_CALLDATA_SIZE는 16M(2**24)로 지정되어 있습니다(한 슬롯에서 저장할 수 있는 샤드 데이터의 최대 크기는 16MB). 이 트랜잭션은 다시 아래와 같은 네트워크 래퍼로 전송됩니다.

```
class BlobTransactionNetworkWrapper(Container):
    tx: SignedBlobTransaction
    # KZGCommitment = Bytes48
    blob_kzgs: List[KZGCommitment, MAX_TX_WRAP_KZG_COMMITMENTS]
    # BLSFieldElement = uint256
    blobs: List[Vector[BLSFieldElement, FIELD_ELEMENTS_PER_BLOB], LIMIT_BLOBS_PER_TX]
```

`blob_kzgs`는 blobs에 대한 각각의 KZG commitment 리스트입니다. blobs는 "유한체 위의 다항식(polynomial over finite field)"으로 4096개의 원소(BLSFieldElement)로 이루어진 벡터입니다. KZG commitment는 이것에 대한 commitment를 계산한 값이 됩니다. 요약하면 데이터와 그것을 검증할 수 있는 암호학적 검증 데이터들이 함께 래핑되어 있습니다(수학적으로 정확하게 이해하는 것이 어렵지만🤔).

옵티미스틱 롤업과 ZK 롤업에서 `blob_kzg`와 `blobs`를 각각 validity proof와 fraud proof를 수행하는 과정에서 이용할 수 있도록 
"precompile"을 제공하는 것도 EIP-4844에 포함되어 있습니다(point evaluation precompile, blob verification precompile). 온체인에서 데이터 가용성을 확인할 수 있는 기능을 제공한다는 의미가 될 것 같습니다.

proto-Danksharding은 아직 연구 단계에 있습니다. 또 "The Merge"라는 중요한 하드포크 이후에 적용될 가능성이 많습니다. 최소 1년 정도는 더 기다려야 구체적인 
세부 내용들이 나올 것 같다는 생각이 듭니다.


[l2fee]: https://l2fees.info/
[blob-tx]: https://notes.ethereum.org/@vbuterin/blob_transactions_simple
[eip-4488]: https://eips.ethereum.org/EIPS/eip-4488
[eip-4844]: https://eips.ethereum.org/EIPS/eip-4844
[shard-spec]: https://github.com/ethereum/consensus-specs/blob/dev/specs/sharding/beacon-chain.md
[danksharding]: https://notes.ethereum.org/@dankrad/new_sharding
[proto-dank]: https://notes.ethereum.org/@vbuterin/proto_danksharding_faq
[danksharding-webinar]: https://www.youtube.com/watch?v=e9oudTr5BE4