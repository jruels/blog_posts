#Docker, Fusion, Profit!

Install docker-machine 

```
curl -L https://github.com/docker/machine/releases/download/v0.3.0/docker-machine_darwin-amd64 > /usr/local/bin/docker-machine   
chmod +x /usr/local/bin/docker-machine
```

Install docker client 

```
brew install docker
```

Create docker container with VMware Fusion (this command gives the VM 2G of RAM)

```
docker-machine create --driver vmwarefusion osxdock --vmwarefusion-memory-size 2048
```
If you run into problems try removing ~/.docker/machine 

This will cause it to re-download boot2docker


At this point you can do lots of things with Docker.  

Install Centos6 and connect to it 

```
docker run -t -i centos:centos6 /bin/bash 
```

The -t option is for TTY and the -i is to make it interactive.  You can also do something like 

```
docker run -t -i centos:latest /bin/bash
```

Which pulls the latest CentOS version.  Also ubuntu:latest, ubuntu:utopic or ubuntu:14.10 work great. 


