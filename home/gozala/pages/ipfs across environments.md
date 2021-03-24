---
title: IPFS across environments
---

## https://hackmd.io/kKBphQ1xS1uBpwtfnnZfuA
## IPFS across environments

This document attempts to capture current state of IPFS components across supported environments to document known gaps.

### [[IPNS]]

Today [[IPNS]] fails to provide reliable mutability primitive. There is no implementation that works across go, nodejs and web, which makes it impractical.

| Routing | web | go | nodejs |
| ------- | --- |----|--------|
| DHT     | ❌  | ✅ |  ❌    |
| pubsub  | ⚠️  | ⚠️ |  ⚠️    |

> ❌ - Not available
> ✅ - Available
> ⚠️ - Not out of the box

- ❌ DHT in web nodes is impractical as most nodes will be undialable and so will be a web node.
- ❌ DHT implementation in nodejs is buggy, untested and unoptimized ([ipfs/js-ipfs#3469][]). 
- ⚠️ PubSub in web nodes only works in swarms formed around ICE servers that are disjoint from go nodes. In practice creating fragmented networks of ephemeral peers.
- ⚠️ IPNS over pubsub is disabled in go by default (Can be enabled by running daemon with `--enable-namesys-pubsub`). Pubusb itself is also disabled by default ([protocol/web3-dev-team#53][]).
- ⚠️ IPNS over pubsub in nodejs still requires DHT to bootstrap overlay network of participating peers. Lack of proper DHT implementation prevents it. Which could be overcome via delegated routing but it does not work out of the box.
    - pubusb implementation needs update, so it culd query pubsub for the record in parallel to DHT query.

      > It appears that in go querying pubsub tends to outperform DHT queries.

- ✅ Only DHT based implementation in go works out of the box. But it still fails to meet user expectations because:
  1. *Slow* publish and resolution.
  2. Requires continues republishing. (see [ipfs/go-ipfs#4435])
ipns over pubsub flag (--enable-namesys-pubsub) enables pubsub in go-ipfs
### PubSub

Today PubSub fails to provide a solution that works across supported environments out of the box. It is possible to setup a network topology in which praticipating nodes are configured to wokr across environments, but there is planty of hardles. 

| go | web | nodejs |
| -- | --- | ------ |
| ⚠️  | ⚠️ |  ⚠️     |


- ⚠️ PubSub is not enabled by default on **go nodes** ([protocol/web3-dev-team#53][])
  > Can be enabled by starting a daemon with `--enable-pubsub-experiment` flag

- PubSub on **web nodes** works through [js-libp2p-webrtc-star][] 
    - ⚠️ Nodes need a rendezvous point to discover other nodes & exchange their [SDP offers (signaling data)][]. We tell users to use their own rendezvous servers.
    - ⚠️ Different rendezvous servers form disjoint swarms (because peers can not discover each other) so PubSub does not work across them.
        - 🚜 Project flare might fill this gap ([protocol/web3-dev-team#21][])

- ⚠️ Browsers limit number of network connections on same origin (~4) and connection across origins (~6). Given that gossipsub wants 6 connections for optimal behavior it is unclear, if pubsub without circuit-relay is even practical
    - **(Needs verification)** WebRTC is not contstrained by those limitations.

- ⚠️ Since Web nodes do not remain online for long periods of time they will rarely have an overlay network ready.


- ⚠️ PubSub does not work **across web and go nodes**
    - ⚠️ go nodes are not discovered at rendezvous points because they do not participate in WebRTC.
    - (**Needs verification**) With delegated routing web nodes could discover go nodes for pubsub topic
        - ⚠️ But even so they can not dial them because
            - They do not have WebRTC addresses
            - They do not have SSL certificates can not be dialed from HTTPS.
            - There is no way to discover circuit-relay nodes to bridge this gap.
    - 💭 In theory some web nodes could dial some go nodes and bridge the disjoint swarms

- ⚠️ PubSub does not work on **nodejs nodes**
    - **(Needs verification)** Broken DHT implementation makes then unable to discover pubsub peers
        - @vasco could you please clarify why does delegated routing not fill this gap ?
            > @gozala did some testing and can get messages across nodejs and go when both are on the same network, but can't when they are on different networks.
    - 💭 @adin: Could js-libp2p use floodsub option to publish messages to all connections ?
    - 💭 @gozala: Could nodejs nodes bridge web and go nodes ?

### Bitwsap

Today we have functioning bitwsap across supported environments, but it depends on a centralized infrastructure run by PL.

| go | web | nodejs |
| -- | --- | ------ |
| ✅  | ⚠️ |  ⚠️     |

- ✅ Go - Just Works™
- ⚠️ In nodejs peer discovery depends on [delegated content routing][] *(at least until we have functioning DHT)*
    - PL provides an infrastructure for this and nodes are configured to take advantage of it.
- ⚠️ Web nodes depend on PL provided [preload nodes][] that also acting as bootstrap nodes.
    - Added content is cached by preload nodes
    - Because web nodes are connected to them content added by web nodes are available to other web nodes
    - ⚠️ [delegated content routing][] does not work in web nodes, because most peers can not be dialed.
        - 💭 Adding WebRTC addresses to go nodes might help here.
        - 💭 Deploying circuit-relay nodes might also help.

### Additional notes

- signaling server is not libp2p node
- we don't support delegates for put and get in browser.
- Routing Record Name: https://github.com/ipfs/specs/blob/master/IPNS.md#routing-record  
- IPNS over PubSub: https://github.com/ipfs/specs/blob/master/naming/pubsub.md
   - not implemented in web/node
 - we have preconfigured delegates that can be leveraged
    - there is an issue about addressbook overhaul
    - still won't be able to talk to them because no overlap in transport
- Can IPNS over pubsub work across all supported environments ?
    - @gozala is under impression that web nodes can not because:
        - Limits on number of network connections
        - They don't remain online for long periods of time and will rarely have overlay network.
- Is there interop between go and node implementations of [ipns over pubsub][] ?
    - @vasco-santos: yes both [ipns-over-pubsub](https://github.com/ipfs/interop/blob/master/test/ipns-pubsub.js) and in theory [ipns-over-dht](https://github.com/ipfs/interop/pull/47)
    - sort of... node doesn't fully support https://github.com/ipfs/specs/blob/master/naming/pubsub.md
- How does record updates propagate through pubsub ?
    - [Per spec][ipns over pubsub] records are published on `/record/${base64url-unpadded(key)}` channel.
    - Can we add more nodes to the infrastrcuture to make IPNS resolution more effective ?
        - They can't know keys, so do we need a general topic for this ?
        - Will general topic introduce a large overhead ?
- What are the performance, reliability and resource usage characteristics of the pubsub (across supported environments) ?
  > [Our pubsub protocols are useful for low-bandwidth, low-latency, real-time, **unreliable** broadcast](https://github.com/protocol/web3-dev-team/pull/53/files#r583300636)
  - Acconding to pubsub spec optimal outbound connections is 6. That is roughly half of the simultanious connections you can have on web, which makes me question practicality of pubsub in web nodes.
      - Note: I suspect WebRTC could have a lot more connections.
      - Could [Circuit Relay][] multiplex incoming / outgoing connections through a single connection ?
  - Given an **unreliable** nature of pubsub how well fit is it for mutability where consistency is needed ?
    > E.g: Fission relies on DNSLink record as source of truth among replicas. Out of date replicas are required to rebase their changes to be able to update.


[Circuit Relay]:https://github.com/libp2p/specs/blob/master/relay/README.md
[ipns over pubsub]:https://github.com/ipfs/specs/blob/master/naming/pubsub.md
[rendezvous]:https://github.com/libp2p/specs/blob/master/rendezvous/README.md

[Pinata]:https://pinata.cloud/
[DNSLink]:https://docs.ipfs.io/concepts/dnslink/
[IPNS]:https://docs.ipfs.io/concepts/ipns/
[Pinning Services API]:https://ipfs.github.io/pinning-services-api-spec/
[Petname]:https://en.wikipedia.org/wiki/Petname

[ipfs/js-ipfs#3469]:https://github.com/ipfs/js-ipfs/issues/3469#issuecomment-775944675
[protocol/web3-dev-team#53]:https://github.com/protocol/web3-dev-team/pull/53
[ipfs/go-ipfs#4435]:https://github.com/ipfs/go-ipfs/issues/4435
[js-libp2p-webrtc-star]:https://github.com/libp2p/js-libp2p-webrtc-star
[SDP offers (signaling data)]:https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/
[protocol/web3-dev-team#21]:https://github.com/protocol/web3-dev-team/pull/21
[delegated content routing]:https://github.com/libp2p/js-libp2p-delegated-content-routing
[Preload nodes]:https://github.com/protocol/bifrost-infra/blob/master/docs/preload.md