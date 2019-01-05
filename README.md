# Pipelines

In this workshop, we'll cover the basics of setting up a simple delivery pipeline, consisting of git hooks and shell commands.

![magic](img/magic.jpeg)

### What and why pipelines?

A *delivery pipeline* is a workflow system for building, validating, and deploying changes into a production environment. Pipelines are essential for supporting the paradigm of *continuous deployment*. A pipeline consists of stages, which typically represents a software engineering process, such as testing, static analysis, acceptance testing, or code review. When fully automated, pipelines allow commits to source code to be automatically tested and "seamlessly" deployed into production environments within minutes.

In practice, the ecosystem for building pipelines can be quite complex.

![complex](img/complex.png)

While more advanced pipelines can be created with tools like Spinnaker and Jenkins, using *simple tools*—such as git and shell commands—can get the job done.

## Workshop

### Before you start

* [Install opunit and node.js](https://github.com/CSC-DevOps/profile#opunit)
* Clone this repo with: `git clone --recursive https://github.com/CSC-DevOps/Pipelines`. Note, `--recursive` is required, as the App directory is a submodule.

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

We will add a **pre-commit** hook for `/App` that will cancel a commit if `npm test` fails. Because `/App` is a submodule, its hooks are located in a slightly different location: `.git/modules/App/hooks`.

![pre-commit](img/pre-commit.png)

```sh
#!/bin/bash

npm install
# Get the exit code of tests.
if npm test; then
  echo "Passed tests! Commit ✅ allowed!"
  exit 0
fi
echo "Failed npm tests. Canceling 🚫 commit!"
exit 1
```

Change the message in App/main.js from "Hi From" to "Bye From". Attempt to commit the file (`git add main.js`; `git commit -m "checkin"`). Confirm the tests fail, preventing the commit from being added.

> ❗️Not working as expected? Make you made the pre-commit script executable (`chmod +x pre-commit`)

### Adding an install and deploy stage

We will add a new directory to publish our source code.

![post-receive](img/post-receive.png)

We need to create two directory paths:

* deploy/production.git — This will serve as our remote repository.
* deploy/production-www — This will hold the contents of our deployed web app.

To create the production.git, we need to do something a little different. We need to create a *bare repository*, that is, a repository without a staging area and working tree. Instead, a bare repository only holds git objects, from which code can be extracted as needed. The advantage of a bare repository is that it helps avoids issues such as having a merge issues on production server.

Inside production.git, run `git init --bare`.

To hold the current version of software, we will use the production-www, and extract it from the bare repository using `git checkout`.

We will use the **post-receive** event, to create a hook to perform the git checkout operation for us.

Create the following post-receive hook for production.git:

```sh
#!/bin/sh
echo "Current location: $GIT_DIR"
GIT_WORK_TREE=../production-www/ git checkout -f
echo "Pushed to production!"
cd ../production-www
npm install --production
```

This script copies over the content of the latest code in production.git into production-www, and installs the appropriate dependencies for the web app.

This script does not run the web app, however. To do that, we will install an utility, [pm2](http://pm2.keymetrics.io/docs/usage/quick-start/), by running `npm install pm2 -g`. pm2 will ensure that the web app will stay running, even if it crashes.

We add the following steps to our script, after npm install:

```sh
npm run stop
npm run start
```

The details for how pm2 is run using [the process.json](http://pm2.keymetrics.io/docs/usage/application-declaration/#json-format), can be found in package.json:
```js
  "scripts": {
    "test": "mocha",
    "start": "node main.js start 5001",
    "deploy": "pm2 start process.json",
    "stop": "pm2 stop process.json"
  },
```

### Adding a git remote; Trying it out

Finally, we need to link the App repository with the *remote* production.git repository. While this is still located on the same machine, in practice, the process would be similar for a remote machine hosting a git repository.

Inside the App/ directory, run the following commands:

    git remote add prod deploy/production.git

Update the message, to be "Hi From production" and commit locally.

You can now push changes in App to remote repo in the following manner.

    git push prod master

You should be able to visit http://localhost:5001/ and see the changes you made in app, and pushed into production!


## Concept questions

* What are some issues that might occur if required to pass tests in a pre-commit hook?
* What are some issues that could occur when running npm install (when testing), and then npm install again in deployment?
* Why is pm2 needed? What problems does this solve? What problems other problems might exist in more complex applications that our pipeline does not address?

## Next steps.

Instead of just targeting local resources and services, we can easily trigger other remote services and tools.

Using, `curl`, we can send HTTP requests to initialize all kinds of tasks. For example, we could modify our hooks to trigger a build on a jenkins server. 

```
curl -X POST http://YOUR_JENKINS_URL/job/YOUR_JOB/build?TOKEN=YOUR_API_TOKEN
```

or send an email:

```
curl smtp://mail.example.com --mail-from myself@example.com --mail-rcpt
receiver@example.com --upload-file email.txt
```

Because you may not have direct access to a git server, such as a repository hosted on GitHub, you can alternatively configure [WebHooks](https://developer.github.com/webhooks/). WebHooks provide the ability to generate HTTP requests with payloads that can allow integration with many different services and tools.

Pipeline events can be published on places like Slack.

![slack](img/slack.png)
