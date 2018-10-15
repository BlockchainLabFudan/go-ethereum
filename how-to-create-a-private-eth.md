## How to create a private Ethereum network


安装go-ethereum：https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum
安装solidity：https://solidity.readthedocs.io/en/latest/installing-solidity.html#building-from-source

搭建私链教程地址：https://omarmetwally.blog/2017/07/25/how-to-create-a-private-ethereum-network/
需要注意的地方： 
1.step2中genesis.json需要修改gasLimit为0xffffffff
2.step3中 bootnodes=enode://148f3....@101.102.103.104:3031 修改为本地地址bootnodes=enode://148f3....@127.0.0.1:3031
3.step5中不需要重新开一个终端， 直接中断step3中的终端 
在原先的命令
geth --datadir=/path/to/data --bootnodes=enode://148f3....@127.0.0.1:3031 
后加上--mine --minerthreads=1 --etherbase=0x… 选项
并在step4输入exit后重新attach


## STEP 1: SET UP A VIRTUAL SERVER AND INSTALL ETHEREUM COMMAND-LINE TOOLS

Many tutorials guide you through deploying contracts using the Ethereum wallet GUI. I’m using the Go Ethereum client (geth) and encourage others to learn how to use the command line interface (CLI). The better you understand the Ethereum client’s internal workings and the anatomy of a blockchain, the more power to you. It doesn’t matter which hosting service you use,  but these instructions assume you’re running an Ubuntu server. [Geth Installation instructions for Ubuntu] 

My private network is called “UCSFnet” at this (fake) IP address 101.102.103.104

Very important: make sure you set aside at least 2GB RAM on your virtual server in order for mining to work. If you skimp on RAM, mining may not work.

Now open a new terminal window, log into your Ubuntu server via SSH and create your working directory called “ucsfnet”

```
ssh root@101.102.103.104
mkdir ucsfnet
cd ucsfnet
mkdir data
mkdir source
```

## STEP 2: CREATE GENESIS BLOCK

The genesis block is the first block of any blockchain and its parameters are specified in “genesis.json”, which I saved under /root/ucsfnet/genesis.json. My genesis block looks something like this:
```
{
"config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },

  "alloc"      : {
  "0x0000000000000000000000000000000000000001": {"balance": "111111111"},
  "0x0000000000000000000000000000000000000002": {"balance": "222222222"}
    },

  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x00001",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000107",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

Note that chainId=1 refers to the main Ethereum network, so it’s important to create a unique chainId for your network so your client doesn’t confuse your private blockchain with the main network. For the sake of illustration and testing, set mining difficulty to a low value. Also make sure you specify a unique nonce to start out with. The alloc field allows you to pre-populate accounts with Ether.

Now navigate to the directory where you created your genesis.json file and initialize the bootnode, a node through which your Ethereum client can join your private network and interact with other nodes connected to your private network.

```
cd /root/ucsfnet/data
geth --datadir=/root/ucsfnet/data init /root/ucsfnet/genesis.json
bootnode --genkey=boot.key
bootnode --nodekey=boot.key
```

Pay attention to the enode address that the previous commands results. You’ll need this for the next step.


## STEP 3: CONNECT TO YOUR EXTERNALLY REACHABLE BOOTNODE
The bootnode you created in the previous step is externally reachable, and in this step we’re going to connect to that node to create a one-node network. As you go through this tutorial and expand your knowledge of Ethereum networking, you’ll be able to add infinitely many nodes to your private network. The following commands create the geth.ipc file you’ll need to interact with your network in the subsequent steps. However after completing this tutorial, you’ll be able to skip this step the next time you connect to your network if you’re developing/testing and not using any networking protocols.

Open a new terminal window:

```
ssh root@101.102.103.104

