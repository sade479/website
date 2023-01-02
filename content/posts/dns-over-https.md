---
title: "Hosting your own DNS over HTTPS server"
date: 2023-01-01T10:40:03-05:00
draft: false
---

DNS over HTTPS is a protocol that encrypts your DNS (the protocol that converts domain names to ip addresses) requests which can help to mitigate ISP spying and bypass basic censorship. While this protocol has its benefits it can also rely on using centralized servers run by large companies such as Google and Cloudflare which you may not trust completely. In this tutorial, I will show you how to set up your own DNS over HTTPS server.

<!--more-->

## Pre-requisites

1. Cloud Server running Ubuntu 22.04 or newer with SSH access (This can be accomplished with other Linux distributions but some of the commands will vary)
2. Domain Name pointing to your server

## Getting Started

1. Update repositories and install Unbound
```
sudo apt update
sudo apt install unbound
```

Unbound is a recursive DNS server that resolves domains by contacting nameservers in a sequence of steps starting from the root servers. Once resolved Unbound will keep domains in a cache to speed up queries in the future for the same domains. Unbound will help us avoid sending requests to a public DNS server such as Google ``8.8.8.8`` or Cloudflare ``1.1.1.1``.

2. Create a config file for Unbound
```
# Create a backup for the original file
sudo cp /etc/unbound/unbound.conf /etc/unbound/unbound.conf.bak

# Creating our own config file
sudo nano /etc/unbound/unbound.conf

server:
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # Can be set to yes if your server has an ipv6 address
    do-ip6: no

    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1
    so-rcvbuf: 1m
```

You can refer to this {{<underline "https://nlnetlabs.nl/documentation/unbound/unbound.conf/" source>}} for all of the options available when configuring Unbound.

3. Start the Unbound service and test if it works
```
sudo service unbound restart
dig @127.0.0.1 -p 5353 +short
dig @127.0.0.1 -p 5353 +short google.com
```
Your output of the dig command should look something like this
```
g.root-servers.net.
b.root-servers.net.
h.root-servers.net.
d.root-servers.net.
l.root-servers.net.
f.root-servers.net.
e.root-servers.net.
i.root-servers.net.
a.root-servers.net.
m.root-servers.net.
c.root-servers.net.
j.root-servers.net.
k.root-servers.net.

142.251.215.238
```

The ip address of the second command will differ from mine since Google has servers all around the world.

4. Install dnsdist
```
sudo apt install dnsdist
```
dnsdist is a multi-purpose DNS load balancer. In this case, we will be using it to convert incoming DNS over HTTP requests to the standard DNS protocol using UDP.

5. Configure dnsdist
```
sudo nano /etc/dnsdist/dnsdist.conf

addDOHLocal("127.0.0.1:8053", nil, nil, "/dns-query", { reusePort=true })
newServer("127.0.0.1:5353")
```

Note ``/dns-query`` can be replaced with any custom path

6. Start the dnsdist service
```
sudo service dnsdist restart
```

## Setting up HTTPS

With our recursive DNS server and DNS over HTTP listener up and running we will want to now set up HTTPS on our server.
I recommenced using Caddy. I have a basic tutorial linked {{<underline "/hosting-a-website-with-caddy" here>}}.

Edit your CaddyFile with the reverse proxy directive it should look something like this.
```
yourdomainhere.com {
        reverse_proxy 127.0.0.1:8053
}
```

Once you have done that you can run ``sudo service caddy restart`` to start the proper service.

Caddy will automatically fetch an HTTPS certificate for you as long as your domain is pointed to your server's ip address.

## Configuring Clients

With a DNS over HTTPS server running we will now want to set up a client.

Depending on your OS there are many ways to go about this. I will list some DNS over HTTPS clients below.

- iOS/macOS Native - {{<underline "https://dns.notjakob.com/tool.html" "Profile Generator">}} - {{<underline "https://apps.apple.com/us/app/dnsecure/id1533413232?platform=ipad" "Configure through app">}}
- Windows - {{<underline "https://yogadns.com/" YogaDNS>}}
- Android - {{<underline "https://play.google.com/store/apps/details?id=com.controld.app" ControlD>}} - The latest versions of Android have native support in WiFi settings
- {{<underline "https://support.mozilla.org/en-US/kb/firefox-dns-over-https" Firefox>}}
- {{<underline "https://www.ghacks.net/2021/10/23/how-to-enable-dns-over-https-secure-dns-in-chrome-brave-edge-firefox-and-other-browsers/" "Chrome and Chrome based (Brave, Edge, Opera, etc.)">}}

The exact process will differ but they all involve adding your custom DNS over HTTPS URL which would be ``https://yourdomainnamehere.com/dns-query``

## Testing

Once DNS over HTTPS is configured on your device you will want to test it. The first step to take here is to simply browse the internet and see if everything is working correctly. With the exception of Firefox websites should not load correctly if you have misconfigured something.

If websites are loading properly the next step is to check if your device is using your custom DNS over HTTPS server. The easiest way to accomplish this is to go to the URL ``test.nextdns.io``.

The output should look something like this
```
{
	"status": "unconfigured",
	"resolver": "your server ip",
	"srcIP": "your ip",
	"server": "vultr-atl-1"
}
```

The ip address next to ``resolver`` should be the actual ip address of your server. If it is congratulations you are successfully using your DNS over HTTPS server to encrypt your DNS requests.

## Conclusion

Congratulations you have successfully set up your own DNS over HTTPS server. I hope this tutorial has helped you.

