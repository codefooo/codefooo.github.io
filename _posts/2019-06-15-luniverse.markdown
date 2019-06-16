---
layout: post
title: "Luniverse BaaS"
date: 2019-06-15
img: coins.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [luniverse, baas, react.js]
---

[루니버스(Luniverse)][luniverse]는 지난 5월 21일 정식으로 오픈한 BaaS(Blockchain as a Service) 기반의 블록체인 서비스 플랫폼들 중 하나입니다. 블록체인 기반의 애플리케이션을 
고려하는 기업이 별도의 서버나 네트워크 구축 없이도 클라우드 상에서 수월하게 애플리케이션을 만들 수 있는 환경을 제공합니다.   

다른 BaaS와의 차이점은 메인 체인(루니버스)을 제공하는 것이 아니라 루니버스를 이용하는 고객사들이 사이드 체인(side chain)을 생성하고 그 사이드 체인에 Dapp을 
배포하는 방식으로 운영된다는 것입니다. 사이드 체인은 대표적인 "layer 2" 기반의 블록체인 확장성 개선 방안 중 하나입니다. 

사이드 체인 상에서 모든 트랜잭션이 발생하고 (소수의 노드가 블록을 생성하며 인센티브가 없는 PoA 합의 방식) 일정 주기마다 메인 체인에 그 해쉬값을 기록하는 방식으로 블록체인 데이터의 무결성을 보장합니다.
루니버스는 허가형 블록체인처럼 노드를 운영하면서도 소위 말하는 "대중적인 서비스"(mass adaption)을 추구하는 하이브리드 형태의 BaaS라고 할 수 있겠습니다.  

### REST-API 기반의 인터페이스

이더리움 같은 경우는 Dapp이 직접 메인 넷의 스마트 컨트랙트와 인터페이스할 수 있지만 루니버스는 애플리케이션이 직접 스마트 컨트랙트와 데이터를 주고 받을 수 없습니다.
루니버스 콘솔(웹)에서 스마트 컨트랙트의 함수와 연결된 트랜잭션 API를 만들어주고(Transaction Action이라고 합니다) 애플리케이션은 API를 호출하는 형태로 Dapp을 개발합니다. 어떻게 보면 일반적인 API서버 구축과 
크게 달라보이지 않습니다.

처음 루니버스 회원 가입을 하면 30일간 무료로 사용가능한 개발 체인(Dev chain)을 제공합니다. 이 개발 체인은 이미 루니버스에서 테스트 용으로 
만들어놓은 사이드 체인입니다. 즉 다른 사용자들과 함께 공유하는 사이드 체인에 각자 자신만의 토큰을 만들어서 사용하는 형태입니다. 
정식 유료 사용자들은 사이드 체인부터 컨소시엄(Organization) 구성까지 새롭게 구축할 수 있습니다. 


![fig00]({{site.baseurl}}/assets/img/luniverse/side_chain.PNG)


사이드 체인은 항상 메인 체인에 디지털 자산인 "메인 토큰(MT)"이 생성되어 있고 그것을 기반으로 교환 비율이 정해지는 "사이드 토큰(ST)"을 발행해야 합니다.
즉 개발 체인 기준으로 보면 이미 <b>FT9754</b>라는 MT가 만들어져 있음을 알 수 있습니다. 회원 가입을 하면 자동으로 FT9754 라는 이름의 토큰 1000개를 지급받게 되고 
이를 기반으로 사이드 체인을 만들 수 있게 됩니다.  

위의 그림을 보면 개발 체인에는 149 명에게 각각 MT가 지급되었고 77개의 ST(즉 사이드 체인)가 존재함을 알 수 있습니다. 사이드 체인에서 블록을 생성하는 노드는 디폴트로 
3개가 제공됩니다. PoA에서는 3개의 노드가 차례로 블록을 생성합니다(PoA에서는 miner가 아니라 sealer라고 합니다). 

![fig01]({{site.baseurl}}/assets/img/luniverse/node.PNG)

루니버스에서 동작하는 Dapp은 이더리움의 개발 리소스를 활용할 수 있습니다. 스마트 컨트랙트 언어는 솔리디티(아직 ^0.5.0은 지원하지 않는 것 같습니다)를 사용합니다.
또 메타 마스크를 통해 직접 트랜잭션에 서명할 수 있어서 다양하게 Dapp을 구성할 수 있을 것 같습니다. 

### Dapp 개발환경

개발자를 위한 문서화가 비교적 잘 갖추어져 있습니다. Atom 에디터 플러그인 형태로 스마트 컨트랙트를 개발할 수 있는 패키지를 제공하고 또 직접 웹을 통해 스마트 컨트랙트를 
배포할 수 있습니다. 간단한 컨트랙트를 사용하여 루니버스의 기능을 알아보도록 하겠습니다. Dapp은 트러플의 react box를 사용하도록 하겠습니다.

