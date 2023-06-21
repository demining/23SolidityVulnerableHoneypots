
<figure class="aligncenter size-full"><img decoding="async" width="1280" height="720" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/043.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2301"></figure></div>


<p></p>



<p>Following the article:&nbsp;<em><a href="https://cryptodeeptech.ru/solidity-forcibly-send-ether-vulnerability/" target="_blank" rel="noreferrer noopener">“Solidity Forcibly Send Ether Vulnerability to a Smart Contract continuation of the list of general EcoSystem security from attacks”</a></em>.&nbsp;In this article, we will continue this topic related to vulnerabilities and traps. In the process of cryptanalysis of various cryptocurrencies, we are increasingly getting loopholes and backdoors. Honeypots work by luring attackers with a balance stored in the smart contract, and what appears to be a vulnerability in the code. Typically, to access the funds, the attacker would have to send their own funds, but unbeknownst to them, there is some kind of recovery mechanism allowing the smart contract owner to recover their own funds along with the funds of the attacker.</p>



<p>Let’s look at a couple different real world examples:</p>



<pre class="wp-block-code"><code>pragma solidity ^0.4.18;

contract MultiplicatorX3
{
    address public Owner = msg.sender;
   
    function() public payable{}
   
    function withdraw()
    payable
    public
    {
        require(msg.sender == Owner);
        Owner.transfer(this.balance);
    }
    
    function Command(address adr,bytes data)
    payable
    public
    {
        require(msg.sender == Owner);
        adr.call.value(msg.value)(data);
    }
    
    function multiplicate(address adr)
    public
    payable
    {
        if(msg.value&gt;=this.balance)
        {        
            adr.transfer(this.balance+msg.value);
        }
    }
}
</code></pre>



<p>In this&nbsp;<a href="https://etherscan.io/address/0x5aa88d2901c68fda244f1d0584400368d2c8e739#code">contract</a>, it seems that by sending more than the contract balance to&nbsp;<code>multiplicate()</code>, you can set your address as the contract owner, then proceed to drain the contract of funds. However, although it seems that&nbsp;<code>this.balance</code>&nbsp;is updated after the function is executed, it is actually updated before the function is called, meaning that&nbsp;<code>multiplicate()</code>&nbsp;is never executed, yet the attackers funds are locked in the contract.</p>



<pre class="wp-block-code"><code>pragma solidity ^0.4.19;

contract Gift_1_ETH
{
    bool passHasBeenSet = false;
    
    function()payable{}
    
    function GetHash(bytes pass) constant returns (bytes32) {return sha3(pass);}
    
    bytes32 public hashPass;
    
    function SetPass(bytes32 hash)
    public
    payable
    {
        if(!passHasBeenSet&amp;&amp;(msg.value &gt;= 1 ether))
        {
            hashPass = hash;
        }
    }
    
    function GetGift(bytes pass)
    external
    payable
    {
        if(hashPass == sha3(pass))
        {
            msg.sender.transfer(this.balance);
        }
    }
    
    function PassHasBeenSet(bytes32 hash)
    public
    {
        if(hash==hashPass)
        {
           passHasBeenSet=true;
        }
    }
}
</code></pre>



<p>This&nbsp;<a href="https://etherscan.io/address/0x75041597d8f6e869092d78b9814b7bcdeeb393b4#code">contract</a>&nbsp;is especially sneaky. So long as&nbsp;<code>passHasBeenSet</code>&nbsp;is still set to false, anyone could&nbsp;<code>GetHash()</code>,&nbsp;<code>SetPass()</code>, and&nbsp;<code>GetGift()</code>. The sneaky part of this contract, is that the last sentence is entirely true, but the problem is that&nbsp;<code>passHasBeenSet</code>&nbsp;is already set to true, even though it’s not in the etherscan&nbsp;<a href="https://etherscan.io/address/0x75041597d8f6e869092d78b9814b7bcdeeb393b4">transaction log</a>.</p>



<p>You see, when smart contracts make transactions to each other they don’t appear in the transaction log, this is because they perform what’s known as a message call and not a transaction. So what happened here, must have been some external contract setting the pass before anyone else could.</p>



<p>A safer method the attacker should have used would have been to check the contract storage with a security analysis tool.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="404" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-15-1024x404.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2267" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-15-1024x404.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-15-300x118.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-15-768x303.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-15-1536x606.png 1536w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-15.png 1777w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>

<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="679" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-16-1024x679.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2268" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-16-1024x679.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-16-300x199.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-16-768x509.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-16.png 1176w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>


<p>Hardly a week passes without large scale hacks in the crypto world. It’s not just centralised exchanges that are targets of attackers. Successful hacks such as the&nbsp;<a href="https://www.coindesk.com/understanding-dao-hack-journalists/" target="_blank" rel="noreferrer noopener">DAO</a>,&nbsp;<a href="https://steemit.com/cryptocurrency/@etheraveum/parity-wallet-hack-explained" target="_blank" rel="noreferrer noopener">Parity1</a>&nbsp;and&nbsp;<a href="https://blog.springrole.com/parity-multi-sig-wallets-funds-frozen-explained-768ac072763c" target="_blank" rel="noreferrer noopener">Parity2</a>&nbsp;have shown that vulnerabilities in smart contracts can lead to losing digital assets worth millions of dollars. Attackers are driven by making profits and with the incredible value appreciation in 2017 in the crypto world, individuals and organisations who hold or manage digital assets are often vulnerable to attacks. Especially smart contracts have become a prime target for attackers for the following reasons:</p>



<ul>
<li><strong>Finality of transactions:</strong>&nbsp;This is a special property of blockchain systems and it means that once a transaction (or state change) took place it can’t be taken back or at least not with grave consequences which in case of the DAO hack led to a hard fork. For an attacker targeting smart contracts, finality is a great property since a successful attack can not easily be undone. In traditional banking systems this is quite different, an attack even though initially successful could be stopped and any transactions could be rolled back if noticed early enough.</li>



<li><strong>Monetising successful attacks is straight forward</strong>: Once the funds of a smart contract can be withdrawn to an attacker’s account, transferring the funds to an exchange and cashing out in Fiat while concealing ones identity is something that the attackers can get away with if they are careful enough.</li>



<li><strong>Availability of contract source code / byte code:&nbsp;</strong>Ethereum is a public blockchain and so at least the byte code of a smart contract is available to anyone. Blockchain explorers such Etherscan allow also to attach source code to a smart contract and so giving access to high level Solidity code to potential attackers.</li>
</ul>



<p id="a9cf">Since we have established now why attackers find smart contracts attractive targets, let’s further look into the circumstances that could decide if a smart contracts gets attacked:</p>



<ol>
<li><strong>Balance</strong>: The greater the balance of a smart contract the more attackers will try to attack it and the more time they are willing to spend to find a vulnerability.<strong>&nbsp;</strong>This is an easier economic equation than for none smart contract targets since the balance that can be potentially stolen is public and attackers have certainty on how profitable a successful attack could be.</li>



<li><strong>Difficulty/Time</strong>: This is the unknown variable in the equation. Yet the approach to look for potential targets can be automated by using smart contract vulnerability scanners. Availability of source code addtionally decreases analyis time while also lowering the bar for potential attackers to hack smart contracts since byte code is harder to read and therefore it takes more skill and time to analyse.</li>
</ol>



<p id="edf6">Taking the two factors above in consideration, one could assume that every smart contract published to the main net with a sufficient balance is analysed automatically by scanners or/and manually by humans for vulnerabilities and is likely going to be exploited if it is in fact vulnerable. The economic incentives and the availability of smart contracts on the public chain have given rise to a very active group attackers, trying to steal from vulnerable smart contracts. Among this larger group of attackers, a few seem to have specialised to hack the hackers by creating seemingly vulnerable smart contracts. In many ways these contracts have resemblance to honeypot systems. They are created to lure attackers with the following properties:</p>



