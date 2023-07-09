# Guide to setting up public keys for SSH
Uploaded on July 8th, 2023

<hr/>

I have a QEMU virtual machine running Ubuntu Server 22.04 LTS on IP 192.168.122.91. I can SSH into it fine from my Arch Linux install, but its not 100% secure because it uses password authentication.

I'm going to walk through the steps to generate an SSH key, trust it on the virtual machine, and force public key authentication.

## Slight disclaimer:
If this guide makes your server unable to be SSHed into or somehow blew up your computer, you were the one who followed this. I may have written the guide, but you followed it. Run with caution, review the commands before running, and be careful with sudo and root.

## Generating a key
First, we need a key.
```bash
[jasedxyz@jasetop ~]$ ssh-keygen

Generating public/private rsa key pair.  
Enter file in which to save the key (/home/jasedxyz/.ssh/id_rsa):
# hit enter/return for the default location (i recommend this for simplicity)
Enter passphrase (empty for no passphrase):
Enter same passphrase again:

Your identification has been saved in /home/jasedxyz/.ssh/id_rsa  
Your public key has been saved in /home/jasedxyz/.ssh/id_rsa.pub  
The key fingerprint is:  
SHA256:LdlIENQ9kvsFBSAA78CVYT2yQjmM30GL6V7mO8O/nUw jasedxyz@jasetop  
The key's randomart image is:  
+---[RSA 3072]----+  =
|    randomart    |
+----[SHA256]-----+
=
```

Okay, our key is in `~/.ssh/id_rsa` and `id_rsa.pub`.

## Put key in GitHub
Now, you can upload your **PUBLIC** key anywhere, but I'm going to put them in GitHub because of the *fancy* commands you can use to import them on Ubuntu

Sign into https://github.com/ , click on your profile, and go into settings. Click on "`SSH and GPG keys`"

Now, back on my computer, type
```bash
[jasedxyz@jasetop ~]$ cat ~/.ssh/id_rsa.pub
# if you changed the path, replace '~/.ssh/id_rsa.pub' with your path.
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpUJ3CAM93DZAwkH6eEmENfF5SJ6/cwQbo6bMQoQo+AqadHW1Cx7ZFayXV5Z80AY0qImAZqYFO1V8aXD7HVoFxgY+QiiTPInfXNptJ7mZ9o6JcA20HC6sJC4PxU8CZsV0NNoQ3+ksEuF/4eNecSU4Eu+gyFpIzkmnhfIQPdSf5LaifHcXOsxqPIsWfxbh/o2l7GmHbRtgfoSz0zud3heQ3oE7jZ56wJux4N+W+Ib4HCuyGDSDSPWqj2IrjJfLtjIWXNJ1o0MhmbfPH6BIp2wZ2Amk+RC7RgsAWF3CU5p3DWyfmnkT+A6fc3pCbIuSp1QTpADgBcYknScwuTFUDCIDSXaXrBwSRTgQ5TT+rQskYQq/N3DBkq3DutfsFv2HPhhxFt1vKhpn4+kDinGMoParfDxTzCZrFILlt6p/pnez8KCpKHMVdgkS1LK9kCqn8oDM9VAchwTNwTY3CrjL0fJcPULT6s2JND5IXgipu1QGlu8ph+drtvWBmD9jcFhCoquU= jasedxyz@jasetop
```
That's our public key, copy it, and go back to GitHub.
Lets hit `New SSH key`. Type in a title (whatever you want), and in the key box, paste in your public key, and then `Add SSH key`.

## Import SSH key
Now, on the server, lets import our newly generated key.
```bash
jasedxyz@qemuserver:~$ ssh-import-id-gh Jased-0001
# replace with your github account obviously
```
Now, in `~/.ssh/authorized_keys`, you should be able to see your key. Double check it or don't.

## Disable password authentication 
To start using our freshly-picked SSH key, we need to change some settings.

```bash
jasedxyz@qemuserver:~$ sudo nvim /etc/ssh/sshd_config
# or your editor of choice
```

Remove the `#` in front of line `38` and `41`
Do the same for line 57, and set it to `no`
Save and exit.

Reload, and exit the SSH session. 
```bash
jasedxyz@qemuserver:~$ sudo systemctl reload sshd
jasedxyz@qemuserver:~$ exit
logout  
Connection to 192.168.122.91 closed.
[jasedxyz@jasetop ~]$ 
```

## Did it work?
Lets reconnect and see if it worked!

```bash
[jasedxyz@jasetop ~]$ ssh jasedxyz@192.168.122.91  
Enter passphrase for key '/home/jasedxyz/.ssh/id_rsa':  
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)
...
jasedxyz@qemuserver:~$
```
It works!
Now my (and if you followed along your) server is protected against brute forcing, phishing, all that bad stuff.
