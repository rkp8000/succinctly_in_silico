---
layout: post
title: "creating a letsencrypt certificate"
description: "a more official way to generate a web certificate for a remote server"
comments: true
---

Last time we talked through how to serve your Jupyter notebooks from a remote server. To do this we had to generate web certificate, which was a small file vouching for the validity of the server on the remote machine. In the previous post we used a self-signed certificate, which is fine in some contexts, but is not trusted by default by most browsers. Further, self-signed certificates will cause problems when connecting to your server through mobile devices; for example, while you might be able to view your notebook through a mobile device, your mobile browser will probably not trust the self-signed certificate enough to let you connect to the kernel, which you need to do to actually run your code.

This time we’re going to walk through setting up a more complicated but more official certificate, one signed by a certified authority and automatically trusted by most browsers. While generating such a certificate was rather challenging in the past and usually required a monetary fee, the recently created service [LetsEncrypt](https://letsencrypt.org) now lets one generate certificates automatically at no charge. That said, the service and the API are still relatively new to the world of people running their own servers, so getting this to work properly on your remote machine requires a bit more command-line know-how than some of our previous tutorials. However, since it took me quite a while to figure out how to get it working, I figured adding another tutorial online at the very least wouldn’t hurt.

Note that if you haven’t set up a Jupyter server yet you should check out my [previous post](https://rkp8000.github.io/succinctly_in_silico/2016/11/17/running-code-on-a-remote-server.html). What we'll do now is replace the self-signed-certificate-generating section detailed in that post with a more involved but more official way of generating a web certificate.

## installing certbot: an automated tool for generating web certificates

To create a LetsEncrypt certificate you’ll need an ACME client tool. Instead of getting into the details of what this is, I’m just going to recommend *certbot*, which is the default client with the most documentation regarding our purposes surrounding it. Installing it is fairly straightforward on a Linux machine, but a little involved on a Mac. If your remote server is a Windows machine, things will be substantially more complicated, but a good place to start is [here](https://letsencrypt.org/docs/client-options/).

#### installing certbot on a Linux machine

Installing `certbot` on Linux is done most easily by simply cloning (copying) the `certbot` repository from where it lives on GitHub onto your computer. You can do this using the program `git`, which you almost certainly have installed if you’re running Linux. Once you’ve [sshed](https://rkp8000.github.io/succinctly_in_silico/2016/11/17/running-code-on-a-remote-server.html) into the remote machine, `cd` into the directory `/opt` using

```cd /opt```

which is the default location for installing third-party software. Then download `certbot` using 

```sudo git clone https://github.com/certbot/certbot```

and enter your password. If you run the command `ls` you should see that there is a new directory inside `/opt` called `certbot`. 

#### installing certbot on a Mac

Installing `certbot` on a Mac is a bit trickier since you’ll need to install it using the program `brew`, a command-line package manager that does not come preinstalled on Mac. If you have `brew` already, great. Otherwise you can install it by following the instructions [here](brewsite.com). A note about using `brew` for these purposes is that the commands we’ll be running can take several minutes sometimes, even on a fast computer, so you might have to be patient. Once you have `brew` installed, make sure you have permissions for `/usr/local`, using 

```
sudo chown my_username /usr/local
```

Once your permissions are all set up, make sure `brew` is up-to-date using

```brew upgrade```

followed by 

```brew update```

(note that if you still run into problems with `brew` permissions later, just make sure that you `sudo chown my_username` whichever directory `Cellar` is located in).

Finally, use `brew` to install `cerbot` by running:

```brew install certbot```

. Getting `brew` to work correctly can be quite tricky at times, especially since it doesn’t let you use `sudo`. Fixing permissions and upgrading/updating were the main things I had to do to get it to work correctly on my Mac, but if you have further problems with `brew`, checkout their [troubleshooting documentation](https://github.com/Homebrew/brew/blob/master/docs/Troubleshooting.md#troubleshooting). If that doesn’t work, I’d recommend the usual route of copying what you think are relevant portions of the error message into a google search to try and figure out whether others have had your same problem in the past and how they’ve solved it. 

## making a new web certificate

Next we need to use `certbot` to make a new certificate. Since we’ll put the certificate files in the directory `/usr/local/letsencrypt` (regardless of whether we’re using a Mac or Linux), make sure to give yourself permissions to `/usr/local` (if you haven’t already) using the command:

```
sudo chown my_username /usr/local
```

#### making a new certificate on Linux

If you’re using Linux, next move into the directory in which you installed `certbot` using

```cd /opt/certbot```

Then create a new certificate using the command:

```sudo ./certbot-auto certonly --standalone --config-dir /usr/local/etc/letsencrypt --logs-dir /usr/local/var/log/letsencrypt --work-dir /usr/local/var/lib/letsencrypt```

. This runs the certbot program (`certbot-auto`), tells it to only make a certificate (`certonly`) with no frills attached (`--standalone`), and it tells it where to put the config, log, and work directory (`--config-dir`, `--logs-dir`, and `--workdir`). If all has gone well you’ll be guided through the procedure for creating a new web certificate, which will involve entering your domain name and some credentials like your email (to figure out your domain name on a Linux machine see [here](https://www.cyberciti.biz/faq/linux-find-domain-hostname/). Note: if you get an error message telling you port 80 or port 443 is in use, that means you probably already have another webserver connected to those ports, which you’ll need to temporarily disable while you create the certificate (this might also happen if you forget the `sudo` part of the command). If the certificate creation is successful, you should get a message that looks something like this:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /usr/local/etc/letsencrypt/live/my.domain.name.edu/fullchain.pem.
   Your cert will expire on 2017-04-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

#### making a new web certificate on Mac

If you’re using a Mac and you installed `certbot` through `brew`, then you can make a new certificate by runring the following command, regardless of what directory you’re in:

```
sudo certbot certonly --standalone --config-dir /usr/local/etc/letsencrypt --logs-dir /usr/local/var/log/letsencrypt --work-dir /usr/local/var/lib/letsencrypt
```

You'll then get walked through the same certificate creation process as described in the previous section. (Note that the domain name for a Mac will be listed as the remote login address under **System Preferences** -> **Sharing**.)

## adding the web certificate path to our jupyter notebook config

The most important bit of information we need to know is the path to the directory where the certificate has been saved. If you’ve followed all the instructions so far and everything has worked out, this directory should be `/usr/local/etc/letsencrypt/live/my.domain.name.edu`.

Note that by default, you probably won’t have much luck if you try to `cd` into this directory, since you don’t have permissions. To fix this, run the following commands:

```
sudo chown my_username /usr/local/etc/letsencrypt/live
```

and 

```
sudo chown my_username /usr/local/etc/letsencrypt/archive
```

Then you should be able to poke around in the directories and see what’s there. The `archive` directory, by the way, is where the certificate files are actually stored, with the `live` directory simply containing symlinks, or "shortcuts" to the most recently created files. What we want to do now is tell `jupyter` the location of these symlinks so that it can access the certificate files when running the server. 

Specifically, we need to tell `jupyter` the location of the symlinks to the certificate file itself, as well as the private key (a file used in the encryption process). If all has gone according to plan so far, these will be named 'cert.pem', and 'privkey.pem', and will be located in the directory `/usr/local/etc/letsencrypt/live/my.domain.name.edu`. 

To tell `jupyter` where these are, go to the directory where the config file lives with

```
cd ~/.jupyter
```

and start editing the config file with

```
nano jupyter_notebook_config.py 
```

(see my [previous post](https://rkp8000.github.io/succinctly_in_silico/2016/11/17/running-code-on-a-remote-server.html) for the basics of using `nano`). Now all you need to do is find the lines that start with `c.NotebookApp.certfile` and `c.NotebookApp.keyfile` (and if they are commented with a `#` in front of them, remove the `#` and the adjacent space), and then edit them so that they read

```
c.NotebookApp.certfile = '/usr/local/etc/letsencrypt/live/my.domain.name.edu/cert.pem'
```

and 

```
c.NotebookApp.keyfile = '/usr/local/etc/letsencrypt/live/my.domain.name.edu/privkey.pem'
```

respectively. Save the file and exit out of `nano` and you should be good to go. Restart the notebook server on the remote machine (while still `ssh`ed into it from your coffee shop laptop) and then navigate to the server address on a web browser on your laptop (remember not to forget the "https"). If all has gone well you should no longer see the warning about an insecure or untrusted certificate. Further, you should be able to log in from a mobile device also and run code just like you would on a laptop. (One final note is that sometimes on a mobile device you will need to view the Desktop version of the Jupyter notebook, e.g., as described [here](https://www.tekrevue.com/tip/request-desktop-site-ios-9/).)

## renewing a web certificate

By default LetsEncrypt certificates expire after 90 days. Therefore before 90 days runs out you'll need to renew it to get another 90 days (you should receive an email a couple weeks before it runs out if you forget). Renewing the certificate, however, is quite easy. Assuming you’re `ssh`ed into the remote computer, just run the command

```
sudo certbot renew
```

if you’re on a Mac, or

```
sudo /opt/certbot/certbot-auto renew
```

if you’re using Linux.

Hopefully this shouldn't be too much of a pain since you only have to do it once every couple of months. If it is, however, you can set up a recurring renewal command using `cron`, a standard program that allows you to schedule recurring tasks, and which comes preinstalled on both Linux and Mac. We won’t go into the details here, but if you’re interested in automating this or other scheduled processes, check out this [tutorial](http://www.unixgeeks.org/security/newbie/unix/cron-1.html) on using `cron`.

Alright, well that’s it for now. As you probably noticed, this process is a bit more complicated than generating a self-signed certificate, and there’s a higher chance of running into problems, but I hope that some of you still find it useful. 

Also note that if you still have problems but are determined to get this working, checkout the `certbot` IRC [here](https://certbot.eff.org/docs/using.html) for additional help.

Until next time--
