---
layout: doc
title: CI/CD Integration
section: Tutorial
---
# {{ page.title }}

## GitHub Actions

Up until now, we have run our tests files locally. Now, we want to integrate them in a CI/CD pipeline (like
GitHub Action or GiLab pipeline). As Hurl is very fast, we're going to run our tests on each commit of our project,
drastically improving the project quality. 

A typical web project pipeline is:

1. build your application, run units tests and static code analysis,
2. publish your application image to a Docker registry,
3. pull your application image and run integration tests.

In this workflow, integration tests are run against a docker image, the same image that wil be used and deploy
in production. In our tutorial, we are going to skip the build and publication phase and only run integration tests
on a prebuild docker image. If you want to check a fully sample, you can go to <https://github.com/jcamiel/quiz>  


### A Basic integration script

1. Create a new empty repository in GitHub

![Create new GitHub repository]({{ '/assets/img/github-new-repository.png' | prepend:site.baseurl }})

{:start="2"}
2. Clone the repository:

```shell
$ git clone https://github.com/jcamiel/quiz2.git
Cloning into 'quiz2'...
warning: You appear to have cloned an empty repository.
$ cd quiz2
``` 

Now, we are going to add our integration tests. We are going to write a first version of our script that will just pull
the Quiz image and run it:

{:start="3"}
3. Create a script named `bin/integration.sh` with the following content:

```bash
#!/bin/bash
set -eu

echo "Starting Quiz container"
docker run --name quiz --rm --detach --publish 8080:8080 ghcr.io/jcamiel/quiz:latest
```

{:start="4"}
4. Make the script executable and run it :

```shell
$ chmod u+x bin/integration.sh
$ bin/integration.sh
Starting Quiz container
5d311561828d6078e84eb4b8b87dfd5d67bde6d9614ad83860b60cf310438d2a 
```

{:start="5"}
5. Verify that our container is up and running:

```shell
$ docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
c685f3887cc1   ghcr.io/jcamiel/quiz:latest   "java -jar app/quiz.…"   3 seconds ago   Up 3 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   quiz
```

Now, we have a basic script that start our container. Before adding our integration tests, we need to ensure that our application server is ready: the container
have started, but the application server can take a few seconds to be really ready.

To do so, we're going to use Hurl and test our health api. With a function `wait_for_url`, we provide Hurl a simple test file
that check a given url to return a `200 OK`. We loop on this function until the check succeed.

{:start="6"}
6. Modify `bin/integration.sh` to wait for the application to be ready:


```bash
#!/bin/bash
set -eu

wait_for_url () {
    echo "Testing $1"
    max_in_s=$2
    delay_in_s=1
    total_in_s=0
    while [ $total_in_s -le "$max_in_s" ]
    do
        echo "Wait ${total_in_s}s"
        if (echo -e "GET $1\nHTTP/* 200" | hurl > /dev/null 2>&1;) then
            return 0
        fi
        total_in_s=$(( total_in_s +  delay_in_s))
        sleep $delay_in_s
    done
    return 1
}

echo "Starting Quiz container"
docker run --name quiz --rm --detach --publish 8080:8080 ghcr.io/jcamiel/quiz:latest

echo "Starting Quiz instance to be ready"
wait_for_url 'http://localhost:8080' 60

echo "Stopping Quiz instance"
docker stop quiz
```






