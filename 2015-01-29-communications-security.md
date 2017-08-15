# Communications Security


_This is a copy of a handout I wrote for a talk on PGP to a group of activists
at a dinner on January 29th, 2015. The intent was to provide a supplement to
the practical, but narrow, walkthrough I'd given for those who wanted to dig
deeper into data security in general._

# Basic Security Principles

  1. **Your security chain is only as strong as its weakest link.** Make sure you understand the strengths and weaknesses of each link in your security chain. Take care to maintain the integrity of every link in your security chain. Using encryption will not help you if you composed your message on a compromised machine.
  2. **Respect "the principle of least privilege."** Never give apps permissions they do not need. Never give user accounts on your machine permissions they do not need. Compartmentalize the components of your security chain as much as possible to minimize losses if one or more component is compromised.
  3. **"Security is a process_,_ not a product."[0]**  All systems are broken and will eventually be compromised. Good security habits will help you limit your losses. Staying informed and continuing to learn about security will help you limit your losses. But it is best to assume that every component in your security chain will eventually fail and need to be replaced someday.
  4. **Prefer open source to closed source.** Open source projects may have bugs just as closed source projects may have bugs. But closed source projects may also contain backdoors and intentional security flaws meant to benefit a few powerful actors by allowing them to turn your technology against you. Assume anything closed source is compromised.
  5. **Use the lowest-level tools you can responsibly use.** Getting closer to the metal of the machine leaves fewer potentially exploitable components underneath you, reducing your "attack surface." However, it can also reduce your ability or inclination to use the tool. Only get as close to bare metal as you can comfortably get.
  6. **Know your opponent.** Have an idea of what your "threat model" is so you can choose the right technology to address that threat model. PGP, for example, will hide the content of your messages, but does nothing to hide the metadata. If you need to hide the fact that you are messaging a particular person at particular times, you will need to add a link to your security chain and use something in addition to PGP to hide that data.
  7. **Prefer "battle-tested" technology to new technology.** It takes time and lots of use to ferret out the bugs in any system. The newer a technology, the more likely it is that your communication will end up compromised by an early bug. Even more likely if the technology also hasn't been subjected to at least one security audit.
  8. **Use side-channels to verify the integrity of your primary communication channels.** For example, typing out the first eight characters in your PGP key's fingerprint to someone in an OTR chat after they've downloaded your public key from Keybase.io to make sure they received the correct key. While it's possible that an attacker has compromised all your communication channels and can alter in-transit messages at will, such a scenario becomes less likely as the number of side-channels increases.
  

# Snowden's Chain

  
There is no shortage of technology on the Internet designed to protect your
identity and communication.[1] Some of it is better than others. These are the
technologies that Edward Snowden used to communicate with Laura Poitras and
Glenn Greenwald. They have been battle-tested and are generally considered
solid choices in most situations.

## PGP

  
"PGP" stands for "Pretty Good Privacy." It was created by Phil Zimmerman in
1991\. PGP keys consist of a private portion and public portion. The public
portion of your PGP key, your "public key," should be shared with anyone you
might want to communicate with. The private portion of your PGP key, your
"private key," should never be shared with anyone. You can use your private
key to "sign" data without encrypting it, allowing everyone who has a copy of
your public key to verify that the signed data they've received has not been
changed since it was signed. You can also use your private key to decrypt data
that has been encrypted using your public key.

This is what Edward Snowden used to protect the content of his email
communication with Poitras and Greenwald, and it's been used by many others
for the same purpose. Watch this space, though: PGP has its shortcomings and
there are a few dissenters in the security research community calling for its
ouster.[2][3] Some services, such as Mailvelope and Keybase.io,[4] are trying
to address some of these concerns.

PGP protects content, not metadata.

Interesting less battle-tested alternatives include Axolotl[5] and
Curve25519.[6]

## OTR

  
OTR stands for "Off The Record." This is a method of real-time chatting via
XMPP/Jabber. It requires an account on an XMPP server somewhere (there are
many) and it requires that you and the person you are chatting with are both
using clients that support OTR messaging. Pidgin[7] augmented with the pidgin-
otr plugin[8] is a popular way to use OTR messaging.

OTR protects content, but exposes metadata to the XMPP server you are using.

Interesting less battle-tested alternatives include Tox[9] and TextSecure.[10]

## Tor

  
Tor stands for "The Onion Router." It bounces your Internet traffic between
many different volunteer-provided computers throughout the world in order to
hide the true destination of your traffic from outside observers and to hide
the true source of your traffic from the servers you are accessing. It adds
and removes layers of encryption around your traffic as it bounces around such
that each computer only knows which computer the traffic is coming from and to
which computer it's supposed to go next.

Snowden routed all his traffic through Tor and used an email address he had
set up on RiseUp.net's hidden service to communicate with Poitras and
Greenwald.

Computers on the edge of the Tor network that provide access to "clearnet"
websites, such as Facebook and Gmail, are called "exit nodes." Any data
received from or sent to clearnet websites by computers using Tor must pass
through these exit nodes. If that data is not protected by SSL or another end-
to-end encryption scheme, it can be stored and altered in transit by the exit
node passing it. Accessing clearnet websites via Tor must therefore be done
very carefully.

