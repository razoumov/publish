<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [0. Prerequisites](#0-prerequisites)
- [1. Introduction to Unix shell](#1-introduction-to-unix-shell)
- [2. Wildcards, redirection to files, and pipes](#2-wildcards-redirection-to-files-and-pipes)
- [3. Loops](#3-loops)
- [4. Shell Scripts](#4-shell-scripts)
  - [Advanced topic: if statements](#advanced-topic-if-statements)
  - [Advanced topic: variables](#advanced-topic-variables)
  - [Advanced topic: functions](#advanced-topic-functions)
  - [Advanced topic: aliases](#advanced-topic-aliases)
- [5. Finding things](#5-finding-things)
  - [Advanced topic: running a command on the results of *find*](#advanced-topic-running-a-command-on-the-results-of-find)
- [6. Text manipulation (DH part: the invisible man)](#6-text-manipulation-dh-part-the-invisible-man)
- [Other advanced bash topics](#other-advanced-bash-topics)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

me                                       | students
---------------------------------------- | ----------------------------------------
(1) log in to socrative.com as a teacher | (1) log in to socrative.com as a student
(2) start a quiz, select 'bash quiz'     | (2) enter provided room name
(3) select 'teacher paced'               |
(4) disable student names                |
(5) start                                |

# 0. Prerequisites

Make sure you can start bash Unix shell.

* might use some form of https://sfu-os.syzygy.ca with github credentials
* personal lesson notes (this file) http://bit.ly/bashmd

# 1. Introduction to Unix shell

* note to self: type "carpentry"
* explain shell prompt (bash)
  * text-based interface to the OS (unlike a GUI): command interpreter + powerful programming language
  * a shell around the kernel (coconut analogy), along with utilities and applications
  * can glue together tools, utilities, compiled binaries, system calls and other scripts
  * follows the classic UNIX philosophy of breaking complex projects into simpler subtasks and chaining
	together components and utilities
  * bash is one of many Unix shell implementations
* why learn Unix shell: very powerful, great for automating workflows, necessary on bigger Unix systems

~~~ {.bash}
$ whoami   # explain what happens when you type a command
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

tab completion in bash

**Quiz 1:** If pwd displays /Users/thing, what will ls ../backup display?

**Quiz 2:** If pwd displays /Users/backup, and -r tells ls to display things in reverse order, what
command will display: pnas-sub/ pnas-final/ original/

**Quiz 3:** What does the command cd without a directory name do?

Creating things:
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

Moving things:
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

Copying things:
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

**Quiz 4:** Suppose that you created a .txt file in your current directory to contain a list of the
statistical tests you will need to do to analyze your data, and named it: statstics.txt. After creating
and saving this file you realize you misspelled the filename! You want to correct the mistake, which of
the following commands could you use to do so?

**Quiz 5:** What is the output of the closing ls command in the sequence shown below?

# 2. Wildcards, redirection to files, and pipes

* open http://bit.ly/bashfile in your browser, it'll download the file bfiles.zip
* unpack bfiles.zip to your Desktop; you should see ~/Desktop/data-shell

~~~ {.bash}
$ cd
$ cd Desktop
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

**Exercise:** build a single command to show the lenth of the longest (number of lines) file

**Exercise:** Try to to the difference between these two commands:
echo hello > test.txt
echo hello >> test.txt

**Quiz 6:** What code would you use to move all the .dat files into the analyzed sub-directory?

**Quiz 7:** Command to list the three files with the least number of lines.

# 3. Loops

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

**Quiz 8:** Output of a loop.

**Quiz 9:** Write a loop.

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
$ echo {a..z}    # can also use letters
$ echo {a..z}{1..10}   # this will produce 260 items
$ echo {a..z}{a..z}    # this will produce 676 items
$ seq 1 2 10      # step=2, so can use: for i in $(seq 1 2 10)
$ for ((i=1; i<=5; i++)) do echo $i; done   # can use C-style loops
~~~

# 4. Shell Scripts

~~~ {.bash}
$ cd ~/Desktop/data-shell/molecules
$ nano middle.sh
	#!/bin/bash         # this is called sha-bang; can be omitted for generic (bash/csh/tcsh) commands
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
    head -15 $1 | tail -5                     # $1 means the first argument to the script
$ bash middle.sh cubane.pdb
$ bash middle.sh propane.pdb
~~~

* head -15 "$1" | tail -5     # placing in double-quotes lets us pass filenames with spaces
* head $2 $1 | tail $3        # what will this do?
* $# holds the number of command-line arguments
* $@ means all command-lines arguments to the script (words in a string)

**Quiz 10:** script.sh in molecules Users/nelle/molecules.

**Exercise:** write a script that takes any number of filenames, e.g., "scriptName.sh cubane.pdb
propane.pdb", and does "ls -l" and "wc -l" on each file separating output from different files by an
empty line.

## Advanced topic: if statements

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

**Exercise:** write a script that complains when it does not receive arguments.

## Advanced topic: variables

We already saw variables that were specific to the script ($1, $@, ...). Variables can be used outside of
the scripts:

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

## Advanced topic: functions

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
    return 1
  fi
  dir=$RANDOM$RANDOM
  mkdir $dir
  cp $@ $dir
  echo look in the directory $dir
}
~~~

**Exercise:** write a function to swap two file names. Add a check that both files exist, before renaming
them.

**Exercise:** write a function archive() that takes a directory as an argument, packs it into a gzipped
tar archive and deletes the original directory.

**Exercise:** write the reverse function unarchive() that replaces a gzipped tar with a directory.

## Advanced topic: aliases

Aliases are one-line shortcuts/abbreviation to avoid typing a longer command, e.g.

~~~ {.bash}
$ alias cp='cp -i'
$ alias mv='mv -i'
$ alias ls='ls -aFh'
$ alias hi='history'
$ alias cedar='ssh -Y cedar.computecanada.ca'
$ alias weather='curl wttr.in/vancouver'
~~~

# 5. Finding things

~~~ {.bash}
$ cd ~/Desktop/data-shell/writing
$ more haiku.txt
~~~

First let's search for text in files:
~~~ {.bash}
$ grep not haiku.txt   # let's find all lines that contain the word 'not'
$ grep day haiku.txt   # now search for word 'day'
$ grep -w day haiku.txt   # search for a separate word 'day' (not 'today', etc.)
$ grep -w "is not" haiku.txt   # matching a phrase
$ grep -w today haiku.txt   # search for 'today'
$ grep -w Today haiku.txt   # search for 'Today'
$ grep -i -w today haiku.txt   # both upper and lower case 'today'
$ grep -n -i -w today haiku.txt   # -n prints out numbers the matching lines
$ grep -n -i -w -v the haiku.txt   # -v searches for lines that do not contain 'the'
$ man grep
~~~

More than two arguments to grep:
~~~ {.bash}
$ grep pattern file1 file2 file3   # all argument after the first one are assumed to be filenames
$ grep pattern *.txt   # the last argument will expand to the list of *.txt files
~~~

**Quiz 11:** grep command.

Now on to finding files:
~~~ {.bash}
cd ~/Desktop/data-shell/writing
$ find . -type d     # search for directories inside current directory
$ find . -type f     # search for files inside current directory
$ find . -maxdepth 1 -type f     # depth 1 is the current directory
$ find . -mindepth 2 -type f
$ find . -name haiku.txt   # finds one file
$ ls data   # shows one.txt two.txt
$ find . -name *.txt   # still finds one file -- why? answer: expands *.txt to haiku.txt
$ find . -name '*.txt'   # finds all three files -- good!
~~~

Let's wrap the last command into $() (called *command substitution*), as if it was a variable:

~~~ {.bash}
$ echo $(find . -name '*.txt')   # will print ./data/one.txt ./data/two.txt ./haiku.txt
$ ls -l $(find . -name '*.txt')   # will expand to ls -l ./data/one.txt ./data/two.txt ./haiku.txt
$ wc -l $(find . -name '*.txt')   # will expand to wc -l ./data/one.txt ./data/two.txt ./haiku.txt
$ grep elegant $(find . -name '*.txt')   # will look for 'elegant' inside all *.txt files
~~~

**Quiz 12:** combining grep and find.

**Exercise:** write a function 'countFiles()' that counts the number of files in each directory that you
pass to it.

## Advanced topic: running a command on the results of *find*

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

# 6. Text manipulation (DH part: the invisible man)

In this section we'll use two tools for text manipulation: *sed* and *tr*. Our goal is to calculate the
frequency of all dictionary words in the novel "The Invisible Man" by Herbert Wells (public
domain). First, let's apply our knowledge of grep to this text:

~~~ {.bash}
$ cd ~/Desktop/data-shell
$ ls   # shows wellsInvisibleMan.txt
$ wc wellsInvisibleMan.txt   # number of lines, words, characters
$ grep invisible wellsInvisibleMan.txt   # see the invisible man
$ grep invisible wellsInvisibleMan.txt | wc -l   # returns 60; adding -w gives the same count
$ grep -i invisible wellsInvisibleMan.txt | wc -l   # returns 176 (most are upper case)
~~~

Let's use the "stream editor" sed:
~~~ {.bash}
$ cat wellsInvisibleMan.txt | sed 's/[iI]nvisible/supervisible/g' > visibleMan.txt   # make him visible
$ sed 's/[iI]nvisible/supervisible/g' wellsInvisibleMan.txt > visibleMan.txt   # exactly the same command
$ man sed
$ grep supervisible visibleMan.txt   # see what happened to the now visible man
$ grep invisible visibleMan.txt
~~~

Now let's remove punctuation from the original file using "tr" (translate) command:
~~~ {.bash}
$ cat wellsInvisibleMan.txt | tr -d "[:punct:]" > invisibleNoPunct.txt
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

**Exercise:** write a script 'countWords.sh' that takes a text file as an argument, and returns the list
of its 100 most common words, i.e. the script should be used as 'bash countWords.sh
wellsInvisibleMan.txt'. The script should not leave any intermediate files. Or even better, write a
function 'countWords()'.

# Other advanced bash topics

- exercise: write a function to compare two directories
- sed, awk
- arithmetics
- permissions
- how to control processes
- GNU_Parallel
- tar, gzip/pigz
- homebrew if enough Macs
