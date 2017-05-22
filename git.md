<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [0. Prerequisites](#0-prerequisites)
- [1. Collaboration problem](#1-collaboration-problem)
- [2. Setup](#2-setup)
- [3. First edit](#3-first-edit)
- [4. Second edit](#4-second-edit)
- [5. Explore history](#5-explore-history)
- [6. Ignore things](#6-ignore-things)
- [7. Remotes](#7-remotes)
- [8. EXERCISE collaborating: get into pairs](#8-exercise-collaborating-get-into-pairs)
- [9. EXERCISE conflicts](#9-exercise-conflicts)
- [10. Putting together a simple web page](#10-putting-together-a-simple-web-page)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Cold and dry, but everything is my favourite color. The two moons may be a problem for Wolfman. But the
Mummy will appreciate the lack of humidity.

## 0. Prerequisites

Make sure (1) you have git in bash and (2) get an account with github.

* personal lesson notes http://bit.ly/gitmd

## 1. Collaboration problem

* File locking has its downsides.
* Better use automatic version control!
* Possible uses: collaboration, keep track of changes and logs, backup, transfer files between systems.

Types of repositories in git:
* regular git repository -- we'll study only these
* bare repository
* git bundle

## 2. Setup

~~~ {.bash}
$ git config --global user.name "Alex Razoumov"
$ git config --global user.email "alex.razoumov@westgrid.ca"
$ git config --global color.ui "auto"
$ git config --global core.editor "nano -w"
$ git config --list
~~~

**Exercise:** configure git on your laptop.

## 3. First edit

~~~ {.bash}
$ cd   # we'll start from the home directory
$ mkdir planets
$ cd planets
$ git init    # only once for each project and its subdirectories
$ git status
$ nano mars.txt   # add the first line to mars.txt
$ git add mars.txt
$ git commit -m "started notes on mars"
$ git log
~~~

**Question:** how would you undo 'git init' inside a subdirectory?

## 4. Second edit

~~~ {.bash}
$ nano mars.txt   # add a second line to mars.txt
$ git status
$ git diff
$ git commit -m "continued mars notes"   # likely will be getting an error message
~~~

Need to stage changes before committing! There are two solutions:

* either
~~~ {.bash}
$ git add mars.txt   # add mars.txt again to the staging area
$ git diff   # difference between working copy and staging area
$ git diff --staged   # difference between staging area and repository
$ git commit -m "continued mars notes"
~~~
		   
* or
~~~ {.bash}
$ git commit -a -m "continued mars notes"   # automatically add files that are already being tracked
~~~

**Exercise:** create bio.txt, add 3 lines, commit to the repository, modify first line, add line 4,
display changes, commit to the repository.

## 5. Explore history

~~~ {.bash}
$ git log
~~~

HEAD is the latest version in the repository, HEAD~1 is the previous, ...

~~~ {.bash}
$ git diff HEAD~1 mars.txt
$ git diff HEAD~2 mars.txt
$ git diff specificVersionNumber mars.txt   # version numbers have 40 characters
$ git diff specificVersionNumberFirstFewUniquesCharacters mars.txt
~~~

Accidentally removed or changed mars.txt. How do we restore it?

~~~ {.bash}
$ git checkout HEAD mars.txt   # restore latest from the repository
$ git checkout specificVersionNumber mars.txt   # restore specific version from the repository
~~~

## 6. Ignore things

~~~ {.bash}
$ mkdir results
$ touch a.dat b.dat c.dat results/a.out results/b.out
$ git status
$ nano .gitignore   # add *.dat and results, one per line
$ git status
$ git add .gitignore
$ git commit -m "Add the ignore file"
$ git status
$ git status --ignored   # show the ignored files
~~~

## 7. Remotes

Why remotes?

There are many different types of remotes: departmental git server, usb key, git bundle, github.

Log into github.com, create a new repository called "planets", copy the repository URL.

~~~ {.bash}
$ git remote add origin https://github.com/razoumov/planets.git # make the github repository a remote
$ git remote
$ git remote -v
$ git push origin master   # to upload to remote, should ask for password.
~~~

If github is confused about your identity, can always specify your github username by hand:
~~~ {.bash}
git remote set-url origin https://razoumov@github.com/razoumov/planets.git
~~~

Check your github repository in the web browser.

## 8. EXERCISE collaborating: get into pairs

Public github repositories provide read access to everyone, but only the owner can write to a
repository. However, the owner can give write access to his/her collaborator.

Open your github repository for writing to your partner: your profile -> repository -> settings ->
collaborators and add their github name.

Everyone, please clone your partner's github repository:
~~~ {.bash}
$ cd   # change to your home directory
$ pwd   # should not show 'planets'
$ git clone https://github.com/yourPartner/planets.git partner
$ cd partner
$ nano question.txt   # make some changes to this file, e.g., ask a question
$ git commit -m -a "describe your changes"
$ git push origin master
~~~

Then each one of you will download your partner's edits:

~~~ {.bash}
$ cd ~/planets
$ git pull origin master
$ cat question.txt    # you should see your partner's edits
~~~

## 9. EXERCISE conflicts

Two partners: pick one of the two repositories, synchronize them.
Next, each add a line at the end of the same file.
First, partner 1 pushes to the repository.
Next, partner 2 tries to push to the repository ... and gets an error.
Partner 2 needs to resolve conflict before he can push.

~~~ {.bash}
$ git pull origin master
$ cat contestedFile.txt
$ nano contestedFile.txt   # edit to resolve conflict: it's up to us what to do
$ git -a -m "resolved"
$ git push origin master
~~~

## 10. Putting together a simple web page

We'll now write some static pages with GitHub pages and Jekyll, using Jekyll Now as a template.

Go to https://github.com/barryclark/jekyll-now, click Fork at the top-right. Once forked, go to the
repository settings and rename it to username.github.io. After the first "git commit + git push" (below)
you'll be able to view your new page at http://username.github.io. Also on mobile! Now let's customize
it!

~~~ {.bash}
$ cd   # change to your home directory
$ pwd   # should not show 'planets' or any other directory containing a git repository
$ git clone https://github.com/razoumov/razoumov.github.io.git
$ cd razoumov.github.io
$ nano _config.yml   # set name, description
$ git status
$ git commit -am "changed the name"
$ git push   # after this reload the website
~~~

How do we add a new post?
~~~ {.bash}
$ cd _posts
$ cp 2014-3-3-Hello-World.md 2016-01-17-test.md
$ nano !$
$ git status
$ git add !$
$ git commit -m "added a post"
$ git push   # after this reload the website
~~~

For other themes, scroll down to "Other forkable themes" in README.md -- each template is a separate
repository.

**Exercise 1:** delete the "You're up and running!" post.

**Exercise 2:** write few words about yourself in ../about.md in upload them to your website.

<!-- intro to pull requests https://github.com/crazapplejuice/flags/blob/master/CONTRIBUTING.md -->
