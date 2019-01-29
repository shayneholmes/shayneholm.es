---
title: "Writing on the go"
date: 2019-01-29T08:05:43-08:00
categories: [technology, productivity]
---
I get to work via public transit, which gives me some time to myself to think, read, and write. To make the most of it, I try to have full-featured tools at hand. For writing, a proper keyboard is crucial: It gets out of my way and lets me write down what's on my mind with a mininumum of fuss. To avoid toting around the bulk (and the distraction) of a full laptop, I use a Bluetooth keyboard with my (Android) phone.

## Removing distraction

Using a Bluetooth keyboard with phone, in a moving bus, has (at least) three headaches:

1. The display is small, so you either see just a few sentences, or squint at the tiny letters.
1. Bumps in the road jostle the display, making it even harder to keep tabs on what you're writing.
1. Other passengers ogle your display.

My solution to all three of these headaches is to just put my phone in my pocket and "type blind". It does introduce the _additional_ headache of not being able to see what I'm typing, but that just makes my writing more stream-of-consciousness, and I've chosen to take that as a positive.

(There's also the issue of learning to touch-type, but that wasn't an issue for me, and is out of scope for this post.)

## Removing surprises

Since I can't see my phone while I'm typing in it, I can't react to changes in the system state: One bump of the home button, and I'm not writing a document anymore but sending keystrokes to my homescreen, which will just ignore them (hopefully).

I can't just turn off the screen (Android doesn't seem to permit typing into an active window with the screen off), so I've got a touch blocker that I enable when I'm in "blind typing" mode. (I used the factotum [Tasker](https://tasker.joaoapps.com/) to make it, but there are some comparable apps on the Play Store.)

With touch out of the picture, there are still things that can disrupt my writing. Most of the things I've found are software bugs in handling cursor key presses.[^1]  After testing over twenty different "note taking" apps, all with various quirks, I punted on them all and went all-console with [Termux](https://termux.com/) and [Vim](https://www.vim.org/). Cursor focus issues begone! (As a bonus, I pulled in my .vimrc from my [dotfiles](https://github.com/shayneholmes/dotfiles) and got my full vim environment working, so I can easily edit and spellcheck on the go, too.)

## Removing friction

With the writing experience sufficiently unsurprising, all that remained was implementing some process improvements to get writing with less fuss. First up, I used Android's keyboard shortcuts to put `Win-X` to Termux. And then, since most of my writing goes in a time-indexed journal, I made the following alias:

```zsh
alias j='vim -c "startinsert!" journal-$(date +%F).txt'
```

Now, with just one `<M-x>j<CR>`, I am off and writing!


[^1]: E.g.: Go to the end of a text field, then press the right arrow. Instantly, your focus changes to a new component.
