---
layout: post
title: "Basics of Wireless Communication"
date: 2015-09-07 17:45:58 -0500
comments: true
categories: [wifi, networking, internet, wireless, overview, school]
---

I've been doing the readings for my [Wireless Networking course][wificrs] at
UTexas, and in the process have dug into much of the basics of radios and
networks that I had ignored in the past. Here, I will try briefly describe what
I have learned. Maybe not everything I will say here is exactly correct, but I
think it's at least _mostly_ correct.

Let's try to start somewhere near the beginning. Our __goal__ is to transfer a
_information_ from one location `LocSND` to another `LocRCV` _conveniently_.
The way we will accomplish that is by having `LocSND` manipulate the
_electromagnetic field_ around `LocRCV`. More specifically, we will _encode_ a
binary _dataframe_ as _modulations_ of a _radio signal_ around a pre-determined
_carrier frequency_.

How do we _do_ that? 
<!-- more -->
We use an LRC circuit to make electrons oscillate in an
antenna. These oscillating electrons ram into loose and excitable electrons in
the antenna's metal, this releases a photon at a particular frequency. Globally
(i.e. within the entire transmitting antenna), enough photons are being
released that it seems to an external observer looking at the produced
electromagnetic (EM) field like there is a continuous signal being emitted.

So we're sending these EM ripples, which are generally at our carrier
frequency. However, if we just sent a basic frequency, there would be no
_information_ in there, so we have to _modulate_ it. We can modulate its
amplitude (A), phase (phi), and frequency (omega), the 3 free
parameters of the equation (in the top left of the equation in the following
gif from "sengpielaudio")

![sine wave](http://www.sengpielaudio.com/Sinuskurve01.gif)

This would give us
1. __Carrier frequency__ --- the EM frequency _inside_ which our signal is
  encoded
2. __amplitude shift keying (ASK)__ -- send signal at _carrier frequency_ by
   modulating the signal's _amplitude_
3. __frequency shift keying (FSK)__ -- similar but modulates _frequency_
4. __phase shift keying (PSK)__ -- again, but modulates _phase_

One simple method would be to say our carrier frequency is 5 Hz, but our band
is actually [4,6] Hz. So whenever the signal is 4 Hz, that means I'm sending a
0, and if the signal is 6 Hz, it means I'm sending a 1, and a new digit starts
every 1 ms. That would be an example of __FSK__.

A fundamental problem that we must solve is that all senders and receivers of
information via EM fields with their antenna(s) are sharing the a single
_medium_ for transmitting that field (viz. the air, etc.). So if `LocSND` sends
a message to `LocRCV_1`, then `LocRCV_2` sitting one foot away can hear that
message loud and clear. This leads to three major issues: __security__,
__multiplexing__, and __interference__.

To __multiplex__ means to send multiple distinct signals through a single
channel. How are we going to send distinct signals to receivers 1 and 2 in such
a way that both can understand the signal meant for them? We can chop up the
frequency band that our transmitter can use into 2 smaller bands, and use each
of those bands as separate carriers. Then we tune the receivers to pick up
frequencies in their respective bands. This is what is called __frequency
division multiplexing (FDM)__, but we can also multiplex across space, time,
and _code_.

Of course, I've only scratched the very surface of what's going on here, but
that's all the time I've got.

[wificrs]: http://www.cs.utexas.edu/~lili/classes/F15-CS386W/
