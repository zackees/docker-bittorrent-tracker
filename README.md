BitTorrent Tracker [![Build Status](https://travis-ci.org/philipphenkel/docker-bittorrent-tracker.svg?branch=master)](https://travis-ci.org/philipphenkel/docker-bittorrent-tracker) [![Dependency Status](https://david-dm.org/philipphenkel/docker-bittorrent-tracker.svg)](https://david-dm.org/philipphenkel/docker-bittorrent-tracker)
==================

Node.js Docker image containing a [BitTorrent tracker](https://wiki.theory.org/BitTorrentSpecification#Tracker_HTTP.2FHTTPS_Protocol).
The Node service is using Feross Aboukhadijeh's excellent [bittorrent-tracker](https://github.com/feross/bittorrent-tracker) library.

Besides HTTP and UDP the tracker also supports WebSockets.

Usage
-----

```console
docker run --rm -i -t henkel/bittorrent-tracker:latest
```

By default port 80 is exposed. In order to run a detached tracker at port 8100, just call

```console
docker run --name bittorrent-tracker -d -p 8100:80 henkel/bittorrent-tracker:latest
```

Updating
--------

The container is stateless and you can easily update to the latest version at any time:

```console
docker stop bittorrent-tracker
docker rm bittorrent-tracker
docker pull henkel/bittorrent-tracker:latest
```

Follow the usage instructions to re-start the service.

License
-------

Copyright (C) 2016 Philipp Henkel

Licensed under the MIT License (MIT). See LICENSE file for more details.


How to setup a free webtorrent tracker for your video distribution needs
========================================================================

By Zach Vorhies

# Who is this for

This document is geared toward developers that want to quickly get a video distribution system up and running as *easily* as possible.

# What is webtorrent?

Webtorrent is the bittorrent file sharing protocol extended to use WebRTC and websockets, enabling file sharing all within the browser without the need for extensions or other native binaries.

# Foot guns

Webtorrent doesn't work for every consumer. Some of your users (30%) will be using an ISP that uses something called "Symmetric NAT", which prevents them from communicating with the peer swarm.

# Terminology

## Trackers and Clients

Most of us understand the concept of a server and a client, common in star topology networks which dominate teh internet. Bittorrent is decentralized so the terms "clients" and "servers" can be replaced with "clients" and "trackers".


### Trackers

Trackers coordinate the swarm, but they don't participate in file sharing. The coordination uses very few resources and transmits very little information per client. This makes the Trackers extremely light weight and use few resources and allows them to exist on very cheap servers. Even singlethreaded javascript implementations of trackers are fast enough to handle all the traffic anyone will ever likely ever need. For example bittorrent-tracker has been benchmarked at 600 connections per second, which translates to 10's of thousands of simultaneous users. Some of the new trackers written in performant Rust code (such as aquatic tracker) can handle upto 80k requests per second with just a single server and can serve 10's of millions of users in a swarm.

When a client wants to connect to the swarm, it will first contact a one or more trackers and request information about the swarm of peers leeching and seeding a file. Swarms are organized around info-hashes embedded in the torrent. The tracker will respond with providing information about some or all of the peers participating in the swarm, which the client will then start interacting with. The peer will now be part of the swarm!

### Clients

A client includes both users that want to download content, as well as those that are uploading it to others.

For example, do you want to upload a video and make it available? You are are actually a client, even though you are serving the file to others.


### Torrent

A torrent file is metadata about the content being shared. It uses magic hashing algorithms to form a block-chain like data structure which ensures integrity of the file similar to proof of work algorithms in Bitcoin. The great thing about torrent files is that they are very small in comparison to the content they are sharing. For example a torrent file representing a 4gb video file will be about 1k of data.

If the exactly same file is used to generate two torrents, they will both be the same.


## Torrent InfoHash

Hashes on top of hashes! This is the hash of the torrent file and represents a unique id for the torrent in question. In the webtorrent world, clients will ask trackers for any torrent files that are represented by the torrent InfoHash and get the torrent file and swarm information.

Psuedo code example:

```js
function respond(infohash) {
	const torrent = hashtable.get(infohash)
	if (torrent === undefined) {
      return Response(404, `Torrent for ${infoHash}'"')
	} else {
	  return Response(200, torrent)
	}
}
```

Technically, two torrents could generate the same InfoHash, but it's so extremely rare that it might as well be impossible.

### MagnetURI

MagnetURI's are giant URL's that contain enough information for a client to find a file it's looking for. 
MagnetURI's == Torrent.InfoHash + one or more trackers

Technically, MagnetURI's with ZERO trackers are valid, but have little use.

Example:

```
magnet:?xt=urn:btih:94993a31534e1a8466230e27be4ab1a5767eb8b5&dn=beavis_and_butthead.mp4&tr=wss%3A%2F%2Fwebtorrent-tracker.onrender.com%2F
```

This is URL encoded form, which when url-decoded produces a more human readable format:

```
magnet:?xt=urn:btih:94993a31534e1a8466230e27be4ab1a5767eb8b5&dn=beavis_and_butthead.mp4&tr=wss://webtorrent-tracker.onrender.com/
```

First there is the required torrent infohash, followed by an optional torrent name "beavis_and_butthead.mp4", followed by a public tracker `wss://webtorrent-tracker.onrender.com/` which is specified to be able to resolve the torrent hash to a full torrent file + swarm of peers.

