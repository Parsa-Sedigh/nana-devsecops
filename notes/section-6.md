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
## 5 - Deploy Application to EC2 Server with Release Pipeline
## 6 - Configure Self-Managed GitLab Runner for Pipeline Jobs
## 7 - Build Application Images on Self-Managed Runner, Leverage Docker Caching