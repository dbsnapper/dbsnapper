# Installation

Get DBSnapper running in your environment with the installation method that best fits your workflow. Whether you're developing locally, deploying in containers, or setting up CI/CD pipelines, we've got you covered.

<p class="img-box">
  <img src="../static/tui/dbs-ui-all-targets.png" alt="DBSnapper Agent User Interface" width="90%">
  <br/>
  <small>DBSnapper Terminal User Interface</small>
</p>

## Choose Your Installation Method

=== "🐳 **Docker (Recommended)**"

    **Perfect for:** Teams, CI/CD, consistent environments, and trying DBSnapper quickly.

    The Docker approach includes all database tools and dependencies pre-installed, making it the most reliable way to run DBSnapper across different environments.

    ### Pull the Latest Image

    ```bash
    docker pull ghcr.io/dbsnapper/dbsnapper:latest
    ```

    ### Quick Start with Docker

    ```bash
    # Interactive shell with all tools included
    docker run -it --rm ghcr.io/dbsnapper/dbsnapper:latest /bin/bash

    # Or run commands directly
    docker run --rm ghcr.io/dbsnapper/dbsnapper:latest dbsnapper --version
    ```

    ### CI/CD Integration

    Use in GitHub Actions:

    ```yaml
    steps:
      - uses: dbsnapper/install-dbsnapper-agent-action@v1
        with:
          version: latest
      - run: dbsnapper build production-db
    ```

    ### Production Deployment

    For production workloads, mount configuration and working directories:

    ```bash
    docker run -d \
      -v ~/.config/dbsnapper:/root/.config/dbsnapper \
      -v ~/.dbsnapper:/root/.dbsnapper \
      -e DBSNAPPER_SECRET_KEY=$SECRET_KEY \
      -e DBSNAPPER_AUTHTOKEN=$AUTH_TOKEN \
      ghcr.io/dbsnapper/dbsnapper:latest
    ```

=== "🍺 **macOS (Homebrew)**"

    **Perfect for:** Local development on macOS with automatic updates.

    ### Install via Homebrew

    ```bash
    # Add the DBSnapper tap
    brew tap dbsnapper/tap

    # Install DBSnapper
    brew install dbsnapper
    ```

    ### Update DBSnapper

    ```bash
    brew upgrade dbsnapper
    ```

    ### Install Database Tools

    You'll also need PostgreSQL and/or MySQL client tools:

    ```bash
    # PostgreSQL tools
    brew install postgresql

    # MySQL tools
    brew install mysql-client
    ```

=== "🐧 **Linux (Package Managers)**"

    **Perfect for:** Linux servers, development machines, and automated deployments.

    ### Debian/Ubuntu (.deb packages)

    ```bash
    # Download latest release
    TAG=$(curl -s https://api.github.com/repos/dbsnapper/dbsnapper/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
    ARCH=linux_x86_64  # or linux_arm64

    wget https://github.com/dbsnapper/dbsnapper/releases/download/v${TAG}/dbsnapper_${ARCH}.deb

    # Install
    sudo dpkg -i dbsnapper_${ARCH}.deb
    ```

    ### RedHat/CentOS/Fedora (.rpm packages)

    ```bash
    # Download latest release
    TAG=$(curl -s https://api.github.com/repos/dbsnapper/dbsnapper/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
    ARCH=linux_x86_64  # or linux_arm64

    wget https://github.com/dbsnapper/dbsnapper/releases/download/v${TAG}/dbsnapper_${ARCH}.rpm

    # Install
    sudo rpm -i dbsnapper_${ARCH}.rpm
    ```

    ### Alpine Linux (.apk packages)

    ```bash
    # Download latest release
    TAG=$(curl -s https://api.github.com/repos/dbsnapper/dbsnapper/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
    ARCH=linux_x86_64  # or linux_arm64

    wget https://github.com/dbsnapper/dbsnapper/releases/download/v${TAG}/dbsnapper_${ARCH}.apk

    # Install
    sudo apk add --allow-untrusted dbsnapper_${ARCH}.apk
    ```

    ### Install Database Client Tools

    ```bash
    # Debian/Ubuntu
    sudo apt-get update
    sudo apt-get install postgresql-client mysql-client

    # RedHat/CentOS/Fedora
    sudo dnf install postgresql mysql

    # Alpine
    sudo apk add postgresql-client mysql-client
    ```

