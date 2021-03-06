Colours in terminal
==================
It's a common confusion about terminal colours... Actually we have this:
* plain ascii
* ansi escape codes (16 colour codes with bold/italic and background)
* 256 colour palette (216 colours + 16 ansi + 24 gray) (colors are 24bit)
* 24bit true colour ("888" colours (aka 16 milion))

```
printf "\x1b[${bg};2;${red};${green};${blue}m\n"
```

The 256 colour palete is configured at start, and it's a 666 cube of
colours, each of them defined as a 24bit (888 rgb) colour.

This means that current support can only display 256 different colours
in the terminal, while truecolour means that you can display 16 milion
different colours at the same time.

Truecolour escape codes doesnt uses a colour palete. It just specifies the
colour itself.

Here's a test case:
```
printf "\x1b[38;2;255;100;0mTRUECOLOR\x1b[0m\n"
```
* or https://raw.githubusercontent.com/JohnMorales/dotfiles/master/colors/24-bit-color.sh
* or http://github.com/robertknight/konsole/tree/master/tests/color-spaces.pl
* or https://git.gnome.org/browse/vte/tree/perf/img.sh?h=vte-0-36
* or just run this:
```sh
awk 'BEGIN{
    s="/\\/\\/\\/\\/\\"; s=s s s s s s s s;
    for (colnum = 0; colnum<77; colnum++) {
        r = 255-(colnum*255/76);
        g = (colnum*510/76);
        b = (colnum*255/76);
        if (g>255) g = 510-g;
        printf "\033[48;2;%d;%d;%dm", r,g,b;
        printf "\033[38;2;%d;%d;%dm", 255-r,255-g,255-b;
        printf "%s\033[0m", substr(s,colnum+1,1);
    }
    printf "\n";
}'
```

Keep in mind that it is possible to use both ';' and ':' as parameters delimiter.

According to Wikipedia[1], this is only supported by xterm and konsole.

[1] https://en.wikipedia.org/wiki/ANSI_color

Currently, there is no support for the 24-bit colour descriptions in the terminfo/termcap database and utilites.
See the discussion thread here: https://lists.gnu.org/archive/html/bug-ncurses/2013-10/msg00007.html

Detection
=========

