---
layout: post
title: "Underhanded Solidity"
date: 2022-08-11
img: park.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [solidity]
---

"언더핸드(underhanded) 솔리디티 컨테스트"는 솔리디티 개발팀이 주최하는 스마트 컨트랙트 보안취약점 컨테스트입니다.  "언더핸드"라는 단어는 
겉으로 보기에는 전혀 문제 없지만 그 이면에 보안취약점을 가지고 있는 스마트 컨트랙트를 의미합니다. 참가자들이 제출한 컨트랙트들을 심사하여 가장 
"언더핸드", 즉 알아차리기 어렵지만 알고보면 정말 단순한 취약점을 지닌 컨트랙트 상위 3개를 골라 시상합니다. 

올해 2022년 수상작들은 솔리디티 [블로그][blog]에 공개되어 있습니다. 하나씩 살펴보도록 하겠습니다. 

## 3위 Michael Zhu
이 [컨트랙트][3rd]는 ERC20 토큰으로 ERC721 표준의 NFT 토큰을 구매하는 컨트랙트입니다. ["solmate"][solmate] 라는 컨트랙트 라이브러리를 활용하여 작성되어 있습니다.
입찰자(bidder)는 `createBid`를 호출하여 원하는 NFT 토큰과 가격을 제시합니다. 이 때 NFT의 가격은 지정된 ERC20 토큰으로 지불합니다.
생성된 입찰건은 다음과 같은 mapping 타입의 장부에 저장됩니다.

<font size="1">
{% highlight javascript %}
mapping(address => mapping(uint160 => mapping(uint256 => uint256))) bids;
{% endhighlight %}
</font>


복잡하게 보이지만 입찰자의 계정을 key로 하고, 두 토큰 컨트랙트의 주소를 XOR 연산한 uint160의 값, 그리고 NFT토큰 ID와 제시한 가격을 저장합니다. 해당되는 NFT 토큰을 가진 사람은 bids에 제시된 가격을 보고 입찰에 응하게 되고 `acceptBid`를 호출하여 제시된 수량의 ERC20 토큰을 수령하고 NFT 소유권을 넘겨 주게 됩니다. 

ERC20 토큰을 전송할 때 위임 전송 transferFrom을 사용합니다. 이미 입찰자는 approve를 통하여 일정 수량을 전송하도록 위임했을 것입니다. 마찬가지로 NFT 토큰 역시 
transferFrom을 사용합니다. 그런데 이 두 함수는 모두 동일한 이름과 동일한 파라미터들을 가지고 있음을 알 수 있습니다.

<font size="1">
{% highlight javascript %}
transferFrom(address _from, address _to, uint256 _value)
transferFrom(address _from, address _to, uint256 _tokenId)
{% endhighlight %}
</font>

공격자 A는 이를 이용하여 입찰자 B가 소유한 NFT를 훔칠 수 있습니다. 입찰자가 생성한 입찰 건을 사용하여, 공격자는 실제로 입찰자가 구매를 원하는 NFT가 없다고 해도 
다음과 같이 `acceptBid`를 호출하는 것이 가능합니다.

<font size="1">
{% highlight javascript %}
acceptBid(
    B,      // bidder
    ERC20,  // ERC721 contract
    420,    // ERC721 token ID
    ERC721, // ERC20 contract
    666     // price
)
{% endhighlight %}
</font>

그런데 ERC721 토큰 컨트랙트와 ERC20 토큰 컨트랙트 주소를 바꾸어서 호출하는 것입니다. price에는 B가 소유한 NFT 토큰 아이디를 넣습니다. transferFrom은 동일하므로 오류없이 실행이 될 것입니다. 그러나 결과는 전혀 다릅니다. 컨트랙트 주소가 바뀌었기 때문에 다음 코드는 NFT 아이디 666번을 A에게 전송하게 됩니다.

<font size="1">
{% highlight javascript %}
erc20Token.safeTransferFrom(B, A, 666);
{% endhighlight %}
</font>


그리고 반대로 B에게는 420 wei에 해당하는 ERC20 토큰만이 전송됩니다. 원래 B는 자신이 입찰한 NFT를 기대하고 있겠지만 오히려 NFT 666번을 도난당하고 소량의 ERC20 토큰만을 
수령하게 되는 것입니다.

