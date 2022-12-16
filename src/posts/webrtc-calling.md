---
title: The multiple faces of WebRTC N-peer calling; Mesh, MCU and SFU
description: Building robust and reliable video calling software. 
date: 2022-09-26
author: Christos Alexiou
tags:
  - webrtc
  - networking
---
It was approximately 458BC when Aeschylus wrote the play _Agamemnon_. The Greek commander has just returned from the long war in Troy, only to be murdered by his wife Clytemnestra. 

The play starts and ends in tragedy, but that's what it is after all. Don't fret though, reader, because the chorus suggests that nothing goes unpunished and that the gods may send Orestes, Agamemnon's son, to restore justice.

Aeschylus will aid the gods with the completion of the scheme `Hubris, Ates, Nemesis, and eventually Tisis` with the help of his masterful writing. While telling the story, he goes into the trouble of describing to the audience one of the ingenious ways the Greeks used to move information across really large geographical areas; _fryctories_

The system of _fryctories_ or άγγαρον πυρ (meaning, _the telling fire_) consisted of Greek soldiers lighting fires on top of mountains and hills across a vast geographical region to transport a message. Unfortunately, the system was binary, so only a certain pre-agreed message could be communicated.

One might say, it was one of the first peer-to-peer systems that relied on technology to transfer information that was used for communication purposes.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lhjnjpinxhsadxw5reur.jpg)

## WebRTC

The need for communication, and especially peer-to-peer, didn't end with the Trojan war. Eventually, people would have to develop solutions that wouldn't rely on fire, weather conditions, and the terrain of a region. With these prerequisites in mind (or not) Justin Uberti and Peter Thatcher wrote a little piece of software that is called WebRTC.

What it does, essentially, is allow direct peer-to-peer communication between parties, particularly for video and audio.

The collection of codecs, encoders, and processing software is usually bundled in APIs that become available to programming languages through browsers or third-party libraries.

As adoption grew for WebRTC (for example, all major video conferencing software in the world takes advantage of it - Zoom, Slack, etc) base level peer-to-peer communication became a bottleneck for developing features that would enable more complex software features to be written. So, the WebRTC community found alternative ways of utilizing it by setting up the topology differently.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9hru2gb221bf8wc3ynvy.png)

## Mesh

Supposing a multiparty where you want to have a group call between the `n` members. We will abstract the calculations in the end, but for now, let's assume that `n = 5`.

The 5 participants of the group are trying to communicate with each other, sharing audio and video. The first and most naive approach to enable this communication is to use a mesh network.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/prpcb2ngkufkpmvz5r85.png)

In this topology, all participants are directly connected with peer-to-peer connections. So, in this case, we have 5 participants with each one having 4 connections to the rest with a total number of 10 connections. We assume that each connection that will be presented in this article, either upstream or downstream, will be a 1mbps connection. 

The total upstream in such a case will be the multiplication of 1 connection with the number of upstream connections we have. So, that would be `4 x 1 Mbps = 4mbps total upstream.`

The downstream, in a similar fashion, will be `4 x 1 Mbps = 4mbps total downstream`. The total bandwidth requirements are then `5 participants x 4 Mbps = 20mbps.`

Taking a step back we can see that:

`total bandwidth required = number of participants x number connections x connection bandwidth`

or 

`TBR = n * (n-1) * abr` where `abr` is average bandwidth requirement

Assuming an average connection bandwidth of 1mbps we can see that for 15 people (which is the average number of students in a school class in the country where the author resides) a typical COVID19-era class would need:

`TBR = 15 * 14 * 1 = 210 Mbps`

Since the user is picking the entire tab this is extremely impractical.

| Participants | Connections (Total) | Downstream | Upstream | Total  |
|--------------|---------------------|------------|----------|--------|
| 5            | 4 (10)              | 4mbps      | 4mbps    | 20mbps |

## MCU

Assuming the same scenario as above, with a 5-member multiparty group call we can improve upon the naivety of the mesh topology by introducing a centralized composition system, or as engineers usually refer to them; media servers.

