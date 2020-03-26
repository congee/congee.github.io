+++
title = "Use a Terminal Like a Boss"
slug = "use-a-terminal-like-a-boss"
date = 2020-03-25
[taxonomies]
tags = ["terminal", "cli", "command", "unix", "productivity"]
+++

Welcome to the rabbit hole! This is an introduction to for beginners to make the most of a terminal. Windows is covered. But, because Microsoft ditched its command line (CLI) battle ground in favor of GUI until a few years ago, this article will shift to *nix environment eventually. Generally, programmers that pursue productivity should be should be proficient at *nix. By no means am I am criticizing Windows. You know, "Finally, it's the year of desktop Linux" has been echoing for 20 years. 🙄 

Hackers are obsessed with abstraction to eliminate repetitive tasks. How could we stand with manually typing everything?

## A Tale of Two Editing Modes
Though the holy wars between two editors Vim and Emacs are fading away, long live their spirit.

Because a single line editing is so efficient in emacs, people made a library called _readline_. Just pay a little more heed you will find it is everywhere. You can use in most application in macOS, interactive cli tools such as ptpython, mycli, and pgcli. There is even a tool with-readline to empower you on tools that do not natively support readline style key bindings. I just list some frequent shortcuts. Youo can find a comprehensive list of shortcuts on [wikipedia](https://en.wikipedia.org/wiki/GNU_Readline#Emacs_keyboard_shortcuts) or its [official docs](https://tiswww.cwru.edu/php/chet/readline/readline.html).

- Ctrl-a moves the cursor to the line start
- Ctrl-w delete a word backwords
- Ctrl-d delete the current character forward
- Ctrl-d also send an EOF, meaning exit when the line is empty
- Ctrl-u clears the content before the cursor
- Ctrl-p takes you to the previous command
- Ctrl-n takes you to be next command
- Alt-f move the cursor forward one word
- Alt-b move the cursor backward one word
- Alt-. insert the last argument of the previous command

Also, there is a cheat sheet¹ for you to remember these key bindings.
\
\
![cheatsheet](/images/readline.cheatsheet.png "Readline Cheat Sheet")

Luckily, although it's not easy to use this emacs style key bindings on cmd.exe, [clink](https://mridgers.github.io/clink) is made available.

In short, the vim mode is best for multi-line editing, while the emacs mode makes you efficient in line editing.


## Auto Completion
Do you really type all the path to change into a directory or look for a file? e.g. `command C:\Users\username\i\am\a\very\deep\folder\and\then\the\filename.ext?`. Surely no. You would stop at `comm` and press TAB to complete the `command`, then type `C:\Users\usern` to get `command C:\Users\username\`. Yeah, a lot faster than typing everything.

Can it be faster? Yes. Modern customizable shells such as bash and fish can complete the whole path for you easily with the help of plugins. Zsh just has this builtin. Just typing `command /U/u/i/d/f/file` followed by tab, you get the whole command line completed.

What if it is a previous command that you have already typed? I believe you the smart know Ctrl-p does the trick if the command is the just the previous one you have entered. You may press Ctrl-p multiple times if it's not the last one.

What do you do if it's buried in thousands of command lines you've entered? Search the history. Most readline applications has Ctrl-r that supports fuzzy matching.

I bet you are not capable of rememrbing command options. And that could be [distraous](https://xkcd.com/1168/). We complete command line options as well.

There is an example that the shell tries to complete the commit that I would like to fixup.
``` shell
~/dev/haskell-ide-engine$ git commit --fixup=
Completing recent commit object name
ee5b98c6  -- [HEAD]    Merge pull request #1684 from jneira/changelog (3 weeks ago)
5ba1655e  -- [HEAD^]   Fix typos in README. Add hint to speed up HIE compilation (3 weeks ago)
a043333c  -- [a043333c] Correct pr url and formatting (3 weeks ago)
02815bb0  -- [HEAD^^]  Merge pull request #1681 from alanz/prepare-1.2 (3 weeks ago)
81acf2ab  -- [81acf2ab] Update Changelog.md (3 weeks ago)
cb2cab09  -- [cb2cab09] Update Changelog.md (3 weeks ago)
1072b60b  -- [1072b60b] Update Changelog.md (3 weeks ago)
fe0b2fea  -- [fe0b2fea] Merge remote-tracking branch 'haskell/master' into prepare-1.2 (3 weeks ago)
4e34be79  -- [HEAD~3]  Merge pull request #1679 from jneira/hask-src-exts-1.22 (3 weeks ago)
ba8ee55c  -- [ba8ee55c] Avoid unliftio-core >= 0.2 (3 weeks ago)
02c1d248  -- [02c1d248] Preparing 1.2 release (3 weeks ago)
2c279e3e  -- [2c279e3e] Add hsimport as source repo package (3 weeks ago)
59dc5b20  -- [59dc5b20] Remove unnecessary floskell source-repo-pkg (3 weeks ago)
d5162dee  -- [d5162dee] Use master versions of hsimport and floskell (3 weeks ago)
de19fbfb  -- [de19fbfb] Set haskell-src-exts lower bound to 1.22 (3 weeks ago)
a4f726de  -- [HEAD~4]  Merge pull request #1678 from alanz/bump-resolvers (3 weeks ago)
7538134e  -- [7538134e] Stick with older unliftio-core (3 weeks ago)
0718c12c  -- [0718c12c] Bump resolvers and hlint to 2.2.11 (3 weeks ago)
c584f982  -- [HEAD~5]  Merge pull request #1665 from jneira/azure-macos (4 weeks ago)
7e45d349  -- [7e45d349] Not share jobs arg between shake and build (4 weeks ago)
Completing commit to be amended
1.2            HEAD           master         origin/HEAD    origin/master
```

None of the shells designed for Windows is able to provide you with these powerful functionalities as far as I know.


## Jummping Around
Imagine you have to constantly switching between two directories `~/src/third_party/package/library/` and `~/src`. It is easy to go back to the short one by just entering `cd` or `cd ~` to the home directory and then `cd src`. But, how to you change into the long directory quickly? Treat these directories like a stack! pushd/popd is supported on both cmd.exe and *nix shells.

`pushd ~/src/third_party/package/library/` takes you to the long directory. `popd` takes you to the previous directory. Mainstream *nix shells support them natively. You don't even need to use pushd/popd expecitly. `cd` just works `cd -` works the same as popd. You can print the stack using `dirs` or `echo $dirstack` to print the stack stored as $dirstack.

What if you have multiple directories, say 5, to jump among? In zsh, `cd -` followed by TAB will let you choose one of the directories in the $dirstack. I am lazier. I just enter `z library` to jump to the long one. There is a well known tool called [autojump](https://github.com/wting/autojump). But people who could not tolerate the slow Python implementation developed their own solutions. [z.lua](https://github.com/skywind3000/z.lua) looks prominent.


## Fuzzy Matchng Everytihng
Typing a few keywords in Google takes you to almost whatever your want, it's also possible in both the desktop and cli environments. In the last decade, quick launchers have been quite prevailing. Alfred hits its hey day for Mac users. Even Apple has copied its fuzzy searching idea into Spotlight on macOs and iOS. On Windows you could find an arbitraty file on disk in milliseconds as long as the filesystem is NTFS, way faster than Windows Key + search. To tell you a secret, I use Listary rather than Everything.

There are many fuzzy matching tools for cli. The most popular one is [fzf](https://github.com/junegunn/fzf). It is powerful combined with other commands via unix pipes. E.g. I would like to print the first 5 lines of a file of which "Rename" is a part of the filename. Yeah, there is a sophisticated command for it `find . -type f -name "*Rename*" -exec head -5 {} \;`. How about his one? ![fzf](/images/screencast.fzf.gif)

The idea has recently gained popularity. People even coined a term [CLUI]('https://blog.repl.it/clui').


## I Have A Nickname
Greed is good. You might want to sequeeze the terminal's ability by reducing more repetitive key strokes. Ctrl-r to restore your good ol' `git log --left-right --graph --oneline --cherry` is not satisfying anymore. Let alone, it returns you some arguments you do not want. Well, make it an alias as `git-cmp`. `git cmp master..develop` is so simple. Here is the screenshot for `git tree`: ![git-tree](/images/git-tree.png). Some shells provide alias such as `alias ll="ls -l"` or `alias ls="ls --color=auto"` by default.


## Summary
To sum up, hackers just like to automate to improve productivity. I did not mention PowerShell, because it's designed for scripting than used as a shell on daily basis. Its ecosystem is not comparable to *nix shell, either.

## References
1. Brian Ward, _[How Linux Works, 2nd Edition](https://nostarch.com/howlinuxworks2)_
