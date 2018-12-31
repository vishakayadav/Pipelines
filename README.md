# Pipelines

In this workshop, we'll cover the basics of setting up a simple pipeline, consisting of git hooks and curl commands. 

While more complex pipelines can be created with tools such as, spinnaker and jenkins, this approach can get the job done.

### Hooks

order of hooks...

```sh
#!/bin/sh

GIT_WORK_TREE=../production-www/ git checkout -f
SHA1=$(git rev-parse HEAD)
MSG=$(git show -s --format=%B $SHA1)
echo "Received push to production: $MSG"
```

### Something with curl...

* Webhooks?