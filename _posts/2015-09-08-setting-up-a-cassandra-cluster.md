---
layout: post
title:  "Setting up a Cassandra 2.1 cluster on AWS Ubuntu 14.04"
date:   2015-09-08 10:00:00 -0500
categories: cassandra aws ubuntu guide
---
# Installation

Launch some new Ubuntu 14.04 instance on AWS and ssh in to each and do the following:

## Install java

{% highlight bash %}
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
{% endhighlight %}

Verify java is installed with:

{% highlight bash %}
java -version
{% endhighlight %}

And then let java setup environment defaults:

{% highlight bash %}
sudo apt-get install oracle-java8-set-default
{% endhighlight %}

## Install Cassandra

Edit the sources file at **/etc/apt/sources.list** and add:

{% highlight bash %}
deb http://www.apache.org/dist/cassandra/debian 21x main
deb-src http://www.apache.org/dist/cassandra/debian 21x main
{% endhighlight %}

Now we need to add GPG keys so lets run:

{% highlight bash %}
gpg --keyserver pgp.mit.edu --recv-keys F758CE318D77295D
gpg --export --armor F758CE318D77295D | sudo apt-key add -
gpg --keyserver pgp.mit.edu --recv-keys 2B5C1B00
gpg --export --armor 2B5C1B00 | sudo apt-key add -
gpg --keyserver pgp.mit.edu --recv-keys 0353B12C
gpg --export --armor 0353B12C | sudo apt-key add -
{% endhighlight %}

And let's install:

{% highlight bash %}
sudo apt-get update
sudo apt-get install cassandra
{% endhighlight %}

After installation Cassandra will be running and we need to kill the process in order to configure the cluster, so run:

{% highlight bash %}
ps -aux | grep cassandra
{% endhighlight %}

Find the cassandra PID and run:

{% highlight bash %}
sudo kill 3896
{% endhighlight %}

And let's delete the data:

{% highlight bash %}
sudo rm -rf /var/lib/cassandra/*
{% endhighlight %}

## Configure the cluster

Since we're launching a three node cluster we really only need to have one seed node that the other two nodes rely on. For now, ssh into whichever server you want the seed server to be and let's edit **/etc/cassandra/cassandra.yaml**:

{% highlight bash %}
sudo nano /etc/cassandra/cassandra.yaml
{% endhighlight %}

Find the follow configuration settings:

{% highlight yaml %}
cluster_name: "<name of your cluster>"
...
seed_provider: 
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          - seeds: "<private ip of seed server>"
...
listen_address: <private ip of current server>
...
rpc_address: <private ip of current server>
{% endhighlight %}

When completed, let's run Cassandra:

{% highlight bash %}
sudo cassandra &
{% endhighlight %}
*The & is so we run Cassandra in the background.*

Now, repeat these steps with the remaining nodes. When you are finished and you've launched Cassandra on all of the nodes (without errors!) you can run:

{% highlight bash %}
nodetool status
{% endhighlight %}