# Distribute your own SPL token on Solana
In this quest we will be working on distributing our tokens to the different Solana wallets. We will learn to transfer the tokens to the wallets by minting or by transferring from one wallet to another wallet.

Hence in this quest we will be working on the distribution feature.

__What are different ways of distributing tokens to the other wallets ?__

1. Minting Token - (Covered in this quest)
2. Transfer Token - (Not covered in this quest, as it deserves it's own quest)
3. Airdrop Token - (Not covered in this quest, as it deserves it's own quest)

__Why distribute your Token ?__

Creating our own token can become the backbone of the DApps but if we are not able to transfer it to other wallets then creating our own token serves no purpose.

Hence to understand all the details of minting tokens to other wallets is very important. Which eventually helps in increasing the total supply of the token.

__What will you get after going through this quest ?__

1. You will be able to transfer your own tokens via the Minting process.
2. You will understand the difference between the key concepts like Minting Token, Transfer Token and Airdrop.
3. You will understand the importance of associated accounts while minting new tokens.

__Prerequisites to work on this quest__

1. Knowledge of crypto wallet (Phantom wallet specifically).
2. A basic react app with wallet connectivity feature.
3. Solana cluster can be : local, devnet or testnet.
4. For better understanding, please go through our first quest on __creating a wallet connection__ with react app.
## Setting up
__Folder structure expected of the react app to run this quest__

- Assuming app name as "DistributeToken"
	- App-Name
		- node_modules
		- public
		- src
			- utils
				- mintTokenToAssociatedAccount.js
			- App.js
		- Index.js
		- Package.json

__Basic terminologies to digest before we jump into the quest__

1. __Minting Token: __When you try to print new currency notes in the real world, it increases the supply of the notes which are already present in the market. In the same, if we need to increase the supply of our token in the market then we mint (print) the tokens to an associated account.  
Minting process will increase the total supply of the custom token.
2. __Transfer Token: __In the real world when you want to lend some money to your friend, you do not ask the government to print more money and give it to you, so that you can lend. What you do is you lend your money whatever you have with you to your friend. This lending doesn't increase the total supply of the fiat currency notes. In the similar way, when a user wants to send custom tokens to another user, then the user transfers tokens to a friend's wallet without ever minting new tokens.  
 This does not increase the total supply of the custom token.
3. __Airdrop Token: __This is the most expensive option of all the above methods as in this feature, there is no expectation that the user is going to have any Solana balance or Associated account with a token.  
This method is very helpful when we do the ICO for the application or when we want to incentivise people for using the services.
## Distribution flow of token via Minting
__![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/4be26db3-bf8e-4362-9a4f-ffbd5c9df8f0.jpg)__

__Steps involved in Token Distribution - Minting Process in the next subquests__
## Connect the application with mint owner wallet
Token distribution can only happen from the owner of the __mint authority __as it was discussed in the "Create SPL token - Web3" quest. Without the mint authority, none of the transactions for the distribution can take place and it will always through the Invalid Signature error.

Only mint authority owners can sign the transaction, hence connecting to the mint owner wallet is important.

File: src/App.js

const provider = window.solana

provider.connect()

provider.on("connect", async() => {

console.log("wallet got connected")

setProviderPub(provider.publicKey)

});

provider.on("disconnect", () => {

console.log("Disconnected from wallet");

});

In the code,

- First we are fetching the "solana" variable from the window object, if the solana variable is present that means the phantom wallet is installed as a chrome extension.
- Once we have the provider, then we are calling the "connect" function to connect with the phantom wallet. It will open a popup to ask the phantom password. Once you provide the correct password, it will connect the wallet to the application.
- Phantom provides a way to listen to the connected or disconnected events by using hooks like "connect" and "disconnect". Once phantom wallet is successfully authorised then it will trigger the "connect" method and we can use this method to start out DApp functionality
## Use the provider instance as the owner of all subsequent transactions.
Once the wallet is connected, then it becomes important to assign the provider as the owner which can be used throughout the feature to sign the transactions related to the token distribution.

File: src/App.js

const owner = provider;
## Fetch the correct public key of mint.
Correctness of the mintkey is very important as mintkey is also the token address which you want to mint. Each mint key is associated to a mint authority (wallet owner mostly). Hence having the correct pair of mintkey and mint authority key is important for minting process.

More info can be found in the "Create SPL token - Web3" quest.

Example: A mint public key is declared below, but it won't work for you as the owner of this mint key is different.

File: src/App.js

const mintPubkey = 'Bw94Agu3j5bT89ZnXPAgvPdC5gWVVLxQpud85QZPv1Eb' //SOLG mint authority
## Fetch the correct public key of the associated account.
Correctness of the associated public key is also very important. Associated accounts mapped with the above mint key can only receive these tokens. It cannot accept any other tokens. As associated accounts are associated with a particular token. Hence having the correct associated public key is important to receive the minted token.

Example: An associatedAccountPubkey is declared below, but it won't work for you as the mapped mint key of this account is different from the mint authority.

File: src/App.js

const associatedAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz'//SOLG mint authority
## mintToken utility function initiate the mint process
In this function, we are using the below parameters

__connection : __Connection instance to the solana cluster. Prefer to use devnet for the non-production application.

__owner: __Wallet provider by which the application is connected with.

__mintOwner: __Mint Owner account will be the same as owner in most of the case while creating the new mint.

__amount: __Number of tokens which needs to be transferred in Lamports.

__mintPubkey: __This is the public key of the mint, which you received while creating the new mint authority.

__associatedAccountPubkey: __This is the public key of the receiver's associated account which was created by mapping  it with __mintPubkey__.

__transaction.add : __This statement will add __mintTo__ internal function which will add the mint transaction schema.

__mintAuthorityPubkey: __This is the public key of the mint authority who has the rights to sign or decline the transaction.

__signAndSendTransaction: __This is the internal function which will verify the transaction by signing it.

File: src/utils/mintTokenToAssociatedAccount.js

async function mintToken({

connection,

owner,

mintOwner, // Account to hold token information

amount, // Number of tokens to issue

mintPubkey,

associatedAccountPubkey, // Account to hold newly issued tokens, if amount > 0

}) {

let transaction = new Transaction();

let signers = \[mintOwner\];

transaction.add(

mintTo({

//when sending for the first time where mint is derived for the first time

mintPubkey,

//when sending for the first time where initialAccount is derived for the first time to receive the newly created SPL token

destination: associatedAccountPubkey,

amount,

mintAuthorityPubkey: owner.publicKey,

}),

);

return await signAndSendTransaction(connection, transaction, mintOwner, signers);

}
## mintTo utility function to complete the mint process
This utility function will return the transaction object which will hold the instructions to make the transaction.

__keys: __It is an array of objects which contains different publicKeys which can be used in the transaction to perform relevant actions like send a token, create a new account, etc... Each object contains properties like pubkey, isSigner, isWritable.

pubkey: publicKey which will be used in the transaction  
isSigner: boolean variable

true: publicKey will be used as a signer

false: publicKey will not be used as a signer

isWritable: boolean variable

true: publicKey’s account will be writable, that means the signer can update the values

false: publicKey’s account can not be writable, that means the signer can not update any value.  
In this case it will be   
__mintPubkey__ : Token address which will be minted to mint new tokens.  
__destination__: This is the pubkey of the destination account.

__minAuthorityPubKey__: This is the public key of the mint authority who can perform the action like sign the transaction to mint new tokens. 

Except the __mintAuthorityPubkey __public key  isSigner:true other keys can be false. As only mintAuthorityPubkey will be used and has the rights to sign the transaction.

__TransactionInstruction: __It will create the instruction by encoding the instruction which will be decoded by the on-chain program to execute the mint.

__encodeTokenInstructionData: __It's a utility function to encode each instruction in bytes array and make it              compatible with on-chain solana programs.

__TOKEN_PROGRAM_ID__ : It is the public key of the deployed on-chain program for the Solana Token, which has all the inbuilt functions to create the mint authority and associated accounts.

File: src/utils/mintTokenToAssociatedAccount.js

function mintTo({ mintPubkey, destination, amount, mintAuthorityPubkey }) {

let keys = \[

{ pubkey: mintPubkey, isSigner: false, isWritable: true },

{ pubkey: destination, isSigner: false, isWritable: true },

{ pubkey: mintAuthorityPubkey, isSigner: true, isWritable: false },

\];

return new TransactionInstruction({

keys,

data: encodeTokenInstructionData({

mintTo: {

amount,

},

}),

programId: TOKEN_PROGRAM_ID,

});

}
## Signing the transactions
__Once we are ready with transaction, then signing it is important - signAndSendTransaction__

In this step we will be going through the flow of signing the mint transaction. To complete the mint process we need to write our block to the solana blockchain. To get this transaction on the blockchain it needs to be signed from the wallet owner and sent for the validation.

Few key variables to be used in above function

- transaction.recentBlockhash : As solana works on __proof of history__, hence every transaction in the solana blockchain requires the latest blockhash to be associated with the new transaction. Without the getRecentBlockhash, the validators will not be able to verify the transaction.
- proof of history__: __Proof of History is a sequence of computation that can provide a way to cryptographically verify passage of time between two events. Validator nodes "timestamp" blocks with cryptographic proofs that some duration of time has passed since the last proof. All data hashed into the proof most certainly have occurred before the proof was generated. The node then shares the new block with validator nodes, which are able to verify those proofs. The blocks can arrive at validators in any order or even could be replayed years later. With such reliable synchronization guarantees, Solana is able to break blocks into smaller batches of transactions called *entries*. Entries are streamed to validators in realtime, before any notion of block consensus.  
This is very different and efficient from the __Proof of Work__ or __Proof of Stake__ where the energy consumption is high as every node in the blockchain has to perform the action to validate a node. The winner gets the reward but other validator’s work gets wasted in the process.  

- wallet.publicKey: Every transaction needs a fee to be paid to the computers who are making the transaction possible. This parameter tells Solana who will be paying the fees for this transaction.   
  
wallet.publicKey = provider.publicKey __  
__  
In Ethereum, we have smart contracts and ethereum accounts which deploys these smart contracts. Hence the address of the smart contract is used to fetch the public function and interact with the smart contract. But there are few functions in smart contracts which can only be executed by the owner of the smart contract. 
- signers.map: List of accounts which will be used to sign the ongoing transaction, that is defined by this list of signers. If any action is going to be taken on this account, it requires the signature using the account’s private key. This ensures that the program never updates the account of a user without the permission of the owner of that account.
- transaction.serialize() : All the data must be stored on the blockchain. To keep the content format agnostic of the programming language used, the data is serialized before storing. 
- skipPreflight: It’s a bool data type
	- true: preflight transaction checks for the available methods before sending the transaction, which involves a very little latency.
	- false: (default value) : It is turned off by default to save the network bandwidth.
- preflightCommitment: Every transaction on Solana goes through a process of finalization. The longer the transaction has existed on the blockchain, the less likely it is to get reverted. A transaction is reverted when the blockchain realizes later that the transaction was actually not supposed to be allowed. However, the process of finalization takes some time. Depending on the security required in your code, you can choose between single, confirmed and finalized security levels. Single means that there is atleast one “confirmation” given to the transaction by a validator on Solana.

File: src/utils/mintTokenToAssociatedAccount.js

async function signAndSendTransaction(

connection,

transaction,

wallet,

signers,

skipPreflight = false,

) {

transaction.recentBlockhash = (

await connection.getRecentBlockhash('max')

).blockhash; // As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated

// Without the getRecentBlockhash, the validators will not be able to verify the transaction

transaction.setSigners(

// fee payed by the wallet owner

wallet.publicKey,

...signers.map((s) => s.publicKey), // we will add all the intermediate signers required for the transaction, in our case mint keypair is also required.

// Hence we have added the mint signer from the signers array.

);

transaction = await wallet.signTransaction(transaction);

const rawTransaction = transaction.serialize(); //Serialization is the process of converting an object into a stream of bytes,

//which can be used by on-chain programs to again de-serialize it to read the instructions

//and perform actions on it.

return await connection.sendRawTransaction(rawTransaction, {

skipPreflight, //preflight transaction check checks for the available methods before sending the transaction, which involves a very little latency

//that is why skipPreflight is generally kept false, to save the network bandwidth.

preflightCommitment: 'single',

//For preflight checks and transaction processing,

//Solana nodes choose which bank state to query based on a commitment requirement set by the client.

//The commitment describes how finalized a block is at that point in time. When querying the ledger state,

//it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.

//For processing many dependent transactions in series, it's recommended to use "confirmed" commitment,

//which balances speed with rollback safety. For total safety, it's recommended to use"finalized" commitment.

});

}
## Making our transaction Blockchain compliant
__LAYOUT __class will help us to build the structure in the required format as same as the struct format required by the Token on-chain program to read the instructions and process it.

BufferLayout provides us the conversion of Layout to array of bytes called __Encoding__ which can be transferred over the network and __Decoding __takes place in the solana on-chain program to find the instruction.

On-chain program perform the action based on the instructions it receives in the form of struct.

It also provides us the options to define the exact data types required by the solana on-chain program to process. Like u8, struct

__Key points to know:__

__u8__ : unsigned 8 bit integer, which means only positive integer will be supported of 8 bit

Example: __BufferLayout.u8('__instruction__')__ means 'instruction' is a variable of an unsigned 8 bit integer.

__struct: __is equivalent to class but it's a very light weight data type as compared to classes in lower level languages like c\+\+,c.

Example: __BufferLayout.struct(\[BufferLayout.nu64('amount')\]) __ here we are defining the struct with the __amount__ variable with its respective data type.

__encodeTokenInstructionData: __It's a utility function to encode each instruction in bytes array and make it              compatible with on-chain Solana programs.

File: src/utils/mintTokenToAssociatedAccount.js

const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));

