# Wrapping It Up

Our journey into the world of game development has come to an end. We have learned enough to
produced a playable game, yet only scratched the surface. Writing this book was a very
enlightening experience, and hopefully reading it inspired or helped someone to get a start.

## Lessons Learned

Building this small tanks game and learning about game development with Ruby certainly had some
nasty bumps along the way, some of them made my head hit the ceiling.

### Ruby Is Slow

This shouldn't be a shocker, because Ruby is a dynamic, interpreted language, but how exactly slow
it is at some points was a staggering discovery. Probably the best evidence is that drawing map
tiles off screen using native extensions was actually faster than doing `Camera#can_view?` checks
that involve simple integer arithmetic and range checks.

If your game is going to deal with large number of entities, Ruby will start letting you down.
Dreaming about going pro? Go for C++, you won't make a mistake here.

Knowing this, keep in mind that Ruby is a wonderful language, that has it's own strengths. It's
great for prototyping and dynamic things. Some 5-10 lines of Ruby could translate into 50-100 lines
of C++. Also, knowing multiple languages makes you a better developer.

### Packaging Ruby Games Sucks

Unless you are releasing your game for tech savvy guys who can `gem install` it, get ready to go
through hell. There is no nice and easy way to create a
standalone executable application from Ruby code that involves native extensions. And you will go
through hell once for every operating system you want to publish your game for.

That's not everything. Want to use the
latest Ruby version? Check if you can make a package for it in your target OS before you start
coding. Thinking of using something that relies on ImageMagick? Too bad, you probably won't be able
to package the game into a native standalone app, at least on OSX. If you are planning on releasing
the game, package early and package often, for every OS, and check if there will be no problems
with native extensions.

### Plan Networked Multiplayer Early

If you are going to build a game, don't make a mistake of thinking "I'll just make it multiplayer
later", start at the very beginning. This was a lesson I learned the hard way. There had to be a
chapter in this book about turning Tanks into multiplayer, but it didn't happen, because it would
require a major rewrite of the code.

### Creating A Well Polished Game Requires Extraordinary Effort

Hacking up a rough prototype is extremely fun. You get to build an engine, wire everything
together. It definitely gives a sense of achievement. Turning it into a great game, however, is a
different story. You can spend hours or even days tweaking how game controls work and still remain
unsatisfied. Every tiny detail can be pushed further. Prefer quality over quantity, and remember
that you probably cannot afford both and actually finish it within next couple of years.

### Start Small, Take Baby Steps

Your first few games should be small experiments, prototypes or demos. Don't attempt to build a
game you wanted to build forever with your first shot. Try reimplementing Tetris, Pacman or
Bejeweled instead. You will find it to be challenging enough, and when you will feel you have the
skills to do something bigger, practice just a little more.

### Don't Reinvent The Wheel

Before doing anything, research. You will probably not get point in poly collision detection better
than W. Randolph Franklin did it [in his research](http://www.ecse.rpi.edu/~wrf/Research/Short_Notes/pnpoly.html).
Even if you think you can do it on your own, learn what others discovered before you. Learn from
other's mistakes, not your own.

# Special Thanks

I would like to thank [Julian Raschke](https://github.com/jlnr) for creating and maintaining
[Gosu](https://github.com/jlnr/gosu) and for all the help on IRC, Gosu forums and GitHub. This book
would not exist without your enormous contribution to Ruby game development scene.

Shout out goes to [Shawn Anderson](https://github.com/shawn42), creator of
[Gamebox](http://gamebox.io). Thank you for moral support and encouragement.
Studying Gamebox source code taught me many things about Gosu and game development.

You can find Julian, Shawn and more game development enthusiasts in #gosu on
[FreeNode](https://freenode.net/).

And most importantly, thank you for reading this book!