<ul>
<li>Balance: Honeypots are created with an initial balance that often seem to be in the range of 0.5–1.0 ETH.</li>



<li>Vulnerability: A weakness in the code that seemingly allows an attacker to withdraw all the funds.</li>



<li>Recovery Mechanism: Allows owner to reclaim the funds including the funds of the attacker.</li>
</ul>



<p id="f13c">Let’s analyse three different types of smart contract honeypots that I have come across over the last couple of weeks.</p>



<h1 class="wp-block-heading" id="c951">honeypot1: Multiplicator.sol</h1>



<p id="3ef1">The contract’s source code was published on&nbsp;<a href="https://etherscan.io/address/0x5aa88d2901c68fda244f1d0584400368d2c8e739#code" target="_blank" rel="noreferrer noopener">Etherscan</a>&nbsp;with a seemingly vulnerable function. Try to spot the trap.</p>


<div class="wp-block-image">
<figure class="aligncenter size-full"><a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/Multiplicator.sol" target="_blank" rel="noreferrer noopener"><img decoding="async" loading="lazy" width="889" height="799" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-17.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2269"></a></figure></div>


<p><strong><a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/Multiplicator.sol" target="_blank" rel="noreferrer noopener">GITHUB</a></strong></p>



<p id="4e77">This is a really a short contract and the&nbsp;<em>multiplicate()</em>&nbsp;function is the only function that does allow a call from anyone else than the owner of the contract. At first glance it looks like by transferring more than the current balance of the contract it is possible to withdraw the full balance. Both statements in line 29 and 31 try to reinforce the idea that&nbsp;<em>this.balance&nbsp;</em>is somehow credited after the function is finished. This is a trap since the&nbsp;<em>this.balance</em>&nbsp;is updated&nbsp;<strong>before</strong>&nbsp;the&nbsp;<em>multiplicate() function&nbsp;</em>is called and so&nbsp;<em>if(msg.value&gt;=this.balance)</em>&nbsp;is never true unless&nbsp;<em>this.balance</em>&nbsp;is initially zero.</p>



<p id="9ae3">It seems that someone has actually tried to call&nbsp;<em>multiplicate()</em>&nbsp;with 1.1 Ether. Shortly after the owner has&nbsp;<a href="https://etherscan.io/tx/0xbf4930b18953d0df8a24857557d480468ad6342d7e9b32ab2c360674fc1696fd" target="_blank" rel="noreferrer noopener">withdrawn</a>&nbsp;the full balance.</p>



<figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="544" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-18-1024x544.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2270" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-18-1024x544.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-18-300x159.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-18-768x408.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-18.png 1216w" sizes="(max-width: 1024px) 100vw, 1024px"></figure>



<h1 class="wp-block-heading" id="6e60">honeypot2: Gift_1_ETH.sol</h1>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/Gift_1_ETH.sol" target="_blank" rel="noreferrer noopener"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-19.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2271" width="892" height="959" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-19.png 754w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-19-279x300.png 279w" sizes="(max-width: 892px) 100vw, 892px"></a></figure></div>


<p><strong><a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/Gift_1_ETH.sol" target="_blank" rel="noreferrer noopener">GITHUB</a></strong></p>



<p id="f0d9">The contract has a promising name, if you want to figure out the trap yourself have a look at the code&nbsp;<a href="https://etherscan.io/address/0x75041597d8f6e869092d78b9814b7bcdeeb393b4#code" target="_blank" rel="noreferrer noopener">here</a>. Also check out the&nbsp;<a href="https://etherscan.io/address/0x75041597d8f6e869092d78b9814b7bcdeeb393b4" target="_blank" rel="noreferrer noopener">transaction log</a>&nbsp;… why did 0xc4126a64c546677146FfB3f3D5A6F6d5A2F94DF1 lose 1 ETH?</p>



<figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="558" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-20-1024x558.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2272" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-20-1024x558.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-20-300x164.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-20-768x419.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-20.png 1218w" sizes="(max-width: 1024px) 100vw, 1024px"></figure>



<figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="542" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-21-1024x542.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2273" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-21-1024x542.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-21-300x159.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-21-768x407.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-21.png 1205w" sizes="(max-width: 1024px) 100vw, 1024px"></figure>



<figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="596" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-22-1024x596.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2274" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-22-1024x596.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-22-300x175.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-22-768x447.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-22.png 1203w" sizes="(max-width: 1024px) 100vw, 1024px"></figure>



<p id="8607">It seems that 0xc4126a64c546677146FfB3f3D5A6F6d5A2F94DF1 did everything right. First&nbsp;<em>SetPass()</em>&nbsp;was called to overwrite&nbsp;<em>hashPass</em>&nbsp;and then&nbsp;<em>GetGift()</em>&nbsp;to withdraw the Ether. Also the attacker made sure&nbsp;<em>PassHasBeenSet()</em>&nbsp;has not been called. So what went wrong?</p>



<p id="0377">One important piece of information in order to understand&nbsp;<em>honeypot2</em>&nbsp;is to clarify what internal transactions are. They actually do not exist according to the specifications in the&nbsp;<a href="https://cryptodeep.ru/doc/paper.pdf" target="_blank" rel="noreferrer noopener">Ethereum Yellow Paper</a>&nbsp;(see Appendix A for terminologies). Transactions can only be sent by External Actors to other External Actors or non-empty associated EVM Code accounts or what is commonly referred to as smart contracts. If smart contracts exchange value between each other then they perform a Message Call not a Transaction. The terminology used by EtherScan and other blockchain explorers can be misleading.</p>


<div class="wp-block-image">
<figure class="aligncenter size-full"><a href="https://cryptodeep.ru/doc/paper.pdf" target="_blank" rel="noreferrer noopener"><img decoding="async" loading="lazy" width="1109" height="406" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-23.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2275"></a></figure></div>


<p id="70f8">It’s interesting how one takes information as a given truth if the data comes from a familiar source. In this case EtherScan does not show the full picture of what happened. The assumption is that the transaction (or message call) should show up in internal transactions tab but it seems that calls from other contracts that have&nbsp;<em>msg.value</em>&nbsp;set to zero are not listed currently. Etherchain on the other hand shows the&nbsp;<a href="https://www.etherchain.org/tx/edde32b2166647e0210bd137c18b8b8833777796f11113cc7257934d5a708823" target="_blank" rel="noreferrer noopener">transaction</a>&nbsp;(or message call) that called&nbsp;<em>PassHasBeenSet()&nbsp;</em>with the correct hash and so denying any future password reset. The attacker (in this case more of a victim) could have also been more careful and actually read the contract storage with&nbsp;<a href="https://github.com/ConsenSys/mythril" target="_blank" rel="noreferrer noopener">Mythril</a>&nbsp;for instance. It would have been apparent that p<em>assHasBeenSet&nbsp;</em>is already set to true.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="279" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-29-1024x279.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2285" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-29-1024x279.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-29-300x82.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-29-768x209.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-29.png 1258w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>

<div class="wp-block-image">
<figure class="aligncenter size-full"><img decoding="async" loading="lazy" width="1190" height="422" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-24.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2276"></figure></div>

<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="433" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-25-1024x433.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2277" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-25-1024x433.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-25-300x127.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-25-768x325.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-25.png 1477w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>

<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="494" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-26-1024x494.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2278" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-26-1024x494.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-26-300x145.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-26-768x371.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-26.png 1453w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>


<h1 class="wp-block-heading" id="b3aa">honeypot3: TestToken</h1>



