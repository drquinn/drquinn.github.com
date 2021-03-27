---
layout: post
title: Run Django Migration with AWS Copilot
---

So you've set up Django with Copilot on Fargate. You've deployed your environment, now you need to run migrations. How to do it? 

Well it wasn't really possible until a few weeks ago, when Copilot [relaesed a new feature](https://aws.amazon.com/about-aws/whats-new/2021/03/aws-copilot-launches-v1-4/) called `exec`.

Previously, I could only run tasks which would run in their own siloed containers. With `exec`, I can run commands and tasks in my running Docker container!

In my case, I have a Django instance running as a Load Balanced Web Service, deployed by AWS Copilot into my `test` environment.

To run migrations, I can jump into the running service and execute the migration:

```
copilot task exec --command "python manage.py migrate"
```

If you want to jump into the container to have a peek around, you can simply run:
```
copilot task exec
```

You then have full command line access to the container.

In case you hit an error I hit when I first ran this:

```
Found only one deployed service core-api in environment test
Execute `/bin/sh` in container allen-core-api in task xxx.
✘ Failed to execute command /bin/sh. Is `exec: true` set in your manifest?
✘ execute command /bin/sh in container core-api: execute command: AccessDeniedException: User: arn:aws:sts::xxx is not authorized to perform: ecs:ExecuteCommand on resource: arn:aws:ecs:us-west-1:xxxx
```

The answer is right there in the error, you need to set `exec: true` in your manifest. So be sure to do that:

```

# Configuration for your containers and service.
image:
  # Docker build arguments. For additional overrides: https://aws.github.io/copilot-cli/docs/manifest/lb-web-service/#image-build
  build: ./Dockerfile
  # Port exposed through your container to route traffic to it.
  port: 8000

cpu: 256       # Number of CPU units for the task.
memory: 512    # Amount of memory in MiB used by the task.
count: 1       # Number of tasks that should be running in your service.
exec: true
```

You will also need to redeploy your service to update the manifest:

```
copilot svc deploy
```

This should allow you to run migrations, or any other mutating executions that need to occur outside of the initial container launch scripts.

