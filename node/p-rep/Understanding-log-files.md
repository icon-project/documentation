---
title: "Understanding log files"
excerpt: "General information about the ICON P-Rep election - https://icon.community/iconsensus/"
---

This document is a guideline for understanding log files as a Public Representative (“P-Rep”) node. P-Reps are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network.

## Intended Audience
We recommend all P-Rep candidates to go through this guideline.

## Pre-requisites
We assume that you have previous knowledge and experience in:

- IT infrastructure management
- Linux or UNIX system administration
- Docker container
- Linux server and docker service troubleshooting

## Operation and Troubleshooting Guide
Using logs, an administrator can figure out the current network situation and fix the problems. 
 
#### loopchain log format

|Date|Time| Process ID | Thread ID | Channel Name | Peer ID| Log Level| File Name (Code Line)|Log Message|
|-----|----|----|----|----|----|----|----|----|
|0729	|19:51:55,580| 480| 4321387392| icon_dex| hx03478b|	INFO|	channel_service.py(70)	|ChannelService : icon_dex, Queue : Channel.icon_dex.7100|
|0729	|20:52:51,803|68150|123145414119424|loopchain_default|	hxb7616f|	WARNING|	block_manager.py(359)|Tried block_height_sync. But failed. The thread is already running|
|0729	|20:32:10,092|61097|4321387392|icon_dex	|hxee4f9e|ERROR|	module_process.py(79)	|module_process.py(79) Process(68469:_ChannelTxReceiverProcess(0x10ba76128)) crash occurred|
|0729	|22:01:49,335|90780|4321387392|icon_dex	|hx86aba2|DEBUG|	block_manager.py(809)	|block_manager:vote_unconfirmed_block (icon_dex/False)|

#### ICONService log format
* ICONService depends on iconcommons package to write logs

|Date |Time |Process|Thread|Level Name|FileName(LineNo)|Tag| Message|
|-----|-----|-----|-----|-----|-----|-----|-----|
|2019-08-07|20:29:35,013	|17074|	140646599931648	|INFO|	icon_inner_service(150)|	IconInnerService|	invoke response with {'txResults': [], 'stateRootHash': 'a7ffc6f8bf1ed76651c14756a061d662f580ff4de43b49fa82d80a4b80f8434a', 'addedTransactions': {}}|




#### Log that confirmed the reset process
- Peer service start complete
```
...
Start Peer Service at port: 7200 start duration(3.181991256)
...
```

- Completion of Genesis Block generation and updated in the blockchain
```
...
Make Genesis Block....
...
ADD BLOCK HEIGHT : 0 , HASH : 97b88d18b0434194463725fefb4921ee06bffdc3903b1ee88cd87e529a291019 , CHANNEL : icon_dex
...
```

- Channel service start complete
```
...
channel_service: init complete channel: icon_dex, state(EvaluateNetwork)
...
```

- Log that confirmed transaction sent successfully 
If the transaction operates successfully, the block that corresponds to that transaction is generated.  
We consider certain transaction are successful if the block has generated after its corresponding transaction has been sent 
```
...
create icx input : {'data': '0x476f6c64203174726f79206f756e6365203a20312c3432362e3130555344', 'stepLimit': '0x989680', 'signature': 'qSmE0OISoWHNEv3L+4KqfTc3GusKWJM/Mk/gw9/F4hpuqAtU/hVgbMsu3CBNprZio5+crmB3xbZChDcZYoWQ7QE=', 'dataType': 'message', 'nid': '0x50', 'from': 'hxb7e4bfb180d1035a74dae3ed9dae2873de0d2a40', 'to': 'hxb7e4bfb180d1035a74dae3ed9dae2873de0d2a40', 'version': '0x3', 'timestamp': '0x58f05b9500d68'}
...
ADD BLOCK HEIGHT : 1 , HASH : f3534687ad62116ef976f34f685b8269eca18badf450350b9932b581a6e5b060, CHANNEL : icon_dex
...
```

#### Log that can check the status of node
- Composition of state machine log 
```
...
Initiating transition from state [STATE NAME A] to state [STATE NAME B]...
Exiting state [STATE NAME A]. Processing callbacks...
Exited state [STATE NAME A]
Entering state [STATE NAME B]. Processing callbacks...
Initiating transition from state [STATE NAME B] to state [STATE NAME C]...
Exiting state [STATE NAME B]. Processing callbacks...
Exited state [STATE NAME B]
...
```

