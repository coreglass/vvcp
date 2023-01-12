<template>
  <v-app>
    <!-- <button id="connect-button" ref="connectButton">Initializing...</button> -->
    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { Core } from "@walletconnect/core";
import { Web3Wallet } from "@walletconnect/web3wallet";
import SignClient from "@walletconnect/sign-client";
import { Web3Modal } from "@web3modal/standalone";

// 第三步 创建web3modal
const web3Modal = new Web3Modal({
  projectId: "4e2eb484382a69a977539fb844ab0e57",
  standaloneChains: ["eip155:1"],
  // walletImages: 
  //   {
  //     // rainbow: "/images/rainbow.webp",
  //     metaMask: "",
  //   },
  
  // mobileWallets: [
  // {
  //   id: 'c57ca95b47569778a828d19178114f4db188b89b763c899ba0be274e97267d96',
  //   name: 'metamask',
  //   links: {
  //     native: 'https://metamask.io/',
  //     universal: 'https://metamask.io/',
  //   },
  // },
  // ]
  desktopWallets: [
  {
    id: 'c57ca95b47569778a828d19178114f4db188b89b763c899ba0be274e97267d96',
    name: 'metamask',
    links: {
      native: 'https://metamask.io/',
      universal: 'https://metamask.io/',
    },
   
  },
],
});
//

// 第一步
const signClient = await SignClient.init({ 
  projectId: "4e2eb484382a69a977539fb844ab0e57",
  metadata: {
    name: "Example Dapp",
    description: "Example Dapp",
    url: "#",
    icons: ["https://walletconnect.com/walletconnect-logo.png"],
  },
});
//
// 第四步 指定连接
const { uri, approval } = await signClient.connect({
  requiredNamespaces: {
    eip155: {
      methods: ["eth_sign"],
      chains: ["eip155:1"],
      events: ["accountsChanged"],
    },
  },
});

console.log('uri------', uri)
console.log('approval------', approval)

console.log('web3Modal----', web3Modal)



console.log('signClient----', signClient)

// 第二步 添加监听器 todo



let session = {topic: ''}
if (uri) {
  web3Modal.openModal({ uri });
  console.log('111')
  session = await approval();
  console.log('session---', session)
  // onSessionConnect(session);
  web3Modal.closeModal();
}
//

const result = await signClient.request({
  topic: session.topic,
  chainId: "eip155:1",
  request: {
    // id: 1,
    // jsonrpc: "2.0",
    method: "personal_sign",
    params: [
      "0x1d85568eEAbad713fBB5293B45ea066e552A90De",
      "0x7468697320697320612074657374206d65737361676520746f206265207369676e6564",
    ],
  },
});

console.log('result----', result)

const core = new Core({
  projectId: '4e2eb484382a69a977539fb844ab0e57'
});


const web3wallet = await Web3Wallet.init({
  core, // <- pass the shared `core` instance
  metadata: {
    name: "Demo app",
    description: "Demo Client as Wallet/Peer",
    url: "www.walletconnect.com",
    icons: [],
  },
});

console.log('web3wallet------', web3wallet)

const namespaces = {
  eip155: {
    accounts: ["eip155:1:0x0000000000..., eip155:2:0x0000000000..."],
    methods: ["personal_sign", "eth_sendTransaction"],
    events: ["accountsChanged"],
    extension: [
      {
        accounts: ["eip155:2:0x0000000000..."],
        methods: ["eth_sign"],
        events: [],
      },
    ],
  },
};

web3wallet.on("session_proposal", async (proposal) => {
  const session = await web3wallet.approveSession({
    id: proposal.id,
    namespaces,
  });
  console.log('session-----', session)
});

const test = uri? uri: ''
await web3wallet.core.pairing.pair({ uri:test });

// web3wallet.on("session_request", async(event) => {
//   const { id, method, params } = event.request;

//   await web3wallet.respondSessionRequest({ id, result: response });
// });
</script>
