# nginx + letsencrypt + docker = multistack web app

Imagine that we have a few different web services which work on different
languages, configuration and environment.

What would be the approach of running them application locally or on live
environment easeily without overengineering and easy deployment process with CI
pipelines.

Obvious first response is to use docker for our applications in different
environmemt.

But how would we route traffic from incoming requests into various docker
containers?

The answer is to use nginx as a reverse proxy and route traffic based on domain
name or path name.

And for taste of being encrypted and work all those services and microservices
behind https we use free let's encrypt certificates and their bots to keep
everything updated automatically.

-more-

text after cut
