---
layout: post
title: "What actually is SSH"
date: 2016-03-06 14:18:53 -0800
comments: true
categories: [networking, protocols, IETF, SSH, terminal, shell, remote execution, system administration, devops]
---

### SSH Tunneling

_"Tunnelling", with two ells, is the British spelling._

A few months ago, I downloaded a tool (an ELK stack) that didn't work right
off the bat due to some sort of misconfiguration. It was running in a Vagrant-
made virtual machine (VM) on my laptop. The Vagrant setup script had forwarded
a local port on my laptop into the VM. So in order to debug it, my coworker
configured a chain of tunnels that enabled him to SSH into the VM on my
laptop.

In the back of my mind, I spent the next few weeks trying to figure out what
SSH port-forwarding is and how its syntax works, then another few weeks to
figure out what reverse port-forwarding is, and another few weeks to find
practical use-cases for each.

Here is my executive summary

    ssh -L [<localhost>:]<localport>:<remotehost>:<remoteport> <gateway>

By default, `<localhost>` will be `localhost`.

What this does is, start a serversocket listening to local address
`localhost:localport` using the "SSH client". When a client establishes a
connection to that address, traffic received from that client will be
encrypted, and forwarded to the `sshd`[aemon] listening on port 22 of
`gateway`. (Only) after the gateway receives this traffic, the `sshd` will
establish a (normal, unencrypted) connection to remote address
`remotehost:remoteport`, and forward the data originally received by the SSH
client there. Response traffic originating from `remotehost:remoteport` will
go back to the `sshd`, back through the encrypted tunnel to the SSH client,
and back to the originating client.

_Reverse_ tunneling by contrast, means that traffic _originating_ on the
remote end will be forwarded to the local end.

## What's SSH

One thing that confused me about SSH forwarding, is that if I don't use some
extra flags to disable it, when I set up port forwarding to a another machine,
I also end up with a shell ready executing commands remotely. What is going on
here? It turns out it is doing what is called "remote command execution".

<!-- more -->

Here's what's happening: SSH is capable of multiplexing multiple communication
"channels" over a single encrypted TCP connection. If you just execute the
normal tunneling command given above, it will create two channels, one for
forwarding the tcp traffic, and another for executing commands visually in
your pseudo-terminal but physically in a remote shell.

When you `ssh` onto a remote machine, a particular sequence of things happen
that I think really clarify what the heck's going on. You can see it happening
if you turn on verbose mode (`-v`).

1. You connect via TCP to the standard SSH-daemon port (22) on the remote
   machine
2. You authenticate it via its public key
    * That's why the first time you connect is says "do you want to trust this
      server's fingerprint?" or something like that (I think saying "yes I
      trust it" implies you have verified that the fingerprint does not belong
      to a *man-in-the-middle* attacker)
3. It authenticates you, either by you supplying a password, or by it finding
   your username in its `~/.ssh/authorized_keys` file, and [using what protocol?] you match it with your private key
4. Y'all negotiate an encryption for the session, over which future
   communications will take place
    * [Maybe you have to choose a specific cryptographic algorithm like in TLS?]
5. Now that communications are encrypted, control and channel data are
   transferred using a binary packet format that contains
    * Encrypted packet length -- answers: when does this packet end?
    * Various random paddings -- to fool the bad guys
    * The encrypted (and possibly compressed) payload -- here's the good stuff
    * A message authentication code (MAC) -- find evidence of corruption/tampering
    * Recipient's integer identifier for this channel -- answers: what _channel_
      did this data come in from?
6. To setup a remote shell, your SSH client requests permission to create a
   special channel specifically designed for the purpose of remote shells
    * At this point you tell the server which ID you're allocating to this
      channel, so that it knows how to fill the last item in the format
      specified above
    * Since each channel performs its own flow-control, you also state your
      "window size" (similar to TCP)
        * Both sides maintain a receive window, and once the sender has sent
          enough bytes to fill it, they will not send more data until you say
          that your window now has more room
7. If the server supports "shell" channels, it will tell you which ID it
   allocates for the new shell channel
    * so that you can put the correct ID in SSH packets in this channel
8. Then you tell the server your psuedo-terminal's display specifics
    * characters horizontally and vertically, as well as pixels horizontally and
      vertically
    * If you were using X11, you would tell the server your screen's specifics
    * If your dimensions change, you can send an update message to the server
9. Now you can send the server the values of environment variables to use
10. Then you request that the server initiate a shell in which to execute the
    commands you're going to give it. You can specify the path to the specific
    shell to use.
11. Then you start sending commands to execute, which the server will pass to
    the shell it created
12. The server will send you back output received from the shell, formatted
    for your terminal
13. You can also send signals for the server to pass to its shell
14. When the remote shell command terminates, the server will send you its
    "exit-status", or if there was a violent termination, it will send an
    "exit-signal"

Note this communication only used one channel, and there can be however many
channels open over the same SSH connection. This is implemented using the
framing layer that puts the data into the higher-level datagram format
described in list item 5 above. Datagrams are then demultiplexed by the
receiver according to the the channel identifiers in the datagram. So one
channel might be used for forwarding packets received on a port registered
with SSH for forwarding, while another is used for the normal SSH remote shell
execution terminals. So that's how we see the effects of the "-L" feature of
SSH described in the beginning.

### It's like HTTP/2

I don't know many protocols, so a lot of this stuff is new territory for me.
When I got curious, to find out more, I started by reading the Wikipedia page,
because in addition to having its explanation, it often has good references
and links to related topics. From there I followed the reference to the IETF
RFC. Maybe it was because I read the Wikipedia page first, but I found the RFC
much easier to follow, and with a better enunciation of what is going on than
the Wikipedia page. I didn't read the RFC word for word except for the first
few sections, but then, based on its structure and formatting, was able to
find the pieces I was interested in surprisingly fast. The reason for that is
the RFC's authors are familiar with explaning such concepts, and spent months
working as a team to increase the document's utter clarity to reduce the
possibility of miscommunication.

In a previous post I looked at the new HTTP/2 IETF RFC. That protocol contains
a similar framing layer over TCP that allows it to multiplex multiple channels
concurrently over a single underlying TCP socket. IIRC, the reason for that
was to ease issues with "head-of-line blocking", which is where one channel's
being slow ends up slowing down all the other channels because of TCP
unecessarily guaranteeing maintaining FIFO order of packets across multiplexed
channels that have no globally-defined order. At the same time opening up many
TCP connections per client is undesirable for the server. So after having seen
this framing-layer thing in two protocols, now I understand it to be some sort
of "pattern" for allowing multiplexing communications over a single TCP link.
One big advantage is less connections per client on the server side. Also, the
TCP algorithm has a "ramp up" time, i.e. the congestion control starts out
with a rather pessimistic window size. By using only one socket, that ramp up
only must be done once. Surely, better knowledge of TCP would lead to more
insight here.

I like to get at least a basic idea of what my favorite tools are actually
doing. This sort of investigation can only go so far because the modern stack
of technologies is so vast and the "lyfe so shorte". My first computer science
professor encouraged us to press the "I believe" button when something seemed
like magic. Still, lifting some of the veil on one tool I use everyday is
satisfying, and hopefully the time taken for others to get to my (still
minimal) level of understanding is reduced by the explanations above.
