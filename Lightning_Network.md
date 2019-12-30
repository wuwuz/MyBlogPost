# Lightning Network

### Basic Problem

Blockchain is too slow.

Two parties want to trade many many times, but require instant confirmation and risk free. 

The lightning network is based on Bitcoin UTXO.

### Case 1: Unilateral Channel

Alice wants to buy a coffee (1 coin) from Bob every day in a week, but they don't want to use blockchain every time. 

**Key Idea:** they can send some incomplete transaction to each other. If they feel necessary, they can simply sign the transaction and close the channel.

**Steps:**

1. Alice signs a **funding tx** (transaction) --- *Open the channel*
   - Input: Her own UTXO (7 coin)
   - Output: 2-of-2 multisig to Alice and Bob (if one player gets both of Alice's sig and Bob's sig, he can spend the money)
2. Bob signs a **refunding tx** and sends to Alice.
   - Input: The funding tx, with Bob's signature (still require Alice's signature to be legal)
   - Output: Alice's address, **but needs to wait after the end of the week**  (time lock)
3. Alice broadcasts the funding tx to the blockchain.
4. In day K, Alice signs a **commitment tx** and sends to Bob (she spends one coin every day)
   - Input: The funding tx, with Alice's signature (still require Bob's signature to be legal)
   - Output : Alice (7-K) coins, Bob K coins.
5. Before the week ends, Bob can simply sign the last commitment tx and broadcast to the blockchain. --- *Close the channel*

**Analysis:**

1. In a normal simulation, Bob can close the channel and collect the coins in any day he wants.  They only need 2 transactions broadcasted to the blockchain.
2. Without the refunding tx, if Bob goes offline, Alice's coins just get frozen forever. Therefore, Alice can refund when the week ends if Bob does not broadcast any commitment tx. The time lock ensures Bob can successfully collect his coin before the week ends even if the refunding tx exists.
3. Disadvantage: 
   1. Alice cannot receive coins from this channel, because Bob can always signs the commitment tx from which he gets the most coins --- the commitment cannot be revoked!
   2. The channel is time-limited.



### Case 2: Bilateral Channel

Alice and Bob want to send money to each other multiple times. 

**Key Idea:** An agreement / a state can be simply represented by $(a, b)$, where Alice gets $a$ coins and Bob gets $b$ coins, while $a+b=F$ (the total amount of funding). To make a trade, they can reach a new agreement / state $(a', b')$ and revoke the old agreement/state.

**Steps:**

1. Open the channel with the funding tx. 

2. They generate new key pairs. Alice and Bob exchange their **revokable commitment txs**. Alice sends the following incomplete tx to Bob : 

   - Input : Funding tx, Alice's signature, (still needs Bob's signature to be legal)
   - Output0 : $a$ coins, Alice's address
   - Output1 : $b$ coins,  condition : (Bob's address &&  40 days after) || (Alice's address && **Bob's special key**)

   Bob sends the mirror tx to Alice.

3. If they want to reach a new agreement $(a', b')$, they firstly generate new key pairs. Then they exchange their new revokable commitment txs. Finally, **they exchange their key pairs (including secret key)** used in the last agreement (**called revoke process**).

4. If anybody wants to close the channel, he/she signs the last commitment tx he/she received and broadcasts it to the blockchain. The counterparty's coin can be drew immediately and his/her own coin can be drew after 40 days.

**Analysis : **

1. Why the above tx is revocable? Consider the case that they reach a new agreement and Bob still wants to broadcast the old transaction. In this case, Alice can get her coin from the output0 of the old tx immediately. In addition, since they have exchanged their special key pairs, Alice holds Bob's special key. Therefore, Alice can get Bob's $b$ coins from the output1 using her own address and Bob's special key before Bob gets the coins. Bob loses all his coins in this case.
2. When they are reaching the new agreement, they needs to exchange their new commitment tx first. At this moment, the new agreement and the old one are both legal, so no offline trade should be confirmed. After that, they exchange their special key pairs and revoke the old agreement, which confirms the trade.
3. To avoid the counterparty to break the rule, one should store all the history key pairs. An technique called ELKREM tree can help with the storage. Every node in the tree is a secret. The left son of one node stores $v(\text{Left Son})=Hash(v||left)$ and vice versa for the right son. If a root of a sub-tree is known, then the whole sub-tree are known because it can be calculated by hash. But we cannot do the opposite way because the hash function is one-way.

### Case 3: Lightning Network 

Alice has a channel with Bob and Bob has a channel with Carol. Now Alice wants to pay Carol $k$ coins.

The naive idea is that Alice sends $k$ coins to Bob and asks Bob to send the coin to Carol. However, the bilateral channel doesn't ensure anything outside the channel. Therefore, Bob can steal the $k$ coins and close the channel.

**Key Idea:** Alice and Bob should reach an agreement $(a, b, k)$, where Bob can spend the $k$ coins only after Carol broadcasts her receipt.

**Steps:**

1. Carol firstly generates a random string $R$ and its hash value $H=Hash(R)$.
2. Carol sends $H$ to Alice.
3. Alice and Bob needs to agree on a new agreement $(a, b, k)$ tx : 
   - Output 0 and output 1 are the same as above.
   - Output2 : $k$ coins, condition (Bob's address and the pre-image of $H$ || Alice's address after 17:00).
4. Bob and Carol also agree on a new agreement $(x, y, k)$ 
   - Output 0 and output 1 are similar as the above one.
   - Output2 : $k$ coins, condition (Carol's address and the pre-image of $H$ || Bob's address after 16:00).
5. Carol broadcasts the receipt $R$ . Two channels can clear the extra outputs.

**Analysis: **

1. Without Carol broadcasts her receipt $R$, Bob cannot spend the $k$ coins from Alice. Therefore, Bob cannot steal the coins. If Carol wants to spend the $k$ coins, she has to broadcast the $R$ to the blockchain and eventually, everybody knows the receipt.
2. The time-lock prevents somebody just goes offline. If the time expires, the $k$ coins can be returned automatically. Between the transition, there should be enough buffer time to ensure the security.



### Reference

https://lightning.network/lightning-network-paper.pdf

https://www.youtube.com/watch?v=Hzv9WuqIzA0

https://ocw.mit.edu/courses/media-arts-and-sciences/mas-s62-cryptocurrency-engineering-and-design-spring-2018/lecture-notes/