우선 사이드 토큰을 생성합니다. 메인 화면의 하단에서 ERC-20 토큰을 생성하면 됩니다. 사이드 토큰은 Dapp의 "토큰 이코노미" 모델에 따라 다양하게 활용될 수 있는 
일종의 유틸리티 토큰의 역할을 할 수 있습니다. 

![fig02]({{site.baseurl}}/assets/img/luniverse/st.PNG)

예제로 사용할 컨트랙트는 SimpleStorage입니다.

{% highlight javascript %}
  
pragma solidity ^0.4.24;
    
contract SimpleStorage {
  
    uint storedData;  
    event Change(string message, uint newVal);
  
    function set(uint x) public {
       storedData = x;
       emit Change("set", x);
    }
  
    function get() public view returns (uint) {
       return storedData;
    }
}
  
{% endhighlight %}

사이드 토큰을 생성한 후 Dapp을 생성합니다. Dapp을 생성한 후에 컨트랙트를 배포하고 트랜잭션 API를 만들 수 있습니다.  

![fig03]({{site.baseurl}}/assets/img/luniverse/dapp.PNG)


스마트 컨트랙트는 파일을 업로드하여 배포할 수 있습니다. 여기서 사용하는 SimpleStorage.sol은 파일이 하나이므로 직접 업로드하면 됩니다. 배포는 
사이드 체인에서 합니다.

![fig04]({{site.baseurl}}/assets/img/luniverse/contract.PNG)

이렇게 배포가 끝나면 앞서 설정한 Dapp으로 들어가서 Transaction API를 구성할 차례입니다. Transaction Management 메뉴로 이동합니다.

![fig05]({{site.baseurl}}/assets/img/luniverse/txapi.PNG)

Transaction Management 에서 `+Create Transaction`을 클릭합니다. 컨트랙트 목록에는 사이드 토큰 컨트랙트와 SimpleStorage 컨트랙트가 
표시될 것입니다. 사용자 컨트랙트인 SimpleStorage를 선택하면 해당 컨트랙트에 구현된 외부 호출 가능한 Function들이 나옵니다. 여기서는 먼저 
set 함수를 선택하겠습니다.

![fig06]({{site.baseurl}}/assets/img/luniverse/txmng.PNG)


Action Name은 REST-API 호출시 사용되는 이름입니다. 매개변수는 화면에서 입력받을 것이므로 넣지 않습니다. 트랜잭션을 작성하고 나면 Dapp에서 
호출할 수 있도록 API가 생성되고 예제 스크립트를 볼 수 있습니다.

![fig07]({{site.baseurl}}/assets/img/luniverse/txspec.PNG)

그런데 외부에서 이러한 API를 호출하려면 키를 발급받아야 합니다. API Key 메뉴에서 Access key를 발급받습니다. 키는 Dapp 하나에 하나씩 발급됩니다.

![fig08]({{site.baseurl}}/assets/img/luniverse/api-key.PNG)

또 클라이언트 IP를 Whitelist에 추가합니다(프론트 애플리케이션에서 직접 API를 호출하는 경우에는 IP를 Whitelist에 등록해주고 API 키를 노출해야 된다는 것이 이상하긴 합니다).

컨트랙트 함수들 중 상태변수에 값을 저장하는 트랜잭션은 전자서명을 필요로 합니다. 전자서명이 필요한 경우에는 API에 `Required`라고 표시됩니다. 
루니버스에서는 이렇게 전자서명이 필요한 트랜잭션의 경우 두 가지 옵션을 제공합니다. 사용자가 직접 서명할 수 있도록 하거나 또는 루니버스에서 
private key를 보관하고 그것으로 서명을 위임할 수 있습니다. 

사용자가 직접 서명하는 경우에는 메타 마스크를 통해 직접 서명하기 때문에 번거로울 수 있습니다. 루니버스는 사용자의 불편함을 
덜어주기 위해 PKMS(Private Key Management Service)라는 기능을 제공합니다.

이러한 두 가지 방식을 직접 Dapp 예제를 통해서 살펴보겠습니다. 우선 트러플 react 박스를 설치합니다(트러플이 미리 설치되어 있어야 합니다). 

{% highlight shell %}
mkdir react-luniverse
cd react-luniverse
truffle unbox react

{% endhighlight %}

react 박스를 사용하면 메타 마스크와 같은 플러그인을 윈도우 객체로 사용할 수 있도록 해주는 함수가 제공되어 편리합니다. App.js의 componentDidMount를 
보면 web3를 컴포넌트 상태에 넣고 컴포넌트에서 참조하는 것을 알 수 있습니다.

먼저 사용자가 직접 서명하도록 하는 경우입니다.
컨트랙트에 있는 set이라는 함수를 API로 호출하려면 호출 URL은 `https://api.luniverse.io/tx/v1.0/transactions/set` 형태가 됩니다.
URL과 API 키를 다음과 같은 상수 값으로 정의하겠습니다.

