+++
date = "2018-07-21T09:30:45+01:00"
draft = false
tags = ["zsh", "linux", "howto"]
title = "How to turn off the beep/bell sound in zsh"

+++

A few weeks ago, I upgraded my Linux OS to [Ubuntu 18.04](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes) (from 16.04). I did a clean install and overall it's been nice - [some function keys that didn't work before](https://askubuntu.com/questions/830948/lenovo-ideapad-brightness-keys-not-generating-any-events-in-ubuntu-16-04-1/) now work great out of the box! However, one thing that's annoyed me is the \*beep\* sounds that the shell makes (both Bash and zsh).

<!--more-->Fixing this in Bash was easy enough with a quick search. I found [this page](https://linuxconfig.org/turn-off-beep-bell-on-linux-terminal) from 2013 that asked me to put `set bell-style none` in my `/etc/inputrc` or my `~/.inputrc`.

However, fixing this in zsh (which is my default shell) was harder than it should have been for some reason so I thought I'd document it ([tl; dr below](#tl-dr)). Surprisingly, all my searches weren't giving me anything. I did learn that [zsh doesn't read inputrc files](https://wiki.archlinux.org/index.php/zsh#Key_bindings) and so my above fix wouldn't work. I was mostly searching for how to turn off the beeps when tabbing through the autocomplete listing. I did find [this comment](https://unix.stackexchange.com/questions/73672/how-to-turn-off-the-beep-only-in-bash-tab-complete#comment244655_73672) on a Unix StackExchange question that said it was possible in zsh but didn't say how ([relevant xkcd](https://xkcd.com/979/)). Some days later when I searched for it again, I found some obscure reference to `LIST_BEEP` and `HIST_BEEP` on [this page from 2005](https://debian-administration.org/article/110/Removing_annoying_console_beeps#comment_3) and a suggestion to look them up in the zsh manual. The next time I searched for these things (query: "zbeep hist_beep list_beep"), the first result was the same page from 2005 (ouch, bad sign) and the second result was a gzipped PostScript version of the zsh manual. I downloaded it but did not open it just then (and found out later that I wasn't able to search inside the PostScript file anyway for some reason). And then finally, on a site called bash2zsh.com I found [this PDF](http://www.bash2zsh.com/zsh_refcard/refcard.pdf) which gave me the answer.

zsh options can be set with `setopt` and unset with `unsetopt`. The option `BEEP` toggles whether zsh beeps on all errors. The option `HIST_BEEP` toggles whether zsh beeps when going beyond history. And the option `LIST_BEEP` toggles beeps on ambigious completion.

I finally ended up adding the following lines to my `.zshrc`:
```zsh
# Turn off all beeps
unsetopt BEEP
# Turn off autocomplete beeps
# unsetopt LIST_BEEP
```

Oh, sweet silence.

## tl; dr
put `unsetopt BEEP` in your `.zshrc`