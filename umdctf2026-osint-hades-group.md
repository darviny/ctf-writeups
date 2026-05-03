---
description: 'chall author: greyroad__'
---

# \[umdctf2026] osint/hades-group

OSINT is one of my favorite CTF categories. it usually requires a certain fortitude to sit with the initial ambiguity, but once you find a lead, the rush is unparalleled compared to other categories. This chall manages to keep the initial ambiguity limited while still giving players the same rush. This is why i have decided to write a writeup for it.

## Overview

The chall comes with a cool background story. Warning: it is lengthy, TLDR at the bottom of this section

> Case brief&#x20;
>
> On February 3, 2026, a confidential tip reached your desk. "Hades Market", a pseudonymous prediction-market venue listing contracts on real-world outcomes involving journalists, streamers, and political activists, had been traced to a private Telegram channel operating as "Hades Group."&#x20;
>
> The market was the front: traders took positions on whether specific individuals would be doxxed, swatted, or exposed by specific dates. The Telegram group was the back: a coordination room where the operator brokered the services that settled those contracts (doxxed profiles, coordinated swatting planning, brokered access to private records), so that their own book cleared in their favor.
>
> A source obtained a **full export of the group's message history** before being removed. The export covers roughly six weeks of activity across about 25 accounts, a mix of regular participants, one-time visitors, and apparent administrators. It is attached as hade&#x73;_&#x65;xport.json. You have been engaged as the independent OSINT investigator._
>
> **Your objective** Identify the group owner: the individual operating the service, the one clearing the spread on every contract Hades Market settled. The owner never posts under a personal account; all their messages appear as anonymous group posts (fro&#x6D;_&#x69;d beginning with "channel"). They are not perfectly silent, however._
>
> Flag Submit UMDCTF{REC-XXXXXXX}, where REC-XXXXXXX is the Document Record ID returned by the final document bot at the end of the owner's investigative chain.

Beside the chat history file, we are also provided with the following Telegram bots:

> * `https://t.me/QuickOSINTSearch_XGBXL_389YBot`: Underground leak database A. Query by username, UID, or phone number.
> * `https://t.me/EyeOfTheGod_ZF231_389YBot`: Underground leak database B. Different source, different coverage. Query by username, UID, or phone number.
> * `https://t.me/SherlockTweaked_9VEZB_389YBot`: Username intelligence. Returns username history and prior lookup activity.
> * `https://t.me/TGObserver_H6J3S_389YBot`: Cached Telegram profile scrapes from a large parsing database. Returns phone numbers associated with a given username.
> * `https://t.me/StickerSleuth_VZBHY_389YBot`: Sticker pack intelligence. Query by sticker set name.
> * `https://t.me/RussiaSearch_D4S38_389YBot`: Russian document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/ChinaSearch_44U32_389YBot`: Chinese document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/USDocs_6C582_389YBot` : US document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/CountrySearch_U6B8V_389YBot`: Generic document database. Accepts full name and country name as arguments. Has partial overlap with the three regional bots but is the only source for certain records.

As you can see, the author put a lot of efforts into preparing for this chall.

**TLDR:** We are given a chat history file from a Telegram channel, and our goal is to find the real idendity of the channel owner. Several databases via Telegram bots are provided to aid us.

## Chat History

The first thing we did was examine the chat history json file, looking for anything out of orindary. Most of the time, this doesn't reveal anything, but it is  effortless, so why not do it.

<figure><img src=".gitbook/assets/Screenshot 2026-05-03 at 11.55.50 AM.png" alt=""><figcaption><p>chat history json file</p></figcaption></figure>

Afterward, we made a frontend to parse the log and present it in a more readable format so we could actually read through the chat history. Features such as color coding the names and filtering by type made the reading experience much more pleasant.

In total, there are only 400 messages so it did not take long to go through them. The contents were what you would expect from a doxxing channel. Dealers selling access to databases, buyers making requests. From this context, we are able to identify a few users who could potentially be the channel owner.

<figure><img src=".gitbook/assets/Screenshot 2026-05-03 at 12.04.33 PM.png" alt=""><figcaption><p>frontend of chat history</p></figcaption></figure>

## Suspects

The first suspect we have was `hermes_locker` or "mason h". He had the most messages in the channel and seemed to be one of the major dealers. There are many instances of him interacting with buyers.

Through his username, we found his alias, phone number and eventually his REC-ID via our telegram bots. We tried using it as our flag, however, it was rejected. I guess that would be too obvious.

<figure><img src=".gitbook/assets/Screenshot 2026-05-03 at 1.43.53 PM (1).png" alt=""><figcaption><p>Using one the Telegram bots, we found info about hermes_locker</p></figcaption></figure>

The second suspect we have was `apollo_trace` or "dani". Similar to `hermes_locker` , he was one of the most active dealers in the channel. We could not find anything from his username directly, but  we were able to find his phone number and REC-ID via his alias `phoebus_cache` . We tried it as the flag, and it was rejected again.

This process went on with a few other users we suspected of being  dealers. All of them lead to dead ends. We were stuck.

## Breakthrough

After going through the usernames in the chat and getting stuck. We took a step back and regrouped our thoughts. These usernames were most likely red herrings, the key is somewhere else.

We reread the challenge description and noticed that we never used one of the bots: **Sticker Search**. However, we did not remember seeing any stickers when we were going through the chat log. So we updated our frontend to filter for any messages that contain stickers. It turns out the sticker message was previously grouped into the channel messages, and we overlooked it.

<figure><img src=".gitbook/assets/Screenshot 2026-05-03 at 2.10.12 PM.png" alt=""><figcaption></figcaption></figure>

In the sticker message, it contains a sticker from `styx_reaction_pack` , and via Sticker Search, we are able to retrieve the Creator UID. Through the Creator UID, we are able to find a record with more than a dozen of aliases. Very suspicious. Very promising.

<figure><img src=".gitbook/assets/Screenshot 2026-05-03 at 2.13.39 PM.png" alt=""><figcaption></figcaption></figure>

The alias that helped us find the channel owner is `zeus_archive` . When we were looking at its records, it showed another user `thanatos_signal` also searched for his record. And we searched `thanatos_signal` , we found his phone number `+49 160 5550 7318` . Using his phone number, we found his name `Niklas Hofmann` . And using his name, we found his REC-ID. And the flag.

> UMDCTF{REC-9305174}

## Takeaway

* CTF challs are usually pretty precise. If a specific is stated in the description (Sticker Search in this case) and it is unused, think hard to see if you are missing anything. We see similar pattern in other challs such as maple-signals.
* This is what a good OSINT chall is like. The player's time is respected and the scope is bounded. There are no aimless googling in the near-infinite internet. Much respects to the author for the efforts.