LAYOUT.addVariant(

7,

BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

'mintTo',

);
## Final code to mint new tokens and distribute it to the associated account
File: src/utils/mintTokenToAssociatedAccount.js

import { TransactionInstruction, Transaction } from "@solana/web3.js";

import \* as BufferLayout from 'buffer-layout';

import { TOKEN_PROGRAM_ID } from "./programIds";

const LAYOUT = BufferLayout.union(BufferLayout.u8('instruction'));

LAYOUT.addVariant(

7,

BufferLayout.struct(\[BufferLayout.nu64('amount')\]),

'mintTo',

);

/\*\*

\* @why : This function will enable you to mint new tokens to user's associated account

\*

\* @Remarks: mintTokenToAssociatedAccount

\* A utility which can mint new custom SPL token to the existing associative token account,

\* which will directly increase the total supply of the custom SPL token.

\* @param {\*} wallet //wallet provider, to sign the transaction in case mintOwner is not same as wallet provider

\* @param {\*} connection //connection instance to the Solana Cluster

\* @param {\*} tokensToMint //Number of custom SPL tokens to mint

\* @param {\*} mintPubkey //Mint's public Key i.e custom SPL token's public key which was created by new createNewMintAuthority process.

\* @param {\*} associatedAccountPubkey //Public key of the account mapped to Mint's public key which was created by the user or SPL token mint authority.

