# Tutorial: Running your first dApp

This is a stripped down version of what [@maheshmurthy](https://github.com/maheshmurthy)
wrote on this medium article:
https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-1-40d2d0d807c2

## How to run a smart contract on a test blockchain

### 1. make a folder for you new app and _start a node project_:
```
$ mkdir dApp
$ npm init
```

### 2. _install web3js_ (this specific version) _and the Solidity compiler_:
```
$ npm install ethereumjs-testrpc web3@0.20.1
$ npm install solc
```

### 3. _create a smart contract_ ` Voting.sol ` and copy this code into it:
```
$ touch Voting.sol
```
```javascript
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

### 4. _start the testrpc_ and leave it running in a terminal window
```
$ node_modules/.bin/testrpc
```

### 5. on another window - open the node console and _initialize the solc and web3 objects_:
  ```
  $ node
  ```
  ```
  Web3 = require('web3')
  web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
  ```
  __Is it running?__ - make sure the web3 object is initailized and communicating with the blockchain:
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
  __[Error]__ If the response above is something like ` Error: Invalid JSON RPC response: undefined ` then make sure the testrpc is running on another terminal window (step 5)

### 6. Compile! - load all the code to a string variable and compile it:
  ```
  code = fs.readFileSync('Voting.sol').toString()
  solc = require('solc')
  compiledCode = solc.compile(code)
  ```

### 7. Deploy!
```
abiDefinition = JSON.parse(compiledCode.contracts[':Voting'].interface)
VotingContract = web3.eth.contract(abiDefinition)
byteCode = compiledCode.contracts[':Voting'].bytecode
deployedContract = VotingContract.new(['Rama','Nick','Jose'],{data: byteCode, from: web3.eth.accounts[0], gas: 4700000})
contractInstance = VotingContract.at(deployedContract.address)
```
## How to interact with the contract

### 1. Create an ` index.html ` and copy the following code:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello World DApp</title>
  <link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
  <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css' rel='stylesheet' type='text/css'>
</head>
<body class="container">
  <h1>A Simple Hello World Voting Application</h1>
  <div class="table-responsive">
    <table class="table table-bordered">
      <thead>
        <tr>
          <th>Candidate</th>
          <th>Votes</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Rama</td>
          <td id="candidate-1"></td>
        </tr>
        <tr>
          <td>Nick</td>
          <td id="candidate-2"></td>
        </tr>
        <tr>
          <td>Jose</td>
          <td id="candidate-3"></td>
        </tr>
      </tbody>
    </table>
  </div>
  <input type="text" id="candidate" />
  <a href="#" onclick="voteForCandidate()" class="btn btn-primary">Vote</a>
</body>
<script src="https://cdn.rawgit.com/ethereum/web3.js/develop/dist/web3.js"></script>
<script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
<script src="./index.js"></script>
</html>
```

### 2. Create a file ` index.js `

__Important__ - make the changes on the commented line!

```javascript
web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
abi = JSON.parse('[{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"totalVotesFor","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"validCandidate","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"bytes32"}],"name":"votesReceived","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"x","type":"bytes32"}],"name":"bytes32ToString","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"uint256"}],"name":"candidateList","outputs":[{"name":"","type":"bytes32"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"candidate","type":"bytes32"}],"name":"voteForCandidate","outputs":[],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"contractOwner","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"inputs":[{"name":"candidateNames","type":"bytes32[]"}],"payable":false,"type":"constructor"}]')
VotingContract = web3.eth.contract(abi);

// In your nodejs console, execute contractInstance.address to get the address at which the contract is deployed and change the line below to use your deployed address
contractInstance = VotingContract.at('0x2a9c1d265d06d47e8f7b00ffa987c9185aecf672');
candidates = {"Rama": "candidate-1", "Nick": "candidate-2", "Jose": "candidate-3"}

function voteForCandidate() {
  candidateName = $("#candidate").val();
  contractInstance.voteForCandidate(candidateName, {from: web3.eth.accounts[0]}, function() {
    let div_id = candidates[candidateName];
    $("#" + div_id).html(contractInstance.totalVotesFor.call(candidateName).toString());
  });
}

$(document).ready(function() {
  candidateNames = Object.keys(candidates);
  for (var i = 0; i < candidateNames.length; i++) {
    let name = candidateNames[i];
    let val = contractInstance.totalVotesFor.call(name).toString()
    $("#" + candidates[name]).html(val);
  }
});
```
### 3. open ` index.html ` in a browser and vote.

If it doesn's work I'm sorry, I did my best. Go on Goolge, Stack Overflow, you know the drill...

## Cheatsheet

__Get total__ votes:
```
contractInstance.totalVotesFor.call('Rama').toLocaleString()
```

__Vote for__ a Rama (this returns a tx hash):
```
contractInstance.voteForCandidate('Rama', {from: web3.eth.accounts[0]})
```
