# Free Retrieval via IPFS

Proposal: https://github.com/protocol/web3-dev-team/pull/52

Start date: 2021-02-22

Demo: https://youtu.be/oeCaKXbhPls

Code: https://github.com/alanshaw/lotus/pull/1

## Evaluation

<!-- impact of these first projects (on our PMF success metrics), what did we learn, what work is continuing forward? -->

The PoC has demonstrated that it is possible to use a Lotus node to expose deal data over IPFS relatively easy. This was done by leveraging the event system exposed by [`go-fil-markets`](https://github.com/filecoin-project/go-fil-markets).

By registering callbacks for deal lifecycle events we are able to determine when a deal is successfully activated. On activation we know that data for a deal has been transferred, the data has been sealed and the deal has been recorded on chain i.e. the deal was successful.

This is the point at which it is safe to start exposing the deal data to IPFS since at this point the deal cannot fail.

In the PoC the deal data is **copied** to a separate blockstore that is passed to bitswap, which effectively exposes the data to IPFS. The code will check the _current_ "ask" to see if the data for the deal can be retrieved free of charge and only copy if so. The PoC provides no facility for "un-exposing" the deal data if the ask changes as there is currently no event to listen to which would enable it.

Copying the data is problematic for the obvious reason that the miner is storing _two_ copies of the data - the sealed deal data **and** the unsealed data exposed to IPFS. If the "Fast Retrieval" option is enabled on the node then it would actually store _three_ copies of the data: sealed, unsealed and unsealed for IPFS.

Note that only "free" data is copied to the free retrieval blockstore, however the API for setting retrieval asks currently applies to ALL data so this is not helpful.

Originally the code inspected the datastore used by graphsync for the data but it was observed that although this data was available earlier in the deal lifecycle it was removed by the time the deal was activated. This is actually okay because relying on data being available from an event would be brittle and prone to future, hard to debug problems if the `go-fil-markets` code ever decided to remove the data at a time before the event being listened for was fired.

Instead, the code interacts with the `SectorManager` API to unseal the data in the same way as would happen when a regular retrieval happens. The unsealed data is copied into the free retrieval blockstore. It is extra work to unseal the data but if the "Fast Retrieval" option is enabled then it's essentially a straight forward copy.

Coping the deal data to a separate blockstore was the fastest way of building a prototype. A hypothesis was that we could avoid the copy and [use the unsealed data](https://github.com/protocol/web3-dev-team/pull/52#issuecomment-783319835) and a [further suggestion for how to do that was made](https://github.com/alanshaw/lotus/pull/1#issuecomment-786787958) using a [Filestore](https://github.com/ipfs/go-filestore) and a fork of the [`go-car` library](https://github.com/ipld/go-car/pull/24). This looked promising and inroads were made towards it but ultimately time did not allow an implementation to materialize.

It should be noted that avoiding the copy in this way would tie the free retrevial via IPFS capability to the "Fast Retrieval" option meaning that you'd only be able to expose deal data via IPFS if the "Fast Retrieval" option was enabled.

### Implementation Notes

Listening for these events from `go-fil-markets` allow Lotus to enable/disable free retrieval via IPFS via a configuration option. It needs to be decided if this is something that _should_ be configurable of if free retrieval via IPFS should always be possible if the retrieval cost is free.

### Impact

The PoC needs to be presented to interested stakeholders for feedback. Regrettably this has not happened yet.

Metrics from the demo video show that it has received significantly more views than other video content on the Filecoin channel. This may suggest strong community interest in this capability.

<img src="https://raw.githubusercontent.com/protocol/the-spark/main/projects/2021/free-retrieval-via-ipfs/demo-video-impact.png"/>

### Summary of Future Work

If the PoC is to be taken forward, the following work would need to be realized:

* Add event for when retrieval asks change so that data can be exposed/un-exposed as necessary.
* Handle deal expiry/renewal. Expired deals should be un-exposed to IPFS. Renewed deals should remain.
* Add configuration option to allow retrieval via IPFS.
    * Verify assumption that this capability is desired to be configurable.
