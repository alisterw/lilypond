# Dockerfile for running [LilyPond](http://lilypond.org/) in a container
This guide is written from the point of view of a macOS user, since on other operating systems it's possible to
install and run LilyPond directly (but [not on macOS since Catalina](http://lilypond.1069038.n5.nabble.com/lilypond-does-not-work-with-Mac-OS-10-15-Catalina-td224189.html)).

This version runs on Ubuntu [Focal Fossa](https://releases.ubuntu.com/20.04/).
At time of writing building the image causes lilypond version [2.20.0-1](http://lilypond.org/website/misc/announce-v2.2)
to be installed.

To use it, first build the image, then run the container as described below.

## Acknowledgements
This Dockerfile is based almost entirely on the one at https://github.com/gpit2286/docker-lilypond.

## Prerequisites
* [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac) must be installed and running.
* Any local versions of LilyPond (possibly installed prior an upgrade to Catalina broke running it locally) should be
uninstalled if you're planning to use the convenience script.

## Building the image
First clone this git repository by running the following in a Terminal window:
```
$ git clone https://github.com/alisterw/lilypond.git
```

To build the image:
```
$ cd lilypond
$ docker build --tag lilypond:1.0 .
```

Once the image is built you can delete the directory containing the cloned repo if you like.

## Running the container
You can check the container works by running it as follows:
```
$ docker run --rm -v "$PWD" -w "$PWD" lilypond:1.0 lilypond --version
GNU LilyPond 2.20.0

Copyright (c) 1996--2015 by
  Han-Wen Nienhuys <hanwen@xs4all.nl>
  Jan Nieuwenhuizen <janneke@gnu.org>
  and others.

This program is free software.  It is covered by the GNU General Public
License and you are welcome to change it and/or distribute copies of it
under certain conditions.  Invoke as `lilypond --warranty' for more
information.
```

## Convenience script
For convenience, it's worth creating a shell script which will allow you to run the container simply by typing `lilypond`
at the command prompt.

To do this, first create a file named `lilypond` in `/usr/local/bin` with the following content:
```
#!/bin/sh
/usr/local/bin/docker run --rm -v "$PWD" -w "$PWD" lilypond:1.0 lilypond "$@"
```

One easy way to create this file is to run `sudo nano /usr/local/bin/lilypond`, then paste in the above content, save by
hitting ctrl-O (not cmd-O) followed by enter, then quit by hitting ctrl-X.

You will probably need to enter your password unless you've run sudo recently.

Once you've created the file make it executable as follows:
```
$ sudo chmod +x /usr/local/bin/lilypond
```

Again, you may need to enter your password.

Now you can just invoke the container with `lilypond`:
```
$ lilypond --version
GNU LilyPond 2.20.0

Copyright (c) 1996--2015 by
  Han-Wen Nienhuys <hanwen@xs4all.nl>
  Jan Nieuwenhuizen <janneke@gnu.org>
  and others.

This program is free software.  It is covered by the GNU General Public
License and you are welcome to change it and/or distribute copies of it
under certain conditions.  Invoke as `lilypond --warranty' for more
information.
```

## Usage with [Frescobaldi](https://frescobaldi.org/)
To use the containerised version of LilyPond with Frescobaldi do the following:
* Launch Frescobaldi
* Select "Preferences..." from the "Frescobaldi" menu
* Select the "LilyPond Preferences" section from the list on the left
* Under "LilyPond versions to use" select the default entry (there will probably only be one anyway)
* Click "Edit..."
* Under "LilyPond Command:" enter `/usr/local/bin/lilypond`
* Click OK on each screen

## Troubleshooting problems with included files
If your LilyPond files include other LilyPond files which are not in the same directory (or below) you will most
likely get an error. This is because of the way we map directories when running the docker container, specifically
because we run it with the option `-v "$PWD"`. This causes the container (and therefore LilyPond) to be able to see the current directory (and anything within it) but not anything above it or alongside it.

To fix this you can mount a directory at a higher level. In general it's good practice not to give a container access to
more of your files than necessary, but if you need to you can change that option from `-v "$PWD"` to one of:
* `-v "$HOME"` - gives LilyPond access to pretty much anything your user has created on your Mac
* `-v /Users` - gives LilyPond access to pretty much anything any user has created on your Mac (assuming you have perms to it)

If you're including files outside of `/Users` then you are almost certainly doing something wrong, but you can make it
work by specifying multiple `-v` options where you explicitly mount each directory. Detailed instructions for that are
outside the scope of this guide, but it's worth noting that
* where possible you will want to mount directories to the same path inside the container as on the host system as this enables hyperlinks to work in e.g. Frescobaldi
* in some cases this won't be possible (specifying `-v /` makes no sense at all, for example), in which case you can mount a host directory elsewhere by using the syntax `- v <host-src>:<container-dst>`
