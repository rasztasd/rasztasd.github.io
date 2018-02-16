---
layout: post
title:  "Integration testing using Docker"
date:   2018-02-16 13:18:27 +0100
categories: programming testing docker
---
Using some kind of virtualization for testing a software is always a good idea. Containers are a kind of virtualization technology where you virtualize
only a smaller subset of the function of a real operating system thus it's more lightweight.

By using containers you can ensure that you program dependencies stays the same in every environment. You don't have to get the correct version of libraries and tools (for example apache http or a specific version of a database management system) for each environment (be it developer machine, staging and production server).

Containers are useful if you want to build/test/run an application with it's dependencies on everywhere.
If you run a container you have 2 ways to put your application code inside the container.
Put your code/application inside the docker image or mount your code/application to your container at runtime by mouting a volume or a directory with your code/application.


Let's implement a sample build/test/run of a simple application with containers and let's see the different methods to implement them.


Sample application
TODO
link source

Build

Test

Run



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