<p id="885c">I have taken the trick from the honeypot contract&nbsp;<a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/WhaleGiveaway1.sol" target="_blank" rel="noreferrer noopener">WhaleGiveaway1</a>&nbsp;(see analysis) and combined it with one of my own ideas. The contract is available&nbsp;<a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/TestToken.sol" target="_blank" rel="noreferrer noopener">here</a>&nbsp;on my Github. Something is missing here …</p>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/TestToken.sol" target="_blank" rel="noreferrer noopener"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-27.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2279" width="860" height="599"></a></figure></div>


<p id="c689">This contract relies on a very simple yet effective technique. It uses a lot of whitespaces to push some of the code to the right and out of the immediate visibility of the editor if horizontal scrolling is enabled (<a href="https://github.com/demining/23SolidityVulnerableHoneypots/blob/main/WhaleGiveaway1.sol" target="_blank" rel="noreferrer noopener">WhaleGiveaway1</a>). When you try this locally in Remix and you purely rely on the scrolling technique like in&nbsp;<em>WhaleGiveaway1</em>&nbsp;then the trick actually does not work. It would be effective if an attacker copies the code and is actually able to exploit the issue locally but then fails on the main net. This can be done using block numbers. Based on what network is used the block numbers vary significantly from the main net.</p>



<p id="d44b"><strong>Ganache:</strong>&nbsp;starts from 0</p>



<p id="3fbe"><strong>Testrpc</strong>: starts from 1150000</p>



<p id="9f75"><strong>Ropsten</strong>: a few weeks ago around 2596174</p>



<p id="4b12"><strong>Main net</strong>: a few weeks ago around 5040270</p>



<p id="0ffe">Therefore the first if statement is only true on the main net and transfers all ETH to the owner. On the other networks the “invisible” code is not executed.</p>



<p id="7157"><em>if (block.number &gt; 5040270 ) {if (_owner == msg.sender ){_owner.transfer(this.balance);} else {throw;}}</em></p>



<p id="5f1d">EtherScan also had the horizontal scrolling enabled, but they deactivated it a few a few weeks ago.</p>



<h2 class="wp-block-heading" id="46ae">TL;DR</h2>



<p id="03cc">Smart contract honeypot authors form a very interesting sub culture among a larger group of hackers trying to profit from vulnerable smart contracts. In general I would like to give anyone the following advice:</p>



<ul>
<li>Be careful where you send your ETH, it could be a trap.</li>



<li>Be nice and don’t steal from people.</li>
</ul>



<p id="9a31">I have created a Github repo for honeypot smart contracts&nbsp;<a href="https://github.com/demining/23SolidityVulnerableHoneypots" target="_blank" rel="noreferrer noopener">here</a>. Should you have any honey pot contracts yourself that you want to share please feel free to push them to the repo or share them in the comments.</p>


<div class="wp-block-image">
<figure class="aligncenter size-full"><img decoding="async" loading="lazy" width="818" height="729" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-28.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2282"></figure></div>


<hr class="wp-block-separator has-alpha-channel-opacity">


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="554" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-70-1024x554.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-1689" srcset="https://cryptodeeptech.ru/wp-content/uploads/2022/12/image-70-1024x554.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2022/12/image-70-300x162.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2022/12/image-70-768x415.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2022/12/image-70.png 1257w" sizes="(max-width: 1024px) 100vw, 1024px"><figcaption class="wp-element-caption"><a href="https://cryptodeep.ru/doc/The_Art_of_The_Scam_Demystifying_Honeypots_in_Ethereum_Smart_Contracts.pdf" target="_blank" rel="noreferrer noopener"><code>https://cryptodeep.ru/doc/The_Art_of_The_Scam_Demystifying_Honeypots_in_Ethereum_Smart_Contracts.pdf</code></a></figcaption></figure></div>


<p>Honeypot programs are one of the best tools that security researchers have ever made to study the new or unknown hacking techniques used by attackers. Therefore, using honeypots in smart contract could be a very good idea to study those attacks.&nbsp;<strong>So what is honeypot in smart contract?</strong></p>



<p><strong>Honeypots in the Blockchain industry is an intentionally vulnerable smart contract that was made to push attackers to exploit its vulnerability. The idea is to convince attackers or even simple users to send a small portion of cryptocurrency to the contract to exploit it, then lock those ethers in the contract.&nbsp;</strong></p>



<p>In this blog post, you are going to see some examples of those honeypots with a detailed technical explanation of how they work. So if you are interested to learn more about this subject just keep reading and leave a comment at the end.</p>



<h2 class="wp-block-heading">What is honeypot in smart contract?</h2>



<p><strong>A honeypot is a smart contract that purports to leak cash to an arbitrary user due to a clear vulnerability in its code in exchange for extra payments from that user. The monies donated by the user to the vulnerable contract get then locked in the contract and only the honeypot designer or attacker will be able to recover them</strong>.</p>



<p>The concept of a honeypot is well known in the field of network security and was used for years by security research. The main objective of using them was to identify new or unknown exploits or techniques already used in the wild. In addition, Honeypots were used to identify zero-day vulnerabilities and report them to vendors. This technique was basically designed to trap black hat hackers and learn from them.</p>



<p>However, with the rise of&nbsp;Blockchain technology&nbsp;and the smart contract concept. </p>



<p>Blockchain is the new trending technology in the market, many companies start to implement it to solve multiple problems. Usually, this technology manages the different types of user information related to their money. Therefore, to secure this technology you should first understand how it works. Blockchain technology can be seen as a 6 layer system that works together. Therefore,&nbsp;<strong>what are the six layers of blockchain technology?</strong></p>



<p><strong>The Blockchain technology is built upon 6 main layers that are:</strong></p>



<ol type="1">
<li><strong>The TCP/IP network</strong></li>



<li><strong>Peer-to-Peer protocols</strong></li>



<li><strong>Consensus algorithms</strong></li>



<li><strong>Cryptography algorithms</strong></li>



<li><strong>Execution (Data blocs, Transactions, …)</strong></li>



<li><strong>Applications (Dapps, smart contracts …)</strong></li>
</ol>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-30.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2286" width="847" height="819" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-30.png 633w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-30-300x290.png 300w" sizes="(max-width: 847px) 100vw, 847px"></figure></div>


<p>Black hat hackers started to use this concept to trap users both with good or bad intentions. </p>



<hr class="wp-block-separator has-alpha-channel-opacity">



<p>The idea is simple, the honeypot designer <a href="https://cryptodeeptech.ru/top-10-solidity-smart-contract-audit-tools" target="_blank" rel="noreferrer noopener">creates a&nbsp;smart contract</a>&nbsp;and puts a clear vulnerability in it. Then hid a malicious code in its smart contract or between its transactions to block the right execution of the withdraw function. Then he deploys the contract and waits for other users to get into the trap.</p>



<p><strong><a href="https://cryptodeeptech.ru/top-10-solidity-smart-contract-audit-tools" target="_blank" rel="noreferrer noopener">Best 10 solidity smart contract audit tools that both developers and auditors use during their audit?</a></strong></p>



<ol type="1">
<li><strong>Slither</strong></li>



<li><strong>Securify</strong></li>



<li><strong>SmartCheck</strong></li>



<li><strong>Oyente</strong></li>



<li><strong>Mythril</strong></li>



<li><strong>ContractFuzzer</strong></li>



<li><strong>Remix IDE static analysis plug-in</strong></li>



<li><strong>Manticore</strong></li>



<li><strong>sFuzz</strong></li>



<li><strong>MadMax</strong></li>
</ol>



<hr class="wp-block-separator has-alpha-channel-opacity">



<blockquote class="wp-block-quote">
<p></p>
<cite>Honestly, the honeypots concept in blockchain is just exploiting the greedy of users that cannot see the whole picture of the smart contract and does not dig deeper to understand the smart contract mechanism and code.</cite></blockquote>



<hr class="wp-block-separator has-alpha-channel-opacity">



