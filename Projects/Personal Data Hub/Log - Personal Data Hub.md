
### Points of Friction
- Context switching: difficulty jumping back in due to unfamiliarity with systems
	- Need to document important processes and add helpers/shortcuts where applicable

---

#### [[2026-10-05]] - Getting started
- Configured and organized directory structure on home server
- Created required volumes and `docker-compose.yml` files for Windmill and OpenBAO
- Wrote reusable OpenBAO python package, to be shared by future jobs for access to OpenBAO for secrets
- Created a POC data-ingest-example repo to test github dependency integration with Windmill job
- Next steps
	- Need to confirm my understanding of the github dependency management
	- Create job for ingestion of real data
		- Start with letterboxd diary?
	- Solidify directory structure and organization
	- Write basic documentation on conventions and processes for creating and maintaining various pieces of infrastructure
		- Windmill
		- OpenBAO & Policies
		- Data pipeline jobs
	- Explore GHA for performing various tasks to help with deployments, etc.

[[2026-10-09]]

- Workstation setup
	- Browser
	- Obsidian
	- VS Code
		- Home Server connection
		- data-ingest-test
		- openbao_client
- **Current Blocker**
	- Need to get this Windmill script up and running. Something is wrong with the import, it doesn't seem to be resolving. Need to investigate further and get this up and running as a POC
	- SOLVED
		- Bad syntax in the import
			- Before: `openbao-client @ git+https://...`
			- After: `openbao-client@git+https://...`
- Next milestone 
	- After the above POC, create a managed job not written/maintained via the Windmill editor. This should be possible, just need to understand how it's all wired together

- After successful dependency resolution test, moving on to maintaining and deploying a job independent of the Windmill UI. 
	- Utilizing wmill cli for deployment
	- Later can automate via CI/CD
- 




