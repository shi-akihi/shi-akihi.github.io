---
layout: post
title: CSE Class Notes 2
date: 2026-01-05 19:55 +0800
categories: [Computer Science, Basics]
tags: [SJTU, Computer Science, Notes, System, Introduction]
author: akihi
description: This is the notes of second half of Computer System Engineering course, which goes through the basic idea of Network, Security and Distributed Computing. 
math: true
---
## Network
### Link Layer
**Purpose**: moving data directly from one point to another
1. Physical transmission
	- Serial transmission(no waiting for response)
	- 不同机器上如何进行频率同步？VCO硬件根据输入信号进行对齐
	- 输入大量0对VCO对齐不友好？Manchester Code(0->01, 1->10)
2. Multiplexing
	- Isochronous: TDM，每个连接在建立的时候在链路上被分配一个time slot（适合数据均匀的情况）
	- Asychronous: 通过packet(header)发送，不需要维护连接状态（适合burst的情况）
3. Error handling
	- Error detection: checksum at the end
	- Hamming distance: A $xor$ B中1的个数，通过增加冗余位，拉开合法编码之间的H-distance  
		- e.g: 4-bit -> 7-bit，重新计算纠错位的值，根据不同错误情况可以纠错1-bit error

### Network Layer
**Function**: **forwarding**(data-plane) and **routing**(control-plane)
IP: Best-effort network
**Routing**
1. Link-state
	- advertisements contain its neighbors and its costs
	- advertise to **everyone**(flooding)
	- Dijkstra
	- 每个顶点都有整个网络的所有拓扑结构
	- 问题：flooding开销大
2. Distance-vector
	- advertisements contain its current costs to every node it's aware of
	- advertise to **neighbor** only
	- 自身维护其到每个node的路径，需要经过哪个neighbor
	- update: A受到B->dst的路径，更新A->dst
	- Problem: 存在infinity问题，会认为没有路的地方有路径
	- Split Horizon: 不向neighbor发经过neighbor到达的node的distance
3. Scale up
	- Hierarchy of routing(route between regions): BGP
	- Topological Addressing
	- Path Vector: includes full path in its routing advertisements
		- refuse path including itself in case of circle
		- graph change: discard any path that a neighbor stops advertising

**NAT**: bridge private networks and public networks
- src的IP和port映射到NAT的公网IP和port

**Ethernet Mapping**
- Broadcast network: every frame is delivered to every station
- ARP: IP <-> MAC(ask for IP, get MAC address)
- Attack: ARP spoofing: when one device ask for IP, hacker can reply with a wrong MAC to poison the ARP cache

**BGP**
- Policies: for financial incentives
	- **Customer/Provider**: customer pays
	- **Peers**: provide mutual access to a subset of each's routing tables
		- Usually peers will tell each other about their customers but not their providers

### End2End Layer
**Purpose**: provide transport protocol to application which can ensure some characteristics
1. Assurance with at-least-once
	- set packet with 随机数
	- resend until receive ACK
	- try limit times before returning error
	- Timeout? No fix timer(congestion collapse, 不同设备间相位同步)
	- Adaptive timeout: 统计RTT数据，根据RTT设置Timeout时间
	- NAK: receiver asks for missing items(sender no timer)
2. Assurance with at-most-once
	- receiver side maintains a table of nonce(Problem: duplicate suppression)
	- 幂等性
3. Assurance of data integrity: add checksum in sender
4. Segment and reassembly of long messages
	- segment contains ID for where it fits
	- Solution: hold early packets in buffer until in order/only ACK in order packets
5. Assurance of Jitter Control: delay all arriving segments and cache
6. End2End Performance
	- Fixed Window: receiver tells sender a window size, window advances when all packets in previous window are acked
	- Sliding Window: Sender advances the window by 1 for each in-sequence ACK it receives
		- handling packet loss: timeout之后重传
		- Window Size $\ge$ RTT * bottleneck data rate
	- TCP Congestion Control
		- Window size $\leq$ Receiver Buffer
		- Idea: increase slowly, drop quickly
		- AIMD(Additive increase, multiplicative decrease)
			- too slow start? do multiplicative increase at the beginning
			- leads to efficiency and fairness
		- **Weakness**: packet loss not always caused by congestion, bias against long RTTs(TxRate = W / RTT)

