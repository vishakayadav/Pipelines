# Pipelines

In this workshop, we'll cover the basics of setting up a simple delivery pipeline, consisting of git hooks and shell commands.

![magic](img/magic.jpeg)

### What and why pipelines?

A *delivery pipeline* is a workflow system for building, validating, and deploying changes into a production environment. Pipelines are essential for supporting the paradigm of *continuous deployment*. A pipeline consists of stages, which typically represents a software engineering process, such as testing, static analysis, acceptance testing, or code review. When fully automated, pipelines allow commits to source code to be automatically tested and "seamlessly" deployed into production environments within minutes.

In practice, pipelines can be quite complex.

![complex](img/complex.png)

While more complex pipelines can be created with tools like Spinnaker and Jenkins, using *simple tools*—such as git and shell commands—can get the job done.

## Workshop

### Before you start

* [Install opunit and node.js](https://github.com/CSC-DevOps/profile#opunit)
* Clone this repo with: `git clone https://github.com/CSC-DevOps/Pipelines`

### Checking progress on workshop

To help you identify if issues exist with the current setup, you can run the following command to check:

```bash
$ cd Pipelines
$ opunit verify local
```
## Hooks

A hook is a mechanism for specifying an action that occurs in response to an event. The action can be used to trigger other events. Thus, hooks can be composed together in order to create a simple pipeline.

Git provides [a hook mechanism](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that can be used to customize responses to events, such as commits or pushes. A hook is run as a shell script in response to certain events that occur when git processes changes to source code.

Some example hook include:

* **pre-commit:** This hook runs when `git commit` is performed on a repo. The hook can inspect the commit and reject it.
* **post-commit:** This hook occurs after a commit is processed by the local repository.
* **post-receive:** This hook occurs on the remote repository after a push has successfully been processed. This hook can be used for notifications or trigger other processes, such as a build.

### Practice

Inside a new directory (`mkdir hook-demo`), create a new git repository with `git init`. Create a post-commit file located in "hook-demo/.git/hooks/post-commit". Finally, you should ensure the post-commit script is exectuable, by running `chmod +x post-commit`.

The script for post-commit might look something like this:

```sh
#!/bin/sh

# In Mac
open https://google.com/
# In Windows
# start https://google.com/
# In Linux
# xdg-open https://google.com/
```

Trigger the commit by create a simple commit in hook-demo. (`touch demo`; `git add demo`; `git commit -m "init"`. You should see the webpage open.)


## A Simple Pipeline

```sh
#!/bin/sh

GIT_WORK_TREE=../production-www/ git checkout -f
SHA1=$(git rev-parse HEAD)
MSG=$(git show -s --format=%B $SHA1)
echo "Received push to production: $MSG"
```

### Something with curl...

* Webhooks?