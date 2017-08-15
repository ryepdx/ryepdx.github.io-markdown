# De-obfuscating a botnet infection


Last night I was working on a PHP website for a client when I stumbled upon
this line of code whitespaced way off the screen on the first line of a few of
our files:
[https://gist.github.com/ryepdx/5016290](https://gist.github.com/ryepdx/5016290
"https://gist.github.com/ryepdx/5016290" )

That looked pretty suspicious to me, so I googled "$GLOBALS['QQOO']" and the
only result that came up was this Pastebin:
[http://pastebin.com/71nwAsj6](http://pastebin.com/71nwAsj6
"http://pastebin.com/71nwAsj6" )

Definitely didn't like the look of that, so I grepped the rest of our files
and found the same code embedded in the same way on four more files. I removed
it from all of them, re-uploaded the cleaned files, and got to work figuring
out what this code was doing.

I copied the code, commented out the call to "$Ill11I1lI" at the end of
"function Q0QQOOQO," and echoed out $Ill11I1lI. This gave me "preg_replace." A
little research revealed that the "/e" flag in preg_replace makes preg_replace
_automatically eval the result._ (This seems like a spectacularly bad design
decision to me, but that's just my opinion.)

So I echoed "Q0QQOOQO(710, 2563)" to see what code they were trying to make
preg_replace run. I got back:
[https://gist.github.com/ryepdx/5016310](https://gist.github.com/ryepdx/5016310
"https://gist.github.com/ryepdx/5016310" )

More preg_replace eval'ing, so I commented out the call to "$Q00Q0QOQQ" and
echoed IlI11lll(721, 2563). I got back a pretty sizable string of code, which
I formatted and then took the liberty of renaming what was pretty obviously a
"get string" function to "get_string." That gave me this code:
[https://gist.github.com/ryepdx/5015935](https://gist.github.com/ryepdx/5015935
"https://gist.github.com/ryepdx/5015935" )

I then did a search for all calls to "get_string" and isolated them to one
line each, then used a little regex-fu to generate this PHP file:
[https://gist.github.com/ryepdx/5015963](https://gist.github.com/ryepdx/5015963
"https://gist.github.com/ryepdx/5015963" )

And then I used the output from that PHP script to create a Python script (I
realize I could have just used PHP, but my Python-fu is stronger) to find and
replace all the calls to "get_string" with their actual returned values:
[https://gist.github.com/ryepdx/5015978](https://gist.github.com/ryepdx/5015978
"https://gist.github.com/ryepdx/5015978" )

That got me a file I could actually read somewhat. I started working my way
down the file, renaming variables like "$QO0OQO" and "$IIl11I" to things like
"$curl" and "$curl_init." After working at it like a crossword puzzle ("What's
a name for the third parameter in fsockopen?") for around half an hour, I was
finally treated to its final, naked, legible form (which I then filled with
helpful comments):
[https://gist.github.com/ryepdx/5016252](https://gist.github.com/ryepdx/5016252
"https://gist.github.com/ryepdx/5016252" )

Basically this code allows the botnet owners to execute arbitrary PHP code on
the infected server. Not a very friendly piece of code!

**TL;DR:** Found obfuscated botnet code on a client's server and spent about an hour and a half making it readable: [https://gist.github.com/ryepdx/5016252](https://gist.github.com/ryepdx/5016252 "https://gist.github.com/ryepdx/5016252" )