=== "📦 **Direct Binary Download**"

    **Perfect for:** Custom deployments, air-gapped environments, or specific architecture needs.

    ### Download from GitHub Releases

    Visit the [DBSnapper Releases](https://github.com/dbsnapper/dbsnapper/releases) page to download the binary for your platform:

    - **macOS**: `dbsnapper_darwin_universal.tar.gz` (Intel & Apple Silicon)
    - **Linux x64**: `dbsnapper_linux_x86_64.tar.gz`
    - **Linux ARM64**: `dbsnapper_linux_arm64.tar.gz`

    ### Installation Steps

    ```bash
    # Download (replace with your platform)
    wget https://github.com/dbsnapper/dbsnapper/releases/latest/download/dbsnapper_linux_x86_64.tar.gz

    # Extract
    tar -xzf dbsnapper_linux_x86_64.tar.gz

    # Make executable and move to PATH
    chmod +x dbsnapper
    sudo mv dbsnapper /usr/local/bin/
    ```

## Verify Installation

After installation, verify DBSnapper is working correctly:

```bash
# Check version
dbsnapper --version

# Check configuration and environment
dbsnapper config check
```

You should see output similar to:

```
DBSnapper Agent - Version: 3.0.0 (abc123) Build Date: 2024-12-01T10:30:00Z
DBSnapper Cloud: Standalone Mode

Checking DBSnapper Configuration
  ✅ Config file not found - use 'dbsnapper config init' to create
  🔵 Postgres Local Engine (pglocal)
    ✅ psql found at /usr/bin/psql
    ✅ pg_dump found at /usr/bin/pg_dump
    ✅ pg_restore found at /usr/bin/pg_restore
  🔵 MySQL Local Engine (mylocal)
    ✅ mysqldump found at /usr/bin/mysqldump
    ✅ mysql found at /usr/bin/mysql
  🔵 Postgres Docker Engine (pgdocker)
    ✅ Docker client connected
    ⚠️  No docker images configured - Docker engines will use defaults
  🔵 MySQL Docker Engine (mydocker)
    ✅ Docker client connected
    ⚠️  No docker images configured - Docker engines will use defaults
  ✅ All supported database engines available
  ⚠️  DBSnapper Cloud not configured - use 'dbsnapper auth token' or sign up at https://app.dbsnapper.com

  ✅ Installation OK
```

## Development Environment Setup

### IDE Integration

**VS Code Extension** - Install the [DBSnapper Extension](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper) for in-editor snapshot management:

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search for "DBSnapper"
4. Click Install

### MCP Server Setup

For AI assistant integration (Claude Desktop, etc.):

```bash
# Start the MCP server (runs on default port)
dbsnapper mcp

# The server will be available for AI assistant integration
```

## Container Orchestration

### Kubernetes Deployment

Deploy DBSnapper in Kubernetes for scheduled operations:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dbsnapper-backup
spec:
  schedule: "0 2 * * *" # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: dbsnapper
              image: ghcr.io/dbsnapper/dbsnapper:latest
              command: ["dbsnapper", "build", "production-db"]
              env:
                - name: DBSNAPPER_SECRET_KEY
                  valueFrom:
                    secretKeyRef:
                      name: dbsnapper-secret
                      key: secret-key
                - name: DBSNAPPER_AUTHTOKEN
                  valueFrom:
                    secretKeyRef:
                      name: dbsnapper-secret
                      key: auth-token
          restartPolicy: OnFailure
```

### Docker Compose

For local development or small deployments:

```yaml
version: "3.8"
services:
  dbsnapper:
    image: ghcr.io/dbsnapper/dbsnapper:latest
    environment:
      - DBSNAPPER_SECRET_KEY=${SECRET_KEY}
      - DBSNAPPER_AUTHTOKEN=${AUTH_TOKEN}
    volumes:
      - ./config:/root/.config/dbsnapper
      - ./snapshots:/root/.dbsnapper
    command: ["tail", "-f", "/dev/null"] # Keep running for manual commands
```

## Performance Considerations

### CPU Configuration

DBSnapper v3.0 supports configurable CPU usage for optimal performance:

```yaml
# Global default
defaults:
  cpus: 4

# Per-target override
targets:
  large_database:
    cpus: 8 # Use more cores for large databases
    snapshot:
      src_url: "postgresql://user:pass@host:5432/large_db"
```

### Memory Requirements

- **Minimum**: 512MB RAM
- **Recommended**: 2GB+ RAM for large databases
- **Container limits**: Set appropriate memory limits in production deployments

## Troubleshooting Installation

### Common Issues

**"Command not found"** after installation:

```bash
# Check if binary is in PATH
which dbsnapper

# If not found, check installation location and add to PATH
export PATH=$PATH:/usr/local/bin
```

**Docker permission denied**:

```bash
# Add user to docker group (requires logout/login)
sudo usermod -aG docker $USER

# Or use rootless Docker
```

**Database tools not found**:

```bash
# Use Docker mode instead of local tools
dbsnapper config check --prefer-docker

# Or install database client packages (see platform sections above)
```

### Verify Database Connectivity

Test your database connections:

```bash
# Test PostgreSQL connection
psql "postgresql://user:pass@host:5432/dbname" -c "SELECT version();"

# Test MySQL connection
mysql -h host -u user -p -e "SELECT VERSION();"
```

## Next Steps

With DBSnapper installed, you're ready to:

1. **[Initialize your first configuration](quick-start.md)** - Start with our guided quick start
2. **[Set up DBSnapper Cloud](https://app.dbsnapper.com/sign_up)** - Enable team collaboration
3. **[Configure your first target](configuration.md)** - Connect to your databases

**Need help?** Check our **[troubleshooting guide](quick-start.md#need-help)** or **[join the community discussions](https://github.com/dbsnapper/dbsnapper/discussions)**.
