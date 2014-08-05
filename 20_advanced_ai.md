# Building Advanced AI

The AI we have right now can kick some ass, but it is too dumb for any seasoned gamer to compete
with. This is the list of current flaws:

1. It does not navigate well, gets stuck among trees or somewhere near water.
2. It is not aware of powerups.
3. It could do better job at shooting.
4. It's field of vision is too small, compared to player's, who is equipped with radar.

We will tackle these issues in current chapter.

## Improving Tank Navigation

Tanks shouldn't behave like Roombas, randomly driving around and bumping into things. They could be
navigating like this:

1. Consult with current AI state and find or update destination point.
2. If destination has changed, calculate shortest path to destination.
3. Move along the calculated path.
4. Repeat.

If this looks easy, let me assure you, it would probably require rewriting the majority of `AI` and
`Map` code we have at this point, and it is pretty tricky to implement with procedurally generated
maps, because normally you would use a map editor to set up waypoints, navigation mesh or other
hints for AI so it doesn't get stuck. Sometimes it is better to have something working imperfectly
over a perfect solution that never happens, thus we will use simple things that will make as much
impact as possible without rewriting half of the code.

### Generating Friendlier Maps

One of main reasons why tanks get stuck is bad placement of spawn points. They don't take trees and
boxes into account, so enemy tank can spawn in the middle of a forest, with no chance of getting
out without blowing things up. A simple fix would be to consult with `ObjectPool` before placing a
spawn point only where there are no other game objects around in, say, 150 pixel radius:

{line-numbers="off"}
~~~~~~~~
class Map
  # ...
  def find_spawn_point
    while true
      x = rand(0..MAP_WIDTH * TILE_SIZE)
      y = rand(0..MAP_HEIGHT * TILE_SIZE)
      if can_move_to?(x, y) &&
          @object_pool.nearby_point(x, y, 150).empty?
        return [x, y]
      end
    end
  end
  # ...
end
~~~~~~~~

How about powerups? They can also spawn in the middle of a forest, and while tanks are not seeking
them yet, we will be implementing this behavior, and leading tanks into wilderness of trees is not
the best idea ever. Let's fix it too:

{line-numbers="off"}
~~~~~~~~
class Map
  # ...
  def generate_powerups
    pups = 0
    target_pups = rand(20..30)
    while pups < target_pups do
      x = rand(0..MAP_WIDTH * TILE_SIZE)
      y = rand(0..MAP_HEIGHT * TILE_SIZE)
      if tile_at(x, y) != @water &&
          @object_pool.nearby_point(x, y, 150).empty?
        random_powerup.new(@object_pool, x, y)
        pups += 1
      end
    end
  end
  # ...
end
~~~~~~~~

We could also reduce tree count, but that would make the map look worse, so we are going to keep
this in our pocket as a mean of last resort.

## Implementing Demo State To Observe AI

Probably the best way to figure out if our AI is any good is to target one of AI tanks with our
game camera and see how it plays. It will give us a great visual testing tool that will allow
tweaking AI settings and seeing if they perform better or worse. For that we will introduce
`DemoState` where only AI tanks will be present in the map, and we will be able to switch camera
from one tank to another.

`DemoState` is very similar to `PlayState`, the main difference is that there is no player. We will
extract `create_tanks` method that will be overridden in `DemoState`.

{line-numbers="off"}
~~~~~~~~
class PlayState < GameState
  attr_accessor :update_interval, :object_pool, :tank

  def initialize
    # ...
    @camera = Camera.new
    @object_pool.camera = @camera
    create_tanks(4)
  end
  # ...
  private

  def create_tanks(amount)
    @map.spawn_points(amount * 3)
    @tank = Tank.new(@object_pool,
      PlayerInput.new('Player', @camera, @object_pool))
    amount.times do |i|
      Tank.new(@object_pool, AiInput.new(
        @names.random, @object_pool))
    end
    @camera.target = @tank
    @hud = HUD.new(@object_pool, @tank)
  end
  # ...
end
~~~~~~~~



