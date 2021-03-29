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

* Website. ✅
  * Documentation. ✅
  * Authentication via Github. ✅
  * API key management UI. ✅
* HTTP API. ✅
  * Storage API. ✅
    * Proxy to Pinata. ✅
    * Failover to in-house IPFS. ✅
  * User Management API. ✅
    * Creation/revocation of API keys. ✅
* JS client library for uploading data. ✅

🟠 P1:

* Implement a service to consume pinned CIDs and attempt storage deals with miners. ✅
* Run a miner that accepts deals. ❌
* Run a miner with [Free Retrieval via IPFS](https://github.com/protocol/web3-dev-team/pull/52) enabled. ❌

🟡 P2:

* Pinning service API compatibility (server side). ✅
* Implement a [prototype batching service](https://github.com/protocol/web3-dev-team/pull/60). ✅

## Evaluation

### Highlights

* 👤 **134** Users
    * Almost 1/4 of all hackers at the event.
* 🏗 ~**10%** of ALL projects used our JS library (182 projects total)
    * 1,219 total downloads from npm (Mar 15 → Mar 21)
* 📌 **1,204** NFTs stored
* 💿 **890MB** of NFT data stored
* 🍿 **248** YouTube views https://youtu.be/aNaj9xNF8OU
    * plus zoom attendees (unknown)
* 🎢 **2.02k** unique visitors to https://nft.storage (Mar 18 → Mar 21)
    * 3.41k page views

### Discoveries

* Max NFT upload size is a thing:
    * [Superrare 50MB](https://help.superrare.co/en/articles/4199365-faq-tokenizing-3d-art-on-superrare)
    * Opensea 100MB
    * Mintbase 16MB
    * Rabible 30MB
    * Mintable 3GB
* NFTs stored by users at this hackathon were *less* than **1MB** on average. Although, unlikely ro be representative and more likely to be test data.
* Padding deals with junk data on the client side to make them bigger and more attractive to miners is not possible right now. Data transfer bugs mean transfers are only likely to succeed for small data.

### Fails

* 3 users had CORS issues.
* Multiple users confused by not retaining filenames using `storeBlob`.
    * We should flip the docs to use `storeDirectory` by default.
* `storeDirectory` confuses users thinking they need to make html input element with support for directories which isnt actually what we want 
* Not able to get a Filecoin node with [Free retrieval via IPFS](https://youtu.be/oeCaKXbhPls) up and running during the event.
* Not being able to bundle an JWT means that users have to sign up to nft.storage - artists are unlikely to have a github account so login via other methods should probably be enabled.
* No deals were attempted with miners _during_ the event.

### Remarks

> dietrich 🗓 2021-03-22 at 12:48
> Quote from a team at NFTHack this weekend:
> > nft.storage really saved my butt, literally 2 lines of code”, they talked about how easy it was to integrate into their React and js build env, etc. Had many comments like that during the judging period - that basically it Just Worked™️ and everyone appreciated how easy it made it for them to focus on building their thing. For 40% of the participants it was their first time doing anything with crypto, so having a non-crypto bridge like nft.storage was 💯