- Log that checks the current state of node which changed from InitComponents → Consensus → BlockHeightSync and confirms current state of node is BlockHeightSync.
``` 
...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex DEBUG core.py(246) Initiating transition from state InitComponents to state Consensus...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex DEBUG core.py(125) Exiting state InitComponents. Processing callbacks...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex INFO core.py(128) Exited state InitComponents
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex DEBUG core.py(118) Entering state Consensus. Processing callbacks...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex DEBUG core.py(246) Initiating transition from state Consensus to state BlockHeightSync...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex DEBUG core.py(125) Exiting state Consensus. Processing callbacks...
0729 19:51:55,792 480 139860382390080 hxb7616f icon_dex INFO core.py(128) Exited state Consensus
0729 19:51:55,793 480 139860382390080 hxb7616f icon_dex DEBUG core.py(118) Entering state BlockHeightSync. Processing callbacks...
...
```


#### Log that contains vote information 

- Composition of block vote log
```
...
Peer vote to : [Height] Hash32([Block Hash]) from [Voter node ID] Peer vote to : 559833 Hash32(0xd6aa69b50c84446b748299ecca2393aaacec6e27e1861c40c08164d942a1aec3) from hx69b9f76becf65050b86efaf7883afa1dd37d14e2
...
```

- Composition of certain node’s Leader complaint vote log
```
...
LeaderVote : 
LeaderVote(rep=[Voter Node ID]), timestamp=[Timestamp], signature=Signature([Signature]), block_height=559901, old_leader=ExternalAddress([Complaint Target Node ID]), new_leader=[New Leader ID]))
...
LeaderVote : 
LeaderVote(rep=ExternalAddress(b'\xb7aoq;e{y@\x1e\x95\x9c\xfcVs\xa5 \x04\xb6\xe6'), timestamp=1564398370451743, signature=Signature(b'|9\x8f\xf3s\xd9\x87\xe5X\x9f\xd7\x87@"4\xef\x9f$\xcb\xa8\xda\x07x\x1e\x86\xadt\x96\x03\xad&_b|5\xaf=\x02\x0e\x16\xb5\xdf\x18\xf789%\x1d\xd5uP1\x81\x84u!\xc4\xd1\xfb\xd8t\xed\xa1\xa8\x01'), block_height=559901, old_leader=ExternalAddress(b'\\\x83-\x81\xd1\x91VHF\x82\x89\x8e!\xbes\x11\x8d\xd5\x95\xbb'), new_leader=ExternalAddress(b'&l\x18\xc9\xcb\xb5\xfc\nE<\x07\xd0\xe0\x95L\x07\xfck\x1f\x0f'))
...
```

- Composition of other node’s Leader complaint vote log
```
...
add_complain complain_leader_id([Complaint Target Node ID]), new_leader_id([New Leader ID]), block_height(559901), peer_id([Voter Node ID]))
add_complain complain_leader_id(ExternalAddress(hx5c832d81d19156484682898e21be73118dd595bb)), new_leader_id(ExternalAddress(hx266c18c9cbb5fc0a453c07d0e0954c07fc6b1f0f)), block_height(559901), peer_id(ExternalAddress(hxacf865e9d5ec9ebd59219599b53ead4c9090d45e))
...
```

#### Log that checks the status of block consensus
- Composition of vote summary log
```
...
Votes : Votes
[State of vote, True/False] : [Number of nodes that have voted]/[Total number of nodes]
Empty : [Number of nodes that have not voted]/[Total number of nodes]
Result : [Number of votes that have received, True: Success, False: Fail, None: Not enough number of votes have received to decide]
Quorum : [ Minimum required number of nodes to pass the vote ]
block height(467458)
block hash(0x329a548d22e589ef4be732c1bcd679e79cfac41f0e750a2ff30c3308b5fdd3e4)
...
```

