<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Introduction](#introduction)
- [Installing Homebrew](#installing-homebrew)
- [Working with packages](#working-with-packages)
- [Getting help](#getting-help)
- [Taps](#taps)
- [Fine-level controls](#fine-level-controls)
- [Mac OS GUI apps](#mac-os-gui-apps)
- [Links](#links)
- [Some useful packages](#some-useful-packages)
  - [ffmpeg (video converter)](#ffmpeg-video-converter)
  - [pass](#pass)
  - [ccrypt](#ccrypt)
  - [dar](#dar)
  - [awscli](#awscli)
  - [imagemagick](#imagemagick)
  - [7zip compression utility](#7zip-compression-utility)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

* These notes at https://github.com/razoumov/publish/blob/master/brew.md.

Abstract: Do you have a Mac? Did you know that BSD Unix is what powers much of OSX? Did you know that
despite this heritage you don't have access to many Linux tools and resources because they are locked out
by Apple?  Would you like to unlock the full power of a Linux system and take control of your computing
destiny? In this workshop you'll walk in with a regular OSX laptop and walk out with a software
application called "Homebrew" installed that will give you the ability to install and run a wide variety
of Linux programs and tools, allowing you to enjoy the full benefits of the free and open source software
community that has made Linux the premiere scientific computing platform.

## Introduction

MacOS is a built on top of Darwin (Apple's open-source Unix OS) which itself is a derivative of various
implementations of BSD, a Unix OS from Berkeley, developed in the 1970s-80s. Being a UNIX-like OS, MacOS
is similar to Linux, but they have different kernel source code (core of the OS).

Linux comes with thousands of open-source tools and applications, and the vast majority of these can also
be compiled from source code on MacOS, as the underlying OSs share the same principles and have very
similar architecture. MacOS includes some fraction of these tools, but many are missing. To install the
ones that are missing, you can download their source code and compile them. In practice, this is tedious
and may involve many manual steps, as often you will have to compile their dependencies by hand and make
sure you have the right versions.

Using a package manager automates these tasks. Homebrew (https://brew.sh) is a package manager for MacOS:

* gives a central catalogue of available packages
* formula = a package definition written in Ruby
* provides simple commands for working with all packages: **list**, **search**, **info**, **install**,
  **update**, **uninstall**
* automatically tracks and installs missing dependencies for each package
* compiles (builds) each package/dependency from source code if a pre-compiled version is not available
* installs formulae into their own Cellar (keg) directory
* symlinks their files into /usr/local
* written in git and ruby (formulae are simple Ruby scripts)
* everything is brew themed

## Installing Homebrew

~~~ {.bash}
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

You might need to download Command Line Tools from Apple. Likely need to install Xcode, and then install
command-line tools:

~~~ {.bash}
xcode-select --install
~~~

It is a good idea to do this early on:

~~~ {.bash}
sudo chown -R $USER /usr/local
~~~

Once in a while run:

~~~ {.bash}
brew doctor      # self-diagnose, will output a lot of details
brew update      # download new formulae (where to find software, lists of dependencies, how to install)
~~~

If you ever want to uninstall Homebrew:

~~~ {.bash}
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
~~~

## Working with packages

~~~ {.bash}
brew search              # search for all packages
for i in echo $(brew search); do echo $i; done | wc -l    # shows 4707 lines
brew search imagemagick
brew info imagemagick
brew install imagemagick
brew list
brew upgrade imagemagick
brew upgrade      # all installed packages
~~~

If you ever want to remove this package:

~~~ {.bash}
brew uninstall imagemagick
~~~

## Getting help

~~~ {.bash}
brew help
brew help <command>
~~~

## Taps

If you cannot find a package in the main Homebrew repository, you can also tap other repositories:

~~~ {.bash}
brew tap user/repo     # shallow-clone https://github.com/user/repo to your laptop (make all its formulae available)
brew tap brewsci/bio
ls /usr/local/Homebrew/Library/Taps/brewsci/homebrew-bio/Formula/    # list packages in this repository
brew tap     # list all installed taps (cloned repositories) on your laptop
brew untap user/repo   # remove the taps (brew will no longer be aware of their formulae)
~~~

## Fine-level controls

~~~ {.bash}
brew pin <formula>    # stop something from being updated/upgraded
brew unpin <formula>   # allow that formulae to update again
brew reinstall <formula> --build-from-source     # force compiling from source
~~~

## Mac OS GUI apps

~~~ {.bash}
brew install cask
brew cask help
brew cask list    # list all installed packages
brew cask search     # list all available cask packages
brew cask search firefox
brew cask info firefox
brew cask install firefox
brew cask install google-chrome
brew cask install iterm2   # better replacement for Terminal
brew cask install iina     # "modern" video player for macOS
brew cask install vlc      # VLC media player
brew cask install xquartz    # Apple X-server
brew cask install bitwarden   # multi-platform password manager (GUI + command line + mobile, completely free)
~~~

## Links

- https://www.quora.com/What-are-the-first-or-must-have-homebrew-packages-that-you-install-on-your-Mac
- Homebrew Demystified http://bit.ly/2JSohmV
- https://docs.brew.sh
- https://docs.brew.sh/FAQ
- https://docs.brew.sh/Formula-Cookbook#homebrew-terminology
- http://searchbrew.com   # search packages; there are quite a few similar sites
- https://gist.github.com/indiesquidge/ec010eca3ffa254788c2
- http://www.bigfastblog.com/homebrew-intro-to-the-mac-os-x-package-installer
- http://osxdaily.com/2018/03/07/how-install-homebrew-mac-os
- https://github.com/Homebrew/homebrew-cask/blob/master/USAGE.md

## Some useful packages

~~~ {.bash}
brew install wget
brew install dos2unix
brew install emacs
brew install grip
brew install htop   # really nice command-line system resource monitor
brew install r
brew install youtube-dl
~~~

### ffmpeg (video converter)

~~~ {.bash}
brew install ffmpeg
ffmpeg -r 10 -i frame%04d.png -c:v libx264 -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" clip.mp4
ffprobe -select_streams v -show_streams clip.mp4 2>/dev/null | grep nb_frames
ffmpeg -i clip.mp4 -vframes 1 -vf "select='eq(n,0)'" -y first.png
ffmpeg -i clip.mp4 -vframes 1 -vf "select='eq(n,850)'" -y last.png
~~~

### pass

More details at https://www.passwordstore.org.

~~~ {.bash}
brew install pass
gpg --full-generate-key       # writes into ~/.gnupg/, at the end type the password
gpg --list-secret-keys --keyid-format LONG   # list GPG keys for which I have both a public and private key
pass init C4153C92CDEC8AF4       # creates ~/.password-store/
pass          # list all entries
pass email    # list email entries only
pass -c clusters/westgrid      # copy password to the clipboard
pass insert email/oneOfYourEmailAddresses     # insert a new password (one line)
pass insert -m email/oneOfYourEmailAddresses     # insert multiple lines
pass generate -n email/razoumov@gmail.com 15     # generate and insert a new 15-character password
pass mv path1 path2     # rename entry
pass rm path1           # remove entry
~~~

### ccrypt

~~~ {.bash}
brew install ccrypt    # strong encryption utility
man ccrypt
ccencrypt file
ccdecrypt file
ccat file
~~~

### dar

~~~ {.bash}
brew install dar
destination=/Volumes/boa/backups
source='-g Documents -g Desktop'
flags='-s 5G -zbzip2 -asecu -w -X "*~" -X "*.o"'
dar $flags -c $destination/pro0 -R $HOME $source
dar $flags -A $destination/pro0 -c $destination/pro1 -R $HOME $source
dar $flags -A $destination/pro1 -c $destination/pro2 -R $HOME $source
dar $flags -A $destination/pro2 -c $destination/pro3 -R $HOME $source
...
dar -l $destination/pro0
dar -l $destination/pro1
dar -l $destination/pro2
...
echo dar -R ~/Movies -O -w -x $destination/pro0 -v -g Documents/notes
echo dar -R ~/Movies -O -w -x $destination/pro1 -v -g Documents/notes
echo dar -R ~/Movies -O -w -x $destination/pro2 -v -g Documents/notes
...
~~~

### awscli

~~~ {.bash}
brew install awscli     # command-line tools for Amazon web services
https://loige.co/aws-command-line-s3-content-from-stdin-or-to-stdout
free 5GB of storage, above that ~USD$0.023/GB/month
aws configure   # one-time: enter your AWS Access Key ID and AWS Secret Access Key, will go into ~/.aws/
aws s3 cp /path/to/localFile s3://bucketyxfuol7p4ve/   # upload file to S3
aws s3 ls s3://bucketyxfuol7p4ve/backups/      # list files in a remote folder
aws s3 cp s3://bucketyxfuol7p4ve/restore.txt -     # cat S3 file into standard output
aws s3 cp s3://bucketyxfuol7p4ve/restore.txt .     # download file from S3
aws s3 rm s3://bucketyxfuol7p4ve/restore.txt   # remove S3 file or folder
aws s3 sync /path/to/local s3://bucketyxfuol7p4ve/backups/   # sync files to remote S3 folder
no command to create a folder: it'll be created automatically when copying or syncing
~~~

### imagemagick

~~~ {.bash}
brew install imagemagick --with-x11   # library and command-line tool for image manipulation;
                                      # recompile with X11 support if you have it installed
http://www.imagemagick.org/Usage
http://www.imagemagick.org/script/command-line-options.php
convert https://images.pexels.com/photos/35600/road-sun-rays-path.jpg -resize 1600x1600 forest.png
convert forest.png forest.jpg
identify -list format   # see the list of support image formats
man convert
ls -l $(which convert)
ls -l /usr/local/Cellar/imagemagick/7.0.7-7/bin/        # check the included utilities
convert https://upload.wikimedia.org/wikipedia/commons/2/2c/NZ_Landscape_from_the_van.jpg -resize 1600x1600 hills.png
convert https://upload.wikimedia.org/wikipedia/commons/4/43/Desert1.jpg -resize 1600x1600 desert.png
convert https://upload.wikimedia.org/wikipedia/commons/e/e0/Clouds_over_the_Atlantic_Ocean.jpg -resize 1600x1600 ocean.png
montage -geometry 60% -shadow *.png mosaic.png
display mosaic.png     # for this need X11 (XQuartz); click on the image for menu
wget https://transfer.sh/15kP5b/plotly.tar.gz
unpack it and cd there
# transp='-fuzz 10% -transparent white'
transp='-transparent white'
convert lines.png population.png combine.png citiesByPopulation.png $transp +append -geometry 1600x row1.png
convert network.png bubbles.png parametric.png $transp +append -geometry 1600x row2.png
convert contour.png orthogonal.png isosurface.png $transp +append -geometry 1600x row3.png
convert row{1..3}.png $transp -append thumbs.png
wget https://www.westgrid.ca/files/wesgrid_logo_2016.png -O logo.png
convert logo.png -fuzz 10% -transparent white logoTransparent.png
convert growth.png -fuzz 10% -transparent 'rgb(86,86,86)' growth.png
~~~


### 7zip compression utility

~~~ {.bash}
brew install p7zip        # port of 7zip from Windows
man convert > convert.txt
gzip convert.txt && ls -l convert.txt.gz   # 6324 bytes
gunzip convert.txt.gz
7z a convert.txt.7z convert.txt            # 6141 bytes
7z a -mx=9 convert.txt.7z convert.txt      # 6121 bytes (highest compression ratio)
7z a landscapes.7z desert.png forest.png hills.png ocean.png   # create an archive
rm desert.png forest.png hills.png ocean.png
7z l landscapes.7z        # list contents
7z t landscapes.7z        # test the archive's integrity
7z e landscapes.7z        # extract from the archive
tar -cf - dirName | 7za a -si landscapes.tar.7z    # pack a directory while keeping owner/group
7za x -so landscapes.tar.7z | tar xf -     # restore it
7z u landscapes.7z newFile1 newFile2 ...    # update the archive
7z d landscapes.7z hills.png    # remove a file from the archive
~~~
