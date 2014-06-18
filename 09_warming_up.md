# Warming Up

Before we start building our game, we want to flex our skills little more, get to know Gosu better
and make sure our tools will be able to meet our expectations.

## Using Tilesets

After playing around with Gosu for a while, we should be comfortable enough to implement a
prototype of top-down view game map using the tileset of our choice. This [ground
tileset](http://opengameart.org/content/ground-tileset-grass-sand) looks like a good place to
start.

However, after downloading and extracting the tileset, it's obvious that
[`Gosu::Image#load_tiles`](http://www.libgosu.org/rdoc/Gosu/Image.html#load_tiles-class_method)
will not suffice, since it only supports tiles of same size, and there is a tileset in the package
that looks like this:

![Tileset with tiles of irregular size](images/07-irregular-tiles.png)

And there is also a JSON file that contains some metadata:

{lang="json",line-numbers="off"}
~~~~~~~~
{"frames": {
"aircraft_1d_destroyed.png":
{
  "frame": {"x":451,"y":102,"w":57,"h":42},
  "rotated": false,
  "trimmed": false,
  "spriteSourceSize": {"x":0,"y":0,"w":57,"h":42},
  "sourceSize": {"w":57,"h":42}
},
"aircraft_2d_destroyed.png":
{
  "frame": {"x":2,"y":680,"w":63,"h":47},
  "rotated": false,
  "trimmed": false,
  "spriteSourceSize": {"x":0,"y":0,"w":63,"h":47},
  "sourceSize": {"w":63,"h":47}
},
...
}},
"meta": {
	"app": "http://www.texturepacker.com",
	"version": "1.0",
	"image": "decor.png",
	"format": "RGBA8888",
	"size": {"w":512,"h":1024},
	"scale": "1",
	"smartupdate": "$TexturePacker:SmartUpdate:2e6b6964f24c7abfaa85a804e2dc1b05$"
}
~~~~~~~~

Looks like these tiles were packed with [Texture Packer](http://www.texturepacker.com). After some
digging I've discovered that Gosu doesn't have any integration with it, so I had these choices:

1. Cut the original tileset image into smaller images.
2. Parse JSON and harness the benefits of Texture Packer.

First option was too much work and would prove to be less efficient, because loading many small
files is always worse than loading one bigger file. Therefore, second option was the winner, and I
also thought "why not write a gem while I'm at it". And that's exactly what I did, and you should
do the same in such a situation. The gem is available on GitHub:

[https://github.com/spajus/gosu-texture-packer](https://github.com/spajus/gosu-texture-packer)

You can install this gem using `gem install gosu_texture_packer`. If you want to examine the code, easiest way is to clone it on your computer:

{lang="console",line-numbers="off"}
~~~~~~~~
$ git clone
~~~~~~~~

Let's examine the main idea behind this gem. it is included in code samples. It's core class is
less than 50 lines of code, Got to love Ruby for that.

<<[02-warmup/tileset.rb](code/02-warmup/tileset.rb)







