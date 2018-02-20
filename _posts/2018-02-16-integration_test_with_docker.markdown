---
layout: post
title:  "Integration testing using Docker"
date:   2018-02-16 13:18:27 +0100
categories: programming testing docker
---
# Why containers are useful
Using some kind of virtualization for testing a software is always a good idea. Containers are a kind of virtualization technology but it doesn't virtualize a whole computer, but only a smaller subset of functions (networking, filesytem and processes).

By using containers you can ensure that your program dependencies stay the same in every environment, just like it would in a virtual machine. You don't have to get the correct version of libraries and tools (for example apache http or a specific version of a database management system, or a specific version of java or go compiler) for each environment (be it a developer machine, staging or production server).

For example you have a java web project which produces a war file which can be deployed to a java web servlet container.
A general workflow for building and testing is the following:

* Build war file of the application (on developer machine and jenkins or any other ci server)
* Deploy war file to a test java web servlet container, most problably on the local machine or jenkins server and run test suit.
* Publish the artifact to somewhere when all the critarias are met. (Build is successful, feature is complete, etc.)
* Others e.g: Sonar analysis of the project source.

Usually each step (build, deploy and publish) need some external dependencies e.g: for building: java compiler, for deploying: web servlet container that will host our application or a RDBMS. For publishing: maven or gradle.

So the idea is to put every external dependencies inside a docker image and run the command inside a docker container that uses the docker image.

It's possible that you use docker only for building a binary artifact and for testing or publishing you don't use it. Or it's possible to use docker only for running an E2E test with lot's of dependencies and don't use it on anywhere else. It's all depending on your application.

# Sample application
Our application is a very small web application written in go lang that will print **Hello world!** to the browser.
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
Call ```make build_in_docker``` in the command line to build the artifacts. Because the ```go build -o build/http_server .``` will put the built artifact to the build folder, which is mounted to the current folder.

So with the above Makefile you can build your go files without having go lang installed on your machine. Only docker is required.

# Sample e2e/integration test code
This is a very simple test. Test whether the http status is 200 or not.
{% highlight go linenos %}

package test

import (
	"fmt"
	"log"
	"net/http"
	"testing"
)

func TestBasicConnectionNoStatusCodeOtherThan200OnIndexURL(*testing.T) {
	resp, err := http.Get("http://localhost:8083")
	if err != nil {
		panic(err)
	}
	fmt.Print("Response code: " + resp.Status)
	if resp.StatusCode != 200 {
		log.Fatalf("Response code is not 200! Most probably the server is misconfigured. Wrong port or the webapp folder is missing!")
	}
	defer resp.Body.Close()
}
{% endhighlight %}

# Run test inside docker
With the below code you will run your web server in a docker container and run **go test -v integration_test.go** in a different container.
{% highlight makefile linenos %}
integration_test_in_docker:
	docker run --network=host -d -v "${PWD}"/build:/bin2 ubuntu:16.04 /bin/sh -c "/bin2/http_server" > .application_server_container_id
	docker run --network=host --rm \
		-v "${PWD}":/go/src/github/rasztasd/sample_go_and_docker/ \
		-w /go/src/github/rasztasd/sample_go_and_docker/ \
		golang:1.9 /bin/sh -c "go test -v integration_test.go"
	cat .selenium_container_id | xargs -I{} docker stop {}
	cat .application_server_container_id | xargs -I{} docker stop {}
{% endhighlight %}


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
