# Assign Environment Variables

A GitHub Action that sets environment variables from YAML key-value pairs for use in subsequent workflow steps.

## Description

This action parses a YAML input containing key-value pairs and assigns them as environment variables that can be accessed by subsequent steps in the same job.

## Inputs

| Input      | Description                                | Required |
| ---------- | ------------------------------------------ | -------- |
| `env-vars` | YAML key-value pairs to add to environment | Yes      |

## Usage

```yaml
- name: Setup Environment Variables
  uses: RabbahSystems/rabbah-actions-library/setup-env-vars@main
  with:
    env-vars: |
      - name: DATABASE_URL
        value: "postgres://localhost:5432/mydb"
      - name: API_KEY
        value: ${{ secrets.API_KEY }}
      - name: NODE_ENV
        value: "production"
```

## YAML Format

The `env-vars` input expects a YAML array where each item has:

- `name`: The environment variable name
- `value`: The environment variable value

```yaml
env-vars: |
  - name: VAR_NAME_1
    value: "variable value 1"
  - name: VAR_NAME_2
    value: "variable value 2"
```

## Examples

### Basic usage

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-env-vars@main
  with:
    env-vars: |
      - name: APP_ENV
        value: "staging"
      - name: DEBUG_MODE
        value: "true"

- name: Use environment variables
  run: |
    echo "App environment: $APP_ENV"
    echo "Debug mode: $DEBUG_MODE"
```

### With secrets

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-env-vars@main
  with:
    env-vars: |
      - name: DATABASE_PASSWORD
        value: ${{ secrets.DB_PASSWORD }}
      - name: API_ENDPOINT
        value: "https://api.example.com"

- name: Connect to database
  run: ./connect-db.sh
```

### Multiple environments

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-env-vars@main
  with:
    env-vars: |
      - name: REDIS_URL
        value: "redis://localhost:6379"
      - name: QUEUE_NAME
        value: "production-queue"
      - name: WORKER_TIMEOUT
        value: "300"
```

