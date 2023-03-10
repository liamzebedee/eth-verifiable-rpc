eth-verifiable-rpc
==================

Want your RPC node to never lie to you? Now you can check, using the power of storage proofs, right from the comfort of `ethers.js`.

This implements a verifiable RPC provider. Contract calls are executed in a local EVM, which dynamically loads ALL contract/storage from a remote RPC node, **securely**, using storage proofs and the `eth_getProof` API.

It means you can query any contract on Ethereum mainnet, and it is locally loaded and executed securely. 

See [this specification](https://hackmd.io/wAfo9dm9S0iMkaXWRIsYUw) for a good background on the underlying tech.

Built for [Dappnet ENS lookups](https://github.com/gliss-co/dappnet-features/issues/9).

❗️ This library is art, and is only tested for ENS lookups right now. ❗️

## Usage.

```sh
npm i verifiable-eth-rpc
```

```js
import { ethers } from "ethers";
import { VerifiedProvider } from "verifiable-eth-rpc";

async function run() {
  const baseProvider = new ethers.providers.InfuraProvider()
  const verifiedProvider = await VerifiedProvider.create(baseProvider)

  const ens = new ethers.Contract(
      "0xd3ddccdd3b25a8a7423b5bee360a42146eb4baf3", // ENS Registry
      [
          "function contenthash(bytes32 node) external view returns (bytes memory)", 
      ],
      verifiedProvider
  )

  const res = await ens.contenthash("0xe6ae31d630cc7a8279c0f1c7cbe6e7064814c47d1785fa2703d9ae511ee2be0c")
  console.log(res)
}
```

## Benchmarks.

For resolving an ENS name, it takes 9s on my slow Aussie internet. Subsequent lookups don't fetch contracts, since immutable code can be cached! It gives a roughly 50% speedup.

Potential idea - prefetch state leaves in a batch by calling `eth_getAccessList`.

```
base) ➜  eth-verifiable-rpc git:(master) ✗ npx ts-node src/ethers-demo.ts
#5 lookup: 9.085s
#4 lookup: 4.870s
#3 lookup: 3.536s
#2 lookup: 3.465s
#1 lookup: 3.335s
#0 lookup: 3.493s
```

## FAQ's.

**What's the trust model?**

This relies on a provider for the latest Ethereum block. So you can use it to "wrap" Infura with verifiable contract calls, or use it with a consensus layer node (beacon chain client) like Helios.

**Can this run in-browser?**

Yes.

**Is this a light node?**

Nope! Even lighter. I chose the term "verifiable RPC" because it conveys better how this works - it's simply a library. It doesn't store any data.

This client does not "track" the blockchain, so doesn't meet the standard of a long-running process to be called a "node". 

**How does this compare with Helios?**

Helios implements this same tech. Honestly, it wasn't ready when I built it. But it seems like they're nearly there lol. Either way, this is useful code.

## Roadmap.

If you wanna build something ambitious, here's where you can take this:

 - [ ] Generate ZK proofs of `eth_call` lookups based on this library. This would mean users don't run the EVM, they just verify a proof, in O(1) network lookups instead of O(N). This could be done using [Kakarot](https://github.com/sayajin-labs/kakarot) and proving using [Giza](https://github.com/maxgillett/giza)/[Sandstorm](https://github.com/andrewmilson/sandstorm).

## Demo.

```js
(base) ➜  verifiable-rpc git:(master) ✗ DEBUG=ethjs,evm:*:*,evm:*,evm npm run start

> verifiable-rpc@1.0.0 start
> ts-node src/

  evm Update fromAccount (caller) nonce (-> 1)) +0ms
  evm ---------------------------------------------------------------------------------------------------- +1ms
  evm message checkpoint +0ms
  evm New message caller=0x0000000000000000000000000000000000000000 gasLimit=115792089237316195423570985008687907853269984665640564039457584007913129639935 to=0x6b175474e89094c44da98b954eedeac495271d0f value=0 delegatecall=no +0ms
  evm Message CALL execution (to: 0x6b175474e89094c44da98b954eedeac495271d0f) +0ms
  evm Reduced sender (0x0000000000000000000000000000000000000000) balance (-> 0) +0ms
  evm Added toAccount (0x6b175474e89094c44da98b954eedeac495271d0f) balance (-> 0) +1s
  evm Start bytecode processing... +2s
  evm:ops:PUSH1 {"pc":0,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":[],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":2,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x80"],"depth":0} +0ms
  evm:ops:MSTORE {"pc":4,"op":"MSTORE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x80","0x40"],"depth":0} +0ms
  evm:ops:CALLVALUE {"pc":5,"op":"CALLVALUE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":[],"depth":0} +0ms
  evm:ops:DUP1 {"pc":6,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x0"],"depth":0} +0ms
  evm:ops:ISZERO {"pc":7,"op":"ISZERO","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x0","0x0"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":8,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x0","0x1"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":11,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x0","0x1","0x10"],"depth":0} +0ms
  evm:ops:JUMPDEST {"pc":16,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x0"],"depth":0} +0ms
  evm:ops:POP {"pc":17,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x0"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":18,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":[],"depth":0} +13ms
  evm:ops:CALLDATASIZE {"pc":20,"op":"CALLDATASIZE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x4"],"depth":0} +0ms
  evm:ops:LT {"pc":21,"op":"LT","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x4","0x24"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":22,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x0"],"depth":0} +12ms
  evm:ops:JUMPI {"pc":25,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x0","0x142"],"depth":0} +12ms
  evm:ops:PUSH1 {"pc":26,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":[],"depth":0} +2ms
  evm:ops:CALLDATALOAD {"pc":28,"op":"CALLDATALOAD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x0"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":29,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231000000000000000000000000d8da6bf26964af9d7eed9e03e53415d3"],"depth":0} +0ms
  evm:ops:SHR {"pc":31,"op":"SHR","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231000000000000000000000000d8da6bf26964af9d7eed9e03e53415d3","0xe0"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":32,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +13ms
  evm:ops:PUSH4 {"pc":33,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +0ms
  evm:ops:GT {"pc":38,"op":"GT","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x7ecebe00"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":39,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x1"],"depth":0} +2ms
  evm:ops:JUMPI {"pc":42,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x1","0xb8"],"depth":0} +2ms
  evm:ops:JUMPDEST {"pc":184,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x70a08231"],"depth":0} +13ms
  evm:ops:DUP1 {"pc":185,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +1ms
  evm:ops:PUSH4 {"pc":186,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +1ms
  evm:ops:GT {"pc":191,"op":"GT","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x313ce567"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":192,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":195,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x10a"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":196,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +0ms
  evm:ops:PUSH4 {"pc":197,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +0ms
  evm:ops:EQ {"pc":202,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x313ce567"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":203,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":206,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x2f2"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":207,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +0ms
  evm:ops:PUSH4 {"pc":208,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +0ms
  evm:ops:EQ {"pc":213,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x3644e515"],"depth":0} +1ms
  evm:ops:PUSH2 {"pc":214,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +1ms
  evm:ops:JUMPI {"pc":217,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x316"],"depth":0} +1ms
  evm:ops:DUP1 {"pc":218,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +1ms
  evm:ops:PUSH4 {"pc":219,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +1ms
  evm:ops:EQ {"pc":224,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x40c10f19"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":225,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":228,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x334"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":229,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +0ms
  evm:ops:PUSH4 {"pc":230,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +0ms
  evm:ops:EQ {"pc":235,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x54fd4d50"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":236,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":239,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x382"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":240,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +0ms
  evm:ops:PUSH4 {"pc":241,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +0ms
  evm:ops:EQ {"pc":246,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x65fae35e"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":247,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x0"],"depth":0} +1ms
  evm:ops:JUMPI {"pc":250,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x0","0x405"],"depth":0} +1ms
  evm:ops:DUP1 {"pc":251,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +1ms
  evm:ops:PUSH4 {"pc":252,"op":"PUSH4","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231"],"depth":0} +1ms
  evm:ops:EQ {"pc":257,"op":"EQ","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x70a08231","0x70a08231"],"depth":0} +1ms
  evm:ops:PUSH2 {"pc":258,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x1"],"depth":0} +0ms
  evm:ops:JUMPI {"pc":261,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x1","0x449"],"depth":0} +0ms
  evm:ops:JUMPDEST {"pc":1097,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x70a08231"],"depth":0} +2ms
  evm:ops:PUSH2 {"pc":1098,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":1101,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b"],"depth":0} +3ms
  evm:ops:DUP1 {"pc":1103,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4"],"depth":0} +0ms
  evm:ops:CALLDATASIZE {"pc":1104,"op":"CALLDATASIZE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0x4","0x4"],"depth":0} +4ms
  evm:ops:SUB {"pc":1105,"op":"SUB","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x4","0x24"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":1106,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20"],"depth":0} +0ms
  evm:ops:DUP2 {"pc":1108,"op":"DUP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20","0x20"],"depth":0} +0ms
  evm:ops:LT {"pc":1109,"op":"LT","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20","0x20","0x20"],"depth":0} +5ms
  evm:ops:ISZERO {"pc":1110,"op":"ISZERO","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20","0x0"],"depth":0} +17ms
  evm:ops:PUSH2 {"pc":1111,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20","0x1"],"depth":0} +1ms
  evm:ops:JUMPI {"pc":1114,"op":"JUMPI","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0xa","stack":["0x70a08231","0x48b","0x4","0x20","0x1","0x45f"],"depth":0} +1ms
  evm:ops:JUMPDEST {"pc":1119,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x70a08231","0x48b","0x4","0x20"],"depth":0} +1ms
  evm:ops:DUP2 {"pc":1120,"op":"DUP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20"],"depth":0} +0ms
  evm:ops:ADD {"pc":1121,"op":"ADD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x20","0x4"],"depth":0} +0ms
  evm:ops:SWAP1 {"pc":1122,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4","0x24"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":1123,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4"],"depth":0} +1ms
  evm:ops:DUP1 {"pc":1124,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x4"],"depth":0} +0ms
  evm:ops:CALLDATALOAD {"pc":1125,"op":"CALLDATALOAD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x4","0x4"],"depth":0} +4ms
  evm:ops:PUSH20 {"pc":1126,"op":"PUSH20","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:AND {"pc":1147,"op":"AND","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0xffffffffffffffffffffffffffffffffffffffff"],"depth":0} +0ms
  evm:ops:SWAP1 {"pc":1148,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":1149,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x4"],"depth":0} +1ms
  evm:ops:ADD {"pc":1151,"op":"ADD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x4","0x20"],"depth":0} +0ms
  evm:ops:SWAP1 {"pc":1152,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x24"],"depth":0} +0ms
  evm:ops:SWAP3 {"pc":1153,"op":"SWAP3","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x24","0x4","0x24","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:SWAP2 {"pc":1154,"op":"SWAP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x4","0x24","0x24"],"depth":0} +0ms
  evm:ops:SWAP1 {"pc":1155,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x24","0x24","0x4"],"depth":0} +0ms
  evm:ops:POP {"pc":1156,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x24","0x4","0x24"],"depth":0} +16ms
  evm:ops:POP {"pc":1157,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x24","0x4"],"depth":0} +0ms
  evm:ops:POP {"pc":1158,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x24"],"depth":0} +0ms
  evm:ops:PUSH2 {"pc":1159,"op":"PUSH2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:JUMP {"pc":1162,"op":"JUMP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x8","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x128f"],"depth":0} +0ms
  evm:ops:JUMPDEST {"pc":4751,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":4752,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":4754,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x2"],"depth":0} +0ms
  evm:ops:MSTORE {"pc":4756,"op":"MSTORE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x2","0x20"],"depth":0} +18ms
  evm:ops:DUP1 {"pc":4757,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +1ms
  evm:ops:PUSH1 {"pc":4758,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +1ms
  evm:ops:MSTORE {"pc":4760,"op":"MSTORE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x0"],"depth":0} +1ms
  evm:ops:PUSH1 {"pc":4761,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":4763,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x40"],"depth":0} +0ms
  evm:ops:SHA3 {"pc":4765,"op":"SHA3","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1e","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x40","0x0"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":4766,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3"],"depth":0} +3ms
  evm:ops:SWAP2 {"pc":4768,"op":"SWAP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xd8da6bf26964af9d7eed9e03e53415d37aa96045","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3","0x0"],"depth":0} +5ms
  evm:ops:POP {"pc":4769,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0x0","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3","0xd8da6bf26964af9d7eed9e03e53415d37aa96045"],"depth":0} +5ms
  evm:ops:SWAP1 {"pc":4770,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x0","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3"],"depth":0} +5ms
  evm:ops:POP {"pc":4771,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3","0x0"],"depth":0} +0ms
  evm:ops:SLOAD {"pc":4772,"op":"SLOAD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x0","stack":["0x70a08231","0x48b","0x85efa08969febcb72bd7c79e3795763c6a77762d27bd830f8777227bf55e86a3"],"depth":0} +0ms
  evm:ops:DUP2 {"pc":4773,"op":"DUP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036"],"depth":0} +1s
  evm:ops:JUMP {"pc":4774,"op":"JUMP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x8","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x48b"],"depth":0} +1s
  evm:ops:JUMPDEST {"pc":1163,"op":"JUMPDEST","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x1","stack":["0x70a08231","0x48b","0x4b47d42c356803515036"],"depth":0} +1s
  evm:ops:PUSH1 {"pc":1164,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036"],"depth":0} +1s
  evm:ops:MLOAD {"pc":1166,"op":"MLOAD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x40"],"depth":0} +0ms
  evm:ops:DUP1 {"pc":1167,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80"],"depth":0} +1s
  evm:ops:DUP3 {"pc":1168,"op":"DUP3","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0x80"],"depth":0} +0ms
  evm:ops:DUP2 {"pc":1169,"op":"DUP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0x80","0x4b47d42c356803515036"],"depth":0} +1ms
  evm:ops:MSTORE {"pc":1170,"op":"MSTORE","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0x80","0x4b47d42c356803515036","0x80"],"depth":0} +1s
  evm:ops:PUSH1 {"pc":1171,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0x80"],"depth":0} +12ms
  evm:ops:ADD {"pc":1173,"op":"ADD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0x80","0x20"],"depth":0} +1s
  evm:ops:SWAP2 {"pc":1174,"op":"SWAP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x4b47d42c356803515036","0x80","0xa0"],"depth":0} +1s
  evm:ops:POP {"pc":1175,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0xa0","0x80","0x4b47d42c356803515036"],"depth":0} +1s
  evm:ops:POP {"pc":1176,"op":"POP","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x2","stack":["0x70a08231","0x48b","0xa0","0x80"],"depth":0} +0ms
  evm:ops:PUSH1 {"pc":1177,"op":"PUSH1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xa0"],"depth":0} +0ms
  evm:ops:MLOAD {"pc":1179,"op":"MLOAD","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xa0","0x40"],"depth":0} +12ms
  evm:ops:DUP1 {"pc":1180,"op":"DUP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xa0","0x80"],"depth":0} +11ms
  evm:ops:SWAP2 {"pc":1181,"op":"SWAP2","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0xa0","0x80","0x80"],"depth":0} +0ms
  evm:ops:SUB {"pc":1182,"op":"SUB","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x80","0x80","0xa0"],"depth":0} +1s
  evm:ops:SWAP1 {"pc":1183,"op":"SWAP1","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x3","stack":["0x70a08231","0x48b","0x80","0x20"],"depth":0} +1s
  evm:ops:RETURN {"pc":1184,"op":"RETURN","gas":"0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","gasCost":"0x0","stack":["0x70a08231","0x48b","0x20","0x80"],"depth":0} +0ms
  evm Received message execResult: [ gasUsed=0 exceptionError=none returnValue=0x000000000000000000000000000000000000000000004b47d4… gasRefund=0 ] +1s
  evm message checkpoint committed +1ms
undefined
Returned: 000000000000000000000000000000000000000000004b47d42c356803515036
gasUsed: 0
```

