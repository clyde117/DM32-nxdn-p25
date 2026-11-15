# User guide: FM, DMR, P25, NXDN and P25 trunk channels

This firmware is experimental and **not verified on a radio** ([hil-checklist.md](hil-checklist.md)). Read this
before you install it:

- Keep a full backup of your radio, including its calibration ([recovery.md](recovery.md)).
- Transmit only where your license allows.
- Transmit on a trunked system only if its administrator authorized you and assigned your unit ID.
- Encrypted calls in any mode are muted. Nothing is ever decrypted.

## Channel modes

| Mode | Screen | What it is |
|---|---|---|
| FM | `FM` | Analog, as the stock radio |
| DMR | `DMR` | Tier II, both timeslots, as RadioFoundry |
| P25 | `P25` | P25 Phase 1 conventional: one frequency (or a repeater pair), a NAC, a talkgroup |
| NXDN | `NXDN` | NXDN48 conventional (6.25 kHz): a RAN and a group |
| P25 trunk | `P25T` | One P25 Phase 1 trunked system. The channel's frequency is the system's control channel; the radio moves to the voice channels by itself |

The radio covers VHF and UHF. 800 MHz trunked systems are out of its band.

## Setting up channels

Use the codeplug editor or a CSV file. The radio's menu also edits NAC, RAN, talkgroup and unit ID. On the NAC or RAN row, OK with nothing typed switches RX
between the channel's own NAC/RAN and every NAC/RAN, keeping the TX one.

| Field (CSV column) | P25 | NXDN | P25 trunk |
|---|---|---|---|
| RX / TX frequency | the channel (TX: the repeater input) | the channel | RX: the control channel. TX: its inbound frequency |
| RX NAC / TX NAC | 000-FFF; RX `any` (F7E or F7F) hears every NAC | - | RX NAC: the control channel's, or `any` |
| RX RAN / TX RAN | - | 0-63; RX 0 (or `any`) hears every RAN | - |
| Talkgroup | 1-65535: where PTT talks, and what plays | 1-65519 | 1-65535: affiliated to, PTT talks to it |
| Hear All Groups | play every talkgroup | play every group | follow every group call on the system |
| WACN | - | - | 5 hex digits, from the system's administrator |
| System ID | - | - | 3 hex digits, from the system's administrator |
| Unit ID | the channel's ID, else the radio's DMR ID | the same; NXDN needs 1-65519 | the same: **the ID the administrator assigned you** |

## DMR repeaters with restricted access (RAS)

A repeater with RAS (restricted access to system) alters the checks of each call's header and embedded link control
with its key, so a radio without the key never sees a valid call and plays nothing. Turn on **digital monitor** (a
short press of the monitor key, SK1): a whole voice superframe in order on the channel's color code and timeslot then
plays as an unknown group call (talkgroup and ID 0), until a header or embedded LC that decodes names it. Calls marked
private (encrypted) still do not play. Without digital monitor the talkgroup can't be told, so nothing plays. Transmit
through such a repeater needs the system's key, which this firmware doesn't have.

## Air band channels from the stock codeplug

The installer's conversion of the original codeplug now keeps air band (and broadcast FM) channels, also those
without a TX frequency: they come over as receive only, FM (played in AM in the air band).

## Writing a DMR message

On a DMR channel, open **Contacts**, focus the person or talkgroup, press **#**, then **3 (Message)**.
Choose **Write...**. Keys **2-9** cycle letters; pause a second between letters on the same key. **0** enters
a space, **1** punctuation, and holding a digit enters that digit. **#** cycles uppercase, numbers, and lowercase.
**Back** deletes; on an empty message it returns to Compose. Hold **Back** to cancel and go Home.

The title shows the input mode and character count, up to **140 characters**. **OK** opens the review, with the
recipient in the title. **Up/Down** scroll the complete text, **Back** returns to editing, and **OK** sends.
Entering the review does not transmit. A refused send keeps the text available for another try; sent text is not
saved across power cycles.

The existing message-format setting chooses ETSI IP text, ETSI short data, or Motorola TMS. The channel supplies
the frequencies, timeslot, and color code; a recipient ID alone is not enough. A private message requests delivery
confirmation where supported; group messages have no confirmation. The new editor has passed host tests and the
DM-32 build, but sending typed text has not been checked on the physical radio.

## Using a P25 or NXDN channel

The screen shows:
- The PTT line with the talkgroup.
- The NAC or RAN.
- The caller's talkgroup and unit ID during a call.

Calls on another talkgroup, NAC or RAN, or encrypted calls, show but do not play.

**Digital monitor**, as on DMR: a short press of the monitor key (SK1) plays every talkgroup and every NAC (P25) or
RAN (NXDN) on this channel; the status row shows `DMon` and a toast confirms it. Another press, Back or a change of
channel ends it. PTT still talks with the channel's own NAC/RAN and talkgroup, and the channel is not changed. On a P25
trunk channel it plays every group of the system.

On P25, the speaker is muted between calls, including after power-on and after releasing PTT. A continuous hiss
after the hotspot has returned to standby is not an expected transmit tail.

PTT sends a call to the channel's talkgroup with your unit ID. The time-out timer applies. Busy channel lockout
refuses PTT while another NAC (RAN) is heard, or any traffic, as the setting says.

## Using a P25 trunk channel

| PTT line | Meaning |
|---|---|
| Searching for the site | No control channel with the channel's WACN and System ID is heard yet |
| Listen only | On the system; this firmware does not transmit on trunked systems (the default build) |
| Not registered | On the system; transmit is built in, but the system has not registered the radio (yet, or it refused) |
| TG n | Registered: PTT asks for a channel to talk to talkgroup n |
| Requesting a channel | PTT is down and the request is out |
| Queued by the system | The system will give a channel when one is free (at most 10 s) |

Group calls on the channel's talkgroup play, and so do all calls with Hear All Groups. The radio goes to the voice
channel and comes back to the control channel after the call.

PTT on a trunk channel (only with a firmware built with trunked transmit) works as follows:
- Hold PTT. The radio asks the system for a channel and talks when the system grants it.
- If the system denies the call, or doesn't answer, the screen shows the refusal until you release PTT.
- The radio sends nothing without PTT, except its registration and its affiliation to the talkgroup. It sends each
  at most every 30 s, and only to the system the channel names.

Limits:

| What | Behavior |
|---|---|
| Phase 2 (TDMA) calls | Not heard: the radio cannot receive or send Phase 2 |
| Control channels | One per channel. For another site, use another channel |
| Call types | No emergency, private calls, data or supplementary services |
| Leaving the channel | De-registration is not sent; the system ages the registration out |
