# `nft.storage` for [NFTHack](https://nfthack.ethglobal.co/)

Project Proposal: https://github.com/protocol/web3-dev-team/pull/62

Work Plan: https://hackmd.io/W1fRyIvbTI6yqqP11CL6_Q

Website Content Plan: https://hackmd.io/d-AoemZJTA2BktWBJKduqA

Diagrams: https://docs.google.com/presentation/d/1tDtoLUeFoHK_Ft-RCm269KoqVqLPPKIFaK2Y3hO7vcg/edit#slide=id.p

Design: https://www.figma.com/file/mxRXqlzGd4MovE85euZtu8/NFT-STORAGE?node-id=0%3A1

Code: https://github.com/ipfs-shipyard/nft.storage

## Summary

`nft.storage` is a single page website and service for storing NFT data on IPFS, backed by Filecoin.

The website is an explainer for what the service is and how to use it but also a place where users can login and generate API keys.

The service is exposed as a HTTP API that accepts file uploads and returns CIDs. The content is pinned to [Pinata](https://pinata.cloud/) and attempts will be made to make a deal with a Filecoin miner to store the data.

🔴 P0 (MVP):

* Website.
  * Documentation.
  * Authentication via Github.
  * API key management UI.
* HTTP API.
  * Storage API.
    * Proxy to Pinata.
    * Failover to in-house IPFS.
  * User Management API.
    * Creation/revocation of API keys.
* JS client library for uploading data.

🟠 P1:

* Implement a service to consume pinned CIDs and attempt storage deals with miners.
* Run a miner that accepts deals.
* Run a miner with [Free Retrieval via IPFS](https://github.com/protocol/web3-dev-team/pull/52) enabled.

🟡 P2:

* Pinning service API compatibility (server side).
* Implement a [prototype batching service](https://github.com/protocol/web3-dev-team/pull/60).
