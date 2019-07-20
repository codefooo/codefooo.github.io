---
layout: post
title: "How To Send Transactions for Ethereum"
date: 2019-07-15
img: road.jpg # Add image post (optional)
description: # Add post description (optional)
tag: [etherum, web3.js, truffle-contract, ethereumjs-tx]
---

자바스크립트 기반의 이더리움 Dapp 에서 컨트랙트의 메소드를 호출하는 방식을 정리합니다. <br/>
이더리움 클라이언트와 HTTP 또는 WebSocket 등으로 인터페이스할 때 사용되는 라이브러리에는 다음과 같은 것들이 있습니다.


* web3.js
* truffle-contract
* ethereumjs-tx


### web3.js

[web3.js][web3] 는 가장 기본적인 이더리움 자바스크립트 라이브러리 입니다. 특히 프론트엔드 애플리케이션에서는 web3.js가 거의 필수적으로 사용됩니다.
[트러플 리액트 박스][truffle-boxes]를 사용하는 경우는 조금 수월하게 web3.js를 사용할 수 있습니다. 리액트 박스에는 이미 다음과 같은 getWeb3.js 라고 하는 모듈이 포함되어 있습니다.


{% highlight javascript %}

import Web3 from "web3";

const getWeb3 = () =>
    new Promise((resolve, reject) => {
        // Wait for loading completion to avoid race conditions with web3 injection timing.
        window.addEventListener("load", async () => {
            // Modern dapp browsers...
            if (window.ethereum) {
                const web3 = new Web3(window.ethereum);
                try {
                    // Request account access if needed
                    await window.ethereum.enable();
                    // Acccounts now exposed
                    resolve(web3);
                } catch (error) {
                    reject(error);
                }
            }
            // Legacy dapp browsers...
            else if (window.web3) {
                // Use Mist/MetaMask's provider.
                const web3 = window.web3;
                console.log("Injected web3 detected.");
                resolve(web3);
            }
            // Fallback to localhost; use dev console port by default...
            else {
                const provider = new Web3.providers.HttpProvider(
                    "http://127.0.0.1:9545"
                );
                const web3 = new Web3(provider);
                console.log("No web3 instance injected, using Local web3.");
                resolve(web3);
            }
        });
    });

export default getWeb3;

{% endhighlight %}

이 경우에는 메타마스크와 같은 크롬 플러그인을 통해 이더리움과 인터페이스할 수 있습니다. 이런 방식을 "injected web3"라고 합니다. 프론트엔드에서는 
다음과 같이 componentDidMount 에서 web3.js를 참조하여 컨트랙트 인스턴스를 생성합니다. 여기서 MyContract는 트러플의 컴파일 결과물(artifact)입니다.

{% highlight javascript %}

...
import MyContract from "./contracts/MyContract.json";
import getWeb3 from "./utils/getWeb3";

async componentDidMount() {

    try {
        // Get network provider and web3 instance.
        const web3 = await getWeb3();        
        const accounts = await web3.eth.getAccounts();
        
        // Get the contract instance.
        const networkId = await web3.eth.net.getId();
        const deployedNetwork = MyContract.networks[networkId];
        const instance = new web3.eth.Contract(
            CoinToFlip.abi,
            deployedNetwork && deployedNetwork.address,
        );

        this.setState({web3, accounts, contract: instance});

    } catch (error) {
        // Catch any errors for any of the above operations.
        alert('Failed to load web3, accounts, or contract. Check console for details.');
        console.log(error);
    }
 }
 
{% endhighlight %}

상태에 저장하고 필요한 곳에서 컨트랙트 인스턴스를 전달받아서 컨트랙트의 메소드를 호출하면 됩니다. 메소드 호출은 대부분 비동기이므로 
async/await를 사용합니다. 프론트엔드에서 직접 이더리움에 배포된 컨트랙트와 인터페이스하는 경우에는 전자서명을 위해 메타마스크가 
동작하게 되는데, 여기서 `accounts[0]`는 메타마스크의 현재 선택된 계정입니다.

{% highlight javascript %}

