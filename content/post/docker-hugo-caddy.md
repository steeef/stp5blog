+++
date = "2016-03-30T13:28:14-07:00"
tags = [ "docker", "hugo", "caddy"]
draft = false
title = "Creating my blog with Docker, Hugo, and Caddy"

+++

## A lonely web server

I pay for a small [DigitalOcean](https://m.do.co/c/2ea5c6b7a45b) droplet
running Debian. I originally created it just to have a remote server with
which to play around with various tools. One of which was to host a static
site via nginx running in a Docker container. Nothing's really changed with
it since that initial setup, save running the container with
systemd-nspawn[^1].

## Let's get blogging

Fast forward a few years, and I've been itching to try out some newer
technologies. I was inspired by a Hacker News post[^2] to start a blog, even if
it's just to use as a personal journal or record of new things I've learned.

I wanted a platform that was easy to install, maintain, and update.
[Hugo](https://gohugo.io) fits the bill:

1. It's written in Go, so it's a statically-compiled binary that's easy to
   install and update.
1. It's fast. It compiles static web sites from templates and Markdown files in
   a few seconds.
1. It's got a lot of themes that are easy to tweak. I'm using
   [Minimalist](https://github.com/digitalcraftsman/hugo-minimalist-theme).
1. Updating it is as simple as adding commits to a git repository.
1. Testing new themes, configurations, and drafting new entries can be done
   anywhere. I can run this against my locally-hosted git repository to spawn
   a local daemon that will listen for new changes while I'm drafting a new
   post: `hugo server --buildDrafts`

## How to host my blog?

I'm familiar with using nginx, and it's a well-established web server. However,
there are a lot of configuration tweaks that need to be done to get it to
securely host a static web site. Another Hacker News post[^3] led me to
[Caddy](https://caddyserver.com/), a secure-by-default web server written in Go
and designed to host static web sites. Like Hugo, it's a statically-compiled
binary, so it's easy to install. It also has nice addons like
[git](https://caddyserver.com/docs/git) that enable extra functionality.

This addon allows Caddy to clone my Hugo repository and run hugo to compile the
blog:

```
 git {
    repo https://github.com/steeef/stp5blog.git
    hook /somehook "secret"
    path ../git
    then hugo --destination=/srv/blog
  }
```

Note that `hook` directive, which GitHub will trigger whenever I push new
updates to the blog's repository (the git addon also configures Caddy to
check once an hour for new updates, by default).

## Operationalization (a real word)

To get all of this daemonized, I'm using a Dockerfile with some static web
files[^4] that runs a Docker container with Caddy and Hugo installed. Caddy
clones my Hugo repository and waits for updates. Easy-peasy.

My blog's git repo is hosted on GitHub here:
[https://github.com/steeef/stp5blog]

## Future enhancements

One thing I'd like to do is start using [Let's
Encrypt](https://letsencrypt.org/). They've made it very simple to
automatically generate TLS certificates and keep them updated. Heck, as of 0.8,
Caddy has support built in for automatically creating Let's Encrypt certs[^5].

## Acknowledgements

- [abiosoft/caddy-docker](https://github.com/abiosoft/caddy-docker) for
  a simple Dockerfile that I used to fork my own repo from.
- [jojomi/docker-hugo](https://github.com/jojomi/docker-hugo) for a Hugo
  Dockerfile.
- [Andrew Codispoti's blog entry](http://www.andrewcodispoti.com/deploy-process/)
  describing the Hugo deploy process.
- [Abiola Ibrahim's blog entry](https://abiosoft.com/moving-to-caddy/) on
  switching to Caddy.

[^1]: https://seanmcgary.com/posts/run-docker-containers-with-systemd-nspawn
[^2]: https://news.ycombinator.com/item?id=11068902
[^3]: https://news.ycombinator.com/item?id=9452606
[^4]: https://github.com/steeef/stp5net
[^5]: https://caddyserver.com/docs/automatic-https