### DNS: Binding IP and Domain Name
- DNS Hierarchy: structure the hostname
- Fault Tolerant: each zone can have multiple name servers
- Three Enhancements
	1. Initial DNS request can go to any name server
	2. Recursion: the name server does all the lookup and return the final IP
	3. Caching: DNS clients and servers keep a cache of names(cache has expire time limit)
- Client can get nearby name server from ISP

**CDN(Content Distribution Network)**
- Network of computers that replicate content across the Internet
- Content providers actively pushes data into the network

## Security

### ROP and CFI
**Security Principles**: least trust

**Stack Buffer Overflow**: inject malicious code to buffer, overwrite return address
- Defense: Data area marked non-executable

**ROP(Return Orientated Programming)**
- Find code gadgets in existed code base(end with ret)
- Push address of gadgets on the stack
- So each gadgets will be connected by return
Defense: 
- Hide the binary file
- ASLR to randomize the code position(fork不会改变偏移量)
- Canary to protect the stack(在ret地址前加入随机数，detect stack overflow, 防止栈被修改)

**CFI: Control Flow Integrity**
Main Idea: pre-determine control flow graph of an application(大部分indirect jump只会去确定的几个位置)
- Use binary rewriting to instrument code with runtime checks, ensure the execution always stays within the statically determined CFG
- Insert a unique bit pattern at every destination and check whether is the same
- Effective against attacks based on illegitimate control-flow transfer

**Blind ROP**
- Stack reading: overwrite a single byte with value X(试出ASLR的偏移量和Canary)
- Find gadgets
	- 先找stop gadget(never crashes, e.g. sleep(10))
	- 修改return address上的address为stop gadget，之后尝试不同return address
	- 目标是找到BROP gadgets(ret前存在pop语句)，埋crash gadget（为了找pop rdi/pop rsi)
	- 通过函数对于不同参数的反应识别关键函数（strcmp：可以改rdx）
		- 找PLT表（记录所有jmp，特征是向下找每一行都能执行）
		- strcmp fingerprint: only two args are readable will not crash
	- 找到write
		- 是否受到字符
- get binary

### Secure Data Flow
**Taint Tracking**: tracking sensitive data flow
- 输入的信息标记为Taint
- 由标记taint的信息计算的到变量都标记为taint
- 给sensitive data标记taint

**Defending Malicious Input**
**TaintCheck**: mark all input data to the computer as tainted, detect when tainted data is used in dangerous ways
- use binary re-writer
- seed - tracker - assert

### Secure Channel
简单加密：
- Replay attack(Solution: add sequence number)
- Reflection attack(Solution: 使用不同的key和seq)

**Diffie-Hellman**
- A pick a, send $g^a \mod p$
- B pick b, send $g^b \mod p$
- $key = g^{ab} \mod p$
- Problem: man-in-the-middle, attacker sends $g^e \mod p$ to both

**RSA Algorithm**: public key & private key
- Certificate: 用CA的私钥签署一条消息包含用户身份和其公钥，接收方用CA的公钥解密

## Distributed and Parallel Computing
### How to scale Computation?
Forward Path: $2*size(W)*B$
Backward Path: calculating the dX and dW, $4*size(W)*B$
#### Single Device Capability
**Single-core Machine**: pipeline / increased clock rate
**Multi-core Machine**: 
- Problem: coherence under memory access hierarchy(cache in different core)
- increase per-core density: more ALUs on a single core(SIMD processing)
	- **Bottleneck: access memory**
- **Roofline Model**: Gflops/s / Flops/Byte(检查算力瓶颈还是访存瓶颈)（斜率为带宽）
- Conditional execution under SIMD
	- Constraint: every SIMD ALUs share the same program counter
	- Solution: masked instruction(execute all the branch)
- SIMT abstraction: threads are grouped in a block, which can synchronize execution

#### System Methods to fully utilize the device
**GPU**
- Basic Building Block: SM(Streaming Multiprocessor)(硬件)
	- core->warp->SM
- SIMT: thread->block->grid(kernel)(硬件抽象)
- **Execution**: 每个SM执行若干个block，一个block有若干个warp，每个周期scheduler会分配warp到各个core上执行（一个warp下所有thread的PC一致）
- **Access Memory**: Register, L1 Cache, L2 Cache, HBM, DRAM
- Why block?
	1. Shared Memory(SRAM)
	2. Block内部线程间同步
- **Memory Hierarchy**