geth --datadir=/root/ucsfnet/data --bootnodes=enode://148f3....@101.102.103.104:3031
```

Replace 148f3…. above with the enode address from Step 2. Make sure to include the enode://  prefix.


## STEP 4: CREATE A NEW ACCOUNT AND CHECK YOUR BALANCE

Start by opening a new terminal window and connecting to your virtual server via SSH:

```
ssh root@101.102.103.104
geth attach /root/ucsfnet/data/geth.ipc 
> eth.accounts
```

You’ll notice that there are no account addresses yet. Now you’ll create your first account. Be sure to replace “mypassword” with a strong password. Remember it and keep it safe; Ethereum is unforgiving in that there is no way to unlock your account if you forget your password!

```
> personal.newAccount("mypassword")
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
```

Remember your account’s address, which is prefixed by “0x”. Notice that your account has zero Ether, but don’t be discouraged. I’ll soon turn you into an Ethereum billionaire (but alas, the Ether you collect on your private network is only good on your private network).


## STEP 5: MINE ON YOUR PRIVATE NETWORK

The purpose of mining here is two-fold. First you create Ether which you’ll need to power your transaction through gas (an Ethereum sub-unit). Second, mining incorporates your transactions into the blockchain. Open a new terminal window and connect to your private server:

```
ssh root@101.102.103.104
geth --datadir=/root/ucsfnet/data --mine --minerthreads=1 --etherbase=0x...
```

The etherbase parameter specifies the address which should receive the Ether you generate by mining. This should contain your wallet address from Step 4. If you check your account balance again (Step 4), you’ll find that you quickly became a billionaire. Congratulations! Again, the Ether you generate on your private network is only good on your private network, but that’s OK. Your newfound knowledge of how to develop for Ethereum is more valuable than all the Ether in the world.


## STEP 6: COMPILE A SIMPLE CONTRACT

Unfortunately, the official Ethereum documentation has not been updated to reflect the fact that compiling using the solC compiler is no longer possible via RPC. This means that you will end up in a rabbit hole if you try to follow the instructions on the official Ethereum documentation and many other tutorials based on official documentation. There are 2 ways to compile contracts, and I encourage you do try both so you understand what’s going on under the hood.

First install the command-line compiler, solC:

```
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

Now open a new terminal window, connect to your server via SSH, and navigate to the directory where your source code will live:

```
cd /root/ucsfnet/source
```

And save this demo “greeter” contract in /root/ucsfnet/greeter.sol:

```
contract mortal {

/* Define variable owner of the type address*/
 address owner;

/* this function is executed at initialization and sets the owner of the contract */
 function mortal() { owner = msg.sender; }

/* Function to recover the funds on the contract */
 function kill() { if (msg.sender == owner) selfdestruct(owner); }
}

contract greeter is mortal {
 /* define variable greeting of the type string */
 string greeting;

/* this runs when the contract is executed */
 function greeter(string _greeting) public {
 greeting = "UCSFnet lives!";
 }

/* main function */
 function greet() constant returns (string) {
 return greeting;
 }
}
```

Now compile this contract using solC:

```
solc --bin --abi  -o /root/ucsfnet/source /root/ucsfnet/source/greeter.sol
```

In the above command, the bin and abi flags tell the solC compiler to generate Ethereum Virtual Machine (EVM) bytecode and an Application Binary Inferface (ABI) file, respectively. The o flag defines an output directory where these files will be created, and the last argument is the location of the contract (*.sol file) you want to compile.

Compiling a contract this way generates two files:

EVM file, indicated by the *.bin extension. This corresponds to the contract bytecode generated by the web-based compiler, which is easier to use and will be described below.

ABI file, indicated by an *.abi extension. An application binary interface is like an outline or template of your contract and helps you and others interact with the contract when it’s live on the blockchain.

Open both files using vim or your text editor of choice to see what they contain. Understanding what these files contain will make deploying and interacting with your contracts much clearer.

Alternatively, you can use the online compiler. This is easier because you can just copy-paste your Solidity code. However both methods are equivalent, and I’ll elaborate on this below. Copy and paste the code in greeter.sol (above) into the online compiler. Give it a second, and then click on the “Contract details…” link. Notice that the contents of the Bytecode field are equivalent to the greeter.bin file that you created using solC. Also notice that the Interface field is equivalent to the contents of the greeter.abi file you created when you compiled using the command-line solC compiler.


## STEP 7: DEPLOY A “GREETER” CONTRACT TO YOUR PRIVATE NETWORK

Copy the contents of the Web3 deploy field in the online compiler. Paste them into a text editor on your computer and note the bolded changes I made:

