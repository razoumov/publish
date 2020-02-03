<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Introduction](#introduction)
- [**Remote part**: logging in](#remote-part-logging-in)
- [Navigating directories](#navigating-directories)
- [Getting help](#getting-help)
- [Creating things](#creating-things)
- [Moving and copying things](#moving-and-copying-things)
- [Working with `tar` and `gzip/gunzip`](#working-with-tar-and-gzipgunzip)
- [**Remote part**: transferring files and folders with `scp`](#remote-part-transferring-files-and-folders-with-scp)
- [**Remote part**: transferring files interactively with `sftp`](#remote-part-transferring-files-interactively-with-sftp)
- [Wildcards, redirection to files, and pipes](#wildcards-redirection-to-files-and-pipes)
- [Loops](#loops)
- [Shell Scripts](#shell-scripts)
  - [Advanced: if statements](#advanced-if-statements)
  - [Advanced: variables](#advanced-variables)
  - [Advanced: functions](#advanced-functions)
  - [Advanced: aliases](#advanced-aliases)
- [Finding things](#finding-things)
  - [Advanced: running a command on the results of *find*](#advanced-running-a-command-on-the-results-of-find)
- [Text manipulation (DH part: the invisible man)](#text-manipulation-dh-part-the-invisible-man)
- [Column-based text processing with `awk` scripting language](#column-based-text-processing-with-awk-scripting-language)
- [Fuzzy finder `fzf`](#fuzzy-finder-fzf)
- [Other advanced bash topics](#other-advanced-bash-topics)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

**UBC Feb-07 tentative program**

- tar, gzip/gunzip, tar + g(un)zip
- pipes? • loops? • aliases?  • scripts and functions • find • grep • find + grep (command substitution)
- `find -exec` • invisible man • awk • permissions • fzf • other?

This file can be found at http://bit.ly/bashmd

**Quiz setup**

instructor                               | students
---------------------------------------- | ----------------------------------------
(1) log in to socrative.com as a teacher | (1) log in to socrative.com as a student
(2) start a quiz, select 'bash quiz'     | (2) enter provided room name
(3) select 'teacher paced'               |
(4) disable student names                |
(5) start                                |


<!-- - a mix of http://bit.ly/bashmd, http://bit.ly/gitcontrolmd, https://hpc-carpentry.github.io/hpc-shell, -->
<!--   and maybe some https://hpc-carpentry.github.io/hpc-intro -->

# Introduction

* Why use a cluster: computing beyond the scale of a desktop (faster, bigger, cost, efficientcy)
* Using HPC systems often involves the use of a shell
  * text-based interface to the OS (unlike a GUI): (1) command interpreter and mechanism to launch tools
    / utilities / compiled binaries / system calls / other scripts and (2) a way to connect standard I/O
    of these tools through pipes to form more complex commands
  * follows the classic UNIX philosophy of breaking complex projects into simpler subtasks and chaining
    together components and utilities
  * a shell around the kernel (coconut analogy), along with utilities and applications
  * commands are often very cryptic (to avoid too much typing)
  * bash is one of many Unix shell implementations
* Why learn Unix shell: very powerful, great for automating workflows, necessary on bigger Unix systems
* Connection to an HPC system is often done via SSH using a terminal on your laptop
  * Linux and Mac laptops have built-in terminals
  * on Windows many terminal emulators; we'll be using a free version of MobaXterm
* We have set up a small cluster training cluster cassiopeia.c3.ca, that features the same software setup
  as our real production clusters
  * in *"Intro to HPC"* we will learn the specifics of working on a cluster: software environment,
    scheduler, compilers, etc.
  * now -- using this cluster -- we will learn how to work with a remote Linux system using the shell,
    the basic Linux commands, working with the file system, how to remote-transfer files, and similar
    introductory topics

# **Remote part**: logging in

Let's log in to cassiopeia.c3.ca using a username userXX (where XX=01..60):

~~~ {.bash}
[local]$ ssh userXX@cassiopeia.c3.ca   # password supplied by the instructor
~~~

* those on Windows please use MobaXterm
* how to tell the difference between the remote and local terminals

# Navigating directories

~~~ {.bash}
$ whoami   # explain what happens when you type a command: finds it, runs it, displays output, new prompt
$ pwd   # explain /, root directory, path, why 'pwd' is so short (traces to early 1970s)
$ ls
$ ls someDirectory
$ ls /
$ ls -F   # -F displays a bit more info (adds trailing / to the names of directories); -F is a flag/option
$ ls -F someDirectory   # can combine both arguments and flags
$ ls -F /someDirectory   # why an error?
$ cd /Users/razoumov   # this is an absolute path
$ cd Documents   # and this is a relative path
$ cd ..   # go one directory up (parent directory)
$ ls ../directoryName   # also a relative path
$ ls -F -a   # -a displays hidden files
$ cd   # go to home directory, the same as 'cd ~'
$ cd -   # go to previous directory
~~~

# Getting help

~~~ {.bash}
$ man ls
$ ls --help
~~~

> Question: Looking at `ls` documentation, what does the -h (--human-readable) option do?

Explain tab completion in bash.

> **Quiz 1:** If pwd displays /Users/thing, what will ls ../backup display?

> **Quiz 2:** If pwd displays /Users/backup, and -r tells ls to display things in reverse order, what
> command will display: pnas-sub/ pnas-final/ original/

> **Quiz 3:** What does the command `cd` without a directory name do?

> **Quiz 4:** Multiple ways to return to the home directory.

# Creating things

~~~ {.bash}
$ mkdir thesis
$ ls -F
$ ls -F thesis
$ cd thesis
$ nano draft.txt   # let's spend few minutes learning how to use nano; can also use other editors
$ ls
$ more draft.txt   # displays a file one page at a time
$ cd ..
$ rm thesis   # getting an error - why?
$ rmdir thesis   # again getting an error - why?
$ rm thesis/draft.txt
$ rmdir thesis
~~~

Also could do 'rm -r thesis' in lieu of the last two commands.

# Moving and copying things

~~~ {.bash}
$ mkdir thesis
$ nano thesis/draft.txt
$ ls thesis
$ mv thesis/draft.txt thesis/quotes.txt
$ ls thesis
$ mv thesis/quotes.txt .   # . stands for current directory
$ ls thesis
$ ls
$ ls quotes.txt
~~~

~~~ {.bash}
$ cp quotes.txt thesis/quotations.txt
$ ls quotes.txt thesis/quotations.txt
$ rm quotes.txt
$ ls quotes.txt thesis/quotations.txt
~~~

More than two arguments to mv/cp:
~~~ {.bash}
$ touch  intro.txt  methods.txt  index.txt   # create three empty files
$ ls
$ mv  intro.txt  methods.txt  index.txt  thesis   # the last argument is the destination directory
$ ls
$ ls thesis
~~~

> **Quiz 5:** Suppose that you created a .txt file in your current directory to contain a list of the
> statistical tests you will need to do to analyze your data, and named it: statstics.txt. After creating
> and saving this file you realize you misspelled the filename! You want to correct the mistake, which of
> the following commands could you use to do so?

> **Quiz 6:** What is the output of the closing `ls` command in the sequence shown below?

# Working with `tar` and `gzip/gunzip`

Let's download some files in Windows' ZIP format:

~~~ {.bash}
$ wget http://bit.ly/bashfile -O bfiles.zip
$ unzip bfiles.zip
$ rm bfiles.zip
$ ls
$ ls data-shell
~~~

ZIP is a compression format from Windows, and it is not very popular in the Unix world. Let's archive the
directory `data-shell` using Unix's native `tar` command:

~~~ {.bash}
$ tar cvf bfiles.tar data-shell/
$ gzip bfiles.tar
~~~

You can also create a gzipped TAR file in one step:

~~~ {.bash}
$ rm bfiles.tar.gz
$ tar cvfz bfiles.tar.gz data-shell/
~~~

Let's remove the directory and the original ZIP file (if still there), and extract directory from our new
archive:

~~~ {.bash}
$ /bin/rm -r data-shell/ bfiles.zip
$ tar xvfz bfiles.tar.gz
~~~

> **Exercise:** Let's create a new subdirectory `~/tmp` with 1000 files inside using `touch a{000..999}`
> and then gzip-archive that subdirectory.

# **Remote part**: transferring files and folders with `scp`

To copy a single file to/from the cluster, we can use `scp`:

~~~ {.bash}
[local]$ scp /path/to/local/file.txt userXX@cassiopeia.c3.ca:/path/on/remote/computer
[local]$ scp local-file.txt userXX@cassiopeia.c3.ca:   # will put into your remote home
[local]$ scp userXX@cassiopeia.c3.ca:/path/on/remote/computer/file.txt /path/to/local/
~~~

To recursively copy a directory, we just add the `-r` (recursive) flag:

~~~ {.bash}
[local]$ scp -r some-local-folder/ userXX@cassiopeia.c3.ca:target-directory/
~~~

You can also use wildcards to transfer multiple files:

~~~ {.bash}
[local]$ scp centos@cassiopeia.c3.ca:start*.sh .
~~~~

With MobaXterm in Windows, you can actually copy files by dragging them between your desktop and the left
pane when you are logged into the cluster (no need to type any commands), or you can click the
download/upload buttons.

> **Exercise:** try to transfer a file from your laptop to the cluster. Then try moving another file in
> the opposite direction.

# **Remote part**: transferring files interactively with `sftp`

`scp` is useful, but what if we don't know the exact location of what we want to transfer? Or perhaps
we're simply not sure which files we want to transfer yet. `sftp` is an interactive way of downloading
and uploading files. Let's connect to a cluster with `sftp`:

~~~ {.bash}
[local]$ sftp userXX@cassiopeia.c3.ca
~~~

This will start what appears to be a shell with the prompt `sftp>`. However, we only have access to a
limited number of commands. We can see which commands are available with `help`:

~~~ {.bash}
sftp> help
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp grp path                     Change group of file 'path' to 'grp'
chmod mode path                    Change permissions of file 'path' to 'mode'
chown own path                     Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afPpRr] remote [local]       Download file
reget [-fPpRr] remote [local]      Resume download file
reput [-fPpRr] [local] remote      Resume upload file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
...
~~~

Notice the presence of multiple commands that make mention of local and remote. We are actually browsing
two filesystems at once, with two working directories!

~~~ {.bash}
sftp> pwd    # show our remote working directory
sftp> lpwd   # show our local working directory
sftp> ls     # show the contents of our remote directory
sftp> lls    # show the contents of our local directory
sftp> cd     # change the remote directory
sftp> lcd    # change the local directory
sftp> put localFile    # upload a file
sftp> get remoteFile   # download a file
~~~

And we can recursively put/get files by just adding `-r`. Note that the directory needs to be present
beforehand:

~~~ {.bash}
sftp> mkdir content
sftp> put -r content/
~~~

To quit, type `exit` or `bye`. 

> **Exercise:** Using one of the above methods, try transferring files to and from the cluster. For
> example, you can download bfiles.tar.gz to your laptop. Which method do you like best?

**Note on Windows**:
* When you transfer files to from a Windows system to a Unix system (Mac, Linux, BSD, Solaris, etc.) this
  can cause problems. Windows encodes its files slightly different than Unix, and adds an extra character
  to every line.
* On a Unix system, every line in a file ends with a `\n` (newline). On Windows, every line in a file
  ends with a `\r\n` (carriage return + newline). This causes problems sometimes.
* You can identify if a file has Windows line endings with `cat -A filename`. A file with Windows line
  endings will have `^M$` at the end of every line. A file with Unix line endings will have `$` at the
  end of a line.
* Though most modern programming languages and software handles this correctly, in some rare instances,
  you may run into an issue. The solution is to convert a file from Windows to Unix encoding with the
  `dos2unix filename` command. Conversely, to convert back to Windows format, you can run `unix2dos
  filename`.

**Note on syncing**: there also a command `rsync` for synching two directories. It is super useful,
especially for work in progress. For example, you can use it the download all the latest PNG images from
your working directory on the cluster.

# Wildcards, redirection to files, and pipes

<!-- * open http://bit.ly/bashfile in your browser, it'll download the file bfiles.zip -->
<!-- * unpack bfiles.zip to your Desktop; you should see ~/Desktop/data-shell -->

~~~ {.bash}
$ cd <parentDirectoryOf`data-shell`>
$ ls data-shell
$ cd data-shell/molecules
$ ls
$ ls p*   # this is a Unix wildcard; bash will expand it to 'ls pentane.pdb propane.pdb'
$ ls *.pdb   # another wildcard, will expand to "ls cubane.pdb ethane.pdb ...'
$ wc -l *.pdb   # list number of lines in each file
$ wc -l *.pdb > lengths.txt   # redirect the output of the last command into a file
$ more lengths.txt
$ sort -n lengths.txt   # sort numerically, i.e. 2 will go before 10, and 6 before 22
$ sort -n lengths.txt > sorted.txt
$ head -1 sorted.txt   # show the length of the shortest (number of lines) file
$ wc -l *.pdb | sort -n | head -1   # three commands can be shortened to one - this is called Unix pipe
~~~

Standard input of a process. Standard output of a process. Pipes connect the two.

> **Exercise:** build a single command to show the lenth of the longest (number of lines) file

> **Exercise:** Try to explain the difference between these two commands:
> ~~~ {.bash}
> echo hello > test.txt
> echo hello >> test.txt
> ~~~

> **Quiz 7:** What code would you use to move all the .dat files into the analyzed sub-directory?

> **Quiz 8:** Command to list the three files with the least number of lines.

# Loops

~~~ {.bash}
$ cd ~/Desktop/data-shell/creatures
$ ls   # shows basilisk.dat unicorn.dat -- let's pretend there are several hundred files here
~~~

Let's say we want to rename
basilisk.dat -> original-basilisk.dat
unicorn.dat -> original-unicorn.dat

We could try

~~~ {.bash}
$ mv *.dat original-*.dat   # getting an error
~~~

Remember if more than two arguments to mv, the last argument is the destination directory, but there is
no directory matching original-*.dat, so we are getting an error. The proper solution is to use loops.

~~~ {.bash}
$ for filename in basilisk.dat unicorn.dat     # filename is the loop variable here
> do
>   ls -l $filename                 # get the value of the variable by placing $ in front of it
> done
~~~

$filename is equivalent to ${filename}

Let's simplify the previous loop:
~~~ {.bash}
$ for f in *.dat
> do
>   ls -l $f
> done
~~~

Let's include two commands per each loop iteration:
~~~ {.bash}
$ for f in *.dat
> do
>   echo $f
>   head -3 $f
> done
~~~

Now to renaming basilisk.dat -> original-basilisk.dat, unicorn.dat original-unicorn.dat:
~~~ {.bash}
$ for f in *.dat
> do
> cp $f original-$f
> done
~~~

> **Quiz 9:** Output of a loop.

> **Quiz 10:** Write a loop.

The general syntax is

~~~ {.bash}
$ for <variable> in <collection>
> do
>   commands with $variable
> done
~~~

where a collection could be a explicit list of items, a list produced by a wildmask, or a collection of
numbers/letters. For example:

~~~ {.bash}
$ echo {1..10}   # this is called brace expansion
$ echo {1,2,5}    # very useful for loops or for including into large paths with multiple items, e.g.
$ cd ~/Desktop/data-shell/creatures
$ ls -l ../molecules/{ethane,methane,pentane}.pdb
$ echo {a..z}    # can also use letters
$ echo {a..z}{1..10}   # this will produce 260 items
$ echo {a..z}{a..z}    # this will produce 676 items
$ seq 1 2 10      # step=2, so can use: for i in $(seq 1 2 10)
$ for ((i=1; i<=5; i++)) do echo $i; done   # can use C-style loops
~~~

# Shell Scripts

We now know a lot of UNIX commands! Wouldn't it be great if we could save certain commands so that we
could run them later or not have to type them out again? As it turns out, this is extremely easy to
do. Saving a list of commands to a file is called a "shell script". These shell scripts can be run
whenever we want, and are a great way to automate our work.

~~~ {.bash}
$ cd ~/Desktop/data-shell/molecules
$ nano middle.sh
	#!/bin/bash         # this is called sha-bang; can be omitted for generic (bash/csh/tcsh) commands
	echo Looking into file octane.pdb
	head -15 octane.pdb | tail -5       # what does it do?
$ bash middle.sh   # the script ran!
~~~

Alternatively, you can change file permissions:

~~~ {.bash}
$ chmod u+x middle.sh
$ ./middle.sh
~~~

Let's pass an arbitrary file to it:
~~~ {.bash}
$ nano middle.sh
	#!/bin/bash
	echo Looking into file $1       # $1 means the first argument to the script
    head -15 $1 | tail -5
$ ./middle cubane.pdb
$ ./middle propane.pdb
~~~

* head -15 "$1" | tail -5     # placing in double-quotes lets us pass filenames with spaces
* head $2 $1 | tail $3        # what will this do?
* $# holds the number of command-line arguments
* $@ means all command-lines arguments to the script (words in a string)

> **Quiz 11:** script.sh in molecules Users/nelle/molecules.

> **Exercise:** write a script that takes any number of filenames, e.g., "scriptName.sh cubane.pdb
> propane.pdb", for each file prints the number of lines and its first five lines, and separates the
> output from different files by an empty line.

## Advanced: if statements

Let's write and run the following script:

~~~ {.bash}
$ nano check.sh
    for f in $@
    do
      if [ -e $f ]      # make sure to have spaces around each bracket!
      then
        echo $f exists
      else
        echo $f does not exist
      fi
    done
$ chmod u+x check.sh
$ ./check.sh a b c check.sh
~~~

* Full syntax is:

~~~ {.bash}
if [ condition1 ]
then
  command 1
  command 2
  command 3
elif [ condition2 ]
then
  command 4
  command 5
else
  default command
fi
~~~

Some examples of conditions (**make sure to have spaces around each bracket!**):

* [ $myvar == 'text' ] checks if variable is equal to 'text'
* [ $myvar == number ] checks if variable is equal to number
* [ -e fileOrDirName ] checks if fileOrDirName exists
* [ -d name ] checks if name is a directory
* [ -f name ] checks if name is a file
* [ -s name ] checks if file name has length greater than 0

> **Exercise:** write a script that complains when it does not receive arguments.

## Advanced: variables

We already saw variables that were specific to scripts ($1, $@, ...). Variables can be used outside of
scripts:

~~~ {.bash}
$ myvar=3     # no spaces permitted around the equality sign!
$ echo myvar     # will print the string 'myvar'
$ echo $myvar     # will print the value of myvar
~~~

Sometimes you see notation:

~~~ {.bash}
$ export myvar=3
~~~

Using 'export' will make sure that all inherited processes of this shell will have access to this
variable. Try defining the variable *newvar* without/with 'export' and then running the script:

~~~ {.bash}
$ nano middle.sh
	#!/bin/bash
    echo $newvar
~~~

You can assign a command's output to a variable to use in another command (this is called *command
substitution*) -- we'll see this later when we play with 'find' command.

~~~ {.bash}
$ printenv    # print all declared variables
$ env         # same
$ unset myvar   # unset a variable
~~~

Environment variables are those that affect the behaviour of the shell and user interface:

~~~ {.bash}
$ echo $HOME
$ echo $PATH
$ echo $PWD
$ echo $PS1
~~~

It is best to define custom environment variables inside your ~/.bashrc file. It is loaded every time you
start a new shell.

## Advanced: functions

Like in any programming language, in bash a function is a block of code that you can access by its
name. The syntax is:

~~~ {.bash}
functionName() {
  command 1
  command 2
  ...
}
~~~

Inside functions you can access its arguments with variables $1 $2 ... $# $@ -- exactly the same as in
scripts. Functions are very convenient because you can define them inside your ~/.bashrc
file. Alternatively, you can place them into a file and then **source** them whenever needed:

~~~ {.bash}
$ source allMyFunctions.sh
~~~

Here is our first function:

~~~ {.bash}
greetings() {
  echo hello
}
~~~

Let's write a function 'combine()' that takes all the files we pass to it, copies them into a
randomly-named directory and prints that directory to the screen:

~~~ {.bash}
combine() {
  if [ $# -eq 0 ]; then
    echo "No arguments specified. Usage: combine file1 [file2 ...]"
    return 1        # return a non-zero error code
  fi
  dir=$RANDOM$RANDOM
  mkdir $dir
  cp $@ $dir
  echo look in the directory $dir
}
~~~

> **Exercise:** write a function to swap two file names. Add a check that both files exist, before
> renaming them.

> **Exercise:** write a function archive() that takes a directory as an argument, packs it into a gzipped
> tar archive (often called *tarball*) and deletes the original directory.

> **Exercise:** write the reverse function unarchive() that replaces a gzipped tarball with a directory.

## Advanced: aliases

Aliases are one-line shortcuts/abbreviation to avoid typing a longer command, e.g.

~~~ {.bash}
$ alias cp='cp -i'
$ alias mv='mv -i'
$ alias ls='ls -aFh'
$ alias hi='history'
$ alias cedar='ssh -Y cedar.computecanada.ca'
$ alias weather='curl wttr.in/vancouver'
~~~

# Finding things

~~~ {.bash}
$ cd ~/Desktop/data-shell/writing
$ more haiku.txt
~~~

First let's search for text in files:
~~~ {.bash}
$ grep not haiku.txt     # let's find all lines that contain the word 'not'
$ grep day haiku.txt     # now search for word 'day'
$ grep -w day haiku.txt        # search for a separate word 'day' (not 'today', etc.)
$ grep -w today haiku.txt   # search for 'today'
$ grep -w Today haiku.txt   # search for 'Today'
$ grep -i -w today haiku.txt       # both upper and lower case 'today'
$ grep -n -i -w today haiku.txt    # -n prints out numbers the matching lines
$ grep -n -i -w -v the haiku.txt   # -v searches for lines that do not contain 'the'
$ man grep
~~~

More than two arguments to grep:
~~~ {.bash}
$ grep pattern file1 file2 file3   # all argument after the first one are assumed to be filenames
$ grep pattern *.txt   # the last argument will expand to the list of *.txt files
~~~

> **Quiz 12:** grep command.

Now on to finding files:
~~~ {.bash}
cd ~/Desktop/data-shell/writing
$ find . -type d     # search for directories inside current directory
$ find . -type f     # search for files inside current directory
$ find . -maxdepth 1 -type f     # depth 1 is the current directory
$ find . -mindepth 2 -type f     # current directory and one level down
$ find . -name haiku.txt      # finds specific file
$ ls data       # shows one.txt two.txt
$ find . -name *.txt      # still finds one file -- why? answer: expands *.txt to haiku.txt
$ find . -name '*.txt'    # finds all three files -- good!
~~~

Let's wrap the last command into $() (called *command substitution*), as if it was a variable:

~~~ {.bash}
$ echo $(find . -name '*.txt')   # will print ./data/one.txt ./data/two.txt ./haiku.txt
$ ls -l $(find . -name '*.txt')   # will expand to ls -l ./data/one.txt ./data/two.txt ./haiku.txt
$ wc -l $(find . -name '*.txt')   # will expand to wc -l ./data/one.txt ./data/two.txt ./haiku.txt
$ grep elegant $(find . -name '*.txt')   # will look for 'elegant' inside all *.txt files
~~~

> **Quiz 13:** combining grep and find.

> **Exercise:** write a function 'countFiles()' that counts the number of files in each directory that
> you pass to it and prints it after the directory name.

## Advanced: running a command on the results of *find*

You always can do something like this:

~~~ {.bash}
$ for f in $(find . -name "*.txt")
> do
>   command on $f
> done
~~~

However, you can actually make it a one-liner:

~~~ {.bash}
find . -name "*.txt" -exec command {} \;       # important to have spaces
~~~

-- this will run the command on each item in the output of find.

# Text manipulation (DH part: the invisible man)

In this section we'll use two tools for text manipulation: *sed* and *tr*. Our goal is to calculate the
frequency of all dictionary words in the novel "The Invisible Man" by Herbert Wells (public
domain). First, let's apply our knowledge of grep to this text:

~~~ {.bash}
$ cd ~/Desktop/data-shell
$ ls   # shows wellsInvisibleMan.txt
$ wc wellsInvisibleMan.txt   # number of lines, words, characters
$ grep invisible wellsInvisibleMan.txt   # see the invisible man
$ grep invisible wellsInvisibleMan.txt | wc -l   # returns 60; adding -w gives the same count
$ grep -i invisible wellsInvisibleMan.txt | wc -l   # returns 176 (includes: invisible Invisible INVISIBLE)
~~~

Let's use the "stream editor" sed:
~~~ {.bash}
$ sed 's/[iI]nvisible/supervisible/g' wellsInvisibleMan.txt > visibleMan.txt   # make him visible
$ cat wellsInvisibleMan.txt | sed 's/[iI]nvisible/supervisible/g' > visibleMan.txt   # this also works (standard input)
$ grep supervisible visibleMan.txt   # see what happened to the now visible man
$ grep -i invisible visibleMan.txt   # see what was not converted
$ man sed
~~~

Now let's remove punctuation from the original file using "tr" (translate) command:
~~~ {.bash}
$ cat wellsInvisibleMan.txt | tr -d "[:punct:]" > invisibleNoPunct.txt    # tr only takes standard input
$ tail wellsInvisibleMan.txt
$ tail invisibleNoPunct.txt
~~~

Next convert all upper case to lower case:
~~~ {.bash}
$ cat invisibleNoPunct.txt | tr '[:upper:]' '[:lower:]' > invisibleClean.txt
$ tail invisibleClean.txt
~~~

Next replace spaces with new lines:
~~~ {.bash}
$ cat invisibleClean.txt | sed 's/ /\'$'\n/g' > invisibleList.txt   # \'$'\n is a shortcut for a new line
$ more invisibleList.txt
~~~

Next remove empty lines:
~~~ {.bash}
$ sed '/^$/d' invisibleList.txt  > invisibleCompact.txt
~~~

Next sort the list alphabetically, count each word's occurrence, and remove duplicate words:
~~~ {.bash}
$ cat invisibleCompact.txt | sort | uniq -c > invisibleWords.txt
$ more invisibleWords.txt
~~~

Next sort the list into most frequent words:
~~~ {.bash}
$ cat invisibleWords.txt | sort -gr > invisibleFrequencyList.txt   # use 'man sort'
$ more invisibleFrequencyList.txt
~~~

> **Exercise:** write a script 'countWords.sh' that takes a text file name as an argument, and returns
> the list of its 100 most common words, i.e. the script should be used as `./countWords.sh
> wellsInvisibleMan.txt`. The script should not leave any intermediate files. Or even better, write a
> function 'countWords()' taking a text file name as an argument.





# Column-based text processing with `awk` scripting language

~~~ {.bash}
cd .../data-shell/writing
cat haiku.txt   # 11 lines
~~~

You can define inline awk scripts with braces surrounded by single quotation:

~~~ {.bash}
awk '{print $1}' haiku.txt    # $1 is the first field (word) in each line => processing columns
awk '{print $1}' haiku.txt    # $0 is the whole line
awk '{print}' haiku.txt       # the whole line is the default action
awk -Fa '{print $1}' haiku.txt   # can specify another separator with -F ("a" in this case)
~~~

You can use multiple commands inside your awk script:

~~~ {.bash}
echo Hello Tom > hello.txt
echo Hello John >> hello.txt
awk '{$2="Adam"; print $0}' hello.txt       # we replaced the second word in each line with "Adam"
~~~

Most common `awk` usage is to postprocess output of other commands:

~~~ {.bash}
/bin/ps aux    # display all running processes in multi-columns output
/bin/ps aux | awk '{print $2 " " $11}'     # print only the process number and the command
~~~

Awk also takes patterns in addition to scripts:

~~~ {.bash}
awk '/Yesterday|Today/' haiku.txt              # print the lines that contain the words Yesterday or Today
~~~

And then you act on these patterns: if the pattern evaluates to True, then run the script:

~~~ {.bash}
awk '/Yesterday|Today/{print $3}' haiku.txt
awk '/Yesterday|Today/' haiku.txt | awk '{print $3}'   # same as previous line
~~~

Awk has a number of built-in variables; the most commonly used is NR:

~~~ {.bash}
awk 'NR>1' haiku.txt    # if NumberRecord >1 then print it (default action), i.e. skip the first line
awk 'NR>1{print $0}' haiku.txt    # last command expanded
awk 'NR>1 && NR < 5' haiku.txt    # print lines 2-4
~~~

> **Exercise:** write a awk script to process `cities.csv` to print only town/city names and their
> population and store it in a separate file `populations.csv`. Try to do everything in a single-line
> command.






# Fuzzy finder `fzf`

* third-party tool, not installed by default
* basic usage: interactive processing of standard input
* advanced usage: key bindings and fuzzy completion

On cassiopeia.c3.ca you can install it into your $HOME with:

~~~ {.bash}
$ /project/shared/fzf/install   # only once for your account (copies config files)
$ source ~/.fzf.bash            # in each new shell, or put this into your ~/.bashrc
~~~

# Other advanced bash topics

- exercise: write a function to compare two directories
- sed, awk
- arithmetics
- permissions
- how to control processes
- GNU_Parallel
- homebrew if enough Macs