\* @param {\*} mintOwner //mintOwner is the mint authority of the mint i.e. custom token, mostly it will be same as wallet provider who is going to make the transaction.

\* @returns

\* {status: false, error:message } //in case of any failure

\* {status: true, signature} //in case of transaction success

\*

\*/

export const mintTokenToAssociatedAccount = async (wallet, connection, tokensToMint, mintPubkey, associatedAccountPubkey, mintOwner) =>{

if(!tokensToMint){

return {status: false, error:"You can't mint 0 tokens"}

}

const tokensToMintInLamports = tokensToMint \* 1000000000;

const signature = await mintToken({

connection,

owner: wallet,

mintOwner,

amount: tokensToMintInLamports,

mintPubkey,

associatedAccountPubkey

})

console.log("Waiting for the signature confirmation")

await connection.confirmTransaction(signature);

console.log("Signature confirmed")

return {signature};

}

async function signAndSendTransaction(

connection,

transaction,

wallet,

signers,

skipPreflight = false,

) {

transaction.recentBlockhash = (

await connection.getRecentBlockhash('max')

).blockhash; // As solana works on proof of history, hence every transaction in the solana blockchain requires the latest blockhash to be associated

// Without the getRecentBlockhash, the validators will not be able to verify the transaction

transaction.setSigners(

// fee payed by the wallet owner

wallet.publicKey,

...signers.map((s) => s.publicKey), // we will add all the intermediate signers required for the transaction, in our case mint keypair is also required.

// Hence we have added the mint signer from the signers array.

);

transaction = await wallet.signTransaction(transaction);

const rawTransaction = transaction.serialize(); //Serialization is the process of converting an object into a stream of bytes,

//which can be used by on-chain programs to again de-serialize it to read the instructions

//and perform actions on it.

return await connection.sendRawTransaction(rawTransaction, {

skipPreflight, //preflight transaction check checks for the available methods before sending the transaction, which involves a very little latency

//that is why skipPreflight is generally kept false, to save the network bandwidth.

preflightCommitment: 'single',

//For preflight checks and transaction processing,

//Solana nodes choose which bank state to query based on a commitment requirement set by the client.

//The commitment describes how finalized a block is at that point in time. When querying the ledger state,

//it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.

//For processing many dependent transactions in series, it's recommended to use "confirmed" commitment,

//which balances speed with rollback safety. For total safety, it's recommended to use"finalized" commitment.

});

}

