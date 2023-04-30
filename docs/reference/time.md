---
title: Time
id: time
---

This page will present you all the functions that can be used to play with time: slowing it down, speeding it up, reversing time, offsetting in time, etc... Each function will be presented following the same model:

* **Type signature**: how the function is declared on the **Haskell** side.
* **Description**: verbal description of the function.
* **Examples**: a small list of examples that you can copy/paste in your editor.

## Speeding up, slowing down

### fast

```haskell
Type: fast :: Pattern Time -> Pattern a -> Pattern a
```

`fast` speeds up a pattern. For example, the following will play the sound pattern `"bd sn kurt"` twice as fast (i.e. so it repeats twice per cycle), and the vowel pattern three times as fast:
```haskell
d1 $ sound (fast 2 "bd sn kurt")
   # fast 3 (vowel "a e o")
```

The first parameter can be patterned, for example to play the pattern at twice the speed for the first half of each cycle and then four times the speed for the second half:

```haskell
d1 $ fast "2 4" $ sound "bd sn kurt cp"
```

You can also use this function by its older alias, `density`.

### fastGap

```haskell
Type: fastGap :: Pattern Time -> Pattern a -> Pattern a
```
`fastGap` (alias `densityGap`) speeds up a pattern like `fast`, but rather than it playing multiple times as fast would it instead leaves a gap in the remaining space of the cycle. For example, the following will play the sound pattern `"bd sn"` only once but compressed into the first half of the cycle, i.e. twice as fast.

```haskell
d1 $ sound (fastGap 2 "bd sn")
```

### slow

```haskell
Type: slow :: Pattern Time -> Pattern a -> Pattern a
```
`slow` slows down a pattern. For example, the following will play the sound pattern `"bd sn kurt"` twice as slow (i.e. so it repeats once every two cycles), and the vowel pattern three times as slow:
```haskell
d1 $ sound (slow 2 "bd sn kurt")
   # slow 3 (vowel "a e o")
```

### sparsity

`sparsity` is a synonym of `slow`.

### hurry
```haskell
Type: hurry :: Pattern Time -> Pattern a -> Pattern a
```

`hurry` is similiar to fast, in that it speeds up a pattern, but it also increases the speed control by the same factor, so if you're triggering samples, the sound gets higher in pitch. For example:

```haskell
d1 $ every 2 (hurry 2) $ sound "bd sn:2 ~ cp"
```

### slowSqueeze

```haskell
Type: slowSqueeze :: Pattern Time -> Pattern a -> Pattern a
```

`slowSqueeze` slows down a pattern according to the given time pattern. It is the slow analogue to `fastSqueeze`. If the time pattern only has a single value in a cycle, `slowSqueeze` becomes equivalent to slow:
```haskell
d1 $ slow "<2 4>" $ s "bd*8"
```

is the same as:
```haskell
d1 $ slowSqueeze "<2 4>" $ s "bd*8"
```

but when the time pattern has multiple values in it the behavior is a little different! Instead, a slowed version of the pattern will be made for each value in the time pattern and then they're all combined together in a cycle, according to the structure of the time pattern. For example:
```haskell
d1 $ slowSqueeze "2 4 8 16" $ s "bd*8"
```
is equivalent to:
```haskell
d1 $ s "bd*4 bd*2 bd bd/2"
```
and:
```haskell
d1 $ slowSqueeze "2 4 [8 16]" $ s "bd*8"
```
is equivalent to:
```haskell
d1 $ s "bd*4 bd*2 [bd bd/2]"
```
### fastSqueeze

```haskell
Type: fastSqueeze :: Pattern Time -> Pattern a -> Pattern a
```
`fastSqueeze` speeds up a pattern by a time pattern given as input, squeezing the resulting pattern inside one cycle and playing the original pattern at every repetition.

To better understand how it works let's compare it with `fast`:
```haskell
d1 $ fast "1 2" $ s "bd sn"
```

```haskell
-- output
(0>½)|s: "bd"
(½>¾)|s: "bd"
(¾>1)|s: "sn"
```

