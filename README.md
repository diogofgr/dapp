# Tutorial: Running your first dApp

This is a stripped down version of what @maheshmurthy wrote on this medium article:
https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-1-40d2d0d807c2 - automatic!

## How to run a smart contract on a test blockchain

1. make a folder for you new app and start a node project:
```
$ mkdir dApp
$ npm init
```

2. install web3js (this specific version) which is an Ethereum Javascrip API:
```
$ npm install ethereumjs-testrpc web3@0.20.1
```

3. install the Solidity code compiler:
```
$ npm install solc
```

4. create a smart contract:
```
$ touch Voting.sol
```

  1. copy this code to ` Voting.sol `:
  ```
  pragma solidity ^0.4.11;
  // We have to specify what version of compiler this code will compile with

  contract Voting {
    /* mapping field below is equivalent to an associative array or hash.
    The key of the mapping is candidate name stored as type bytes32 and value is
    an unsigned integer to store the vote count
    */

    mapping (bytes32 => uint8) public votesReceived;

    /* Solidity doesn't let you pass in an array of strings in the constructor (yet).
    We will use an array of bytes32 instead to store the list of candidates
    */

    bytes32[] public candidateList;

    /* This is the constructor which will be called once when you
    deploy the contract to the blockchain. When we deploy the contract,
    we will pass an array of candidates who will be contesting in the election
    */
    function Voting(bytes32[] candidateNames) {
      candidateList = candidateNames;
    }

    // This function returns the total votes a candidate has received so far
    function totalVotesFor(bytes32 candidate) returns (uint8) {
      if (validCandidate(candidate) == false) throw;
      return votesReceived[candidate];
    }

    // This function increments the vote count for the specified candidate. This
    // is equivalent to casting a vote
    function voteForCandidate(bytes32 candidate) {
      if (validCandidate(candidate) == false) throw;
      votesReceived[candidate] += 1;
    }

    function validCandidate(bytes32 candidate) returns (bool) {
      for(uint i = 0; i < candidateList.length; i++) {
        if (candidateList[i] == candidate) {
          return true;
        }
      }
      return false;
    }
  }
  ```

5. start the testrpc and leave it running in a terminal window
```
$ node_modules/.bin/testrpc
```

6. on another window:
  1. open the node console:
  ```
  $ node
  ```

  2. initialize the solc and web3 objects:
  ```
  > Web3 = require('web3')
  > web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
  ```

  3. make sure the web3 object is initailized and communicating with the blockchain:
  ```
  > web3.eth.accounts

  ['0x9c02f5c68e02390a3ab81f63341edc1ba5dbb39e',
  '0x7d920be073e92a590dc47e4ccea2f28db3f218cc',
  '0xf8a9c7c65c4d1c0c21b06c06ee5da80bd8f074a9',
  '0x9d8ee8c3d4f8b1e08803da274bdaff80c2204fc6',
  '0x26bb5d139aa7bdb1380af0e1e8f98147ef4c406a',
  '0x622e557aad13c36459fac83240f25ae91882127c',
  '0xbf8b1630d5640e272f33653e83092ce33d302fd2',
  '0xe37a3157cb3081ea7a96ba9f9e942c72cf7ad87b',
  '0x175dae81345f36775db285d368f0b1d49f61b2f8',
  '0xc26bda5f3370bdd46e7c84bdb909aead4d8f35f3']
  ```
  If the response above is something like:
  ```
  Error: Invalid JSON RPC response: undefined
  ```
  then make sure the testrpc is running on another terminal window (step 5)

7. Compile the contract:
  1. load all the code to a string variable
  ```
  > code = fs.readFileSync('Voting.sol').toString()
  ```
  2. compile it
  ```
  > solc = require('solc')
  > compiledCode = solc.compile(code)
  ```

8. Deploy!
```
> abiDefinition = JSON.parse(compiledCode.contracts[':Voting'].interface)
> VotingContract = web3.eth.contract(abiDefinition)
> byteCode = compiledCode.contracts[':Voting'].bytecode
> deployedContract = VotingContract.new(['Rama','Nick','Jose'],{data: byteCode, from: web3.eth.accounts[0], gas: 4700000})
> deployedContract.address
> contractInstance = VotingContract.at(deployedContract.address)
```
## How to interact with the contract

(to be continued)
