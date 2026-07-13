# Handling Secrets

Secrets (passwords, API keys, tokens, certificates, connection strings) are the most dangerous things in your codebase that should never be in your codebase. A leaked secret is a direct path to your data, your infrastructure, and your users' trust. Protecting them is asset protection under [the standing What](WHAT-MATTERS-MOST.md#the-standing-what): a leak spends every currency it counts, at once, without ceiling and without undo.

This document states the rules that are not negotiable, the practices that are broadly proven, and the scenarios that need further decisions. It is language-agnostic: the principles apply regardless of stack.

## Rules

These are absolute. There are no exceptions, no "just this once," no "just for testing." Elsewhere these standards allow a rule to bend when it is visibly and defensibly in service of the principles ([What Matters Most § Bending the Rules](WHAT-MATTERS-MOST.md#bending-the-rules)). Not here. A bend is defensible only when a bad one can be caught and undone, and a leaked secret cannot be unleaked. No defense survives that challenge, so none is entertained.

**Never commit secrets to version control.** Not in source files, not in configuration, not in comments, not in branches you plan to delete. Git history is permanent: a secret that was committed and then removed is still in the history, recoverable by anyone with access to the repo. This applies equally to private repositories.

This rule targets secrets in readable form. Tools that commit *encrypted* secrets (SOPS, git-crypt, ansible-vault) are a defensible pattern with real adoption, but understand that they move the problem to key management rather than solving it. A configuration mistake or a leaked key re-creates the original exposure, now with the false confidence of "it's encrypted."

**Never log secrets.** Not at DEBUG level, not at any level. Secrets in logs migrate to dashboards, log aggregators, error tracking services, and screenshots in bug reports. Once a secret is in a log line, you have lost control of where it goes.

**Never pass secrets as command-line arguments.** Command-line arguments are visible in process listings (`ps`), shell history files, and process monitoring tools. Use environment variables or file-based injection instead.

**Never hardcode secrets.** Not in source code, not in configuration files checked into the repo, not in Dockerfiles, not in CI configuration. "Just for local development" is not an exception: hardcoded secrets migrate to production with alarming reliability.

**Never include secrets in error messages or user-facing output.** A stack trace that includes a connection string, an error dialog that echoes back an API key, a debug page that dumps the environment — all of these are leaks.

**Never treat `.gitignore` as a security boundary.** It prevents *accidental* commits and is a useful safety net, but it is trivially overridden with `git add -f`. Treat it as a reminder, not a guarantee.

## Practices

These are proven approaches. Your specific tooling choices may vary, but the patterns are well-established.

**Environment variables for application configuration.** The [twelve-factor app](https://12factor.net/config) pattern: secrets live in the environment, not in the repo. For local development, tools like `direnv` (with `.envrc` gitignored and `.envrc.sample` committed) provide a clean workflow. The sample file documents what variables are needed and what shape the values take, without containing real values.

**A secret manager for anything shared or production.** AWS Secrets Manager, HashiCorp Vault, 1Password, Azure Key Vault, or equivalent: a system designed for storing, accessing, and rotating secrets. Secrets are stored encrypted, access is auditable, and rotation is possible without redeploying code. The specific tool depends on your infrastructure; the principle is: use a purpose-built system, not a shared spreadsheet or a pinned message in a chat channel.

**Least privilege.** A secret should grant the minimum access needed for its purpose. A credential for a read-only service should not have write access. A token scoped to one API should not grant access to others. When creating credentials, start with the narrowest scope and widen only when a specific need demands it.

**Rotate compromised secrets immediately.** When you suspect a secret has been exposed (committed to a repo, logged, sent in a message), revoke it first, assess impact second. The window of exposure matters more than fully understanding the blast radius before acting, and revoking first is the right call even though it can interrupt legitimate consumers: a brief outage is cheaper than an open door. Then: create a new secret, update all legitimate consumers, and verify the old secret no longer grants access.

Rotation is the only real remedy for a secret that reached version control. History-rewriting tools (git filter-repo, BFG) can scrub your copy, but they cannot reach clones, forks, or forge caches. Scrub if you can, and treat the secret as public regardless.

**Pre-commit scanning.** `detect-secrets` (or equivalent) runs as the first hook in the pre-commit sequence (before linting, type checking, or tests). It catches secrets in staged changes before they enter history. These tools are not foolproof (they rely on pattern matching and entropy analysis), but they catch the most common mistakes and fail fast on the worst possible error. See [Python Standards § Pre-commit and Quality Gates](languages/python/STANDARDS.md#pre-commit-and-quality-gates) for the standard Python hook sequence.

This is the one gate where timing is everything. For every other check, CI catching what a skipped hook missed costs nothing: the defect stops before the shared branch, and a broken commit in history is harmless. A committed secret is different: the harm is the commit itself, and by the time CI's scan fires, history already has it. The pre-commit scan is the only gate that prevents; CI's scan is an alarm. When it sounds, you haven't been saved; you've been told to rotate.

**Separate secrets by environment.** Development, staging, and production must use different credentials. A development secret that works against production infrastructure is a leak waiting to happen.

## Scenarios Requiring Decisions

The rules and practices above are settled. The scenarios below are not: they require decisions about tooling, process, and acceptable trade-offs. Each is stated as a problem with the properties a good solution should have.

### Local Development

**The problem:** A new developer clones the repo and needs secrets to run the project (database credentials, API keys, service tokens). How do they get them?

**Properties of a good solution:**
* New developers can get working credentials without asking someone to look them up and paste them into a chat message
* The mechanism is documented in the README or CONTRIBUTING guide
* Credentials are scoped to the development environment, never production
* The process is the same every time, not dependent on who happens to be available to help

### CI/CD Pipelines

**The problem:** Automated builds, tests, and deployments need secrets (registry credentials, deployment keys, API tokens for external services). These must be available to the pipeline without being stored in the repo.

**Properties of a good solution:**
* Secrets are injected by the CI platform's secret management, not checked into pipeline configuration
* Access to CI secrets is auditable: you can see who added or changed a secret
* Secrets are scoped to the pipeline and environment that needs them (a staging deploy doesn't use production credentials)
* Rotation is possible without modifying pipeline configuration in the repo

### Deployed Services

**The problem:** A running application needs secrets at runtime (database connections, API keys, encryption keys). How are they delivered to the application?

**Properties of a good solution:**
* Secrets are injected at startup (via environment, mounted files, or a secret manager SDK), not baked into the deployment artifact
* The application never writes secrets to disk, logs, or telemetry
* Secrets can be rotated without redeploying the application, or at minimum, without rebuilding it
* Access is auditable: you can determine which services have access to which secrets

### Shared and Team Secrets

**The problem:** Some credentials are used by multiple people or services (a shared API key for a third-party service, a team-wide access token, a database credential used by several applications).

**Properties of a good solution:**
* The secret lives in a shared secret manager, not in someone's personal notes or a team chat channel
* Access is granted per-person or per-service, not by sharing the raw value
* When a team member leaves or a service is decommissioned, their access can be revoked without rotating the secret for everyone else
* There is a clear owner responsible for each shared secret

### One-Off Scripts and Tools

**The problem:** A quick utility script, a data migration, or a one-time report needs a database password or an API key. The temptation to hardcode is strongest here because the script is "temporary."

**Properties of a good solution:**
* Even temporary scripts read secrets from the environment or a secret manager, never from the source code
* If the script is committed to the repo (even to a `scripts/` directory), it follows the same rules as any other code
* If the script is truly one-off and never committed, it still does not hardcode secrets, because habits formed in "temporary" code migrate to production code

### Credentials for External Operators

**The problem:** Your deliverable is software that external operators (people outside your organization) run to interact with your infrastructure. They need credentials for your database, your cloud services, or your APIs. You do not control their environment, their security practices, or their credential storage.

This scenario is fundamentally different from the others because the secret crosses an organizational boundary. You cannot rely on your own infrastructure to broker access. You cannot audit how credentials are stored on their end. You cannot rotate secrets without coordinating with people you don't manage. A leak by any single operator can compromise shared infrastructure.

**Properties of a good solution:**
* Credentials are short-lived: issued on demand and expired automatically, not long-lived keys that persist indefinitely
* Each operator gets unique credentials, so a compromise can be contained and revoked without affecting others
* Credentials grant the minimum privilege needed for the operator's specific tasks
* Access is auditable: you can see which operator accessed what, and when
* Revocation is immediate and does not require the operator's cooperation
* The operator never needs to see, copy, or store raw infrastructure credentials (database passwords, AWS access keys); the application handles authentication transparently