This will give bd played in the first half cycle and bd sn in the second half. On the other hand, using `fastSqueeze`;

```haskell
fastSqueeze "1 2" $ s "bd sn"
```
```haskell
--output
(0>¼)|s: "bd"
(¼>½)|s: "sn"
(½>⅝)|s: "bd"
(⅝>¾)|s: "sn"
(¾>⅞)|s: "bd"
(⅞>1)|s: "sn"
```
The original pattern will play in the first half and two repetitions of the original pattern will play in the second half. That is, every repetition contains the whole pattern.

If the time pattern has a single value, it becomes equivalent to fast:
```haskell
d1 $ fastSqueeze 2 $ s "bd sn"
-- is equal to
d1 $ fast 2 $ s "bd sn"
-- and equivalent to
d1 $ s "[bd sn]*2"
```

## Zooming in, Zooming Out

### compress

```haskell
Type: compress :: (Time, Time) -> Pattern a -> Pattern a
```

`compress` takes a pattern and squeezes it within the specified time span (i.e. the 'arc'). The new resulting pattern is a sped up version of the original.

```haskell
d1 $ compress (1/4, 3/4) $ s "[bd sn]!"
```

In the above example, the pattern will play in an arc spanning from 25% to 75% of the duration of a cycle. It is equivalent to:
```haskell
d1 $ s "~ [bd sn]! ~"
```

Another example, where all events are different:
```haskell
d1 $ compress (1/4, 3/4) $ n (run 4) # s "arpy"
```

It differs from `zoom` in that it preserves the original pattern but it speeds up its events so to match with the new time period.

### zoom

```haskell
Type: zoom :: (Time, Time) -> Pattern a -> Pattern a
```

Plays a portion of a pattern, specified by the beginning and end of a time span (known as an 'arc'). The new resulting pattern is played over the time period of the original pattern:

```haskell
d1 $ zoom (0.25, 0.75) $ sound "bd*2 hh*3 [sn bd]*2 drum"
```

In the pattern above, zoom is used with an arc from 25% to 75%. It is equivalent to this pattern:

```haskell
d1 $ sound "hh*3 [sn bd]*2"
```

Here’s an example of it being used with a conditional:
```haskell
d1 $ every 4 (zoom (0.25, 0.75)) $ sound "bd*2 hh*3 [sn bd]*2 drum"
```

### within

```haskell
Type: within :: Arc -> (Pattern a -> Pattern a) -> Pattern a -> Pattern a
```

Use `within` to apply a function to only a part of a pattern. within takes two arguments: a start time and an end time, specified as floats between `0` and `1`, which are applied to the relevant pattern. Note that the second argument must be greater than the first for the function to have any effect.

For example, to apply `fast 2` to only the first half of a pattern:
```haskell
d1 $ within (0, 0.5) (fast 2) $ sound "bd*2 sn lt mt hh hh hh hh"
```

Or, to apply `(# speed "0.5")` to only the last quarter of a pattern:

```haskell
d1 $ within (0.75, 1) (# speed "0.5") $ sound "bd*2 sn lt mt hh hh hh hh"
```

### stretch

```haskell
Type: stretch :: Pattern a -> Pattern a
```

Stretch takes a pattern, and if there's silences at the start or end of the current cycle, it will zoom in to avoid them.

```haskell
d1 $ note (stretch "~ 0 1 5 8*4 ~") # s "superpiano"
-- is the same as
d1 $ note "0 1 5 8*4" # s "superpiano"
```

You can pattern silences on the extremes of a cycle to make changes to the rhythm:

```haskell
d1 $ note (stretch "~ <0 ~> 1 5 8*4 ~") # s "superpiano"
```

## Shifting time


### off

```haskell
Type: off :: Pattern Time -> (Pattern a -> Pattern a) -> Pattern a -> Pattern a
```

`off` is similar to superimpose, in that it applies a function to a pattern, and layers up the results on top of the original pattern. The difference is that `off` takes an extra pattern being a time (in cycles) to shift the transformed version of the pattern by.