async function mintToken({

connection,

owner,

mintOwner, // Account to hold token information

amount, // Number of tokens to issue

mintPubkey,

associatedAccountPubkey, // Account to hold newly issued tokens, if amount > 0

}) {

let transaction = new Transaction();

let signers = \[mintOwner\];

transaction.add(

mintTo({

//when sending for the first time where mint is derived for the first time

mintPubkey,

//when sending for the first time where initialAccount is derived for the first time to receive the newly created SPL token

destination: associatedAccountPubkey,

amount,

mintAuthorityPubkey: owner.publicKey,

}),

);

return await signAndSendTransaction(connection, transaction, mintOwner, signers);

}

function mintTo({ mintPubkey, destination, amount, mintAuthorityPubkey }) {

let keys = \[

{ pubkey: mintPubkey, isSigner: false, isWritable: true },

{ pubkey: destination, isSigner: false, isWritable: true },

{ pubkey: mintAuthorityPubkey, isSigner: true, isWritable: false },

\];

return new TransactionInstruction({

keys,

data: encodeTokenInstructionData({

mintTo: {

amount,

},

}),

programId: TOKEN_PROGRAM_ID,

});

}

const instructionMaxSpan = Math.max(

...Object.values(LAYOUT.registry).map((r) => r.span),

);