{% highlight javascript %}

const LuniverseAPI = "https://api.luniverse.io/tx/v1.0";
const LuniverseTxAPI = `${LuniverseAPI}/transactions`;
const configAPI = {headers: {'Authorization': 'Bearer <API Key>'}};

{% endhighlight %}


set이라는 함수는 uint256 타입의 숫자를 매개변수로 받습니다. 데이터는 from과 inputs를 키로 하여 전달합니다. 중요한 것은 사용자가 직접 서명을 해야 한다는 것입니다. 
이를 위해서 set API를 호출하면 서명 되지 않은 트랜잭션 객체를 Dapp으로 리턴합니다. Dapp에서 이 트랜잭션 객체를 다시 `web3.eth.sendTransaction`을 통해서 전송하면 
서명이 되지 않았으므로 메타 마스크의 서명 요청 팝업이 나타나게 됩니다.

{% highlight javascript %}

setValueSign = async () => {

    const web3 = this.state.web3;
    const account = this.state.accounts[0];
    const data = {from: account, inputs: {x: this.state.val}};
    const response = await axios.post(`${LuniverseTxAPI}/set`, data, configAPI);

    if (account === response.data.data.rawTx.from) {
        const result = await web3.eth.sendTransaction(response.data.data.rawTx);
        console.log(web3.eth.abi.decodeParameters(['string', 'uint256'], result.logs[0].data));
    }

  }

{% endhighlight %}

루니버스의 블록 생성 시간은 1초이므로 사이드 체인에 기록된 리턴이 즉시 온다고 보면 되겠습니다. 리턴 되는 값은 이벤트로 정의한 logs 객체를 포함하고 있습니다.
`emit Change("set", x)`을 `web3.eth.abi.decodeParameters`으로 디코딩할 수 있습니다.

다음에는 PKMS 기능을 사용하여 직접 서명하는 과정없이, 마치 일반적인 API 호출처럼 사용할 수 있습니다. 이 경우에는 임의의 계정을 사용할 수는 없고 반드시 루니버스에서 
생성한 계정을 사용해야 합니다. REOA List(Remote EOA) 메뉴에서 계정을 미리 생성할 수 있고 Dapp에서 API 호출로 생성할 수도 있습니다.


![fig09]({{site.baseurl}}/assets/img/luniverse/reoa.PNG)


API를 통해서 사용자에게 계정을 발급하는 경우 사용자는 임의의 문자열을 key로 전달하여 키페어를 생성합니다. 사용자의 private key는 루니버스에 보관됩니다.
따라서 Dapp에서 사용자의 계정을 데이터베이스 등에 저장할 필요가 있습니다(사용자는 일반 사이트 가입 절차를 통해 본인 인증을 수행할 필요가 있습니다).

{% highlight javascript %}

createAccount = async () => {

    if (this.state.userKey !== '') {
        const data = {userKey: this.state.userKey, walletType: "LUNIVERSE"};
        const response = await axios.post(`${LuniverseAPI}/wallets`, data, configAPI);

        console.log(response.data.data.address);
    }
  }

{% endhighlight %}

트랜잭션 호출은 다음과 같습니다. from 계정이 이미 등록된 REOA라면 PKMS의 private key로 트랜잭션을 서명합니다. 리턴되는 값은 트랜잭션 ID입니다. 이더리움에서는 가스라는 트랜잭션 처리 비용이 들지만 루니버스 사이드 체인에서는 가스가 없습니다. 
{% highlight javascript %}

setValue = async () => {

    if (this.state.reoa !== '') {
        const account = this.state.reoa;
        const data = {from: account, inputs: {x: this.state.val}};
        const response = await axios.post(`${LuniverseTxAPI}/set`, data, configAPI);        
    }
  }

{% endhighlight %}

조회의 경우에는 전자서명이 필요 없으므로 일반적인 API 호출과 같습니다.

{% highlight javascript %}
getValue = async () => {

    const account = this.state.accounts[0];
    const data = {from: account};
    const result = await axios.post(`${LuniverseTxAPI}/get`, data, configAPI);
    this.setState({storageValue: result.data.data.res[0]})

}
{% endhighlight %}

### 사이드 토큰 잔액

사이드 토큰 잔액은 다음과 같은 API로 조회할 수 있습니다. 메인 토큰 이름(FT9754) 과 사이드 토큰(PTC) 이름으로 조회합니다.

<font size="2">
{% highlight javascript %}
getBalance = async () => {

    const web3 = this.state.web3;
    const account = this.state.accounts[0];

    const response = await axios.get(`${LuniverseAPI}/wallets/${account}/FT9754/PTC/balance`, configAPI);
    console.log(response.data.data.balance);
    this.setState({balance: web3.utils.fromWei(response.data.data.balance, 'ether')});

  }
{% endhighlight %}
</font>

[luniverse]: https://www.luniverse.io/
