---
layout: post
title: CSE Class Notes 1
date: '2025-11-14 22:57:00 +0800'
categories: [Computer Science, Basics]
tags: [SJTU, Computer Science, Notes, System, Introduction]
author: akihi
description: This is the notes of first half of Computer System Engineering course, which mainly covers the concept in file system, especially distributed file systems. Here is part of the material I prepared for the mid-term examination, which is also helpful for studying. 
math: true
---


## Inode-fs
**Unix FS**(top->down): 
1. Symbolic link layer： name files on other disks
	- Soft link: new inode(target file path)
	- Hard link: no new inode
2. Absolute path name layer: root directory
3. Path name layer: Paht name -> inode number (Hierarchy of directories and files, link/unlink)
4. File name layer: file name -> inode number (mapping table in directory)
5. Inode number layer: inode number -> inode (inode table)
6. File layer: block index number in inode -> block number on disk (inode: metadata of file)
7. Block layer: block number -> block data (block: basic unit in file system, 不等同于sector)
	- Super Block: one per system: size, free block number / list(bitmap), metadata

**File name is not a part of file**, but data of a directory and metadata of a file system. 
Hard links are equal. 
No links to directories -> acyclic graph

## Fs-API
**FD** -> fd_table(memory per process)
**File Cursor**: shared / not shared -> file_table(memory)
进程通过fd_table找到对应的file index，再到file_table中寻找对应的inode_num和file_cursor

Delete after open but before close: the inode is not freed until the first process called CLOSE
**Renaming**: 
- LINK(from_name, to_name)
- UNLINK(from_name)

## Dist-sys
Scalable web apps: high request rate, massive data, transparent scale
Routing Methods: Consistent hashing(find the caching server of the related key-value)
**CAP Theorem**: Consistency, Availability, Partition tolerance 2 out of 3
**Distributed Systems Properties**: scalability, performance, fault tolerance, consistency, usability

## Remote Procedure Call
**Other File System Type**: 
- FAT: linked list 1-1 with blocks
	- Follow lists to get blocks
	- Use free list instead of bitmap
	- 随机读性能差，不支持hard link
	- Directory: filename -> file number, size

Filesystem+RPC: a form of distributed system
Idea: build RPC atop of the socket interface
**RPC stub**: 
- Client:
	- put arguments in request
	- send to server
	- wait for response
- Server:
	- wait for message
	- get parameters
	- call the procedure according to the parameters
	- put the results in response
	- send the response to the client
**Message**: Service ID, Service Parameter, using marshal(不能使用指针)
**Message Type**: Standardized encoding, Binary formats
**Automatic Stub Generation**: argument type, function name

**RPC facing failure**: 
- At-leasts-once semantics: retry until getting response
- At-most-once semantics
	- server saves transaction ID(可以实现exactly-once，但是无法确定删除的时间)
	- Idempotent(幂等): run any number of times without harm. 保证at-most-once

## Distributed File System
**Remote Access Model**
## NFS: Network File System
**Goals**: 
- transparency
- recovery from failure: server stateless, client retries
- high performance: caching

System calls -> RPC
- 少open, close
- 返回file handler
- read需要offset

**NFS Clients**
- 对应用程序提供open，close接口
- fd <-> fh (根据file attributes创建本地inode)
- Why no open? 在client端实现相关内容
- file handler: inode number + generation number
- Delete after open: return error(unlike local Unix FS)

**Problem: performance overhead**
- Caching at client(file data, file attribute, pathname bindings)
- Cache coherence
	- close-to-open consistency
		- open: compare last modification time with local cache
		- close: send cached writes to the server
	- read/write consistency
		- NFS guarantee coherence only for certain operations
		- validation
			- compare last modification time, invalidate cached data if remote is more recent
			- always invalidate data after some time

## Key-Value Store
### System Modeling
1. Build an internal abstraction of the system, distill the main components that determine the system behavior
2. Qualitative Model
3. Typically needs to be parameterized(static / input)

Random **write** < Sequential **write**: log-structured file(append only)
**Get** Latency: add index(key -> offset) (e.g. in-memory hash index)
- fat pointer: virtual memory address + offset in a file
Prevent a file from growing forever? Compaction
Range-query? Hash table -> B+Tree(no log-structure, value in leaf node)
- Trade-off: Get, Insert, Update are slow

## Consistency
**Consistency Model**: defines rules for the apparent order and visibility of updates
- Eventual -> Causal -> Sequential -> Linearizability -> Strict

**Eventual Data Consistency**: Eventually all the data becomes the same
- Deterministic sort all posts upon syncing
	- ensure both nodes have the same updates in the log
	- ordered lists of updates at each time
	- reapply sorted updates after the sync
