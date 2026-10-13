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


