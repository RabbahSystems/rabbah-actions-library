# Setup DigitalOcean Token Action

> 🔑 **Decode and export DigitalOcean API token for use in workflows**

## 📋 Description

Simple action that decodes a base64-encoded DigitalOcean API token and exports it as the `DO_API_TOKEN` environment variable for all subsequent steps in your workflow.

## 📥 Inputs

| Input           | Description                           | Required |
| --------------- | ------------------------------------- | -------- |
| `encoded_token` | Base64 encoded DigitalOcean API token | ✅ Yes   |

## 📤 Outputs

Sets `DO_API_TOKEN` environment variable that's available to all subsequent steps in the job.

## 🎯 Usage

### In Workflows

```yaml
- name: Setup DigitalOcean Token
  uses: your-org/shared-actions/setup-do-token@main
  with:
    encoded_token: ${{ secrets.DIGITALOCEAN_TOKEN }}

- name: Use the token
  run: |
    # DO_API_TOKEN is now available
    doctl auth init --access-token $DO_API_TOKEN
```

### In Other Composite Actions

```yaml
- name: Setup DO Token
  uses: your-org/shared-actions/setup-do-token@main
  with:
    encoded_token: ${{ inputs.do_api_token }}

- name: Run Ansible with DO inventory
  run: |
    ansible-playbook -i inventory/digitalocean.yml playbook.yml
    # The DO_API_TOKEN env var is available for the dynamic inventory
```

## 🔐 Creating the Encoded Token

1. Get your DigitalOcean API token from the [DigitalOcean Control Panel](https://cloud.digitalocean.com/account/api/tokens)

2. Encode it:

   ```bash
   echo -n "your-do-api-token-here" | base64
   ```

3. Add to GitHub Secrets:
   - Go to **Settings** → **Secrets and variables** → **Actions**
   - Add secret named `DIGITALOCEAN_TOKEN` with the encoded value

## 💡 Why Base64 Encode?

While GitHub Secrets are already encrypted, base64 encoding adds an extra layer:

- Prevents accidental exposure in logs (even if secret masking fails)
- Makes it explicit that the token needs decoding
- Consistent with other encoded secrets in the infrastructure

## 🔗 Related Actions

- [setup-ansible](../setup-ansible) - Often used together for DigitalOcean inventory
- [configure-ansible](../configure-ansible) - Configures Ansible with DO dynamic inventory

## 📄 License

Internal use only.

---

**Maintained by:** DevOps Team  
**Version:** v1.0.0
