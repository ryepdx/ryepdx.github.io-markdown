# Building a web crawler for extracting GPS data from JPGs (part 2)


**Recap:**

    >>> f = open("gps_exif.jpg", "rb")  

    >>> f.seek(20)  

    >>> f.read(10)  
    '\xff\xe1#\x86Exif\x00\x00'
  
** Big-endian, little-endian:**

The endianness of the JPEG is determined by the four bytes following the APP1
opening headers. 'II\x2a\x00' stands for Intel, whose CPUs are little-endian,
and 'MM\x00\x2a' stands for Motorola, whose 680x0 CPUs are big-endian.
([Source](http://en.wikipedia.org/wiki/Endianness#Endianness_in_files_and_byte_swap))

    >>> f.read(4)  
    'MM\x00*'
  
Awesome, we have a big-endian file on our hands. So the higher-order byte will
come first in this encoding. Concretely, this means that the number 256 will
be encoded as '\x01\x00' instead of '\x00\x10' (as it would be in little-
endian encoding).

"So how do I deal with endianness," you ask? Here are a couple functions I
totally copied from [an example
script](http://fetidcascade.com/public/minimal_exif_reader.py) on
[fetidcascade.com](http://fetidcascade.com/):

      def s2n_motorola(self, str):  
        x = 0  

        for c in str:  
          x = (x << 8) | ord(c)  

        return x


      def s2n_intel(self, str):  
        x = 0  
        y = 0  

        for c in str:  
          x = x | (ord(c) << y)  
          y = y + 8  

        return x
  
These will translate the hex values for you depending on whether your file is
big-endian or little-endian.

All the offsets for the IFDs will be in terms of bytes from the end of the
null-terminated "Exif" string we saw earlier. For purposes of our exploration,
we'll want to keep track of where this point is. Otherwise we won't be able to
'seek' to the appropriate positions. (An alternative would be to read the rest
of the APP1 segment into a string and simply use array index notation to grab
the values we need, but I'm writing this with an eye to processing values as
they are streamed so we can download only the data we need and no more.)

Since we just read four bytes past the end of the "Exif" string, we'll need to
back up by four bytes to get our start position.

    >>> f.tell()  
    34L  

    >>> f.seek(30)  

    >>> start = f.tell()  

    >>> start  
    30L
  
Now we skip the four "endianness" bytes and get the four bytes that follow.
This is the offset from the end of the "Exif" string at which the fields
begin.

    >>> f.seek(start+4)  

    >>> f.read(4)  
    '\x00\x00\x00\x08'

So...

    >>> f.seek(start+8)
  
...puts us at the beginning of the IFDs. Almost. Read the next two bytes to
get the total number of IFDs the JPEG contains and _then_ you will be up
against the fields themselves.

    >>> f.read(2)  
    '\x00\x0b'
  
Our JPEG contains 11 IFDs.

Alright, before we dive into getting values from these fields, it's important
to note that, much like directories, IFDs can be recursive. That is to say,
IFDs can contain more IFDs. That 11 we read up there is just the number of
_top-level IFDs._ Within those IFDs, there may be _more_ IFDs, just as a
directory can contain more directories.

Each IFD begins with a two-byte code indicating what kind of IFD it is. (IFDs
that contain other IFDs start with the code '\x87\x69', FYI.) So we read the
next two bytes to find out what type of IFD comes first:

    >>> f.read(2)  
     '\x01\x0f'
  
Reading from the handy-dandy [EXIF spec PDF](http://exif.org/Exif2-2.PDF), I
see that this is the IFD field corresponding to "Make." (I imagine this field
holds the make of the camera that took the picture.)

The next two bytes contain a code corresponding to the IFD's datatype:

    >>> f.read(2)  
     '\x00\x02'
  
Okay, so looking at the spec I see that this corresponds to ASCII data. I've
reproduced the datatype table below for your convenience.

1 = BYTE: An 8-bit unsigned integer.  
2 = ASCII: An 8-bit byte containing one 7-bit ASCII code. The final byte is
terminated with NULL.  
3 = SHORT: A 16-bit (2-byte) unsigned integer.  
4 = LONG: A 32-bit (4-byte) unsigned integer.  
5 = RATIONAL: Two LONGs. The first LONG is the numerator and the second LONG
expresses the denominator.  
7 = UNDEFINED: An 8-bit byte that can take any value depending on the field
definition.  
9 = SLONG: A 32-bit (4-byte) signed integer (2's complement notation).  
10 = SRATIONAL: Two SLONGs. The first SLONG is the numerator and the second
SLONG is the denominator.

The next four bytes indicate how many values of that type comprise this IFD.

    >>> f.read(4)  
    '\x00\x00\x00\x06'
  
So it looks like the Make IFD is comprised of six one-byte values. Since this
is greater than four bytes, the next four bytes will give us the offset where
the values begin. If this IFD's values were four bytes or less, the values
would be stored here. (Note that values less than four bytes are right-padded,
so if the IFD's value was 0x0a 0x34 and two bytes long, it would be recorded
as '\x0a\x34\x00\x00'.)

    >>> f.read(4)  
    '\x00\x00\x00\x92'
  
So we go to that offset and get the six bytes there...

    >>> f.seek(start+0x92)  

    >>> f.read(6)  
    'Apple\x00'
  
This picture was taken with an Apple device!

Okay, so obviously we know now how to, in general, find our EXIF fields and
get their values. You can refer to the spec for more information if you need
it.

Now that we know how to grab EXIF data on the fly from an image, we can start
building our webcrawler. But that'll have to wait until next time.

_(To be continued...)_

