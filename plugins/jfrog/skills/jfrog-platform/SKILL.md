---
name: jfrog-platform
description: >
  JFrog Platform specialist for Artifactory, Xray, and Curation workflows.
  Use when user mentions JFrog, Artifactory, Xray, repositories, package
  vulnerabilities, curation status, artifact search, AQL, DevSecOps reports,
  license compliance, or software supply chain security. Use when user says
  "create a JFrog project", "check if package is safe", "what vulnerabilities
  does X have", "do we use package X", "create a repository", "check curation
  status", "find artifact X", "show security report", "check license for X",
  "list our projects", or "set up a new repo". Do NOT use for CI/CD build info,
  user/permission management, Xray policy configuration, or artifact promotion.
metadata:
  author: JFrog
  version: 3.0.0
  mcp-server: jfrog
  category: security
  tags: [artifactory, xray, curation, supply-chain-security, devops]
---

# JFrog Platform Skill

This skill provides workflow orchestration, domain terminology, and business logic for the JFrog Platform MCP tools. It does not repeat tool descriptions or parameter schemas - those are already provided by the MCP server. Instead, it teaches you *when and how to chain tools together* and the domain knowledge needed to use them effectively.

## Platform Context

The JFrog MCP Server is a remote, cloud-hosted server maintained by JFrog - no local installation or upgrades needed. It is available only for JFrog Cloud (SaaS) subscriptions; self-hosted instances are not supported.

**Platform URL format:** `https://<team>.jfrog.io`

**Authentication:** Handled entirely via OAuth. No API keys or tokens are required. If tools are unavailable or returning unauthorized errors, see Troubleshooting below.

## Domain Terminology

These concepts are critical for understanding user intent and selecting the right tools:

### Catalog vs. Artifactory - The Core Distinction

**JFrog Catalog** is a global intelligence database about the open-source ecosystem. It contains metadata on 12M+ public packages: vulnerabilities, licenses, dependencies, OpenSSF scores, and malicious package flags. It does not know what your organization uses. It answers: "What do we know about this package globally?"

**Artifactory** is the organization's private binary repository - the central source of truth for all packages actually used internally. It knows which packages your developers have downloaded, which versions live in which repositories, and what's flowing through your environment. It answers: "What do *we* use?"

**Routing rule for user queries:**
- "Is X safe?" / "What vulnerabilities does X have?" / "Tell me about package X" --> Use Catalog tools (public intelligence)
- "Do we use X?" / "What versions of X are in our repos?" --> Use Artifactory tools (internal data, identifiable by `rt_package` in the tool name)
- "Are we exposed to CVE-X?" / "Is this package safe AND do we use it?" --> Use BOTH - Catalog for vulnerability data, then Artifactory for internal exposure
- If the intent is ambiguous (e.g., "check log4j for me"), default to the full security assessment workflow below

**Important:** The Artifactory package tools are for dependency/package analysis only. They are NOT for build artifacts or CI/CD build information. If a user asks about builds, clarify this boundary.

### Curation Status

JFrog Curation is a gatekeeper that sits between the public open-source world and your Artifactory. It vets packages *before* they enter your development environment. A curation status check returns one of three values:

- **approved** - the package version has been vetted and is safe to use
- **blocked** - the package version has been rejected by policy; advise the user not to proceed
- **inconclusive** - the package version hasn't been fully evaluated yet; do NOT treat this as approved. Advise the user to check again later or escalate to their security team

### License Compliance

When checking a package via Catalog, license information is included in the metadata. Common concerns to flag for the user:
- **Copyleft licenses** (GPL, AGPL) may require the organization to open-source derivative work
- **Permissive licenses** (MIT, Apache-2.0, BSD) are generally safe for commercial use
- **No license / unknown** - flag this as a risk; the package may not be legally safe to use

Always surface the license type when reporting package metadata. Let the user or their legal team make the final call on compliance.

### AQL (Artifactory Query Language)

AQL is JFrog's query language for searching artifacts across Artifactory. It can search by name, path, repository, type, creation date, custom properties, and more. Use AQL-based search tools when users ask to find specific artifacts, check what versions are deployed, or audit what's in use across the organization.

### The Supply Chain Flow

```
Public Registry (npm, PyPI, Maven Central...)
    --> JFrog Catalog (global OSS intelligence)
    --> JFrog Curation (organizational gatekeeper: approved/blocked/inconclusive)
    --> Artifactory (now in your org's repos, used by your teams)
    --> JFrog Xray (continuous scanning of what's already inside)
```

### Repository Types

- **local** - hosts internal/proprietary packages your org produces
- **remote** - proxies and caches external public repositories (e.g., npmjs.org, Maven Central)
- **virtual** - aggregates local and remote repos under a single URL for developers
- **federated** - mirrors local repos across multiple JFrog instances for multi-site orgs

**Virtual repository constraint:** When creating a virtual repository, the default deployment repository MUST be included in the `repositories` list before it can be set as default. This is a platform requirement that is not enforced by the tool schema - always validate this during configuration.

## Workflows

### Workflow 1: Package Security Assessment

**Trigger:** "is package X safe?", "check package X", "should we use X?", "tell me about package X", "check license for X"

Run these steps in sequence without waiting between calls:

1. **Check package metadata** - look up the package in JFrog Catalog. Check: is it flagged as malicious? Note the license type and latest version. If flagged as malicious, STOP and warn the user immediately. Flag any license concerns (copyleft, unknown, no license).

