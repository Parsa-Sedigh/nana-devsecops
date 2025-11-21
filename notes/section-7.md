# Section 7 - Image Scanning - Build Secure Docker Images

## 1 - Overview of Image Security (7 - Image Scanning - Build Secure Docker Images)
Security is layered. There is a possibility of exploitation on every level.

We include the building docker img in CD.

Security of CD includes:
- docker img security
- aws security
  - secure deployment to aws
  - secure aws account

## 2 - Configure Automated Security Scanning in Application Image
We may have a secure application code, by doing sanitizing user input, avoiding injection attacks and ..., but we can still build an insecure
docker img which will allow hacking through the container. So app is secure but the app runtime env(container) may not be.

Docker itself has docker scout for scanning images.

**Q: We use docker images to actually execute the jobs. Because we can put whatever tools we want to use, in that image, otherwise, we'd have to
SSH into the server that is executing the job and install those tools ourselves.**

### Add image scanning job
We use **trivy**.

We want to scan the img after img was built, but before it gets pushed to the img repo. So if it has security issues, we can avoid pushing it. 
But for now, we allow pushing the img to ecr to do some checks there as well, and in the next job(trivy) pull it from ecr and scan it.
Kinda dumb!

Trivy job is not a proper use case for shell executor, instead we use docker executor. Because we dont want to dependent on the runner config and also
in this job, we need tools that are too specific to be installed on the runner. So we use docker executor and we use the shared runner, because
we dont need the specific tools installed on our self-managed runner(which makes us to use shell executor).

**If we need tools that are used in multiple jobs, we can install them directly on the runner, but that requires us to use the
shell executor (or another persistent executor like virtual machine or Parallels). With the Docker executor,
tools cannot be installed permanently on the runner host because each job runs in an ephemeral container.**

Some security tools won't fail by default when they find an issue, like trivy.

3 - Analyze & Fix Security Issues from Findings in Application Image

4 - Automate Uploading Image Scanning Results in DefectDojo

5 - Docker Security Best Practices

6 - Configure Automated Image Security Scanning in ECR Image Repository

7 - Overview of Automated Application Code and Image Scanning Steps