```
var _greeting = 'UCSFnet lives!';

var browser_ballot_sol_greeterContract = web3.eth.contract([{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"type":"constructor"}]);

var browser_ballot_sol_greeter = browser_ballot_sol_greeterContract.new(

  _greeting,

  {

    from: web3.eth.accounts[0],

    data: '0x6060604052341561000f57600080fd5b6040516103dd3803806103dd833981016040528080518201919050505b5b336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055505b6040805190810160405280600d81526020017f48656c6c6f2c20576f726c642100000000000000000000000000000000000000815250600190805190602001906100b99291906100c1565b505b50610166565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061010257805160ff1916838001178555610130565b82800160010185558215610130579182015b8281111561012f578251825591602001919060010190610114565b5b50905061013d9190610141565b5090565b61016391905b8082111561015f576000816000905550600101610147565b5090565b90565b610268806101756000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806341c0e1b514610049578063cfae32171461005e575b600080fd5b341561005457600080fd5b61005c6100ed565b005b341561006957600080fd5b61007161017f565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100b25780820151818401525b602081019050610096565b50505050905090810190601f1680156100df5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16141561017c576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b5b565b610187610228565b60018054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561021d5780601f106101f25761010080835404028352916020019161021d565b820191906000526020600020905b81548152906001019060200180831161020057829003601f168201915b505050505090505b90565b6020604051908101604052806000815250905600a165627a7a7230582069d50e4318daa30d3f74bb817c3b0cb732c4ec6a493eb108266c548906c8b6d70029',

    gas: '1000000'

  }, function (e, contract){

   console.log(e, contract);

   if (typeof contract.address !== 'undefined') {

        console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);

   }

})
```

Save the above as myContract.js.

Before we proceed, make sure your account balance is non-zero and that your account is unlocked. If your balance is too low, or if your account is locked, you won’t be able to deploy your contract. Follow the above steps to make sure you generate Ether by mining, and unlock your account as follows (replacing mypassword with the password you invented while creating your account):

Open new terminal window:


```
ssh root@101.102.103.104 
geth attach /root/ucsfnet/data/geth.ipc 
> web3.fromWei(eth.getBalance(eth.accounts[0]), "ether")
> personal.unlockAccount(eth.accounts[0], "mypassword")
```

If I’ve lost you, pay attention here because this is important. This is where the Ethereum documentation hasn’t been updated yet, creating confusion regarding how contracts are compiled. Note that the web3.eth.contract() function takes as an argument an ABI. This is the same as the greeter.abi file you created using the solC compiler! Also note that the data field is equivalent to the greeter.bin EVM bytecode, with the exception that it’s prefixed by “0x”. 

Time to deploy your contract:


```
loadScript(myContract.js)
```

You should see something like:
```
Contract mined! address: 0x4000737c8bd7bbe3dee190b6342ba1245f5452d1 transactionHash: 0x0a4c798467f9b40f2c4ec766657d0ec07c324659ea76fcc9c8ad28fc0a192319
```

Congratulations! Your contract lives at the followin address on your private blockchain:

0x4000737c8bd7bbe3dee190b6342ba1245f5452d1

Make a note of this. You’ll need your contract’s address to interact with it (and allow others to do the same).

If this didn’t work, make sure you’re actively mining in another window so that your transaction is incorporated into the blockchain.


## STEP 8: INTERACTING WITH A CONTRACT

Again, this is another place where the Ethereum documentation hasn’t yet been updated and refers to deprecated functions. With your geth client fired up, try the following:

```
> var abi = '[{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"type":"constructor"}]'
> var abi = JSON.parse(abi)
> var contract = web3.eth.contract(abi)
> var c = contract.at("0x4000737c8bd7bbe3dee190b6342ba1245f5452d1")
> c.greet()
```

The first 3 inputs above define what the contract looks like via its ABI. The contract.at() method takes as an argument the address where your contract lives.

You should see the following output:

```
UCSFnet lives!
```

Big caveat: geth chokes if any of your inputs contain illegal characters. Id est:

```
“ (illegal) is not equal to  " (legal) 

        and

‘ (illegal) is not equal to ' (legal) 
```

## CONCLUSION

We covered a lot of ground here, from setting up your own Ethereum network to compiling your contract (and avoiding rabbit holes created by outdated documentation) to deploying the contract and interacting with it. Whew! If you ever get stuck and need to start over, just reboot your server and delete your blockchain (understanding the implications of what this means…):

```
rm -R /root/ucsfnet/data/geth/chaindata
```

