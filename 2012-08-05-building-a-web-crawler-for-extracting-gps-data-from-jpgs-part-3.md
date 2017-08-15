# Building a web crawler for extracting GPS data from JPGs (part 3)


At this point things are going to pick up a bit. I wrote a Python class to
handle reading GPS data from a JPEG using what I learned from walking through
the file with the [EXIF spec](http://exif.org/Exif2-2.PDF) and the [Fetid
Cascade example](http://fetidcascade.com/public/minimal_exif_reader.py) at
hand. I've posted the Python class on GitHub as part of the [gps-exif-
webcrawler repository I started for this project](https://github.com/ryepdx
/gps-exif-webcrawler). I haven't had as much time to spend honing my Python
skills as I used to have, so there are probably a few things about it that
could stand some tightening up. (If you see anything, please drop me a line.
I'm always interested in improving the quality of my code.)

Really though, you need to look at the class in that GitHub repository. I'll
sum it up for you here, but I'm not about to reproduce and explain it line-by-
line. That would be tedious for both of us.

The class was written with a bufferless stream reader in mind. It never
requests data at a position the stream reader has already passed. The only
small difficulty with that approach was the fact that all IFD tags are 10
bytes long, with 4 bytes allocated for the tag's value. If the tag's value is
longer than 4 bytes, it stores an offset pointing to the beginning of the
tag's actual value in the 4 bytes it's allocated. The offset can be anywhere
in the APP1 layer after the IFD tags have ended. I got past that by keeping a
dictionary of tag metadata tuples keyed by offset and then sorting it by its
keys. (Like I said, it was a _small_ difficulty.)

The only thing that bothers me about that approach is my use of tuples. I may
change those to simple objects or dictionaries sometime today. I already had
one incident in development where I got the values out of order while
unpacking them and I imagine that will keep happening if this code changes at
all in the future.

There was also a problem with the fact that ASCII values are stored as null-
terminated strings, with the null included in the tag's value count. Each
character in the string is considered a single value. "Foo" would have a value
count of 4, since there's a null at the end. While that is perfect for C-based
EXIF readers, it meant that each of the strings I read in with Python looked
["l", "i", "k", "e", " ", "t", "h", "i", "s", ".", "\x00"]

So ASCII characters were a definite special case and had to be handled
differently from everything else, which detracted from the code's generic-
ness. But sometimes these things can't be avoided, I guess.

There are also alot of "magic values" in the code. Realistically, I may or may
not change that in the future. It's probably something I'll give a go at some
point because I find it embarassing.

The last thing that really bothers me is the length of the read_gps function.
Minus the pydoc at the beginning, it's 132 lines long. Most of that is caused
by the fact that I only wanted one function in the class modifying the stream.
I figured it would make the code a lot more maintainable if there weren't five
or six methods all changing the stream reader's position. I'm not sure if I
made the right trade-off or not.

Despite these shortcomings, I did comment the code heavily, include pydoc-
style summaries of each function, and include a simple unit test along with a
JPEG I constructed for testing. Hopefully those will be enough to help you
gain an even deeper understanding of EXIF data and help you along in your own
projects!

Next up: putting together a lightweight HTTP stream reader and constructing a
simple webcrawler to grab JPEG URLs from some source, feed them into our
ExifGPSReader, and save the resulting values.

_(To be continued...)_

