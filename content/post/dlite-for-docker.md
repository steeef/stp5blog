+++
date = "2016-03-31T10:04:49-07:00"
tags = ["docker", "dlite"]
draft = true
title = "Using Docker in OS X with DLite"

+++

## The Past

If you've ever used Docker in OS X, you're probably familiar with the pain of
being forced to run a Linux VM to test Docker containers. I've personally spent
hours troubleshooting problems while using boot2docker and its successor,
docker-machine. Historically, relying on VirtualBox for anything in OS
X development has never been painless (though I'll admit it's improved over the
past few years).

## xhyve

Luckily, there's a better way. OS X's kernel has ties to the BSD kernel, and
has benefitted from a lot of improvements that were originally developed for
BSD. One of these is [bhyve](http://www.bhyve.org/), the BSD Hypervisor. OS
X has a port named [xhyve](http://xhyve.org). The benefit of using xhyve over
something like VirtualBox or VMware Fusion is that it's lightweight and runs
entirely in the userspace. The catch is that you need to be running at least OS
X Yosemite 10.10.3, and on hardware that supports it. To check, use `sysctl
kern.hv_support`, which should return something like the following:

```
> sysctl kern.hv_support
kern.hv_support: 1
```

## DLite

Now that we have a hypervisor, we need a way to automate the use of Docker with
it: [DLite](https://github.com/nlf/dlite). This is a utility written in Go
(surprise!) that automates the process of getting an xhyve Linux VM that we can
use to talk to the Docker daemon directly. Here's how I got it running with
[Homebrew](http://brew.sh):

```
brew install dlite
sudo dlite install
dlite start
```

That gets you the ability to use Docker's commands to interact with the Docker
daemon running on the DLite VM. The `sudo` part is necessary to set up some
system-level stuff, including:

- an NFS export of the `/Users` directory (for sharing with containers)
- a socket at `/var/run/docker.sock` that Docker can access directly

Note that you don't need Homebrew to get DLite installed. You can build with
Go, or even just download the binary from the GitHub releases page.

Now you should be able to interact with Docker the same way you would on
a Linux host, with commands like:

```
docker ps
docker run -d -v /Users/me/test:/test 80:80 someimage
```

DLite sets up a hosts file entry so that you can access any bound ports on
`local.docker` (if this doesn't work, run `dlite ip` to discover the IP
address), and also creates a launchctl entry so that the DLite VM is started
upon login. You really don't need to interact with dlite much after that except
to possibly upgrade it in the future.
