---
layout: post
title:  "Getting Started With Disco MapReduce"
date:   2017-02-17 13:00:17 -0500
categories: disco tutorial
---
A while back, I wanted to learn how to use Hadoop. I prefer to self-host my stuff as it's free (as in pizza), and free (as in speech).

I almost immediately ran into issues getting it set up. First off, it turns out Hadoop uses Java, which I hate. It also has insane memory requirements, which my homelab couldn't handle. To top it off, the installation docs were too confusing for something that I would use for 10 minutes and then never touch again.

Then I found [Disco](http://discoproject.org).

Unlike Hadoop, Disco has libraries for use in Python, which I find slightly more acceptable than Java. It also has an extremely small footprint. I was able to run a testing worker with 256mb of RAM, 1 CPU, and 1 GB of hard drive space (including all of its dependencies).

The one remaining problem, is that following the installation instructions for Disco has probably been one of the most frustrating pieces of software I've ever had to install. You know you've spent too much time trying when you're reading a 5 year old GitHub issue, tracking down an Erlang issue related to a cross-language pipe library, all just by following the "Getting Started" guide. Of course, [there's an XKCD for that](https://xkcd.com/1742/).

This guide is written as a fool-proof guide for anyone else who wants to get up and running with Disco with minimal hassle.

## 1. Prerequisites
There are several ways of accomplishing this. I personally used my Proxmox cluster split out over 6 different machines, all behind a private subnet with a firewall. You can do whatever as long as you meet the following requirements.

- At least one machine (preferably more, dedicated boxes like docker containers or VMs)
- CentOS 7 (anything else and you are on your own)
- Ansible server (preferred, as I have playbooks in a Gist link below, but this is not required)
- A configurable DNS server in your network (if you think you can cheat with the HOSTS file, you are in for a world of hurt)
- A free afternoon (Seriously, this could take a while)

## 2. Hello bash my old friend
Go ahead and turn your machines on or provision your VMs. You can do one, or two, (or twenty, as I did). Make sure all of them are running CentOS 7. The minimum specs I got away with was 1 CPU, 512mb of RAM, and 1GB of hard drive space. If you are creating multiple "workers", each one only needs 256mb of RAM. 512mb is only needed for the "master" node. Of course, if you intend on running bigger datasets than me, you will need to increase this number.

For each box you create, you will need to create a corresponding entry on your DNS server. How to do this is beyond the scope of this article and depends on what software you choose to use for this. I was able to just add the entries to my pfSense router using the configuration page. If your router can't do this, take a look at a BIND server.

Now boot all of the boxes and run the following command.

{% highlight bash %}
$> yum install openssh-server openssh-client openssl-devel -y
$> ssh-keygen -N '' -f ~/.ssh/id_dsa
$> service sshd restart
{% endhighlight %}

This sets everything up for SSH connections between the master node and the workers (This step is also required if you wish to use Ansible, as described below).

## 3. Installing Erlang on the first try (sometimes)

Now we have to install Erlang. This is a pain to set up on CentOS for some reason. All the tutorials I found were either 5 years old or not for my distro. The solution I found was Kerl. Type the following commands into the terminals on all the machines.

{% highlight bash %}
$> curl -O https://raw.githubusercontent.com/kerl/kerl/master/kerl
$> chmod a+x kerl
{% endhighlight %}

This gets Kerl all set up so we can install Erlang. Before we clone and build Erlang though, we need to install some additional dependencies.

{% highlight bash %}
$> yum install tar gcc make perl ncurses-devel git -y
{% endhighlight %}

Now we can install Kerl. Run the following commands.

{% highlight bash %}
$> ./kerl build 17.5 17.5
$> ./kerl install 17.5 erlang/17_5/
$> . /root/erlang/17_5/activate
$> erl -version
{% endhighlight %}

If that works without any errors, you are now good to go. Now we can install Disco.

## 4. Time for the (actual) installation

{% highlight bash %}
$> git clone git://github.com/discoproject/disco.git
$> cd disco
{% endhighlight %}

At this point, what you need to do next depends on the type of installation you are performing. If you are installing the master node (or doing a single-machine install), run this:

{% highlight bash %}
$> make install
{% endhighlight %}

Otherwise, if you are creating a worker node that doesn't need the web GUI, run this instead:

{% highlight bash %}
$> make install-node
{% endhighlight %}

Now, on both the master and worker nodes, set up the python libraries.

{% highlight bash %}
$> cd lib
$> python setup.py install --user
$> cd ..
{% endhighlight %}

## 5. Developing trust in your cluster (using math and crypto-stuff)

The last part is to set up the Erlang cookie and SSH keys. On the master server, run the disco service and stop it by running the following.

{% highlight bash %}
$> bin/disco start
// Wait a few seconds...
$> bin/disco stop
{% endhighlight %}

Now run the following on your master node. Replace NODE with the DNS-resolvable name of each of your worker boxes.

{% highlight bash %}
$> ssh-copy-id localhost
$> scp ~/.erlang.cookie localhost:
// Now run these two commands for every worker box you have,
// replacing NODE with the DNS resolvable name.
$> ssh-copy-id NODE
$> scp ~/.erlang.cookie NODE:
{% endhighlight %}

Finally, run the following on all the nodes:

{% highlight bash %}
$> bin/disco nodaemon
{% endhighlight %}

At this point, you should be able to follow the instructions in the documentation on [adding all of your nodes to the web GUI](http://disco.readthedocs.io/en/develop/start/install.html#add-nodes-to-disco) and running the [test wordcount program](http://disco.readthedocs.io/en/develop/start/install.html#test-the-system).