<font size="1">
{% highlight javascript %}
erc721Token.transferFrom(A, B, 420);
{% endhighlight %}
</font>


각 토큰 컨트랙트 주소 자체를 파라미터로 받을 수 있도록 한 것이 문제일 수도 있고 또 스토리지를 절약하기 위해 두 컨트랙트를 XOR했는데 이 경우에 두 컨트랙트가 서로 바뀌어도 동일한 값이 나오므로 주소를 교환하여 호출하는 것이 가능했습니다.


## 2위 Santiago Palladino
Santiago Palladino는 오픈제펠린에서 잘 알려진 개발자입니다. 이것은 오더북 기반의 ERC20 토큰 판매를 간단하게 구현한 [Exchange][2nd] 컨트랙트 입니다. 오더북 기반의 토큰 판매는 온체인에서 처리되기 어렵기 때문에 오프체인에서 매수주문을 저장하고 조건이 맞으면(일부 또는 전체) 매도자가 서명을 하고 매수자가 그 데이터를 컨트랙트에 제출하는 방식으로 이루어집니다.

주문은 다음과 같은 구조체로 표현됩니다.

<font size="2">
{% highlight javascript %}
struct Order {
    address referrer;
    address token;
    uint128 rate;
    uint24 nonce;
    uint256 amount;
    uint8 orderType;
}
{% endhighlight %}
</font>

레퍼러(referrer)는 중개 수수료에 해당하고 1%를 정했습니다. 솔리디티에서는 소수점 연산이 불가능하므로 1%, 즉 0.01을 분수로 표현합니다. 
즉 분자(REFERRAL_FEE)를 100, 분모(REFERRAL_FEE_DENOMINATOR)를 10000으로 표현하여 0.0001까지 가능하도록 합니다. 분자는 어떤 비율을 나타내는 값으로 uint 타입에 
저장할 수 있고 나중에 실제 비율을 계산할 때는 분모로 나누어주게 됩니다.  

비율(rate)는 이더 당 판매되는 토큰의 수량입니다. 즉 지불한 이더에 대해 다음과 같은 수량의 토큰을 판매합니다. 여기서도 분수로 표현하기 위해 
RATE_DENOMINATOR = 2**64 를 사용합니다.

<font size="1">
{% highlight javascript %}
uint256 tokensPurchased = msg.value * order.rate / RATE_DENOMINATOR;
{% endhighlight %}
</font>
 

거래가 매칭되면 다음과 같이 토큰과 이더를 교환합니다. 여기서 `msg.sender`는 서명한 주문을 제출하는 매수인 계정이 됩니다.

<font size="2">
{% highlight javascript %}
IERC20(order.token).transferFrom(seller, msg.sender, tokensPurchased);
if (fee > 0) payable(order.referrer).transfer(fee);
payable(seller).transfer(msg.value - fee);
{% endhighlight %}
</font>


매수인은 주문정보를 Order 구조체의 각 항목들에 설정하여 오프체인에 저장합니다. 토큰 매도 체결은 이 주문을 매도인이 전자 서명함으로써 이루어집니다.
전자서명할 때는 주문정보를 컨트랙트의 `getOrderHash`라는 함수의 로직대로 해시한 후 서명합니다. 매수인은 이렇게 체결된 주문을 컨트랙트의 `executeOrder`을
호출하면서 전달합니다. 이 함수에서는 주문정보와 전자서명이 일치함을 확인하고 이상없으면 토큰과 이더를 각 거래 당사자들에게 전송합니다.

문제는 전자서명된 주문 데이터를 임의로 만들 수 있다는 것에 있습니다(사실 이것을 발견하기란 매우 어렵긴 하겠지만). 우선 정상적인 주문 데이터를 
만들겠습니다(값 자체는 중요하지 않음).


<font size="1">
{% highlight javascript %}

const referrer = "0xAd36...c9a1";
const token = "0x5FbD...0aa3";
const rate = 100;
const nonce = 0;
const amount = ethers.utils.parseEther("0.01");
const orderType = 0;

const order = [referrer, token, rate, nonce, amount, orderType];
{% endhighlight %}
</font>

이것을 컨트랙트의 `getOrderHash`에 전달하고 리턴받은 해시에 매도인이 서명을 하게 됩니다. 서명 대상이 되는 데이터는 컨트랙트 주소가 다시 포함되므로 
다음과 같은 데이터에 대한 해시를 서명한 것이 되겠습니다.

