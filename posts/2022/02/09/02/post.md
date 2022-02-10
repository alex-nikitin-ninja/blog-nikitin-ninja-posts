# nginx + letsencrypt + docker = multistack web app

Imagine that we have a few different web services which work on different
languages, environment with its own configuration set.

What would be the approach of running applications on staging or live
environment easily without overengineering and easy deployment process with CI
pipelines.

Obvious first response is to use `docker` for wrapping our applications into
different environmemts.

But how would we route traffic from incoming requests into various docker
containers?

The answer is to use `nginx` as a reverse proxy and route traffic based on domain
name or path name.

And for taste of being encrypted and have all those services and microservices
behind https we use free let's encrypt certificates and their bot to keep
everything up-to-date automatically.

-more-

## Input data

We have a FQDN: chiken.com

We have a VPS or bare metal server


