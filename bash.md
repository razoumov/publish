<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [0. Prerequisites](#0-prerequisites)
- [1. Introduction to Unix shell](#1-introduction-to-unix-shell)
- [2. Wildcards, redirection to files, and pipes](#2-wildcards-redirection-to-files-and-pipes)
- [3. Loops](#3-loops)
- [4. Shell Scripts](#4-shell-scripts)
- [5. Finding things](#5-finding-things)
- [6. DH part: invisible man](#6-dh-part-invisible-man)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

me                                       | students
---------------------------------------- | ----------------------------------------
(1) log in to socrative.com as a teacher | (1) log in to socrative.com as a student
(2) start a quiz, select 'bash quiz'     | (2) enter provided room name
(3) select 'teacher paced'               |
(4) disable student names                |
(5) start                                |

## 0. Prerequisites

Make sure you can start bash Unix shell.

* might use some form of https://sfu-os.syzygy.ca with github credentials
* personal lesson notes (this file) http://bit.ly/bashmd

## 1. Introduction to Unix shell

* note to self: type "carpentry"
* explain shell prompt (bash)
  * text-based interface to the OS (unlike a GUI)
  * a shell around the kernel (coconut analogy), along with utilities and applications
* why learn Unix shell?
* explain file systems: part of OS managing files and directories

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

## 2. Wildcards, redirection to files, and pipes

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

## 3. Loops

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

## 4. Shell Scripts

~~~ {.bash}
$ cd ~/Desktop/data-shell/molecules
$ nano middle.sh
	head -15 octane.pdb | tail -5             # what does it do?
$ bash middle.sh   # the script ran!
~~~

Let's pass an arbitrary file to it:
~~~ {.bash}
$ nano middle.sh
	head -15 $1 | tail -5                     # $1 means the first argument to the script
$ bash middle.sh cubane.pdb
$ bash middle.sh propane.pdb
~~~

* head -15 "$1" | tail -5     # placing in double-quotes lets us pass filenames with spaces
* head "$2" "$1" | tail "$3"   # what will this do?
* "$@"   # means all command-lines parameters to the script

**Quiz 10:** script.sh in molecules Users/nelle/molecules.

<!-- **Exercise:** write a script that takes any number of filenames and does "ls -l" and "wc -l" on each file -->

## 5. Finding things

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

Let's wrap the last command into $(), as if it was a variable:
~~~ {.bash}
$ echo $(find . -name '*.txt')   # will print ./data/one.txt ./data/two.txt ./haiku.txt
$ ls -l $(find . -name '*.txt')   # will expand to ls -l ./data/one.txt ./data/two.txt ./haiku.txt
$ wc -l $(find . -name '*.txt')   # will expand to wc -l ./data/one.txt ./data/two.txt ./haiku.txt
$ grep elegant $(find . -name '*.txt')   # will look for 'elegant' inside all *.txt files
~~~

**Quiz 12:** combining grep and find.

## 6. DH part: invisible man

Now let's apply our knowledge of grep to a real text, "The Invisible Man" by Herbert Wells.

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

Now let's remove punctuation using "tr" (translate) command:
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
wellsInvisibleMan.txt'. The script should not leave any intermediate files.
