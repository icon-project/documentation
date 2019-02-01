---
title: "Parallel Processing"
excerpt: "Executing transactions in parallel on virtual environment"
---

Normally, transactions should be executed sequentially.
To improve performance, there are two approaches.

* Reducing execution time for each transactions
* Executing transactions in parallel

We are talking about second approache.

The reason, why we execute transactions sequentially, is that previous transactions
may influence the execution of the current transaction.
If previous transactions don't influence the execution, then we can
execute it in parallel with previous ones.

If we know relavant accounts of transactions, we may execute indepdent
transactions in parallel.

## Simple transfer

Simple transfer may change balance of 3 accounts (sender, receiver and treasury).
We may handle treasury deposit after executing whole transactions based on the
results of the transactions. So, only 2 accounts are relavant accounts.

So, we may apply parallel processing on following transactions.

1) Transfer A to B : 100
2) Transfer C to D : 100
3) Transfer B to C : 100

In sequential processing, it takes T (time for simple transfer) x 3.
But, if we executes (1) and (2) in parallel, it takes T x 2.

![Parallel simple transfer](parallel-processing-1.png)

So, we can executes some of transactions in parallel, and it reduces
time to get final state of the transactions in the block.

Same strategy can also be applied to the smart contract.

## Application on a smart contract

A transaction to the smart contract is mapped to method call of the SCORE.
A normal method of SCORE may change the state of any accounts, because
it may call methods of other SCORE. So, basically, a method of SCORE
needs write-permission to all accounts.

But, some methods may require limited permission to accounts.
In that case, we may call independent methods in parallel.
To do this, we need extra information of the method.

With this information, we may make virtual world state for each transaction.
So, when it changes the world, it does affect real world state if it's allowed.
Otherwise, it results in transaction failure.

With this scheme, we can execute two transactions in parallel on following
configuration.

![Parallel execution environment](parallel-processing-2.png)

After executing them, they release locked accounts to let other transactions
can compose own environments locking required accounts.