<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />

    <input id="searchId" type="Text" v-on:click="clearInput($event)" value="findAPeer()" />
    <button v-on:click="findAPeer($event)">Find Peer</button>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import Libp2p from "libp2p";
import Websockets from "libp2p-websockets";
import WebRTCStar from "libp2p-webrtc-star";
import { NOISE } from "libp2p-noise";
import Secio from "libp2p-secio";
import Mplex from "libp2p-mplex";
import Boostrap from "libp2p-bootstrap";
import KadDHT from "libp2p-kad-dht";
import GossipSub from "libp2p-gossipsub";
import PeerId from "peer-id";
//import WebRTCDirect from "libp2p-webrtc-direct";

export default {
  name: "App",
  components: {
    HelloWorld,
  },
  methods: {
    findAPeer: findPeer,
    clearInput: clearComponent,
  },
};

var libp2pInstance = null;
function findPeer(event) {
  window.console.log(event);

  const peer = PeerId.createFromB58String(document.getElementById("searchId").value);
  libp2pInstance.peerRouting
    .findPeer(peer)
    .then((result) => {
      window.alert(result.multiaddrs[0].toString());
    });
}

function clearComponent(event) {
  event.srcElement.value = "";
  window.console.log(event);
}

async function createLibp2p() {

  const libp2p = await Libp2p.create({
    dialer: {
      maxParallelDials: 150, // 150 total parallel multiaddr dials
      maxDialsPerPeer: 4, // Allow 4 multiaddrs to be dialed per peer in parallel
      dialTimeout: 10e3, // 10 second dial timeout per peer dial
    },
    addresses: {
      // Add the signaling server address, along with our PeerId to our multiaddrs list
      // libp2p will automatically attempt to dial to the signaling server so that it can
      // receive inbound connections from other peers
      listen: [
        "/ip4/127.0.0.1/tcp/9090/ws/p2p-webrtc-star" //wss:443 not enabled now
        //"/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star", //,
        //"/dns4/wrtc-star2.sjc.dwebops.pub/tcp/443/wss/p2p-webrtc-star",
      ],
    },
    modules: {
      transport: [Websockets, WebRTCStar],//WebRTCDirect, 
      connEncryption: [NOISE, Secio],
      streamMuxer: [Mplex],
      peerDiscovery: [Boostrap],
      dht: KadDHT,
      pubsub: GossipSub,
    },
    config: {
      peerDiscovery: {
        autoDial: true,
        bootstrap: {
          enabled: true,
          list: [
            "/ip4/127.0.0.1/tcp/6031/ws/p2p/16Uiu2HAmJqs2jSBJWhxRSCjnkMM6CReqGu3fEvAqcy5wPBbtoUMH", //Bootnode levantado con pubsub-node
           // "/dns4/ams-1.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLer265NRgSp2LA3dPaeykiS1J6DifTC88f5uVQKNAd",
           // "/dns4/lon-1.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLMeWqB7YGVLJN3pNLQpmmEk35v6wYtsMGLzSr5QBU3",
           // "/dns4/sfo-3.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLPppuBtQSGwKDZT2M73ULpjvfd3aZ6ha4oFGL1KrGM",
           // "/dns4/sgp-1.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLSafTMBsPKadTEgaXctDQVcqN88CNLHXMkTNwMKPnu",
           // "/dns4/nyc-1.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLueR4xBeUbY9WZ9xGUUxunbKWcrNFTDAadQJmocnWm",
           // "/dns4/nyc-2.bootstrap.libp2p.io/tcp/443/wss/p2p/QmSoLV4Bbm51jM9C4gDYZQ9Cy3U6aXMJDAbzgu2fzaDs64",
          ],
        },
        webRTCStar: {
          enabled: true,
        }
      },
      dht: {
        kBucketSize: 20,
        enabled: true,
        clientMode: true,
        randomWalk: {
          enabled: true, // Allows to disable discovery (enabled by default)
          interval: 300e3,
          timeout: 10e3,
        },
      },
      pubsub: {
        enabled: true,
        emitSelf: true,
      },
    },
    metrics: {
      enabled: true,
    },
    peerStore: {
      persistence: true,
      threshold: 1,
    },
  });
  return libp2p;
}

createLibp2p().then((libp2p) => {
  // Listen for new peers

  window.console.log(libp2p);
  libp2pInstance = libp2p;
  window.console.log(`Your PeerID is ${libp2p.peerId.toB58String()}`);

  libp2p.on("peer:discovery", (peerId) => {
    window.console.log(`Found peer ${peerId.toB58String()}`);
  });

  // Listen for new connections to peers
  libp2p.connectionManager.on("peer:connect", (connection) => {
    window.console.log(`Connected to ${connection.remotePeer.toB58String()}`);
  });

  // Listen for peers disconnecting
  libp2p.connectionManager.on("peer:disconnect", (connection) => {
    window.console.log(
      `Disconnected from ${connection.remotePeer.toB58String()}`
    );
  });
  libp2p.start().then(() => {
    window.console.log("Libp2p STARTED");
  });
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
