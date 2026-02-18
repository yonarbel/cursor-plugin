---
name: jfrog-platform
description: Interact with the JFrog Platform for artifact management, repository operations, security scanning, and DevSecOps workflows. Use when the user mentions JFrog, Artifactory, Xray, artifact search, repository setup, vulnerability scanning, package curation, or supply-chain security.
---

# JFrog Platform

The JFrog Platform is the user's software supply chain management system. It provides artifact management (Artifactory), security scanning (Xray), package curation, and DevSecOps capabilities. The JFrog MCP Server connects your AI assistant directly to the platform.

**Platform URL format:** `https://<team>.jfrog.io`

## Prerequisites

Before using JFrog MCP tools, the following must be in place:

### 1. Admin Enables MCP on the JFrog Platform

An administrator must enable the JFrog MCP Server on the JFrog Platform Deployment (JPD):

1. Log in to the JFrog Platform as an **admin user**.
2. Navigate to **Administration** > **General** > **Settings**.
3. Locate the **MCP Server** section and toggle it **ON**.
4. Save the configuration.

> **Important:** The JFrog MCP Server is available only for **JFrog Cloud (SaaS)** subscriptions. Self-hosted instances are not supported.

### 2. Configure MCP Client (Cursor)

After the admin enables MCP, each developer adds the server to their Cursor configuration:

1. Open Cursor Settings → **Tools & MCP**.
2. Add a new MCP server with this configuration:

```json
{
  "mcpServers": {
    "jfrog": {
      "url": "https://<JFROG_PLATFORM_URL>/mcp"
    }
  }
}
```

Replace `<JFROG_PLATFORM_URL>` with your organization's JFrog Platform URL (e.g., `mycompany.jfrog.io`).

3. Save and **restart Cursor**.
4. An **OAuth authorization window** opens in your browser — follow the prompts to authorize access.

No API keys or tokens are required. Authentication is handled entirely via OAuth.

## MCP Capabilities

The JFrog MCP Server is a **remote, cloud-hosted** server maintained by JFrog. It is always up-to-date with the latest tools — no local installation or upgrades needed.

### Resource Management

- Create and manage **projects**
- Create and configure **repositories** (local, remote, virtual)
- Manage repository permissions and assignments

### Artifact Search

- Execute **AQL (Artifactory Query Language)** queries to find artifacts
- Search across repositories, builds, and properties
- Filter by name, path, type, creation date, and custom properties

### Catalog & Curation

- Query the **JFrog Catalog** for package information and versions
- Check **vulnerability status** of open-source packages
- Verify **curation status** — whether a package is approved for use
- Check **license compliance** for dependencies

### Security Monitoring

- Generate real-time **DevSecOps reports**
- Get vulnerability details including **severity** and **applicability**
- Monitor **critical CVEs** affecting your organization
- Track security posture across projects and repositories

## Usage Patterns

### Creating a Project with Repositories

When a user asks to create a new project (e.g., "Create a project called my-app with Docker and npm support"), the MCP server handles all the complexity — creating the project, setting up local/remote/virtual repositories for each package type, and assigning them to the project.

### Checking Dependency Security

When reviewing code or adding dependencies, query the JFrog Catalog to:
1. Check if the package version has known vulnerabilities
2. Verify the package is approved (not blocked by curation policies)
3. Review license compatibility

### Searching for Artifacts

Use AQL-based search to find specific artifacts, check what versions are deployed, or audit what's in use across the organization.

## Quick Decision Guide

| User Request | Action |
|---|---|
| "Create a project" | Use JFrog MCP to create project and repositories |
| "Set up a repository" | Use JFrog MCP to create and configure the repository |
| "Is this package safe?" | Query JFrog Catalog for vulnerabilities and curation status |
| "Find artifact X" | Use AQL search through JFrog MCP |
| "Show security report" | Generate DevSecOps report via JFrog MCP |
| "What vulnerabilities do we have?" | Query security monitoring tools |
| "Check license for package X" | Query JFrog Catalog for license information |

## Troubleshooting

| Problem | Solution |
|---|---|
| MCP tools not available | Verify admin has enabled MCP on the JFrog Platform |
| OAuth window doesn't appear | Restart Cursor completely; check the platform URL is correct |
| "Unauthorized" errors | Re-authenticate by restarting Cursor; check your JFrog permissions |
| Tools return empty results | Verify you have access to the relevant projects/repositories |
| Platform URL unknown | Ask your JFrog administrator or check your organization's JFrog login URL |
