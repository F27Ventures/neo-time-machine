# neo-time-machine
Travel back in time to a particular block in the NEO Blockchain

# Current Status
This is under rapid development, no current working version yet

# Idea
Provide a service that can replicate the Blockchain at a specified Block X. This would essentially be the state of the Blockchain at Block X allowing you to:
* See all the transactions that have happened so far
* See the current asset balances of all the NEO Addresses
* Query the storage of all smart contracts at that point in time
* Testinvoke smart contracts based on the state of the Blockchain

# Problems
The naive approach is to just create a new Node, and wait for the blocks to sync until you reach Block X.
A fresh installation can take hours if not days to get to the most recent block, and this will get slower over time as blockcount increases. This would be O(m) for m = blocks

A better approach is to keep a store of the downloaded blocks (chain.acc) and then play the block ingestion through until Block X and then pause. However, this would still be O(m), albeit with a much faster play speed than the naive approach.

The optimal speed approach would be to have a compute instance for every single blockheight and it would stay paused at that height so you could just SSH or JSON-RPC that Frozen-Node and immediately be able to access the state.

However, that would mean having a new instance for every single blockheight. Curretly that would mean over 3 Million instances (!).

# Optimisation Goals
Our goals:
- Speed: Must be able to load and query the blockchain state at block X within Y ms (where Y is minimised)
- Cost-Effective: Must be able to do this as blockheight --> Infinity without costing too much

# Proposed Solution
* Store the Blockchain state at blocks 0, 100,000, 200,000, ... in increments of 100,000 as Checkpoints
* This file is an immediately loadable file into neo-python and would take seconds to process not hours or days
* If your desired blockheight is a multiple of 100K, then you're done
* Else, download a partial-chain.acc for the diff from the previous 100K block (max size would be 99,999 blocks) and play-through

Rough calculations:
* For a 4 Million Block blockchain, this would be just 40 servers.
* Assume each server would initially load their starting blockheight and keep that state.
* In order to get to some height from X to X+99,999, they would play forward or backwards. Assume that this file is already in the filesystem (except for the latest Server (which needs to keep updating)). 
* Assuming that you can play forward 0-99,999 (average 50K, worst case 99,999) in <60 seconds.

The problem happens if you have many users wanting to access the same server but at slightly different blockheights (so the server has to keep playing forward and rewinding to get to the desired blockheight).

Possible solution: spawn multiple neo-python instances on the same server - just be careful of RAM usage filling up.

# Problems

* This service still costs the operator money because of server, storage, networking fees.
* JSON-RPC requests generally are light, except for testinvoke/invoke which execute the NEO-VM.

> May need a charging model to make this sustainable.

# Discussion

* How performant do you need it? Can you tolerate an initially slow load time and then fast performance for batch requests at a particular block?
* Can we run this as Google Cloud function (or AWS Lambda) so it only charges ofr execution time?




