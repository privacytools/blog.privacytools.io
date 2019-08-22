---
title: Tor on privacytools.io
author: Jonah
layout: post
canonical: "https://write.privacytools.io/jonah/tor-on-privacytools-io"
tags:
  - tor
  - announcements
---

We're excited to announce the launch of Tor connectivity to the privacytools.io homepage, and we hope to get Tor working on the rest of our services as soon as possible.

**Update 5/6:** Some people asked me for a more detailed post and guide on how to set this kind of thing up on their own servers, so [I went ahead and did that](https://write.privacytools.io/jonah/securing-services-with-tor-and-alt-svc). Hopefully if you're a service operator or you just like that kind of stuff it'll be helpful to you!<!--more-->

## The Homepage

The homepage is now accessible via our new Tor hidden service: `privacy2zbidut4m4jyj3ksdqidzkw3uoip2vhvhbvwxbqux5xy5obyd.onion`! This setup in particular is a pretty standard Tor setup, so I won't go into too many details. We're using a v3 hidden service (as you can tell by the enormous domain) with the following options:

```
HiddenServiceNonAnonymousMode 1
HiddenServiceSingleHopMode 1
```

Why are we using non-anonymous mode and single-hop mode, you ask? It's mainly to optimize latency: By enabling single-hop mode, we're able to cut two hops out of the connection. This is not in any way detrimental to the anonymity or security of our users, it merely reduces the anonymity of *our* own server. This is of course fine, because we operate our servers over clearnet domains anyways with public IP addresses.

The main thing we are now accomplishing is not anonymity for ourselves, but increased security for our users. Your connection to the privacytools.io homepage is now completely secured end-to-end, no relying on exit nodes that could meddle with your traffic. It'll also be a bit faster, since we're reducing the load on the Tor exit nodes.

## Other Services

With the rest of our services, we're going to be taking a different approach to the traditional .onion domain. Remember [Cloudflare's Onion Service](https://blog.cloudflare.com/cloudflare-onion-service/)? We're basically recreating their setup on our own servers to create secure Tor connections *without using .onion domains*, at least visibly. That means that if you go to `write.privacytools.io` in the Tor Browser, for example, you'll see `https://write.privacytools.io/` in your browser, but the connection will actually be over the Tor network without the use of any exit nodes!

### alt-svc

How can we do this exactly? We're using an HTTP header called [`alt-svc`](https://tools.ietf.org/html/rfc7838). Originally designed to facilitate HTTP/2 and SPDY connections, and now commonly used for QUIC, alt-svc allows us to tell your browser that the website you're visiting is also accessible via Tor. The Tor Browser in response, makes a connection to our servers using the information within that header rather than connecting the normal way with DNS lookups and exit nodes.

The drawback to this is you need to actually connect to our websites once, so your browser has a chance to download the header and recognize that it should be using an onion connection. Because we use HTTPS with HSTS preloading this shouldn't be a security issue, since your initial connection will be made over HTTPS. That does mean that the initial connection will still take place over an exit node. After it receives that header, the information is cached, and the browser will continue to make any future connections solely over the Tor network.

### When will this happen?

We've already enabled `alt-svc` support for this WriteFreely instance, for Searx (search.privacytools.io), for our Matomo analytics platform, and for the homepage. (Yes, that means that connections to our homepage will be made over Tor regardless of whether you use the new .onion domain or the standard clearnet domain).

In the future we'll enable `alt-svc` on all our services, after we finish initial testing on the ones we enabled today.

### Is it working?

The issue with `alt-svc` at the time of writing is how new it is to the Tor network. The Tor Browser has supported connections to hidden services in this manner since Tor Browser 8.0, but doesn't make it obvious whether or not the connection actually works. So at this time, the circuit UI doesn't show the current route correctly. This is currently an open issue in the Tor bug tracker, [#27590](https://trac.torproject.org/projects/tor/ticket/27590), that I hope will be resolved in the coming weeks.

Tor for Android also supports `alt-svc`, but I have not been able to test whether or not it displays the circuit graph correctly. I assume it also does not, until #27590 is resolved.

We will most likely not roll out Tor using `alt-svc` among the rest of our services until it's better implemented in the Tor Browser.

### Discuss

You are welcome to [discuss our new Tor integration on our forum](https://forum.privacytools.io/t/the-privacytools-io-homepage-is-now-tor-accessible/172)!
