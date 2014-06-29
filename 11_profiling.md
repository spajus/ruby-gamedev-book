# Optimizing Game Performance

To make games that are fast and don't require a powerhouse to run, we must learn how to find and
fix bottlenecks. Good news is that if you wasn't thinking about performance to begin with, your
program can usually be optimized to run twice as fast just by eliminating one or two biggest
bottlenecks.

## Profiling Ruby Code To Find Bottlenecks

We will try to find bottlenecks in our Tanks prototype game by profiling it with
[`ruby-prof`](https://github.com/ruby-prof/ruby-prof).

It's a ruby gem, just install it like this:

{lang="console",line-numbers="off"}
~~~~~~~~
$ gem install ruby-prof
~~~~~~~~

There are several ways you can use `ruby-prof`, so we will begin with the easiest one. Instead of
running the game with `ruby`, we will run it with `ruby-prof`:

{lang="console",line-numbers="off"}
~~~~~~~~
$ ruby-prof 03-prototype/main.rb
~~~~~~~~

The game will run, but everything will be ten times slower as usual, because every call to every
function is being recorded, and after you exit the program, profiling output will be dumped
directly to your console.

Downside of this approach is that we are going to profile everything there is, including the
super-slow map generation that uses Perlin Noise. We don't want to optimize that, so in order to
find bottlenecks in our play state rather than map generation, we have to keep playing at dreadful
2 FPS for at least 30 seconds.

This was the output of first "naive" profiling session:

![Initial profiling results](images/19-naive-profile.png)

It's obvious, that `Camera#viewport` and `Camera#can_view?` are top CPU burners. This means either
that our implementation is either very bad, or the assumption that checking if camera can view
object is slower than drawing the object off screen.

Here are those slow methods:

{line-numbers="off"}
~~~~~~~~
class Camera
  # ...
  def can_view?(x, y, obj)
    x0, x1, y0, y1 = viewport
    (x0 - obj.width..x1).include?(x) &&
      (y0 - obj.height..y1).include?(y)
  end
  # ...
  def viewport
    x0 = @x - ($window.width / 2)  / @zoom
    x1 = @x + ($window.width / 2)  / @zoom
    y0 = @y - ($window.height / 2) / @zoom
    y1 = @y + ($window.height / 2) / @zoom
    [x0, x1, y0, y1]
  end
  # ...
end
~~~~~~~~

It doesn't look fundamentally broken, so we will try our "checking is slower than rendering"
hypothesis by short-circuiting `can_view?` to return `true` every time:

{line-numbers="off"}
~~~~~~~~
class Camera
  # ...
  def can_view?(x, y, obj)
    return true # short circuiting
    x0, x1, y0, y1 = viewport
    (x0 - obj.width..x1).include?(x) &&
      (y0 - obj.height..y1).include?(y)
  end
  # ...
end
~~~~~~~~

After saving `camera.rb` and running the game without profiling, you will notice a significant
speedup. Hypothesis was correct, checking visibility is more expensive than simply rendering it.
That means we can throw away `Camera#can_view?` and calls to it.

But before doing that, let's profile once again:

![Profiling results after short-circuiting `Camera#can_view?`](images/20-naive-profile-2.png)

We can see `Camera#can_view?` is still in top 3, so we will remove
`if camera.can_view?(map_x, map_y, tile)` from `Map#draw` and for now keep it like this:

{line-numbers="off"}
~~~~~~~~
class Map
  # ...
  def draw(camera)
    @map.each do |x, row|
      row.each do |y, val|
        tile = @map[x][y]
        map_x = x * TILE_SIZE
        map_y = y * TILE_SIZE
        tile.draw(map_x, map_y, 0)
      end
    end
  end
  # ...
end
~~~~~~~~

After completely removing `Camera#can_view?`, profiling session looks like dead-end - no more low
hanging fruits on top:

![Profiling results after removing `Camera#can_view?`](images/21-naive-profile-3.png)

The game still doesn't feel fast enough, FPS occasionally keeps dropping down to ~45, so we will
have to do profile our code in smarter way.

## Advanced Profiling Techniques

We would get more accuracy when profiling only what we want to optimize. In our case it is
everything that happens in `PlayState`, except for `Map` generation. This time we will have to use
[`ruby-prof`](https://github.com/ruby-prof/ruby-prof) API to hook into places we need.

`Map` generation happens in `PlayState` initializer, so we will leverage `GameState#enter` and
`GameState#leave` to start and stop profiling, since it happens after state is initialized. Here is
how we hook in:

{line-numbers="off"}
~~~~~~~~
require 'ruby-prof'
class PlayState < GameState
  # ...
  def enter
    RubyProf.start
  end

  def leave
    result = RubyProf.stop
    printer = RubyProf::FlatPrinter.new(result)
    printer.print(STDOUT)
  end
  # ...
end
~~~~~~~~

Then we run the game as usual:

{lang="console",line-numbers="off"}
~~~~~~~~
$ ruby 03-prototype/main.rb
~~~~~~~~

Now, after we press `N` to start new game, `Map` generation happens relatively fast, and then
profiling kicks in, FPS drops to 15. After moving around and shooting for a while we hit `Esc` to
return to the menu, and at that point `PlayState#leave` spits profiling results out to the console:

![Profiling results for `PlayState`](images/22-play-profile.png)

We can see that
[`Gosu::Image#draw`](http://www.libgosu.org/rdoc/Gosu/Image.html#draw-instance_method) takes up to
20% of all execution time. Then goes
[`Gosu::Window#caption`](http://www.libgosu.org/rdoc/Gosu/Window.html#caption-instance_method), but
we need it to measure FPS, so we will leave it alone, and finally we can see
[`Hash#each`](http://www.ruby-doc.org/core-2.1.2/Hash.html#method-i-each), which is guaranteed to
be the one from `Map#draw`, and it triggers all those Gosu::Image#draw calls.

## Optimizing Inefficient Loops

According to profiling results, we need to optimize this method:

{line-numbers="off"}
~~~~~~~~
class Map
  # ...
  def draw(camera)
    @map.each do |x, row|
      row.each do |y, val|
        tile = @map[x][y]
        map_x = x * TILE_SIZE
        map_y = y * TILE_SIZE
        tile.draw(map_x, map_y, 0)
      end
    end
  end
  # ...
end
~~~~~~~~

But we have to optimize it in more clever way than we did before. If instead of looping through all
map rows and columns and blindly rendering every tile or checking if tile is visible we could
calculate the exact map cells that need to be displayed, we would reduce method complexity
and get major performance boost. Let's do that.






