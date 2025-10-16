# 5 - Vulnerability Scanning for Application Dependencies

## 1 - Software Composition Analysis - Security Issues in Application Dependencies (5 - Vulnerability Scanning for Application Dependencies)
Until now, we were testing our own code for security issues, but we also need to scan the deps as well.

We can't use SAST tools here, instead we use SCA, since we're dealing with the deps, not our actual code.