<font size="1">
{% highlight javascript %}
const packed = ethers.utils.solidityPack(
    ["address", "address", 
     "uint128", "uint24", 
     "address", "uint256", 
     "uint8"], 
    ["0xAd36...c9a1", "0x5FbD...0aa3", 
     100, 0, 
     "0x63ee...C849", "10000000000000000", 
     0]
);
{% endhighlight %}
</font>

해시 전의 encodePacked 된 데이터는 아래와 같습니다. Order 구조체의 각 항목에 해당되는 값들로 나누어서 볼 수 있습니다. 이 형태를 잘 눈여겨 보도록 합시다. 
0x5fbd...0aa3는 토큰 컨트랙트 주소이고, 0x63ee...c849는 Görli에 배포된 Exchange [컨트랙트][exchange] 입니다. 

<font size="1">
{% highlight shell %}
ad36...c9a1
5fbd...0aa3
00000000000000000000000000000064
000000
63ee...c849
000000...0000000002386f26fc10000
00
{% endhighlight %}
</font>

이 값을 해시한 것을 전자서명하면 다음과 같은 서명 데이터를 얻게 됩니다.

<font size="1">
{% highlight javascript %}
const orderHash = ethers.utils.keccak256(packed);
const {v,r,s} = new ethers.utils.SigningKey(sellerPrivateKey).signDigest(orderHash);

r = 0x20ee...fcb4
s = 0x6ecc...ef00
v = 28
{% endhighlight %}
</font>

만약에 이와 동일한 구조의 가짜 주문에 대해 이미 서명된 데이터를 얻을 수 있다면 매도인의 의사와 상관없이 토큰을 가져갈 수 있을 것입니다. 하지만 그게 가능할까요? 어떻게 서명이 포함된 가짜 주문을 만들 수 있을까요?

토큰 판매는 Exchange 컨트랙트가 토큰을 전송할 수 있도록 위임하는 approve 과정(매도인이 수행)이 필요합니다. approve 트랜잭션 해시를 이더스캔에서 조회하면 
전자 서명을 포함하여 더 많은 정보를 얻을 수 있습니다. 이 정보들을 사용하여 트랜잭션을 다시 구성해보겠습니다. 아래는 트랜잭션 [0x33fe...][txhash]의 
정보를 재구성한 것입니다.

<font size="1">
{% highlight javascript %}
const apporveTx = {
        type: 2,
        nonce: 8, 
        to: "0x1160e6cB390B8D756AEbF52eda0D74cf97272749",
        value: "0x0",         
        maxPriorityFeePerGas: "0x9502f900",
        maxFeePerGas: "0x4a817c800",
        gasLimit: "0xb73a",
        data: "0x095ea7b300000000000000000000000063ee5864f7fa0becfcee56093d654120e7e3c8490000000000000000000000000000000000000000000422ca8b0a00a425000000",
        chainId: 5,
}
{% endhighlight %}
</font>

여기서 to 는 토큰 컨트랙트(0x1160...)이고 data 항목에 들어간 것이 바로 approve 호출로, Exchange 컨트랙트(0x63ee...)에 전체 수량을 전송할 수 있도록 위임한 것을 알 수 있습니다. 이것을 다시 RLP serialize하면 다음 결과를 얻게 됩니다.

<font size="1">
{% highlight javascript %}
ethers.utils.serializeTransaction(apporveTx).slice(2);

02f86d0508849502f9008504a817c80082b73a941160e6cb390b8d756aebf52eda0d74cf9727274980b844095ea7b300000000000000000000000063ee5864f7fa0becfcee56093d654120e7e3c8490000000000000000000000000000000000000000000422ca8b0a00a425000000c0
{% endhighlight %}
</font>

이렇게 인코딩된 것을 Order 구조체의 타입에 맞추어 다시 쓸 수 있습니다.

<font size="1">
{% highlight shell %}
02f8...3a94
1160...2749
80b844095ea7b3000000000000000000
000000
63ee...c849
000000...00000000422ca8b0a00a425000000                                                                 
c0
{% endhighlight %}
</font>

