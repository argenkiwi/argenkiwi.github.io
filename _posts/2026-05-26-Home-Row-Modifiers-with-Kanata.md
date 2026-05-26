---
layout: post
title: "Home Row Modifiers with Kanata"
date: 2026-05-26
tags: [Kanata, Keyboard, Home Row Modifiers, Kenkyo]
---

Home row modifiers are a popular technique for reducing finger movement: modifier keys like Shift, Ctrl, or Alt are assigned to the home row keys (A, S, D, F, etc.) as hold actions, while their regular characters are emitted on tap. The challenge is making this feel natural — avoiding accidental modifier triggers while typing at speed, and keeping modifiers responsive when you actually need them.

This post describes my current approach, implemented in [Kenkyo](https://github.com/argenkiwi/kenkyo/tree/kanata/keys) using [Kanata](https://github.com/jtroo/kanata).

### Configuration Variables

```scheme
(defvar
  streak-count 3
  streak-time 450
  tap-timeout 200
  hold-timeout 300
  chord-timeout 50
  lkeys (
    q w e r t
    a s   f g
    z   c   b
  )

  rkeys (
    y u i o p
    h j   l ;
    n   ,   /
  )
)
```

These variables let you tune the behaviour without touching the templates themselves. The `lkeys` and `rkeys` lists define which keys count as "same-side" neighbours — important for the `tap-hold-release-keys` resolution strategy described below.

### The `flowtap` Template

```scheme
(deftemplate flowtap (flow tap)
  (switch 
    ((key-timing $streak-count less-than $streak-time)) $flow break
    () $tap break
  )
)
```

The name is borrowed from the QMK [flow tap](https://docs.qmk.fm/tap_hold#flow-tap) concept. Kanata keeps track of the time between recent key presses, so we can detect whether the user is in the middle of typing (a _streak_) or is pausing between words.

If `streak-count` or more keys have been pressed within `streak-time` milliseconds, the `flow` action is used; otherwise, the `tap` action applies. With the values above, three keys pressed within 450 ms means "typing at speed". Set `streak-count` to at least the maximum number of modifiers you might hold simultaneously to avoid false negatives.

### The `charmod` Template

```scheme
(deftemplate charmod (char mod)
  (t! flowtap
    $char
    (tap-hold-release-timeout $tap-timeout $hold-timeout $char $mod $char reset-timeout-on-press)
  )
) 
```

`charmod` is the simplest home row modifier strategy. When idle, it uses `tap-hold-release-timeout` to distinguish a tap (emit the character) from a hold (activate the modifier). When typing at speed, it simply emits the character, bypassing the hold logic entirely.

This is Kenkyo's default for bottom-row modifiers and for the SpaceFN pattern (spacebar as a layer modifier). It works well when you have dedicated thumb-cluster keys for typing modifiers like Shift. If you don't, typing Shift+something at speed requires a deliberate pause — see `charmodkeys` below.

### The `charmodkeys` Template

```scheme
(deftemplate charmodkeys (char mod keys)
  (t! flowtap
    (tap-hold-release-keys $tap-timeout $hold-timeout $char $mod $keys)
    (tap-hold-release-timeout $tap-timeout $hold-timeout $char $mod $char reset-timeout-on-press)
  )
)
```

`charmodkeys` trades a small amount of latency for more responsive modifiers while typing at speed. Instead of immediately emitting the character during a streak, it uses `tap-hold-release-keys`: the key resolves to a tap or hold based on what key is pressed or released next.

The `$keys` list is the set of keys that, when pressed after this key, force a tap resolution — essentially "if the next key is on the same side as this one, this was almost certainly a character, not a modifier". This makes Shift and AltGr usable at typing speed without needing a pause.

### Left and Right Side Helpers

```scheme
(deftemplate lcharmod (char mod)
  (t! charmodkeys $char $mod $lkeys)
)

(deftemplate rcharmod (char mod)
  (t! charmodkeys $char $mod $rkeys)
)
```

These are convenience wrappers that apply `charmodkeys` with the appropriate same-side key list. A modifier on the left hand uses `$lkeys` (so pressing another left-hand key forces a tap), and vice versa.

For keys that don't need to be used while typing at speed — bottom-row modifiers, the spacebar — `charmod` remains the right choice.

### Putting It Together

The general rule is:

- Use `lcharmod` / `rcharmod` for modifiers you regularly combine with keys on the same hand while typing (primarily Shift and AltGr on each side).
- Use `charmod` for everything else: bottom-row modifiers, the spacebar, or any modifier that only needs to work during deliberate pauses.

The `streak-time` value is worth tuning to your own typing speed. Starting at 325 ms and increasing toward 450 ms will make output smoother if you find characters are occasionally missed mid-word.
