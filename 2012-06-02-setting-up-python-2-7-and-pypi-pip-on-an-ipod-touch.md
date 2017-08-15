# Setting up Python 2.7 and PyPI (pip) on an iPod Touch


I just wanted to harvest tweets over a period of about a month or two, but I
didn't want to pay for all that processor time on an EC2 instance. I tried
running my Python script on my shared hosting account, but the admins kept
killing the 'screen' containing it. I would have used my desktop computer, but
I can't sleep with it on. And my laptop couldn't provide the uptime I needed.

First thought: [Raspberry Pi](http://www.raspberrypi.org/). But getting one
looked like a rather lengthy process, and I didn't want to have to wait weeks
or months to proceed with my project.

I happened to have a 2nd generation iPod Touch running iOS 4.2.1 that was just
collecting dust. After trying and failing to get Linux on it, after wrangling
with Darwin and reading walkthroughs, I'm ready to share the steps that
eventually got me there. Hopefully they'll help you.

**How I did it (minus the dead ends and false starts)**

  1. Downloaded the latest version of [Redsn0w](http://www.redsn0w.us/).
  2. Downloaded [the iPod Touch 2G 4.2.1 firmware](http://appldnld.apple.com/iPhone4/061-9855.20101122.Lrft6/iPod2,1_4.2.1_8C148_Restore.ipsw). (Thanks to [k1ttyrain](http://www.kittyra1n.com/) for the file.)
  3. Ran Redsn0w and went through the jailbreak process.
  4. Used Cydia to install OpenSSH, wget, BerkeleyDB, and SQLite 3. (I also installed screen so I could run my script and leave it running after ending my SSH session, but you don't have to.)
  5. [SSHed into my iPod](http://guides.macrumors.com/SSH_into_your_iPod_%2528Windows%2529).
  6. Changed my root password with passwd.
  7. `wget http://yangapp.googlecode.com/files/python_2.7.2-5_iphoneos-arm.deb`
  8. `dpkg -i python_2.7.2-5_iphoneos-arm.deb`
  9. `wget --no-check-certificate https://github.com/pypa/virtualenv/raw/develop/virtualenv.py`
  10. `python virtualenv.py env`
  
At that point I used the copy of pip in the env/bin directory that virtualenv
created to install the modules I needed and used the copy of python in the
same env/bin directory to run my script in a screen I detached. Now I've got a
quiet little iPod Touch plugged into the wall, slurping down tweets and saving
them into date-stamped files!

**Edit:** Added mention of SQLite 3 and BerkeleyDB in step four.
