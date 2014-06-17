## Music And Sound

Our previous program was clearly missing a soundtrack, so we will add one. A background music will
be looping, and each explosion will become audible.

<<[01-hello/hello_sound.rb](code/01-hello/hello_sound.rb)

Run it and enjoy the cinematic experience. Adding sound really makes a difference.

{lang="console",line-numbers="off"}
~~~~~~~~
$ ruby 01-hello/hello_sound.rb
~~~~~~~~

We only added couple of things over previous example.

~~~~~~~~
@music = Gosu::Song.new(
  self, media_path('menu_music.mp3'))
@music.volume = 0.5
@music.play(true)
~~~~~~~~

`GameWindow` creates
[`Gosu::Song`](http://www.libgosu.org/rdoc/Gosu/Song.html) with `menu_music.mp3`, adjusts the
volume so it's a little more quiet and starts playing in a loop.

~~~~~~~~
def self.load_sound(window)
  Gosu::Sample.new(
    window, media_path('explosion.mp3'))
end
~~~~~~~~

`Explosion` has now got `load_sound` method that loads `explosion.mp3` sound effect
[`Gosu::Sample`](http://www.libgosu.org/rdoc/Gosu/Sample.html). This sound effect is loaded once in
`GameWindow` constructor, and passed into every new `Explosion`, where it simply starts playing.

Handling audio with Gosu is very straightforward. Use
[`Gosu::Song`](http://www.libgosu.org/rdoc/Gosu/Song.html) to play background music, and
[`Gosu::Sample`](http://www.libgosu.org/rdoc/Gosu/Sample.html) to play effects and sounds that can
overlap.
