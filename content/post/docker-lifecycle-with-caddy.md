+++
date = "2016-05-06T09:53:40-07:00"
draft = true
tags = ["caddy", "docker"]
title = "Docker lifecycle with Caddy"

+++

Caddy was recently updated to 0.8.3[^1]. My website's Docker image was built
with the previous version, so I had to update the corresponding Dockerfile and
rebuild it on the Docker Hub[^2].

To update the container running on my DigitalOcean droplet, I just ran the
following:

```
docker pull steeef/stp5net
docker stop stp5net
docker rm stp5net
docker run -d --restart unless-stopped <environment-specific-options-here>  --name
stp5net steeef/stp5net
```

Notice I'm using the `--restart unless-stopped` restart policy[^3] for the `run`
command. Docker runs as a daemon on my host, and this is a pretty simple way to
ensure the container's normally running. Note that this particular policy was
added in Docker 1.9. Previous versions can use `always` for similar
functionality.

[^1]: https://caddyserver.com/blog/caddy-0_8_3-released
[^2]: https://hub.docker.com/r/steeef/stp5net
[^3]: https://docs.docker.com/engine/reference/run/#restart-policies-restart
