---
title: "Parallel Processing"
excerpt: "Executing transactions in parallel"
---

Various factors affect blockchain performance. One of the major factors is the **execution time** of transactions. In this article, we illustrate our R&D approach to increase blockchain performance by reducing the transaction execution time. 

## Transaction Execution Time 
Transactions Per Second (TPS) is the number of transactions that a network can process in a single second. TPS is a key metric of blockchain performance. 
A transaction is measured from the submission to the network until it is included in the block. In ICON, the number of transactions included in a block are always guaranteed to be executed within one block generation time, so it is a good representation of transaction throughput and it is proportional to the TPS.  

TPS is mainly affected by following factors.
* Travel time of a transaction to the consensus node 
* Communication time among consensus nodes
* Execution time of the transaction 

Travel and communication times are network bound, and there have been continuous attempts to optimize the network protocol in blockchain. Further optimizing the network protocol is not trivial as it involves the risk of breaking the liveness and safety properties of the blockchain network.
On the other hand, reducing the execution time has not been touched yet. If we can reduce the transaction execution time, we can improve the performance of a blockchain without risking network stability.

## Parallel Execution

The most straightforward approach to reduce transaction execution time is using stronger computing power. This is called scale-up or vertical scaling. However, increasing computing power is expensive and has physical limitations. Instead, in modern computing, a more common approach is increasing overall throughput with the parallel execution using multiple cores.

That said, in blockchain, sequential execution of transactions has been the norm, because a blockchain network represents a world state - a collection of accounts and their states, and every consensus node must agree on the state change after a transaction. Since previous transactions can affect the execution of the current transaction, it is safer to execute transactions sequentially.

On closer look, however, some transactions are disjointed or independent, meaning that they do not write on the same state variables during their execution. These independent transactions can be executed in parallel without any conflict with other transactions. In the subsequent sections, we will explain in more detail how this can be implemented in two different cases.


### Coin transfer

Coin transfer can change the balance of at most three accounts at a time: 1) sender, 2) receiver, and 3) treasury accounts.
A treasury is a special account for deposit, and we can process all the deposit requests to the treasury in bulk after executing other transactions. Therefore, only two accounts (sender and receiver) are execution-order sensitive.

Let's take a look at an example of the following transactions in a block in the order of submission.

1) Transfer A to B: 100
2) Transfer C to D: 100
3) Transfer B to C: 100

Let T be the execution time for each coin transfer. 
In sequential processing, total execution time is T x 3.
However, if we execute (1) and (2) in parallel, it takes T x 2. We know we can execute (1) and (2) at the same time, because they do not access the same accounts.  

![Parallel simple transfer](parallel-processing-1.png)

We examined that if we can execute independent transactions in parallel, the total execution time of the transactions in a block can be effectively reduced.
The same approach can be applied to smart contract execution.


### Smart contract

A transaction sent to a smart contract (SCORE) is a method call.
SCORE may change the state of any account, because it can make a call to other SCORE. In general, methods of SCORE need a write-permission to all accounts.
However, each specific method requires limited permission to the accounts. Thus, if we have extra information about which accounts the method needs an access, we can identify independent SCORE methods and execute them in parallel.

With the extra information about the method, we can create a virtual world state (a partial view of the world) for each transaction.
The transaction can only change the world state when the method is designated to do so. Otherwise, any attempt to change the world state beyond its permission will result in a failed transaction.

The diagram below illustrates how the parallel execution of two transactions are managed.

![Parallel execution environment](parallel-processing-2.png)

After executing transactions, each environment releases the locked accounts, then other transactions can create their own execution environments with the required accounts being locked.

To enable parallel processing, SCORE methods should be annotated with the required accounts. Careful design, such as splitting database and code, will enhance the parallelism. 
