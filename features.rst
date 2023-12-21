Features
========

- An efficient, lightweight reimplementation of electrum-server
- Fast synchronization of Dimecoin mainnet from genesis.  Recent
  hardware should synchronize in well under 2 hours.  The fastest
  time to height 5,500,000 (mid December 2023) reported is under 1h.  On
  the same hardware JElectrum would take around 4 days and
  electrum-dime-server probably around 1 month.
- Various configurable means of controlling resource consumption and
  handling bad clients and denial of service attacks.  These include
  maximum connection counts, subscription limits per-connection and
  across all connections, maximum response size, per-session bandwidth
  limits, and session timeouts.
- Minimal resource usage once caught up and serving clients; tracking the
  transaction mempool appears to be the most expensive part.
- Mostly asynchronous processing of new blocks, mempool updates, and
  client requests.  Busy clients should not noticeably impede other
  clients' requests and notifications, nor the processing of incoming
  blocks and mempool updates.
- Daemon failover.  More than one daemon can be specified, and
  ElectrumX-Dime will failover round-robin style if the current one fails
  for any reason.
- Peer discovery protocol removes need for IRC
- Coin abstraction makes compatible altcoin and testnet support easy.

Implementation
==============

ElectrumX-Dime does not do any pruning or throwing away of history.  As long as it is feaible, and it appears
efficiently achievable for the foreseeable future with plain Python, this feature should remain.

The following all play a part in making it efficient as a Python
blockchain indexer:

- aggressive caching and batching of DB writes
- more compact and efficient representation of UTXOs, address index,
  and history.  Electrum-Dime Server stores full transaction hash and
  height for each UTXO, and does the same in its pruned history.  In
  contrast ElectrumX-Dime just stores the transaction number in the linear
  history of transactions.  ElectrumX-Dime can determine block
  height from a simple binary search of tx counts stored on disk.
  ElectrumX-Dime stores historical transaction hashes in a linear array on
  disk.
- placing static append-only metadata indexable by position on disk
  rather than in levelDB.  Currently not possible for histories.
- avoiding unnecessary or redundant computations, such as converting
  address hashes to human-readable ASCII strings with expensive bignum
  arithmetic, and then back again.
- better choice of Python data structures giving lower memory usage as
  well as faster traversal
- leveraging asyncio for asynchronous prefetch of blocks to mostly
  eliminate CPU idling.  As a Python program ElectrumX-Dime is unavoidably
  single-threaded in its essence; we must keep that CPU core busy.

Python's :mod:`asyncio` means ElectrumX-Dime has no (direct) use for threads
and associated complications.