# What Are We Going To Build?

This question is of paramount importance. The answer will usually determine if you will likely to
succeed. If you want to overstep your boundaries, you will fail. It shouldn't be too easy either.
If you know something about programming already, I bet you can implement Tic Tac Toe, but will you
feel proud about it? Will you be able to say "I've built a world!". I wouldn't.

## Graphics

To begin with, we need to know what kind of graphics are we aiming for. We will instantly rule out
3D for several reasons:
- We don't want to increase the scope and complexity
- Ruby may not be fast enough for 3D games
- Learning proper 3D graphics programming requires reading a separate book that is several times
  thicker than this one.

Now, we have to swallow our pride and accept the fact that the game will have simple 2D graphics.
There are three choices to go for:
- Parallel Projection
- Top Down
- Side-Scrolling

*Parallel Projection* (think Fallout 1 & 2) is pretty close to 3D graphics, it requires detailed art
if you want it to look decent, so we would have a rough start if we went for it.

*Top Down* view (old titles of Legend of Zelda) offers plenty of freedom to explore the environment
in all directions and requires less graphical detail, since things look simpler from above.

*Side Scrolling* games (Super Mario Bros.) usually involve some physics related to jumping and
require more effort to look good. Feeling of exploration is limited, since you usually move from
left to right most of the time.

Going with Top Down view will give us a chance to create our game world as open for exploration as
possible, while having simple graphics and movement mechanics. Sounds like the best choice for us.

## Game Development Library

Implement it all yourself or harness the power of some game development library that offers you
boilerplates and convenient access to common functions? If you're like me, you would definitely
want to implement it all yourself, but that may be the reason why I failed to make a decent game so
many times. If you will try to implement it all yourself, you will most likely end up
reimplementing some existing game library, poorly. It won't take long while you reach a point where
you need to interface with underlying operating system libraries to get graphics. And guess if
those bindings will work in a different operating system?

So, swallow your pride again, because we are going to use an existing game development library.
Good news is that you will be able to actually finish the game, and it will be portable to Windows,
Mac and Linux. We will still have to build our own game engine for ourselves on top of it, so don't
think it won't be fun.

There are [several game libraries](https://www.ruby-toolbox.com/categories/game_libraries)
available for Ruby, but it's a simple choice, because [Gosu](http://www.libgosu.org/) is head and
shoulders above others. It's very mature, has a large and active community, and it is mainly
written in C++ but has first class Ruby support, so it will be both fast and convenient to use.

Many of other Ruby game libraries are built on top of Gosu, so it's a solid choice.

## Theme



