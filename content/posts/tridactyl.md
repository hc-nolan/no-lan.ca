---
date: '2025-07-21T18:08:18-04:00'
draft: true
title: 'Tridactyl'
---

Using the mouse sucks. Navigating purely with the keyboard is awesome. Here's how you can use most modern websites without ever taking your hand away from the keyboard.

<!--more-->

## Why dislike mice?

Throughout 2024 my preferred IDE and general text editor became Neovim. I spent probably about 500 hours over the year educating myself about it and, frankly, completely drank the Kool-Aid. I am now obsessed with Neovim and earnestly think that its community is one of the greatest of all open source software communities.

The number one thing I love about Neovim is **not having to move my hands from the keyboard**. My wrist has been broken twice, and I find it quite straining to constantly move my hand to the mouse and back to the keyboard. Neovim inherits the core design principles from Vim and Vi before it; Vi was built before computer mice were commonplace and most computer users did everything by keyboard.

Beyond the practical dimension, I just find clicking things with the mouse to be dissasitfying and slow. Learning the Vim keybindings and flying through menial tasks at the blink of an eye is incredibly fun (to me).

After spending most of 2024 building new habits suited to my Neovim workflow, I started to get really annoyed by the everyday web browsing experience - particularly, how much I had to use the mouse for things that would be much faster and less taxing on my wrist if I could do them via keyboard. At first I considered setting up macros that would manipulate the cursor, but then I thought about how there is a Vim-mode plugin for every IDE, so I [looked into Vim-mode plugins for the browser](https://infosec.exchange/@chunned/113079673827758773), and found [Tridactyl](https://github.com/tridactyl/tridactyl).

### Note for heathens (non-Firefox users)

Tridactyl is a Firefox-only extension. While I would love to go off on a tangent trying to convince you to use Firefox or one of its derivatives, to keep things on topic, I'll just say that there are some alternatives for Chrome. The best seems to be [Vimium](https://vimium.github.io/). I used it a little bit at work and it has all the basic features of Tridactyl, but is missing many of the more advanced ones. Vimium is available for Firefox as well.

## Tridactyl

If you are interested in using Tridactyl, I recommend closing this post and just following the tutorial with `:tutor`. Otherwise, here are the basics.

In the spirit of Vi, Tridactyl introduces different modes for any given browsing window:
- Normal mode: what you're in now. Press `Escape` from any other mode to get back to this mode.
- Hint mode: highlight elements to perform actions on. Entered with `f`
- Visual mode: highlight elements to select. Entered with `v`
- Command mode: execute commands manually, same as Vim's command mode. Entered with `:`
- Ignore mode: disable Tridactyl for the current page. Entered with `Ctrl-Alt-Esc`