The following plays a pattern on top of itself, but offset by an eighth of a cycle, with a distorting bitcrush effect applied:

```haskell
d1 $ off 0.125 (# crush 2) $ sound "bd [~ sn:2] mt lt*2"
```

The following makes arpeggios by adding offset patterns that are shifted up the scale:

```haskell
d1 $ slow 2 $
  n (off 0.25 (+12) $ off 0.125 (+7) $ slow 2 "c(3,8) a(3,8,2) f(3,8) e(3,8,4)")
  # sound "superpiano"
```

### press

```haskell
Type: press :: Pattern a -> Pattern a
```

`press` delays a sound for half the time in its slot. In mini notation terms, it basically turns every instance of `a` into `[~ a]`. Every beat then becomes an offbeat, and so the overall effect is to syncopate a pattern.

```haskell
do
  resetCycles
  d1 $ stack [
    press $ n "~ c'maj ~ c'maj" # s "superpiano" # gain 0.9 # pan 0.6,
    s "[bd,clap sd bd sd]" # pan 0.4
    ] # cps (90/60/4)
```

In this example, you can hear that the piano chords play between the snare and the bass drum. In 4/4 time, they are playing in the 2 and a half, and 4 and a half beats.

```haskell
do
  resetCycles
  d1 $ stack [
    press $ n "~ [c'maj ~] ~ ~" # s "superpiano" # gain 0.9 # pan 0.6,
    press $ n "~ g'maj ~ ~" # s "superpiano" # gain 0.9 # pan 0.4,
    s "[bd,clap sd bd sd]"
    ] # cps (90/60/4)
```

Here, the C major chord plays before the G major. As the slot that occupies the C chord is that of one eighth note, it is displaced by `press` only a sixteenth note.

### pressBy

```haskell
Type: pressBy :: Pattern Time -> Pattern a -> Pattern a
```

`pressBy` is similar to `press`, but it takes one additional parameter which is the displacement of the pattern, from 0 (inclusive) to 1 (exclusive). `pressBy 0.5` is equivalent to `press`.

You can pattern the displacement to create interesting rhythmic effects:

```haskell
d1 $ stack [
s "bd sd bd sd",
pressBy "<0 0.5>" $ s "co:2*4"
]
```

```haskell
d1 $ stack [
s "[bd,co sd bd sd]",
pressBy "<0 0.25 0.5 0.75>" $ s "cp"
]
```

### rotL

```haskell
Type: rotL :: Time -> Pattern a -> Pattern a -> Pattern a
```

`rotL` Shifts a pattern back in time by the given amount, expressed in cycles. This will skip to the fourth cycle when evaluated:

```haskell
do
{
  resetCycles;
  d1 $ rotL 4 $ seqP [
    (0, 12, sound "bd bd*2"),
    (4, 12, sound "hh*2 [sn cp] cp future*4"),
    (8, 12, sound (samples "arpy*8" (run 16)))
  ]
}
```

Useful when building and testing out longer sequences.

### rotR

`rotR` is the opposite of `rotL` as it shifts the pattern forwards in time.

### spin

```haskell
Type: spin :: Pattern Int -> ControlPattern -> ControlPattern
```

`spin` will play the given number of copies of the given control pattern at once. For `n` copies, each successive copy will be offset in time by an additional `1/n` of a cycle, and also panned in space by an additional `n1`. This function works particularly well on multichannel systems.

```haskell
d1 $ slow 3 $ spin 4 $ sound "drum*3 tabla:4 [arpy:2 ~ arpy] [can:2 can:3]"
```

### weave

```haskell
Type: weave :: Time -> ControlPattern -> [ControlPattern] -> ControlPattern
```

`weave` applies one control pattern to a list of other control patterns, with a successive time offset. For example:

```haskell
d1 $ weave 16 (pan sine)
  [sound "bd sn cp",
   sound "casio casio:1",
   sound "[jvbass*2 jvbass:2]/2",
   sound "hc*4"
  ]
```