이렇게 나누어서 보면 놀랍게도 앞서 Order 구조체의 각 필드 타입들과 잘 일치하는 것을 알게 됩니다. 토큰 컨트랙트 주소 0x1160...2749도 정확히 일치합니다. 
교환비율인 rate는 매우 큰 값으로 매수인에게 유리하게 작용합니다. 적은 이더를 가지고도 많은 수량의 토큰을 받을 수 있으니까 말입니다. 마지막 c0는 
orderType인데 0이나 1의 값을 가져야 하지만 컨트랙트 로직에서는 0이 아니면 전부 "PARTIAL" 판매로 간주하므로 전혀 문제 되지 않습니다.

더 재미있는 것은 이미 이 데이터에 대한 서명(r,s,v)까지도 확보했다는 것입니다. 다시 말해서 매도인의 approve 트랜잭션을 그대로 가져다가 서명된 주문 
데이터로 사용할 수 있다는 의미가 되겠습니다.  

하지만 항상 모든 approve 트랜잭션이 Order 구조체와 일치하는 것은 아닌 것 같습니다. maxFeePerGas 값이 작아서 매칭이 안되는 경우가 있었습니다. 
또 비율(rate)이 지나치게 높으므로 허용 매도수량보다 크면 거래가 되지 않을 것입니다. 주문에 대한 서명을 해보면 approve의 서명 값과 정확히 일치함을 알 수 있습니다.

<font size="1">
{% highlight javascript %}
const packed = ethers.utils.solidityPack(
    ["address", "address", 
     "uint128", "uint24", 
     "address", "uint256", 
     "uint8"], 
    ["0x02f86...3a94", "0x1160...2749", 
     "0x80b8...0000", "0x000000", 
     "0x63ee...C849", "0x0000...0000422ca8b0a00a425000000", 
     "0xc0"]
    );

const orderHash = ethers.utils.keccak256(packed);
const { v, r, s } = new ethers.utils.SigningKey(sellerPrivateKey).signDigest(orderHash);

r = 0x4e99...c57f
s = 0x46c8...df35
v = 27
{% endhighlight %}
</font>

결과적으로 토큰 홀더들의 approve 트랜잭션 중에 이렇게 일치하는 트랜잭션을 골라서 가짜 주문을 만들 수 있고 그 서명 값도 동일하므로 이것을 전송하여 
마치 정상적인 주문이 체결되는 것처럼 할 수 있습니다.


## 1위 Tynan Richards

이 [컨트랙트][1st]는 유니스왑과 유사한 간단한 DEX를 구현한 것으로 솔리디티 언어 자체가 보안 취약점이 될 수 있음을 보여주고 있습니다. 컨트랙트는 페어로 이루어진 
토큰 수량의 곱이 상수로 유지되면서(constant product) 가격이 유지되는 일반적인 마켓메이커를 구현합니다.

유동성은 페어된 토큰 수량의 기하평균(geometric mean)으로 평가되는데 sqrt(x*y)로 계산하며, 컨트랙트 내에서 sqrt에 대한 연산을 수행하는 
함수를 뉴턴-랩슨 방법으로 구현하였습니다. 유동성을 제공하는 사용자들에게는 LP토큰을 지급하여 잔액이 점차 증가되고 나중에는 거래 수수료가 
붙어서 인출할 수 있게 됩니다.

컨트랙트 관리자도 수수료를 받습니다. 이것을 관리수수료(admin fee)라고 하여 1%를 책정했는데, 만약 해당 풀의 유동성이 20% 증가했다면 관리자는 0.2%의 수수료를 
가져갈 수 있습니다. 그런데 여기서 제약조건이 하나 추가됩니다. 관리 수수료를 인상하는 경우 유동성 공급자들은 인상 전에 자신들이 제공한 토큰들을 
인출할 수 있습니다. 관리 수수료가 인상되면 7일 후에 적용되므로 수수료율이 높다고 생각되면 그 기간동안 인출하도록 하는 것입니다.

<font size="1">
{% highlight javascript %}

uint256 feeAmount = (amountAccrued * adminFee) / ONE;
uint256 liquidity = geometricMean(_balanceA, _balanceB);
uint256 amountA = (feeAmount * _balanceA) / liquidity;
uint256 amountB = (feeAmount * _balanceB) / liquidity;

{% endhighlight %}
</font>

