---
date: 2000-01-01T00:00:00+00:00
title: drone build promote
author: bradrydzewski
weight: 1
aliases:
- /cli-deploy
---

This subcommand promotes a build to the target environment. Please note this command requires push privileges to the repository.

```
NAME:
   drone build promote - promote a build

USAGE:
   drone build promote [command options] <repo/name> <build> <environment>

OPTIONS:
   --param value, -p value  custom parameters to be injected into the job environment.
```

Example usage, promotes a build by build number:

```
$ drone build promote octocat/hello-world 42 production
```

Example usage promotes in the pipeline:

```
kind: pipeline
name: default

steps:
- name: test
  image: node
  environment:
    foo: ${foo}
    baz: ${baz}
    PARAM1: ${PARAM1}
  commands:
  - npm install express@$PARAM1
  - npm test
```

Example usage, with custom parameters:

```
$ drone build promote octocat/hello-world 42 production \
  --param=foo=bar \
  --param=baz=qux \
  --param=param1=3.0.0
```
