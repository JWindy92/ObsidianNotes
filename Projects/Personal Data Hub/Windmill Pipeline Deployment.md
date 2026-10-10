## Overview

A self-hosted framework for deploying and executing Python data pipelines using GitHub, Windmill, Docker, and persistent host storage.

### Architecture

- **GitHub —** `**letterboxd-pipeline**`**:** Reusable Python package and pipeline logic.
    
- **GitHub —** `**windmill-workspace**`**:** Windmill scripts and deployment configuration.
    
- **GitHub Actions:** Deploys workspace changes to Windmill.
    
- **Windmill server:** Manages scripts, jobs, and scheduling.
    
- **Windmill worker:** Resolves dependencies and executes Python jobs.
    
- **Host filesystem:** Stores persistent pipeline output.
    

Data flow:

`GitHub → GitHub Actions → Windmill Server → Windmill Worker → Host Storage`

## 1. Repository Structure

### Pipeline package repository

Example: `letterboxd-pipeline`

```
letterboxd-pipeline/
├── pyproject.toml
├── src/
│   └── letterboxd_pipeline/
│       ├── __init__.py
│       ├── rss.py
│       ├── export.py
│       └── rss_cli.py
└── tests/
```

Responsibilities:

- Fetch and parse source data.
    
- Transform and validate data.
    
- Export results to a specified location.
    
- Provide reusable functions and tests.
    
- Remain independent of Windmill-specific APIs.
    

### Windmill workspace repository

Example: `windmill-workspace`

```
windmill-workspace/
├── wmill.yaml
├── f/
│   ├── demos/
│   └── pipelines/
│       └── letterboxd_ingest/
│           └── run.py
└── .github/
    └── workflows/
        └── deploy.yml
```

Responsibilities:

- Define Windmill entrypoints.
    
- Declare external Python dependencies.
    
- Control which scripts are deployed.
    
- Keep orchestration separate from reusable pipeline logic.
    

## 2. Adding a Pipeline

### Step 1: Implement the pipeline package

Create or update a dedicated Python package repository.

- Put reusable logic in the package.
    
- Define explicit inputs and outputs.
    
- Add tests.
    
- Keep source-specific transformations separate from orchestration.
    

### Step 2: Create a Windmill entrypoint

Create a script at `f/pipelines/<pipeline_name>/run.py`.

Example:

```
# requirements:
# my-pipeline @ git+https://github.com/<owner>/my-pipeline.git

from my_pipeline import run_pipeline

def main():
    return run_pipeline(
        output_path="/data/pipelines/my_pipeline/raw/output.json"
    )
```

Replace the package name, repository URL, import, function, and output path with the actual implementation.

**Dependency rules:**

- Use the distribution name in the requirements declaration.
    
- Use the Python import name in `import` statements.
    
- Use the complete `git+https://` URL for Git dependencies.
    
- Ensure the repository has valid Python packaging metadata, normally `pyproject.toml`.
    
- Private repositories require Git authentication available to the Windmill worker.
    

### Step 3: Configure persistent storage

Host directory convention:

```
/srv/data/pipelines/
└── <pipeline_name>/
    ├── raw/
    ├── processed/
    └── archive/
```

The default Windmill worker mounts the host directory using Docker Compose:

```
volumes:
  - /srv/data/pipelines:/data/pipelines
```

Use the container path inside pipeline scripts:

`/data/pipelines/<pipeline_name>/...`

The corresponding host path is:

`/srv/data/pipelines/<pipeline_name>/...`

Create required directories on the host and ensure the worker has permission to write to them.

### Step 4: Include the script in workspace synchronization

Update `wmill.yaml`:

```
includes:
  - "f/demos/**"
  - "f/pipelines/<pipeline_name>/**"
```

Preserve existing includes that are still needed. Confirm the new script is within the configured synchronization scope.

### Step 5: Deploy through GitHub Actions

Commit and push changes to the configured deployment branch.

The deployment workflow:

1. Checks out the workspace repository.
    
