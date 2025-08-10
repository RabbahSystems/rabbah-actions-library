# Setup Secret Files Action

A GitHub composite action that dynamically constructs secret names and writes decoded secret content to files.

## Description

This action allows you to dynamically construct GitHub secret names using a prefix and suffix pattern, then decode and write the secret content to a specified file. This is particularly useful for managing environment-specific configuration files where secrets follow a naming convention.

## Inputs

| Input      | Description                         | Required | Example                              |
| ---------- | ----------------------------------- | -------- | ------------------------------------ |
| `prefix`   | Prefix to prepend to secret name    | ✅       | `dev`, `uat`, `prod`                 |
| `suffix`   | Suffix to append to secret name     | ✅       | `_env_file`, `_firebase_credentials` |
| `filename` | Filename to create in the workspace | ✅       | `.env`, `firebase-adminsdk.json`     |

## How It Works

1. **Normalizes inputs**: Converts both prefix and suffix to uppercase
2. **Constructs secret name**: Combines `{PREFIX}{SUFFIX}` (e.g., `DEV_ENV_FILE`)
3. **Writes file**: Decodes the secret content and writes it to the specified filename

## Usage

### Basic Example

```yaml
- name: Setup Environment File
  uses: RabbahSystems/rabbah-actions-library/setup-secret-files
  with:
    prefix: dev
    suffix: _env_file
    filename: .env
```

This will:

- Look for a secret named `DEV_ENV_FILE`
- Write the decoded content to `.env`

### Multiple Files Example

```yaml
steps:
  - name: Setup Environment File
    uses: RabbahSystems/rabbah-actions-library/setup-secret-files
    with:
      prefix: ${{ github.ref_name }}
      suffix: _env_file
      filename: .env

  - name: Setup Firebase Credentials
    uses: RabbahSystems/rabbah-actions-library/setup-secret-files
    with:
      prefix: ${{ github.ref_name }}
      suffix: _firebase_credentials
      filename: firebase-adminsdk.json
```

## Secret Naming Convention

This action expects secrets to follow the pattern: `{PREFIX}{SUFFIX}`

### Examples:

- `prefix: "dev"` + `suffix: "_env_file"` = `DEV_ENV_FILE`
- `prefix: "uat"` + `suffix: "_firebase_credentials"` = `UAT_FIREBASE_CREDENTIALS`
- `prefix: "prod"` + `suffix: "_config"` = `PROD_CONFIG`

## Required Secrets

Ensure your repository has the appropriate secrets configured. For the examples above, you would need:

```
DEV_ENV_FILE
UAT_ENV_FILE
PROD_ENV_FILE
DEV_FIREBASE_CREDENTIALS
UAT_FIREBASE_CREDENTIALS
PROD_FIREBASE_CREDENTIALS
```

## Important Notes

- **Secret content should be base64 encoded** in GitHub secrets
- The action will decode the content automatically when writing to files
- Both prefix and suffix are converted to uppercase for consistency
- The action assumes secrets exist - if a secret is missing, the step will fail

## Error Handling

If the constructed secret name doesn't exist in your repository secrets, the action will fail with an error indicating the missing secret.

## File Location

Files are created in the current working directory of the GitHub Actions runner. Use relative paths if you need files in subdirectories:

```yaml
- name: Setup Config in subdirectory
  uses: RabbahSystems/rabbah-actions-library/setup-secret-files
  with:
    prefix: prod
    suffix: _config
    filename: config/app.json
```

## Security Considerations

- Secret content is written to the runner's filesystem temporarily
- Files are accessible to subsequent steps in the same job
- Ensure your secrets contain only the necessary data
- Consider using environment-specific secrets for better security isolation