2. **Check vulnerabilities** - get known vulnerabilities for the specific version in question (if no version specified, check the latest). Report severity breakdown and notable CVEs.

3. **Check curation status** - verify whether this version is approved, blocked, or inconclusive. If blocked, advise the user not to proceed.

4. **Check internal usage** - query Artifactory to see if the org already uses this package. If yes, identify which versions and which repositories are affected.

5. **Synthesize** - provide a clear risk summary combining all findings. Include: malicious status, license type, vulnerability count by severity, curation decision, and internal exposure footprint.

### Workflow 2: Vulnerability Investigation

**Trigger:** "what is CVE-X?", "are we affected by CVE-X?", "check vulnerability X"

1. **Get vulnerability details** - look up the CVE in Catalog. Report affected packages, versions, and severity.

2. **Check internal exposure** - for each affected package, query Artifactory to find which versions exist in the org. Cross-reference with the affected version ranges.

3. **Report exposure** - clearly state which internal repositories contain vulnerable versions, and identify safe versions to upgrade to.

### Workflow 3: DevSecOps Security Report

**Trigger:** "show security report", "what vulnerabilities do we have?", "security posture", "how are we doing on security?"

1. **Generate report** - use the DevSecOps reporting tools to get a real-time security overview.

2. **Highlight critical items** - focus on: critical/high severity CVEs, applicability status (whether a CVE is actually exploitable in context), and trends over time if available.

3. **Summarize actionably** - present findings grouped by severity, with clear next steps for the most critical issues.

### Workflow 4: Artifact Search

**Trigger:** "find artifact X", "search for X", "what versions of X are deployed?", "find all Docker images from last week"

1. **Build the query** - use AQL-based search tools to find artifacts matching the user's criteria. Common filters: name, path, repository, type, creation date, custom properties.

2. **Present results** - list matching artifacts with their repository, path, and relevant metadata.

3. **Offer follow-up** - suggest checking vulnerabilities or curation status for any packages found.

### Workflow 5: Project Setup

**Trigger:** "create a project", "set up a new JFrog project", "I need a new project"

Creation operations are additive and irreversible (there is no delete capability). Follow this flow strictly:

1. **Check existing state** - list current projects to show what already exists
2. **Gather requirements** through conversation:
   - Project name and key
   - What package types (npm, Maven, Docker, PyPI, etc.)
   - Whether repositories need Xray indexing
   - Repository types needed (local, remote, virtual)
3. **Check for conflicts** - verify the proposed project_key doesn't already exist
4. **Present the full plan** to the user with all configuration details
5. **Wait for explicit confirmation** - NEVER create without user approval
6. **Execute creation** - create the project first, then each repository. Project creation should happen only once, at the end of the conversation.

### Workflow 6: Internal Dependency Audit

**Trigger:** "what packages do we use?", "audit our dependencies", "what versions of X are in our repos?"

1. **Query internal packages** - look up the package in Artifactory to find all versions present in the org
2. **Cross-reference with Catalog** - for each version found, optionally check for known vulnerabilities and license info
3. **Report** - list versions, their repository locations, license types, and any known security issues

### Workflow 7: Repository Management

**Trigger:** "list repos", "create a repository", "what repositories do we have?"

**For listing:** retrieve and present the current repository list.

**For creation** (additive and irreversible):
1. List existing repos to show current state
2. Gather requirements: repo name, package type, repo type (local/remote/virtual/federated)
3. Ask whether Xray indexing should be enabled
4. If virtual: confirm the default deployment repo is included in the repositories list (see Virtual repository constraint above)
5. Present configuration and get explicit user confirmation
6. Execute creation

## Response Guidelines

- When reporting vulnerabilities, include: severity, affected versions, and safe upgrade targets when available
- When reporting package metadata, always include the license type
- When multiple tools are needed, chain them in sequence without asking the user to wait between calls
- If a tool returns empty or inconclusive results, state that clearly - do not speculate
- For security assessments, err on the side of caution and flag potential risks
- Format package and version information in clear, scannable tables when listing multiple items

## Scope Boundaries

This skill does NOT cover:
- CI/CD builds or build info (these are NOT packages - use build-focused tools instead)
- User management or permissions
- Xray policies, watches, or rules configuration
- Artifact promotion or distribution
- Access tokens or authentication management

If asked about something outside scope, clearly state what you can and cannot do.

## Troubleshooting

### MCP tools not available
- Verify an admin has enabled the MCP Server on the JFrog Platform: Administration > General > Settings > MCP Server > ON
- MCP is only available for JFrog Cloud (SaaS) subscriptions

### OAuth / "Unauthorized" errors
- Re-authenticate by restarting the MCP client (e.g., Cursor)
- Verify the platform URL is correct (`https://<team>.jfrog.io/mcp`)
- Check that your JFrog user has permissions to access the relevant projects and repositories

### Tools return empty results
- Verify the package name and ecosystem type are correct (npm vs pypi naming conventions differ)
- Check that you have access to the relevant projects/repositories
- For internal queries, the package may simply not exist in the organization's Artifactory

### Curation status is "inconclusive"
- The package version hasn't been fully evaluated yet
- Do NOT treat this as approved
- Advise the user to check again later or escalate to their security team

### Platform URL unknown
- Ask the JFrog administrator or check the organization's JFrog login URL
- Format is always `https://<team>.jfrog.io`
