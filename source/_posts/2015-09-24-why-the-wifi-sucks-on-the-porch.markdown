---
layout: post
title: "Why the WiFi sucks on the porch"
date: 2015-09-24 16:31:30 -0500
comments: true
categories: [wireless, networking, WiFi, school]
---

## Why is WiFi so slow on the porch?

The problem at my house is that WiFi connectivity is generally fine inside the
house, but while sitting on the porch it is nearly impossible for anyone to
browse the Hinternets. I think a lesson of my [Wireless Networking][qiu] course
explains this observation.

In Wireless networking we have the situation called the [**Hidden Terminal
Problem**][htp], which is the following. Node `A` is too far from node `C` to
hear its transmissions, but both are in range of node `B` which sits between
`A` and `C`. Suppose `C` is currently transmitting data to `B`. Now `A` checks
whether any transmissions are currently happening, and finds that there are
none (because it is out of range of `C`). So `A` goes ahead and sends data to
`B`. Now `B` can't understand either data packet because they collided and
interfered with each other in an unrecoverable way because they were both sent
in the same channel.

I think the porch scenario is simply an example of the _Hidden Terminal
Problem_ above. In my room, my laptop can hear many of my neighbors' WiFi LAN
networks, but it's the same set that my WiFi router sitting in the closet can
hear. However outside, where there's less cause for signal attenuation, my
laptop can hear many more WiFi LAN networks than the router inside. So the
router checks whether there is congestion, doesn't hear any, and sends the
data. But there *is*, in fact, congestion, and my computer doesn't receive the
data properly. The data is therefore not _ACKd_, the router times out on the
_ACK_, and has to retransmit, and so on. This makes for a *far* slower
Hinterconnectivity outside on the porch than in my room.

Maybe the lesson learnt is that the WiFi router should be situated in a place
where it can hear more of the outside noise so that it can compensate better
for that noise.

[htp]: http://www.wikiwand.com/en/Hidden_node_problem
[qiu]: http://www.cs.utexas.edu/~lili/classes/F15-CS386W/
