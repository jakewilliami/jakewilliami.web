---
layout: post
title: First Notes from a <code>UNIX</code>, <code>Git</code> and <code>R</code> Workshop
---

Though I had expressed in my first notebook, in late 2018 at nerve.fancy.chained, that I wanted to "learn terminal" (I obviously didn't know what "terminal" was...), and though I had used ample Excel over the summer of 2018--2019 working in Sue Schenk's lab., I didn't actually do any "proper" programming until relatively recently.

I only really went to it for <code>R</code>, but I am so glad I did.  I guess it was the push-start I needed.  After this, I talked to D. about <code>Git</code> and we ended up starting to make a Minecraft mod together.  The rest was history, I suppose.  I started using Julia in late September, 2019, after using Java with D. for a month or so.  Those were great times.  It was getting warmer, after August, and D. and I were still at university together.  We would come home and work on the mod, sometimes with a drink of Coke, and stay doing so till it was dark and we were hungry.  During that time, I did a <i>lot</i> of programming in <code>Bash</code>.  I created my [<code>scripts</code>](https://github.com/jakewilliami/scripts.git/) repository, which hosted all of these very buggy programmes written in bash.  It was in this playground where I also learned a little about other languages: namely <code>Perl</code>, <code>Ruby</code>, <code>Rust</code>, and <code>Python</code>.

Transcribed here are my initial, very shorthand notes from a workshop I went to at V.U.W. on <code>UNIX</code>, <code>Git</code>  and <code>R</code>.

---

## <code>UNIX</code>, <code>Git</code>, and <code>R</code> workship &mdash;: Day 1

### Bash (Bourne Again Shell)

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
>   - <code>sort -n lengths.txt | head -n 1</code>
> Sorts lengths.txt and then of those, prints the first line

  - <code>man</code> = manual
  - <code>cut -d</code> = separate each line
  - <code>-f</code> = give back fields
  - <code>-d</code> = delmiter
    - <code>-d ,</code> = comma as your delimiter
<code>man &lt;command&gt;</code> is your friend!
  - <code>uniq</code> = fills out adjacent matching lines (only unique commands)

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
> Also in this time I decided on a colour scheme for my terminal.  Choose something nice to look at that makes you comfortable.

#### Loops
  - <code>&gt;</code>: when pressed enter, this is telling you that you need more information to complete the command.
> ```bash
> cd ~/Desktop/Git_Unix_R_Workshop/Day_1/data-shell/creatures
> for filename in *.dat
> do
>     head -n 2 $filename | tail -n 1
> done
> ```

> "Don't name your variables <code>cheesecake</code>; you'll have no idea what they're doing when you come back to it in three weeks' time."

Note that the following code excerpts are *NOT* equivalent:
> ```bash
> for datafile in *.pdf; do ls *.pdb; done
> ```
> ```bash
> for datafile in *.pdf; do ls $datafile; done
> ```

  - <code>&lt;cmd&gt; && &lt;cmd 2&gt;</code> = run <code>&lt;cmd&gt;</code>; if successful, run <code>&lt;cmd 2&gt;</code>
  - <code>grep</code> = Global Regular Expression Print (finding text)
    - E.g.,
    - <code>grep &lt;word&gt; &lt;file&gt;</code>
    - <code>-i</code> = case <u>in</u>sensitive
    - <code>-w</code> = whole <u>w</u>ord
    - <code>-n</code> = line <u>n</u>umbers
    - <code>-v</code> = when it *doesn't* match (in<u>v</u>ert)
    - E.g.,
    - <code>grep -E '^.o' haiku.txt</code>
  - <code>find</code> = finding files
    - E.g.,
    - <code>find . -type [f&or;d]</code>
    - <code>find . -name '*.txt'</code>
    - <code>find . -type f -mtime -1 -user jakeireland</code> = finds files updated in past day but user <code>jakeireland</code>
  - <code>'$'&lt;name&gt;</code> = call a variable named <code>&lt;name&gt;</code>

### <code>git</code>
Version control system: track changes of file over time

Git has become popular in version control; and scalable!  Originally developed by Linux kernel guy.

> Note: since this workshop, I have learned that, the Linux guy (whose name is Linus) has two tools named after him: the Linux operating system (similar-sounding to his name), and <code>git</code>, because he is one (self-proclaimed; I am not insulting him)!

(Android = Linux kernel!  So yes, you *have* heard of Linux).

A kernel is the part of the operating system that talks to the device

We first need to configure out <code>git</code> environment, for our terminal to know our <code>git</code> credentials.  We run
```bash
git config --global user.name "<name>"
```

If colaborating, be carefule of differing operating systems; line endings can cause merge issues.

Now we want to say
```bash
git init
```
to initialise the repository.  

Running <code>ls -la</code> will give us a long listing and include hidden files within the directory (files beginning with <code>.</code>).  Some files need to be there but aren't useful to use (only the computer), so they often remain hidden.

We can run 
```bash
git status
```

to see the commits.  

Note: You can actually write a git repo anywhere you have access.

```bash
rm -rf .git
```
Will remove any trace of the git repository, <u>r</u>ecursively and <u>f</u>orcefully.

#### Tracking changes

If you are using the terminal-based text editor <code>nano</code>, <kbd>Ctrl</kbd> + <kbd>O</kbd> = write <u>O</u>ut = save.

```bash
git add <filename>
```
The previous command gives tracking file.  It tells git that yo u are "staging" that file.  Staging is the area where you're telling <code>git</code> to track.

  - <code>git commit</code> actually *tracks* the file.  Each commit has a unique hash code, often shortened because it can be, for ease of reference.
  - <code>git log</code> helps to see what you have committed in the past.
  - <code>nano &lt;file&gt;</code> = edit file
  - <code>git diff</code> = difference (looks at changes)
    - <code>-a /&lt;file&gt;</code> (initial)
    - <code>+</code> [added]
    - <code>b /&lt;file&gt;</code> (final)
  - <code>git diff --stages</code>
  - <code>git log -1</code> = last log notes
    - E.g., 
    - <code>git log --oneline --graph --all --decorate</code>

So what we do:
  1. We write some changes to our code
  2. <code>git log</code>
  3. <code>git diff</code>
  4. <code>git commit -m "commit message (something helpful to read later)"</code>
  5. <code>git log</code>

  - <code>mkdir &lt;dir&gt;</code> &rarr; <code>git add &lt;dir&gt;</code>
    - This does not track the directory!  Low key because it doesn't actually need to.
    - > I didn't elaborate on this at the time, but I beleive this is because it tracks all of the *files* within the directory.
  - <code>HEAD</code> = last change you committed
  - <code>git diff HEAD &lt;file&gt;</code> = last change
    - <code>git diff HEAD ~n &lt;file&gt;</code> = number of changes you want to go back to

But what if you have lots of these changes?  This is where hash codes are useful!:
  - <code>git diff &lt;hash code&gt;</code>

  - <code>git checkout -- mars.txt</code> = goes back to a previous version (similar to <code>rm</code>, so be careful!).
  - <code>git &lt;cmd&gt; --help</code> is equivalent to <code>man &lt;git cmd&gt;</code>.

Recall that <code>touch</code> creates an empty file.  <code>nano .gitignore</code> creates a hidden file (by <code>.</code>).

  - <code>git add -f &lt;file&gt;</code> to overwrite ignored files (in <code>.gitignore</code>).
  - <code>git status --ignored</code> tells us what we have ignored.

An important note: <code>git</code> &ne; GitHub.

  - <code>git</code> is the tool that we have been using
  - GitHub is a web service that allows people to collaborate through somewhere.  Other such web services include GitLab, BitBucket, etc.

> These are some notes on changing my bash prompt, and other things, after day 2...
> To change your prompt:
> > ```bash
> > sudo nano /etc/bashrc # or use your favourite text editor
> > ```
> > > This is actually wrong, in hindsight.  You change the <code>$HOME/.bashrc</code> file...
> Then type
> > ```bash
> > export PS1="<desired prompt>"
> > ```
> To list colour codes in their respective colours, I ran this loop:
> > ```bash
> > for colour in {1..255} # this is a sequence of integers from 1 to 255 inclusive
> >     do echo -en "\033[38;5;${colour}m38;5;${colour}\n"
> > done | column -x
> > ```
> Let me attempt to explain this.  `echo` prints "<what is in quotes>".  The `-n` option for echo tells the command not to print the trailing new line characters.  The `-e` option for echo tells the echo command that within the argument there is an escape code.  In our case, our escape code is `\033`, which in turn tells bash that whatever succeeding that, between `[` and `m`, should be ignored as a string.  In our case, we get that
> ```bash
> echo -en "\033[<u>           </u>m<u>           </u>"
> ```
> This is our text formatting code, which tells whatever follows after `m` and before the closing `"` what colour to be.  Finally, (&ast;) writes out the colour code [sic].  
> 
> We have this embedded in a loop for all numbers in 1&ndash;255.
> 
> The `\n` at the end of echo creates a new line. 
> 
> > Note: I now realise that the `-n` is redundant when we are adding `\n` anyway...
> 
> I also discovered the command
> ```bash
> tput
> ```
> for colours.  I'm not sure how this works with bold, but I have the following:
> > ```bash
> > for colour in {1..256}
> >     do echo -en "$(tput setaf ${colour})\$(tput setaf ${colour})\n"
> > done | column -x
> > echo
> > ```

### Git continued (with Wes Harrell, now)

GitHub allows you to share your changes with other people. 
