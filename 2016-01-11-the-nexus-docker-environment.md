# The Nexus Docker Environment


I work as a developer for [Nexus](http://nexusdev.us). One of the things I've
done to make it easier for new developers to come on board and start
contributing code was to create a Docker image that contains a pre-built
development environment containing everything a new developer might need. It
also helps with being able to reproduce each other's bugs, as our environments
are all very similar. I've been using it myself every day now for about a
month, so I figured it's time I write a post introducing it.

The image contains an installation of the Solidity compiler, the Go Ethereum
client, Dapple, and [a few other things](https://github.com/NexusDevelopment
/devenv-docker#devenv-docker). The Docker image is up on Docker Hub as
[ryepdx/nexus_dev](https://hub.docker.com/r/ryepdx/nexus_dev/), available for
anyone to pull down. The Dockerfile itself, along with a README detailing the
installation instructions, can be found [on
Github](https://github.com/NexusDevelopment/devenv-docker).

The image requires some configuration for enabling key-based SSH login and for
setting up any Github SSH keys you might have. The configuration steps are
outlined in detail in the README on Github. An abbreviated version of the
installation instructions can also be found below. Note that it is assumed you
already have an SSH key pair you can use for logging in to the Docker image.
If you don't, Atlassian has [a guide to generating SSH
keys](https://confluence.atlassian.com/bitbucketserver/creating-ssh-
keys-776639788.html) that covers Windows, Linux, and OS X.



    $ git clone https://github.com/NexusDevelopment/devenv-docker.git  

    $ cd devenv-docker  

    $ ./docker-run --name nexus ryepdx/nexus_dev  

    $ ifconfig

    ...  

    docker0 Link encap:Ethernet HWaddr 02:42:2A:19:B8:B2  

     inet addr:172.17.42.1 Bcast:0.0.0.0 Mask:255.255.0.0  

    ...

    $ ftp 172.17.42.1 2121

    Name: dev  

    Password:

    230 OK. Current directory is /home/dev  

    Remote system type is UNIX.  

    Using binary mode to transfer files.




    ftp> put ~/.ssh/docker.pub .ssh/authorized_keys  

    local: /home/ryepdx/.ssh/docker.pub remote: .ssh/authorized_keys  

    200 PORT command successful  

    150 Connecting to port 42773  

    226-File successfully transferred




    ftp> put ~/.ssh/github.key .ssh/github.key  

    local: /home/ryepdx/.ssh/github.key remote: .ssh/github.key  

    200 PORT command successful  

    150 Connecting to port 42773  

    226-File successfully transferred




    ftp> put ~/.ssh/github.pub .ssh/github.pub  

    local: /home/ryepdx/.ssh/github.pub remote: .ssh/github.pub  

    200 PORT command successful  

    150 Connecting to port 54191  

    226-File successfully transferred




    ftp> bye  

    221 Goodbye.




    $ ssh -i ~/.ssh/docker.key -p 2222 dev@172.17.42.1




    (nexus)dev@c41e6f18b925:~$ passwd  

    Changing password for dev.  

    (current) UNIX password:  

    Enter new UNIX password:  

    Retype new UNIX password:  

    passwd: password updated successfully  

    (nexus)dev@c41e6f18b925:~$ chmod 400 ~/.ssh/*.key*


  
If you already have an Ethereum chain synced up on your host machine, you
might also consider mounting the ~/.ethereum directory on your host machine to
/home/dev/.ethereum in your Docker container using Dockers "-v" flag during
the "./docker-run" step. This prevents duplication of data and saves you some
hard drive space.

Once your Docker container is all configured, you can opt to write your code
using an external IDE over FTP, or you can use the copy of vim included in the
image. I personally use vim and keep all my vim configuration files [on
Github](https://github.com/ryepdx/dotfiles) so all I have to do to get vim set
up the way I like it is to just pull them down.

One of the really nice things about having my development environment in a
Docker image has been the ease with which I can tear everything down and bring
everything back up again. If I do something that borks my environment, I don't
have to spend a bunch of time debugging it. I can just delete my Docker
container and start again. I've also successfully leveraged
[Unison](https://www.cis.upenn.edu/~bcpierce/unison) and a Digital Ocean
droplet running CoreOS to provide myself with a remote development environment
that stays synchronized with my local environment. This allows me to SSH into
my Digital Ocean server with my phone or a tablet to write some code or run
some tests whenever the mood strikes me. It's like using a cloud-based IDE,
except I have more control over my experience and my data.

Hopefully you find this Docker image useful. If you have any questions or need
help, get in contact with me and I'll do what I can.

