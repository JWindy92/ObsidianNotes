---
date: 2026-10-13
tags:
  - documentation
---
> [!note] Self Hosted Runner notes
> - Separate CI from deployment. A job that builds and tests an image shouldn't automatically have the same permissions as one that deploys to your server.
> - All repositories don't necessarily need the runner. 
> 	- Build Docker images on GitHub-hosted runners when possible
> 	- Reserve home server for jobs that need local resources or deployment access


### General Migration Steps
- Creating a new org is pretty simple, choose a name, default settings were fine for me
- Migrating the runner
	- Need a new PAT at the org level, created with Read/Write access for Self-Hosted runners
		- The PAT is simply used to request a registration token, so the token doesn't need repository access. 
		- These permissions simply allow it to request the registration token