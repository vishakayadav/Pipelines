# Pipelines

In this workshop, we'll cover the basics of setting up a simple pipeline, consisting of git hooks and curl commands. 

While more complex pipelines can be created with tools such as, spinnaker and jenkins, this approach can get the job done.

### Prereqs



### Checking progress on workshop

To help you identify if issues exist with the current setup, you can run the following command to check:

```bash
$ opunit verify local
```


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