Tor also provides access to a set of websites called "hidden services." Hidden
services can be accessed without going out into the clearnet, thus keeping
secret the IP addresses of the servers hosting those hidden services and
protecting users from malicious exit nodes.

It is usually best to browse with Javascript off when using Tor. Javascript
can be used to fingerprint your browser and reduce your anonymity. It has also
been used in the past to serve exploits that compromised the identities of Tor
users. The Tor Browser Bundle ships with the NoScript plugin installed and
turned on by default. It is usually best to leave it turned on.

Tor, when used properly, can protect the content and metadata of your Internet
traffic while it is in transit. What you choose to reveal to the server on the
other end can, of course, still compromise you. And while Tor can protect your
anonymity, it can also draw unwanted attention. Tor traffic is apparent to
outside observers, and some intelligence agencies may consider the fact that a
person is using Tor grounds for subjecting that person to additional scrutiny.

One of the popular, though somewhat less battle-tested, alternatives to Tor is
I2P.[11]

## Anonymous VPNs

  
VPN stands for "Virtual Private Network." A VPN essentially allows you to use
another computer's Internet connection as if it were your own. They are a
common way to get around content restrictions in countries like China, Saudi
Arabia, and Australia. They are also quite popular with savvy torrenters
looking to keep their downloading somewhat secret.

Since any additional scrutiny whatsoever could have proven disastrous for him,
Snowden used an anonymous VPN to hide the fact that he was using Tor. This
also provided him with an additional layer of protection in the event that Tor
was somehow compromised, since the attacker would only see the IP address of
the VPN server he was connected to.

Anonymous VPNs make it a point to not keep logs of connections to their
servers. Such logs could prove a problem if they ever fell into hostile hands,
since they would reveal the sources and destinations of all the traffic that
passed through the VPN service. Of course, there is no way of knowing whether
or not a VPN service is being true to its word and not keeping logs, but it is
generally a better bet to use a VPN provider that at least _claims_ to not
keep logs.[12] At least if you're using Tor the only thing such a leak would
reveal is the fact that you were using Tor.

If you want to go a step further to protect your identity, you could also sign
up for the VPN service from a public wifi access point using either a clean
browser or an incognito/private browser window and pay using either Bitcoin or
a prepaid debit card purchased in-person with cash. This allows you to
withhold as close to all identifying information to the VPN service as
possible while also keeping the charge for the VPN service off your credit or
debit card.

The process of connecting to a VPN varies in complexity from provider to
provider. Many will provide a tool to make the process easier. Once you have
connected, make sure that websites now see you at a different IP address and
also that your DNS requests are not leaking your true IP address.[13]

# Greenwald's ~~Chain~~ Link

  
Greenwald initially lost contact with Snowden because he was unable to get PGP
set up. While it's good to be familiar with all the technologies in Snowden's
chain, sometimes one just needs something that will get one up and running
with a minimum of fuss. Fortunately, there's a Linux for that.

## Tails

  
Tails[14] stands for "The Amnesiac Incognito Live System." It's designed to be
run from a DVD, USB drive, or SD card, and it remembers nothing from session
to session. Each time you boot into it, it's a totally clean system. It also
routes all traffic through Tor by default and comes with OTR and PGP tools
preinstalled.

If you choose to run Tails from an SD card or USB drive, make sure that they
are clean and remain clean. Be particularly careful with USB drives, as it was
recently revealed that the USB standard itself contains a flaw that can be
exploited to load malicious firmware onto USB devices.[15] If you use a USB
drive (assuming it's not an IronKey[16]) use a brand new one and only ever
plug it into one computer.

The more you know!

* * *

## Footnotes

  
[0] https://www.schneier.com/essays/archives/2000/04/the_process_of_secur.html

[1] https://www.eff.org/secure-messaging-scorecard

[2] http://blog.cryptographyengineering.com/2014/08/whats-matter-with-pgp.html

[3] http://secushare.org/PGP

[4] I have a couple Keybase.io invites if anyone wants them:
http://keybase.io/ryepdx

[5] Used by Pond (https://pond.imperialviolet.org/) and TextSecure
(https://whispersystems.org/#encrypted_texts)

[6] Used by miniLock (https://minilock.io/)

[7] https://www.pidgin.im/

[8] https://otr.cypherpunks.ca/

[9] https://tox.im/

[10] https://whispersystems.org/#encrypted_texts

[11] https://geti2p.net/en/

[12] There are a couple ways to better your odds, however. Articles on the
subject have been published by Lifehacker (http://lifehacker.com/how-do-i
-know-if-my-vpn-is-trustworthy-508866499) and Deep Dot Web
(http://www.deepdotweb.com/2014/07/08/is-your-vpn-legit-or-shit/), among
others.

[13] What Is My IP (http://www.whatismyip.com/) will tell you what IP address
websites are seeing. DNS Leak Test (https://www.dnsleaktest.com/) and... DNS
Leak Test (http://dnsleak.com/) will both tell you what IP address they see
and whether your DNS requests are coming from the same IP address.

[14] https://tails.boum.org

[15] https://srlabs.de/badusb/

[16] http://www.ironkey.com/

