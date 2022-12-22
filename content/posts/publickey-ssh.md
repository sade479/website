---
title: "How to configure public key authentication in SSH"
date: 2022-12-21T11:26:28-05:00
draft: false
---

Public key authentication can be a fast and secure way in logging into an SSH server as opposed to using the traditional password-based login. Having a public key authentication setup can also be needed when setting up automated scripts that involve running commands on a remote server. In this tutorial, I will show you how to get started with public key authentication.

<!--more-->

## Pre-requisites

1. Server running SSH
2. SSH client on your local machine

## Getting Started

1. Generate an SSH keypair on the client
```
ssh-keygen -t rsa
```
You will be prompted to enter a filename and a passphrase. You can just press enter for a default file name on the first prompt and setting no passphrase on the second prompt (I recommend setting a passphrase).

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/test/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/test/.ssh/id_rsa
Your public key has been saved in /home/test/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:33PtRZ5EaCRaOzzbPGuAWapGzv4d2+Zdx3C52ZoaZyQ test@test
The key's randomart image is:
+---[RSA 3072]----+
|           o .   |
|          + + .  |
|         . * o . |
|          = B . .|
|       .S+ oE+ooo|
|      + .. ..o+B=|
|       =  ..+o++O|
|      o   . =B.++|
|       ... o++o..|
+----[SHA256]-----+
```

2. Copy the public key to your remote server

### Method 1 - Use ssh-copy-id

```
ssh-copy-id -i .ssh/id_rsa.pub test@'your server ip/domain'
```

The output should look something like this:
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
expr: warning: '^ERROR: ': using '^' as the first character
of a basic regular expression is not portable; it is ignored
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
test@'your server ip/domain' password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'test@'your server ip/domain'"
and check to make sure that only the key(s) you wanted were added.
```

### Method 2 - Manually copy the public key

You will need to use this method if your local system does not have ssh-copy-id (For example you are using OpenSSH on Windows.)

1. Copy the contents of your public key
```
cat .ssh/id_rsa.pub
```
Note you can replace ``cat`` with ``notepad`` in Windows to display the public key

The output should look something like this
```
ssh-rsa AAAAsdfgsdfgsdfgsdfgsfgsdfgsdgsd= `your username`@`your computer`
```
Note the actual public key will be longer this is just an example to show you what it would look like

2. SSH into your server `ssh user@'your server ip/domain'`

3. Use nano or your favorite command line text editor to open the authorized keys file
```
nano .ssh/authorized_keys
```

4. Paste what you have copied before at the end of the file (you may or may not have existing content in that file)

## Testing public key authentication

With your keypair generated and your public key successfully copied to your remote server, you will now want to test it to see if everything went smoothly.

You will just need to SSH as normal
```
ssh 'your user'@'your server ip/domain'
```

If you have completed the previous steps you will be prompted for your passphrase and after entering it you should be automatically logged in.

## (Optional) Disabling password-based authentication

With public key authentication configured you may want to disable password-based authentication for better security.

To do this open `/etc/ssh/sshd_config` in nano or your favorite command line text editor.
Find the line `PasswordAuthentication yes` and change it to `PasswordAuthentication no`.

Once you have done that simply run `sudo service sshd restart`

## Conclusion

Congratulations you have successfully configured public key authentication on your SSH server. I hope this tutorial has helped you.
If you have disabled password-based authentication I highly recommend keeping a backup of your keypair and having an alternate way of accessing your server such as through a KVM (physical or virtual).






