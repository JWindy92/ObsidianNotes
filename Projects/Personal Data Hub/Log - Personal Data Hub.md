
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
- Before continuing, it would be nice to have some kind of automation around deployment. I am going to explore hosting a self-hosted GHA runner on my server to handle automated builds and deployments

[[2026-10-10]]

- Today's Objective: Stand up GHA runner on server. Ensure it can build and deploy the windmill-workspace repo for automated deployments
- With help of ChatGPT was able to reverse engineer old GHA runner experiment and get it up and running, was able to get up and running pretty quick

```
gha-self-hosted-runner  | Current runner version: '2.338.0'
gha-self-hosted-runner  | 2026-10-10 12:51:03Z: Listening for Jobs
gha-self-hosted-runner  | 2026-10-10 12:54:38Z: Running job: test
gha-self-hosted-runner  | 2026-10-10 12:54:41Z: Job test completed with result: Succeeded
```

- After getting self-hosted runner set up, we created a deployment workflow for the `windmill-workspace` repository that deploys new scripts to our windmill instance automatically
- Working on adapting old letterboxd script to a data pipeline
	- BLOCKER: requests seem to be blocked by CloudFlare, I suspect they locked this method down. Exploring alternative options, such as Playwright, to simulate normal browser activity

- Turns out LB exposes an RSS endpoint for every user. It only provides the last 50 entries, but is a good solution for continuous updates.
- Todo:
	- [x] #task Display data on basic [[streamlit]] dashboard ✅ 2026-10-10
	- [ ] #task/letterboxd Write letterboxd data to DB rather than local JSON 🔽 
	- [ ] #task/letterboxd Add script to ingest historical backfill from manually downloaded file 🔼 
- Progress
	- Got GHA runner up and running for windmill-workspace repo
	- Successfully built new version of letterboxd extract process that uses the public RSS feed of my profile to return movie data
		- Currently saves to plain json file on server.
	- Created simple [[Streamlit]] dashboard to display data returned from letterboxd


[[2026-10-11]]
- Ended yesterday having gotten a good amount done. Letterboxd RSS extract up and running on an hourly schedule, populating a dashboard on my home server
- **Minor Issue**: GHA self-hosted runners cannot run at the account level. Requires creation of GH Organization to assign a single runner to multiple repos
	- [ ] #task Create personal GitHub organization 🆔 zusn49 🔼 
	- [ ] #task Migrate repos to new organization 🆔 licu6q ⛔ zusn49 🔼 
	- [ ] #task Reconfigure GHA runner to point to GitHub org ⛔ licu6q 🔼 
- Today I'll export my full diary manually and then create a script to add this to the dataset utilized by the dashboard 
- From there I'm not sure what I want to take up yet, some possibilities
	- Add more advanced filtering/querying of letterboxd data for specific use cases
	- I started looking into the feasibility of ingesting F1 data for high performance telemetry processing. Would be a very cool project, but complex
		- May want to have our data ingestion workflow more solidified before jumping into the deep end with something so complicated
	- Simple sports data. Already did some of this in `stats-n-data` repo, could easily migrate to a pipeline and get this running on a schedule
	- Explore Bark & other forms of custom notifications
	- Financial data (SimpleFIn)
	- Calendar integrations
	- Google Sheets integration
	- [ ] #task Document data ingestion ideas somewhere to keep a running list active
		- Automate with tag query??

[[2026-10-12]]
- Goals for today
	- [x] Add Simplefin Ingestion pipeline
	- [x] Display Account balances on dashboard (simple)
	- [ ] Ingest full historical Letterboxd diary
	- [ ] Document basic pipeline integration
- [ ] #task Create pipeline boilerplate and auto initialization script
- Got another pipeline up and running, still very basic. Currently just very basic pulls and storing data in local flat files.
	- Currently no data is growing in size, all pulls are overwrites. When I get full historical letterboxd data ingested, I'll likely want to begin thinking about getting the database writes working.