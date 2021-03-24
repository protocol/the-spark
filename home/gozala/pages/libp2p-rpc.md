---
title: libp2p-rpc
---

## 
> Bi-directional RPC over libp2p streams.
https://github.com/carsonfarmer/libp2p-rpc
## Install
### `npm i @nullify/libp2p-rpc`
## **Usage**
### 
```javascript
import create from "@nullify/libp2p-rpc";
// We need a libp2p bundle as derived from js-ipfs, js-libp2p, or the below...
import node from "@nullify/libp2p-bundle";

const run = async () => {
  // Create two peers for communicating
  const node1 = await node({
    multiaddrs: ["/ip4/0.0.0.0/tcp/4007", "/ip4/0.0.0.0/tcp/4008/ws"],
  });
  const node2 = await node();

  // Create a "server" peer (all peers can be both client and peer)
  await create(node1, { ping: () => "pong" });

  // Create a "client" peer and keep reference to dialer function
  const dial = create(node2, { pong: () => "ping" });

  // Start em up!
  await node1.start();
  await node2.start();

  // Get reference to node1's multiaddress
  const addrs = node1.multiaddrs.map(
    (ma) => `${ma.toString()}/p2p/${node1.peerId.toB58String()}`
  );

  // Dial node1 and establish rpc client
  const remote = await dial(addrs[0]);

  // Ping node1/remote and get result
  const result = await remote.ping();
  console.log(result);

  // Spin em' down
  await node1.stop();
  await node2.stop();
  process.exit();
};

run();
```
###
