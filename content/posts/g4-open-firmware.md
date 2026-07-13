+++
title = 'Getting USB Boot to work on a Powerbook G4 - or why OpenFirmware is the literal spawn of satan himself'
date = 2026-07-13T17:15:54+01:00
draft = true
+++

> [!TIP] TL;DR
> For the A1106 specifically - using the right USB port, use the command ``probe-usb boot usb0/disk@1:,\\:tbxi`` in OpenFirmware to boot. Read further if you want to see me slowly lose my sanity trying to work this out.
# Preamble

A few months ago, I acquired a PowerBook G4 on eBay for a ridiculously good price.
![](/images/posts/g4-open-firmware/ebay_order.png)

I've adored the PowerBook G4 for *years*, and I managed to score an A1106 - it has the top-spec 1.67GHz G4 processor, but without the high-res display on the final model.

It shipped with Mac OSX 10.5 Leopard installed and fully updated - and thanks to the seller *not wiping the disk*, it still had a worrying amount of personal files on the disk. 

> As a small aside - if you're selling any device with the original storage medium, assume anybody who gets their hands on it *will* go through the drive. Always securely erase your drives!!

Now, I wanted to start afresh with a new installation of OSX 10.5 Leopard on a brand new SSD - unfortunately, the DVD drive was completely and utterly bollocksed, and I don't have the tools nor time to get it fixed. Even in 2005, you could usually get a computer booting off of a USB, so it should be a piece of cake to get the image on a USB drive, and get everything installed in the space of an hour, right? I just need to pop in the BIOS and manually boot off of it.

__*...right??*__

---

# OpenFirmware
Enter stage: OpenFirmware.
> [!NOTE] Disclaimer
> I'm not at *all* an expert on PPC-era Macs, so this will be a rather abridged summary. I've tried to be as accurate as possible, but please do shout at me if I've gotten anything wrong!

With the PowerMacs 7200, 7500, and 9500, Apple introduced the PCI bus - and alongside, a new firmware adopted from Sun Microsystems called OpenFirmware. Apple couldn't just use a BIOS; that was (until worryingly recently) designed specifically around the architecture of the *original* IBM PC, and it'd be a fool's errand to get that ported to PowerPC.

This lasted all the way up until the introduction of the Intel-based Macs - meaning the Powerbook G4 I've got here uses it!

OpenFirmware is an odd beast. It's developed in (and requires you to use) Forth(**!!**), and is completely command-line based [^1]. No buttons, or wizards, or even useful documentation built-in - it's about as obtuse as it gets. 

**IMAGE OF OPENFIRMWARE HERE**

I could go on for hours about the strange intricacies of OF. To keep this *relatively* concise, I'll focus on just the implementation on my Powerbook G4.

## The VERY little I worked out on my own.
So, what the hell can I actually do with this? The bits I worked out were:
- Eject the CD-ROM using ``eject cd``
- Shutdown the machine using ``shut-down``

...and that's all I could work out without research.

So, to Google I went!

---

# USB Booting: The first and second fruitless attempts.
## 1: StackOverflow
First of all - there seems to be no good single source for information on OpenFirmware for Macs. All the archived Apple pages are either pointlessly sparse or stupidly technical. I had to rely on forum posts as any kind of vaguely authoritative information on this.

