## xx-3 - Secret Scanning with GitLeaks - Local Environment
Devops approaches security in a non-traditional way. Security of a devops workflow is not centralized. Securing a devops workflow is
often **distributed** among various tools and processes. So rather than doing it at one time and as late as possible which is the
traditional approach, devops takes that security validation of everything and distributes it throughout the workflow and put
multiple security checks where it's relevant.

So if it's application security, it puts that in application developer workflow and ... .

This is what gives us devsec ops.

Gitleaks scans latest source code as well as historical changes(commits).

## xx-4 - Pre-commit Hook for Secret Scanning & Integrating GitLeaks in CI Pipeline