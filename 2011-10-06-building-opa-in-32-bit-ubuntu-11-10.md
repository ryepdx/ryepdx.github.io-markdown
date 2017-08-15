# Building Opa in 32-bit Ubuntu 11.10


Heard of [Opa](http://opalang.org/)? It's a full web app stack by MLstate. It
focuses on allowing developers to create highly scalable, highly interactive,
rich web applications as easily as possible.

Here's "Hello World" in Opa:  
`server = Server.one_page_server("Hello", ( -> <>Hello web))`

And here's a chat room in 20 ELOC:  
`import stdlib.themes.bootstrap

type message = {author: string /**The name of the author (arbitrary string)*/  
; text: string /**Content entered by the user*/}

@publish room = Network.cloud("room"): Network.network(message)

user_update(x: message) =  
line =

{x.author}:

{x.text}

  

  
do Dom.transform([#conversation +Dom.scroll_to_bottom(#conversation)

broadcast(author) =  
text = Dom.get_value(#entry)  
message = {~author ~text}  
do Network.broadcast(message, room)  
Dom.clear_value(#entry)

start() =  
author = Random.string(8)

Network.add_callback(user_update, room)}&gt;

broadcast(author)}/&gt;

broadcast(author)}&gt;Post

  

  

  
server = Server.one_page_bundle("Chat",  
[@static_resource_directory("resources")],  
["resources/css.css"], start)  
`

In Opa, the Javascript, HTML, and database models are all generated from the
same code.

I was intrigued, so I decided to give it a spin. Unfortunately my laptop was
running Ubuntu 11.04 on a 32-bit processor. At the time of this writing, the
only binaries available are for 64-bit Linux or OS X. But I decided to try
building from source anyway. Haven't quite got it figured out yet, but I've
made some progress! If anyone can help me out, that'd be great.

My steps so far:

  1. Update Ubuntu to 11.10 if you're running an earlier version. Opa requires OCaml 3.12, but the latest version of OCaml in the 11.04 repositories is only 3.11.
  2. Install dependencies:  
`sudo apt-get install ocaml libssl-ocaml-dev libcryptokit-ocaml-dev libzip-
ocaml-dev libocamlgraph-ocaml-dev ocaml-ulex camlp4 camlp4-extra`

  3. Download and decompress the Opa source package from [Github](https://github.com/MLstate/opalang).  
  
`wget https://github.com/MLstate/opalang/tarball/master  
tar -xzf master`  

  4. Configure and build.  
  
`cd MLstate-opalang*  
./configure -ocamldir /usr/lib/ocaml  
make`

  

  
And that's as far as I've gotten so far. At that point I get the following
error:  
`  
File "buildinfos/buildInfos.ml", line 14, characters 0-3:  
Error: Syntax error  
Command exited with code 2.  
Failure:  
No cache, no trx exe: sorry, cannot build libnet/http/request.ml from
libnet/http/request.trx.  
You may want to re-run with TRX_OVERRIDE set.  
Compilation unsuccessful after building 521 targets (512 cached) in 00:00:04.  
make: *** [all] Error 2

`

I intend to do some digging about to solve this issue. I'll keep this post
updated, so keep checking back.

