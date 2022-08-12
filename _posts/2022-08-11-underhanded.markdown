---
layout: post
title: "Underhanded Solidity"
date: 2022-08-11
img: park.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [solidity]
---

"언더핸드(underhanded) 솔리디티 컨테스트"는 솔리디티 개발팀이 주최하는 스마트 컨트랙트 보안취약점 컨테스트입니다.  "언더핸드"라는 단어는 
겉으로 보기에는 전혀 문제 없지만 그 이면에 보안취약점을 가지고 있는 스마트컨트랙트를 의미합니다. 참가자들이 제출한 컨트랙트들을 심사하여 가장 
"언더핸드", 즉 알아차리기 어렵지만 알고보면 정말 단순한 취약점을 지닌 컨트랙트 상위 3개를 골라 시상합니다. 

올해 2022년 수상작들은 솔리디티 [블로그][blog]에 공개되어 있습니다. 하나씩 살펴보도록 하겠습니다. 

## 3위 Michael Zhu
이 [컨트랙트][3rd]는 ERC20 토큰으로 ERC721 표준의 NFT 토큰을 구매하는 컨트랙트입니다. [solmate][solmate]라는 컨트랙트 라이브러리를 활용하여 작성되어 있습니다.
특정 NFT를 구매하길 원하는 입찰자(bidder)는 `createBid`를 호출하여 원하는 NFT 토큰과 가격을 제시합니다. 이 때 NFT의 가격은 지정된 ERC20 토큰으로 지불합니다.
생성된 입찰건은 다음과 같은 mapping 타입의 장부에 저장됩니다.

<font size="2">
{% highlight javascript %}
mapping(address => mapping(uint160 => mapping(uint256 => uint256))) bids;
{% endhighlight %}
</font>


조금 복잡한 구조를 지니고 있지만 입찰자의 계정을 key로 하고, 두 토큰 컨트랙트의 주소를 XOR 연산한 uint160의 값, 그리고 NFT토큰 ID와 제시한 가격을 저장합니다. 해당되는 NFT 토큰을 가진 사람은 bids에 제시된 가격을 보고 입찰에 응하게 되고 `acceptBid`를 호출하여 제시된 수량의 ERC20 토큰을 수령하고 NFT 소유권을 넘겨 주게 됩니다. 

ERC20 토큰을 전송할 때 위임 전송 transferFrom을 사용합니다. 이미 입찰자는 approve를 통하여 일정 수량을 전송하도록 위임했을 것입니다. 마찬가지로 NFT 토큰 역시 
transferFrom을 사용합니다. 그런데 이 두 함수는 모두 동일한 이름과 동일한 파라미터들을 가지고 있습니다.

<font size="2">
{% highlight javascript %}
transferFrom(address _from, address _to, uint256 _value)
transferFrom(address _from, address _to, uint256 _tokenId)
{% endhighlight %}
</font>

공격자 A는 이를 이용하여 입찰자 B가 소유한 NFT를 훔칠 수 있습니다. 입찰자가 생성한 입찰 건을 사용하여, 공격자는 실제로 입찰자가 구매를 원하는 NFT가 없다고 해도 
다음과 같이 `acceptBid`를 호출하는 것이 가능합니다.

<font size="2">
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

원래 ERC721 토큰 컨트랙트와 ERC20 토큰 컨트랙트 주소를 바꾸어서 호출하는 것입니다. 그리고 price에는 B가 소유한 NFT 토큰 아이디를 넣습니다. transferFrom은 동일하므로 오류없이 실행이 될 것입니다. 그러나 결과는 전혀 다릅니다. 컨트랙트 주소가 바뀌었기 때문에 다음 코드는 NFT 아이디 666번을 A에게 전송하게 됩니다.

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
Santiago Palladino는 오픈제펠린에서 잘 알려진 개발자입니다. 이것은 오더북 기반의 ERC20 토큰 판매를 간단하게 구현한 [컨트랙트][2nd]입니다. 오더북 기반의 토큰 판매는 온체인에서 처리되기 어렵기 때문에 오프체인에서 매수주문을 저장하고 조건이 맞으면(일부 또는 전체) 매도자가 서명을 하고 매수자가 그 데이터를 컨트랙트에 제출하는 방식으로 이루어집니다.

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
즉 분자(REFERRAL_FEE)를 100, 분모(REFERRAL_FEE_DENOMINATOR)를 10000으로 표현하여 0.0001까지 가능하도록 합니다. 

비율(rate)는 이더 당 판매되는 토큰의 수량입니다. 즉 지불한 이더에 대해 다음과 같은 수량의 토큰을 판매합니다. RATE_DENOMINATOR = 2**64로 분수로 비율을 표현합니다.

<font size="2">
{% highlight javascript %}
uint256 tokensPurchased = msg.value * order.rate / RATE_DENOMINATOR;
{% endhighlight %}
</font>
 

거래가 매칭되면 다음과 같은 토큰과 이더를 교환합니다. 여기서 `msg.sender`는 매도인이 서명한 주문을 제출하는 매수자가 됩니다.

<font size="2">
{% highlight javascript %}
IERC20(order.token).transferFrom(seller, msg.sender, tokensPurchased);
if (fee > 0) payable(order.referrer).transfer(fee);
payable(seller).transfer(msg.value - fee);
{% endhighlight %}
</font>


매수인은 주문정보를 Order 구조체의 각 항목들에 설정하여 오프체인에 저장합니다. 토큰 매도는 이 주문을 매도인이 전자서명함으로써 이루어집니다.
전자서명할 때는 주문정보를 컨트랙트의 `getOrderHash`라는 함수를 호출하여 해시한 후 서명합니다. 매수인은 이렇게 체결된 주문을 컨트랙트의 `executeOrder`에 
전달하면 비로소 주문이 처리됩니다. 이 함수에서는 주문정보와 전자서명이 일치함을 확인하고 토큰과 이더를 거래 당사자들에게 각각 전송하게 됩니다.

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

이것을 컨트랙트의 `getOrderHash`에 전달하면 됩니다. 후에 매도인이 이 해시에 서명을 하게 됩니다. 서명 대상이 되는 데이터는 컨트랙트 주소가 다시 포함되므로 
다음과 같은 형태의 데이터에 대한 해시에 대해 서명을 합니다.

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

해시 하기 전에 encodePacked 된 데이터는 다음과 같습니다. Order 구조체의 각 항목에 해당되는 값들로 나누어서 볼 수 있습니다. 이 형태를 잘 눈여겨 보도록 합시다. 
0x63ee...c849는 배포된(Görli) 컨트랙트의 [주소][exchange]입니다. 

<font size="1">
{% highlight shell %}
ad363...c9a1
5fbdb...0aa3
00000000000000000000000000000064
000000
63ee...c849
000000...0000000002386f26fc10000
00
{% endhighlight %}
</font>

이 값을 해시한 것을 전자서명합니다.

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





## 1위 Tynan Richards









[blog]: https://blog.soliditylang.org/2022/04/09/announcing-the-underhanded-contest-winners-2022/
[solmate]: https://github.com/transmissions11/solmate
[3rd]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission17_MichaelZhu
[2nd]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission10_SantiagoPalladino
[1st]: https://github.com/ethereum/solidity-underhanded-contest/tree/master/2022/submissions_2022/submission9_TynanRichards
[exchange]: https://goerli.etherscan.io/address/0x63ee5864f7fa0becfcee56093d654120e7e3c849

