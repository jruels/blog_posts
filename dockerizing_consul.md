# Dockerizing Consul #

Following this guide to install [Consul](https://www.digitalocean.com/community/tutorials/an-introduction-to-using-consul-a-service-discovery-system-on-ubuntu-14-04)

Built a Dockerfile that builds an Ubuntu 14.04 container and installs wget, unzip and tmux and then does a wget on consul. 


	# vim:set ft=dockerfile:


	FROM ubuntu:14.04
	MAINTAINER Jason Smith <jason@smithss.org>

	ENV APPDIR /app
	WORKDIR $APPDIR

	RUN apt-get update && apt-get install -y \
      unzip \
      tmux \
      wget

	# wget consul
	ENV CONSULVER 0.5.2_linux_amd64
	ENV CONSULURL https://dl.bintray.com/mitchellh/consul/$CONSULVER.zip
	RUN wget $CONSULURL -O consul-$CONSULVER.zip && \
    unzip consul-$CONSULVER.zip && \
    rm consul-$CONSULVER.zip

	# environment variables
	#ENV PATH ${PATH}:$APPDIR/bin
	#ENV RERUN_MODULES /usr/lib/rerun/modules
	#ENV RERUN_COLOR true

	WORKDIR $APPDIR

	CMD []

##Docker Compose
###Installation
Mac:   
`brew install bash-completion`   

Download docker-compose   

	curl -L https://github.com/docker/compose/releases/download/1.3.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

Download docker-compose bash-completion script:

	curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose --version | awk 'NR==1{print $NF}')/contrib/completion/bash/docker-compose > /usr/local/etc/bash_completion.d/docker-compose
	
Docker-compose yaml file
	
	cs01:
      build: .
      command: tail -f /dev/null
    cs02:
      build: .
      command: tail -f /dev/null
    cs03:
      build: .
      command: tail -f /dev/null
    cc01:
      build: .
      command: tail -f /dev/null
      
      

	
Need to figure out how to add the IP/hostname to /etc/hosts in each container. Doesn't seem to be a good way yet using docker-compose. Maybe figure out a way to do this using Ansible? 


Install tmux

	apt-get tmux 
Use tmux for all the following commands.    

On CS01

	consul agent -server -bootstrap -data-dir /tmp/consul
This starts consul in bootstrap mode.     

On CS02/CS03 

	consul agent -server -data-dir /tmp/consul

Back on CS01

Detach tux so you can run the following to join other nodes to consul cluster. 

	consul join cs02 cs03
This should return instantly. To check it was successful run: 

	consul members
At this point we need to remove the bootstrap server and have it re-join as a regular server.  This is important because at this point the bootstrap server can make decisions without consulting the other nodes in the cluster. The servers are supposed to act as a quorum and having a "super server" is not a good situation. 

Stop consul service on CS01: 

	CTRL+C 
	
Now join CS01 back to the cluster. 
Log into either CS02 or CS03 and run:

	consul join cs01
	
Now confirm all 3 servers are in the cluster. 
	
	consul members
You should end up with something like:

	Node  Address          Status  Type    Build  Protocol  DC
	cs03  172.17.0.7:8301  alive   server  0.5.2  2         dc1
	cs02  172.17.0.6:8301  alive   server  0.5.2  2         dc1
	cs01  172.17.0.5:8301  alive   server  0.5.2  2         dc1
	
Now all 3 servers are in the cluster and new servers can be added by running the above consul command without -bootstrap. 

Now let's look into what it takes to configure a client that will interact with the server cluster. 

First let's download the Web UI. 

Download the latest version from [Consul](https://www.consul.io/downloads.html)

**NOTE** How do I wget the web UI in client container without having server containers do it too? Dockerfile? docker-compose?

	wget https://dl.bintray.com/mitchellh/consul/0.5.2_web_ui.zip
	unzip *.zip
	rm *.zip 

Now you should have a dist directory which contains the needed files for the consul web UI. We just need to give it the directory when we connect to the cluster. 

We will connect without the `server` flag since this is a client connecting. By default it is configured to only listen on the loopback interface so we'll need to specify the public IP to enable remote access. 

When we join we will pass the IP address of one of the servers to join the cluster immediately. We also could have done this when we joined the servers earlier. Currently consul's -client argument only accepts IP address so you cannot use the hostname. 

	consul agent -data-dir /tmp/consul -client 172.17.0.8 -ui-dir dist/ -join cs01
	
Now let's confirm the client joined successfully. On CS01 run: 

	consul members 
	
You should see the cc01 joined as client. 

	Node  Address          Status  Type    Build  Protocol  DC
	cs02  172.17.0.6:8301  alive   server  0.5.2  2         dc1
	cs03  172.17.0.7:8301  alive   server  0.5.2  2         dc1
	cc01  172.17.0.8:8301  alive   client  0.5.2  2         dc1
	cs01  172.17.0.5:8301  alive   server  0.5.2  2         dc1
	
Because the client joined with it's public IP rather than the default loopback IP if you run any consul commands on the client you need to add the `-rpc-addr` argument. So if you wanted to see the list of members from the client you would run something like this: 

	consul members -rpc-addr=172.17.0.8:8400
	
