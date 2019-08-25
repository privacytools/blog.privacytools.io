---
title: Adding Custom DNS over HTTPS Resolvers to DNSCloak
author: nitrohorse
editor: Jonah
layout: post
tags:
  - guides
  - dns
  - ios
---

**[DNSCloak](https://apps.apple.com/us/app/dnscloak-secure-dns-client/id1452162351)** is an [open-source](https://github.com/s-s/dnscloak) DNSCrypt and DNS over HTTPS (DoH) client for iOS; giving users the ability to encrypt their DNS requests. While highly configurable, its user interface can be unintuitive and doesn't easily allow users to add custom DoH resolvers apart from the default ["public-resolvers" list that DNSCrypt provides](https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v2/public-resolvers.md).

This guide will walk you through setting up DNSCloak to connect to any public resolver that supports either DNSCrypt or DoH.

### Adding a Custom Resolver

DNSCloak provides a "Config Editor" which allows you to modify various aspects of the program, add custom servers, etc:

![](/assets/img/2019-08-24-dnscloak/config-editor.jpeg){: .w-50}

We can learn about the various configuration options from the [example config file](https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-proxy/example-dnscrypt-proxy.toml) in dnscrypt-proxy's code repository. If you scroll all the way to the bottom, you'll find a `[static.'myserver']` section along with a `stamp` property. This stamp is for adding the [DNS stamp](https://dnscrypt.info/stamps-specifications) of your resolver.

> Server stamps encode all the parameters required to connect to a secure DNS server as a single string. Think about stamps as QR code, but for DNS.

At the time of writing this post, CZ.NIC's DoH resolver is the only one [PrivacyTools.io suggests](https://www.privacytools.io/providers/dns/#icanndns) that doesn't provide their users with a DNS stamp on their website, making adoption a bit more difficult. But thankfully, we can create a DNS stamp ourselves.

DNSCrypt hosts a [DNS stamp calculator](https://dnscrypt.info/stamps/) (which you can also [download, compile, and run offline](https://github.com/jedisct1/vue-dnsstamp)) that we can fill out with the info CZ.NIC provides on their webpage to generate our stamp.

We'll need three things about the DoH resolver:
1. IP address
2. Host name
3. Path

Navigate to [CZ.NIC's webpage](https://www.nic.cz/odvr/) and scroll down to "How to turn on DNS-over-HTTPS (DoH)" to note the URL (https://odvr.nic.cz/doh).

![](/assets/img/2019-08-24-dnscloak/cz-nic-doh.png){: .w-100}

Next, scroll up to the setup sections for either Windows, macOS, or Linux and copy one of the IPv4 addresses of the DoH resolver (`193.17.47.1` or `185.43.135.1`).

![](/assets/img/2019-08-24-dnscloak/cz-nic-ips.png){: .w-50}

Now we can paste what we've gathered into the stamp calculator:
- IP address: `193.17.47.1`
- Host name: `odvr.nic.cz`
- Path: `/doh`

![](/assets/img/2019-08-24-dnscloak/cz-nic-stamp.png){: .w-100}

We'll find that the DNS stamp is `sdns://AgMAAAAAAAAACzE5My4xNy40Ny4xAAtvZHZyLm5pYy5jegQvZG9o`.

Let's copy and paste our new configuration into DNSCloak's Config Editor at the bottom:

```
[static.'CZ.NIC-193.17.47.1']
stamp = 'sdns://AgMAAAAAAAAACzE5My4xNy40Ny4xAAtvZHZyLm5pYy5jegQvZG9o'
```

![](/assets/img/2019-08-24-dnscloak/config-editor-cz-nic.jpeg){: .w-50}

Save by selecting the checkmark icon on the top right, and now we can use our new configuration:

![](/assets/img/2019-08-24-dnscloak/dnscloak-cz-nic.jpeg){: .w-50}

Connect, and let's finally validate DNSCloak is working as expected by visiting [DNSLeakTest.com](https://dnsleaktest.com/):

![](/assets/img/2019-08-24-dnscloak/dnsleaktest-cz-nic.jpeg){: .w-100}

### Adding Cloudflare's Resolver for Firefox

Another public DoH resolver that we may want to use is Cloudflare's public resolver for _Firefox_ [which has a stricter logging policy than Cloudflare's default resolver](https://forum.privacytools.io/t/logging-differences-between-cloudflares-default-dns-over-https-resolver-and-their-resolver-for-firefox/1451).

We can [generate a stamp](https://dnscrypt.info/stamps/) with this information:
- IP address: `1.1.1.1`
- Host name: `mozilla.cloudflare-dns.com`
- Path: `/dns-query`

![](/assets/img/2019-08-24-dnscloak/cloudflare-mozilla-stamp.png){: .w-100}

You can now paste this into DNSCloak's Config Editor and start using the resolver.

```
[static.'Cloudflare Resolver for Firefox']
stamp = 'sdns://AgUAAAAAAAAABzEuMS4xLjEAGm1vemlsbGEuY2xvdWRmbGFyZS1kbnMuY29tCi9kbnMtcXVlcnk'
```

![](/assets/img/2019-08-24-dnscloak/config-editor-cf-moz.jpeg){: .w-50}

![](/assets/img/2019-08-24-dnscloak/dnscloak-cf-moz.jpeg){: .w-50}
