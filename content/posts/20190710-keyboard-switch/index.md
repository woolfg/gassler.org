---
title: "The \"painful\" switch from a German to an English keyboard layout on Linux (Fedora)"
date: "2019-07-10"
slug: switch-keyboard-german-to-english-linux-fedora
tags: [linux, german, fedora, coding]
images: [hand.jpg]
summary: "I have used the German keyboard for more than two decades and decided to switch to the English keyboard. See how to switch on Fedora without losing the German Umlauts and read about the \"painful\" experience."
---

Typing on a keyboard as a developer --- the interface between the computer and the human brain. Most probably you also have optimized this "interface" for decades and usually other people have problems to follow you if you make use of your optimized workflow of typing and pressing shortcut combos on your keyboard.

So why the hell should you change your keyboard from a German layout to an English one? The German keyboard layout is quite bad for developers, think about the moves of your fingers to type `{` or `\` on your German keyboard. Especially keyboard-oriented environments like vim were invented using the English layout. Of course, there are even more optimized developer layouts such as http://www.kaufmann.no/roland/dvorak/ but let's stick to the basics.

I heard a lot about other developers who love the English layout so I always wanted to switch to it. Unfortunately, every time I said "now I switch", there was an emergency or a project under time pressure and I switched back due to time constraints to finish the work as quickly as possible. Motivated by an awesome blog post by my colleague [Matthias Endler](https://twitter.com/matthiasendler) https://matthias-endler.de/2018/keyboard/ who loves to optimize things and the opportunity of having less pressure during my sabbatical, I did the switch.

# How to switch from a German to the US keyboard layout using Fedora

I use Fedora and i3 on a Lenovo notebook, so I had to find a good solution for using the US layout while having quick access to all German "Umlauts". It wasn't that easy to find a good solution but based on https://blog.florianheinle.de/englische-tastatur-umlaute I came up with the following solution that works for me:
I added the following part to `/usr/share/X11/xkb/symbols/us` (it gets overwritten by `dnf` updates from time to time, so be aware of that or automate it, for me it was just easier so far to patch the file again when my custom layout gets overwritten)

```
partial alphanumeric_keys
xkb_symbols "wolfi" {

include "us(basic)"
name[Group1]= "English (US, german umlauts on u o a s, euro on e, cent on c)";

key <AD03> { [   e, E, EuroSign, EuroSign ] };
key <AD07> { [   u, U, udiaeresis, Udiaeresis ] };
key <AD09> { [   o, O, odiaeresis, Odiaeresis ] };
key <AC01> { [   a, A,  adiaeresis, Adiaeresis   ] };
key <AC02> { [   s, S, ssharp, U1E9E   ] };
key <AB03> { [   c, C, cent] };

include "level3(ralt_switch)"

};
```

You can activate the custom layout with `setxkbmap us wolfi` which results in an US layout having the Umlauts accessible via the right `Alt` key. I prefer that over the imho very cumbersome "dead keys" layout. To use this layout permanently you can create the file `/etc/X11/xorg.conf.d/00-keyboard.conf` with the content:

```
Section "InputClass"
Identifier "system-keyboard"
MatchIsKeyboard "on"
Option "XkbLayout" "us,at"
Option "XkbVariant" "wolfi,nodeadkeys"
EndSection
```

# Did it work and is it worth it?

I did the switch a couple of weeks ago and also forced me to keep the layout while fixing a severe server outage a couple of days later (writing a lot of shell commands). The biggest problem for me was the German keyboard of my notebook. I wasn't aware of that before, but sometimes I take a really quick look on the keyboard to re-calibrate my hands. While doing this calibration look the key labels triggered my German keyboard layout in my mind. I was always falling back to German and was pressing German keys.

{{< figure src="keyboard.jpg" title="I had to put stickers on my keyboard to trick my brain" >}}

The learned patterns were just too strong, so I decided to buy English keyboard layout stickers on Amazon (see on the picture). Having those, I could already saw an improvement in the first hour and after a couple of days later, I was sold: almost no mistakes, faster programming, and finally, the understanding why vim commands are that strange, at least on a German keyboard ;)

An now, after some weeks, I am still satisfied and happy that I did the switch. I am faster and pair programming gets easier as most people use English keyboards. Maybe it is painful for a couple of days, but it is definitely worth it, and it is also good to know that you can still break patterns that you have repeated and practiced over 20 years.