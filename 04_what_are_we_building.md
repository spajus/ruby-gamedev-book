# What Are We Going To Build?

This question is of paramount importance. The answer will usually determine if you will likely to
succeed. If you want to overstep your boundaries, you will fail. It shouldn't be too easy either.
If you know something about programming already, I bet you can implement Tic Tac Toe, but will you
feel proud about it? Will you be able to say "I've built a world!". I wouldn't.

## Graphics

To begin with, we need to know what kind of graphics we are aiming for. We will instantly rule out
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

Since you are probably as bad in drawing things as I am, you could still wonder how we are going
to get our graphics. Thankfully, there is this [opengameart.org](http://opengameart.org). It's like
GitHub of game media, we will surely find something there. It also contains audio samples and
tracks.

## Game Development Library

Implement it all yourself or harness the power of some game development library that offers you
boilerplates and convenient access to common functions? If you're like me, you would definitely
want to implement it all yourself, but that may be the reason why I failed to make a decent game so
many times.

If you will try to implement it all yourself, you will most likely end up reimplementing some
existing game library, poorly. It won't take long while you reach a point where you need to
interface with underlying operating system libraries to get graphics. And guess if those bindings
will work in a different operating system?

So, swallow your pride again, because we are going to use an existing game development library.
Good news is that you will be able to actually finish the game, and it will be portable to Windows,
Mac and Linux. We will still have to build our own game engine for ourselves on top of it, so don't
think it won't be fun.

There are [several game libraries](https://www.ruby-toolbox.com/categories/game_libraries)
available for Ruby, but it's a simple choice, because [Gosu](http://www.libgosu.org/) is head and
shoulders above others. It's very mature, has a large and active community, and it is mainly
written in C++ but has first class Ruby support, so it will be both fast and convenient to use.

Many of other Ruby game libraries are built on top of Gosu, so it's a solid choice.

## Theme And Mechanics

Choosing the right theme is undoubtedly important. It should be something that appeals to you,
something you will want to play, and it should not imply difficult game mechanics. I love MMORPGs,
and I always dreamed of making an open world game where you can roam around, meet other players,
fight monsters and level up. Guess how many times I started building such a game? Even if I
wouldn't have lost the count, I wouldn't be proud to say the number.

This time, equipped with logic and sanity, I've picked something challenging enough, yet still
pretty simple to build. Are you ready?

*Drumroll*...

We will be building a multi directional shooter arcade game where you control a tank, roam around
an island, shoot enemy tanks and try not to get destroyed by others.

If you have played [Battle City](http://en.wikipedia.org/wiki/Battle_City_(video_game)) or
[Tank Force](http://en.wikipedia.org/wiki/Tank_Force), you should easily get the idea. I believe
that implementing such a game (with several twists) would expose us to perfect level of difficulty
and provide substantial amount of experience.

We will use a subset of [these gorgeous graphics](http://www.praire-chicken.com/chabull/tr.html)
which are [available on opengameart.org](http://opengameart.org/users/chabull), generously provided
by [Csaba Felvegi](https://www.google.com/search?q=Csaba+Felvegi).

## Publishing The Game On Steam

When we will have a working, polished game ready, we will submit it to
[Steam Greenlight](http://steamcommunity.com/greenlight) and hope it gets accepted. If it does,
the book will also cover basics of publishing games on Steam platform. It's fun to build things you
love, but it's even more enjoyable when you get paid for it!