In the above, the `pan sine` control pattern is slowed down by the given number of cycles, in particular `16`, and applied to all of the given sound patterns. What makes this interesting is that the `pan` control pattern is successively offset for each of the given sound patterns; because the `pan` is closed down by 16 cycles, and there are four patterns, they are 'spread out', i.e. with a gap of four cycles. For this reason, the four patterns seem to chase after each other around the stereo field. Try listening on headphones to hear this more clearly.

You can even have it the other way round, and have the effect parameters chasing after each other around a sound parameter, like this:

```haskell
d1 $ weave 16 (sound "arpy" >| n (run 8))
  [vowel "a e i",
   vowel "i [i o] o u",
   vowel "[e o]/3 [i o u]/2",
   speed "1 2 3"
  ]
```

### weaveWith

```haskell
Type: weaveWith :: Time -> Pattern a -> [Pattern a -> Pattern a] -> Pattern a
```

`weaveWith` (formerly known as `weave'`) is similar to the above, but `weaves` with a list of functions, rather than a list of controls. For example:
```haskell
d1 $ weaveWith 3 (sound "bd [sn drum:2*2] bd*2 [sn drum:1]")
  [fast 2,
   (# speed "0.5"),
   chop 16
  ]
```

## Reversing time
### rev

```haskell
Type: rev :: Pattern a -> Pattern a
```

`rev` returns a 'reversed' version of the given pattern.

For example `rev "1 [~ 2] ~ 3"` is equivalent to rev `"3 ~ [2 ~] 1"`.

Note that `rev` reverses on a cycle-by-cycle basis. This means that `rev (slow 2 "1 2 3 4")` would actually result in `(slow 2 "2 1 4 3")`. This is because the `slow 2 `makes the repeating pattern last two cycles, each of which is reversed independently.

In practice `rev` is generally used with conditionals, for example with `every`:

```haskell
d1 $ every 3 rev $ n "0 1 [~ 2] 3" # sound "arpy"
```
or `jux`:

```haskell
d1 $ jux rev $ n (iter 4 "0 1 [~ 2] 3") # sound "arpy"
```

### jux

```haskell
Type: jux :: (ControlPattern -> ControlPattern) -> ControlPattern -> ControlPattern
```

The `jux` function creates strange stereo effects, by applying a function to a pattern, but only in the right-hand channel. For example, the following reverses the pattern on the righthand side:

```haskell
d1 $ slow 32 $ jux (rev) $ striate' 32 (1/16) $ sound "bev"
```

When passing functions to functions like `jux` and `every`, it’s possible to chain multiple transforms together with ., for example this both reverses and halves the playback speed of the pattern in the righthand channel:

```haskell
d1 $ slow 32 $ jux ((# speed "0.5") . rev) $ striate' 32 (1/16) $ sound "bev"
```

### juxBy

```haskell
Type: juxBy :: Pattern Double -> (ControlPattern -> ControlPattern) -> ControlPattern -> ControlPattern
```

With `jux`, the original and effected versions of the pattern are panned hard left and right (i.e., panned at 0 and 1). This can be a bit much, especially when listening on headphones. The variant `juxBy` has an additional parameter, which brings the channel closer to the centre. For example:
```haskell
d1 $ juxBy 0.5 (fast 2) $ sound "bd sn:1"
```

In the above, the two versions of the pattern would be panned at `0.25` and `0.75`, rather than `0` and `1`.

## Swing

### swingBy

```haskell
Type: swingBy :: Pattern Time -> Pattern Time -> Pattern a -> Pattern a
```

The function `swingBy x n` breaks each cycle into `n` slices, and then delays events in the second half of each slice by the amount `x`, which is relative to the size of the (half) slice. So if `x` is `0` it does nothing, `0.5` delays for half the note duration, and `1` will wrap around to doing nothing again. The end result is a shuffle or swing-like rhythm. For example:

```haskell
d1 $ swingBy (1/3) 4 $ sound "hh*8"
```

will delay every other `"hh" 1/3` of the way to the next `"hh"`.

### swing

```haskell
Type: swing :: Pattern Time -> Pattern a -> Pattern a
```