MagnetURI's are generated by `seeders`, such as can be found by `webtorrentseeder.com`


## STUN Servers (optional)

There are many free stun servers and as a result you'll likely not have to configure this at all since your program will use sensible defaults (e.g. Google's free STUN servers). But it's worth it to explain in simple english since it's used in the documenation elsewhere.

Because webtorrent uses WebRTC to connect with peers, it needs to have information about it's real IP address. Because of Network Address Translation (NAT), the IP / Port that your computer uses is mostly likely different than your actual public IP address when you communicate with other computers. The translation from your public ip and port to your local ip and port is handled by your ISP. To make a public connection with a peer, you need to get the real public ip address. To get this your computer will contact a STUN server, which will respond with the IP and Port (and other routing information) necessary to form this WebRTC peer connection.

STUN servers are lightweight and use very little bandwidth. Therefore you can find lots and lots of free STUN servers.

Some clients however are behind a special kind of NAT called a "Symetric NAT" which prevents peers from joining a swarm (technically they could download from peers, but this appears unimplemented). This will become apparent when they use the STUN server to try and resolve their connection information, which will fail. So what do we do in this case?

## TURN Servers (optional)

TURN servers are a relay which a file is transfered through. Its works because Symettric NATs have no problem connecting to a host on the internet, so a TURN server acts as the middle man between two Symetric NATS. TURN servers are expensive, high bandwidth and it's unlikely to find a public TURN server that is free. Infact, most of them require credentials.

TURN servers are unnecessary for legal content, because if the Symettric NAT client can't connect then it can simply download the file from a public server as a fallback.

## ICE Candidate

When a client wants to join the swarm, it will convert to an ICE candidate after it has it's connection information provided by a STUN server.


# Getting up and running

## Easy mode - use public trackers

This is super easy because you don't have to deploy any tracker software to the internet. You can just start swarming by visiting a few websites:
  1. Goto [webtorrentseeder.com](https://webtorrentseeder.com)
  2. Leave the tracker field at the defaults
  3. And then upload a file. You'll start seeding immediatly
    a. Unless you are behind a Symetric NAT, in which case try again through a VPN.
  4. Copy the magnet link
  5. Open another [webtorrentseeder.com](https://webtorrentseeder.com) browser window and paste in the magnet URI, the file should start downloading.
    a. Important! Make sure each browser window is running by itself. If it's part of a collection of tabs then many browsers will suspend the websocket network connection. if you notice that the connection is extremely slow, then try tearing out the tab into it's own group and it's likely the connection will return to full throttle.

Public trackers have been pretty stable so for small time content thats not critical (daily videos, etc) it's probably fine that you use a public tracker to manage your peer swarm.
