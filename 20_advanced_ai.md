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

We will also want to display a smaller version of score in top-right corner of the screen, so let's
add some adjustments to `ScoreDisplay`:

{line-numbers="off"}
~~~~~~~~
class ScoreDisplay
  def initialize(object_pool, font_size=30)
    @font_size = font_size
    # ...
  end

  def create_stats_image(stats)
    # ...
    @stats_image = Gosu::Image.from_text(
      $window, text, Utils.main_font, @font_size)
  end
  # ...
  def draw_top_right
    @stats_image.draw(
      $window.width - @stats_image.width - 20,
      20,
      1000)
  end
end
~~~~~~~~

And here is the extended `DemoState`:

<<[13-advanced-ai/game_states/demo_state.rb](code/13-advanced-ai/game_states/demo_state.rb)

To have a possibility to enter `DemoState`, we need to change `MenuState` a little:

{line-numbers="off"}
~~~~~~~~
class MenuState < GameState
  # ...
  def update
    text = "Q: Quit\nN: New Game\nD: Demo"
    # ...
  end
  # ...
  def button_down(id)
    # ...
    if id == Gosu::KbD
      @play_state = DemoState.new
      GameState.switch(@play_state)
    end
  end
end
~~~~~~~~

Now, main menu has the option to enter demo state:

![Overhauled main menu](images/52-main-menu.png)

![Observing AI in demo state](images/53-demo-state.png)

## Visual AI Debugging

After watching AI behavior in demo mode for a while, I was terrified. When playing game normally,
you usually see tanks in "fighting" state, which works pretty well, but when tanks go roaming, it's
a complete disaster. They get stuck easily, they don't go too far from the original location, they
wait too much.

Some things could be improved just by changing `wait_time`, `turn_time` and `drive_time` to
different values, but we certainly have to do bigger changes than that.

On the other hand, "observe AI in action, tweak, repeat" cycle proved to be very effective, I will
definitely use this technique in all my future games.

To make visual debugging easier, build yourself some tooling. One way to do it is to have global
`$debug` variable which you can toggle by pressing some button:

{line-numbers="off"}
~~~~~~~~
class PlayState < GameState
  # ...
  def button_down(id)
    # ...
    if id == Gosu::KbF1
      $debug = !$debug
    end
    # ...
  end
  # ...
end
~~~~~~~~

Then add extra drawing instructions to your objects and their components. For example, this will
make `Tank` display it's current `TankMotionState` implementation class beneath it:

{line-numbers="off"}
~~~~~~~~
class TankMotionFSM
  # ...
  def set_state(state)
    # ...
    if $debug
      @image = Gosu::Image.from_text(
          $window, state.class.to_s,
          Gosu.default_font_name, 18)
    end
  end
  # ...
  def draw(viewport)
    if $debug
      @image && @image.draw(
        @object.x - @image.width / 2,
        @object.y + @object.graphics.height / 2 -
        @image.height, 100)
    end
  end
  # ...
end
~~~~~~~~

To mark tank's desired gun angle as blue line and actual gun angle as red line, you can do this:

{line-numbers="off"}
~~~~~~~~
class AiGun
  # ...
  def draw(viewport)
    if $debug
      color = Gosu::Color::BLUE
      x, y = @object.x, @object.y
      t_x, t_y = Utils.point_at_distance(x, y, @desired_gun_angle,
                                         BulletPhysics::MAX_DIST)
      $window.draw_line(x, y, color, t_x, t_y, color, 1001)
      color = Gosu::Color::RED
      t_x, t_y = Utils.point_at_distance(x, y, @object.gun_angle,
                                         BulletPhysics::MAX_DIST)
      $window.draw_line(x, y, color, t_x, t_y, color, 1000)
    end
  end
  # ...
end
~~~~~~~~

Finally, you can automatically mark collision box corners on your graphics components. Let's take
`BoxGraphics` for example:

{line-numbers="off"}
~~~~~~~~
# 13-advanced-ai/misc/utils.rb
module Utils
  # ...
  def self.mark_corners(box)
    i = 0
    box.each_slice(2) do |x, y|
      color = DEBUG_COLORS[i]
      $window.draw_triangle(
        x - 3, y - 3, color,
        x,     y,     color,
        x + 3, y - 3, color,
        100)
      i = (i + 1) % 4
    end
  end
  # ...
end

# 13-advanced-ai/entities/components/box_graphics.rb
class BoxGraphics < Component
  # ..
  def draw(viewport)
    @box.draw_rot(x, y, 0, object.angle)
    Utils.mark_corners(object.box) if $debug
  end
  # ...
end
~~~~~~~~

As a developer, you can make yourself see nearly everything you want, make use of it.