`swing` is an alias for `swingBy (1/3)`.

### ghost

```haskell
Type: ghost :: Pattern ValueMap -> Pattern ValueMap
```

`ghost` adds quieter, pitch-shifted, copies of an event after the event, emulating ghost notes that are common in drumming patterns.

```haskell
d1 $ stack [ ghost $ sound "~ sn", sound "bd*2 [~ bd]" ]
```

The example above creates a kick snare pattern with ghost notes applied to the snare hit.

### ghost'

```haskell
Type: ghost' :: Time -> Pattern ValueMap -> Pattern ValueMap
```

`ghost'` is a variation from `ghost` above, where you can also specify the base delay used to create the pattern of ghosts notes.

```haskell
d1 $ stack [ ghost' (1/16) $ sound "~ sn", sound "bd*2 [~ bd]" ]
```

The example above creates a kick snare pattern with ghost notes applied to the snare hit. The 1/16 is a sixteenth of a cycle, but that doesn't mean ghost notes will be displaced exactly by this amount: this is just the base value from where repetitions are calculated.

### ghostWith

```haskell
Type: ghostWith :: Time -> (Pattern a -> Pattern a) -> Pattern a -> Pattern a
```

This variation of `ghost` (formerly named `ghost''`) adds another parameter to the ones present in `ghost'`, which is the function used to modify the ghost notes. So here you can decide which changes will be applied to the ghost notes compared to the original notes.

```haskell
d1 $ slow 2 $ ghostWith (1/8) (id) $ sound "sn"
```

In this first example, ghost notes will be identical than the original.

```haskell
d1 $ slow 2 $ ghostWith (1/16) ((|*| gain 1.1) . (|> begin 0.05)) $ sound "sn"
```

The example above applies ghost notes to the snare hit, but these notes will be louder, not quieter, and the sample will have it's beginning slightly cut.

## Inside and outside

### inside

```haskell
Type: inside :: Pattern Time -> (Pattern a -> Pattern b) -> Pattern a -> Pattern b
```
`inside` carries out an operation 'inside' a cycle. For example, `while rev "0 1 2 3 4 5 6 7"` is the same as `"7 6 5 4 3 2 1 0"`, `inside 2 rev "0 1 2 3 4 5 6 7"` gives `"3 2 1 0 7 6 5 4"`.

What this function is really doing is 'slowing down' the pattern by a given factor, applying the given function to it, and then 'speeding it up' by the same factor. In other words, this:
```haskell
inside 2 rev "0 1 2 3 4 5 6 7"
```
Is doing this:
```haskell
fast 2 $ rev $ slow 2 "0 1 2 3 4 5 6 7"
```

.. so rather than whole cycles, each half of a cycle is reversed.

### outside
```haskell
Type: outside :: Pattern Time -> (Pattern a -> Pattern b) -> Pattern a -> Pattern b
```
`outside` is the inverse of the `inside` function. `outside` applies its function outside the cycle. Say you have a pattern that takes 4 cycles to repeat and apply the `rev` function:

```haskell
d1 $ rev $ cat [s "bd bd sn",s "sn sn bd", s"lt lt sd", s "sd sd bd"]
```
The above generates:
```haskell
d1 $ rev $ cat [s "sn bd bd",s "bd sn sn", s "sd lt lt", s "bd sd sd"]
```

However if you apply `outside`:
```haskell
d1 $ outside 4 (rev) $ cat [s "bd bd sn",s "sn sn bd", s"lt lt sd", s "sd sd bd"]
```

