---
title: Securing Services with Tor and alt-svc
author: Jonah
layout: post
canonical: "https://write.privacytools.io/jonah/securing-services-with-tor-and-alt-svc"
tags:
  - tor
  - sysadmin
  - guides
---

Some people called for me to write a more technically detailed/in-depth guide to setting up Tor with `alt-svc` after [we set it up on privacytools.io](https://write.privacytools.io/jonah/tor-on-privacytools-io), so while I do it all over again on our Mastodon server, I figured I'd write this post! Plus, it'll make it easier for me if I need to do this again in the future :)<!--more-->

When Cloudflare [introduced their Onion Service](https://blog.cloudflare.com/cloudflare-onion-service/) last year, it marked an important milestone in Tor adoption and connectivity. Not only did they add Tor support to nearly all their websites, which will certainly help with reducing the number of captchas seen by Tor users across the internet, it introduced a new and very interesting way to handle Tor traffic.

What we're doing here is reimplementing Cloudflare's setup on our own machines. The `privacytools.io` websites and services are now Tor-enabled completely transparently to all users using the Tor browser, because of an HTTP header called `alt-svc`.

## What is alt-svc?

`alt-svc` is a HTTP header that allows your server to inform a web browser about alternative ways to reach your website. Initially it was developed for SPDY and now [QUIC](https://en.wikipedia.org/wiki/QUIC) support for websites in browsers that support it, but now we can use it to tell browsers (the Tor Browser specifically) that our site is available at an .onion domain rather than a clearnet IP address.

What this means is when you connect to `privacytools.io` in your Tor browser, instead of your computer running a DNS lookup and making TCP requests to `145.239.169.56:443`, it will see that a `.onion` connection is available and make TCP requests to `privacy2zbidut4m4jyj3ksdqidzkw3uoip2vhvhbvwxbqux5xy5obyd.onion:443` instead, *completely transparently* and keeping the hostname identical!

This is highly beneficial for server administrators for the following reasons:

1. **The domain name stays the same.** Instead of adding a `.onion` domain to Nginx, we're sending all requests using the same hostname. If you're familiar with the OSI 7 layer model, we're replacing the Transport (layer 4) TCP layer with a .onion connection while not touching anything on the Application layer (layer 7).
2. **It requires no special webserver configuration.** Because the hostname stays the same, requests will reach your webserver with the same Host header. To Nginx and your web application, it should appear the same as a normal connection.

Because of this, using `alt-svc` is a drop-in replacement for TCP connections and should be able to be implemented on *any* web application stack without making *any* modifications to your code. All you need is Tor installed, and a single header added to Nginx.

Using `alt-svc` headers is not necessarily a replacement for .onion domains. It's an *opportunistic* use of the Tor network, not enforced. Some downsides to keep in mind:

1. **Initial connections will go through an exit node!** The Tor Browser won't know about your .onion address until it connects to the site for the first time.
2. **It isn't quite 100% supported**. It is supported in Tor Browser 8.0 and Tor Browser for Android, but not necessarily anything else. It's an HTTP header, so it won't work for other protocols.
3. **There's** (*currently*) **no visual indicators a connection is secure.** This is an open issue ([#27590](https://trac.torproject.org/projects/tor/ticket/27590)) so I hope it will be fixed soon, but currently it just appears as if you're connecting via an exit node.

The first point is probably going to be the most concerning for most service hosters. We decided that for our purposes this didn't matter that much: We currently host all our services in the clearnet anyways, we enforce HTTPS use for End-to-End Encryption, and we don't need to hide ourselves. Using `alt-svc` is just *added* security for Tor users, and it functioning essentially transparently means that people don't need to change their current behaviors when visiting our websites. We're also willing to wait for visual indicators to be implemented in the browser.

If you *need* all your users to connect via the Tor network, or if visual security is important to your services/industry/etc, you do need to host your services on an .onion domain directly, no way around it for the moment (until perhaps alt-svc is implemented in DNS and there's a way to enforce its use).

## Setting up a Hidden Service

First thing's first, we need to actually install Tor. I'm going to be installing Tor on Ubuntu 18.04 for the following steps. But you should [consult the latest Tor instructions](https://2019.www.torproject.org/docs/tor-onion-service.html.en#zero) to make sure you're doing things however they currently want you to, or to find guides for your own platform.

If you don't have it installed already, install `apt-transport-https`:

```
apt update
apt install apt-transport-https
```

This will allow your package manager to access all metadata and packages over HTTPS when using apt.

Now we'll need to add the official Tor browser sources. Open `/etc/apt/sources.list` with your favorite editor and add [the following lines](https://2019.www.torproject.org/docs/debian.html.en#ubuntu) to the end:

```
deb https://deb.torproject.org/torproject.org bionic main
deb-src https://deb.torproject.org/torproject.org bionic main
```

Then, add their signing key:

```
curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -
```

Now, update your packages:

```
apt update
```

Finally, we'll install `tor`. We'll also install `deb.torproject.org-keyring`, which keeps the signing key up to date, which the Tor project recommends:

```
apt install tor deb.torproject.org-keyring
```

## Hidden Service Configuration

Now we can setup our .onion domain. Open `/etc/tor/torrc` and add the following setting:

```
SocksPort 0
```

Then, find the `# This section is just for location-hidden services` section and add the following configuration:

```
HiddenServiceDir /var/lib/tor/service1/
HiddenServiceVersion 3
HiddenServicePort 443 127.0.0.1:443
HiddenServiceNonAnonymousMode 1
HiddenServiceSingleHopMode 1
SafeLogging 1
```

`SocksPort 0` disables the socks proxy, which we don't need since we're only functioning as a service.

`HiddenServiceDir /var/lib/tor/service1/` is the directory that will store our private keys, and *should not be publicly accessible*. It is not your web root.

`HiddenServiceVersion 3` enables the new v3 onion addresses, which are more secure and longer than v2 addresses.

`HiddenServicePort 443 127.0.0.1:443` redirects traffic to youronion[...]domain.onion:443 to 127.0.0.1:443, which should be your webserver. You can point this IP to another server on your LAN as necessary, or add similar lines for other ports you want to host.

`HiddenServiceSingleHopMode 1` is used to optimize latency a bit. It removes two hops in the Tor circuit by moving your server closer to the "rendezvous point".

`HiddenServiceNonAnonymousMode 1` is enabled because of the last setting. Removing the two hops in the circuit decreases the anonymity of your server significantly, but has no effect on the security or anonymity of your users!

The last two options are enabled because we're assuming you're not really running an anonymous service, rather just want to add additional security to an existing clearnet service. In that situation, trading the anonymity of your server for increased performance makes sense, but you can omit those two lines if that's a concern for you.

`SafeLogging 1` further prevents potentially sensitive information from being leaked to logs.

Now we can save the `torrc` file and restart Tor:

```
systemctl restart tor
```

Sidenote: if you're running Tor in an LXC container, sometimes it doesn't start correctly, but [there's a workaround](https://www.reddit.com/r/TOR/comments/8cftk3/having_trouble_with_tor_running_in_lxd_container/). For most people this won't matter, but I use LXC containers heavily so this would've saved me a good hour of research.

## Adding Headers

You can find your hostname at `/var/lib/tor/service1/hostname`:

```
root@social:~# cat /var/lib/tor/mastodon/hostname
2tjcxjzxgql6wilo3bu777pkvigx2wqwauxf7lyvvmsotjgrqiwwg7id.onion
```

And that address should be accepting traffic on port 443. The important thing to remember is, your users will never need to use or know this address! This entire process is transparent to your users.

To enable traffic on this address, we just need to add an alt-svc header formatted like this:

```
alt-svc: h2="2tjcxjzxgql6wilo3bu777pkvigx2wqwauxf7lyvvmsotjgrqiwwg7id.onion:443"; ma=86400; persist=1
```

Of course, replace `2tjcxjzxgql6wilo3bu777pkvigx2wqwauxf7lyvvmsotjgrqiwwg7id.onion` with your own hostname.

The field “ma” (max-age) indicates how long in seconds the client should remember the existence of the alternative service and “persist” indicates whether alternative service cache should be cleared when the network is interrupted.

To add this header in Nginx, add the following to your `server { ... }` block:

```
add_header Alt-Svc 'h2="2tjcxjzxgql6wilo3bu777pkvigx2wqwauxf7lyvvmsotjgrqiwwg7id.onion:443"; ma=86400; persist=1';
```

For other webservers, consult their instructions for adding HTTP headers.

Confirm your Nginx configuration is correct:

```
nginx -t
```

...and reload your webserver!

```
systemctl reload nginx
```

Now if you view the headers of your site (using Inspect Element or a service like [securityheaders.com](https://securityheaders.com/?q=https%3A%2F%2Fsocial.privacytools.io%2F&hide=on&followRedirects=on)), you should see your `alt-svc` service listed, and you're good to go!

## Conclusion

Unfortunately there's no easy way to definitively tell if it's working at the time of writing. `alt-svc` support is very new to the Tor network, but it's making great progress.

This isn't a suitable replacement for actual *hidden* services, but if you operate a clearnet website, this is a great way to help speed up your site for Tor users and reduce load on the exit nodes in the network.

Almost every privacytools.io service is using this method already, I hope this inspires you to convert your own clearnet websites to Tor compatible ones!

[Discuss this post on Privacy Forum!](https://forum.privacytools.io/t/securing-services-with-tor-and-alt-svc/369?u=jonah)

#tor #sysadmin