async handleClick() {

   const {web3, accounts, contract} = this.state;     
   const result = await contract.methods.placeBet(this.state.checked).send({from:accounts[0]});   
     
}
{% endhighlight %}

단순 조회의 경우에는 전자서명이 필요하지 않습니다. 
 

{% highlight javascript %}

getBalance = async () => {

    const {web3, accounts, contract} = this.state;    
    const balance = await contract.methods.balanceOf(accounts[0]).call();
    this.setState({balance: web3.utils.fromWei(balance, "ether")});
    
    return balance;
}
{% endhighlight %}

직접 web3.js를 사용할 수도 있지만 truffle-contract라는 라이브러리를 사용할 수도 있습니다.

### truffle-contract

[truffle-contract][truffle-contract]는 트러플에서 컴파일된 컨트랙트를 자바스크립트 애플리케이션에서 쉽게 사용할 수 있도록 해주는 라이브러리입니다.
물론 앞의 예제처럼 truffle-contract없이 web3.js만을 사용할 수도 있습니다.

{% highlight javascript %}

...
import truffleContract from 'truffle-contract';
import MyContract from './contracts/MyContract.json';

componentDidMount = async () => {
    try {
        // Get network provider and web3 instance.
        const web3 = await getWeb3();

        // Use web3 to get the user's accounts.
        const accounts = await web3.eth.getAccounts();

        // Get the contract instance.
        const Contract = truffleContract(MyContract);
        Contract.setProvider(web3.currentProvider);
        const instance = await MyContract.deployed();

        this.setState({web3, accounts, contract: instance});

    } catch (error) {
        // Catch any errors for any of the above operations.
        alert('Failed to load web3, accounts, or contract. Check console for details.');
        console.log(error);
    }
};


async handleClick() {

    const {web3, accounts, contract} = this.state;         
    const result = await contract.placeBet(this.state.checked, {from:accounts[0]});

}


{% endhighlight %}



### 서명되지 않은 트랜잭션

경우에 따라서는 백엔드 서버의 API를 호출하여 컨트랙트의 메소드를 호출하기도 합니다. 하지만 사용자의 전자서명이 필요하므로 서버에서는 
서명되지 않은 트랜잭션을 만들어서 프론트엔드로 리턴하고 사용자의 서명을 요청할 수 있습니다. 예를 들어 백엔드에서는 다음과 같이 서명되지 않은 트랜잭션 객체를 리턴할 수 있습니다. 


{% highlight javascript %}

write = async (ctx) => {

    const {userAddr} = ctx.request.body;
    
    const contract = new web3.eth.Contract(abi, MyContractAddress, {from: userAddr});
    //The encoded ABI byte code to send via a transaction or call.
    const data = contract.methods.placeBet(param).encodeABI(); 
    
    let txObject = {};
    const txCount = await web3.eth.getTransactionCount(userAddr);    
    
    txObject["nonce"] = web3.utils.toHex(txCount);
    txObject["from"] = userAddr;
    txObject["to"] = MyContractAddress;
    txObject["data"] = data;
    txObject["gasLimit"] = web3.utils.toHex(3000000);
    txObject["gasPrice"] = web3.utils.toHex(web3.utils.toWei('20','gwei'));
    
    ctx.body = {success: true, rawTx: txObject};
}

{% endhighlight %}

트랜잭션 트랜잭션 객체를 생성할 때는 해당 계정의 순번(nonce)을 `web3.eth.getTransactionCount`으로 구하고 컨트랙트의 함수 호출과 전달 파라미터를 
ABI 인코딩하여 data로 전달합니다. 

이렇게 만들어진 트랜잭션 객체는 다시 프론트엔드로 리턴되고  프론트엔드에서는 전자서명을 추가하여(메타마스크가 동작) 이더리움으로 전송하게 됩니다.

{% highlight javascript %}

const result = await axios.post('/eth/write', {userAddr:accounts[0]});

if (result !== undefined && result.data !== undefined && result.data.rawTx !== undefined) {    
    const tx = await web3.eth.sendTransaction(result.data.rawTx);
}

{% endhighlight %}  


### ethereumjs-tx

