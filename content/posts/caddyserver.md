---
title: "Hosting a website with Caddy"
date: 2022-12-16T09:50:04-05:00
draft: false
---

Having a personal website can be a great way in putting yourself out there on the internet. In this tutorial, I will show you how to get a basic web server up and running with a valid HTTPS certificate.

<!--more-->

## Why do you need HTTPS?

Before we start this tutorial, It is important to mention why we need HTTPS in the first place even for websites that might not collect any personal information. Below I have some reasons why.

- Modern web browsers including Chrome and Firefox will mark your site as “Not Secure” without HTTPS
- Search engines including Google will use HTTPS and one of their ranking factors
- The latest version of the HTTP protocol including HTTP/2 and HTTP/3 both require HTTPS
- Some ISPs will hijack consumer connections to insert their ads inside your website

## What is Caddy?

Caddy is a modern web server written in the Go language. Its main claim to fame is the ease of setting up a web server with HTTPS.

## Getting Started

To begin this tutorial you will need a Linux server with SSH access running Ubuntu, Debian or any other Debian-based derivative (such as Raspbian for Raspberry PIs), if you do not have one you can easily order a cloud server from DigitalOcean for as little as $4 a month.

In addition, you will also want to have a domain name.

To get started you will need to run the following commands.
The latest version of the following commands can be found {{< underline "https://caddyserver.com/docs/install#debian-ubuntu-raspbian" here >}}

1. Install required dependencies for Caddy
```
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
```
2. Install the signing key for Caddy
```
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
```
3. Add the Caddy repo to your system
```
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
```
4. Update the apt repos on your system and install Caddy
```
sudo apt update
sudo apt install caddy
```

Once you have run these commands successfully you should be able to open a web browser and type in the IP address of your server in the address bar (use 127.0.0.1 or localhost if you installed Caddy on your main system)

You should see something like this:

![caddy default home page](/images/caddyhome.png)

## Turning on HTTPS

Congratulations your website is now up and running now it is time to enable HTTPS on your website and get the lock icon showing in your visitors' browsers.

First you will want to edit the DNS configuration of your domain name to make it point to your server. This process will vary slightly depending on the domain registrar that you have used. All you need to do is add an A and AAAA (may not be applicable of your server does not support ipv6) record that is pointing to your server.

I have attached a screenshot showing the final result in Cloudflare's control panel, again the interface may vary depending on your domain registrar.

![dns control panel for cloudflare](/images/cloudflare-dns.png)

Next you will want to edit the Caddy configuration file using your favorite command line text editor. I recommend using nano for beginners but feel free to use your preferred editor.

To open the Caddy configuration file in nano you will use:
```
sudo nano /etc/caddy/Caddyfile
```

The file contents will look something like this:
```
# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

:80 {
        # Set this path to your site's directory.
        root * /usr/share/caddy

        # Enable the static file server.
        file_server

        # Another common task is to set up a reverse proxy:
        # reverse_proxy localhost:8080

        # Or serve a PHP site through php-fpm:
        # php_fastcgi localhost:9000
}

# Refer to the Caddy docs for more information:
# https://caddyserver.com/docs/caddyfile
```

All you have to do is change ``:80`` to ``yourdomainhere.com``

When you have made that change you will want to run:
```
sudo service caddy stop
sudo service caddy start
```
Once you have done that you should see a lock icon in your browser and the ability to view the web certificate. You won't have to worry about renewing the certificate when it expires as Caddy will do that for you automatically.

![https in microsoft edge](/images/https-example.png)

## Adding your files

Now that your website is fully secured you probably want to add files to your website including HTML, CSS, and/or JavaScript.

You will first want to create a directory to store your files. To do this you will want to run the following commands.

1. Create the directory to store your website files
```
sudo mkdir /var/www/
```
Note: /var/www can be replaced with any directory you want

2. Give ownership of that directory to your user
```
sudo chown -R youruser /var/www/
```
3. Adjust permissions so Caddy can access your directory
```
chmod -R 755 /var/www/
```
4. Edit your Caddy config file to point to the new directory
```
yourdomainhere.com {
        # Change this line here to /var/www or your custom path
        root * /usr/share/caddy

        # Enable the static file server.
        file_server
}
```
Make sure to run ``sudo service caddy stop`` and ``sudo service caddy start`` afterwards

With your directory created you will need an SFTP client to upload your website files to your server. 

To accomplish this you can use {{<underline "http://winscp.net" WinScp>}} for Windows or {{<underline "http://cyberduck.io" Cyberduck>}} for macOS.

Login Screen for WinSCP (use SSH login):

![login for winscp](/images/winscp.png)

Once your logged in just navigate to your custom directory on the right side and the folder containing your local website files on the left side. From there you can simply drag and drop files from your local machine to your remote server.

![winscp files screen](/images/winscp-files.png)

## Conclusion

Congratulations you now have everything you need to run a web server with HTTPS support. 

To learn more about Caddy and all the things you can accomplish with it visit their documentation {{<underline "https://caddyserver.com/docs/" here>}}.
