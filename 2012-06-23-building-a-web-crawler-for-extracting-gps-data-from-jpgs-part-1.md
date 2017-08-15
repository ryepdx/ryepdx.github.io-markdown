# Building a web crawler for extracting GPS data from JPGs (part 1)


**Motivation:**

A few weeks ago I attended a talk about the ways our mobile devices can
compromise our privacy without us realizing it. One of the things mentioned
was the GPS data automatically embedded in the EXIF data of pictures taken
with GPS-enabled devices such as smartphones.

I had heard about this already, but on this occasion it entered my head that
it might be fun to crawl the web for pictures containing GPS and time data and
build a database of those pictures. With such a database, I could build an
animated map showing the pictures being taken over the course of any given
timeframe. In other words, I could choose to display a month of pictures over
the course of half an hour and watch as all the pictures taken that month pop
up over the places they were taken on the map, fading after a few seconds to
keep the map from getting too crowded. Or I could just have all the pictures
taken a certain day pinned on the map, available for me to browse and examine,
giving me a general feeling for that day as seen through the eyes of
strangers. Or I could do the same without any time constraints and see a place
as it has been photographed by many different people. Not much practical
value, I suppose, but I find a certain beauty in it.

To build such a crawler, I would need to examine a lot of JPGs as quickly as
possible. All the EXIF libraries I found required opening the whole image
first, which seemed at cross-purposes to my particular need. The GPS data is
usually contained within the first few hundred bytes of a file, so why should
I have to download the entirety of a multi-megabyte file only to find out that
there never was any GPS data in it at all?

So I decided to roll my own in Python, with some thought to porting it to C if
I ended up wanting even more speed. (Or if I ended up wanting to brush up on
my C skills.)

**Caveat:**

The file I'm working with is a
[JFIF](http://www.fileformat.info/format/jpeg/egff.htm "JFIF format
information" ) JPEG. These are identical to "regular" JPEGs except for the
addition of an APP0 layer between the SOI and APP1 layers. The APP0 layer
begins with the 5-byte, null-terminated "JFIF" string.

**File Structure:**

Rather than working with [the official EXIF 2.3
specification](http://www.jeita.or.jp/japanese/standard/book/CP-3451B_E/ "The
official EXIF 2.3 specification" ), I decided to work with [a slightly more
readable PDF of the EXIF 2.2 specification](http://exif.org/Exif2-2.PDF)
supplied by [EXIF.org](http://exif.org/). I also found [the Syntax and
Structure section of the JPEG Wikipedia
article](http://en.wikipedia.org/wiki/JPEG#Syntax_and_structure) to be very
useful.

The segments we are concerned with are presented in order of their appearance.

  1. SOI - Start of Image. 2 bytes long, offset 0. Indicates what type of JPEG this is.
  2. APP_n_ \- Application-specific data. Variable length and offset. Marked by the pair of bytes FF E_n_. The 2 byte marker is then followed by 2 more bytes indicating the length of the segment. These 2 bytes include themselves in the length, so if you have just read 16 as the segment length, reading the next 14 bytes will read out the rest of the segment.
  
EXIF data is contained within the APP1 and APP2 segments. The data we're
after, the GPS data, is in APP1. Data within the EXIF segments are broken up
into [Image File
Directories](http://en.wikipedia.org/wiki/Exchangeable_image_file_format#Technical
"The technical structure of the EXIF segments" ) (IFDs) that contain image
meta-data. The pointer to the beginning of the GPS IFD is marked with 0x8825.

**Walking Through a JPEG:**

Here's a transcript from a Python shell session demonstrating what I'm talking
about:

    >>> # First, we import the os module. This will be useful to us later.  
    ... import os

    >>> # First we open the JPEG for reading as a binary file.  
    ... f = open('gps_exif.jpg', 'rb')

    >>> f.read(2)  
    '\xff\xd8'  

    >>> # ^ The SOI marker

    >>> f.read(2)  
    '\xff\xe0'  

    >>> # ^ The APP0 marker

    >>> f.read(2)  
    '\x00\x10'  

    >>> # ^ This means the APP0 segment is 16 bytes long,  
    ... # including the 2 bytes we just read.  
    ... # So we skip ahead by 14 bytes to skip the APP0 segment...  
    ... f.seek(14, os.SEEK_CUR)

    >>> f.read(2)
    '\xff\xe1'  

    >>> # ^ The APP1 marker we were looking for!  
    ...  

    >>> f.read(2)  
    '#\x86'  

    >>> # ^ The length of the APP1 segment.  
    ... # Note that the ASCII code for # is 0x23. When reading a file like this, sometimes  
    ... # Python prints the ASCII character corresponding to the hex value we're looking  
    ... # at. So the length of the APP1 segment is 0x23 0x86, or 9094, bytes.  
    ... f.tell()  
    24L  

    >>> # Since we're currently at position 24, and since we are already 2 bytes  
    ... # into the APP1 segment, this means that the SOI, APP0, and APP1 segments  
    ... # comprise the first 9116 bytes of the file. (9094 + 24 - 2 = 9116)  
    ... # If we were feeling lazy, we could actually just pass those bytes on to an  
    ... # EXIF library and it should have no problem parsing our data for us.  
    ...

    >>> f.seek(9116)  

    >>> f.read(2)  
    '\xff\xdb'  

    >>> # ^ This marks the beginning of a quantization table.  
    ... # Demonstrates that we've done our math correctly and reached the end of the  
    ... # APP1 segment.
  
**Finding the GPS data in the APP1 segment:**

As noted in the section above, the most developer-efficient way to proceed at
this point would be to just take the bytes from the beginning of the file
through the end of the APP1 segment and pass them on to an EXIF library. This
would save us quite a bit of developer time. We could even spawn a thread to
take care of this parsing and start downloading the next file immediately to
improve efficiency. But for the sake of learning, and because it would be even
more efficient to stop downloading the APP1 header as soon as we get the data
we're interested in, we're going to press on and figure out how to recognize
and extract the GPS data as soon as we get it.

But I have work due in the classes I'm taking, so that will have to wait for
now.

_(To be continued...)_