<p>What actually makes this concept even more dangerous in the context of blockchain is that implementing a honeypot is not really difficult and does not require advanced skills. In fact, any user can implement a honeypot in the blockchain, all it needs is the actual fees to deploy such a contract in the blockchain.</p>



<p>In fact, in the blockchain, the word “attacker” could be given to both the one who deploys the smart contract honeypot and the one trying to exploit it (depending on his intention). Therefore, in the following sections of this blog post, we will use the word “deployer” to the one who implements the honeypot and “user” to the one trying to exploit that smart contract.</p>



<h2 class="wp-block-heading">What are the types of smart contract honeypots?</h2>



<p><strong>Honeypots in smart contract can be divided into 3 main categories depending on the used techniques:</strong></p>



<ul>
<li><strong>EVM based smart contract honeypots</strong></li>



<li><strong>Solidity compiler-based smart contract honeypots</strong></li>



<li><strong>Etherscan based smart contract honeypots</strong></li>
</ul>



<p>The main idea of honeypot in the network context is to supervise an intentionally&nbsp;<a href="https://cryptodeeptech.ru/using-components-with-known-vulnerabilities-prevention/" target="_blank" rel="noreferrer noopener">vulnerable component</a>&nbsp;to see how it can be exploited by hackers. However, in smart contract the main idea is to hide a behavior from users and trick them to send ether to gain more due to the vulnerability exploitation.</p>



<p><strong>six things you should do to prevent using components with known vulnerabilities:</strong></p>



<ul>
<li><strong>Use components from official repositories</strong></li>



<li><strong>Remove unused components</strong></li>



<li><strong>Only accept components with active support</strong></li>



<li><strong>Put a vulnerability management system for you components</strong></li>



<li><strong>Put in place a components firewall</strong></li>



<li><strong>Remove or replace components with a stopped support</strong></li>
</ul>



<hr class="wp-block-separator has-alpha-channel-opacity">



<p>Therefore, what actually defines each smart contract honeypot category is the used technique to hide that information from users.</p>



<p>The first category of smart contract honeypot is based on the way the EVM instruction is executed. It is true that the EVM follow an exact set of rules, however, some instruction requires a very good experience with the way EVM works to be able to detect the honeypot otherwise the user could easily be fooled.</p>



<p>The second category of smart contract honeypot is related to the solidity compiler. In other words, the smart contract honeypot builder should have a good experience with smart contract development and a deep understanding of how Solidity compiler would work. For example, the way inherence is managed by each version of the solidity compiler, or when overwriting variables or parameters would happen.</p>



<p>The third category of smart contract honeypot is based on hiding things from the users. Most users that try to exploit a program look for the easier way to do so (quick wins). Therefore, they may not take the time to analyze all parts of the vulnerable smart contract. This user behavior leads to locking his money in the smart contract. In this blog post, we are going to discuss 4 techniques used by deployers to hide an internal behavior from the users and therefore fool the user.</p>



<hr class="wp-block-separator has-alpha-channel-opacity">



<blockquote class="wp-block-quote">
<p></p>
<cite>In my opinion the second category of honeypots is the most difficult to detect as it require a deep knowledge of the solidity compiler.</cite></blockquote>



<hr class="wp-block-separator has-alpha-channel-opacity">



<h2 class="wp-block-heading">EVM based smart contract honeypots</h2>



<p>The EVM-based smart contract honeypots have only one subtype called balance disorder. I think the best way to understand how this type of smart contract honeypots works, is by example. So take a look at the following example:</p>



<figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="198" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-31-1024x198.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2287" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-31-1024x198.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-31-300x58.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-31-768x148.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-31-1536x297.png 1536w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-31.png 1645w" sizes="(max-width: 1024px) 100vw, 1024px"></figure>



<p>This example is taken from the following contract:&nbsp;<a href="https://etherscan.io/address/0x8b3e6e910dfd6b406f9f15962b3656e799f60d2b#code">https://etherscan.io/address/0x8b3e6e910dfd6b406f9f15962b3656e799f60d2b#code</a></p>



<p>A quick look at this function from a user, he can easily understand that if he sends while calling this function more than what the contract balance actually has, then everything in the contract plus what he sends will be sent back to him. Which is obviously a good deal.</p>



<p>However, what a user could miss in this quick analysis of the smart contract is that the contract balance will be incremented as soon as the function of the call is performed by the user. This means that the msg.value will always be lower than the contract balance no matter what you do. Therefore, the condition will never be true and the contract will be locked in this contract.</p>



<p>Another example of the balance disorder type of honeypot could be found here:</p>



<p><a href="https://etherscan.io/address/0xf2cf114be39a48aa2321ed39c1f132da0c51e453">https://etherscan.io/address/0xf2cf114be39a48aa2321ed39c1f132da0c51e453</a></p>



<p>By visiting this link you can see that there is no source code out there. So there are two ways to analyze this contract. The first one and the most difficult is to get the bytecode of this smart contract and then try to understand and reverse engineer it. Or the second way is to try to decompile it using different tools available to get an intermediate and easy-to-understand source code.</p>



<p>I personally used the second technique to accelerate the analysis and simply used the Etherscan default decompile. In the smart contract you want to decompile you can click here:</p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="434" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-32-1024x434.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2288" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-32-1024x434.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-32-300x127.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-32-768x325.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-32.png 1421w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>


<p>And wait for a moment about 30 seconds to get the source code.</p>



<p>By taking a look at the source code, and especially at the “multiplicate” function you can now easily see the same logic as the previously explained example.</p>


<div class="wp-block-image">
<figure class="aligncenter size-large is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-33-1024x235.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2289" width="856" height="196" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-33-1024x235.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-33-300x69.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-33-768x176.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-33.png 1193w" sizes="(max-width: 856px) 100vw, 856px"></figure></div>


<p>The condition in line 24 will never be verified and the money will be stuck in the contract.</p>



<h2 class="wp-block-heading">Solidity compiler-based smart contract honeypots</h2>



<p>As I said, this category of smart contract honeypots is based on some deep knowledge about how the Solidity compiler works. In the following subsection, I will give you 4 techniques that are used to build this kind of smart contract honeypots. However, other unknown techniques might be used in the wild, and I will do my best to update this blog post whenever I found a new one. Please comment below and tell me if you know a technique that was not noted in this blog post.</p>



<h3 class="wp-block-heading">Inheritance Disorder technique</h3>



<p>One of the most confusing systems in solidity language or even in other programming languages is inheritance. A lot of hidden aspects in this concept could be used by deployer to fool the users and work contrary to what is expected.</p>



<p>In solidity language, a smart contract can implement the inheritance concept by using the word “is” followed by the different smart contract that this one wants to inherit their source code. Then only one smart contract is created and the source code from the other contracts is copied into it.</p>



<p>To better understand how such a mechanism could be exploited to create honeypots please take a look at the following examples:</p>



<p><strong>Example1:</strong></p>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-34.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2290" width="841" height="1107" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-34.png 622w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-34-228x300.png 228w" sizes="(max-width: 841px) 100vw, 841px"></figure></div>


<p>You can find this contract here:&nbsp;<a href="https://etherscan.io/address/0xd3bd3c8fb11429b0deee5301e72b66fba29782c0#code">https://etherscan.io/address/0xd3bd3c8fb11429b0deee5301e72b66fba29782c0#code</a></p>



<p>If you take a look at this contract source code, you can easily notice that it has an obvious vulnerability related to access control. The function setup allows a user to change the owner of this contract without checking if he is the actual owner. Therefore, the user would be able to execute the withdraw function to get the money.</p>



<p>However, this analysis assumes that the isOwner() function inherited from the Ownable contract is going to check the local variable Owner.</p>



