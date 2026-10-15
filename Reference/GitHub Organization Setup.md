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
- Ran into a few issues with the `data-dashboard` deployment workflow. I don't believe this was working to begin with. Remediation was multi-step but all related to the internal docker/docker-compose installation and config
	- missing `docker` & `docker-compose`
	- had to sync docker user id with system id for socket access
	- added bind mounts for the docker socket and `data-dashboard` service directory