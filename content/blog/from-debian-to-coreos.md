+++
date = "2016-09-23T12:05:10-07:00"
draft = false
tags = ["coreos", "docker", "debian", "caddy", "rkt"]
title = "From Debian to CoreOS"

+++

## Moving to CoreOS

Just a quick entry here. Historically this site has been running on a [Digital
Ocean](https://www.digitalocean.com) Debian droplet. I liked having a remote
Linux host that I could run random things on, including this blog.

However, recently I've only been using it to run things within Docker
containers. Since [CoreOS](https://coreos.com/) was designed for exactly this
purpose, I decided to try setting up a single-host cluster as a DO droplet. It
was super-easy to set this up as a replacement for the old Debian host. And
I even went as far as automating the container set up with cloud-config
userdata[^1]. Now if I need to replace this host for whatever reason, I'll just
spin up a new one with the same userdata. Easy-peasy.

## Future Changes

One thing I had to change with my Docker container was to have Caddy use
CloudFlare to create an SSL cert. I'd rather use Let's Encrypt, but when I was
testing it I was impatient with how quickly the IP needed to be changed, and
since it's behind CloudFlare anyway, it was an easy fix. I'll have to figure
out a better way to keep using Let's Encrypt and replacing hosts in the future.

I also wanted to try using [rkt](https://coreos.com/rkt/) instead of Docker to
run my containers. The fact that there are less layers between systemd and the
running container is appealing. Unfortunately, there's a bug with docker2aci
that drops things like setting limits or adding capabilities.[^2] The container
I'm using runs as a non-root user, and I didn't want to have to change that
just to switch to rkt. Once that bug is addressed, I'll definitely give it
another go.

[^1]: https://gist.github.com/steeef/d4649aa2523a17fd53e926a188273a07
[^2]: https://github.com/coreos/rkt/issues/3179
