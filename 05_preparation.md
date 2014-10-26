# Preparing The Tools

While writing this book, I will be using Mac OS X (10.9), but it should be possible to run all the
exampls on other operating systems too.

Gosu Wiki has "Getting Started" pages for
[Mac](https://github.com/jlnr/gosu/wiki/Getting-Started-on-OS-X),
[Linux](https://github.com/jlnr/gosu/wiki/Getting-Started-on-Linux) and
[Windows](https://github.com/jlnr/gosu/wiki/Getting-Started-on-Windows), so I will not be going
into much detail here.

## Getting Gosu to run on Mac Os X

If you haven't set up your Mac for development, first install Xcode using App Store.
System Ruby should work just fine, but you may want to use
[Rbenv](https://github.com/sstephenson/rbenv) or [RVM](http://rvm.io/) to avoid polluting system
Ruby. I've had trouble installing Gosu with RVM, but your experience may vary.

To install the gem, simply run:

{lang="console",line-numbers="off"}
~~~~~~~~
$ gem install gosu
~~~~~~~~

You may need to prefix it with `sudo` if you are using system Ruby.

To test if gem was installed correctly, you should be able to run this to produce an empty black
window:

{lang="console",line-numbers="off"}
~~~~~~~~
$ irb
irb(main):001:0> require 'gosu'
=> true
irb(main):002:0> Gosu::Window.new(320, 240, false).show
=> nil
~~~~~~~~

Most developers who use Mac every day will also recommend installing [Homebrew](http://brew.sh/)
package manager, replace Terminal app with [iTerm2](http://www.iterm2.com/) and use
[Oh-My-Zsh](http://ohmyz.sh/) to manage ZSH configuration.

# Getting The Sample Code

You can find sample code at GitHub:
[https://github.com/spajus/ruby-gamedev-book-examples](https://github.com/spajus/ruby-gamedev-book-examples).

Clone it to a convenient location:

{lang="console",line-numbers="off"}
~~~~~~~~
$ cd ~/gamedev
$ git clone git@github.com:spajus/ruby-gamedev-book-examples.git
~~~~~~~~

Complete source code for the game that was built while writing this book is also available on
GitHub. TODO add links when they are available.

# Other Tools

All you need for this adventure is a good text editor, terminal and probably some graphics editor.
Try [GIMP](http://www.gimp.org/) if you want a free one. I'm using
[Pixelmator](http://www.pixelmator.com/), it's wonderful, but for Mac only. A noteworthy fact is
that Pixelmator was built by fellow Lithuanians.

When it comes to editors, I don't leave home without Vim, but as long as what you use makes you
productive, it doesn't make any difference. Vim, Emacs or Sublime are all good enough to write
code, just have some good plugins that support Ruby, and you're set. If you really feel you need
an IDE, which may be the case if you are coming from a static language, you can't go wrong with
[RubyMine](http://www.jetbrains.com/ruby/).

