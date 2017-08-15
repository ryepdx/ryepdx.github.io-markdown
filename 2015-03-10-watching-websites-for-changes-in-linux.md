# Watching Websites for Changes in Linux


(Some of this may also apply to OS X, since it's [Unix-
like](http://en.wikipedia.org/wiki/Unix-like).)

_**Edit:** The general idea does translate well to OS X, but there are a few
key differences which [Maymay](http://maymay.net) has noted [in the
comments](http://ryepdx.com/2015/03/watching-websites-for-changes-in-
linux/#comment-5760)._

I was digging around in my home directory today and came across a script I'd
written last May during one of Github's outages to alert me when it came back
up. Yesterday I turned out a one-liner to watch a website and alert me when it
changed, so I was struck by how my problem-solving approach had changed in the
intervening months as I continued to learn more about the utilities available
on the Unix command line.

First, the script from last May:

    #!/bin/bash

    headers=`curl -G https://github.com/ -I -s | grep HTTP`

    if [[ $headers == *200* ]]  
    then  
     `paplay ~/Downloads/echo_ping.ogg`  
    fi
  
Essentially this script just grabs the response headers from
https://github.com/ and looks for the HTTP code. Then it looks for the string
"200" anywhere within the HTTP code string. If a "200" code is returned, it
uses paplay to play a sound file I'd downloaded from the command line.

A few things stick out to me here:

  1. The "-G" argument to curl is unnecessary, since I'm not passing any data in my request and curl defaults to GET requests anyway. I could have also omitted the "-s" argument, since we're immediately grepping for the "HTTP" line.
  2. I could have used "aplay" or "pacmd" instead of "paplay," though the only difference between them is that "aplay" is more likely to be on a system by default, as I understand it. Regardless, I also went out and downloaded a sound file to play, which took some time. I could have instead used something like "espeak" to make my computer alert me with a robotic voice, which might have saved me the trouble of looking for a unique alert sound for Github's return.
  3. The script doesn't handle periodic checking. In all likelihood, I just popped the script into my [crontab](https://en.wikipedia.org/wiki/Cron). That's a bit of extra hassle in itself.
  4. Once the sound has played, that's it. If I step away from my computer and Github comes back while I was away, I'll have missed the alert sound. Which means I was still checking Github manually every time I came back after stepping away from the computer. So inefficient!
  
If I were to rewrite this today, it'd probably look like this:

    watch -n 300 -g curl -s -I https://github.com/ && espeak "Github is up." && xmessage "Github is up."
  
Here I'm telling "watch" to check the output of "curl -s -I
https://github.com/" every 300 seconds (-n 300) and then exit when it changes
(-g). Upon exit, "espeak" will make my computer tell me verbally that Github
is back, and then once _that's_ done, "xmessage" will pop up a message box
saying the same.

The simplicity is great, and it's always nice to see evidence of one's growing
proficiency with a tool, but "xmessage" really makes this vastly superior to
my old script. The message box it produces is ugly as anything, but it's much
better-suited to the purpose than "notify-send" is. Sometimes you want a
notice to hang around until acknowledged, and "xmessage" fits the bill nicely.

