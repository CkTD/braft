# Overriew
This is a fork from baidu/braft. Some features are added or changed for large-scale distributed storage systems.

- The snapshot protocol is changed to on-demand mode. The original braft from Baidu assumes that the FSM (finite state machine) is volatile, so it maintains at least one full snapshot at all times, which is used to recover the FSM if the system restarts. However, in the real world, the FSM is more likely a persistent storage engine, e.g., RocksDB, so it is unnecessary for Raft to keep snapshots, and the initialization phase doesn't need a snapshot. When Raft finds it needs to send a snapshot to a slow peer, the snapshot is taken on demand and destroyed after use. This change reduces space amplification, which is very helpful for storage systems. Since the snapshot protocol is changed, the old code is broken because their FSM may rely on the snapshot kept in Raft for recovery during the initialization phase. This is also the main reason why I created a new repository.

- A new MergedLogStorage is added for the efficiency of multi-raft. All nodes on a single server can share a single log stream, which is very useful to avoid random writes when you have lots of Raft nodes on one single server and serve a high TPS (transactions per second).

- A new arbiter role is added to reduce costs. Compared to the witness support in the original Braft from Baidu, when the system is functioning normally, the arbiter node only handles heartbeats and does not require append entries. The arbiter have even lower cost(lower cpu and network use) and avoid the problem faced by the witness, which needs to temporarily become the leader.

# Overview
An industrial-grade C++ implementation of [RAFT consensus algorithm](https://raft.github.io/) and [replicated state machine](https://en.wikipedia.org/wiki/State_machine_replication) based on [brpc](https://github.com/brpc/brpc). braft is designed and implemented for scenarios demanding for high workload and low overhead of latency, with the consideration for easy-to-understand concepts so that engineers inside Baidu can build their own distributed systems individually and correctly.

It's widely used inside Baidu to build highly-available systems, such as:
* Storage systems: Key-Value, Block, Object, File ...
* SQL storages: HA MySQL cluster, distributed transactions, NewSQL systems ...
* Meta services: Various master modules, Lock services ...

# Getting Started

* Build [brpc](https://github.com/brpc/brpc/blob/master/docs/cn/getting_started.md) which is the main dependency of braft.

* Compile braft with cmake
  
  ```shell
  $ mkdir bld && cd bld && cmake .. && make
  ```

* Play braft with [examples](./example).

* Installing from vcpkg
  
  You can download and install `braft` using the [vcpkg](https://github.com/Microsoft/vcpkg) dependency manager:
  ```sh
  git clone https://github.com/Microsoft/vcpkg.git
  cd vcpkg
  ./bootstrap-vcpkg.sh
  ./vcpkg integrate install
  ./vcpkg install braft
  ```
  The `braft` port in vcpkg is kept up to date by Microsoft team members and community contributors. If the version is out of date, please [create an issue or pull   request](https://github.com/Microsoft/vcpkg) on the vcpkg repository.

# Docs

* Read [overview](./docs/cn/overview.md) to know what you can do with braft.
* Read [benchmark](./docs/cn/benchmark.md) to have a quick view about performance of braft
* [Build Service based on braft](./docs/cn/server.md)
* [Access Service based on braft](./docs/cn/client.md)
* [Cli tools](./docs/cn/cli.md)
* [Replication Model](./docs/cn/replication.md)
* Consensus protocol:
  * [RAFT](./docs/cn/raft_protocol.md)
  * [Paxos](./docs/cn/paxos_protocol.md)
  * [ZAB](./docs/cn/zab_protocol.md)
  * [QJM](./docs/cn/qjm.md)

# Discussion

* Add Weixin id ***zhengpf__87*** or ***xiongk_2049*** with a verification message '**braft**', then you will be invited into the discussion group. 
