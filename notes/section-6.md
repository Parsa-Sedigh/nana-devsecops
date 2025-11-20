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
## 7 - Build Application Images on Self-Managed Runner, Leverage Docker Caching