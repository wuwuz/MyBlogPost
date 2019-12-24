# Byzantine Agreement, Made Trivial

Simple but beautiful idea from Silvio Micali. 

### BA

$n$ players in total and $t$ adversaries among those players. Every player holds an input bit $b_i$. After few rounds of communication, they should have an output $out_i$. 

A BA protocol should satisfy:

1. Agreement: for every honest player, they should have the same output.
2. Consistent: If at the beginning, all the honest players hold the same input bit $b$, their output should be $b$.

### Micali's Protocol

It requires on public-secret-key digital signature, a random oracle $H$ (or just a hash function) and a public random string $R$ (finite). It achieves (n, t)-BA with prob 1.

Let $n=3t+1$.

#### Basic Idea

If we have a random 'magic bit' $c$ shared between all players, then the following protocol is (n, t)-BA with prob 1/2.

1. Players broadcast their input bit $b_i$.
2. For the player $i$, he counts the vote for 0 and 1 (including himself).
3. If the votes for 0 $\ge2t+1$, then he outputs 0.
4. Vice versa for 1.
5. If player $i$ cannot determine the output, he outputs $c$.

Consistency is trivial. The protocol will halt in the very first step 3 or 4.

Agreement: 

Notice that a player cannot reach step 3 and 4 at the same time. It means there are one player receives $\ge2t+1$ votes for 0 and another player receives $\ge 2t+1$ votes for 1. The size of the intersect of those two set is $>t$, because $n=3t+1$, which means an honest player misbehaves. 

Therefore, if some players reach step 3, then other players will reach step 5. With prob 1/2, the magic bit $c$ will equal to 0. Vice versa for the case when some players reach step 4.

We can actually run this protocol multiple rounds that we set the inputs of those players as their outputs of last round.

#### Implementation of the 'magic bit'

In the beginning of the round, the players sent the signed message $SIG_i(R,\gamma)$, where $R$ is a random string shared before the protocol and $\gamma$ is the counter for the number of rounds.

Then for every player, he should set the 'magic bit' as the last bit of $\min_{i\in N} H(SIG_i(R,\gamma))$, where $H$ is a random oracle (or just a hash function). If some players refuse to send their message, simply ignore it during the comparison.

As the information of $R$ and $\gamma$ is public, the attacker can only choose to send his message or refuse to send. We consider two situations:

1. $\min_{i\in N} H(SIG_i(R,\gamma))$ belongs to a honest player (prob 2/3). That is good then everybody has the same magic bit. In this case, the honest player will come to an agreement with prob. 1/3 (2/3 * 1/2 =1/3).
2. $\min_{i\in N} H(SIG_i(R,\gamma))$ belongs to an adversary (prob 1/3). The attacker can choose to send his message to some players while refusing to send to others. In this case, the player cannot reach an agreement.

Therefore, we have a BA protocol that can be reach to an agreement with prob. 1/3 in every round.

#### Bound the number of rounds

Firstly, we observe that if those honest players come to an agreement in a round, then in the next round, they will stop in step 3 or 4. And all rounds after that will be the same case.

Therefore, we add two extra 'sub-round' before one round of the above protocol runs.

**Sub-round(0): All players set their magic bits to 0. If they stop at step 3, then output 0 and HALT.**

**Sub-round(1): All players set their magic bits to 1. If they stop at step 1, then output 1 and HALT.**

**Sub-round(2): All players set their magic bit as the above protocol.** 

**If not HALT, else go to next sub-round(0).**

Analysis: if some player output 0 and HALT, then other players will go into step 5 with magic bits to 0, which means they reach an agreement of 0. Then the BA-protocol can be halted because they reach an agreement. Vice versa for output 1 and HALT. If not, sub-round(2) can let the players come to an agreement with prob. 1/3. Therefore, the agreement probability will converge to 1.