<p>Unfortunately, this is not what will actually happen. The inheritance creates a different variable for each contract even if they have the same name. The variable Ownable.Owner is totally different than the ICO.Owner. Therefore, when the user will call the setup() function, this one will change the value of ICO.Owner and not Ownable.Owner. This means that the result of the isOwner() will remain the same.</p>



<p><strong>Example2</strong></p>



<p>Another example of this same type of solidity compiler-based honeypot can be found here. The same logic applies to this smart contract. The Owner variable will not change by calling the setup() function.</p>



<h3 class="wp-block-heading">Skip Empty String Literal</h3>



<p>Another tricky behavior in solidity compiler that may not be very easy to discover is the skip empty string literal.&nbsp;<strong>The skip empty string literal problem happens in solidity when a function is called with an empty string as a parameter.</strong>&nbsp;This is a known bug in solidity&nbsp;<strong>compilers before 0.4.13</strong>&nbsp;here is a reference for it.</p>



<p>The encoder skips the empty string literal “” when used as a parameter in a function call. As a result, the encoding of all subsequent arguments is moved left by 32 bytes, causing the function call data to be malformed.</p>



<p>This kind of honeypot could be easily detected, by just looking at the solidity compiler version and then scrolling down the source code to see if there is any use of the empty string in a function call. However, a knowledge of this bug is required to detect the problem in the smart contract.</p>



<p>Here is a simple example of this honeypot:</p>



<p>Check the following smart contract:&nbsp;<a href="https://etherscan.io/address/0x2b990227344300aded3a072b3bfb9878b209da0a#code">https://etherscan.io/address/0x2b990227344300aded3a072b3bfb9878b209da0a#code</a></p>



<p>The source code is a little bit long so I will put just the most important functions:</p>



<figure class="wp-block-image size-large is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-35-1024x284.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2291" width="846" height="234" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-35-1024x284.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-35-300x83.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-35-768x213.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-35.png 1307w" sizes="(max-width: 846px) 100vw, 846px"></figure>



<p>In the divest() function line 83, the external function call to loggedTransfer() with the empty string will result in shifting the parameters by 32 bytes which leads to replacing the target address from msg.sender to the owner address. Therefore, the user will send the money to the owner of the contract and not his own address. This simply means that the user will never be able to retrieve the money he sent to this smart contract.</p>



<blockquote class="wp-block-quote">
<p></p>
<cite>This behavior happens&nbsp;<strong>only</strong>&nbsp;in case of calling the function externally with the this.function().</cite></blockquote>



<hr class="wp-block-separator has-alpha-channel-opacity">



<h3 class="wp-block-heading">Type Deduction Overflow</h3>



<p><strong>The Solidity compiler offers a nice feature that helps developers declare a variable without knowing exactly what type it would be. This could be made by creating a variable with the keyword “var” and the compiler will deduce what type is better for that result.</strong>&nbsp;However, this technique may cause a problem called type deduction overflow.</p>



<p>This problem could be used in a smart contract honeypot to cause a revert and then lock the money on the contract. To better illustrate this problem please take a look at the following source code:</p>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-36.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2292" width="946" height="581"></figure></div>


<p>You can check the whole code here:</p>



<p><a href="https://etherscan.io/address/0x48493465a6a2d8db8616a3c7288a9f81d54a8835#code">https://etherscan.io/address/0x48493465a6a2d8db8616a3c7288a9f81d54a8835#code</a></p>



<p>In this contract the Double() function allow a user to double his money by first sending at least more than one ether and then looping to create the value of the ethers that will be sent to the user. This seems to be a nice and easy smart contract to exploit.</p>



<p>However, this contract loop will never reach even half of the value sent by the user. The reason behind this is the way the variable “i” is declared. The “var” keyword, will create a variable with a type of uint8 due to the 0 value affected to it in the beginning. The code should loop till it gets to msg.value which is a uint256 and the value would be more than 1 with 18 digits. However, the size of the “i” variable can only reach 255 then once incremented will get back to 0. Therefore, the loop will end and all that the user will receive is 255 wei.</p>



<h3 class="wp-block-heading">Uninitialized Struct</h3>



<p>The uninitialized structure is a common problem in solidity and could be seen both as a vulnerability and as a way to trick users. In this blog post, I am going to discuss the tricky part of this problem. However, if you want me to discuss how this could be a vulnerability, please comment below and I will be happy to make a blog post about it.</p>



<p><strong>An uninitialized structure problem happens when a structure variable is not initialized at the moment of its creation. When a structure variable is not initialized in the same line as its creation with the keyword “new”, the solidity compiler point that variable to the first slot of the smart contract. This simply means the variable will be pointing to the first variable of the smart contract. Once the developer starts affecting values to the structure variable, the first element value of the structure will overwrite the first variable value.</strong></p>



<p>This concept is used by smart contract honeypots deployer to trick users to send money to exploit an obvious vulnerability in it.</p>



<p>Here is an example of such a honeypot:</p>



<p><a href="https://etherscan.io/address/0x29ed301f073f62acc13a2d3df64db4a3185f1433#code">https://etherscan.io/address/0x29ed301f073f62acc13a2d3df64db4a3185f1433#code</a></p>


<div class="wp-block-image">
<figure class="aligncenter size-full is-resized"><img decoding="async" loading="lazy" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-37.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2293" width="908" height="280"></figure></div>


<p>This contract asks the user to guess a number while betting with some of his money. The secret value that a user is going to guess is stored in the first slot of the smart contract. For a quick analysis of this contract, the user would assume that the contract is vulnerable as even private variables could be seen in the Blockchain.</p>



<p>However, once the user will call the play() function and send money to it, the function will create a structure “game” in line 51 without correctly initializing it. This means that this structure variable will point to the first slot (variable secretNumber). In addition, the game.player will be the variable that will overwrite the secretNumber variable. Therefore, the user “would not” will not be able to correctly guess the number.</p>



<p><em>Actually, in this example, the honeypot could be bypassed to retrieve the money. If you take a look at the value affected to the game.player variable that overwrite the secretNumber. You will see that it is simply the sender’s address. Therefore, the value the user should send, is simply his address converted to decimals.</em></p>



<blockquote class="wp-block-quote">
<hr class="wp-block-separator has-alpha-channel-opacity">
<cite>Most techniques used in this category of smart contract honeypots can easily be detected by users if they first try to compile and test their exploit in a local environment or a test chain. However, most of the time the vulnerabilities in those smart contracts are so easy to spot and exploit. Therefore, with a small portion of greediness and self-confidence, the users do not even think twice and directly execute their exploit on the mainnet.</cite></blockquote>



<hr class="wp-block-separator has-alpha-channel-opacity">



<h2 class="wp-block-heading">Etherscan based smart contract honeypots</h2>



<p>All the smart contracts that we have seen until now, exploit a solidity language gap of knowledge in the user. However, in this section of this blog post, the deployer exploits some features related to etherscan platform to hide some important information that may trick users.</p>



<h3 class="wp-block-heading">Hidden State Update</h3>



<p>The Etherscan platform helps developers and any Ethereum Blockchain user to debug his smart contract or track his transactions. Therefore, the platform display user’s transaction and internal messages that are performed by smart contracts. However, one of the features of Etherscan is that it does not show internal messages with an empty value.</p>



<p>Therefore, smart contract honeypot deployer exploit this feature to trick users and change the smart contract behavior. Here is an example to better understand this concept:</p>



<p>Check the following smart contract:&nbsp;<a href="https://etherscan.io/address/0x8bbf2d91e3c601df2c71c4ee98e87351922f8aa7#code">https://etherscan.io/address/0x8bbf2d91e3c601df2c71c4ee98e87351922f8aa7#code</a></p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="653" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-38-1024x653.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2294" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-38-1024x653.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-38-300x191.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-38-768x490.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-38.png 1217w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>


