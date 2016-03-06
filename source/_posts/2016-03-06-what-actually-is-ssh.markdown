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
set up a reverse port-forward from my laptop onto an internal server. Then he
set up a forward port-forward to that server from his laptop. Now commands
issued at his laptop's terminal were being executed on the VM on my laptop. My
mind was so utterly blown I became dizzy.

It took me weeks to figure out what the SSH port-forwarding is and how its
syntax works, and another few weeks to figure out what reverse port-forwarding
is, and another few weeks to find practical use-cases for each.

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

That's a mouthful, but I think it's a pretty clear explanation.

_Reverse_ tunneling by contrast, means that traffic _originating_ on the
remote end will be forwarded to the local end.

## What's SSH

One thing that confused me about SSH forwarding, is that if I don't use some
extra flags to disable it, when I set up port forwarding to a another machine,
I also end up with a shell ready executing commands remotely. What is going on
here? What does it mean to do remote command execution?

So here's what's happening: SSH is capable of multiplexing multiple
communication "channels" over a single encrypted TCP connection. If you just
execute the normal tunneling command given above, it will create two channels,
one for forwarding the tcp traffic, and another for executing commands
visually in your pseudo-terminal but physically in a remote shell.

When you `ssh` onto a remote machine, a particular sequence of things happen
that I think really clarify what the heck's going on.

1. You connect via TCP to the standard SSH-daemon port (22) on the remote
   machine
2. You authenticate it via its public key
    * That's why the first time you connect is says "do you want to trust this
      server's fingerprint?" or something like that (I think saying "yes I
      trust it" implies you have verified that the fingerprint does not belong
      to a *man-in-the-middle* attacker)
3. It authenticate's you, either by you supplying a password, or by it finding
   your username in its `~/.ssh/authorized_keys` file, and having you match it
   with your private key
4. Y'all negotiate an encryption for the session, over which future
   communications will take place
5. Now that communications are encrypted, control and channel data are
   transferred using a binary packet format that contains
    * Encrypted packet length
    * Various random paddings
    * The encrypted (and possibly compressed) payload
    * A message authentication code (MAC)
    * Recipient's integer identifier for this channel
6. To setup a remote shell, your SSH client requests to create a "shell" channel
    * At this point you tell the server which ID you're allocating to this
      channel
    * Since each channel has flow-controlling, you also state your window size
        * Both sides maintain a receive window, and once the sender has sent
          enough bytes to fill it, they will not send more data until you say
          that your window now has more room
7. If the server supports "shell" channels, it will tell you which ID it has
   allocated to this channel, so that you can put the ID in SSH packets in
   this channel
8. Then you tell the server your psuedo-terminal's display specifics
   (characters horizontally and vertically, as well as pixels horizontally and
   vertically)
    * For X11, you would tell the server your screen's specifics
    * If your dimensions change, you can send an update message to the server
9. Then you can send the server the values of environment variables to use
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

<!-- more -->

Note this communication only used one channel, and there can be however many
channels open over the same SSH connection because of the framing layer and
the channel identifiers. So another channel might be used for forwarding
packets received on a port to which the SSH program is listening, and that's
how we'd end up with the "-L" feature described above.

### It's like HTTP/2

I don't know many protocols, so it was fun to look into this one. I started on
the Wikipedia page, and then moved on to the IETF RFC cited by the Wikipedia
page. As I've noticed many times recently, the RFC is _easier_ to understand
than the Wikipedia page. The reason for that is probably similar to the reason
why the RFC is easier to understand than my explanation above, and is
potentially related to the fact that the RFC's authors are familiar with
explaning such concepts, and spent months working as a team to increase the
document's utter clarity so that there can be no miscommunication.

Another protocol that I have looked at is the new HTTP/2 IETF RFC. That
protocol contains a similar framing layer over TCP that allows it to multiplex
multiple channels concurrently over a single underlying TCP socket. The reason
for that was to ease issues with "head-of-line blocking" which is where one
channel's being slow ends up slowing down all the other channels because of
TCP unecessarily guaranteeing order over multiplexed channels that have no
globally-defined order. After having seen this framing-layer thing in two
protocols, now I understand it to be some sort of "pattern" for allowing
multiplexing communications over a single TCP link. An advantage to this is
decreased port usage on the server who may be handling multiple channels for
many different clients at once. Another advantage is that the TCP algorithm
has a "ramp up" time, i.e. the congestion control starts out with a rather
pessimistic window size. By using only one socket, that ramp up only must be
done once. I'm sure there are many other advantages that if I had a better
grasp of TCP would be obvious.

I don't like using tools when I really have _no clue_ what they're doing.
However doing so is inevitable because the modern stack of technologies is so
vast and the "lyfe so shorte". Since my first computer science class I was
taught to press my "I believe" button when something seemed like magic. Still,
lifting some of the veil on one of the tools I use everyday is very
satisfying, and hopefully the time taken for others to get to my (still
minimal) level of understanding is reduced by the explanations above.