2. Installs the Windmill CLI.
    
3. Authenticates using the `WMILL_TOKEN` GitHub secret.
    
4. Connects to the Windmill server.
    
5. Synchronizes included scripts.
    
6. Generates or updates script metadata and resolves dependencies.
    

Configuration:

- `WMILL_TOKEN`: GitHub Actions secret.
    
- `WMILL_BASE_URL`: Windmill server URL, if used by the workflow.
    
- Workspace: `home`.
    

Dependency resolution occurs during deployment and may fail before the script is executable.

### Step 6: Execute and verify

After deployment:

1. Open the script in Windmill.
    
2. Run it manually.
    
3. Inspect the job result and logs.
    
4. Verify that the output exists on the host.
    
5. Validate its format and contents.
    
6. Run it again to verify repeatability.
    

For the Letterboxd pipeline, the expected output is:

`/srv/data/pipelines/letterboxd/raw/activity.json`

## 3. Infrastructure Conventions

### Docker

- Keep the Windmill server and workers on compatible, matching versions.
    
- Use the same resolved `${WM_IMAGE}` value for the server and both workers.
    
- Avoid mixing a pinned image digest with a moving `main` tag.
    
- Mount persistent output storage in the worker that executes the script.
    
- Do not delete the PostgreSQL volume when troubleshooting worker or deployment problems.
    

### GitHub Actions

- Treat the workspace repository as the source of truth for scripts.
    
- Store tokens in GitHub Secrets, not in the repository.
    
- Deploy through the existing workflow rather than editing scripts manually in Windmill.
    
- Review deployment logs for synchronization and dependency-resolution errors.
    

### Storage

- Use `/srv/data/pipelines/` for persistent pipeline data.
    
- Separate raw, processed, and archived outputs where appropriate.
    
- Avoid storing durable outputs only inside a container's writable filesystem.
    
- Use predictable, pipeline-specific directory names.
    

## 4. Troubleshooting

|Symptom|Likely cause|First check|
|---|---|---|
|Dependency cannot be resolved|Invalid package name or registry-only requirement|Git URL in the requirements declaration|
|Lockfile generation fails|Invalid package metadata, Git access, or dependency conflict|Worker logs and `pyproject.toml`|
|Dependency job remains queued|Worker unavailable or unhealthy|`docker compose ps` and worker logs|
|Worker repeatedly exits|Image-version or migration mismatch|Compare resolved Windmill image versions|
|Script succeeds but file is missing|Incorrect path or missing mount|Compare container and host paths|
|Permission denied when writing|Host directory permissions|Inspect `/srv/data/pipelines/`|
|Script is not deployed|Script excluded from synchronization|Check `wmill.yaml`|

## 5. Current Implementation Status

- Reusable Letterboxd Python package
    
- Windmill workspace repository
    
- GitHub Actions deployment workflow
    
- Git dependency resolution
    
- Successful Windmill job execution
    
- Persistent output at `/srv/data/pipelines/letterboxd/raw/activity.json`
    

## 6. Potential Improvements

- Create a standard pipeline template.
    
- Establish consistent logging and job return values.
    
- Add input/output validation and automated tests.
    
- Establish retry and failure-notification conventions.
    
- Document scheduling conventions.
    
- Define retention and archival policies.
    
- Standardize configuration for non-secret settings.
    
- Integrate OpenBao only for pipelines that require secrets.
    
- Monitor failed jobs and stale outputs.
    

## Design Principles

1. **Separate logic from orchestration.** Python packages should be reusable outside Windmill.
    
2. **Treat Git as the source of truth.** Deploy scripts through version control.
    
3. **Make persistence explicit.** Write durable output to mounted host storage.
    
4. **Keep infrastructure simple.** Introduce shared frameworks when repeated needs justify them.
    
5. **Make failures observable.** Log enough information to diagnose dependency, execution, and storage problems.
    
6. **Use secrets only when necessary.** Public-data pipelines should not require OpenBao merely because it is available.