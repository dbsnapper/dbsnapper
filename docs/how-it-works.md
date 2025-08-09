# How DBSnapper Works

DBSnapper transforms database snapshot management from a manual, error-prone process into an automated, secure workflow that integrates seamlessly with modern development practices.

## The Developer Problem

Traditional approaches to database snapshots face critical challenges:

- **Security risks** - Full production data exposed in development environments
- **Compliance violations** - PII and sensitive data in the wrong hands
- **Manual overhead** - Complex dump/restore processes that developers avoid
- **Team bottlenecks** - Database admins become gatekeepers for realistic test data
- **Infrastructure sprawl** - Multiple database instances for different teams and purposes

**DBSnapper solves these problems with a three-step workflow that keeps your data secure while making realistic datasets accessible to your entire team.**

<p class="img-box">
  <img src="/static/dbs-architecture.jpg" alt="DBSnapper Architecture" width="90%">
  <br/>
  <small>DBSnapper Architecture - Bring Your Own Infrastructure Security Model</small>
</p>

## The DBSnapper Workflow

### Step 1: Snapshot - Capture Production Reality

<div style="text-align: center;">
  <img src="/static/hiw/hiw-snapshot-2x.png" style="width: 120px; float: left; margin-right: 10px;">
</div>

**Create point-in-time database snapshots with zero infrastructure changes.**

DBSnapper connects to your existing databases using read-only credentials and creates compressed snapshots using native database tools (`pg_dump`, `mysqldump`). These snapshots capture the current state of your data, including schema, indexes, and relationships.

**Key Benefits:**

- **No infrastructure changes** - Works with existing database setups
- **High-performance capture** - Multi-core processing with configurable CPU usage
- **Schema filtering** - Capture only the data you need (PostgreSQL)
- **Native compatibility** - Uses standard database tools for maximum compatibility

**Modern Integration:**

- **CI/CD friendly** - Automated snapshot creation in GitHub Actions
- **Container-ready** - Run in Docker, Kubernetes, or serverless environments
- **Cloud storage** - Automatic upload to your S3, R2, or other cloud storage

---

### Step 2: Sanitize - Remove Sensitive Data

<div style="text-align: center;">
  <img src="/static/hiw/hiw-sanitize-2x.png" style="width: 120px; float: right; margin-left: 10px;">
</div>

**Transform production data into development-safe datasets automatically.**

DBSnapper's sanitization engine removes or masks Personally Identifiable Information (PII) and other sensitive data using configurable SQL queries. The process maintains referential integrity while ensuring compliance with data protection regulations.

**Advanced Sanitization Features:**

- **Three-tier priority system** - Target-specific, global, or file-based sanitization rules
- **Ephemeral processing** - No persistent storage of sensitive data during sanitization
- **Template support** - Use environment variables for dynamic sanitization
- **Base64 encoding** - Handle complex SQL queries safely in configuration files

**Compliance & Security:**

- **GDPR/CCPA ready** - Built-in patterns for common compliance requirements
- **Audit trails** - Track what data was sanitized and when
- **Your infrastructure only** - Sensitive data never leaves your environment
- **Encryption at rest** - All configuration and snapshots encrypted with your keys

---

### Step 3: Share - Distribute Securely

<div style="text-align: center;">
  <img src="/static/hiw/hiw-share-2x.png" style="width: 150px; float: left; margin-right: 10px;">
</div>

**Enable your entire team to access realistic, sanitized data instantly.**

Share sanitized snapshots through DBSnapper Cloud with fine-grained access control. Team members can discover, download, and load snapshots directly from their development environment without involving database administrators.

**Team Collaboration Features:**

- **SSO integration** - Use your existing Okta, Auth0, or other SSO provider
- **Group-based access** - Share snapshots with specific teams automatically
- **Presigned URLs** - Secure, time-limited access to snapshot downloads
- **Multi-environment support** - Different snapshots for staging, development, and testing

**Developer Experience:**

- **IDE integration** - Load snapshots directly from VS Code
- **Command-line access** - Simple `dbsnapper load` commands
- **Terraform provider** - Infrastructure as Code for snapshot management
- **API access** - Programmatic integration with your existing tools

---

## Key Architectural Principles

### 🔒 **Security by Design**

**Your Data Stays Yours** - DBSnapper follows a "bring your own infrastructure" model. Your production data never leaves your environment during processing, and sanitized snapshots are stored in your own cloud storage accounts.

### ⚡ **Performance Optimized**

**Multi-Core Processing** - Configure CPU usage per target for optimal performance. PostgreSQL operations can leverage multiple cores for both snapshot creation and restoration.

### 🔧 **Developer-First Experience**

**Zero-Config Operation** - Complex database operations reduced to single commands. Whether you're using Docker, CI/CD pipelines, or your local machine, DBSnapper adapts to your workflow.

### 🏗️ **Enterprise Ready**

**Scale with Your Team** - From individual developers to large engineering organizations, DBSnapper scales with role-based access, audit trails, and integration with your existing security infrastructure.

## Modern Development Integration

### Continuous Integration

Automate snapshot creation in your CI/CD pipelines:

```yaml
# GitHub Actions example
- uses: dbsnapper/install-dbsnapper-agent-action@v1
- run: dbsnapper sanitize production-db --new-set
```

### Infrastructure as Code

Manage snapshots with Terraform:

```hcl
resource "dbsnapper_target" "api_db" {
  name = "production-api"
  sanitization_rules = ["remove_pii.sql"]
  storage_profile = "team-s3"
}
```

### AI Assistant Integration

Use the MCP server for AI-powered database operations:

```bash
dbsnapper mcp  # Start MCP server for Claude Desktop
```

## Next Steps

Ready to implement this workflow in your environment?

1. **[Install DBSnapper](installation.md)** - Get started with the method that fits your setup
2. **[Quick Start Guide](quick-start.md)** - Choose your path and get running in minutes
3. **[Join DBSnapper Cloud](https://app.dbsnapper.com/sign_up)** - Enable team collaboration and secure sharing

**Questions?** Check our **[Philosophy](philosophy.md)** page to understand the principles behind DBSnapper's design, or dive into the **[Configuration Guide](configuration.md)** for advanced setup options.
