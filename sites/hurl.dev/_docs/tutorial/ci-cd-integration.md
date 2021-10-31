---
layout: doc
title: CI/CD Integration
section: Tutorial
---
# {{ page.title }}

## GitHub Actions

Up until now, we have run our tests files locally. Now, we want to integrate them in a CI/CD pipeline (like
GitHub Action or GiLab pieline). As Hurl is very fast, we're going to run our tests on each commit of our project,
drastically improving the build.

A typical web project pipeline is:

1. build your server, run units tests and static code analysis
2. publish your application image to a Docker registry
3. pull and 

