---
layout: post
title: "From ANSI to Ergo: Remapping the Transition"
date: 2026-06-27
tags: [Keyboards, Ergonomics, KMonad, Kanata, Keyd, Kenkyo]
---

A common story in the mechanical keyboard community goes like this: someone starts suffering from repetitive strain injury (RSI) or general hand pain and decides to transition to an ergonomic keyboard. To make the transition easier, they look for a large layout with plenty of keys. However, once the keyboard arrives, they realize the learning curve is steep. Frustrated, they ask for help in forums, convince themselves they bought the wrong board, and buy another one. Those who don't quit often end up with an expensive collection of ornamental keyboards.

I wanted to avoid this trap. Instead of buying a series of physical keyboards, I decided to find a layout that adapted directly to my hands, requiring minimal movement and finger displacement. The goal: a 34-key, column-staggered, split ergonomic keyboard layout. 

But how do you transition from a standard 104-key ANSI keyboard to a layout with only 34 keys? Two things need to happen:
1. Identify which keys are truly redundant or unnecessary.
2. Efficiently map the remaining keys using a layered system.

### Bridging the Gap in Software

The main challenge was that my standard keyboard was not programmable. I couldn't just change the physical keys or the firmware. That’s when I discovered [KMonad](https://github.com/kmonad/kmonad)—a powerful tool that allows you to reprogram any keyboard at the software/OS level. This enabled me to test and refine a 34-key layout directly on my standard ANSI keyboard before purchasing any ergonomic hardware.

Over time, I migrated my setup to more modern and lightweight alternatives:
- [keyd](https://github.com/rvaiya/keyd): A highly efficient keyboard remapping daemon for Linux.
- [Kanata](https://github.com/jtroo/kanata): A cross-platform remapper focused on advanced features like tap-hold, home row mods, and custom combos.

### Introducing Kenkyo (謙虚)

Through this software-first approach, I open-sourced my custom layout under the name [Kenkyo](https://github.com/argenkiwi/kenkyo) (which means *humility* in Japanese). I chose this name as a contrast to the dominant [Miryoku](https://github.com/manna-harbour/miryoku) (*allure* in Japanese) layout. 

While Miryoku is highly custom and fully featured, Kenkyo aims to be a humble, non-disruptive, software-driven bridge that allows standard ANSI keyboard users to gradually transition to a layered 34-key workflow.

By remapping the keyboard in software first, you can build the muscle memory for a split, column-staggered layout without spending a fortune on physical boards you might end up shelving.