**Optimizing Attention**
- Memory Hierarchy: SRAM > HBM > DRAM
- Goal: Minimize HBM load
	1. naive->$O(NMK)$
	2. change loop order->$O(NMK/BB)$
	3. Tiling: 分块矩阵乘法
		- Read: $O(NMK/BM + NMK/BN)$
		- Write: $O(NM)$
- Overall close to $O(N^2)$

#### System Methods to do computation on multiple devices
**MapReduce Programming Interface**
- Map: Partition input data, group all the intermediate values, pass them to Reduce(pass to Intermediate File)
- Reduce: partition key space, accept a key & a set of values, merge the values
- Intermediate File is on local disk of Map Worker
- Reduce Worker needs to **sort** intermediate files to ensure all occurrences of the same key are grouped
- **Fault Tolerance**
	- Detect: Master sends heartbeat
	- Recovery: re-execution
	- Master Failure: checkpoint(master state is persisted to GFS: worker state, location of IF)
	- Problem: bad records(master sees two failure of the same record, skip!)
- **Optimization for Locality**: schedule map worker on its input chunk server
- **Performance**: spawn backup copies of tasks, first wins
- Problem: not fit for multi-stage execution

**Computation Graph**
- Job Manager, Daemon Process
- Scheduling Rules: prefer executing a vertex near its inputs
- Fault Tolerance: re-execution, run upstream vertices recursively, finish first

### Distributed Training
Parallelization: data parallelism / model parallelism
#### Data Parallelism
- high GPU utilization
	- overlapping data loader on CPU
	- optimize synchronization part
- **Allreduce**
	1. parameter server
		- Problem: huge bandwidth requirement at the PS
	2. Co-located & sharded PS: each server handles the reduce of a portion of data($N/P$)
		- Problem: high fan-in: 同时和所有server连接，降低network性能
	3. De-centralize approach
		- Problem: high communication data($O(N*(P-1))$)
	4. **Ring Allreduce**: only connect to peers, use sharded data
		- each process send one partition, compute it and send to the next process
		- will reorder the computation
		- $O(2*p)$ round / $2*(p-1)*N/p$ communication per machine
#### Model Parallelism
##### Pipeline Parallelism
- Partition on the Layer, each partition is on a device
- Problem: bubbles during forward-backward process
- Techniques: micro-batching(breaking batches into smaller pieces)
	- bubble size depends on partitions / ratio of bubble overhead depends on m
	- Bubble time fraction: $\frac{p-1}{m}$
##### Tensor Parallelism
- Partition the parameters of a layer
- Forward Process: each node send $(P-1)*\frac{di*B}{P}=di*B$（simple forward, 只有all gather部分)
- Backward Process
	- $\nabla_W$ : no need of communication
	- $\nabla_X$ : need Allreduce $2*\frac{di*B}{P}*(P-1)$ (ring reduce)
##### Summary
Tensor Parallelism的communication cost高于Pipeline Parallelism(Pipeline Parallelism只需要传递一个矩阵)
- NVLink: Tensor Parallelism
- RDMA: Pipeline Parallelism

## Raft
**High Level Approach**
- Leader election
- Log replication
- Safety

**Basics**
- **Server States**
	- Leader: receive client requests, replicate logs to followers
	- Follower: only respond to requests from leaders or candidates
- **Term**: logical clock to identify obsolete information
- **Heartbeats & Timeouts**
	- Leader send heartbeats to followers to maintain authority in a term
		- if timeout, follower assumes leader has crashed and starts a new election
- **Election**
	1. change its state to the candidate state
	2. increment current term
	3. vote for itself, send *RequestVote RPC* to all other servers until 
		- receive votes from majority
		- receive heartbeat from a valid leader
		- no one wins(timeout)
	- **Safety**: each server only vote for one per term
	- **Liveness**: use random timeout before retry
- **Log Replication**
	- **Log Structure**: log entry(index, term, command)
	1. send commands to leader
	2. leader appends to its log
	3. leader send RPC to followers(until succeed)
	4. once commited, leader passes command to state machine, returns results and notifies followers to commit
	- **Handle Inconsistency**
		- check if the preceding entries is the same
		- if reject, leader will decrement *nextIndex* and try again(找到leader和follower最早匹配的entry位置并覆盖follower之后的所有entry)
	- **Safety of the Commit Entry**
		- During election, choose candidate with log most likely to contain all committed entries
		- Voter denies to vote if its log is more complete
	- **Commit Rules**
		- the entry must be stored on a majority of servers
		- at least one entries from current term must also be stored on a majority of servers(so it will not be overwritten by new leaders)