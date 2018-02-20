---
layout: post
title:  "Integration testing using Docker"
date:   2018-02-16 13:18:27 +0100
categories: programming testing docker
---
# Why containers are useful
Using some kind of virtualization for testing a software is always a good idea. Containers are a kind of virtualization technology but it doesn't virtualize a whole computer, but only isolates the processes so it's more lightweight.

By using containers you can ensure that you program dependencies stays the same in every environment. You don't have to get the correct version of libraries and tools (for example apache http or a specific version of a database management system) for each environment (be it developer machine, staging and production server).

Containers are useful if you want to build/test/run an application with it's dependencies on everywhere.

What I mean by building application is you create a binary inside docker but after that you don't use docker anymore.

Testing means you test the code runs inside a docker and the tested code can run inside the same docker container, a different docker or on the host machine.

Running means you distribute your application in a container format. That means your application is not distributed as a library but as docker image.

If you run a container you have two ways to put your application code inside the container.
Put your code/application inside the docker image or mount your code/application to your container at runtime by mouting a volume or a directory with your code/application.

Normally you would put things that don't change too often inside your image and you try to put your often changing content like your source code into the container by adding it as a volume or mouting it.

Another good practice is if possible then never build docker containers and try to use containers that already has all it's dependencies and try to use it's default settings. I will show any example on this later.

I don't want to go into details on how to make a docker image. You either use a dockerfile and docker build it or you make changes inside a container and commit it as an image.

# Sample application
Our application is a very small go web application that will print **Hello world!** to the browser.
{% highlight go linenos %}

package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello world!")
	})
	log.Fatal(http.ListenAndServe(":8080", nil))
}
{% endhighlight %}

# Build application inside docker

{% highlight makefile linenos %}
do_build:
	go build -o build/http_server .

build_in_docker:
	docker run -w /go/src/github.com/rasztasd/sample_go_and_docker -v `pwd`:/go/src/github.com/rasztasd/sample_go_and_docker golang:1.9 /bin/sh -c "make do_build"

{% endhighlight %}
Call ```make build_in_docker``` in the command line to build the artifacts. Because the ```go build -o build/http_server .``` will put the built artifact to the build folder, which is mounted to the current folder, after the build the artifact will be visible.

If we didn't mount our current folder to the docker or mount to a different location, then the artifacts after the build would no be visible automatically and we would have to ```docker cp``` it.

So with the above Makefile you can build your go files without having go lang installed on your machine. Only docker is required.

# Test your application
The tests usually needs the same dependencies with some additional dependencies as your applications. For example you need the go lang tools for running the unit tests. So it's a good idea to run our tests (unit and  integration_test) inside a docker container as well.






link source

Build
TODO
copy artifact from container.
1. mount the target file location.
2. copy with docker cp after the build has run.

Test

Run



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
