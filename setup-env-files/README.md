# Setup Environment Files

A composite GitHub Action that creates environment files from YAML key-value pairs with base64 encoded values.

## Description

This action takes YAML-formatted key-value pairs where values are base64 encoded, decodes them, and writes them to an environment file (default: `.env`). It's useful for securely passing environment variables to workflows while keeping sensitive values encoded.

## Inputs

| Input      | Description                                     | Required | Default |
| ---------- | ----------------------------------------------- | -------- | ------- |
| `filename` | Filename to write to                            | No       | `.env`  |
| `env-vars` | YAML key/value pairs to add to environment file | Yes      | -       |

## Usage

```yaml
- name: Setup Environment Files
  uses: ./setup-env-files
  with:
    filename: ".env"
    env-vars: |
      - key: "DATABASE_URL"
        value: "cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bsb2NhbGhvc3Q6NTQzMi9kYg=="
      - key: "API_KEY"
        value: "bXlfc2VjcmV0X2FwaV9rZXk="
```

## How It Works

1. Installs `yq` tool for YAML parsing
2. Writes the input YAML to a temporary `config.yml` file
3. Processes each key-value pair:
   - Extracts the key and base64-encoded value
   - Decodes the value using `base64 -d`
   - Writes `key=decoded_value` to the specified file

## Requirements

- Linux runner (uses `yq_linux_amd64`)
- `sudo` access for installing `yq`
- Base64 encoded values in the YAML input

## Example Output

Given the usage example above, the resulting `.env` file would contain:

```
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=my_secret_api_key
```

## Security Considerations

- Values should be base64 encoded before passing to this action
- The temporary `config.yml` file contains encoded values and is created in the workspace
- Decoded values are written to the specified environment file
- Use GitHub secrets for sensitive data and encode them before passing to this action

