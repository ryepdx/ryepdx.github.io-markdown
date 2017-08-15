# Getting the Compaq nx6125 Broadcom Wireless card to work in Ubuntu 11.10


I remember trying to get Ubuntu to work on my Compaq nx6125 my freshman year
of college. The first and biggest hurdle was getting the wireless card to
work. Recently, after a few OS changes, I decided to go back to Ubuntu for
that machine. Imagine my dismay when I discovered the documentation on the
process hadn't changed much since my freshman year! Fortunately with a little
digging I was able to find a much simpler process. If you're trying to get
your Broadcom card to work in Ubuntu 11.10 (and possibly earlier versions, for
all I know), simply run:

`sudo apt-get install firmware-b43-installer`

And that's it. Your wireless card should work.

**Edit:** If you're not using Ubuntu and want to get your Broadcom 43xx wireless card to work in your particular flavor of Linux, you can find more documentation on the subject on [this particular page of the Linux Wireless website](http://wireless.kernel.org/en/users/Drivers/b43). That's where I discovered that handy little bit of command-line wizardry.