There's no reliable way, and ncurses/terminfo's maintainer expressed he has no intent on introducing support. S-Lang author added a check for $COLORTERM containing either "truecolor" or "24bit" (case sensitive). In turn, [VTE](https://bugzilla.gnome.org/show_bug.cgi?id=754521), [Konsole](https://bugs.kde.org/show_bug.cgi?id=371919) and [iTerm2](https://gitlab.com/gnachman/iterm2/issues/5294) set this variable to "truecolor" (it's been there in VTE for a while, it's relatively new and maybe still git-only in Konsole and iTerm2).

This is obviously not a reliable method, and is not forwarded via sudo, ssh etc. However, whenever it errs, it errs on the safe side: does not advertise support whereas it's actually supported. App developers can freely choose to check for this same variable, or introduce their own method (e.g. an option in their config file), whichever matches better the overall design of the given app. Checking $COLORTERM is recommended though, since that would lead to a more unique desktop experience where the user has to set one variable only and it takes effect across all the apps, rather than something separately for each app.

Here are terminals discussions:
==============================

Now **supporting** truecolour
----------------------------

* [st](http://st.suckless.org/) (from suckless) [delimeter: semicolon] -  http://lists.suckless.org/dev/1307/16688.html
* [konsole](http://kde.org/applications/system/konsole/) [delimeter: colon, semicolon] - https://bugs.kde.org/show_bug.cgi?id=107487
* [iterm2](http://www.iterm2.com/) [delimeter: colon, semicolon] - since v3 version
* [qterminal](https://github.com/qterminal/qterminal) [delimeter: semicolon] - https://github.com/qterminal/qterminal/issues/78
* [alacritty](https://github.com/jwilm/alacritty) [delimeter: semicolon] - written in Rust
* [kitty](https://github.com/kovidgoyal/kitty) [delimeter: semicolon] - uses OpenGL
* [cool-retro-term](https://github.com/Swordfish90/cool-retro-term) [delimeter: semicolon]
* [Black Screen](https://github.com/shockone/black-screen) [delimeter: semicolon] - crossplatform, HTML/CSS/JS-based
* [Hyper.app](https://hyper.is/) [delimeter: semicolon] - crossplatform, HTML/CSS/JS-based (Electron)
* [Tera Term](http://en.sourceforge.jp/projects/ttssh2/) [delimeter: colon, semicolon] - **Windows platform**
* [ConEmu](https://github.com/Maximus5/ConEmu) [delimeter: semicolon] - **Windows platform**
* [FinalTerm](http://finalterm.org/) [delimeter: semicolon] - [abandoned](http://worldwidemann.com/finally-terminated/), iTerm2 [borrowing it's ideas and features](http://iterm2.com/shell_integration.html).
* [MacTerm](https://github.com/kmgrant/macterm) [delimeter: semicolon] - **Mac OS X platform**
* [mintty](https://mintty.github.io/) [delimeter: semicolon] **Cygwin and MSYS/MSYS2** since commit https://github.com/mintty/mintty/commit/43f0ed8a46c6549cb9a3ea27abc057b5abe13bdb (2.0.1 release) - **Windows platform**
* Windows 10 bash console, since [Windows Insiders build 14931](https://blogs.msdn.microsoft.com/commandline/2016/09/22/24-bit-color-in-the-windows-console/)
* all [libvte](http://ftp.gnome.org/pub/GNOME/sources/vte/) based terminals (since 0.36 version) [delimeter: colon, semilocon] -  https://bugzilla.gnome.org/show_bug.cgi?id=704449
    * **libvte**-based [Gnome Terminal](https://help.gnome.org/users/gnome-terminal/stable/)
    * **libvte**-based [sakura](http://www.pleyades.net/david/projects/sakura)
    * **libvte**-based [xfce4-terminal](http://docs.xfce.org/apps/terminal/start) - since [0.6.90](https://github.com/xfce-mirror/xfce4-terminal/releases/tag/xfce4-terminal-0.6.90) release, if compiled with GTK+3
    * **libvte**-based [Terminator](http://gnometerminator.blogspot.com/p/introduction.html) - since [1.90](https://launchpad.net/terminator/+announcement/14358) release
    * **libvte**-based [Terminix](https://github.com/gnunn1/terminix) - written in D. Similar user interface as for Terminator.
    * **libvte**-based [Lilyterm](http://lilyterm.luna.com.tw/) - since commit https://github.com/Tetralet/LilyTerm/commit/72536e7ba448ad9ef1126ce45fbde3a3407a271b
    * **libvte**-based [ROXTerm](http://roxterm.sourceforge.net/)
    * **libvte**-based [evilvte](http://www.calno.com/evilvte/) - no release yet, version from git https://github.com/caleb-/evilvte
    * **libvte**-based [Termit](https://github.com/nonstop/termit)
    * **libvte**-based [Termite](https://github.com/thestinger/termite)
    * **libvte**-based [Tilda](https://github.com/lanoxx/tilda)
    * **libvte**-based [tinyterm](https://code.google.com/p/tinyterm)
    * **libvte**-based [Pantheon Terminal](https://launchpad.net/pantheon-terminal)
    * **libvte**-based [lxterminal](http://sourceforge.net/projects/lxde) - with **--enable-gtk3** configure flag.
    * **libvte**-based [mlterm](http://mlterm.sourceforge.net) - with **--with-gtk=3.0** configure flag

But there are bunch of libvte-based terminals for GTK2 so they are listed in the another section.

Also, while this one is not exactly a terminal, but a terminal replayer, it still worth mentioning:

  * [asciinema](http://asciinema.org) player: https://github.com/asciinema/asciinema-player

Parsing ANSI colour sequences, but approximating them to 256 palette
-------------------------------------------------------------------

* xterm (though doing it wrong: "it uses nearest colour in RGB colour space, with a usualfalse assumption about orthogonal axes")
* [urxvt aka rxvt-unicode](http://software.schmorp.de/pkg/rxvt-unicode.html) -  since [Revision 1.570](http://cvs.schmorp.de/rxvt-unicode/src/command.C?revision=1.570&view=markup&sortby=log&sortdir=down) http://lists.schmorp.de/pipermail/rxvt-unicode/2016q2/002261.html
* linux console (since v3.16): https://github.com/torvalds/linux/commit/cec5b2a97a11ade56a701e83044d0a2a984c67b4


Note about colour differences: a) RGB axes are not orthogonal, so you cannot use sqrt(R^2+G^2+B^2) formula, b) for colour differences there is more correct (but much more complex) [CIEDE2000](http://en.wikipedia.org/wiki/Color_difference#CIEDE2000) formula (which may easily blow up performance if used blindly) [2].

[2] https://github.com/neovim/neovim/issues/793#issuecomment-48106948

Terminal multiplexers
---------------------

* [tmux](http://tmux.github.io/) - starting from version 2.2 (support since [427b820...](https://github.com/tmux/tmux/commit/427b8204268af5548d09b830e101c59daa095df9))
* [screen](http://git.savannah.gnu.org/cgit/screen.git/) - has support in 'master' branch, need to be enabled (see 'truecolor' option)
* [pymux](https://github.com/jonathanslenders/pymux) - tmux clone in pure Python (to enable truecolour run pymux with `--truecolor` option)
* [dvtm](https://github.com/martanne/dvtm) - not yet supporting True Colour https://github.com/martanne/dvtm/issues/10

**NOT supporting** truecolour
----------------------------

* [Terminology](https://www.enlightenment.org/about-terminology) (Enlightenment) - https://phab.enlightenment.org/T746
* [mrxvt](https://sourceforge.net/projects/materm) (looks abandoned) - https://sourceforge.net/p/materm/feature-requests/41/
* [aterm](http://www.afterstep.org/aterm.php) (looks abandoned) - https://sourceforge.net/p/aterm/feature-requests/23/
* [fbcon](https://www.kernel.org/doc/Documentation/fb/fbcon.txt) (from linux kernel) - https://bugzilla.kernel.org/show_bug.cgi?id=79551
* FreeBSD console - https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=191652
* [yaft](https://github.com/uobikiemukot/yaft) framebuffer terminal - https://github.com/uobikiemukot/yaft/issues/12
* [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/) (patched version [3] {xterm-like approximation to 256 colors} and [4] {real true colors} available) - **Windows platform**
* libvte and GTK2 - based:
     * **libvte**-based [GTKTerm2](http://gtkterm.feige.net/)
     * **libvte**-based [stjerm](https://github.com/stjerm/stjerm) (looks abandoned) - https://github.com/stjerm/stjerm/issues/39

[3] You can download patched version here https://github.com/rdebath/PuTTY

[4] You can download patched version here https://github.com/halcy/PuTTY

Here are another console programs discussions:
============================================

Supporting True Colour:

* [mc](http://www.midnight-commander.org) - since [682a5...](http://www.midnight-commander.org/changeset/682a5116edd20b8ba81743a1f7495c883b0ce644). See also [ticket #3724](http://www.midnight-commander.org/ticket/3724) for truecolor themes.
* [irssi](https://github.com/irssi/irssi) - since [PR #48](https://github.com/irssi/irssi/pull/48)
* [neovim](https://github.com/neovim/neovim) - since commit [8dd415e887923f99ab5daaeba9f0303e173dd1aa](https://github.com/neovim/neovim/commit/8dd415e887923f99ab5daaeba9f0303e173dd1aa);  need to set [termguicolors](https://neovim.io/doc/user/options.html#%27termguicolors) to enable true color.
* [vim](https://github.com/vim/vim) - (from 7.4.1770) since commit [8a633e3427b47286869aa4b96f2bfc1fe65b25cd](https://github.com/vim/vim/commit/8a633e3427b47286869aa4b96f2bfc1fe65b25cd); need to set [termguicolors](https://github.com/vim/vim/blob/master/runtime/doc/version8.txt#L202) to enable true color.
* [emacs](https://www.gnu.org/software/emacs/) - since commit [e463e57...](http://git.savannah.gnu.org/cgit/emacs.git/commit/?id=e463e57), require custom terminfo
* [elinks](http://repo.or.cz/w/elinks.git) - [configure.in:1410](http://repo.or.cz/w/elinks.git/blob/HEAD:/configure.in#l1410) (./configure --enable-true-color)
* [s-lang](http://lists.jedsoft.org/lists/slang-users/2015/0000020.html) library -  (since pre2.3.1-35, for 64bit systems)
* [timg](https://github.com/hzeller/timg) - Terminal Image Viewer
* [tv](https://github.com/daleroberts/tv) - tool to quickly view high-resolution multi-band imagery directly in terminal
* [mpv](https://github.com/mpv-player/mpv) - video player with support of console-only output (since https://github.com/mpv-player/mpv/commit/dd02369c3223fda5bcb2658b15404d43232bb38f)
* [radare2](https://github.com/radare/radare2) - reverse engineering franework; since [be46b9...](https://github.com/radare/radare2/commit/be46b9da31c7d2b6ac30ea3c62a32016f9482177) commit.

Not supporting True Colour:

* mutt - http://dev.mutt.org/trac/ticket/3674
* neomutt - https://github.com/neomutt/neomutt/issues/85
* ncurses library - https://lists.gnu.org/archive/html/bug-ncurses/2013-10/msg00007.html
* termbox library - https://github.com/nsf/termbox/issues/37
* mcabber - https://bitbucket.org/McKael/mcabber-crew/issue/126/support-for-true-color-16-millions-colors
* tig - https://github.com/jonas/tig/issues/227
* joe - https://sourceforge.net/p/joe-editor/patches/116/