<p>This contract might be used as a honeypot, as the user could be fooled by the initial value of the variable passHasBeenSet. By checking the Etherscan data he would not be able to see any transaction that has changed the value of passHasBeenSet. Therefore, he would assume that the value didn’t change and attempt to exploit the contract.</p>



<p>To do that, the user would try to exploit the contract by sending more than one ether to the contract using the GetGift() after setting the hashPass using SetPass() function.</p>



<p>However, the passHasBeenSet variable might be already changed by another contract and that would not be seen in the etherscan platform.</p>



<h3 class="wp-block-heading">Straw Man Contract</h3>



<p>This technique is built upon showing a source code for a contract that is not actually the one used by the contract. For example, the deployer could build a contract that requires another library and that that library address is initialized during the deployment of the contract or by calling a specific function.</p>



<p>At this stage, there is nothing that holds the deployer from using another contract address that is totally different than the one that the source code is displayed in Etherscan.</p>



<p>Unfortunately, this really a tricky honeypot and a really difficult technique to discover from a user. I mean the user should verify the addresses of the deployed contract and the different transactions and data passed to the contract to be able to find this issue. Moreover, even if the user tries to test this smart contract in a different contract, he will use the smart contract code displayed by the attacker and he will see a normal behavior. Which makes it even more difficult to find the issue.</p>



<p>Here is an example of such a honeypot, try to take a look at it and see what makes this smart contract a honeypot:</p>



<p><a href="https://etherscan.io/address/0xdc5c87ba250b65a83042333f1101940b74312a65#code">https://etherscan.io/address/0xdc5c87ba250b65a83042333f1101940b74312a65#code</a></p>


<div class="wp-block-image">
<figure class="aligncenter size-large"><img decoding="async" loading="lazy" width="1024" height="657" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/image-39-1024x657.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2295" srcset="https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-39-1024x657.png 1024w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-39-300x192.png 300w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-39-768x493.png 768w, https://cryptodeeptech.ru/wp-content/uploads/2023/06/image-39.png 1208w" sizes="(max-width: 1024px) 100vw, 1024px"></figure></div>


<hr class="wp-block-separator has-alpha-channel-opacity">



<p id="2b25"><a href="https://etherscan.io/" rel="noreferrer noopener" target="_blank">Etherscan</a>&nbsp;is an Ethereum blockchain explorer that, besides other features, allows developers to submit the code of the smart contracts they deploy. The main benefit of this feature is that it allows users to check what contracts do by reading their source code. Etherscan makes sure that the code matches the smart contract as deployed.</p>



<p id="1b96">The list of verified contracts is long. As of this writing, Etherscan offers the source code for 26055 contracts, which can be browsed&nbsp;<a href="https://etherscan.io/contractsVerified" rel="noreferrer noopener" target="_blank">here</a>.</p>



<p id="2d9a">On a lazy Sunday afternoon I decided to casually browse it to see what kind of contracts people were running and get a sense of what people use the blockchain for, and how well written and secure these contracts are. Most contracts I found implemented tokens, crowdsales, multi-signature wallets, ponzis, and.. honeypots!</p>



<p id="b7a1">Honeypot contracts are the most interesting findings to me. Such contracts hold ether, and&nbsp;<em>pretend</em>&nbsp;to do so insecurely. In short, they are scam contracts that try to fool you into thinking you can steal the ether they hold, while in fact all you can do is&nbsp;<em>lose</em>&nbsp;ether.</p>



<p id="9120">A common pattern they follow is, in order to retrieve the ether they hold, you must send them some ether of your own first. Of course, if you try that, you’re in for a nasty surprise: the smart contract eats up your ether, and you find out that the smart contract does not do what you thought it did.</p>



<p id="88d3">In this post I will analyze a couple honeypot contracts I came across, and explain what they&nbsp;<em>seem</em>&nbsp;to do, but&nbsp;<em>really</em>&nbsp;do.</p>



<h2 class="wp-block-heading" id="9d02">The not-really-insecure non-lottery</h2>



<p id="3032">The first contract I will go through implements a lottery that, apparently, is horribly insecure and easy to steal from with a guaranteed win. I have come across several of these. The last instance I found is deployed at address 0x8685631276cfcf17a973d92f6dc11645e5158c0c, and its source code can be read&nbsp;<a href="https://etherscan.io/address/0x8685631276cfcf17a973d92f6dc11645e5158c0c#code" rel="noreferrer noopener" target="_blank">here</a>. I am copying the code below for convenience. Can you spot the bait? Can you tell why, if you try to exploit it, you will actually lose ether?</p>



<pre class="wp-block-preformatted">pragma solidity ^0.4.23;// CryptoRoulette<br>//<br>// Guess the number secretly stored in the blockchain and win the whole contract balance!<br>// A new number is randomly chosen after each try.<br>//<br>// To play, call the play() method with the guessed number (1-16).  Bet price: 0.2 ethercontract CryptoRoulette {<br>    uint256 private secretNumber;<br>    uint256 public lastPlayed;<br>    uint256 public betPrice = 0.001 ether;<br>    address public ownerAddr;    struct Game {<br>        address player;<br>        uint256 number;<br>    }<br>    Game[] public gamesPlayed;    constructor() public {<br>        ownerAddr = msg.sender;<br>        shuffle();<br>    }    function shuffle() internal {<br>        // randomly set secretNumber with a value between 1 and 10<br>        secretNumber = 6;<br>    }    function play(uint256 number) payable public {<br>        require(msg.value &gt;= betPrice &amp;&amp; number &lt;= 10);<br>        Game game;<br>        game.player = msg.sender;<br>        game.number = number;<br>        gamesPlayed.push(game);        if (number == secretNumber) {<br>            // win!<br>            msg.sender.transfer(this.balance);<br>        }        //shuffle();<br>        lastPlayed = now;<br>    }    function kill() public {<br>        if (msg.sender == ownerAddr &amp;&amp; now &gt; lastPlayed + 6 hours) {<br>            suicide(msg.sender);<br>        }<br>    }    function() public payable { }<br>}</pre>



<p id="6bad">It’s easy to tell that the&nbsp;<code>shuffle()</code>&nbsp;method sets&nbsp;<code>secretNumber</code>&nbsp;to 6. Hence, if you call&nbsp;<code>play(6)</code>and send it 0.001 ether, you will always win your ether plus whatever the balance of the contract is, namely 0.015 ether. Easy money, right? Wrong.</p>



<p id="848e">What’s the trick? Look closely at how&nbsp;<code>play()</code>&nbsp;is implemented. It declares a variable&nbsp;<code>Game game</code>, but does not initialize it. It will therefore default to a pointer to slot zero of the contract’s storage space. Then, it stores your address in its first member, storage slot 0, and the submitted number in the second one, that maps to storage slot 1. So, in practice, this will end up overwriting the contract’s&nbsp;<code>secretNumber</code>&nbsp;with the attacker account’s address, and&nbsp;<code>lastPlayed</code>&nbsp;with the number submitted.</p>



<p id="8ee2">Then, it will compare&nbsp;<code>secretNumber</code>, which is now your account’s address, with the number you submitted. Since you can only submit numbers smaller than 10, you can only win if your account’s address is within the range 0x0 to 0x0a. (Don’t bother trying to bruteforce-search for one account in that small range! Simply unfeasible.)</p>



<p id="aa97">So, the comparison will fail, and the contract will keep your ether. Of course, the attacker can at any time call&nbsp;<code>kill()</code>&nbsp;to retrieve the ether.</p>



<h2 class="wp-block-heading" id="3aeb">The not-really-insecure non-riddle</h2>



