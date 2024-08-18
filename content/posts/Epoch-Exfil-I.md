+++
title = 'Epoch Exfil I'
date = 2024-08-15T02:39:15-04:00
+++

`This will be a series of posts detailing my research on using NTP for exfiltration. In these upcoming entries, 
I intend to help those investigating how attackers could exploit this technique to transfer data in and out of a network, 
as well as explore methods for detecting such activities.`

---

**ToC**

- [What is the Network Time Protocol?](#WhatNTP)
- [Client implementation](#Client)

---

<a name="WhatNTP" />

### What is the Network Time Protocol?

`The Network Time Protcol Version 4 (NTPv4) is defined in` [RFC 5905](https://www.rfc-editor.org/rfc/rfc5905)

```
... the Network Time Protocol version 4 (NTPv4), which is widely used to
synchronize system clocks among a set of distributed time servers and clients.
```

`In short NTP is used to synchronize system clocks between computers.`

`When talking about NTP there are three implementations..`

- Primary server - synchronized to a reference clock directly traceable to UTC (e.g., GPS, Galileo, etc.)
- Secondary server - has one or more upstream servers and one or more downstream servers or clients.
- Client - synchronizes to one or more upstream servers, but does not provide synchronization to dependent clients.

`We will be focusing on the client implementation.`

---

<a name="Client" />

### Client implementation

#### Time format

[Section 6](https://www.rfc-editor.org/rfc/rfc5905#section-6) `provides details regarding the time values and their respective formats.`

- There are three NTP time formats, a 128-bit date format, a 64-bit timestamp format, and a 32-bit short format
- The 64-bit timestamp format is used in packet headers and other places with limited word size.
- All NTP time values are represented in twos-complement format
- Bits numbered in big-endian

---

#### Packet information

`Jumping down to` [Section 7.3](https://www.rfc-editor.org/rfc/rfc5905#section-7.3) `we find information about the packet`

- The NTP packet is a UDP datagram
    
`This tells us we need to create a UDP socket.`
`The next are the Packet Header Format and the Packet Header Values.`

---

##### Packet Header Format

```
The NTP packet header shown in Figure 8 has 12 words followed by optional
extension fields and finally an optional message authentication code
(MAC) consisting of the Key Identifier field and Message Digest field.
```

```
                      Figure 8: Packet Header Format

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |LI | VN  |Mode |    Stratum     |     Poll      |  Precision   |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Root Delay                            |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Root Dispersion                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                          Reference ID                         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                     Reference Timestamp (64)                  +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Origin Timestamp (64)                    +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Receive Timestamp (64)                   +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                      Transmit Timestamp (64)                  +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      .                                                               .
      .                    Extension Field 1 (variable)               .
      .                                                               .
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      .                                                               .
      .                    Extension Field 2 (variable)               .
      .                                                               .
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                          Key Identifier                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      |                            dgst (128)                         |
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
---

##### Packet Header Values

```
LI Leap Indicator (leap): 2-bit integer warning of an impending leap
second to be inserted or deleted in the last minute of the current
month with values defined in Figure 9.
```

```
           +-------+----------------------------------------+
           | Value | Meaning                                |
           +-------+----------------------------------------+
           | 0     | no warning                             |
           | 1     | last minute of the day has 61 seconds  |
           | 2     | last minute of the day has 59 seconds  |
           | 3     | unknown (clock unsynchronized)         |
           +-------+----------------------------------------+

                         Figure 9: Leap Indicator
```

---

```
VN Version Number (version): 3-bit integer representing the NTP
version number, currently 4.
```

---

```
Mode (mode): 3-bit integer representing the mode, with values defined in Figure 10.
```

```
                      +-------+--------------------------+
                      | Value | Meaning                  |
                      +-------+--------------------------+
                      | 0     | reserved                 |
                      | 1     | symmetric active         |
                      | 2     | symmetric passive        |
                      | 3     | client                   |
                      | 4     | server                   |
                      | 5     | broadcast                |
                      | 6     | NTP control message      |
                      | 7     | reserved for private use |
                      +-------+--------------------------+

                       Figure 10: Association Modes
```
---

`The most important part of the NTP packet is the first 8 bits (byte). It informs the server about`

- A time leap (LI)
- The NTP version being used (VN)
- Mode (in our case client)

`I will not be going into detail about the other values because we will be overwritting them to exfiltrate data.`

---

# Next post

`In the next blog post we will start writing our NTP client and server to test our exfiltration method..`
[Next](/posts/epoch-exfil-ii/)

---
