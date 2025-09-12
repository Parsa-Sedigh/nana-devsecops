# 3 - Application Vulnerability Scanning

## xx-3 - Secret Scanning with GitLeaks - Local Environment
Devops approaches security in a non-traditional way. Security of a devops workflow is not centralized. Securing a devops workflow is
often **distributed** among various tools and processes. So rather than doing it at one time and as late as possible which is the
traditional approach, devops takes that security validation of everything and distributes it throughout the workflow and put
multiple security checks where it's relevant.

So if it's application security, it puts that in application developer workflow and ... .

This is what gives us devsec ops.

Gitleaks scans latest source code as well as historical changes(commits).

## xx-4 - Pre-commit Hook for Secret Scanning & Integrating GitLeaks in CI Pipeline
The gitlab runners could have linux, windows and each job could be assigned to a completely different runner each time.

Now when you run the tools using their docker images:
- you don't care what OS the runner has
- you make the pipeline generic and CI/CD platform-independent(github actions, jenkins or ...)

The ideal approach is to do the scan before developer pushes the code. So we prevent any security issues from ending up in the
git history. We can check this pre-commit.

To create a pre-commit git hook:
1. create `pre-commit` file in .git/hooks dir
2. make that file executable: `chmod +x .git/hooks/pre-commit`

Note that devs can still bypass this hook. So we need to put these checks at the pipeline as well.

With this, we can avoid security issue from landing in the code in an earlier step.

NOTE: We want to shift security checks as early as possible in the developer workflow.

## xx-5 - False Positives & Fixing Security Vulnerabilities
- False positive: The tool thinks there's a problem, but really it is not.
- False negative: the tool doesn't catch the problem.

We don't want to fail the build in gitleaks job because of false-positives, so add `allow_failure: true` in the ci.

To set pipeline ci/cd vars, go to settings>ci/cd>variables.

When you leak some secret and later find out about it, quickly change that secret and make sure to not leak it again by removing it completely.

## xx-6 - Integrate SAST Scans in Release Pipeline
We scanned the code for hard-coded credentials, but is the code itself secure?

We validate the code for security vulnerabilities using SAST(static application security testing).

Nodejs has njsscan tool.

Whenever gitlab starts a job, it mounts the entire application folder to the entrypoint of the container used for executing the job.

We can fail the build based on severity of security issues.

Oftentimes we wanna combine multiple tools for security scanning to make sure we don't miss any issues.