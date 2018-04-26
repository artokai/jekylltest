---
title: "SharePoint Framework and GitLab Continuous Integration"
header:
  teaser: "images/posts/2016-12-13_teaser.jpg"
tags:
- SharePoint
- SPFX
- GitLab
- Continuous Integration
---

Microsoft provides quite nice build tools for the upcoming
[SharePoint Framework](https://github.com/SharePoint/sp-dev-docs).
In addition to building the project using `gulp build`, you can also 
run your unit tests using `gulp test`. Having unit tests is nice, 
but having them executed automatically whenever you commit code to your source 
control system is even nicer. In this blog post I'll demonstrate
how you can setup automatic builds and unit testing using 
[GitLab](https://about.gitlab.com/) and [Docker](https://www.docker.com/).

In GitLab builds are executed by something called "build runners". Build
runners are servers/processes which execute build scripts whenever new commits are 
made to the source code repositories. To configure your repository to run these 
automatic builds, just add a configuration file called `.gitlab-ci.yml' in 
the root of your project. This tells GitLab to pick up your commits and execute
build tasks defined in that configuration file. 

<figure class="align-center">
  <img class="align-center" src="/images/posts/2016-12-13_buildagent.jpg" style="max-width: 710px" alt="GitLab Build Runner"/>
</figure>

The nice thing about these build runners is that they can be configured to 
support Docker images. This means that we do not need to install node, gulp,
or any other build tools required by the SharePoint Framework on the build
servers themselves. We can just tell GitLab to execute the build commands
on a Docker image that contains the necessary build tools. This way we can use the same
build servers to build multiple different kind of projects without worrying 
that their dependencies would somehow conflict which each other. 

The Docker image we use in this setup can be found in the [Docker Hub](https://hub.docker.com/) 
as [artokai/spfx-ci](https://hub.docker.com/r/artokai/spfx-ci/). It has all the 
necessary SPFX build tools installed and it also contains a preloaded set of 
npm packages required by the SPFX framework. I'll explain the preloading part a little bit later...

The simplest version of actual configuration file (`.gitlab-ci.yml`) which triggers the automatic
builds looks like this: 

```yaml
image: artokai/spfx-ci

stages:
  - build
  - test

before_script:
  - npm install --no-optional

execute-build:
  stage: build
  script:
  - gulp build

execute-test:
  stage: test
  script:
  - gulp test
```

Basically we just tell GitLab to use our Docker image when building the project 
and then we define two different stages (`gulp build` and `gulp test`) for the 
actual build process. The `before_script` section is used to download all required
npm packages before `build` and `test` stages are executed. This is required since you probably
don't have the `node_modules`  in your source control system and you need to download those on 
build agent (or actually on the Docker image running on the build agent).

The configuration listed above works but it does have some problems. Whenever a new commit is made,
the build process is executed on a "clean" build agent. This means that all of the files contained
in the `node_modules` (30 000 files, ~200MB) are downloaded and installed **on every commit**. 
And to make things worse, a clean environment is created for all of the 
stages defined in your `.gitlab-ci.yml` file. This means that the full SPFX framework is downloaded
and installed **twice** whenever you make a change in your source code repository.

Our Docker image already contains a set of common dependencies required by the SharePoint Framework. 
So we can speed up the build process by "prepopulating" our project's `node_modules`-directory before 
running the `npm install` command. But instead of copying all those files, we're just going to 
create a symlink pointing to our pre-prepared folder. (The `npm prune` command is there to make sure
that the `node_modules` directory does not end up having any "extraneous" packages.)

```yaml
image: artokai/spfx-ci

stages:
  - build
  - test

before_script:
  - ln -s /var/spfx_precache/node_modules/ node_modules
  - npm prune
  - npm install --no-optional

execute-build:
  stage: build
  script:
  - gulp build

execute-test:
  stage: test
  script:
  - gulp test
```  

This change prevents our build agents from downloading all of those regular SPFX dependencies every time 
it builds our projects. But what if our project has any additional project specific dependencies? Those 
will still be downloaded on every build. Luckily GitLab supports also 
[caching](https://docs.gitlab.com/ce/ci/yaml/#cache), which allows us to pass a set of files and 
folders from one build to the next. 

I first tried caching the entire `node_modules` directory, but caching 30 000 files took way too 
much time. Therefore I ended up caching just the local npm cache which gets created when we install
our project specific packages for the first time. When the project specific modules are
required on the next build, npm finds them in the local cache folder and installs them from there.

Since GitLab supports caching only under the project directory, we need to tell npm to
use a project specific cache folder (`.npm_cache`) during the build. The size of this folder will 
be quite small since all the regular SPFX modules are already present
in our symlinked `node_modules` folder and won't therefore be downloaded and cached by npm.

```yaml
image: artokai/spfx-ci

cache:
  key: "$CI_BUILD_REF_NAME"
  paths:
    - .node_cache/

stages:
  - build
  - test

before_script:
  - ln -s /var/spfx_precache/node_modules/ node_modules
  - npm prune
  - mkdir -p .node_cache
  - npm install --no-optional --cache .node_cache --cache-min Infinity

execute-build:
  stage: build
  script:
  - gulp build

execute-test:
  stage: test
  script:
  - gulp test
```

The next step would naturally be uploading the resulting JavaScript files to a CDN as 
part of the automatic build process. This would allow them to be automatically updated 
in our staging environment whenever all of the unit tests have passed. But this will
be a topic for another blog post...
