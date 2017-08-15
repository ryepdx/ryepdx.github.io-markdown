# Ethereum Micropayment Channels


_This is a repost of an answer I gave on the Ethereum StackExchange. It
describes a simple micropayment channel implementation I've had bumping around
in my head for the past week or so. I spent enough time writing it that I
figured I'd share it here as well._

**Q: What are payment channels? Can they be implemented on Ethereum?**

Absolutely. In fact, there's a project currently underway to implement an
Ethereum Lightning Network, which uses micropayment transaction channels,
called [Raiden](https://github.com/brainbot-com/raiden).

For those who aren't already familiar with microtransaction channels, here's a
primer. Feel free to skip the next two paragraphs if you're already familiar
with the mechanism:

In the Bitcoin version, participants lock funds in a multisig "smart contract"
(i.e., one or more Bitcoin transactions) and partially sign a pair of
commitment transactions which are spent from by four timelocked multisig
"commitment" transactions. The spending transactions are set up such that
either party can send what's owed the other party immediately at the cost of
having to wait some number of blocks for their funds to be released. Each time
the state of the channel is updated, previous states are invalidated by
participants exchanging the keys their commitment transactions spend to, thus
allowing the counterparty to steal all the funds in the multisig only if a
participant broadcasts an old commitment transaction.

If you want to understand all of that better, then read the first 29 pages of
[the Lightning Network paper](http://lightning.network/lightning-network-
paper.pdf). It's not important that you do in the context of this answer
though. Suffice it to say, a microtransaction channel requires two parties to
be able to update the channel's state off-chain, settle to the blockchain at
any time, and take all the funds in the channel if the counterparty attempts
to write an old channel state to the blockchain.

In Ethereum, this can be simplified quite a bit. One implementation might
consist of a smart contract which accepts and verifies a signed transaction
command and a nonce. (We can't just use native transactions for signing and
verification like Bitcoin does because Ethereum transactions use an
incrementing nonce to prevent playback attacks.)

Participants would then send ether to the smart contract after creating and
exchanging signed spending transaction commands similar to the ones in the
Bitcoin channels. In particular, Alice would sign a transaction command
sending funds immediately to Bob and locking funds for herself to be released
after some amount of time, and vice versa. The smart contract would contain
the logic that any funds timelocked for the counterparty can be claimed by
producing a transaction command signed by the counterparty with a higher nonce
than the one which the counterparty broadcast.

This requires quite a bit less signing and private key juggling than the
Bitcoin implementation does, and is (IMO) quite a bit easier to understand as
well. How exactly [Raiden](https://github.com/brainbot-com/raiden) does it,
I'm not sure. I haven't gotten around to reading their code just yet. Their
implementation is likely more complex, though, as it looks like they're trying
to really extend the capabilities of the Lightning Network to support more
than just currency transfer. For something as simple as currency transfer via
bidirectional transaction channels, though, the scheme described here should
certainly work.

