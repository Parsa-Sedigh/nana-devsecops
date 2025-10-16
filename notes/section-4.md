# 4 - Vulnerability Management and Remediation
## xx-1 - Generate Security Scanning Reports (4 - Vulnerability Management and Remediation)
Use "job artifacts" to export & save security report files.

## 2 - Introduction to DefectDojo, Managing Security Findings, CWEs
Pull defect dojo image.

## 3 - Automate Uploading Security Scan Results to DefectDojo
We write a python script to upload various scan reports to defectdojo.

When you create artifacts in gitlab pipeline, it saves the artifacts and in the following jobs, it automatically downloads any artifacts
from all the previous jobs. So we have access to all the prev artifacts in our curr job.

In gitlab, jobs in the same stage, run in parallel.

## 4 - Fix Security Issues Discovered in the DevSecOps Pipeline
Do not fail the pipeline until the security analysis tools are set up correctly and we have automatic uploading of the scannings and
have the dashboards available for devs.

Use parameterized vals or placeholders in sql queries to mitigate sql injection.