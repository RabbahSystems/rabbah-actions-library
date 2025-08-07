# Setup Ansible Action

> 🔧 **Automated Ansible installation with Python environment setup and intelligent caching**

## 📋 Description

This composite action sets up a complete Ansible environment for your GitHub Actions workflows. It handles Python installation, pip package management with caching, and installs Ansible along with commonly needed dependencies.

## ✨ Features

- 🐍 Python 3.10 installation via official setup-python action
- 💾 Intelligent pip cache management for faster subsequent runs
- 📦 Installs Ansible with additional packages (requests, google-auth)
- ✅ Automatic version verification
- 🚀 Zero configuration required

## 📥 Inputs

This action requires no inputs and works out of the box.

## 📤 Outputs

This action provides no explicit outputs but sets up the environment with:

- Python 3.10 available in PATH
- pip (latest version)
- ansible
- requests library
- google-auth library

## 🎯 Usage

### Basic Usage

```yaml
name: Deploy Infrastructure

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Ansible
        uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

      - name: Run Ansible Playbook
        run: |
          ansible-playbook -i inventory/production playbook.yml
```

### With Ansible Vault

```yaml
- uses: actions/checkout@v4

- name: Setup Ansible
  uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Create Vault Password File
  run: echo "${{ secrets.ANSIBLE_VAULT_PASSWORD }}" > .vault_pass

- name: Run Encrypted Playbook
  run: |
    ansible-playbook \
      --vault-password-file .vault_pass \
      -i inventory/production \
      encrypted-playbook.yml
```

### With Dynamic Inventory

```yaml
- uses: actions/checkout@v4

- name: Setup Ansible
  uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- name: Run with Dynamic Inventory
  run: |
    ansible-playbook -i inventory/aws_ec2.yml playbook.yml
```

### With Galaxy Requirements

```yaml
- uses: actions/checkout@v4

- name: Setup Ansible
  uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Install Galaxy Requirements
  run: |
    ansible-galaxy collection install -r requirements.yml
    ansible-galaxy role install -r requirements.yml

- name: Run Playbook
  run: ansible-playbook -i inventory/production site.yml
```

## 🔧 Technical Details

### Python Version

- Uses Python 3.10 (stable and widely compatible)
- Managed via `actions/setup-python@v5`

### Caching Strategy

- Caches pip packages based on requirements.txt hash
- Cache key: `${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}`
- Falls back to OS-specific cache if no exact match

### Installed Packages

```
ansible       # Latest version
requests      # HTTP library for API interactions
google-auth   # Google Cloud authentication
```

## 🐛 Troubleshooting

### Issue: Cache not being used

**Solution:** Ensure you have a `requirements.txt` file in your repository, even if empty, for consistent cache keys.

### Issue: Ansible version conflicts

**Solution:** If you need a specific Ansible version, fork this action and modify the pip install command:

```bash
pip install ansible==2.10.7 requests google-auth
```

### Issue: Missing Python dependencies

**Solution:** You can install additional packages after this action:

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Install Additional Dependencies
  run: pip install boto3 azure-cli
```

## 📊 Performance

Typical execution times:

- **First run:** 45-60 seconds (downloads and installs everything)
- **Cached runs:** 5-10 seconds (restores from cache)

## 🔄 Version Compatibility

| Component     | Version                                     | Notes               |
| ------------- | ------------------------------------------- | ------------------- |
| Python        | 3.10                                        | LTS version, stable |
| pip           | Latest                                      | Auto-upgraded       |
| Ansible       | Latest                                      | Use latest stable   |
| GitHub Runner | ubuntu-latest, macOS-latest, windows-latest | Tested on all       |

## 🤝 Contributing

To modify this action:

1. Fork the repository
2. Make changes to `action.yml`
3. Test thoroughly
4. Submit a pull request

## 📜 Changelog

### v1.0.0 (2025-01-08)

- Initial release
- Python 3.10 setup
- Pip caching
- Ansible installation with requests and google-auth

## 🔗 Related Actions

- [setup-ssh](../setup-ssh) - Configure SSH for remote deployments
- [setup-terraform](../setup-terraform) - Terraform installation (coming soon)
- [setup-kubectl](../setup-kubectl) - Kubernetes CLI setup (coming soon)

## 📄 License

Internal use only. Property of [Your Organization].

## 💬 Support

For issues or questions:

- Create an issue in this repository
- Contact: devops-team@your-org.com
- Slack: #devops-support

---

**Maintained by:** DevOps Team  
**Action Version:** v1.0.0  
**Last Updated:** 2025-01-08
