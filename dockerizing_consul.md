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

###Docker Compose

