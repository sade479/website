---
title: "Setting up an adblocking DNS server"
date: 2022-12-19T13:13:03-05:00
draft: false
---

The modern internet is full of ads and trackers that can slow your devices down and put your privacy at risk. Traditional browser-based ad blockers such as AdBlock Plus can get the job done within your browser but the rest of your network is unaffected. In this tutorial, I will show you how to install AdGuard Home a DNS-based adblocker on your home network.

<!--more-->

## What is DNS?

Before starting it is important to understand what DNS is and how it plays a role in adblocking. DNS is the protocol used to convert a domain name like ``example.com`` to an IP address ``192.168.1.1``.

An adblocking DNS server expands on this concept by utilizing domain blocklists of common ad-tracking domains.

For example, let's say ``adtracker.net`` normally resolves to ``10.20.37.5`` on a standard DNS server. An adblocking DNS server will replace that IP address either with ``0.0.0.0`` an invalid IP address or an NXDOMAIN response which indicates that the domain does not exist.

## Pre-requisite

All you need is a machine running Linux (I will be using Ubuntu 22.04 in this tutorial). The machine can be something like a Raspberry PI, an old desktop your not using, or a virtual machine running on a hypervisor like VirtualBox.

## Setting up a static IP address

Before getting started with setting up AdGuard Home we will first want to ensure our machine has a static IP address in our network. The easiest way to accomplish this is by setting up a DHCP reservation in your router.

The way to accomplish this differs from router to router. For example, using an ASUS router you would go to the admin page by going to ``http://192.168.1.1`` in your web browser. From there you would click on LAN on the left-hand side then click on the DHCP server tab.

![dhcp reservations on an asus router](/images/asus-dhcp.png)

Once on that page, you want to give your Linux machine a static IP address by selecting its MAC address (you can use the ``ip a`` command to find out) and putting a memorable IP address. In the network above something like ``192.168.1.100`` would be a sensible option but this is up to you.

When you do set up a static IP address from your router you will want to reboot your Linux machine. You can use ``ip a`` to check if everything is working correctly.

Unfortunately, I cannot give specific instructions for every router so you will have to research this issue on your own. I suggest searching for ``DHCP reservation 'your router model'`` in your favorite search engine to get started.

## Getting Started

To get started setting up AdGuard Home follow the steps below.

1. Go the AdGuard Home {{<underline "https://github.com/AdguardTeam/AdGuardHome/releases/" releases>}} page on GitHub

![two adguard releases on github](/images/adguard-releases.png)

There will be many versions of AdGuard Home in the list. You will want to scroll down to the Linux versions. From there you will want to right-click and copy the link with your specific CPU architecture amd64 (for standard PCs or VMs), arm64 (64-bit Raspberry PIs or other arm single board computers), or armv7 (32bit Raspberry PIs or other arm single board computers).

You may need to click the assets drop-down arrow and/or click the show all assets link to view the full list.

![github show all assets link](/images/github-showall.png)

![github assets drop-down arrow](/images/github-assets-arrow.png)

2. Download AdGuard Home using wget
```
wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.107.21/AdGuardHome_linux_amd64.tar.gz
```
Note replace the link with the one you copied earlier. Should be in a similar format.

3. Extract the tar archive
```
tar xzf 'file you just downloaded'.tar.gz
```
4. Navigate to the AdGuard Home folder
```
cd AdGuardHome
```

## Setting up AdGuard Home

Now that you have downloaded AdGuard Home to our system it is time to run it and start our initial configuration.

The way to do this is to run the application directly: ``sudo ./AdGuardHome``.

From there open up a web browser and go to ``http://"SERVER IP":3000``.

You should be greeted with a screen like so.

![intital setup screen for adguard home](/images/adguard-gettingstarted.png)

In the next screen of the setup process, you will want for AdGuard Home to either listen on all interfaces or listen on the interface with your current network address. In some Linux distributions due to the presence of systemd-resolved you will have to select a specific interface for the actual DNS server.

![setup screen for selecting ports](/images/adguard-interfaces.png)

After that, you will have to set up a username and password.

![setup screen for username and password](/images/adguard-password.png)

Once that is done you will have completed the initial AdGuard Home setup. From there you will be prompted to login and after doing so you will be presented with the main dashboard.

![main dashboard for adguard home](/images/adguard-dashboard.png)

Now that you have successfully set up AdGuard Home you will want to close your browser and go back to the terminal where you started AdGuard Home. Use CTRL+C to exit the process.

Run the following command to install AdGuard Home as a service. This will run AdGuard Home in the background and will automatically startup the main process when your system boots up.

```
sudo ./AdGuardHome -s install
```

## Configuring your network to use AdGuard Home

With AdGuard Home up and running you will now want to configure your router to use your custom DNS server. Once again this process will differ from router to router.

In an ASUS router, you will want to go back to the admin page where you have set up the static IP address. From there you will want to put in your custom DNS IP in the ``DNS Server 1`` box. In addition, you will want to set the option ``Advertise router's IP in addition to user-specified DNS`` to no.

![dns settings on asus router](/images/asus-dns.png)

If your router allows you to put more than one DNS server just leave the other boxes blank.

I also cannot give specific instructions for every router here so I suggest searching ``changing DNS server 'your router model'`` to get started.

## Conclusion

Congratulations you have successfully set up an adblocking DNS server on your local network. I have not even begun to scratch the surface of what AdGuard Home can do. I suggest going through the interface and trying out all of the different settings to see what you can accomplish with AdGuard Home.

Some features include:
- Parental Controls
- Custom Block lists
- Custom blocking of certain services
- And more!





