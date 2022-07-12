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
