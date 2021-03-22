# Sunrise over island

This is a simple SVG animation of the sun rising over an island in the ocean. To view it, simply open the webpage and watch as, over the course of 3 minutes and 20 seconds, the sun comes up. Note that it won't work on any screen smaller than 800x600px. This was made using the awesome [Celestine library](https://github.com/celestinecr/celestine), so if you liked this, please check that out!

## How it works

It's pretty simple. The island and tree are actually one path: a Beziér curve for the shore, a small line for the trunk, and several triangles for the leaves. The sun is, obviously, just a red circle. There are two copies of each shape, however: one for the above-water "real" version, and another for the reflection in the water. The reflected shapes all have a filter applied, so they jitter a bit, like real water. The sunrise itself is just a simple animation of the circle moving up, with an animated mask to hide any parts below the horizon, synced with the color of the sky brightening from black to off-white.

## Hack it!

IF you want to modify the animation itself, the main code is in `src/main.cr`. However, if you just want to configure the existing animation, you can edit `src/config.cr` with whatever values you desire. Feel free to fork and PR this however you want, but please give credit if you share any creations.
