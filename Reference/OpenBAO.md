# OpenBao — Basic Operations

> [!NOTE]
> OpenBao is running in Docker with HTTP internally (`tls_disable = true`).
> The CLI therefore uses:
>
> `BAO_ADDR=http://127.0.0.1:8200`
>
> The commands below assume you are **inside the OpenBao container**, unless otherwise noted.

## Enter the OpenBao container

From the host:

    docker compose exec \
      -e BAO_ADDR=http://127.0.0.1:8200 \
      openbao \
      sh

Once inside:

    bao status
    bao login
    bao ...

Exit:

    exit

---

# Initial Setup

## Initialize OpenBao

Only do this **once** for a new OpenBao data store:

    bao operator init

This generates:

- Unseal keys
- Initial root token

> [!WARNING]
> Store the unseal keys and root token securely.
>
> Do **not** put them in Git, Docker Compose, `.env`, Windmill, or application configuration.

## Unseal OpenBao

Check status:

    bao status

If:

    Sealed    true

Run:

    bao operator unseal

Enter one unseal key.

Repeat with additional **different** unseal keys until:

    Sealed    false

For example, if the threshold is 3 of 5:

    Key 1 ─┐
    Key 2 ─┼─> OpenBao ─> UNSEALED
    Key 3 ─┘

### After a server reboot

OpenBao will normally start sealed again:

    bao status
    bao operator unseal
    bao operator unseal
    bao operator unseal

Repeat until `Sealed` is `false`.

---

# Authentication

## Log in

    bao login

Enter the appropriate token when prompted.

### Root token

The initial root token is for **administration only**.

Do not give it to Windmill or application code.

## Log out

    bao logout

---

# Secrets

## List secret engines

    bao secrets list

Typical output:

    Path          Type
    ----          ----
    cubbyhole/    cubbyhole
    identity/     identity
    secret/       kv
    sys/          system

### What the paths mean

| Path | Purpose |
|---|---|
| `secret/` | Application secrets |
| `cubbyhole/` | Private storage associated with a specific token |
| `identity/` | Users, groups, identities, and authentication relationships |
| `sys/` | OpenBao administration/control API |

For this setup, **`secret/` is where application credentials live**.

## Enable KV v2

For a new OpenBao installation:

    bao secrets enable -path=secret kv-v2

This creates the `secret/` KV secrets engine.

## Store a secret

Example:

    bao kv put secret/data_ingest \
      api_key="test-api-key-123"

With KV v2, OpenBao internally displays the underlying path as:

    secret/data/data_ingest

This is normal. The CLI abstracts the KV v2 `/data/` API path.

## Read a secret

    bao kv get secret/data_ingest

Example:

    ===== Data =====
    Key         Value
    ---         -----
    api_key     test-api-key-123

## Delete a secret

    bao kv delete secret/data_ingest

KV v2 uses versioning, so this is a soft delete of the current version.

---

# Policies

Policies define **what an authenticated client is allowed to do**.

## Windmill read-only policy

Create this file on the host:

    openbao/config/windmill.hcl

Contents:

    path "secret/data/*" {
      capabilities = ["read"]
    }

    path "secret/metadata/*" {
      capabilities = ["read"]
    }

This allows:

    Read secrets       ✓
    Read metadata      ✓
    Write secrets      ✗
    Delete secrets     ✗
    Administer Bao     ✗

## Create/update a policy

From inside the OpenBao container:

    bao policy write windmill /openbao/config/windmill.hcl

## Read a policy

    bao policy read windmill

## List policies

    bao policy list

---

# Application Tokens

## Create a token for a policy

    bao token create -policy=windmill

Example output:

    token           hvs.xxxxxxxxxxxxxxxxx
    token_accessor  ...
    token_duration  768h
    token_policies  ["default" "windmill"]

The **token** is what Windmill will use.

Windmill does **not** use:

- The OpenBao root token
- The OpenBao unseal keys

The credential hierarchy is:

    YOU
    │
    └── Root token
          │
          └── Administer OpenBao

    WINDMILL
    │
    └── Windmill token
          │
          └── windmill policy
                │
                └── Read permitted secrets

---

# Testing an Application Token

Authenticate using the application token:

    bao login

Enter the Windmill token.

Test allowed access:

    bao kv get secret/data_ingest

This should succeed.

Test prohibited access:

    bao kv put secret/data_ingest \
      api_key="this-should-fail"

This should return a permission-denied error.

This verifies that the Windmill token has read-only access.

---

# Token Expiration

Application tokens have a lifetime.

Our initial Windmill token was created with:

    768h

This is approximately:

    32 days

## Check token information

While authenticated with the token:

    bao token lookup

Look for:

- `creation_time`
- `expire_time`
- `ttl`
- `policies`

## When the Windmill token expires

The simple approach for this home-server setup is:

### 1. Authenticate as an administrator

    bao login

Use the root/admin credential.

### 2. Create a replacement token

    bao token create -policy=windmill

Save the new token securely.

### 3. Update the credential in Windmill

Replace the expired token with the new token.

### 4. Test Windmill

Run a script that retrieves a secret from OpenBao.

### 5. Revoke the old token

Only after confirming the new token works:

    bao token revoke <OLD_TOKEN>

> [!IMPORTANT]
> Never revoke the old token before confirming the replacement works.

## Future improvement

Manual token rotation is acceptable while learning and for a small home-server setup.

For a more mature deployment, consider machine authentication such as:

- AppRole
- OIDC/JWT
- Short-lived tokens
- Automated token renewal

The architectural goal is:

> Windmill should authenticate to OpenBao using a machine identity, not the OpenBao root credential.

---

# OpenBao Health / Status

    bao status

Useful fields include:

- `Initialized`
- `Sealed`
- `Storage Type`
- `HA Enabled`
- `Cluster Name`

## Raft status

    bao operator raft list-peers

Our current setup is a single-node Raft cluster.

---

# Network Access

From inside the OpenBao container:

    http://127.0.0.1:8200

From another container on the `orchestration` Docker network:

    http://openbao:8200

Windmill workers therefore access OpenBao using:

    http://openbao:8200

> [!IMPORTANT]
> Do not use `127.0.0.1` from the Windmill container.
>
> Inside a container, `127.0.0.1` refers to that container itself.

---

# Current Architecture

    GitHub
      │
      ├── Pipeline repositories
      └── Shared OpenBao client
                │
                ▼
           ┌───────────┐
           │ Windmill  │
           │           │
           │ Worker    │
           └─────┬─────┘
                 │
          Windmill token
                 │
                 ▼
           ┌───────────┐
           │ OpenBao   │
           │           │
           │ secret/   │
           │  data_ingest
           │    api_key
           └───────────┘

    Root token
        │
        └── Human administration only

    Unseal keys
        │
        └── Unlock OpenBao after startup/restart

## Important Credentials

| Credential | Used by | Purpose |
|---|---|---|
| Unseal keys | Administrator | Unseal OpenBao |
| Root token | Administrator | Full OpenBao administration |
| Windmill token | Windmill | Read permitted application secrets |

> [!WARNING]
> Never give applications the OpenBao root token or unseal keys.