관리 수수료는 풀의 가장 최근 유동성(amountAccrued)에 대하여 수수료율을 적용합니다. 페어를 이루는 각 토큰의 잔량에 대해 `feeAmount/liquidity` 만큼 
관리자 계정으로 전송합니다. 

앞서 말한 것처럼 이 수수료율을 높이는 경우 7일 후에 적용이 됩니다. 수수료율을 변경하는 함수는 `changeAdminFee`입니다.

<font size="1">
{% highlight javascript %}
function changeAdminFees(uint256 newAdminFee) external onlyAdmin nonReentrant {
    emit AdminFeeChanged(retireOldAdminFee(), setNewAdminFee(newAdminFee));
}
{% endhighlight %}
</font>

`retireOldAdminFee`는 기존 비율이 적용된 관리 수수료를 먼저 인출하는 함수이고 다음에 새로운 수수료율을 적용하는 함수 `setNewAdminFee`을 호출하도록 되어 있습니다. 

<font size="1">
{% highlight javascript %}

function retireOldAdminFee() internal returns (uint256) {
    // Claim admin fee before changing it
    _claimAdminFees();
    // Let people withdraw their funds if they don't like the new fee
    nextFeeClaimTimestamp = block.timestamp + 7 days;

    return adminFee;
}
{% endhighlight %}
</font>

그런데 문제는 여기에 있습니다. 이벤트 `AdminFeeChanged`를 발생시키면서 그 안에 인라인으로 수수료 인출과 수수료 설정 함수를 호출했습니다. 일반적으로, 그리고 당연히 
순서대로 실행될 것이라고 생각합니다. 하지만 과연 그럴까요? 물론 일반적으로 앞에 있는 것이 먼저 실행되고 그 다음이 실행되는 것은 맞습니다. 그러나 이벤트의 경우는 그 반대라는 사실을 미처 몰랐을 것입니다.

`setNewAdminFee`이 먼저 실행되기 때문에 7일 후에 적용되는 로직은 아무 의미가 없습니다. 즉 관리자가 인상된 수수료율을 즉시 적용할 수 있다는 말이 되겠습니다. 만약 관리자가 터무니 없이 수수료를 높이는 경우, 컨트랙트 코드만 보고 7일의 유예 기간을 그대로 믿은 유동성 공급자들은 낭패를 볼 것입니다.

좀 더 간단한 예를 들어보도록 하겠습니다. 솔리디티 기본 내장 함수에 `addmod(x,y,z)`가 있습니다. 이것은 `(x+y)%z` 입니다. 

<font size="2">
{% highlight javascript %}
addmod(10,5,2); // 1
{% endhighlight %}
</font>

만약 다음과 같은 경우는 어떻게 계산될까요?

<font size="2">
{% highlight javascript %}

uint256 i = 5;
uint256 result = addmod(i, 5, i++); 

{% endhighlight %}
</font>

아마 대부분은 이렇게 계산할 것입니다. `addmod(5,5,6)` 그래서 결과는 4라고 말입니다(또는 (5+5)%5=0). 그러나 실제로는 `addmod(6,5,5)`가 되어 1이 나옵니다. 나머지 연산은 
나누는 수가 0이 되면 의미가 없기 때문에 세번째 파라미터의 표현식부터 평가합니다. 따라서 5를 리턴하고 나서 하나를 증가시키고 그래서 첫번째 i에는 6이 들어갑니다.

이것은 사실 버그도 아니고 단순히 솔리디티의 특징에 기인한 것으로 개발자들의 상식(?)을 잠깐 벗어나는 부분일 수 있습니다. 올해 "언더핸드" 컨테스트의 취지에 걸맞는 
문제라고 할 수 있을 것 같습니다.


[blog]: https://blog.soliditylang.org/2022/04/09/announcing-the-underhanded-contest-winners-2022/
[solmate]: https://github.com/transmissions11/solmate
[3rd]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission17_MichaelZhu
[2nd]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission10_SantiagoPalladino
[1st]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission9_TynanRichards
[exchange]: https://goerli.etherscan.io/address/0x63ee5864f7fa0becfcee56093d654120e7e3c849
[txhash]: https://goerli.etherscan.io/tx/0x33fe4bcc1231956806b8c454b4b4422aec64da237f69adf22d81bbc0534e37da