One might refer to this topology as "the star" approach since the media server sits "in the middle" composing the upstream videos of each participant into a single stream that then sends to the participants who receive it downstream.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ur98ql15g6c19pd8kb62.png)

This topology requires extra CPU as the media server controlled by the provider needs to provide a composite video to each of the participants. At least, it seems like this approach has a 1-1 match as each one of the members sends 1 upstream and receives 1 downstream in return.

This means that each user is sending 1 Mbps of data while also receiving an equivalent amount back.

In this case for the user of this topology:

`total bandwidth required = n x (average bandwidth requirement x 1 + average bandwidth requirement x 1)`

or 

`TBR = n * (2 * abr)`

Supposing the 15 people classroom example, a call between them would require:

`TBR = 15 * (2 * 1) = 30 Mbps`

The only concern is now CPU time, but that is something that we can mitigate either by scaling horizontally or vertically, provided that we can run the media servers in a distributed fashion.

MCU seems promising both for the individual users and also for the network.

| Participants | Connections (Total) | Downstream | Upstream | Total   |
|--------------|---------------------|------------|----------|---------|
| 5            | 1 (5)               | 1 mbps     | 1 mbps   | 10 mbps |

## SFU

The most modern approach to date is to use an SFU (Selective Forwarding Unit), something that could be resembled a routing service. With the SFU, each participant sends its upstream towards the SFU media server and then that will selectively route this to all other participants.

How much information is going to send, what will that be, and to who depends on the specific way the SFU was programmed along with the network policy defined within the SFU.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/74beuqqxlwsazc5o77ol.png)

In the case of 5 participants, there are going to be 5 connections for each one of them, which translates to 25 total connections across the network.

Each participant has 1 upstream connection and 4 downstream connections, all originating from the SFU media server. Supposing we use the same Average Connection Bandwidth with the other topologies, the upstream bandwidth is 1 Mbps and the downstream is 4 Mbps.

This makes the total bandwidth requirement across the network to be 25 Mbps.

The approach is costlier for the user because while the upstream bandwidth remains at 1 Mbps, the downstream bandwidth is now in direct relationship with the number of participants that the call group will have.

But, the CPU cost of the SFU architecture is far lesser than the equivalent cost required from the MCU architecture.

Increasing the level of abstraction we can see that:

`total bandwidth required = number of participants x (upstream bandwidth + downstream bandwidth)`

or 

`TBR = n * (abr * (n - 1) + abr * 1)` 

In the example of 15 students attending a video call, the total bandwidth requirement would be 

`TBR = 15 * (1 * 14 + 1 * 1) = 225 mbps`

| Participants | Connections (Total) | Downstream | Upstream | Total   |
|--------------|---------------------|------------|----------|---------|
| 5            | 5 (25)              | 4 mbps     | 1 mbps   | 25 mbps |

## Conclusion

There is _no one solution that fits all sizes_ in the case of multiparty video/audio streaming or calling. Depending on the specific application requirements the programmer can make decisions on how to architect the network topology so that she gets the most value out of the available resources, including virtual network bandwidth, CPU time, and user resources.

Each topology has its benefits and drawbacks in different use cases. 

As a rule of thumb, it seems that the mesh network approach fits solutions that limit the number of participants and want to avoid centralized infrastructure costs but bearing in mind that users will have to pick up the network costs.

The MCU is resource intensive and requires a centralized infrastructure that is responsible for compositing media. Encoding is an expensive process that will require increased resources if the solution needs to be scaled. On the bright side, the bandwidth requirements for the user are minimal.

The middle ground is covered by the SFU architecture. Although still requiring centralized infrastructure, the SFU media server is essentially a byte shifter, forwarding media streams to each participant, making it way less CPU intensive than the MCU approach. This keeps the upstream bandwidth requirements for the user minimal but is increasing the downstream requirement.

With some clever programming and the use of efficient codecs and compression, it's possible to decrease the upstream and downstream bandwidth requirements and eliminate some connections in certain cases. We won't get into details this time as it would be out of scope.
