# 6 - Build a CD Pipeline

## 1 - Overview of a CICD Pipeline (6 - Build a CD Pipeline)

## 2 - Introduction to Security Layers for AWS Access
When we use docker hub image repository, by default, the credentials of the repo is the same as the account created it.
But when using aws ECR, the credentials of the image repo is different than our aws account. So aws has 2 security layers here.

So first we have to enter our aws account with aws user credentials(the main door), then we can access one of the services
using the service credentials.

So we need 2 sets of credentials.

Each aws user has 2 types of credentials:
- through UI
- through CLI using access key pairs

**Creating access keys for root user is a bad security practice.**

## 3 - Integrate CICD Pipeline with AWS ECR
In gitlab CI/CD variables, set `AWS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` with the val of newly created aws access key.
With these, we can authenticate to aws through CLI. Now we need credentials for authenticating in ECR.
The password for ecr is actually a temporary access token which will expire after some time, so we don't get a static password for ecr.

## 4 - Configure Application Deployment Environment on EC2 Server
### Create ec2 instance
Install docker on the ec2 instance.

### Install docker on ec2
First thing to do before ssh is to restrict the permissions on the private ssh key because otherwise aws will reject our request.
```shell
# This means zero permissions for anybody other than the owner and even owner has limited permission
chmod 400 <path to the downloaded ssh pem file>

# With aws ubuntu, we get a default `ubuntu` user
ssh -i <path to the pem file> ubuntu@<public ip of ec2>
```

Install docker:
```shell
# Always do this on fresh servers
sudo apt update

# Type `docker` to see suggested commands for installing docker
sudo apt install docker.io

# Now we have docker command available but only with `sudo`. We have to run it's command with sudo. We have to add the current user
# to docker user group.
sudo usermod -aG docker <current user like `ubuntu`>

# Now you can run docker commands without sudo

# Now we have to login from the ec2 into our ecr repo
# So we need to isntall aws CLI and set credentials of aws and default region and ... .
# So we should automate infra provisioning.
# Type `aws` to see the suggested commands for installing aws.
sudo apt install awscli

# set env vars that awscli expects:
export AWS_ACCESS_KEY_ID=
export AWS_ACCOUNT_ID=
export AWS_DEFAULT_REGION=
export AWS_SECRET_ACCESS_KEY=

# We dont need to specify region here, because it's gonna pickup the default region
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com

# Now the server can pull the image from ecr.
```

## 5 - Deploy Application to EC2 Server with Release Pipeline
### Add deploy job: configure ssh client
```shell
# pbcopy will copy the content to clipboard
cat <path to the ec2 ssh private key> | pbcopy
```

Then create a variable as SSH_PRIVATE_KEY on gitlab runner. Set it's type as file not text. After pasting the value, add an empty new line at the end. To make gitlab
recognize the file as it's correct format.

When we have `x || true` in shell, if the cmd `x` fails, || true will ignore the err.

### Access application in browser
First open the application port on the security group. Then access it in browser using <public ip addr of ec2 : port>

Instances > security > security group > edit inbound rules > add new inbound rule > port range: port 3000 > allow from anywhere.

## 6 - Configure Self-Managed GitLab Runner for Pipeline Jobs
We want to use an ec2 server for our **own** self-managed gitlab runner(instead of using a shared runner provided by gitlab).

We give ec2 instance an aws role that will be able to authenticate with aws account and access any services within aws that we will need inside the pipeline.
This means we don't need a separate aws **user credentials** because we're gonna use an aws **role**.

### Create ec2 instance for self-hosted runner
Launch a new ec2 instance.

Name: gitlab-runner

We need more resources for things like building docker images, choose at least t2.medium or better with t2.large

Storage: at least 16GB, for being safe: 20GB

Now we wanna turn this ec2 instance into a gitlab runner. First we have to install gitlab runner there and then register it with the project.

### Install & register gitlab runner
```shell
# limit the access to the ssh key file of the ec2 instance:
chmod 400 <path to download ssh key file>.pem

ssh -i <path to download ssh key file>.pem ubuntu@<instance ip>

sudo apt update
```

To install gitlab runner, in settings>CI/CD of gitlab > Runners > New project runner.

Then use `ec2, shell` as runner tags.

Now to install runner on the ec2:
```shell
sudo curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

sudo apt install gitlab-runner

# register runner with our project
sudo gitlab-runner register
  --url https://gitlab.com
  -- token <token> -- <token> is unique per runner. Whenever creating a new runner, it gets a token
```

![](img/6-6-1.png)

Now you should see the runner in `Assigned project runners`.

## 7 - Build Application Images on Self-Managed Runner, Leverage Docker Caching
### Run build job on self-managed runner to leverage caching
```shell
docker
sudo apt install docker.io
aws
sudo apt install awscli

# with docker, we need to add a user to docker group so that it can execute docker cmds without using `sudo`
# The `ubuntu` user needs to run docker commands:
sudo usermod -aG docker ubuntu

# Add gitlab runner user to the docker group, because gitlab runner user which was created when we installed gitlab runner,
# will need to run docker cmds:
sudo usermod -aG docker gitlab-runner
```

### docker image layer caching
Gonna do 2 improvements:
1. replace dind running env with executing docker **directly** on the server
2. when using docker or dind executors: in build_image job logs, it takes 7-8 mins because everytime, every single layer in the image is created from scratch. Because the job env
that runs the job is always a new docker container, so each time the job runs in a separate and isolated container. So there's no leftover from
prev job, so it starts from a clean state which has pros & cons. **Pro: We dont have any leftovers, Con: we can't leverage caching. We want to leverage caching
using shell executor.** This is because shell executor runs jobs directly on the server, **so the image stays on the server**.
So on the next run, it already has the img from prev build.

Docker by default caches each layer, saved on local filesystem

### install docker and aws on runner

### Adjust build_image job
We wanna run build_image job on our self runner instead of gitlab shared runners. **We can specify each job to run on which runner.**

### Troubleshooting tips
```shell
# Check available space
df -h

# Intermediary and dangling images can be deleted with: 
docker image prune
```

If we cancel a running job on our self-managed runner, and then retry, it would go into a pending state. The gitlab runner crashes when we cancel the job.
We can fix the job not being picked up by self-managed runner by restarting runner:
```shell
sudo gitlab-runner start
sudo gitlab-runner status

# Instead, you can run this. THe & doesnt block the terminal. The run cmd is faster because it proactively tells runner to go look if there are any jobs
# to be picked.
sudo gitlab-runner run &
```

### Docker using cache

### Free up disk space
Currently, the app ec2 server pulls the new images from ecr and it went out of space. So we need to free up space by removing dangling images:
```shell
docker rmi <img id>
```