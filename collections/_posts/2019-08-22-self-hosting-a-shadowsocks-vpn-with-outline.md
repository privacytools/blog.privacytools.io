---
title: Self-hosting a Shadowsocks VPN with Outline
author: Jonah
layout: post
cover: /assets/img/2019-08-22-outline-cover.jpg
tags:
  - guides
  - vpn
  - shadowsocks
---

**Outline** is a suite of open-source software developed for journalists to safely access their network and the internet while traveling in countries where their activities may be monitored or censored.

### Prerequisites

All you will need to complete this guide is a computer running Windows, macOS, or Linux. You will also need to know some basic commands: [How to SSH](https://www.howtogeek.com/311287/how-to-connect-to-an-ssh-server-from-windows-macos-or-linux/) in to a server you purchase. We will also assume you know how to purchase and set up a Linux server with SSH access, more info in Step 2.

### Step 1 — Download & Install Outline Manager

Outline allows you to setup and configure your servers from an easy-to-use management console called Outline Manager, which can be downloaded from [getoutline.org](https://getoutline.org/en/home). It has binaries available for Windows, macOS, and Linux.

Simply download and install the Outline Manager application to your computer.

![](/assets/img/2019-08-22-outline-install.png){: .w-100}

### Step 2 — Choose a Server Provider

Outline has the ability to create servers on three different providers automatically: DigitalOcean, Google Cloud, and Amazon Web Services. In some situations, Google Cloud or AWS may be preferable, because they are less likely to be blocked by hostile ISPs/governments and will therefore allow you to more likely circumvent internet censorship. However, keep in mind that the server provider you choose—like any VPN provider—will have the technical ability to read your internet traffic. This is much less likely to happen when using a cloud provider versus a commercial VPN, which is why we recommend self-hosting, but it is still possible. Choose a provider you trust.

Another factor to consider is your provider's network and latency. Choosing a server closer to you (geographically speaking) will give you better latency, and choosing a server with good bandwidth (>1 Gbps) will minimize the performance hit when using the VPN. Both factors are important to keeping a good browsing experience, but keep in mind using *any* VPN will always be slower than just your plain old internet connection.

Finally, if you want to go with DigitalOcean you can use my affiliate link to receive a $50 credit: [https://m.do.co/c/fb6730f5bb99](https://m.do.co/c/fb6730f5bb99). That's 10 months of free VPN hosting, at $5/month/server. Don't feel obligated to use this link, but you'll receive free credit, and if you spend $25 with DigitalOcean after using it I will get credited, which will enable me to continue writing guides like this! DigitalOcean has a great performing network in my personal experience, and in the experience of the Outline team it works well in regard to circumventing censorship: Not many IP addresses of theirs are blocked.

For this guide we are not going to use an automatic provider in Outline Manager, rather we will manually configure a Linux server. We are using Debian 10. Other distros may work as well, but you may need to install Docker manually.

### Step 3 — Configure Your Server

First, we need to install `curl`. Connect to your server via SSH and enter the following commands:

```
sudo apt update
sudo apt install curl
```

Next open Outline Manager on your local machine and you should be given 4 options to configure a server. Select the "Set Up" button under the "Advanced, Set up Outline anywhere" option.

![](/assets/img/2019-08-22-outline-server-setup.png){: .w-100}

Outline will give you a string to paste. Connect to your server over SSH and paste the code in the Outline Manager box into the Terminal. The process will take a minute or two and will ask you a couple questions. You can just press enter to accept the default configuration whenever it asks.

After it completes, it will give you a long line starting with `{"apiUrl"` (depending on your Terminal's color support it will appear as green). Copy that line, and paste it in the second box back in Outline Manager. Then, click "Done".

### Step 4 — Connect Your Devices

Download the Outline app on the device you want to connect. Outline has applications for the following operating systems:

- [Android](https://play.google.com/store/apps/details?id=org.outline.android.client)
- [iOS](https://itunes.apple.com/us/app/outline-app/id1356177741)
- [Windows](https://raw.githubusercontent.com/Jigsaw-Code/outline-releases/master/client/stable/Outline-Client.exe)
- [macOS](https://itunes.apple.com/us/app/outline-app/id1356178125)
- [Chrome OS](https://play.google.com/store/apps/details?id=org.outline.android.client)
- [Linux](https://raw.githubusercontent.com/Jigsaw-Code/outline-releases/master/client/stable/Outline-Client.AppImage)

You should also be able to use any [Shadowsocks client](https://shadowsocks.org/en/download/clients.html), including alternative clients for each operating system and a client for OpenWRT routers.

Back in Outline Manager, select your server in the sidebar. On the far right side of "My access key" there is an icon of a laptop and phone. Click that icon, and select "Connect This Device" in the popup window. It will give you a string to copy, starting with `ss://`. Simply paste that string into the configuration of any Shadowsocks client to add your server!

Once you add your server, that's it! In the Outline clients it's just a matter of pressing "Connect", and all your traffic will be proxied through your server! You can use this connection to keep your traffic safe when you're on public WiFi networks, or just to keep your browsing hidden from your ISP.

### Conclusion

That should be all you need to get your very own VPN up and running! **Do not share your access key with anyone**, this is the key starting with `ss://`. If you want to grant other users access to your server, click "Add a new key" in Outline Manager and give them a new, unique key. If you share a key, anyone with knowledge of that key will be able to see all the traffic of anyone else using the key.

With Outline, there is no need to worry about the security of your server. Everything is set to automatically update with no intervention required!
