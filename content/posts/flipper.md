---
title: "What I Use My Flipper Zero For"
date: 2026-06-22T09:12:10-06:00
draft: false
tags: ['flipper, infosec, hardware']
---

I think it can be safely said that the Flipper Zero is one of the most iconic hacker devices of this decade. Filled to the brim with features, and yet tragically over-hyped, I think some people came away from the project feeling... disappointed? I can't relate. Even though I'm not using using my Flipper to unlock cars ([yet](https://forum.dangerousthings.com/t/car-unlock-and-ignition-advice/5624/45)), here are some things I _am_ using it for, and some ideas that I'd like to explore.

# Capabilities
At a base level, the Flipper Zero boasts interactions with several mediums/protocols. [The docs](https://docs.flipper.net/zero) provide a good overview, the big ones are:
- NFC / RFID reader and emulator
- Sub-Ghz transceiver
- Infrared transceiver
- iButton reader and emulator
- GPIO connectors
- USB-C

Flipper Apps (...faps *puke*) can be written to interact with all of these interfaces. The result is not just an out-of-the-box multi-tool that can interact and emulate various protocols, but a platform for which the community can write and expand the functionality of the device. While most of my use-cases utilize stock features, there are dozens of community apps, alternative firmwares, 3rd party hardware extensions that allow for some exciting new capabilities.

# Use Cases
Anyway, here's what I actually use my Flipper for

## USB Serial Adapter
That's right, probably the most exciting feature ever (/s). Using the Flipper's GPIO and USB-C connectors, the device can be used to connect to serial interfaces. UART is the one I end up talking over the most. Various hardware devices expose UART interfaces for debugging purposes, and many manufactures don't disable it when shipping a product. If you can find UART on a devices, there's a 50% chance you get an instant root shell.

To connect, I use jumper wires to connect the Rx of the target device to the Tx of the Flipper, then the Tx of the device to the Rx of the Flipper. I start the app (GPIO -> USB-UART Bridge), and plug the Flipper into my PC over USB-C. To connect, I use the following command:


`screen /dev/ttyACM0 115200`

note: you may need to adjust your [dialout permissions](https://askubuntu.com/questions/133235/how-do-i-allow-non-root-access-to-ttyusb0) to get the connection to work.

![](/images/Flipper-NAS.jpg "Connecting to my WD PR4100 NAS over an exposed UART connection")

UART isn't, like, a secret or anything. You can purchase USB to serial connectors [online](https://www.amazon.com/Serial-Adapter-Female-FT232RL-Windows/dp/B07RBK2P47), or use other hardware devices like the [Tigard](https://github.com/tigard-tools/tigard). A reoccurring theme with the Flipper is that it doesn't introduce anything new; other devices have already been made for each of these use-cases. But having a Flipper means that I don't have to buy a dedicated cable. In the case of UART, it even has a handy interface that that shows the exchange of bytes over the wire, which I find handy for debugging.

## NFC Cloning
NFC is probably one of the more popular uses of the Flipper. Within my own life the ability to read and emulate a variety of tags has come in super handy.

When I lived in Rochester, NY, my girlfriend's apartment had 'upgraded' their access control system from a proprietary iButton (which I _wasn't_ able to emulate with a Flipper) to an NFC system using Mifare Classic cards. This was my first hands-on introduction to the world of NFC protocols, attacks, and the how convenient the Flipper is for cloning and sharing Mifare cards.

The TLDR of [the attack](https://github.com/aalex954/cloning-MIFARE-classic-1k) is that the keys used to encrypt the various data sectors of Mifare Classic cards can be guessed using a dictionary-style attack, or cryptographic nonces can collected from a reader and used to derive the encryption keys. With keys in hand, and temporary access to a target NFC card, an attacker can read the sector data used by the door lock. The Flipper Zero can be used to store various key lists, capture nonces, [calculate encryption keys](github.com/equipter/mfkey32v2), and emulate stolen card data. In practice this let me clone my girlfriend's apartment key (consensually) so that I could sleep in during her morning shifts, or leave her surprise flowers when she was away.

When we moved to Colorado, our new apartment complex also used [Mifare Classic fobs](https://securiotec.com/product/schlage-9691t-multi-technology-fob-credential/) for the apartment [deadbolts](https://securiotec.com/product/schlage-control-smart-deadbolt-lock-satin-nickel-greenwich-be467-grw-619/). Looking past the glaring [physical vulnerabilities](https://www.youtube.com/watch?v=mGR3h6KTntc) of this deadbolt, it was another opportunity for me to use the Flipper for creating spare keys. It also prompted me to get an xM1 implant (gen 2). I can write to this chip via the Flipper, or any Android phone, and it lets me emulate a perfect clone of apartment key. My physical fob is now a spare key that guests can use.

Mifare Classic cards also pop up in a lot of hotels. These are slowly being phased out in favor of Mifare Ultralight C tags which are not as easily cloneable, but it's still fun to write a card to my hand during a trip.

I should probably mention NFC cards that aren't just Mifare Classic. HID iClass pops up occasionally at work. The Flipper has a [PicoPass](https://gitlab.com/bettse/picopass) app that can be used to read and clone iClass legacy cards. I think there are options for iClass SE and Seos cards, but I honestly forget. HID had a big security moment a decade ago when their [iClass master keys got leaked](https://blog.kchung.co/reverse-engineering-hid-iclass-master-keys/).

![](/images/Flipper-iClass.jpg "Reading an iClass card with the PicoPass app")

ONE LAST NOTE. Some systems don't actually care about NFC card data. Some will only look at the UID of a given card. There was a silly instance at a friend's office where I presented a random Mifare card to a badge-enabled vending machine. "Welcome Evan Schwartz", said the machine. The odds of a colliding UID feel so low, be we actually got a couple of collisions just from Mifare cards that I already had saved on the Flipper. While amused, my friend did not let me vend anything on Mr. Schwartz' behalf...


## RFID Cloning
Similar but different, there have been some good RFID use-cases for the Flipper too. A perk of RFID tags is that they contain much less data and typically have no security features; you can clone and emulate practically any 125khz tag you find.

Savvy readers may have viewed the product page for the Schlage deadbolt I talked about under earlier and noted that the corresponding tags are "Multi-Technology". It was a surprise to me too when my cloned Mifare tag couldn't unlock my apartment's gym. Turns out Mifare is used for the apartment deadbolts and HID Prox is used for everything else (exterior doors, gym, bike room, etc.). TLDR I got a second [implant](https://dangerousthings.com/product/xem/) and cloned the RFID portion of the tag to it with my Flipper. Now i have access to the rest of my apartment. This same tag can be re-written to access my work's office, the [denhack hackerspace](https://denhac.org), and the occasional client visitor badge.

In case you don't want to inject yourself, there are plenty of [normal writable RFID tags](https://www.amazon.com/Writable-Proximity-Rewritable-Universal-Security/dp/B0F4MNBFRQ/141-2643660-1530346) that you can buy. These tags use the T5577 chip which is able to emulate 99% of RFID card types. They come in tag form, sticker form, probably some other forms idk. My dad bought a couple of these and I used my Flipper to clone his apartment fob to them; now he has a spare key.

One thing I don't see talked about too often is the bruteforcing of RFID badges. This is something I've been doing for work as a part of on-site penetration tests. The idea is simple: A company using RFID badges will typically order a large pack of badges to start. These badges are enrolled into the access control system, mapping the badge number to the digital identity and access rights of an employee. The issue is that the badges are likely in sequential order. With knowledge of one badge number, the hexadecimal value can be incremented or decremented to find other valid badge numbers. This is useful in situations where you have a visitor badge (low privileged) but are trying to gain access to a server room (high privilege); simple increment your badge number until the reader lets you in. We've been able to do this on a number of client sites using [built-in apps](https://lab.flipper.net/apps/fuzzer_rfid) on the Flipper.


# Sub-Ghz
I'll be honest, I haven't done much with the sub-ghz module. I haven't broken into any cars, I haven't hacked any garage doors, I haven't even opened a [Tesla charging port](https://www.youtube.com/shorts/zomxhCqvBFg).
The one thing I've done is cloned the Hunter ceiling fan in my old apartment. By using the FLipper's frequency analysis module I could record and replay the radio signals sent my the fans remote to turn on the fan and toggle the lights.
Exciting stuff, I know...

# iButton
I bet you haven't seen a SINGLE click-bait TikTok showing the Flipper's iButton functionality. But maybe that speaks more to iButton then it does the Flipper. Maybe it's more common in European areas, but I've seen it used only twice in the USA.

My old college org, CSH, used to use them for access control to rooms and services. I don't have much to say really, the tags have static values which a Flipper can and read and emulate flawlessly.

# TODO
I've got a laundry list of things that are on my Flipper Zero bucket list. This blog post is already getting pretty long, so I'll try to make this quick:

## NFC Downgrade Attacks
This attack was brought to my attention by a [DEFCON 28 talk given by Babak and Iceman](https://www.youtube.com/watch?v=ghiHXK4GEzE), both titans in the RF access control space. I remember their explanation being a little complex, but the attack is simple once you understand it. It takes advantage of badge readers in "transition mode", a mode which allows readers to accept both highly secure badge protocols (DESFIRE, Seos) as well as legacy protocols (HIDProx, etc.). In their demo, the duo weaponizes an HID reader to read a secure credential, capture its corresponding wiegand data, and write it to an insecure credential. Because the target reader is in a transition mode, accepts both technologies, and the both cards evaluate to the same wiegand data, both the secure credential and the cloned, insecure, credential are evaluated the same.

This is a really cool attack. I've got a bad habit of using the [HID management app](https://play.google.com/store/apps/details?id=com.hidglobal.pacs.readermanager&hl=en) to look at the supported protocols of readers I find in the wild. 80% of these readers support both low and high frequency cards, implying that they are susceptible to this attack.

How does the Flipper Zero come in? You can buy [modules](https://www.redteamtools.com/sam-adams-for-flipper-zero/) that contain the secure element (SAM) for HID readers. This let's you use a Flipper to read the data from DEFIRE and Seos cards. [Apps](https://github.com/bettse/seader) can them be used to emulate that data in the form of less secure cards. At the moment there isn't a lot of DESFIRE and Seos in my life, but I'm the proud owner of a SAMadams and I can't wait for the opportunity to use it.
 
## Ulta-High Frequency
The Flipper Zero also has [3rd party modules](https://www.redteamtools.com/flippermeister/) that let it interact with Ulta-High Frequency (UHF) cards. These are used by the garage door opener at my apartment complex. I don't think you can emulate with the device, but I think you can read and write, and it also has SAM support. My excuse for not doing anything yet is that their aren't many use-cases in my life, but this would be a fun one for malicious access to a building or complex via a cloned garage sticker.

## BadUSB
The BadUSB app let's the Flipper emulate a Human Interface Device (HID) in order to interact with devices as if it were a keyboard or mouse. This opens up some pretty cool avenues for kiosk breakouts (when only a USB port is accessible) or for opportunistic initial access scenarios where you could plug into an unlocked PC, run a quick script, and get a beacon callback. I haven't set up any that. One feature that would be really cool, which I don't think will be reasonable until the [Flipper One](https://blog.flipper.net/flipper-one-we-need-your-help/) comes out, is a Flipper variant of Quick Creds. This is a [Bash Bunny payload](https://github.com/hak5/bashbunny-payloads/blob/master/payloads/library/credentials/QuickCreds/readme.md) that emulates a network adapter and runs responder against the target to coerce NTLM credentials from the host. The Flipper isn't really set up with a networking stack, but maybe this could be a good vibe-coding opportunity.

## Key Cloning
You won't find this one on TikTok. There is a (...)[fap](https://github.com/zinongli/KeyCopier) that allows a user to decode known key formats in the field. Simply hold up a key to the screen and adjust the lines to fir the biting. You could then take this biting to a key cutting machine to create a duplicate, or use templates to model and 3D print a copy. I honestly don't have that many keys in my life, but when the time comes I'll be ready. Denhac even has a key cutting machine.

## Combination Lock Cracking
[Another one-off app](https://github.com/zinongli/KeyCopier). This one isn't using the Flipper for anything other than some quick maths. Masterlock combination locks have a known vulnerability (made popular my Samy Kamkar, my hero) that allows a combination to be derived in 8 attempts or less. I'll do this one day.

## Sub-Ghz Bruteforce
This one is really cool and I don't see a lot of people talking about it. There are [apps](https://github.com/tobiabocchi/flipperzero-bruteforce) on the alternative firmwares that can bruteforce certain sub-ghz protocols. These protocols are often used for physical access gates. One cool application is wireless handicap door buttons. The codes for these are typically 4 DIP switches, which is easily bruteforceable in a couple of minutes, potentially allowing access through an otherwise locked door.

## Passport Reader
If you didn't know, your passport has an NFC tag in it. With knowledge of the passport number, the rest of the passport data can be read. [This Flipper app can do that](https://github.com/bettse/passy). I'm just to scared to put in my passport number.

## Sentry Safe Bypass
Awhile back someone found the backdoor code that opens all Sentry-branded safes. [This Flipper app](https://github.com/H4ckd4ddy/flipperzero-sentry-safe-plugin) lets you open up Sentry safes Hollywood-Style by hooking up some serial cables and hitting "unlock". How cool is that?!