- Log that checks the status of a block, which it reachedQuorum, and generated successfully
```
...
Votes : Votes
True : 19/20
Empty : 1/20
Result : True
Quorum : 14
block height(467458)
block hash(0x329a548d22e589ef4be732c1bcd679e79cfac41f0e750a2ff30c3308b5fdd3e4)
...
```

- Log that checks the status of a block which it failed to reach the quorum and failed to generate
```
...
Votes : Votes
False : 19/20
Empty : 1/20
Result : False
Quorum : 14
block height(467459)
block hash(0xc7f1c87ad156dc320ea00eb52178405a563d301cf884db0bdafaa458dc8827ef)
...
```

- Log that checks the status of a block which hasn’t received  enough votes for success or failure. 
```
...
Votes : Votes
True : 1/20
Empty : 19/20
Result : None
Quorum : 14
block height(559921)
block hash(0xeba36d86559bc2e63875483f0ccc25425be7b07e1cf8edc7d7e7964818b4a95c)
...
```

#### Leader complaint related log
- Composition of Leader complaint consensus summary log
```
...
complain_result vote_result(Votes
[New Leader ID] : [Number of nodes that voted on same Leader]/[Total number of nodes]
Empty : [Number of nodes that haven’t voted]/[Total number of nodes]
Result : [Number of votes that have received, New Leader ID: Success, False: Fail, None: Not enough number of votes have received to decide]
Quorum : [Minimum required number of nodes to pass the vote]
block height(559901)
old leader([Leader Complaint Target ID]))
...
```

- Leader complaint consensus success
```
...
complain_result vote_result(Votes
ExternalAddress(hx266c18c9cbb5fc0a453c07d0e0954c07fc6b1f0f) : 10/19
Empty : 9/19
Result : ExternalAddress(hx266c18c9cbb5fc0a453c07d0e0954c07fc6b1f0f)
Quorum : 10
block height(559901)
old leader(hx5c832d81d19156484682898e21be73118dd595bb))
...
```

- Leader complaint consensus is still in process.
```
...
complain_result vote_result(Votes
ExternalAddress(hx266c18c9cbb5fc0a453c07d0e0954c07fc6b1f0f) : 8/19
Empty : 11/19
Result : None
Quorum : 10
block height(559901)
old leader(hx5c832d81d19156484682898e21be73118dd595bb))
...
```

#### Log that Message Queue Down and Channel Close 
- Log that explains Message Queue and Disconnection information
```
...
0612 10:07:43,241 417 140184164996928 hx3cb16b WARNING base_connection.py(180) Socket closed when connection was open
0612 10:07:43,241 417 140184164996928 hx3cb16b WARNING connection.py(1360) Disconnected from RabbitMQ at 127.0.0.1:5672 (0): Not specified
0612 10:07:44,307 417 140184164996928 hx3cb16b WARNING connection.py(1360) Disconnected from RabbitMQ at 127.0.0.1:5672 (-1): I/O operation on closed file.
0612 10:07:45,480 417 140183664043776 hx3cb16b WARNING channel.py(1034) Received remote Channel.Close (404): "NOT_FOUND - no queue 'Channel.icon_dex.7100' in vhost '/'" on <Channel number=1 OPEN conn=<SelectConnection OPEN socket=('127.0.0.1', 48402)->('127.0.0.1', 5672) params=<ConnectionParameters host=127.0.0.1 port=5672 virtual_host=/ ssl=False>>>
...
0612 12:10:50,486 498 140275171411776 hx3cb16b icon_dex WARNING connection.py(1360) Disconnected from RabbitMQ at 127.0.0.1:5672 (320): CONNECTION_FORCED - broker forced connection closure with reason 'shutdown'
0612 12:10:50,498 498 140275171411776 hx3cb16b icon_dex ERROR __init__.py(85) Service Stop by: MQ Connection lost.
...
```

