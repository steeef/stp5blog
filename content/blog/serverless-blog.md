+++
title = "Serverless Blog"
draft = false
tags = ["aws", "cloudfront", "s3", "serverless","travis"]
date = "2017-03-03T14:19:28-08:00"

+++

# Hello from CloudFront

As of today, this blog is now being served by Amazon S3, and cached globally by
CloudFront. You can say it's "serverless", even if the term isn't quite true
(after all, it's always running on someone's server). The transition wasn't too
easy, but it was my first crack at hosting a site purely in S3, and it was
a chance to learn to use CloudFront and [Travis CI](https://travis-ci.org/).

# The Road to "Serverless"

When it started, this blog was hosted in a DigitalOcean Debian droplet with
Caddy as its webserver. You can read the first entry [here]({{< relref
"docker-hugo-caddy.md" >}}). Most recently, I [transitioned to a CoreOS
droplet]({{< relref "from-debian-to-coreos.md" >}}). Somewhere along the way
I put CloudFlare in front of it to handle SSL termination and caching.

I still like using CoreOS, and my DigitalOcean account isn't going anywhere. But at
least for the purposes of this blog, it doesn't really need a full-blown server
to run it, even while it's running as a Docker container.

After writing a post, all I really need is somewhere to run Hugo to generate
the static content, and then some place to upload it. `$job.current` has
already allowed me to explore much of Amazon Web Services, so I knew I could
host it in an S3 bucket as a static website[^1]. The challenge was moving from
CloudFlare to Amazon CloudFront as a CDN.

# CloudFront: Plenty of Options

My experience with CloudFlare has been pretty painless. Most of the defaults
just work, and they give you enough tips to get a stable, cached website with
SSL enabled.

CloudFront is very powerful, but it's easy to shoot yourself in the foot,
either by setting the TTL too high and wondering why your content is
missing/outdated, or invalidating the entire cached copy of your website and
spending $0.005 per object (yikes). I'm still within my free year, so no money
lost just yet, but I can definitely see how to shoot myself in the foot (or
wallet in this case).

Getting Amazon Certificate Services to generate an SSL cert for the domain was
the work of a few minutes.

# Travis CI

So now that I have a bucket, it's pretty trivial to run `hugo` to generate the
content and upload it to s3. However, it's kind of a pain to maintain that as
a separate step in my workflow. I'd rather hand it off to an automated build
system. Enter: Travis CI. Travis CI is traditionally a CI tool (go figure), but
like Jenkins, it's pretty easy to extend it to perform CD. After looking up
a few examples, I was able to write a `.travis.yml` file that installed Hugo,
built the static files, and then used `s3cmd` to synchronize the files with my
S3 bucket.

Now, a push to master on GitHub kicks off a Travis build and an update to the
S3 bucket. As long as I'm creating new entries, I don't have to worry about
stale cache entries in CloudFront.

# The Future

I'm testing various tweaks to caching and building my site to see what works
best, as well as keeping an eye on events in AWS that could cost me money in
the future. So far, things are working pretty well.

# The Code

You can see the code for this entire site on GitHub here:
https://github.com/steeef/stp5blog.

# Links

[This blog post](https://lustforge.com/2016/02/27/hosting-hugo-on-aws/) was
very helpful in getting me started. The code snippets made it simple to modify
for my needs. I got good pointers on using Travis CI with Hugo from [this
  post](http://continuousfailure.com/post/s3_blog/).

[^1]: https://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html
