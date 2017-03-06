+++
date = "2017-03-06T10:59:35-08:00"
title = "Secrets in AWS"
draft = false
tags = ["aws", "credstash", "kms", "secrets", "systems manager", "parameter store"]

+++

## Managing secrets in the cloud

Moving hosted services to cloud-based archictectures has introduced a lot of
different pain points, some new, some pre-existing that become more of an
issue. One such issue is secrets[^1].

There have been a number of different discussions and solutions for this
problem, including:

- [Hacker News: Ask HN: In a microservice architecture, how do you handle managing secrets?](https://news.ycombinator.com/item?id=10927043)
- [Docker GitHub: Secrets: write-up best practices, do's and don'ts, roadmap](https://github.com/docker/docker/issues/13490)

The main question here is: "How do you expose secrets to only those services
that require them, without exposing them to those that don't, and at the same
time make their lifecycle (rotating/replacing/expiring) easy to maintain?"

## Managing secrets in AWS

The folks at `$job.current` asked me to come up with a secrets management
solution last year. We're fully-hosted in the AWS cloud, so your requirements
may be different. Here were mine:

- Find an official AWS service. If none exist, find a whitepaper published by
  Amazon detailing a recommended approach.
- It should use KMS (Amazon's Key Management Service) to encrypt the secrets.
- It should be Highly-Available (HA) and Fault-Tolerant, such that it can withstand
  failure of a single instance.

We wanted to avoid having to stand up an entire cluster on virtual machines
(EC2 instances, in this case) and instead leverage any AWS service designed to
be fault-tolerant/HA.

## My choices

Before I selected anything, I wanted to see if there was an official way to
manage secrets in AWS. At the time I looked, Amazon recommended a combination
of encrypted S3 buckets containing credentials or config files and IAM
roles[^2]. It leverages the reliability of S3, but it's still a very manual
process to get everything set up, especially for existing hosts and services.

I narrowed down the candidates to a handful. They were generally in one of two
categories:

* Clustered hosts
  - [HashiCorp Vault](https://www.vaultproject.io/)
  - [Lyft Confidant](https://lyft.github.io/confidant/)
  - [Square Keywhiz](https://square.github.io/keywhiz/)
* Built on/Utilizing AWS services
  - [Credstash](https://github.com/fugue/credstash)
  - [Biscuit](https://github.com/dcoker/biscuit)
  - [Sneaker](https://github.com/codahale/sneaker)

Vault is the most-commonly used here. Two knocks against it though:

1. Requires setting up and maintaining a cluster of EC2 instances.
2. Doesn't use KMS.

The knocks against Confidant and Keywhiz are similar. However, Confidant does
use KMS.

Credstash and Biscuit, on the other hand, use DynamoDB to store credentials,
and KMS to encrypt/decrypt them. The main difference between them is the
language they're written in: Credstash is written in Python, Biscuit in Go.
Sneaker is similar, but stores things in S3 buckets.

## The choice, at the time

I ended up choosing Credstash for the following reasons:

- It's written in Python, a language with which `$job.current` was comfortable.
- It uses AWS services (KMS and DynamoDB) that are designed for
  HA/Fault-Tolerance, and that can be given access to via IAM roles and
  policies.

In particular, the fact that Credstash could use a binary to print credentials
to stdout meant I could run an application directly from the command line
without having to store credentials in a config file, like so:

```
java -Dsome.property="$(credstash get prd.some.property)" -jar someapp.jar
```

Using Credstash did come with a few downsides though:

1. You still need a way to automate the creation/modification of secrets, as
   well as publish their existence for other engineers to use.
2. Getting Credstash on the host means ensuring Python is available with the
   proper modules installed via some method like pip or virtualenvs. Pycrypto
   is a requirement, which must be built for the same architecture it's running
   on.

## Enter Parameter Store

Credstash worked pretty well for a while, but those downsides mentioned were
still ones I wanted to address, and they became more apparent the more secrets
we relied upon, and the more environments they were used in.

Luckily, Amazon releases new features all the time, and especially around the
month of November, during its annual re:Invent conference. Last year (2016) was
no exception, and the service that interested me the most was [EC2 Systems
Manager](https://aws.amazon.com/ec2/systems-manager/). Originally named SSM
(for Simple Systems Manager), it was a way to automate Windows hosts, both
those running in AWS as well as on premise (i.e., in physical datacenters).
Things like the [Run Command](https://aws.amazon.com/ec2/run-command/) could be
used to run arbitrary scripts against running EC2 instances in an ad-hoc
fashion, or as part of an automation.

The big change the came out of re:Invent was that a lot of the services within
SSM that were Windows-only were now available for Linux instances, and came as
part of the standard EC2 console. The most useful part to me was their official
secrets store: [Parameter Store](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/systems-manager-paramstore.html).

Like Credstash, it uses KMS to encrypt/decrypt secrets (when using the
`SecureString` type). Better, though, is that it comes as part of the AWS SDK.
You can use secrets within your own favorite language instead of being forced
to set things up outside of the application. It's also trivial to use from the
command line with the AWS CLI (which comes for free with Amazon Linux, but is
also pretty easy to install on your favorite distro). The above example would
be something like this in AWS CLI:

```
java -Dsome.property="$(\
  aws ssm get-parameters \
  --names prd.some.property \
  --with-decryption \
  --output text \
  --query Parameters[0].Value \
)" -jar someapp.jar
```

## Caveats

There are a few restrictions that may make Parameter Store a non-starter for
some:

1. There is a soft-limit of 100 parameters per account (may be one that can be
   increased in the future similar to other soft-limits).
2. Each parameter can be at most 1024 characters long. This would prevent you
   from being able to use it for something like a cryptographically-strong GPG
   key or SSL certificate. For that Amazon still recommends encrypted S3
   buckets.

That said, the fact that Amazon now has a widely-available and official secrets
management solution that can be managed by their powerful IAM policies makes my
job a lot easier, and I'd definitely recommend it to anyone looking to avoid
setting up extra infrastructure in AWS.

[^1]: When I say "secrets", I'm including anything that shouldn't be shared publicly in plaintext, like database passwords or RSA keys.
[^2]: https://aws.amazon.com/blogs/security/using-iam-roles-to-distribute-non-aws-credentials-to-your-ec2-instances/
