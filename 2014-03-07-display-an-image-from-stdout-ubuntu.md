# Display an image from stdout [ubuntu]


To send bitcoin from my mobile phone to a plaintext address on my laptop, I
like to use QR codes. At first I used websites like qrstuff.com to generate
the QR codes, but leaving the command line just to generate and display an
image felt like too much of a hassle to me. Then I started using the `feh` and
`qrencode` utilities in Ubuntu's package repositories, but that involved
creating an image file on my hard drive and then deleting it afterward, which
felt messy. So I found a quick and easy way to generate and display a QR code
from the command line without creating an intermediate file.

The setup:

    sudo apt-get install qrencode giblib-dev libimlib2 libcurl4-openssl-dev libpng-dev libX11-dev libXinerama-dev
    git clone https://github.com/derf/feh.git
    cd feh
    make
    sudo make install  

Note: Though there is a copy of `feh` already in the Ubuntu package
repositories, it's an older version that does not support reading images from
stdin. Thus it is necessary to compile our own copy from the git repository.

To use the new `qrencode` and `feh` combo to, for example, display the QR code
for Andreas Antonopoulos' Dorian Nakamoto assistance fund:

    qrencode 1Dorian4RoXcnBv9hnQ4Y2C1an6NJ4UrjX -o - | feh -  
  
In the example above, the `-o` option on `qrencode` specifies where the
generated image should be output, and passing `-` to that option specifies
that it should be sent to stdout. Likewise, the `-` passed to `feh` specifies
that `feh` should read the image from stdin.

Finally, because I am extraordinarily lazy, I created a function in my
~/.bashrc file to save my future self a few extra keystrokes:

    function qrcode () {  
        qrencode $@ -o - | feh -;  
    }  
  
Which turns the previous example into the slightly more succinct:

    qrcode 1Dorian4RoXcnBv9hnQ4Y2C1an6NJ4UrjX

(Originally posted to [Coderwall](https://coderwall.com/p/s6f_0w))
