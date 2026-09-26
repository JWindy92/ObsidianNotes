```yaml

project:
  name: data-proj
  stage: poc
  purpose: >
    Validate that a Windmill Python job can authenticate to OpenBao using
    AppRole, retrieve a scoped secret, and use it without exposing the secret
    in logs or job results.

secret:
  purpose: poc-test-value
  engine:
    type: kv
    version: 2
    mount: kv
  logical_path: jobs/data-proj/poc
  policy_api_path: kv/data/jobs/data-proj/poc
  fields:
    username: poc-user
    api_key: replace-with-random-poc-value
  access:
    capabilities:
      - read
    allow_list: false
    allow_write: false
    allow_delete: false

openbao:
  address:
    internal: https://openbao:8200
  tls:
    enabled: true
    verify: true
    ca_certificate_path: /run/secrets/openbao-ca.crt

  auth:
    method: approle
    mount: approle
    role_name: windmill-data-proj-poc
    policy_name: windmill-data-proj-poc

  issued_token:
    ttl: 30m
    max_ttl: 60m
    renewable: false

  secret_id:
    ttl: 24h
    usage_limit: 0
    rotation:
      strategy: manual
      rotate_before_expiry: true

windmill:
  workspace: poc
  folder: data-proj
  worker_group: data-jobs-poc

  credentials:
    storage: restricted-windmill-resource
    resource_name: openbao-data-proj-poc
    expose_globally: false
    fields:
      address: https://openbao:8200
      role_id: replace-with-role-id
      secret_id: replace-with-secret-id
      auth_mount: approle
      kv_mount: kv
      kv_version: 2
      ca_certificate_path: /run/secrets/openbao-ca.crt

  execution:
    mode: manual
    schedule_enabled: false
    timeout_seconds: 300
    retries: 0
    concurrent_runs: 1
    maximum_result_bytes: 1048576

python_library:
  name: homeops-openbao
  initial_version: 0.1.0
  python:
    minimum_version: "3.11"

  distribution:
    method: git
    revision: v0.1.0
    mutable_branch_allowed: false

  configuration:
    openbao_address_env: OPENBAO_ADDR
    role_id_env: OPENBAO_ROLE_ID
    secret_id_env: OPENBAO_SECRET_ID
    auth_mount_env: OPENBAO_AUTH_MOUNT
    kv_mount_env: OPENBAO_KV_MOUNT
    kv_version_env: OPENBAO_KV_VERSION
    ca_certificate_env: OPENBAO_CA_CERT
    timeout_env: OPENBAO_TIMEOUT_SECONDS

  defaults:
    auth_mount: approle
    kv_mount: kv
    kv_version: 2
    timeout_seconds: 10
    tls_verification: true
    token_caching: execution-only

  required_features:
    - approle-authentication
    - kv-v2-read
    - tls-verification
    - connection-timeout
    - safe-error-messages

data_project:
  name: data-proj
  entrypoint: data_proj.main.main
  secret_path: jobs/data-proj/poc

  poc_behavior:
    description: >
      Read the POC secret and return a safe result proving that the expected
      fields were available. Do not return either secret value.
    expected_result:
      status: success
      secret_retrieved: true
      fields_present:
        - username
        - api_key

  prohibited_output:
    - openbao-client-token
    - role-id
    - secret-id
    - username-value
    - api-key-value
    - environment-dump
    - authorization-headers

networking:
  docker_network: services
  openbao_hostname: openbao
  openbao_port: 8200
  publish_openbao_to_lan: false
  allow_windmill_to_openbao: true
  allow_openbao_to_internet: false

host:
  root: /srv
  paths:
    infrastructure: /srv/infrastructure
    openbao_compose: /srv/infrastructure/compose/openbao
    windmill_compose: /srv/infrastructure/compose/windmill
    data_project: /srv/jobs/data-proj
    python_library: /srv/libraries/homeops-openbao
    openbao_data: /srv/data/openbao
    windmill_data: /srv/data/windmill
    runtime: /srv/runtime
    secrets: /srv/secrets
    backups: /srv/backups

validation:
  success_criteria:
    - openbao-is-initialized-and-unsealed
    - openbao-health-check-passes
    - approle-login-returns-short-lived-token
    - approle-can-read-poc-secret
    - approle-cannot-read-unrelated-secret
    - windmill-can-install-pinned-library-version
    - windmill-job-can-retrieve-poc-secret
    - job-result-does-not-contain-secret-values
    - job-logs-do-not-contain-secret-values
    - invalid-secret-id-causes-safe-failure
    - sealed-openbao-causes-safe-failure

  intentionally_deferred:
    - production-business-logic
    - scheduled-execution
    - automatic-secret-id-rotation
    - response-wrapped-secret-ids
    - jwt-workload-identity
    - token-renewal
    - high-availability-openbao
    - automatic-unseal
    - private-python-package-registry
    - production-monitoring
    - production-backups
```

