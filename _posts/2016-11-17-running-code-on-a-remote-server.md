---
layout: post
title: running code on a remote server
description: how to set up a Jupyter notebook server
comments: true
---

In a few of my previous posts I mentioned that an appealing feature of Jupyter notebooks is that you can interact with them on your local machine (say, your laptop in a coffee shop) but have the code itself run on a remote server (say, a big ol' computer in your lab space that never gets turned off). This allows you to offload all the data storage and number crunching to a "workhorse" computer so that your laptop doesn’t get overwhelmed. Yet since all of your interactions take place through a Jupyter notebook in the web browser, it still very much feels like it’s all happening locally. That is, you don’t have to worry about how to send commands to or receive results from the remote machine -- the browser takes care of all of that for you using all the standard tricks of the internet.

In this post I’m going to walk through precisely how to serve Jupyter notebooks from a remote server and how to navigate to said server through your local web browser so that you can free up your personal computer from its computational hardship. While there are a few tutorials out there already, my plan here is to be a bit more comprehensive for those who are newer to Jupyter and coding. Plus, this is all still new enough technology that having another tutorial out on the internet at the very least couldn’t hurt. Anyhow, I apologize if this post feels a bit long, but this is mostly a product of my being extra wordy, just to make sure everything is clear. If you keep the big picture in mind, however, most of the steps are actually quite straightforward.

## Preliminaries

#### Remotely accessing the host computer

The first thing you'll need is an "always on" *host* computer that you can access remotely. Most companies and labs probably have a few of these sitting around (and should if they don’t, lest they be left behind by by the whirlwind of the 21st century), but you can also use services like [Heroku](https://www.heroku.com) to borrow servers from the cloud. For the purposes of this tutorial, and since I’m (unfortunately) not yet too familiar with cloud computing, we’ll assume that your host computer is literally a computer sitting in some company or lab space, and for which you have remote login privileges. And in fact, I'd like to think this covers the 90% use case of scientific projects. The method we’ll be using to login to our host computer from our local coffee shop laptop, by the way, is called `ssh`, which stands for "secure shell" and is a protocol for communication between two computers over a network. To make sure that remote access is enabled, follow [these instructions](http://osxdaily.com/2011/09/30/remote-login-ssh-server-mac-os-x/) for Mac operating systems, and for Ubuntu, follow [these instructions](http://ubuntuhandbook.org/index.php/2016/04/enable-ssh-ubuntu-16-04-lts/). For other operating systems, I'd suggest searching for "enable ssh <your operating system>" on the search engine of your choice. This kind of thing is pretty standard in the grand scheme of things, so it shouldn’t be too much of an issue (although using Windows as a server computer can be a bit tricky). Once you've enabled remote access, your computer should tell you what its domain name/web address is.

Right, so assuming we’ve enabled `ssh`ing into our host machine, we’re now going to `ssh` into it. On our coffee shop laptop we therefore need to open up the terminal application (on Mac or Linux this is already installed [in the Applications/Utilities on a Mac], but for Windows you’ll need to [install a bash shell first](http://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)). Once the terminal starts up you should see a command prompt with your username, maybe some other stuff, and eventually a `$`. After the `$` type:

`ssh myusername@domain.name.of.host.computer.edu`

with `myusername` and `domain.name.of.host.computer.edu` replaced by your username and computer domain name, respectively. For example, I (rkp) might log in to a computer using `ssh rkp@basementcomputer.fancydept.washington.edu`. When you hit enter you'll be asked for your password, which you should type in and hit enter again (for newbies, you won’t see any asterisks in the password field, it will just be blank). If you remembered your username and password correctly, you should now be sshed into the host computer. By the way, at this point, if you are not terribly familiar with the terminal/command line, I would also highly recommend learning a few [command line basics](http://lifehacker.com/5633909/who-needs-a-mouse-learn-to-use-the-command-line-for-almost-anything). Most importantly, for this tutorial it will help to be familiar with the `cd` and `ls` commands, which change directories and list the contents of a directory, respectively, as well as with [tab completion](http://www.howtogeek.com/195207/use-tab-completion-to-type-commands-faster-on-any-operating-system/), which will save you a lot of keystrokes and typing errors. [This tutorial](https://linuxacademy.com/blog/linux/linux-commands-for-beginners-sudo/) also gives a good concise overview of how the `sudo` command works, which is useful when your normal user account doesn’t have sufficient permissions to do something you need to do.

#### Installing anaconda

The final thing we’ll need installed on our host computer is Anaconda, i.e., our good ol’ scientific python distribution. If you have graphical access to the host computer (through screen sharing or just physically plugging into a keyboard and monitor) you can download it from [their website](https://www.continuum.io/downloads). If you only have command line access, however, you'll have to download it through the command line. To do this, first get the url of the command line installer from [their website](https://www.continuum.io/downloads). This is most easiliy done by right clicking the "command line installer" button on said website and selecting "copy link" to copy it into your clipboard. For example, at the time of writing the url Python 3 command line installer for Python 3 is "https://repo.continuum.io/archive/Anaconda3-4.2.0-MacOSX-x86_64.sh". Then run the following in your terminal (assuming you’re still sshed into the host computer):

```
cd /path/to/where/anaconda/should/be/installed
curl -O "https://repo.continuum.io/archive/Anaconda3-4.2.0-MacOSX-x86_64.sh"
```

except replacing the web address with the one you’ve copied from the Anaconda website. This will probably take a few minutes, but when it's completed you'll be returned to the command prompt. As usual, you can use `ls` to check to make sure it got downloaded. Then follow the rest of the instructions on the Anaconda website for installing from the command line.

### Setting up a Jupyter notebook server

Okay, now that we’ve got everything installed on the host computer we can actually set up the Jupyter server. The term "server", by the way, can refer either to the physical computer that accepts remote communication over a network, or to a specific program that accepts said remote communication. In our scenario the host computer is the server in the hardware sense, and the Jupyter server will be a server in the software sense. This confused me a while back so I thought I'd throw it in here. Setting up a Jupyter server that runs on the host computer and allows you to interact with Jupyter from within the web browser on your coffee shop laptop requires four things: (1) creating a web certificate, (2) choosing a password (3) setting up the configuration file with the web certificate and password, and (4) starting the server. These four things are going to happen while we're sshed in to the host computer.

#### Creating a web certificate

A web certificate is a little file attached to a website that a browser can look at it to see if the website is legitimate or not. Since we’re going to be setting up a *secure* server that uses encrypted communication, this is something we're going to need. Luckily, creating one from the command line is pretty simple. The first thing we'll need to do is to `cd` into the directory `/Users/myusername/.jupyter`, which is where all the Jupyter settings live. The `.` at the start of the name indicates that its hidden, so if you’re in the directory `/Users/myusername`, you’ll have to use the command `ls -a` to view it (the `-a` flag means to show hidden files also). To check and make sure you’re in the correct directory use the command `pwd`, which prints out the path of your current directory.

Once we're inside the `.jupyter` directory we can create what's called a self-signed certificate, which is kind of like the minimum stamp of legitimacy for a website. (Next time we'll talk about how to use a more proper certificate signed by a trusted authority.) Luckily, this is quite easy to do. All that’s necessary to make one is to run the command:

```openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mykey.key -out mycert.pem```

This makes a certificate valid for 365 days and stores it in the file `mycert.pem`. It also creates a key file called `mykey.key`, which contains a corresponding public key for the certificate, which is used to encrypt communication between the host computer and your coffee shop laptop. You can name these files whatever you want, but `mycert.pem` and `mykey.key` will be just fine for our purposes. If you ran the previous command inside the `.jupyter` directory, the paths to these files will be `/Users/myusername/.jupyter/mycert.pem` and `/Users/myusername/.jupyter/mykey.key`, respectively. Remember these or copy them somewhere, since we’ll need them in just a little bit.

#### Choosing a password

The next thing we need to do is to create a password, so that not just anyone with the correct web address can access our Jupyter notebook server. To do this securely we'll use IPython (interactive Python, which should have been installed when you installed Anaconda), which you can start with the command

`ipython`

Once IPython has started up, type 

`from notebook.auth import passwd`

and hit enter. Then type

`passwd()`

and hit enter. This will ask you to type in a password (again, you won't see any asterisks) and then verify it, and it will spit out a string that is a *hashed* (encrypted, more or less) version of your password, which is much safer to carry around than your password itself. This is because although the same password will always yield the same hash, it's next to impossible to figure out the original password from the hashed version. The next thing we’ll do is copy the hashed password, which should look something like `'sha1:22bc1130c7e5:8281c216cce0d14b896eaf613e62738ee572b813'` and save it somewhere, since we'll have to stick in the configuration file in the next section. Then exit IPython using

`exit()`

#### Setting up the configuration file

To set up the configuration file, first make sure you're in `/Users/myusername/.jupyter` and then type `ls` to see if a file exists called `jupyter_notebook_config.py`. If it exists, great. If not, create one using the command:

`jupyter notebook --generate-config`.

Then use `ls` again to make sure it was created. 

Our goal now is to edit a few of the settings in this configuration file to tell jupyter to act like a server and accept incoming requests over the internet. To do this, make sure you're in the `.jupyter` directory and then open the config file using:

`nano jupyter_notebook_config.py`

In this command, `nano` is just the name of a very simple text editor that runs in the terminal window. If you’re more familiar with another text editor such as `vi`, feel free to use it (what's important is that we use a *plain text* editor -- don't try to use Microsoft Word!). Our goal is to change a few lines in the config file, and `nano` is the easiest text editor to use. To save the file use CTRL + O and to exit and return to the command prompt use CTRL + X. 

Anyhow, once you’ve opened the file if you take a look around you'll see that it starts with some introductory messages, which are all in the form of comments (i.e., the nonempty lines all start with "#"), and then a myriad of options which you can set by "uncommenting" them, that is by removing the "# " from before the line and then setting the relevant variable to the value you want it to be. 

We now need to set the paths to the certificate and key file, the notebook IP, the notebook password, the open browser flag, and the notebook port. As an example, to set the path to the certificate file, scroll down until you see the commented line 

`# c.NotebookApp.certfile = u''`

And change it to 
`c.NotebookApp.certfile = u'/Users/myusername/.jupyter/mycert.pem'`

replacing `myusername` with your actual username. Note that if for some reason the commented version of the line doesn’t exist, just go ahead and add in the uncommented version yourself. The `u` by the way, stands for unicode (a modern encoding standard). Next, find the line

`# c.NotebookApp.keyfile = u''`

and change it to

`c.NotebookApp.keyfile = u'/Users/myusername/.jupyter/mykey.key'`.

Again--and the same holds true for the rest of this section--if the commented version of a line is missing, don’t worry about it and just add in the uncommented version. 

Next, change the line 

`# c.NotebookApp.ip = ''`

to 

`c.NotebookApp.ip = '*'`

This will allow us to access the notebook by navigating to the *host* computer’s web address. To set your password, change the line

`# c.NotebookApp.password = u''`

to 

`c.NotebookApp.password = u'sha1:22bc1130c7e5:8281c216cce0d14b896eaf613e62738ee572b813'`

but replacing this with the hashed version of *your* password that you created in the previous section. 

Next, change

`# c.NotebookApp.open_browser = True`

to 

`c.NotebookApp.open_browser = False`

which will prevent the browser from opening on the host computer when we start the notebook server.

Finally, change

`# c.NotebookApp.port = 8888`

to 

`c.NotebookApp.port = 8888`

i.e., just uncomment this line. This bit tells Jupyter what port to run the notebook server out of, which is kind of like an extension if you think of the web address as a telephone number. If you know about ports 'n' stuff already, feel free to choose whatever port you want, but `8888` is the default.

Whew, that was a lot of settings, but hopefully it wasn’t actually. This is really just a slightly fancy way of turning an application's knobs and dials so that it does what we need it to do. Anyhow, in the end you should have the following uncommented lines somewhere in the configuration file:

`c.NotebookApp.certfile = u'/Users/myusername/.jupyter/mycert.pem'`

`c.NotebookApp.keyfile = u'/Users/myusername/.jupyter/mykey.key'`

`c.NotebookApp.ip = '*'`

`c.NotebookApp.password = u'sha1:22bc1130c7e5:8281c216cce0d14b896eaf613e62738ee572b813'`

`c.NotebookApp.open_browser = False`

`c.NotebookApp.port = 8888`

When all of this is done, save the file and exit `nano` to return to the `.jupyter` directory. That concludes the setting-the-configuration-file section. Onto the final steps.

#### Running the server

Before actually starting the server we need to `cd` into the directory where all our notebook files live. I generally run the server out of a `repositories` directory, which contains all my projects, since I like to work on all of them using the same notebook server, but you can serve a single project’s directory if you want, or any directory at all, in fact. This bit is up to you. Once you’ve `cd`ed into the directory you want to serve, starting the server is simple. Just type

`jupyter notebook`

and hit enter. You should be greeted by a response that looks something like this:

```
[I 21:31:24.082 NotebookApp] Serving notebooks from local directory: /Users/myusername/Dropbox/Repositories
[I 21:31:24.082 NotebookApp] 0 active kernels
[I 21:31:24.082 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/
[I 21:31:24.082 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

This is good. It means the server is running. Just to make sure, open up a web browser on your coffee shop laptop and navigate to the address of your host computer followed by the port number. The key part to remember here is to preface the address with "https://" instead of just "http://". This is because we’re using a secure server. If you use "http://" it won’t work. For example, if my host’s address were "basementcomputer.fancydept.washington.edu" and the notebook server was running out of port 8888 (as we configured it to do), in my web browser on my local machine (my coffee shop laptop), I would navigate to the address: "https://basementcomputer.fancydept.washington.edu:8888".

If everything has gone according to plan so far you’ll probably be greeted with a page telling you that someone on the other end is using a sketchy web certificate, and you should be careful when visiting the site. Now, normally one should avoid sites that trigger this warning message, but since this is actually the server we’ve just configured, there’s nothing to worry about, so press whatever buttons are necessary for your browser to let you through. If you do this, you should be taken to a login page and asked to enter your password, which is just the password you originally entered in IPython to generate its hash. And voila, you’re in!

Feel free to make a notebook or two and run a bit of code just to make sure everything works. If everything is working, your code cells should execute and it should feel just like the notebook is running as if it lived on your local computer.

#### Keeping the server running when we’re no longer sshed into the host computer

Now, the last thing is that it's a bit annoying to have to leave the ssh connection and the terminal window on your coffee shop laptop open in order to keep the Jupyter server running on the host computer. It would be much nicer if you could ssh into the host, start the Jupyter server, disconnect from the host, and then have all of your subsequent usage of the Jupyter server be strictly through the web browser. Luckily this is possible using the `nohup` command and the `&` operator.

First, if the server is still running in the terminal window, press CTRL + C to stop it. Next, assuming you're still in the directory you want to the server to run out of, type

`nohup jupyter notebook &`

and hit enter. This will start the server running as a background process (that's what the `&` does) and write all the messages that would otherwise be written to the terminal to a file called "nohup.out" instead (that's what the `nohup` does). After you run this command, the terminal should output something like the following:

```
[1] 26479
myusername:/path/where/jupyter/server/is/running$ appending output to nohup.out
```

The first line indicates the process id of the jupyter server, which you can use to reference it later. In this example it's 26479 but will probably be something different every time you run it. If you press enter one more time, you’ll get taken back to the command line prompt, indicated by the reappearance of the `$` sign. 

Now you can use `exit` to close the ssh session into the host computer, and then you can close the terminal application on your coffee shop laptop. When you log in to the server from your web browser on your coffee shop laptop again (https://basementcomputer.fancydept.washington.edu:8888 in case you forgot) everything should be just as it was before, and you should be able to make notebooks and execute code cells.

If you need to shut down the Jupyter server, you can do this by sshing back into the host computer using the terminal application on your coffee shop laptop, and typing the command 

`kill 26479`

and pressing enter, where 26479 is replaced with whatever process id has been attached to your instance of the server. If you don't remember the process id of the server you can run the command

`ps aux | grep jupyter`

which will output all processes running that included the term `jupyter` in them. This means you’ll probably get an output like 

```
myusername   26499   0.0  0.0  2435864    792 s000  S+   10:01PM   0:00.00 grep jupyter
myusername   26479   0.0  0.5  2491488  39884 s000  S    10:01PM   0:00.65 /Users/myusername/anaconda/bin/python /Users/myusername/anaconda/bin/jupyter-notebook
```

The first process listed here corresponds to the process-listing process (the thing we just ran), and the second to the Jupyter server of interest. The second column contains the process id (which as you’ll note here, is 26479). Note that sometimes these two processes will be listed in the opposite order.

### Wrapping up

Alright, I suppose that given the length of the preceding sections this might seem like a bit of an involved process, especially if you’re new to the command line and sshing, but I hope that it at least feels like a straightforward and logical series of steps. However, I also hope you can see why taking the time to set up a Jupyter server on a remote, "always on" computer would be worth it.

The most substantial benefit of running a Jupyter server on a remote host is that you can now run pretty heavy duty code without relying on the processing power of your coffee shop laptop and without compromising its battery life. This is also particularly nice when your code interacts with large datasets that are much more easily stored on a server computer than on your local machine. A further benefit, however, and one that is quite convenient, is that once the Jupyter server is running, the only thing you need to access it is a web browser. Therefore, if you find yourself using a computer at the library, say, or using your friend’s computer, all you have to do is navigate to the web address of the Jupyter server and you can do everything you were doing before without having to worry about downloading or installing anything at all.

One other thing I like to do, so that it feels even more directly as if all the processing power and data storage of the host computer is simply an invisible extension of my coffee shop laptop is to sync my codebase between the two computers using Dropbox. This way I can make all my edits on my coffee shop laptop using a feature-rich IDE such as PyCharm, and whenever I save a file, the corresponding file on the host computer will be immediately updated. This isn’t that necessary when simply running a Jupyter cell or two, but since we’ve already [discussed](http://rkp8000.github.io/succinctly_in_silico/the-Jupyter-Notebook-as-an-interface-to-a-larger-code-base/) that the majority of your codebase should not actually be in Jupyter cells, but rather simply interfaced by them, it can be nice to use a real IDE to edit your codebase instead of Jupyter’s built-in text editor.

### P.S. - a note about web certificates

The only slightly sketchy thing we’ve done to make this all work is to use a self-signed web certificate. This is the thing that caused our web browser to raise a warning about entering an untrusted site, remember? While this shouldn't pose a problem for most intents and purposes, it is indeed better practice, if you have the time, to use a certificate that will be trusted by a web browser, that is, a certificate signed by a standard certificate authority. Further, having a trusted web certificate will allow you to access your server from mobile devices, whereas doing this with a self-signed certificate has proved rather challenging in my experience.

In the past, getting a valid non-self-signed certificate was quite a hassle and could even cost a little bit of money. Over the last year or so, however, a service called `letsencrypt` has emerged that allows any user to automatically obtain a valid, trusted web certificate, which is perfect for running a Jupyter server. This process, though straightforward, is a bit more involved and took me a while to figure out how to do correctly given the dearth of tutorials currently available on the internet. In the [next post](https://rkp8000.github.io/succinctly_in_silico/2017/01/31/creating-a-letsencrypt-certificate.html), I'll discuss how to set up a letsencrypt certificate in detail.

Until next time--
