---
layout: post
title: First Notes from a <code>UNIX</code>, <code>Git</code> and <code>R</code> Workshop
---

Though I had expressed in my first notebook, in late 2018 at nerve.fancy.chained, that I wanted to "learn terminal" (I obviously didn't know what "terminal" was...), and though I had used ample Excel over the summer of 2018--2019 working in Sue Schenk's lab., I didn't actually do any "proper" programming until relatively recently.

I only really went to it for <code>R</code>, but I am so glad I did.  I guess it was the push-start I needed.  After this, I talked to D. about <code>Git</code> and we ended up starting to make a Minecraft mod together.  The rest was history, I suppose.  I started using Julia in late September, 2019, after using Java with D. for a month or so.  Those were great times.  It was getting warmer, after August, and D. and I were still at university together.  We would come home and work on the mod, sometimes with a drink of Coke, and stay doing so till it was dark and we were hungry.  During that time, I did a <i>lot</i> of programming in <code>Bash</code>.  I created my [<code>scripts</code>](https://github.com/jakewilliami/scripts.git/) repository, which hosted all of these very buggy programmes written in bash.  It was in this playground where I also learned a little about other languages: namely <code>Perl</code>, <code>Ruby</code>, <code>Rust</code>, and <code>Python</code>.

Transcribed here are my initial, very shorthand notes from a workshop I went to at V.U.W. on <code>UNIX</code>, <code>Git</code>  and <code>R</code>.

---
## <code>UNIX</code>, <code>Git</code>, and <code>R</code> workship &mdash;: Day 1

Bash is programmable!
  - The GUI is intuitive and user-friendly
     - Bash can do the same things, but automatically, > 1,000 times.
  - <code>ls</code> = listing command
  - The file system arranges things in hierarchy
  - <code>pwd</code> = print working directory
Note: the <code>ls</code> command, unless specified otherwise, will list the contents of the present working directory
  - <code>cd</code> = takes you back to your home directory
  - <code>-</code> = tells it is an option for a command (also called a "flag")
  - <code>cd &#x3008;path&#x3009;</code> = sets working directory
  - <code>ls &#x3008;path&#x3009;</code> = lists contents of path
  - <code>man ls</code> = find the <code>man</code>(ual) page
    - Press q to quit.
    - Some systems may require you to use <code>ls --help</code>.
  - <code>ls -l</code> = long listing (in bytes)
  - <code>ls -l -h</code> &equiv; <code>ls -lh</code> = human readible form
  - <code>ls ~</code> &equiv; <code>ls $HOME</code> &equiv <code>ls</code>
Paths usually start with <code>/</code>
  - <code>cd</code> = change directory
  - <code>cd ..</code> = moves to parent directory
  - <code>cd ../../</code> = parent of parent
  - <code>ls -a</code> = list all (including hidden files)
  - <code>.</code> = shortcut for current directory
  - <code>~/&#x3008;path&#x3009;</code> = expands to home directory + path
  - <code>cd -</code> = previous working directory

#### Relative vs. absolute paths
  - <code>mkdir</code> = make directory
  - <code>mv</code> = move
  - <code>touch</code> = make file
  - <code>cp</code> = copy
  - <code>rm</code> = remove
  - <code>rm -i</code> = are you sure you want to delete?
  - <code>more &#x3008;file&#x3009;</code> = shows file
  - '<code>&ast;</code>' = names ending with <code>&ast;&#x3008;name end&#x3009;</code>
  - '<code>?</code>' = single character wild-card
  - <code>clear</code> = clear terminal (note: can still scroll up unless configured otherwise)

> Note:
> A note on spaces in path names: they need to be "escaped" using a backslash:
> ```bash
> cd Victoria\ University/
> ```

  - <code>open &lt;path&gt;</code> = opens a path
  - <code>wc</code> = word count
  - <code>wc -c</code> = characters
  - <code>wc -l &ast;.pdb > lengths.txt</code> = makes lengths file and puts lengths of pdb files in there
  - <code>cat</code> = concatenate [similar to <code>more</code>]
  - '<code>&gt;</code>' = redirects output into file
  - '<code>&gt;&gt;</code>' _appends_ file (adds to it instead of overwriting it)
  - <code>head</code> = prints n head lines of each file
  - <code>tail</code> = ... bottom lines
  - <code>head 2 &ast;.pdb</code> = first 2 lines of all pdb files
  - '<code>|</code>' = "pipe" (kind of like subset)

> E.g.
> <code>sort -n lengths.txt | head -n 1</code><br></br>
> Sorts lengths.txt and then of those, prints the first line

  - <code>man</code> = manual
  - <code>cut -d</code> = separate each line
  - <code>-f</code> = give back fields
  - <code>-d</code> = delmiter

> Note:
> After day 1 of the tutorial, I actually went home and changed my bash prompt.  The following code, I put in my `.bashrc`:
> ```bash
> # get current branch in git repo
> function parse_git_branch() {
> 	BRANCH=$(git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/\1/')
> 	if [ ! "${BRANCH}" == "" ]
> 	then
> 		STAT=$(parse_git_dirty)
> 		echo "[${BRANCH}${STAT}]"
> 	else
> 		echo ""
> 	fi
> }
>
> # get current status of git repo
> function parse_git_dirty {
> 	status=$(git status 2>&1 | tee)
> 	dirty=$(echo -n "${status}" 2> /dev/null | grep "modified:" &> /dev/null; echo "$?")
> 	untracked=$(echo -n "${status}" 2> /dev/null | grep "Untracked files" &> /dev/null; echo "$?")
> 	ahead=$(echo -n "${status}" 2> /dev/null | grep "Your branch is ahead of" &> /dev/null; echo "$?")
> 	newfile=$(echo -n "${status}" 2> /dev/null | grep "new file:" &> /dev/null; echo "$?")
> 	renamed=$(echo -n "${status}" 2> /dev/null | grep "renamed:" &> /dev/null; echo "$?")
>	  deleted=$(echo -n "${status}" 2> /dev/null | grep "deleted:" &> /dev/null; echo "$?")
>   bits=''
> 	if [ "${renamed}" == "0" ]; then
> 		bits=">${bits}"
> 	fi
> 	if [ "${ahead}" == "0" ]; then
> 		bits="*${bits}"
> 	fi
>	 if [ "${newfile}" == "0" ]; then
> 		bits="+${bits}"
> 	fi
> 	if [ "${untracked}" == "0" ]; then
> 		bits="?${bits}"
> 	fi
> 	if [ "${deleted}" == "0" ]; then
> 		bits="x${bits}"
> 	fi
>	 if [ "${dirty}" == "0" ]; then
> 		bits="!${bits}"
>	 fi
> 	if [ ! "${bits}" == "" ]; then
> 		echo " ${bits}"
> 	else
> 		echo ""
> 	fi
> }
>
> # make prompt pretty
> PS1="\n\[\033[0;31m\]\342\224\214\342\224\200\$()[\[\033[1;38;5;2m\]\u\[\033[0;1m\]@\033[1;33m\]\h: \[\033[1;34m\]\W\[\033[1;33m\]\[\033[0;31m\]]\[\033[0;32m\] \[\033[1;33m\]\`parse_git_branch\`\[\033[0;31m\]\n\[\033[0;31m\]\342\224\224\342\224\200\342\224\200\342\225\274 \[\033[0;1m\]\$\[\033[0;38m\] "
> export PS1
> ```

