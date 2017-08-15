# An RNG Method for Ethereum


Last night I was thinking about how to do random number generation in
Ethereum. It's a difficult problem, given the fact that the blockchain is, and
must be, public and deterministic. Using a future block hash can work in
certain applications if the properties required of the selected block are kept
secret until several blocks after the block has been mined. Even in this
scenario, though, it'd be possible for a powerful malicious miner or
consortium of miners to just consistently skew the distribution of random
values and affect the overall outcome of an RNG-dependent dapp over the long
haul. This especially becomes a problem under proof-of-stake, as computing
capacity that might have otherwise had to go toward mining is freed up for
block hash mutation. Sophisticated users may notice the skew in such a
scenario, but I expect most would not. Such a tactic might go unnoticed for a
long time.

Relying on simple oracles is another possible solution, and one which I think
carries great merit: hashing together the results from multiple reputable RNG
oracles providing values from hardware-based RNGs would be efficient and
easily audited. However, this solution has the unfortunate property of being
somewhat trustful. It could certainly work, but it would require a good
reputation system and no small amount of vigilance.

In certain scenarios, the best solution may be to leverage user input to
provide entropy. In particular, imagine a game on the blockchain run in rounds
twice the size necessary to prevent malicious miners from rolling back the
results of each round. For the sake of the example, let's say the figure is 24
blocks.

The first 12 blocks would comprise a commitment round: players would submit
hashes of (heavily) salted two-bit values indicating a preference for heads
(1) or tails (0) and a preference for flipping the coin before moving on to
the next player (1) or leaving the coin as it is (0), along with some uniform,
pre-determined amount of ether. The submissions would be sorted by hash during
this round.

The next 12 blocks would comprise a revelation round: as players submit the
their original salted bets, the coin (which starts at "heads") is either
switched to the opposite side ("tails," initially) or left alone in accordance
with the player's preferences. The final state of the coin after running
through all the submissions in the order they were sorted into during the
previous round determines which players win and which ones lose. Those who
have lost receive half their ether back while the other half gets sent to the
winners, along with _all_ the ether of those who did not reveal their bets.
(The requirement for _apparent_ bet uniformity is a simplifying assumption:
otherwise payouts and penalties would have to be calculated based on relative
bet size, which could weaken the incentives involved and would certainly
complicate the example.)

In this case I've stuck with randomizing a single bit value, but the technique
could be generalized out to greater bit lengths. Imagine for example the same
game as described above, only with each player betting on a larger number,
with payouts being determined by the user's proximity to the number resulting
from XORing or SHAing their number together with all the other bet numbers.
This sort of game could even itself be used as an on-chain RNG oracle if the
user requesting an on-blockchain random number places a bet to ensure they are
not observing a game comprised entirely of colluding parties intent on mucking
up the oracle's results.

