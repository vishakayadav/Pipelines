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

### Creating git hooks

To create a new git hook, one simply creates a new shell script inside the `.git/hooks/` directory. The shell script should be the same name as the corresponding event defined by git.

Some example events include:

* **pre-commit:** This hook will run when `git commit` is performed on a repo. The hook can inspect the commit and reject it (by exiting with a non-zero exit code, e.g., `exit 1`).
* **post-commit:** This hook will run after a commit is processed by the local repository.
* **post-receive:** This hook will run on a remote repository after a push has successfully been received and processed. This hook can be used for notifications or trigger other processes, such as a build.

### Practice

We will illustrate a simple hook by opening a webpage whenever a commit is created in a repo.

![hook demo](img/hook-demo.png)

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

We will create a simple pipeline that runs tests, installs, and "deploys" an application into production based on a commit.

### Our target application

Inside the `App/` directory, there is a simple node.js application. Go ahead and setup the app locally by running `npm install` inside the App/ directory.

Run the command: `npm start`, you should see output that looks something like:

```
$ npm start

> app@0.9.1 start .../App
> node main.js start 5001

Example app listening at http://:::5001
```

Visit http://localhost:5001 in your web browser. You should see the message, "Hi From &lt;random number&gt;"

Terminate the application (Control-C). Verify you can run the test with `npm test` and see two tests passing.

### Adding a test stage.

We will add a hook that will cancel a commit if `npm test` fails.

![pre-commit](img/pre-commit.png)

```sh
#!/bin/bash

npm install
npm test
# Get the exit code of tests.
if npm test; then
  echo "Passed tests! Commit ✅ allowed!" 
fi
echo "Failed npm tests. Canceling 🚫 commit!"
exit 1
```

## Concept questions

* What are some issues that might occur if requiring to pass tests in a pre-commit hook?

## Next steps.


* Curl...
* Webhooks...