function encodeTokenInstructionData(instruction) {

let b = Buffer.alloc(instructionMaxSpan);

let span = LAYOUT.encode(instruction, b);

return b.slice(0, span);

}
## How to run this quest
1. Install phantom wallet chrome extension and add some SOL to the wallet.
2. Create a react app 
	1. npx create-react-app distribute-token
	2. cd distribute-token
3. Copy paste the below initiator code in the App.js to invoke mintTokenToAssociatedAccount.js

import './App.css';

import { Connection, PublicKey } from "@solana/web3.js";

import \* as web3 from '@solana/web3.js';

import { useEffect, useState } from 'react';

import { mintTokenToAssociatedAccount } from './utils/mintTokenToAssociatedAccount';

import { transferCustomToken } from './utils/transferCustomToken';

const NETWORK = web3.clusterApiUrl("devnet");

const connection = new Connection(NETWORK);

function App() {

const \[provider, setProvider\] = useState()

const \[providerPubKey, setProviderPub\] = useState()

const \[mintSignature, setMintTransaction\] = useState()

const \[tokenSignature, setTokenTransaction\] = useState()

const mintTokenToAssociateAccountHandler = async () =>{

try{

const tokensToMint = 1

const mintPubkey = 'Bw94Agu3j5bT89ZnXPAgvPdC5gWVVLxQpud85QZPv1Eb' //SOLG mint authority

const associatedAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz'

const owner = provider;

const transactionSignature = await mintTokenToAssociatedAccount(owner, connection, tokensToMint, new PublicKey(mintPubkey), new PublicKey(associatedAccountPubkey), provider)

setMintTransaction(transactionSignature.signature)

}catch(err){

console.log(err)

}

}

const transferTokenToAssociateAccountHandler = async () =>{

try{

const tokensToMint = 1

const fromCustomTokenAccountPubkey = '8M8HtFqrMyfiVfvzFQPGb8TWRWZEGFxbFakeKaC7eBEz' //associated account's public key of the connected wallet

const toAssociatedAccountPubkey = 'EfhdzcbMiAToWYke12ZqN8PmYmyEgRWdFaSKBEhxXYir' //associated account's public key of the receiver's wallet

const transactionSignature = await transferCustomToken(provider, connection, tokensToMint, new PublicKey(fromCustomTokenAccountPubkey), new PublicKey(toAssociatedAccountPubkey))

setTokenTransaction(transactionSignature.signature)

}catch(err){

console.log(err)

}

}

const connectToWallet = () =>{

if(!provider && window.solana){

setProvider(window.solana)

}

if(!provider){

console.log("No provider found")

return

}

if(provider && !provider.isConnected){

provider.connect()

}

}

useEffect(() => {

if (provider) {

provider.on("connect", async() => {

console.log("wallet got connected")

setProviderPub(provider.publicKey)

});

provider.on("disconnect", () => {

console.log("Disconnected from wallet");

});

}

}, \[provider\]);

useEffect(() => {

if ("solana" in window && !provider) {

console.log("Phantom wallet present")

setProvider(window.solana)

}

},\[\])

return (





 {providerPubKey ? 'Connected' : 'Connect'} to wallet {providerPubKey ? (providerPubKey).toBase58() : ""}

 {mintSignature ? `Minted new token, signature: ${mintSignature}`: 'Mint New Token'} 

 {tokenSignature ? `Token transferred, signature: ${tokenSignature}`:'Transfer Token' } 





);

}

export default App;

4. To install the required dependencies:  “npm install” from the root folder of the project.

     5. To run the project : “npm run start” from the root folder of the project.

     6. Navigate to[ http://localhost:3000](http://localhost:3000) 

     7. Click on the 'Connect to wallet' button to connect with the wallet.

8. Once connected, click on "Mint new token" to mint a new token to the given associated account.
## What next?
__What can you build taking this quest as a base ?__

1. Distribute your own token to the associated accounts via minting.
2. You can increase the total supply of your custom tokens.
3. Incentivise the user if they use your services with your custom tokens.