The first post I found on Google was [This](https://apple.stackexchange.com/questions/11128/how-do-i-make-my-1-5-ghz-powerbook-g4-boot-from-a-usb-stick) StackOverflow post from 2011. This was for a 1.5GHz PowerBook G4 - really similar to mine! (this will be important later) The best answer has an easy command to enter (``boot usb1/disk@1:,\\yaboot``). ``yaboot`` is specifically for Linux images, so I replaced it with ``,\\tbxi`` for Mac OS X.

The result?: **FIND EXACT OUTPUT FROM OF FOR THIS** Trying every single USB port, it did the same.

Ah, but, the second answer says you need to specify the exact partition of the drive you've got plugged in - that makes sense! I'll amend the partition number, and try ``boot usb1/disk@1:00,\\tbxi``!

...nope. Nothing.

I gave up on this post, and looked elsewhere.

## 2: MacRumours Forum
Perfect! The MacRumours forum is a bloody excellent resource, so finding [this](https://forums.macrumors.com/threads/guide-new-method-booting-from-usb-on-powerpc-macs.2403368/) post was a godsend, right?

Ha. Hahaha. Hahahahaha. Ha.

I had to find two things - the partition ID on the drive and the USB port ID it was plugged into. The first was easy! The second?

### USB IDs in OpenFirmware
For whatever reason, even though it can boot from them, this post suggests OpenFirmware **cannot detect any USB storage devices plugged in.** To get past this, the post suggests to do three things:
- Unplug every USB device from your laptop,
- Plug in a USB keyboard,
- Do ``dev usb0`` to select the USB port, then ``ls`` to list all devices connected, increasing the USB port number until you've checked all of them.
- The USB that has a ``keyboard@X`` value but NOT a ``mouse@x`` value is your port!

So, I plugged my shittiest, most basic keyboard into the Powerbook, used it to type in the commands, and...

**Nothing.** It wouldn't detect the keyboard. The keyboard I was currently typing on.

This was frustrating as hell, but ultimately fine - I'd just try them all.

### probe-usb

This guide had a new command prefixed on the front - ``probe-usb``. I have absolutely no clue what this does, but at least it's something different!

The command I have now is ``probe-usb boot usb0/disk:3,\\:tbxi``. 

Trying this makes OpenFirmware completely freeze, and there's nothing I can do to recover it. Ballsack.

It's fine, I just have to try every USB port number until it works - it'll take a goddamn age, but it'll work eventually, right?

Nope, nothing. Absolutely nada.

# Grasping at Straws Until Something Works
At this point, I'm getting really annoyed and desparate. The first page I found was [this](https://lifedigital2010.wordpress.com/2011/04/03/how-to-install-mac-osx-from-usb-on-powerbook-g4/) ancient WordPress page on USB booting on a Powerbook G4. To start with, this guide requires you to...list USB storage devices? 

The previous post seemed to suggest this **wasn't possible**, or at least not reliable - huh??

Using ``dev / ls``, I listed every device on the laptop - aaand nothing. No disks whatsoever. It *did* have an interesting tidbit - you can specify the exact path you want to boot from. It also changes the ``disk:3`` part to be ``disk@1:3``?

Using this new-found knowledge, I constructed a new command - ``boot usb0/disk@1:3,\System\Library\CoreServices\BootX``.

...nothing. 

What about appending probe-usb? 

That just crashes it. Great.

I won't go over the half-dozen pages I went through after that in much detail, but safe to say, absolutely nothing worked. It felt like my specific PowerBook G4 was cursed, and didn't want to do anything that any of the guides thought it should. I decided to leave it for the week.

---

# Frankensteining This Shit into Doing My Bidding
I came back after a week, and decided to take stock of everything I'd done so far.

This was:
* I can't detect the USB port.
* There seem to be at least two different ways to get OpenFirmware to target your disk: using an @, and not using an @?
* ~~OpenFirmware was created by the devil specifically to punish those who can't be arsed to fix a DVD drive~~

I'd failed to find any other relevant resources, so fuck it, let's just try some shit.

First, I tried ``boot usb0/disk:3,\System\Library\CoreServices\BootX`` - removing the @. Nothing.

I tried that with probe-usb - another freeze.

Next, fuck it, I'll do ``probe-usb boot usb0/disk@1:3\\:tbxi`` - yep, froze again.

I went through a *lot* of variations with no success, so had a look at the articles I'd seen before. I looked at the StackOverflow page, and realised it *wasn't* specifying a partition ID. 

I tried this again. Nothing.

I tried it with probe-usb. It immediately booted into the goddamn MacOS 10.5 installer.

# So, what have we learned?
First of all - OpenFirmware is a godforsaken piece of shit, and is exactly what I'd expect to come out of Sun Microsystems.

After this whole fucking polava, I did a bit of digging. Different generations of PowerBook have different OpenFirmware versions. I can't confirm this, but it seems the differences between different revisions, even minor ones, is surprisingly big. Commands that run fine on one Mac will fail on another, or do something completely different. I've got a feeling all of the guides I saw would work absolutely perfectly on other devices.

Remember the StackOverflow page? That was for a 2004 1.5GHz PowerBook G4 - so similar to mine that I assumed it would be identical! Turns out, that runs OpenFirmware 4.8.6 - mine looks to run 4.9.1. I've got a feeling that command would work perfectly if I had that specific laptop.


[^1]: https://www.martinnobel.com/techresearch/the-open-firmware-wiki: The Open Firmware Wiki, Mark Nobel