---
layout: post
title: "From ANSI to Ergo: Remapping the Transition"
date: 2026-06-27
tags: [Keyboards, Ergonomics, KMonad, Kanata, Keyd, Kenkyo]
---

A common story in the mechanical keyboard community goes like this: someone starts suffering from repetitive strain injury (RSI) or general hand pain and decides to transition to an ergonomic keyboard. To make the transition easier, they look for a large layout with plenty of keys. However, once the keyboard arrives, they realize the learning curve is steep. Frustrated, they ask for help in forums, convince themselves they bought the wrong board, and buy another one. Those who don't quit often end up with an expensive collection of ornamental keyboards.

I wanted to avoid this trap. Instead of buying a series of physical keyboards until I found the one, I learned what features my ideal keyboard should have. I settled on the following: a 34-key, column-staggered, split ergonomic keyboard layout. 

Next, I needed to find a way to reduce the number of keys I was using on my ANSI keyboard to 34 or fewer by:

1. Identifying which keys were truly redundant or unnecessary.
2. Efficiently mapping the remaining keys using a layered system.

But didn't I need a programmable keyboard for that?

### Bridging the Gap in Software

That’s when I discovered [KMonad](https://github.com/kmonad/kmonad): a powerful tool that allows you to reprogram any keyboard at the software/OS level. This enabled me to test and refine a 34-key layout directly on my standard ANSI keyboard before purchasing any ergonomic hardware.

### Introducing Kenkyo (謙虚)

Through this software-first approach, I open-sourced my custom layout under the name [Kenkyo](https://github.com/argenkiwi/kenkyo) (which means *humility* in Japanese). I chose this name as a contrast to the dominant [Miryoku](https://github.com/manna-harbour/miryoku) (*allure* in Japanese) layout. 

While Miryoku is highly custom and fully featured, Kenkyo aims to be a simple, non-disruptive, software-driven bridge that allows standard ANSI keyboard users to gradually transition to a layered 34-key workflow.

Over time, I migrated my setup to more modern and lightweight alternatives:
- [keyd](https://github.com/rvaiya/keyd): A highly efficient keyboard remapping daemon for Linux.
- [Kanata](https://github.com/jtroo/kanata): A cross-platform remapper focused on advanced features like tap-hold, home row mods, and custom combos.

By remapping the keyboard in software first, I was able to build muscle memory. By the time I received my first ergonomic keyboard, I could be productive with it right away. To this day, I still use software for my layout, which allows me to switch between my ergo and laptop keyboards painlessly.