<p id="6a13">This is&nbsp;<a href="https://etherscan.io/address/0x3CAF97B4D97276d75185aaF1DCf3A2A8755AFe27#codepragma" rel="noreferrer noopener" target="_blank">another fun one</a>. It had me scratching my head for a while. However, there is a<em>&nbsp;</em>huge giveaway that the contract is up to something nasty right away. But let’s not get ahead of ourselves.</p>



<p id="897c">Here is its code. Can you spot the supposed vulnerability? And, can you tell why an exploit won’t work? And what is the giveaway I was talking about?</p>



<pre class="wp-block-preformatted">contract G_GAME<br>{<br>    function Play(string _response)<br>    external<br>    payable<br>    {<br>        require(msg.sender == tx.origin);<br>        if(responseHash == keccak256(_response) &amp;&amp; msg.value&gt;1 ether)<br>        {<br>            msg.sender.transfer(this.balance);<br>        }<br>    }<br>    <br>    string public question;<br>    address questionSender;<br>    bytes32 responseHash;<br> <br>    function StartGame(string _question,string _response)<br>    public<br>    payable<br>    {<br>        if(responseHash==0x0)<br>        {<br>            responseHash = keccak256(_response);<br>            question = _question;<br>            questionSender = msg.sender;<br>        }<br>    }<br>    <br>    function StopGame()<br>    public<br>    payable<br>    {<br>       require(msg.sender==questionSender);<br>       msg.sender.transfer(this.balance);<br>    }<br>    <br>    function NewQuestion(string _question, bytes32 _responseHash)<br>    public<br>    payable<br>    {<br>        require(msg.sender==questionSender);<br>        question = _question;<br>        responseHash = _responseHash;<br>    }<br>    <br>    function() public payable{}<br>}</pre>



<p id="69d1">The code supposedly implements a riddle. It sets up a question, and, if you can tell what the answer is, it will presumably send you its balance, currently a little more than 1 ether. Of course, to produce an answer, you must send an ether first, which you will get back if you are correct. The code seems fine, but there is a dirty trick: notice how&nbsp;<code>NewQuestion</code>&nbsp;allows&nbsp;<code>questionSender</code>&nbsp;to submit a hash that does not match&nbsp;<code>_question</code>. So, as long as this function isn’t used, we should be alright.</p>



<p id="4024">Can we tell what the question and answer are? If you read the&nbsp;<a href="https://etherscan.io/address/0x3CAF97B4D97276d75185aaF1DCf3A2A8755AFe27" rel="noreferrer noopener" target="_blank">transaction history of the contract on etherscan</a>, it appears that the&nbsp;<a href="https://etherscan.io/tx/0xa31f023b306fd48facbf5ce54a9c5690edbf1ad90caa676785f431ae49a1ab69" rel="noreferrer noopener" target="_blank">2nd transaction</a>&nbsp;sets up the question. It’s even more obvious if you click the “Convert to UT8” button on etherscan. This reveals the question “<em>I am very easy to get into,but it is hard to get out of me. What am I?</em>”, and the answer “<em>TroublE</em>”.</p>



<p id="e1d1">Since this transaction is called, according to etherscan, after the creation of the contract,&nbsp;<code>responseHash</code>&nbsp;is going to be zero, and will become&nbsp;<code>keccak265("TroublE")</code>. Then, there is a third transaction that loads up one ether in the contract. So, apparently, we could call&nbsp;<code>Play("TroublE")</code>&nbsp;and send one ether to get two ether back. Too good to be true? Probably. Let’s make sure.</p>



<p id="de1f">We can make sure we will the contract’s ether by inspecting the state of the smart contract. Its variables are not&nbsp;<code>public</code>, but still all it takes is just a few extra strokes to retrieve their values by querying the blockchain.&nbsp;<code>questionSender</code>&nbsp;and&nbsp;<code>responseHash</code>&nbsp;are the 2nd and 3rd variables, so they will occupy slots 1 and 2 on the storage space of the smart contract. Let’s retrieve their values.</p>



<pre class="wp-block-preformatted">web3.eth.getStorageAt(‘0x3caf97b4d97276d75185aaf1dcf3a2a8755afe27’, 1, console.log);</pre>



<p id="fde3">The result is `0x0..0765951ab946f3a6f0379680a6b05fb807d52ba09`. That spells trouble (pun intended) for an attacker, since the transaction setting up the question came from an account starting with<code>0x21d2</code>. Something’s up.</p>



<pre class="wp-block-preformatted">web3.eth.getStorageAt(‘0x3caf97b4d97276d75185aaf1dcf3a2a8755afe27’, 2, console.log);</pre>



<p id="00d4">The result is `0xc3fa7df9bf24…`. Is this the hash of “TroublE”?</p>



<pre class="wp-block-preformatted">web3.sha3('TroublE');</pre>



<p id="3044">That call returns&nbsp;<code>0x92a930d5...</code>, so it turns out that, if we were to call&nbsp;<code>Play("TroublE")</code>&nbsp;and send 1 ether, we’d actually lose it. But how is it possible that the hashes do not match?</p>



<p id="9236">Notice how&nbsp;<code>StartGame</code>&nbsp;does nothing if&nbsp;<code>responseHash</code>&nbsp;is already set. Clearly, that second transaction did not alter the state of the contract, so it must have already been set before this transaction. But how is it possible that&nbsp;<code>responseHash</code>&nbsp;was already initialized, if that was the first transaction after the creation of the contract?</p>



<p id="3571">After some serious head scratching, I found a recent&nbsp;<a href="https://medium.com/@gerhard.wagner/the-phenomena-of-smart-contract-honeypots-755c1f943f7b">interesting post</a>&nbsp;on honeypot contracts that explains that Etherscan does not show transactions between contracts when&nbsp;<code>msg.value</code>&nbsp;is zero. Other blockchain explorers such as Etherchain&nbsp;<em>do</em>&nbsp;show them. Surely enough, etherchain reveals&nbsp;<a href="https://www.etherchain.org/account/3caf97b4d97276d75185aaf1dcf3a2a8755afe27" rel="noreferrer noopener" target="_blank">a couple additional transactions</a>&nbsp;in the contract’s history, where a contract at&nbsp;<code>0x765951..</code>&nbsp;modifies&nbsp;<code>responseHash</code>&nbsp;via a zero-value transactions.</p>



<p id="a22d">So let’s check these transactions; perhaps the ether can still be stolen? To track what happened, we need to decode these calls. We can get the contract’s ABI&nbsp;<a href="https://etherscan.io/address/0x3CAF97B4D97276d75185aaF1DCf3A2A8755AFe27#code" rel="noreferrer noopener" target="_blank">from Etherscan</a>, and the internal transaction data from the “parity traces” of Etherchain (<a href="https://www.etherchain.org/tx/4581ff1f1242b21c16f4f8bf0b0de19153c95d1461b9105543629d005e18c956/parityTrace" rel="noreferrer noopener" target="_blank">first</a>,&nbsp;<a href="http://a5b368a448ae104747db475797aa6157e5db0729e3d90f46c0250061596cccf7/" rel="noreferrer noopener" target="_blank">second</a>). That’s all we need to decode the transactions into human readable format.</p>