The result` is:
```haskell
d1 $ rev $ cat [s "bd sd sd", s "sd lt lt", s "sn sn bd", s "bd bd sn"]
```

Notice the whole idea has been reversed. What this function is really doing is 'speeding up' the pattern by a given factor, applying the given function to it, and then 'slowing it down' by the same factor. In other words, this:
```haskell
d1 $ slow 4 $ rev $ fast 4 $ cat [s "bd bd sn",s "sn sn bd", s"lt lt sd", s "sd sd bd"]
```
This compresses the idea into a single cycle before `rev` operates and then slows it back to the original speed.

## Delay functions

See also: [Effects/Delay](https://tidalcycles.org/docs/reference/audio_effects#delay)

### echo

```haskell
Type: echo :: Pattern Integer -> Pattern Rational -> Pattern Double -> ControlPattern -> ControlPattern
```

`echo` applies a type of delay to a pattern. It has three parameters, which could be called `depth`, `time` and `feedback`. `depth` is and integer, and `time` and `feedback` are floating point numbers.

This adds a bit of echo:

```haskell
d1 $ echo 4 0.2 0.5 $ sound "bd sn"
```

The above results in `4` echos, each one `50%` (that's the `0.5`) quieter than the last, with 1/5th (that's the `0.2`) of a cycle between them.

It is possible to reverse the echo:
```haskell
d1 $ echo 4 (-0.2) 0.5 $ sound "bd sn"
```

### echoWith

```haskell
Type: echoWith :: Pattern Int -> Pattern Time -> (Pattern a -> Pattern a) -> Pattern a -> Pattern a
```

`echoWith` is similar to `echo` described above, but instead of just decreasing volume to produce echoes, `echoWith` applies a function each step and overlays the result delayed by the given time.
```haskell
d1 $ echoWith 3 (1/3) (# vowel "{a e i o u}%2") $ sound "bd sn"
```

In this case there are two overlays delayed by 1/3 of a cycle, where each has the vowel filter applied.
```haskell
d1 $ echoWith 4 (1/6) (|* speed "1.5") $ sound "arpy arpy:2"
```
In the above, three versions are put on top, with each step getting higher in pitch as `|* speed "1.5"` is successively applied.

### stut

```haskell
Type: stut :: Pattern Integer -> Pattern Double -> Pattern Rational -> ControlPattern -> ControlPattern
```
_Deprecated_: use [echo](https://tidalcycles.org/docs/reference/time#echo) instead.

### stutWith

```haskell
Type: stutWith :: Pattern Int -> Pattern Time -> (Pattern a -> Pattern a) -> Pattern a -> Pattern a
```

_Deprecated_: use [echoWith](https://tidalcycles.org/docs/reference/time#echowith) instead.

## Time shorthands

When dealing with time functions, many times we need to specify times shorter than a cycle by using fractions or decimal numbers.

Alternately, we can use textual shorthands to refer to the most common durations.

For example, we can swap `0.25` or `1/4` for the shorthand `q`, which stands for a *q*uarter of a cycle.

These three examples are equivalent:

```haskell
d1 $ off 0.25 (|+ n 7) $ n "c e" # sound "supermandolin"
d1 $ off (1/4) (|+ n 7) $ n "c e" # sound "supermandolin"
d1 $ off "q" (|+ n 7) $ n "c e" # sound "supermandolin"
```

Here's the current list of shorthands available:

```
w = 1 (whole)
h = 1/2 = 0.5 (half)
t = 1/3 (third)
q = 1/4 = 0.25 (quarter)
f = 1/5 = 0.2 (fifth)
x = 1/6 (siXth)
e = 1/8 = 0.125 (eighth)
s = 1/16 = 0.0624 (sixteenth)
```

We can prefix these shorthand with a number to have multiples. These two examples sound the same:

```haskell
d1 $ stack [
s "[bd,co sd bd sd]",
pressBy "<0 0.25 0.5 0.75>" $ s "cp"
]

d1 $ stack [
s "[bd,co sd bd sd]",
pressBy "<0 q h 3q>" $ s "cp"
]
```

For a 32nd, you could do `0.5s`:

```haskell
d1 $ echo 4 "0.5s" 0.9 $ sound "hh"
```

You can only use these shorthands on any function that receives a `Pattern`. This will work:

```hasell
d1 $ s "bd" # delaytime "x" # delay 0.8 # delayfb 0.4
```

But this won't (as `compress` needs a `Time`, not a `Pattern Time`):

```haskell
d1 $ compress ("q", "3q") $ s "[bd sn]!" -- ERROR
```
