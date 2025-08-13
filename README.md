# ChartDB Database Sync Action

Automatically sync your database schema with [ChartDB](https://chartdb.io) to keep your ER diagrams up-to-date.

## Features

- 🔄 Automatic database schema synchronization
- 📊 Updates ChartDB diagrams in real-time
- 🔒 Secure handling of credentials via GitHub Secrets
- 🐳 Docker-based for consistent execution
- 🗄️ Supports PostgreSQL, MySQL, MariaDB, and MSSQL

## Usage

### Basic Example

```yaml
name: Sync Database with ChartDB
on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * *'  # Daily sync at midnight

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Sync Database Schema
        uses: chartdb/chartdb-action@v1
        with:
          db-host: ${{ secrets.DB_HOST }}
          db-port: '5432'
          db-database: ${{ secrets.DB_DATABASE }}
          db-username: ${{ secrets.DB_USERNAME }}
          db-password: ${{ secrets.DB_PASSWORD }}
          db-type: 'postgresql'
          chartdb-api-token: ${{ secrets.CHARTDB_API_TOKEN }}
          chartdb-diagram-id: ${{ secrets.CHARTDB_DIAGRAM_ID }}
```

### With GitHub-hosted Database

```yaml
name: Sync GitHub Actions Database
on:
  workflow_dispatch:
  push:
    paths:
      - 'migrations/**'
      - 'schema/**'

jobs:
  sync:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      
      - name: Run migrations
        run: |
          # Your migration commands here
          
      - name: Sync with ChartDB
        uses: chartdb/chartdb-action@v1
        with:
          db-host: 'localhost'
          db-port: '5432'
          db-database: 'testdb'
          db-username: 'testuser'
          db-password: 'testpass'
          db-type: 'postgresql'
          chartdb-api-token: ${{ secrets.CHARTDB_API_TOKEN }}
          chartdb-diagram-id: ${{ secrets.CHARTDB_DIAGRAM_ID }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `db-host` | Database host address | Yes | - |
| `db-port` | Database port | No | `5432` |
| `db-database` | Database name | Yes | - |
| `db-username` | Database username | Yes | - |
| `db-password` | Database password (use secrets!) | No | - |
| `db-type` | Database type (`postgresql`, `mysql`, `mariadb`, `mssql`) | Yes | - |
| `chartdb-api-token` | ChartDB API token (use secrets!) | Yes | - |
| `chartdb-diagram-id` | ChartDB diagram ID to sync with | Yes | - |
| `network` | Docker network mode | No | `host` |

## Outputs

| Output | Description |
|--------|-------------|
| `status` | Sync status (`success` or `failed`) |

## Setup

### 1. Get ChartDB Credentials

1. Sign up at [ChartDB](https://chartdb.io)
2. Create or open your diagram
3. Navigate to diagram settings
4. Generate an API token
5. Copy your diagram ID from the URL or settings

### 2. Configure GitHub Secrets

Add the following secrets to your repository:

1. Go to Settings → Secrets and variables → Actions
2. Add these secrets:
   - `DB_HOST`: Your database host
   - `DB_DATABASE`: Your database name
   - `DB_USERNAME`: Database username
   - `DB_PASSWORD`: Database password
   - `CHARTDB_API_TOKEN`: Your ChartDB API token
   - `CHARTDB_DIAGRAM_ID`: Your ChartDB diagram ID

### 3. Add Workflow

Create `.github/workflows/chartdb-sync.yml` in your repository with one of the examples above.

## Security Considerations

- **Always use GitHub Secrets** for sensitive information
- Never commit credentials directly in workflow files
- Consider using environment-specific secrets for different stages
- Use least-privilege database users for syncing
- Rotate API tokens regularly

## Database Support

### PostgreSQL
```yaml
db-type: 'postgresql'
db-port: '5432'  # default
```

### MySQL
```yaml
db-type: 'mysql'
db-port: '3306'  # default
```

### MariaDB
```yaml
db-type: 'mariadb'
db-port: '3306'  # default
```

### Microsoft SQL Server
```yaml
db-type: 'mssql'
db-port: '1433'  # default
```

## Advanced Usage

### Conditional Sync

```yaml
- name: Sync with ChartDB
  if: github.ref == 'refs/heads/main'
  uses: chartdb/chartdb-action@v1
  with:
    # ... your configuration
```

### Multiple Environments

```yaml
jobs:
  sync-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: chartdb/chartdb-action@v1
        with:
          db-host: ${{ secrets.STAGING_DB_HOST }}
          chartdb-diagram-id: ${{ secrets.STAGING_DIAGRAM_ID }}
          # ... other staging config

  sync-production:
    runs-on: ubuntu-latest
    environment: production
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: chartdb/chartdb-action@v1
        with:
          db-host: ${{ secrets.PROD_DB_HOST }}
          chartdb-diagram-id: ${{ secrets.PROD_DIAGRAM_ID }}
          # ... other production config
```

### With Status Check

```yaml
- name: Sync with ChartDB
  id: chartdb-sync
  uses: chartdb/chartdb-action@v1
  with:
    # ... your configuration

- name: Check sync status
  if: steps.chartdb-sync.outputs.status == 'failed'
  run: |
    echo "ChartDB sync failed! Check the logs above."
    exit 1
```

## Troubleshooting

### Connection Issues

If the action fails to connect to your database:

1. **Check network accessibility**: Ensure your database is accessible from GitHub Actions runners
2. **Verify credentials**: Double-check all secrets are correctly set
3. **Firewall rules**: Add GitHub Actions IP ranges to your database firewall if needed
4. **Use self-hosted runners**: For private databases, consider using self-hosted runners

### Docker Network Modes

The action uses `host` network mode by default. For other configurations:

```yaml
with:
  network: 'bridge'  # or 'none', 'container:name', etc.
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT

## Support

- 📧 Email: support@chartdb.io
- 🐛 Issues: [GitHub Issues](https://github.com/chartdb/chartdb-action/issues)
- 📖 Documentation: [ChartDB Docs](https://docs.chartdb.io)

## Related

- [ChartDB](https://chartdb.io) - Visual database design tool
- [ChartDB Syncer](https://github.com/chartdb/chartdb-syncer) - Docker image for database synchronization