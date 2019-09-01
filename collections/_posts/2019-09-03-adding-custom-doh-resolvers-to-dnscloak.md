---
title: Adding Custom DNS over HTTPS Resolvers to DNSCloak
author: nitrohorse
editor: Jonah
layout: post
tags:
  - guides
  - dns
  - dnscrypt
  - ios
---

**[DNSCloak](https://apps.apple.com/us/app/dnscloak-secure-dns-client/id1452162351)** is an [open-source](https://github.com/s-s/dnscloak) DNSCrypt and DNS over HTTPS (DoH) client for iOS, which gives users the ability to encrypt their DNS requests through the use of an on-device VPN profile.

While highly configurable, its user interface can be unintuitive to less tech-savvy users and doesn't easily allow users to use custom DoH resolvers, apart from the default ["public-resolvers" list](https://github.com/DNSCrypt/dnscrypt-resolvers/blob/master/v2/public-resolvers.md) that the DNSCrypt project provides.

Before diving in, it's important to understand that while there is a lot of nuance to DNSCrypt and DoH, these two DNS protocols essentially achieve the same goals: They both provide users with the ability to encrypt all DNS traffic to the users' desired [upstream provider(s)](https://www.privacytools.io/providers/dns/#icanndns), while preventing [DNS hijacking](https://en.wikipedia.org/wiki/DNS_hijacking), [spoofing](https://en.wikipedia.org/wiki/DNS_spoofing), and eavesdropping by 3rd parties.

The development of these DNS protocols is exciting, and unlike Android 9 which has [built-in support for DNS over TLS](https://support.google.com/android/answer/9089903) (another protocol with similar goals), iOS unfortunately does not allow users to enable encrypted DNS as easily ([but may in the future](https://dnsdisco.com/iOS-dns-proxy-post.html)). Thus, DNSCloak fills the gap for iOS users to start benefitting from these protocols now.

This guide will walk you through setting up DNSCloak to connect to any public resolver that supports either DNSCrypt or DoH.

### Adding a Custom Resolver

DNSCloak provides a "Config Editor" which allows you to "override or add any [dnscrypt-proxy option](https://github.com/jedisct1/dnscrypt-proxy/wiki/Configuration)."

![](/assets/img/2019-08-24-dnscloak/config-editor.jpeg){: .w-50}

You can learn more about the various configuration options from the [example configuration file](https://github.com/jedisct1/dnscrypt-proxy/blob/master/dnscrypt-proxy/example-dnscrypt-proxy.toml) in dnscrypt-proxy's code repository. But, if you scroll all the way to the bottom you'll find a `[static.'myserver']` section along with a `stamp` property. This stamp is for adding your resolver's [DNS stamp](https://dnscrypt.info/stamps-specifications), an encoded string that contains all the required information needed to connect to an encrypted DNS resolver. You can think about stamps as QR code, but for DNS.

### Generating a Stamp

Some providers will provide you with a DNS stamp pre-made for you. If your provider does this, great! You can skip ahead to the next section. At the time of writing this post, CZ.NIC's DoH resolver is the only provider [listed on privacytools.io](https://www.privacytools.io/providers/dns/#icanndns) that doesn't provide their users with a DNS stamp on their website, making adoption a bit more difficult. Thankfully however, we can create a DNS stamp ourselves.

To generate a DNS stamp, DNSCrypt hosts a [DNS stamp calculator](https://dnscrypt.info/stamps/) (which you can also [download, compile, and run offline](https://github.com/jedisct1/vue-dnsstamp)) that we can fill out with the information from our DNS provider. We'll be using CZ.NIC's information as an example to generate our stamp.

We will need to know three things about the DoH resolver you choose:
1. IP address
2. Host name
3. Path

Browse to [CZ.NIC's webpage](https://www.nic.cz/odvr/)—there is an English language option at the top of the page—and scroll down to "How to turn on DNS-over-HTTPS (DoH)" and note the URL (in this case, `https://odvr.nic.cz/doh`).

![](/assets/img/2019-08-24-dnscloak/cz-nic-doh.png){: .w-100}

Next, find the IPv4 addresses of the DoH resolver in any of the Windows, macOS, or Linux setup sections, and copy one of them (in this case, `193.17.47.1` or `185.43.135.1`).

![](/assets/img/2019-08-24-dnscloak/cz-nic-ips.png){: .w-50}

Now we can paste what we've gathered into the stamp calculator:
- IP address: `193.17.47.1`
- Host name: `odvr.nic.cz`
- Path: `/doh`

![](/assets/img/2019-08-24-dnscloak/cz-nic-stamp.png){: .w-100}

We'll find that the DNS stamp is `sdns://AgMAAAAAAAAACzE5My4xNy40Ny4xAAtvZHZyLm5pYy5jegQvZG9o`.

### Adding Resolvers to DNSCloak

Now that we have a DNS stamp generated, we can copy and paste our new configuration into the bottom of DNSCloak's Config Editor, like so:

```
[static.'CZ.NIC-193.17.47.1']
stamp = 'sdns://AgMAAAAAAAAACzE5My4xNy40Ny4xAAtvZHZyLm5pYy5jegQvZG9o'
```

![](/assets/img/2019-08-24-dnscloak/config-editor-cz-nic.jpeg){: .w-50}

Select the checkmark icon in the top right corner to save your configuration, and it should be good to go!

![](/assets/img/2019-08-24-dnscloak/dnscloak-cz-nic.jpeg){: .w-50}

Get connected, and we can finally validate DNSCloak is working as expected by visiting [DNSLeakTest.com](https://dnsleaktest.com/):

![](/assets/img/2019-08-24-dnscloak/dnsleaktest-cz-nic.jpeg){: .w-100}

### Adding Cloudflare's Resolver for Firefox

Another public DoH resolver that we may want to use is Cloudflare's public resolver for _Firefox_ [which has a stricter logging policy than Cloudflare's default resolver](https://forum.privacytools.io/t/logging-differences-between-cloudflares-default-dns-over-https-resolver-and-their-resolver-for-firefox/1451).

We can [generate a stamp](https://dnscrypt.info/stamps/) with this information:
- IP address: `1.1.1.1`
- Host name: `mozilla.cloudflare-dns.com`
- Path: `/dns-query`

![](/assets/img/2019-08-24-dnscloak/cloudflare-mozilla-stamp.png){: .w-100}

You can now paste the following stamp we generated into DNSCloak's Config Editor and start using the resolver.

```
[static.'Cloudflare Resolver for Firefox']
stamp = 'sdns://AgUAAAAAAAAABzEuMS4xLjEAGm1vemlsbGEuY2xvdWRmbGFyZS1kbnMuY29tCi9kbnMtcXVlcnk'
```

![](/assets/img/2019-08-24-dnscloak/config-editor-cf-moz.jpeg){: .w-50}

![](/assets/img/2019-08-24-dnscloak/dnscloak-cf-moz.jpeg){: .w-50}

### Summary

Keep in mind that encrypted DNS _won't hide the host name_ (for example, `blog.privacytools.io`) of the sites you visit from your ISP due to [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication#Security_implications)*.

If you're looking for anonymity, you can use [Tor Project's](https://www.torproject.org) [Onion Browser](https://onionbrowser.com/) (but keep in mind [its limitations](https://github.com/OnionBrowser/OnionBrowser/wiki/Traffic-that-leaks-outside-of-Tor-due-to-iOS-limitations)). On the other hand, if you want to hide your browsing history from your ISP, you can look into [self-hosting a VPN](https://blog.privacytools.io/posts/self-hosting-a-shadowsocks-vpn-with-outline/) or using [WireGuard](https://apps.apple.com/us/app/wireguard/id1441195209?ls=1) (if supported) or [Passepartout](https://passepartoutvpn.app/) with a [VPN provider](https://www.privacytools.io/providers/vpn/) you can trust.

But for additional security and increased privacy from 3rd parties, encrypted DNS is a great place to start.

&#42; At the time of this post, _encrypted_ SNI is [available for testing](https://blog.mozilla.org/security/2018/10/18/encrypted-sni-comes-to-firefox-nightly/) in only in Firefox Nightly, and will hopefully become integrated into other browsers and platforms in the near future.