<pre class="wp-block-preformatted">const abiDecoder = require('abi-decoder');<br>const Web3 = require('web3');<br>const web3 = new Web3();const abi = [{“constant”:false,”inputs”:[{“name”:”_question”,”type”:”string”},{“name”:”_response”,”type”:”string”}],”name”:”StartGame”,”outputs”:[],”payable”:true,”stateMutability”:”payable”,”type”:”function”},{“constant”:false,”inputs”:[{“name”:”_question”,”type”:”string”},{“name”:”_responseHash”,”type”:”bytes32"}],”name”:”NewQuestion”,”outputs”:[],”payable”:true,”stateMutability”:”payable”,”type”:”function”},{“constant”:true,”inputs”:[],”name”:”question”,”outputs”:[{“name”:””,”type”:”string”}],”payable”:false,”stateMutability”:”view”,”type”:”function”},{“constant”:false,”inputs”:[{“name”:”_response”,”type”:”string”}],”name”:”Play”,”outputs”:[],”payable”:true,”stateMutability”:”payable”,”type”:”function”},{“constant”:false,”inputs”:[],”name”:”StopGame”,”outputs”:[],”payable”:true,”stateMutability”:”payable”,”type”:”function”},{“payable”:true,”stateMutability”:”payable”,”type”:”fallback”}];const data1 = '0x1f1c827f000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000000464920616d2076657279206561737920746f2067657420696e746f2c627574206974206973206861726420746f20676574206f7574206f66206d652e205768617420616d20493f0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000754726f75626c4500000000000000000000000000000000000000000000000000';const data2 = '0x3e3ee8590000000000000000000000000000000000000000000000000000000000000040c3fa7df9bf247d144f6933776e672e599a5ed406cd0a15a9f2da09055b8f906700000000000000000000000000000000000000000000000000000000000000464920616d2076657279206561737920746f2067657420696e746f2c627574206974206973206861726420746f20676574206f7574206f66206d652e205768617420616d20493f0000000000000000000000000000000000000000000000000000';abiDecoder.addABI(abi);<br>console.log(abiDecoder.decodeMethod(data1));<br>console.log(abiDecoder.decodeMethod(data2));</pre>



<p id="202d">Running this code, we get the following result:</p>



<pre class="wp-block-preformatted">{ name: ‘StartGame’,<br>  params: [ { name: ‘_question’,<br>              value: ‘I am very easy to get into,but it is hard to get out of me. What am I?’,<br>              type: ‘string’ },<br>            { name: ‘_response’,<br>              value: ‘TroublE’,<br>              type: ‘string’ }<br>  ]<br>}<br>{ name: ‘NewQuestion’,<br>  params: [ { name: ‘_question’,<br>              value: ‘I am very easy to get into,but it is hard to get out of me. What am I?’,<br>              type: ‘string’ },<br>            { name: ‘_responseHash’,<br>              value: ‘0xc3fa7df9bf247d144f6933776e672e599a5ed406cd0a15a9f2da09055b8f9067’,<br>              type: ‘bytes32’ }<br>  ]<br>}</pre>



<p id="ca72">We learn that the first transaction sets the answer to&nbsp;<code>keccak256("TroublE")</code>, but the second one sets the answer to a hash value for which we don’t know the original data! Again it’s quite easy to miss that the second call does not use&nbsp;<code>_question</code>&nbsp;to compute the hash; instead, it’s set to an arbitrary value that does not match the string provided in the previous call, although the question does match.</p>



<p id="6753">So, unless we can find out a value that produces the given hash, possibly via a dictionary attack or a bruteforce search, we’re out of luck. And, given how sophisticated this honeypot is, I would assume trying to bruteforce the hash is not going to work out very well for us.</p>



<p id="5810">Unraveling this honeypot took quite some effort. Its creator is ultimately counting on attackers trusting the etherscan data, which does not contain the full picture.</p>



<h2 class="wp-block-heading" id="7e22">The giveaway</h2>



<p id="7aa6">I said this contract contains a dead giveaway that its creator is playing tricks. This is in this line:</p>



<pre class="wp-block-preformatted">require(msg.sender == tx.origin);</pre>



<p id="0a8b">What this line achieves is, it prevents contracts from calling&nbsp;<code>Play</code>. This is because&nbsp;<code>tx.origin</code>&nbsp;is always an “external account”, and never a smart contract. Why is this useful for the attacker? A way to safely attack a contract is to call them from an “attack contract” that reverts execution if it didn’t gain ether from attack:</p>



<pre class="wp-block-preformatted">function attack() {<br>    uint intialBalance = this.balance;<br>    attack_contract();<br>    require (this.balance &gt; initialBalance);<br>}</pre>



<p id="f556">This way, unless the attacker’s contract’s balance increases, the transaction fails altogether. The creator of the honeypot wants to prevent an attacker from using this trick to protect themselves.</p>



<hr class="wp-block-separator has-alpha-channel-opacity">



<h2 class="wp-block-heading">Literature:</h2>



<ul>
<li><em><a href="https://cryptodeep.ru/doc/paper.pdf" target="_blank" rel="noreferrer noopener">ETHEREUM: A SECURE DECENTRALISED GENERALISED TRANSACTION LEDGER BERLIN VERSION beacfbd – 2022-10-24 DR. GAVIN WOOD FOUNDER, ETHEREUM &amp; PARITY</a></em></li>



<li><em><a href="https://cryptodeep.ru/doc/From_Smart_to_Secure_Contracts_Automated_Security_Assessment_and_Improvement_of_Ethereum_Smart_Contracts.pdf" target="_blank" rel="noreferrer noopener">From Smart to Secure Contracts: Automated Security Assessment and Improvement of Ethereum Smart Contracts Christof Ferreira Torres</a></em></li>



<li><em><a href="https://cryptodeep.ru/doc/The_Art_of_The_Scam_Demystifying_Honeypots_in_Ethereum_Smart_Contracts.pdf" target="_blank" rel="noreferrer noopener">The Art of The Scam: Demystifying Honeypots in Ethereum Smart Contracts Christof Ferreira Torres, Mathis Steichen, and Radu State, University of Luxembourg</a></em></li>



<li><em><a href="https://cryptodeep.ru/doc/A_survey_of_attacks_on_Ethereum_smart_contracts.pdf" target="_blank" rel="noreferrer noopener">A survey of attacks on Ethereum smart contracts Nicola Atzei, Massimo Bartoletti, and Tiziana Cimoli</a></em></li>
</ul>



<hr class="wp-block-separator has-alpha-channel-opacity">



<h2 class="wp-block-heading" id="3a4f">Conclusion</h2>



<p id="b5fa">Honeypots are a moral grey area for me. Is it OK to scam those who are looking to steal from contracts? I don’t think so. But I do not feel very strongly about this. In the end, if you got scammed, it is because you were searching for smart contracts to steal from to begin with.</p>



<p id="7e6e">These scams play on the greed of people who are smart enough to figure out an apparent vulnerability in a contract, yet not knowledgeable enough to figure out what the underlying trap is.</p>



<p id="18d0">If you want to get deeper into Smart Contract security, check this amazing wargame called&nbsp;Capture the Ether. It’s a fun way to hone your skills and train your eye for suspicious Solidity code.</p>



<hr class="wp-block-separator has-alpha-channel-opacity">



<p><strong><a href="https://github.com/demining/CryptoDeepTools/tree/main/23SolidityVulnerableHoneypots" target="_blank" rel="noreferrer noopener">GitHub</a></strong></p>



<p><strong><a href="https://t.me/cryptodeeptech" target="_blank" rel="noreferrer noopener">Telegram:&nbsp;https://t.me/cryptodeeptech</a></strong></p>



<p><a href="https://youtu.be/UrkOGyuuepE" target="_blank" rel="noreferrer noopener"><strong>Video: https://youtu.be/UrkOGyuuepE</strong></a></p>



<p><strong><a href="https://cryptodeeptech.ru/solidity-vulnerable-honeypots" target="_blank" rel="noreferrer noopener">Source: https://cryptodeeptech.ru/solidity-vulnerable-honeypots</a></strong></p>



<hr class="wp-block-separator has-alpha-channel-opacity">



<figure class="wp-block-image size-full"><img decoding="async" width="1280" height="720" src="./Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots - CRYPTO DEEP TECH_files/043.png" alt="Phenomenon from Blockchain Cryptocurrency Solidity Vulnerable Honeypots" class="wp-image-2301"></figure>
	</div><!-- .entry-content -->