![Visual debugging of AI behavior](images/54-visual-debugging.png)

Although it hurts the framerate a little, it is very useful when building not only AI, but the
rest of the game too. Using this visual debugging together with Demo mode, you can tweak all the AI
values to make it shoot more often, fight better, and be more agile. We won't go through this
minor tuning, but you can find the changes by [viewing changes introduced in `13-advanced-ai`](https://github.com/spajus/ruby-gamedev-book-examples/compare/8727db2...0e3c926).

## Making AI Collect Powerups

To even out the odds, we have to make AI seek powerups when they are required. The logic behind it
can be implemented using a couple of simple steps:

1. AI would know what powerups are currently needed. This may vary from state to state, i.e. speed
and fire rate powerups are nice to have when roaming, but not that important when fleeing after
taking heavy damage. And we don't want AI to waste time and collect speed powerups when speed
modifier is already maxed out.
2. `AiVision` would return closest visible powerup, filtered by acceptable powerup types.
3. Some `TankMotionState` implementation would adjust tank direction towards closest visible
powerup in `change_direction` method.

### Finding Powerups In Sight

To implement changes in `AiVision`, we will introduce `closest_powerup` method. It will query
objects in sight and filter them out by their class and distance.

{line-numbers="off"}
~~~~~~~~
class AiVision
  # ...
  POWERUP_CACHE_TIMEOUT = 50
  # ...
  def closest_powerup(*suitable)
    now = Gosu.milliseconds
    @closest_powerup = nil
    if now - (@powerup_cache_updated_at ||= 0) > POWERUP_CACHE_TIMEOUT
      @closest_powerup = nil
      @powerup_cache_updated_at = now
    end
    @closest_powerup ||= find_closest_powerup(*suitable)
  end

  private

  def find_closest_powerup(*suitable)
    if suitable.empty?
      suitable = [FireRatePowerup,
                  HealthPowerup,
                  RepairPowerup,
                  TankSpeedPowerup]
    end
    @in_sight.select do |o|
      suitable.include?(o.class)
    end.sort do |a, b|
      x, y = @viewer.x, @viewer.y
      d1 = Utils.distance_between(x, y, a.x, a.y)
      d2 = Utils.distance_between(x, y, b.x, b.y)
      d1 <=> d2
    end.first
  end
  # ...
end
~~~~~~~~

It is very similar to `AiVision#closest_tank`, and parts should probably be extracted to keep the
code dry, but we will not bother.

### Seeking Powerups While Roaming

Roaming is when most picking should happen, because Tank sees no enemies in sight and needs to
prepare for upcoming battles. Let's see how can we implement this behavior while leveraging the
newly made `AiVision#closest_powerup`:

{line-numbers="off"}
~~~~~~~~
class TankRoamingState < TankMotionState
  # ...
  def required_powerups
    required = []
    health = @object.health.health
    if @object.fire_rate_modifier < 2 && health > 50
      required << FireRatePowerup
    end
    if @object.speed_modifier < 1.5 && health > 50
      required << TankSpeedPowerup
    end
    if health < 100
      required << RepairPowerup
    end
    if health < 190
      required << HealthPowerup
    end
    required
  end

  def change_direction
    closest_powerup = @vision.closest_powerup(
      *required_powerups)
    if closest_powerup
      @seeking_powerup = true
      angle = Utils.angle_between(
        @object.x, @object.y,
        closest_powerup.x, closest_powerup.y)
      @object.physics.change_direction(
        angle - angle % 45)
    else
      @seeking_powerup = false
      # ... choose random direction
    end
    @changed_direction_at = Gosu.milliseconds
    @will_keep_direction_for = turn_time
  end
  # ...
  def turn_time
    if @seeking_powerup
      rand(100..300)
    else
      rand(1000..3000)
    end
  end
end
~~~~~~~~

It is simple as that, and our AI tanks are now getting buffed on their spare time.

## Seeking Health Powerups After Heavy Damage

To seek health when damaged, we need to change `TankFleeingState#change_direction`:

{line-numbers="off"}
~~~~~~~~
class TankFleeingState < TankMotionState
  # ...
  def change_direction
    closest_powerup = @vision.closest_powerup(
      RepairPowerup, HealthPowerup)
    if closest_powerup
      angle = Utils.angle_between(
        @object.x, @object.y,
        closest_powerup.x, closest_powerup.y)
      @object.physics.change_direction(
        angle - angle % 45)
    else
      # ... reverse from enemy
    end
    @changed_direction_at = Gosu.milliseconds
    @will_keep_direction_for = turn_time
  end
  # ...
end
~~~~~~~~

This small change tells AI to pick up health while fleeing. The interesting part is that when tank
picks up `RepairPowerup`, it's health gets fully restored and AI should switch back to
`TankFightingState`. This simple thing is a major improvement in AI behavior.

## Evading Collisions And Getting Unstuck

While observing AI navigation, it was noticeable that tanks often got stuck, even in simple
situations, like driving into a tree and hitting it repeatedly for a dozen of seconds. To reduce
the number of such occasions, we will introduce `TankNavigatingState`, which would help avoid
collisions, and `TankStuckState`, which would be responsible for driving out of dead ends as
quickly as possible.

To implement these states, we need to have a way to tell if tank can go forward and a way of
getting a direction which is not blocked by other objects. Let's add a couple of methods to
`AiVision`:

{line-numbers="off"}
~~~~~~~~
class AiVision
  # ...
  def can_go_forward?
    in_front = Utils.point_at_distance(
      *@viewer.location, @viewer.direction, 40)
    @object_pool.map.can_move_to?(*in_front) &&
      @object_pool.nearby_point(*in_front, 40, @viewer)
        .reject { |o| o.is_a? Powerup }.empty?
  end

  def closest_free_path(away_from = nil)
    paths = []
    5.times do |i|
      if paths.any?
        return farthest_from(paths, away_from)
      end
      radius = 55 - i * 5
      range_x = range_y = [-radius, 0, radius]
      range_x.shuffle.each do |x|
        range_y.shuffle.each do |y|
          x = @viewer.x + x
          y = @viewer.y + y
          if @object_pool.map.can_move_to?(x, y) &&
              @object_pool.nearby_point(x, y, radius, @viewer)
                .reject { |o| o.is_a? Powerup }.empty?
            if away_from
              paths << [x, y]
            else
              return [x, y]
            end
          end
        end
      end
    end
    false
  end

  alias :closest_free_path_away_from :closest_free_path
  # ...
  private

  def farthest_from(paths, away_from)
    paths.sort do |p1, p2|
      Utils.distance_between(*p1, *away_from) <=>
        Utils.distance_between(*p2, *away_from)
    end.first
  end
  # ...
end
~~~~~~~~

`AiVision#can_go_forward?` tells if tank can move ahead, and `AiVision#closest_free_path` finds a
point where tank can move without obstacles. You can also call
`AiVision#closest_free_path_away_from` and provide coordinates you are trying to get away from.

We will use `closest_free_path` methods in newly implemented tank motion states, and
`can_go_forward?` in `TankMotionFSM`, to make a decision when to jump into navigating or stuck
state.

Those new states are nothing fancy:

<<[13-advanced-ai/entities/components/ai/tank_navigating_state.rb](code/13-advanced-ai/entities/components/ai/tank_navigating_state.rb)

`TankNavigatingState` simply chooses a random free path, changes direction to it and keeps driving.

<<[13-advanced-ai/entities/components/ai/tank_stuck_state.rb](code/13-advanced-ai/entities/components/ai/tank_navigating_state.rb)

`TankStuckState` is nearly the same, but it keeps driving away from `@stuck_at` point, which is set
by `TankMotionFSM` upon transition to this state.

{line-numbers="off"}
~~~~~~~~
class TankMotionFSM
  STATE_CHANGE_DELAY = 500
  LOCATION_CHECK_DELAY = 5000

  def initialize(object, vision, gun)
    # ...
    @stuck_state = TankStuckState.new(object, vision, gun)
    @navigating_state = TankNavigatingState.new(object, vision)
    set_state(@roaming_state)
  end
  # ...
  def choose_state
    unless @vision.can_go_forward?
      unless @current_state == @stuck_state
        set_state(@navigating_state)
      end
    end
    # Keep unstucking itself for a while
    change_delay = STATE_CHANGE_DELAY
    if @current_state == @stuck_state
      change_delay *= 5
    end
    now = Gosu.milliseconds
    return unless now - @last_state_change > change_delay
    if @last_location_update.nil?
      @last_location_update = now
      @last_location = @object.location
    end
    if now - @last_location_update > LOCATION_CHECK_DELAY
      puts "checkin location"
      unless @last_location.nil? || @current_state.waiting?
        if Utils.distance_between(*@last_location, *@object.location) < 20
          set_state(@stuck_state)
          @stuck_state.stuck_at = @object.location
          return
        end
      end
      @last_location_update = now
      @last_location = @object.location
    end
    # ...
  end
  # ...
end
~~~~~~~~

What this does is automatically change state to navigating when tank is about to hit an obstacle.
It also tracks tank location, and if tank hasn't moved 20 pixels away from it's original direction
for 5 seconds, it enters `TankStuckState`, which deliberately tries to navigate away from the
`stock_at` spot.

AI navigation has just got significantly better, and it didn't take that many changes.

