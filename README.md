# 🚀 Shared GitHub Actions Library

A collection of reusable GitHub Actions for our organization's CI/CD workflows.

## 📚 Available Actions

| Action                               | Description                                         | Version  |
| ------------------------------------ | --------------------------------------------------- | -------- |
| [**setup-ansible**](./setup-ansible) | Install and configure Ansible with Python 3.10      | `v1.0.0` |
| [**setup-ssh**](./setup-ssh)         | Configure SSH private key for deployment operations | `v1.0.0` |

## 🎯 Quick Start

To use any action from this repository in your workflow:

```yaml
- uses: RabbahSystems/rabbah-actions-library/action-name@main
  with:
    # action inputs
```

## 📖 Actions Overview

### 🔧 setup-ansible

Automates the installation of Ansible along with Python and pip package management. Includes intelligent caching for faster workflow runs.

**Use Case:** Perfect for infrastructure automation, configuration management, and deployment workflows.

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@main
```

### 🔐 setup-ssh

Securely configures SSH private keys for automated deployments and remote operations.

**Use Case:** Essential for secure server deployments, Git operations over SSH, and remote command execution.

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

## 🏗️ Repository Structure

```
shared-actions/
├── README.md                 # This file
├── setup-ansible/
│   ├── action.yml           # Action definition
│   └── README.md            # Action documentation
├── setup-ssh/
│   ├── action.yml           # Action definition
│   └── README.md            # Action documentation
└── .github/
    └── workflows/
        └── test-actions.yml  # Testing workflows
```

## 🔄 Versioning Strategy

We follow semantic versioning for our actions:

- **Major versions** (`v1`, `v2`): Breaking changes
- **Minor versions** (`v1.1.0`): New features, backward compatible
- **Patch versions** (`v1.0.1`): Bug fixes

### Using Specific Versions

```yaml
# Latest version (recommended for development)
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

# Specific version (recommended for production)
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@v1.0.0

# Major version (gets updates automatically)
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@v1
```

## 🤝 Contributing

### Adding a New Action

1. Create a new directory with your action name
2. Add `action.yml` with the action definition
3. Create a comprehensive `README.md`
4. Update this index README
5. Submit a pull request

### Action Guidelines

- Keep actions focused on a single responsibility
- Use descriptive names and clear documentation
- Include usage examples
- Add input validation where appropriate
- Test thoroughly before merging

## 🔒 Security

- Never commit sensitive data
- Use GitHub Secrets for credentials
- Review security alerts regularly
- Keep dependencies updated

## 📝 License

This repository is private and for internal use only within our organization.

## 🆘 Support

For issues or questions:

1. Check the individual action's README
2. Review existing issues in this repository
3. Contact the DevOps team
4. Create a new issue with the `help-wanted` label

## 📊 Usage Statistics

Track action usage across the organization:

```bash
gh api graphql -f query='
{
  organization(login: "your-org") {
    repositories(first: 100) {
      nodes {
        name
        defaultBranchRef {
          target {
            ... on Commit {
              history(first: 100) {
                nodes {
                  message
                }
              }
            }
          }
        }
      }
    }
  }
}'
```

---

**Maintained by:** DevOps Team  
**Last Updated:** 2025-01-08  
**Status:** ✅ Active
