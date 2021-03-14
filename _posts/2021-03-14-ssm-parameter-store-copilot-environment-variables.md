---
layout: post
title: Store Environment Variables Across Different Environments (Test/Stage/Production) with SSM Parameter Store for Copilot
---

In the past, I've managed my ECS and Fargate deployments with the AWS and ECS CLI. Being a good record keeper, I try to avoid clicking randomly at the UI until things work (which can be surprisingly effective if you don't care to ever reproduce your steps).

I've also written Node and Python CLI apps to do a bunch of this orchestration. These apps start to get amazingly complex very quickly.

Well lucky for us, AWS released Copilot. Copilot replaces some of the scripts and apps I've built to try to manage these things. I found it after looking for latest best practices on deploying staging environments. Copilot has the environement promotion pathway, which can be hooked up to Git commits, giving us automated environemnt deployments into Fargate. Joy.

It's still missing quite a bit to fully manage an enviroment. But I'm currently using it as a nice starting point. I spin up the base environemnent then manually add pieces on as needed. They plan to support more things, like RDS, in the future and hopefully I'll be able to bring those in as they come.

I was surprised to find a critical detail missing from the Copilot docs. It includes some great information, and one page with [best practices on secrets management](https://aws.github.io/copilot-cli/docs/developing/secrets/). I need to manage secrets as environment variables which differ per environment.

The solution is to use SSM Parameter Store, with [parameter labels](https://aws.amazon.com/blogs/mt/use-parameter-labels-for-easy-configuration-update-across-environments/), and map things yourself between Copilot and SSM.

Here is an example scenario.

```
My app: some-app
My environemnts: 'test', 'staging', 'production'
Env variable: DATABASE_PASSWORD
```

You will first want to add your environment variable secrets as SSM paramers to AWS.

For your test environemnt:
```
$ aws ssm put-parameter \
    --name /copilot/applications/some-app/environments/test/database_password \
    --value 'reallygoodpassword' \
    --type SecureString \
    --tags Key=copilot-environment,Value=test Key=copilot-application,Value=some-app
```

For your production environemnt:
```
$ aws ssm put-parameter \
    --name /copilot/applications/some-app/environments/production/database_password \
    --value 'productionqualitypassword' \
    --type SecureString \
    --tags Key=copilot-environment,Value=production Key=copilot-application,Value=some-app
```

And then in your manifest, you map the environment variable you want exposed to your ECS instances to the SSM parameter key.

```
secrets:
  DATABASE_PASSWORD: /copilot/applications/some-app/environments/test/database_password

# You can override any of the values defined above by environment.
environments:
  production:
    secrets:
      DJANGO_SETTINGS_MODULE: /copilot/applications/some-app/environments/production/database_password
```

You can use whatever string you would like as the key in SSM, I liked to follow the pattern genereated by Copilot for the SSM parameters it created automatically.

Now when you look at your container through the AWS console, you will not see these secrets exposed or mapped. You will see the environment variables pointing at the SSM keys. This makes it difficult to know if things are working from the outside and hard to debug. But they are mapped in the running container when your application goes to access them.
      