- Log that Channel has Closed
```
...
0705 23:31:59,373 8 140098670356224 hx287b4d ERROR callback.py(239) Calling <bound method BlockingChannel._on_channel_closed of <BlockingChannel impl=<Channel number=1 CLOSED conn=<SelectConnection OPEN socket=('127.0.0.1', 25850)->('127.0.0.1', 5672) params=<ConnectionParameters host=127.0.0.1 port=5672 virtual_host=/ ssl=False>>>>> for "1:Channel.Close" failed
...
0705 23:31:59,374 8 140098844378944 hx287b4d WARNING client_async.py(59) Message returned. Probably destination queue does not exists: ReturnedMessage:{'app_id': None,
'body_size': 234,
'cluster_id': None,
'consumer_tag': None,
'content_encoding': None,
'content_type': 'application/python-pickle',
'correlation_id': b'',
'delivery_mode': 2,
'delivery_tag': None,
'exchange': '',
'expiration': None,
'headers': {'FuncName': 'ChannelInnerTask.vote_unconfirmed_block'},
'message_id': None,
'priority': 128,
'redelivered': None,
'reply_to': None,
'routing_key': 'Channel.icon_dex.7100',
'synchronous': False,
'timestamp': None,
'type': 'None',
'user_id': None}
...
```


#### Error Situation etc.
- A log that is generated when level DB has failed to generate
```
...
Fail! Create key value store
Fail to create key value store. path=[Level DB File Path]
...
```

- A log that is generated when peer service stops working. The corresponding reason will be display in [message]
```
...
Service Stop by [message].
...
```

- A log that is generated when the corresponding peer has to vote on higher block than expected. 

```
...
block height validate fail: expected height is 4, voting block info b6c23b0069a9db72075ea3d5b1196ab6468ff3f6dc8fb506fcaa83f98f29d6d2, 5
...
```

- Error log when channel option is not set for [Channel Name]
A log that is generated when channel option of [Channel Name] has not set. 
```
...
peer auth init fail cause : '[Channel Name]'
...
```

- A log that is generated when previous block hash of newly generated block and block hash of last block on the blockchain is different.  
```
...
Block(559830, 880b4ae57a058dcc499cfc9f9aabc01da2eeed98d7018d7d2e61e1a36a7f6bbf, PrevHash(33e152a3c6e24909b763a6a89ddc4ea001de350b7bbc0d9822515bc5c1a3ff47), Expected(796068857b50a0e5c074647ca9d9ab209a5d82305369f7e37d691417ec158536).
...
```

- A log that is generated when CommitState of newly generated block that is given and  CommitState of newly generated block that is verified is different.  
```
...
Block(88273, 5b067ae1ffcc4f396868f5d9a8e8c7a8d2d7ffa9fb7234ea909557512f8e24dd, CommitState(OrderedDict([('icon_dex', '7e21bf8db2b8a21483fd32cd49ae976d6711c40eb39494d0c8947f625cf6c8b5')])), Expected(OrderedDict([('icon_dex', '3bb2a5f8c43756b2de923bdd9c87ce92b567289c1244f7391ae3bdc5376fb8f6')])).
...
```
- A log that is generated when timestamp of newly generated block that is given is expired. The following example explains the log that generated due to the time difference between servers. This log generated when the timestamp of generated block that is given is already in the future compared to the timestamp of newly verified block. 

```
...
Block(8091, 9e6bb2c491fef90c055179c5a97edea3d094f584696548957b58408ef4978eed timestamp(1559964244236603 is invalid. prev_block timestamp(1559964183615384), current timestamp(1559964244218025
...
```

- A log that is generated when given leader information of new block and expected leader information is different. 

```
...
Block(15928, 6d2649e87fbc849f6ee33af9983c62d085d9f6f51dde466c49e77953d7131f29, Leader(hxb3b670d839dbac54e393167dc484c1f479895e25), Expected(hxd7c3dd9941d5c761fb1c6da7310432f56a4fb6ca).
...
```

- A log that is generated when the block height is different from that of the last block of the blockchain
```
...
Block(88273, 5b067ae1ffcc4f396868f5d9a8e8c7a8d2d7ffa9fb7234ea909557512f8e24dd, Height(88273), Expected(6). ...
```

- A log that is generated when a transaction that matches with the Transaction Hash cannot be found
```
...
get_tx_info error : tx_hash([Transaction Hash]) not found
...
```

- A log that is generated when block synchronization fails
```
...
fail block height sync: [Message]
...
```

- A log that is generated when peer-to-peer connection is lost
```
...
__broadcast_run_async fail(<_Rendezvous of RPC that terminated with (StatusCode.UNAVAILABLE, Connect Failed)>)
...
```


