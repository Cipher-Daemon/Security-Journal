# Discord Café :coffee: :sandwich:
## The Logs that keep on tripping!

All started at around 11AM local time that my firewall found malicious traffic triggering the signature `ET CINS Active Threat Intelligence Poor Reputation IP group 155`. From what i remember this happens at random from time to time but nothing came of it. That is until it happened again at lest 1-2 hours after the first instance and this got me a bit worried. 

## Who, What and Where?

I checked my firewall logs and it shows my computer being both source and destination of the IDS/IPS triggering but in the source saw that its from `104[.]29[.]137[.]220`. When doing an IP lookup only info that i can find is that its a cloudflare IP, most possibly that its a CDN.

How did the request happen? I tried to look in my PiHole DNS logs to see if i can get any info but there was never a name resolution to that specific IP or even in the `104[.]29[.]0[.]0/16` range when reading and grep'ing the logs from `/var/pihole/logs`

Next option on my mac was to continuously run the `netstat` and `nettop` command to only filter UDP traffic and redirect the output either whenever it refreshes for nettop or every 2 seconds for netstat to their respected txt file to look at later, what made it really suspicious was that it was using a random port for both source and destination.

## More Logs

When checking my firewall logs i notice that there was roughly 10 concurrent logs that have happened in the last second that are completely random in the 104[.]29[.]0[.]0/16 space with the packet size being 36 bytes for each one. I suspect that this is a C2 type of beaconing traffic. Not only that but when looking at previous threats it was also in that same IP space.

What made it also suspicious was when it did triggered again i try to grep the 2 txt files that were filtering and dumping UDP sessions and nothing came back with the example command `cat UDP.txt|grep -F "104.29"`. This made me feel uneasy suspecting that i may have a rootkit!

## Investigation

When browsing ipabuse database for `104[.]29[.]137[.]220` there was at least 2 submitted the day of when this happened. one report showed that it was brute-force and another for hacking. Looking at this was not good, this made me really suspicious that either something got in or some zero day that my mac got exposed of.

After work i try to keep a close eye on if my IDS/IPS would trigger again, for the next 4.5 hours nothing has happened which made time for me to setup a network sniffer. I got my windows laptop, updated wireshark, set my switch port to be mirrored from the uplink port, got an ethernet cable to sniff traffic and got things started.

In wireshark i set a capture filter of `net 104[.]29[.]0[.]0/16` because for one, it doesn't need to see EVERYTHING that passes through my uplink and the other dont want to go through a needle in a hay stack situation.

Double check my filters once more and made sure my ethernet adapter was set to promiscuous mode. for the first 20-30 minutes nothing was happening suspecting that either it wasn't working or wasn't setup right. Decided to use my mac like normal and next thing that happened was it gathered 50 packets going to the address space that i setup.

What was interesting is not the random source/destination IP/ports but the actual data that was sent. When checking the packets it was only sending 8 bytes of data, the data packets in HEX were `1337cafe06000000` & `1337f00d03000000`

"Leet food and Leet cafe?" I asked myself, i checked google on the exact hex data and nothing showed up, i try to look at the IPs again from the most recent IDS/IPS warning to see if anything useful came up, Lo and behold I got some hint on what it is, it was a link to a GitHub issue post stating that there was something wrong with voice chat over UDP to the 104[.]29[.]0[.]0/16 address space.

This got me thinking, I decided to do a test call to a friend of mine and when I immediately called the wireshark capture was flooding in, this confirms that the CDN IP space from cloudflare was simple for discord all along!

## Conclusion

Thankfully this whole thing was benign. It gave me ease of mind that it was discord doing some sort of beaconing, probably for checking new messages, call request or whatever it needed to do in the background.

What made this interesting is that this happens on the desktop client of discord, never seen this happen on other platforms like the iOS or iPadOS version of discord.
