---
description: 'chall author: greyroad__'
---

# \[umdctf2026] osint/hades-group

OSINT is one of my favorite categories in CTF. They usually require a sort of fortitude to stomach with the initial ambiguity, but once you find a lead, the rush is unparalleled in other categories. This chall manages to keep the initial ambiguity limited while giving the same rush to player. This is why i have decided to do a writeup on it.

## Overview

The chall comes with a cool background story. ( warning: it is lengthy, tldr at bottom of this section)

> Case brief On February 3, 2026, a confidential tip reached your desk. "Hades Market", a pseudonymous prediction-market venue listing contracts on real-world outcomes involving journalists, streamers, and political activists, had been traced to a private Telegram channel operating as "Hades Group."&#x20;
>
> The market was the front: traders took positions on whether specific individuals would be doxxed, swatted, or exposed by specific dates. The Telegram group was the back: a coordination room where the operator brokered the services that settled those contracts (doxxed profiles, coordinated swatting planning, brokered access to private records), so that their own book cleared in their favor.
>
> A source obtained a **full export of the group's message history** before being removed. The export covers roughly six weeks of activity across about 25 accounts, a mix of regular participants, one-time visitors, and apparent administrators. It is attached as hade&#x73;_&#x65;xport.json. You have been engaged as the independent OSINT investigator._
>
> **Your objective** Identify the group owner: the individual operating the service, the one clearing the spread on every contract Hades Market settled. The owner never posts under a personal account; all their messages appear as anonymous group posts (fro&#x6D;_&#x69;d beginning with "channel"). They are not perfectly silent, however._
>
> Flag Submit UMDCTF{REC-XXXXXXX}, where REC-XXXXXXX is the Document Record ID returned by the final document bot at the end of the owner's investigative chain.

Beside the chat history file, we are also provided the following Telegram bots:

> * `https://t.me/QuickOSINTSearch_XGBXL_389YBot`: Underground leak database A. Query by username, UID, or phone number.
> * `https://t.me/EyeOfTheGod_ZF231_389YBot`: Underground leak database B. Different source, different coverage. Query by username, UID, or phone number.
> * `https://t.me/SherlockTweaked_9VEZB_389YBot`: Username intelligence. Returns username history and prior lookup activity.
> * `https://t.me/TGObserver_H6J3S_389YBot`: Cached Telegram profile scrapes from a large parsing database. Returns phone numbers associated with a given username.
> * `https://t.me/StickerSleuth_VZBHY_389YBot`: Sticker pack intelligence. Query by sticker set name.
> * `https://t.me/RussiaSearch_D4S38_389YBot`: Russian document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/ChinaSearch_44U32_389YBot`: Chinese document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/USDocs_6C582_389YBot` : US document database. Returns identity records. Query by full name or phone number.
> * `https://t.me/CountrySearch_U6B8V_389YBot`: Generic document database. Accepts full name and country name as arguments. Has partial overlap with the three regional bots but is the only source for certain records.

As you can see, the author has a lot of efforts preparing for this chall.

TLDR: We are given a chat history file of a Telegram channel, our goal is to find the real idendity of the channel owner. Databases are provided to aid us.

## Chat History

The first thing we did was examining the chat history json in a raw text, looking for anything out of order. Most of the time, we won't find anything but it is also effortless. So why not do it.

Afterward, we made a frontend to parse the log and make them into a more readable format so we can actually read the chat log. Features such as color coding the names and filtering by types of messages make reading experience much more pleasant.

In total, there are only 400 messages so it doesnt take long to go through them. The contents are what you would expect from a doxxing channel. Dealers selling their database, buyers making requests. From these context, we are able to deduce a few users who could potentially be the channel owner.

## Suspects

The first suspect we have is `hermes_locker` or "mason h". He has the most messages in the channel and seems to be one of the major dealers. There are many instances of him interacting with buyers.  Through his username, we found his alias, phone number and eventually his REC-ID via our telegram bots. We tried using it as our flag, however, it was rejected. I guess that would be too obvious.

The second suspect we have is `apollo_trace` or "dani". Similar to `hermes_locker` , he is one of the most active dealers in the channel. We couldn't find anything from his username directly, however, we are able to find his phone number and REC-ID via his alias `phoebus_cache` . We tried it as flag, and it was again, rejected.

This process went on with a few other users who we suspect are dealers. All of them lead to deadends. We are stuck.

## Breakthrough

After going through the usernames in the chat and being stuck. We took a step back and regroup our thoughts. These usernames are most likely red herrings, the key is somewhere else.

We reread the challenge description and noticed that we never used one of the bots – Sticker Search. However, we don't remember seeing any stickers when we were going through the chat log. So we updated our frontend to filter for any messages that contain stickers. It turns out the sticker message was previously grouped into the channel messages, and we overlooked it.

In the sticker message, it contains a sticker from `styx_reaction_pack` , and via Sticker Search, we are able to retrieve the Creator UID. Through the Creator UID, we are able to find a record with more than a dozen of aliases. Very suspicious. Very promising.

The alias that helped us find the channel owner is `zeus_archive` . When we were looking at its records, it showed another user `thanatos_signal` also searched for his record. And we searched `thanatos_signal` , we found his phone number `+49 160 5550 7318` . Using his phone number, we found his name `Niklas Hofmann` . And using his name, we found his REC-ID. And the flag.

> UMDCTF{REC-9305174}

## Takeaway

* CTF challs are usually pretty precise. If a specific is stated in the description (Sticker Search in this case) and it is unused, think hard to see if you are missing anything. We see similar pattern in other challs such as maple-signals.
* This is what a good OSINT chall is like. The player's time is respected and the scope is bounded. There are no aimless googling in the near-infinite internet. Much respects to the author for the efforts.
