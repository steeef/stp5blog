+++
date = "2016-04-01T10:04:04-07:00"
draft = true
tags = ["caddy", "docker", "lets-encrypt"]
title = "Caddy and Let's Encrypt"

+++

*Note: this is a follow-up to my earlier post on setting up Caddy with Docker,
which is [here]({{<relref "post/docker-hugo-caddy.md" >}}).*

I've just enabled Caddy's Automatic HTTPS function, which leverages [Let's
Encrypt](https://letsencrypt.org/) to generate a key and get a signed
certificate as soon as the server starts up. It's free and simple. Awesome!

I was able to figure this out by reading the official documetation on
Automatic HTTPS[^1] and Abiola Ibrahim's example Dockerfile for Caddy, which
included a nice section on how to persist the `.caddy` directory[^2].

[^1]: https://caddyserver.com/docs/automatic-https
[^2]: https://github.com/abiosoft/caddy-docker#lets-encrypt-auto-ssl