- a total order among operations
	- Problem: **unsynchronized clock** across different devices
	- Ideal order: Causal ordering
- **Lamport Clock**: a logical clock
	1. Each server keeps a clock T
	2. Increment T as real time passes
	3. T = max(T' + 1, T), when receive message from another server

Lamport clock gives a order to every message -> too strong
**Partial Order**: vector lock
- Local lamport clock and cached copies of clocks in other servers
- Modify $T_i=max(T_i,T'_i+1)$ if see T from another server

**Strong Consistency Model**: everything has one copy
- strict consistency: sorted based on issue time

## Consistency under Failure: Atomicity
**Completion-to-issue**: if operation a's completion time notified by the client is before the issue time of b, then b must be observed executed later than a
**Linearizability**: Client can not infer the execution order of concurrent operations

### Implementing linearizability in replicated data storage
**Principle**: one writer rule
- only one process is allowed to do the read/write
- then propagate to other processes

**Primary backup implementation**:
- In order updates: primary must use some seq number: all replicas apply writes in the order of seq number
- Performance issues & reliability issues(raft)

### Atomicity
Ensure a set of operations written to a file is all-or-nothing. 

**Shadow Copy**: copy on write(replace the original file with updates only if all writes is successful)
- Goal: file system APIs are all-or-nothing
- **Journaling**
	1. record changes in journal
	2. commit journal
	3. update
- Mitigation: only protect metadata via journaling
- Drawbacks: multiple clients share the same file

## Realizing atomicity: Logging+Checkpoint
Journaling Drawbacks: 
- everything is written to disk twice
- hard to generalize to multiple files

### Logging for Atomicity
**Idea**: avoid updating the disk state until we can recover it after failure
Transaction and Commit Point: marking atomic units

**Recovery of Commit Log**(redo-only logging)
1. Travel from start to end
2. Re-apply the updates recorded in a complete log entry
**Cons of Redo-only logging**
- All updates must be buffered in memory
- Log file is continuously growing

**Undo Logging**
- Should contain sufficient information to undo uncommited transaction
- Log records
	- records from different transaction may interleave -> need pointer to trace operations from the same TX
- Recovery rules
	1. Travel from end to start
	2. Mark all transaction's log record w/o CMT log and append ABORT log
	3. Undo ABORT log from end to start
	4. Redo CMT log from start to end

### Checkpointing
**Idea**: save the system state into a compact form so that the restore is faster
**Checkpoint in logging**
Observation: uncommited updates are only in page cache
Challenge: maintain correctness
1. wait till no operation in progress
2. write a CKPT to the log
3. Flush page cache
4. Discard all the log records except CKPT record

**Checkpointing in LLM Training**
- Checkpointing must be atomic -> shadow copy
- Reduce fault tolerance overhead(checkpoint overhead + recomputation)

**Checkpoint in Eventual Consistency**
**Challenge**: data is decided by multiple servers, cannot simply use all the record
**De-centralized Approach**: update T is stable if all nodes have seen all updates with time <= T
**Centralized Approach**: 
1. One primary server
	- Assign a total commit order CSN to each write
	- Complete timestamp
	- Any write with a known CSN is stable
2. All stable writes are ordered
3. CSNs are exchanged between servers
Notes：A server asks the primary to assign CSN for all tentative writes

## Before-or-After Atomicity and Serializability
Linearizability: serial execution of read / write -> Serial execution of actions
**Goal**: before-or-after atomicity
**Global Lock**: 
- The action must acquire the global lock before executing and release it after it commits. 
- Trade performance for correctness

**Fine-grained Locking**: each shared data has a lock
- The action must acquire it before accessing it and release it after the data access finishes. 
- Not ensure before-or-after
- **Solution**: acquire all locks first and release them at last

**2PL(Two phase locking)**
- The action must acquire the shared data's lock before accessing it, and once the lock is released, no lock can be held. 

**Serializability**: run actions concurrently, and have it appears as if they can ran sequentially
How to check it?
**View Serializability**: the final written state as well as intermediate reads are the same as in some serial schedule(informal definition)
**Conflict Serializability**
- Two operations conflict if:
	- they operate on the same data object
	- at least one of them is write
	- they belong to different transactions
- the order of its conflicts is the same as the order of conflicts in some sequential schedule
Conflict Serializability > View Serializability

## Serializability, OCC & Transaction
Proof of 2PL: 反证法
2PL Problem: 
- Phantom Problem: Only lock the things you touch is insufficient
- Solutions: Predicate lock; Range locks in a B-tree index or lock entire table

**Deadlock**
Two phase locking is **pessimistic**: before proceed each TX must waiting for conflicting TX to release lock
**Methods**: 
1. Acquire locks in a pre-defined order(must know all the data set before access)
2. Detect deadlock by calculating the conflict graph & abort one TX to break the cycle
3. Using heuristics to pre-abort TXs

**Optimistic concurrency control**
1. Concurrent local processing
	- Reads data in a read set
	- Buffers writes into a write set
2. Validation serializability in critical section
	- compare the data's **version**
3. Commit the results in critical section or abort
**Phase 2 & 3 should execute in a critical section**
- Two-phase locking(not deadlock since read set is known)
- No need for read set lock(validation <=> lock)
	- elements in read set is **neither changed or locked**

Locking Preliminary: atomic compare and swap
- Lock prefix to ensure an instruction is atomically executed on a memory address

## Transaction & Multi-site Transaction
OCC Problems
- False Abort & Live Lock
	- Under high contention, OCC may continuously abort

System Principle behind OCC: 
- **Fast Path & Slow Path**
	- use a fast path to accelerate execution
	- validate that the fast path matches the results
	- if validation fails, go to slow path

RTM(Restricted Transactional Memory)
- xbegin / xend to mark the begin and end of transaction
- if fail, go to fallback
- Problems
	- limited working set
	- external events break the RTM execution

Transaction: an abstraction to manage data to guarantee ACID properties
**Consistency in TX**: defined by applications
**Atomicity is not sufficient for application consistency**

**Read-only or Read-mostly transactions**
**Idea**: multi-versioning concurrency control
- A set of version represents a snapshot of the database
- **Read**: only read from a consistent snapshot at a start time
- **Write**: install a new version of data instead of overwriting the existing one

**Optimize OCC**
1. Concurrent local processing
2. Commit the results in critical section
- **No validation is need**
**Problem: Partial snapshot**
Solution: enforcing write atomicity of snapshot with locking
- get commit timestamp after locking

MVCC ensures no race for reads, but not writes
- Validation: check whether another TX has installed a new snapshot after the commiting TX's start time

**Write skew anomaly**
- Case: R(X), W(Y) & R(Y), W(X)
- Fix: validate the read set in read-write TX

**Multi-site transaction**
Two-phase Commit
- High-layer TX: coordinates the execution of lower-layer TXs
- Low-layer TX: specific reads and writes that execute on a single machine
- Phase 1: 
	- Delay the commitment of lower-layer TXs
	- Lower-layer transactions either abort or tentatively committed
	- Higher-layer transaction evaluate lower situation
- Phase 2:
	- Higher layer decides whether low-layer TXs will commit or abort
	- coordinate the commitment of lower layers
- Idea: the coordinator is responsible for committing all the TXs

## Consistency under Partial Failure
**Goal**: all sites either all commits or all aborts under partial failure
**Multi-site Atomicity**
- Principles
	- Follow the coordinator's decision
	- Log sufficient state to tolerate failures
- Problem: if the coordinator fails, the server cannot know whether to release the lock

**Primary-Backup Model**
- Replicated State Machine
- **View Server**: keeps a table that maintains a sequence of view, manage primary and backup; coordinator make requests to view server asking who is primary
- Rules when facing network partitions
	- Primary must wait for backups to accept each request
		- check if it is in the same view of the primary
	- Non-primary must reject direct coordinator requests
- **Quorum**
	- Confirm a write after writing to at least Qw of replicas
	- Read at least Qr agree on the data or witness value
- Together
	1. One server is responsible for ordering inputs received from clients
	2. After receives an input, sync up to W backups before ACK
	3. If the primary fails, select R backups, and chooses the one with the most up-to-date inputs

### Paxos
**Single-decree Paxos(agree on single value)**
- Challenge: support multiple writers
- Phase 1: 
	1. Leader creates proposal N and send to quorum
	2. Acceptor: 
		- N > any previous proposal: 
			1. replay with the highest past proposal number & value
			2. promise to ignore all IDs < N
		- else: ignore
- Phase 2: 
	1. Leader receives enough promise (majority)
		1. set value to the proposal V, if any accepted value returned, replace V with the returned one
		2. send accept request to quorum with the chosen value V
	2. Acceptor:
		- If promise still holds:
			1. register the value V
			2. send accepted message to Proposer / Learners
		- else: ignore
- Phase 3: 
	1. Learner: respond to the client or take action on the request
- When is the value V chosen? **Leader receives a majority accepted**
- Acceptor fails after sending promise: Nh
- Acceptor fails after receiving accept: Nh, Na, Va
- 用于View Server, but single-decree is not enough

**Multi-Paxos**(value -> log)
Basic Multi-Paxos: correct but insufficient(2 Rounds of RPC)
Solution: 
1. Leader election: prepare message batching, prepare multiple writes at a time
	- Reason: if there is only one writer for most of the time, batching prepare message and the following accept stages will succeed in most cases(multiple writers still work)
Problem: hole in log