전자서명을 클라이언트측의 사용자가 아닌 서버에서 해야 하는 경우도 있습니다. 이 때에는  [ethereumjs-tx][ethereumjs-tx]를 사용할 수 있습니다. 
자바스크립트 서버 애플리케이션은 보통 Node.js 기반에서 구현되기 때문에 Node.js에서 이더리움 트랜잭션을 다룰 때 사용할 수 있는 모듈로 이해하면 되겠습니다.

<font size="2">
{% highlight javascript %}

const ethTx = require('ethereumjs-tx');
const Web3 = require('web3');
const { GANACHE } = require('./eth.config');
const web3 = new Web3(GANACHE);
const abi = require('../../../_frontend/src/contracts/ERC20.json').abi;

write = async (ctx) => {

    const accounts = await web3.eth.getAccounts();
    const account = accounts[0];

    const privateKey = Buffer.from("995f43d7ac736ed...1b1e057dfcbe3be3f995ffe88b8fc", "hex");
    const MyContract = "0x609932B741237979187F7bf65A04631cb18eD166";

    const contract = new web3.eth.Contract(abi, MyContract);

    const {recipient, value} = ctx.request.body;

    const tokens = value * 10;
    const data = contract.methods.transfer(recipient, web3.utils.toWei(String(tokens), "ether")).encodeABI();


    try {
        const txCount = await web3.eth.getTransactionCount(account);

        const txObject = {
            nonce: web3.utils.toHex(txCount),
            from: account,
            to: MyContract,
            data: data,
            gasLimit:web3.utils.toHex(3000000),
            gasPrice:web3.utils.toHex(web3.utils.toWei('20','gwei')),
        }

        const tx = new ethTx(txObject);
        tx.sign(privateKey); // sign a transaction with a given private key(32 bytes)
        const serializedTx = tx.serialize();

        // web3.eth.sendSignedTransaction(signedTransactionData [, callback])
        // Signed transaction data in HEX format
        const result = await web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'));

        ctx.body = {success: true, txHash: result.transactionHash};

    } catch (err) {
        console.log(err);
        ctx.throw(500);
    }
}

{% endhighlight %}  
</font>

전자서명을 위한 private key는 하드코딩하기 보다는 접근제한된 별도의 파일로 분리하는 것이 바람직합니다.


{% highlight javascript %}
const fs = require("fs");
const key = fs.readFileSync("private.key").toString().trim();

const privateKey = Buffer.from(key, "hex");

...
{% endhighlight %}  

서버 애플리케이션에서도 truffle-contract를 사용할 수 있습니다.  전자서명을 위해서 truffle-hdwallet-provider를 추가합니다.

<font size="2">
{% highlight javascript %}

...
const HDWalletProvider = require("truffle-hdwallet-provider");
const TruffleContract = require("truffle-contract");
const contract = require('./contracts/MyContract.json'); //truffle artifacts

write = async (ctx) => {

    const accounts = await web3.eth.getAccounts();
    const account = accounts[0];
   
    const privateKey = Buffer.from("995f43d7ac736ed...1b1e057dfcbe3be3f995ffe88b8fc", "hex");    
    const MyContract = TruffleContract(contract);
 
    const {recipient, value} = ctx.request.body;

    const provider = new HDWalletProvider(privateKeys, "http://localhost:7545");
    MyContract.setProvider(provider);
    const contract = await MyContract.deployed();

    const tokens = value * 10;

    try {

        const tx = await contract.transfer(recipient, web3.utils.toWei(String(tokens), "ether"), {from: account});
        ctx.body = {success: true, tx};

    } catch (err) {
        console.log(err);
        ctx.throw(500);
    }
}
{% endhighlight %}  
</font>

[truffle-boxes]: https://www.trufflesuite.com/boxes/react
[web3]: https://web3js.readthedocs.io/en/1.0/index.html
[ethereumjs-tx]: https://www.npmjs.com/package/ethereumjs-tx
[truffle-contract]: https://www.npmjs.com/package/truffle-contract
[truffle-hdwallet]: https://www.npmjs.com/package/truffle-hdwallet-provider


