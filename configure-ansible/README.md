# Configure Ansible Action

> ⚙️ **Intelligent Ansible configuration with pre-flight checks and parameter management**

## 📋 Description

This composite action handles all the repetitive Ansible configuration tasks, including parameter setup, pre-flight checks, inventory verification, and connectivity testing. It eliminates boilerplate code and ensures consistent Ansible execution across all your workflows.

## ✨ Features

- 🎯 Dynamic parameter configuration (limit, verbosity, check mode)
- 📋 Automatic inventory verification with host counting
- 🔌 Built-in connectivity testing
- 📊 Beautiful configuration display with GitHub Actions annotations
- 🔄 Extra variables support
- 📤 Outputs for use in subsequent steps
- 🎨 Color-coded terminal output with collapsible groups
- 📝 Job summary generation for workflow visibility

## 📥 Inputs

| Input               | Description                                            | Required | Default | Type      |
| ------------------- | ------------------------------------------------------ | -------- | ------- | --------- |
| `hosts`             | Target hosts or host group                             | ✅ Yes   | -       | `string`  |
| `inventory`         | Path to Ansible inventory file                         | ✅ Yes   | -       | `string`  |
| `limit`             | Limit execution to specific hosts                      | ❌ No    | `''`    | `string`  |
| `verbosity`         | Ansible verbosity level (`-v`, `-vv`, `-vvv`, `-vvvv`) | ❌ No    | `''`    | `string`  |
| `dry_run`           | Run in check mode                                      | ❌ No    | `false` | `boolean` |
| `test_connectivity` | Test connectivity before execution                     | ❌ No    | `true`  | `boolean` |
| `verify_inventory`  | Verify and display inventory                           | ❌ No    | `true`  | `boolean` |
| `display_config`    | Display run configuration                              | ❌ No    | `true`  | `boolean` |
| `extra_vars`        | Additional variables (key=value format)                | ❌ No    | `''`    | `string`  |

## 📤 Outputs

| Output            | Description                         | Example                      |
| ----------------- | ----------------------------------- | ---------------------------- |
| `limit_flag`      | The limit flag for ansible commands | `--limit web-01,web-02`      |
| `verbosity_flag`  | The verbosity flag                  | `-vv`                        |
| `check_flag`      | The check mode flag                 | `--check`                    |
| `extra_vars_flag` | Extra variables flag                | `--extra-vars env=prod`      |
| `all_flags`       | All options combined                | `--limit web-01 -vv --check` |

## 🎯 Usage Examples

### Basic Usage

```yaml
name: Deploy Application

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
        with:
          ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}

      - uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

      - name: Configure Ansible
        id: ansible_config
        uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
        with:
          hosts: webservers
          inventory: inventory/hosts.yml

      - name: Deploy
        run: |
          ansible-playbook playbooks/deploy.yml \
            -i inventory/hosts.yml \
            --extra-vars "hosts=webservers" \
            ${{ steps.ansible_config.outputs.all_flags }}
```

### With All Options

```yaml
- name: Configure Ansible with Full Control
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: all
    inventory: inventory/production.yml
    limit: "web-*,!web-staging"
    verbosity: "-vv"
    dry_run: true
    test_connectivity: true
    verify_inventory: true
    display_config: true
    extra_vars: "env=prod version=1.2.3"

- name: Run Playbook
  run: |
    ansible-playbook playbooks/site.yml \
      -i inventory/production.yml \
      --extra-vars "hosts=all" \
      ${{ steps.ansible_config.outputs.limit_flag }} \
      ${{ steps.ansible_config.outputs.check_flag }} \
      ${{ steps.ansible_config.outputs.verbosity_flag }} \
      ${{ steps.ansible_config.outputs.extra_vars_flag }}
```

### Minimal Configuration (Skip Checks)

```yaml
- name: Quick Ansible Setup
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: localhost
    inventory: inventory/local.yml
    test_connectivity: false
    verify_inventory: false
    display_config: false

- name: Run Local Playbook
  run: |
    ansible-playbook playbooks/local.yml \
      -i inventory/local.yml \
      ${{ steps.ansible_config.outputs.all_flags }}
```

### Using Individual Output Flags

```yaml
- name: Configure Ansible
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: databases
    inventory: inventory/prod.yml
    limit: "db-primary-*"
    dry_run: true

- name: Run Database Maintenance
  run: |
    # Use individual flags for more control
    ansible-playbook playbooks/db-maintenance.yml \
      -i inventory/prod.yml \
      --extra-vars "hosts=databases" \
      ${{ steps.ansible_config.outputs.limit_flag }} \
      ${{ steps.ansible_config.outputs.check_flag }}

- name: Run Database Backup (always, even in dry-run)
  run: |
    # Selectively use flags - skip check_flag for backup
    ansible-playbook playbooks/db-backup.yml \
      -i inventory/prod.yml \
      --extra-vars "hosts=databases" \
      ${{ steps.ansible_config.outputs.limit_flag }}
```

## 🔧 Advanced Usage

### Dynamic Inventory with Cloud Providers

```yaml
- name: Setup SSH and Ansible
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}

- uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Configure for AWS Dynamic Inventory
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: "tag_Environment_production"
    inventory: inventory/aws_ec2.yml
    limit: "tag_Type_webserver"
    extra_vars: "ansible_user=ec2-user"

- name: Deploy to AWS
  run: |
    ansible-playbook playbooks/aws-deploy.yml \
      -i inventory/aws_ec2.yml \
      --extra-vars "hosts=tag_Environment_production" \
      ${{ steps.ansible_config.outputs.all_flags }}
```

### Conditional Execution Based on Environment

```yaml
- name: Configure Ansible
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: ${{ inputs.environment }}-servers
    inventory: inventory/${{ inputs.environment }}.yml
    dry_run: ${{ inputs.environment != 'prod' }}
    verbosity: ${{ inputs.environment == 'dev' && '-vv' || '' }}

- name: Deploy
  run: |
    ansible-playbook playbooks/deploy.yml \
      -i inventory/${{ inputs.environment }}.yml \
      --extra-vars "hosts=${{ inputs.environment }}-servers" \
      ${{ steps.ansible_config.outputs.all_flags }}
```

## 🎨 GitHub Actions UI Features

### Visibility Enhancements

The action uses GitHub Actions special commands for better visibility:

#### 1. **Annotations** (Appear as highlighted messages)

- Blue notices for configuration info
- Yellow warnings for dry-run mode
- Red errors for critical issues

#### 2. **Collapsible Groups** (Organize long output)

```
▼ 📦 Ansible Configuration Summary
  Target Hosts: webservers
  Combined Flags: --limit web-01,web-02 --check
```

#### 3. **Job Summary** (Appears in Summary tab)

A formatted markdown table with your configuration appears in the workflow summary

### Output Example in GitHub Actions UI

```
▼ Configure Ansible and Run Pre-flight Checks
  ▼ Configure Ansible Options
    ℹ️ Ansible Configuration: Starting configuration for hosts: webservers
    🔧 Configuring Ansible execution parameters...
    ✓ Limit configured: web-01,web-02
    ⚠️ Dry Run Mode: This is a DRY RUN - no changes will be made
    ▼ 📦 Configuration Summary
      Combined Flags: --limit web-01,web-02 --check
  ▼ Display Run Configuration
    ╔══════════════════════════════════════════════════════╗
    ║           Ansible Run Configuration                  ║
    ╚══════════════════════════════════════════════════════╝
```

## 🐛 Troubleshooting

### Issue: Output flags not available in my workflow

**Solution:** Make sure you set an `id` for the configure-ansible step:

```yaml
- name: Configure Ansible
  id: ansible_config # This ID is required!
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
```

Then reference outputs using: `${{ steps.ansible_config.outputs.flag_name }}`

### Issue: Inventory verification fails

**Solution:** Ensure the inventory file path is relative to the repository root and the file exists. The action runs from your repository root.

### Issue: Connectivity test fails but hosts are reachable

**Solution:** This might be expected for new hosts that haven't been provisioned yet. The action continues execution even if connectivity fails. You can disable the test:

```yaml
test_connectivity: false
```

## 🔄 Migration Guide

### Before (Repetitive Code)

```yaml
- name: Setup Variables
  run: |
    if [ -n "${{ inputs.limit }}" ]; then
      echo "ANSIBLE_LIMIT=--limit ${{ inputs.limit }}" >> $GITHUB_ENV
    else
      echo "ANSIBLE_LIMIT=" >> $GITHUB_ENV
    fi

    if [ -n "${{ inputs.verbosity }}" ]; then
      echo "ANSIBLE_VERBOSITY=${{ inputs.verbosity }}" >> $GITHUB_ENV
    else
      echo "ANSIBLE_VERBOSITY=" >> $GITHUB_ENV
    fi

    if [ "${{ inputs.dry_run }}" = "true" ]; then
      echo "ANSIBLE_CHECK=--check" >> $GITHUB_ENV
    else
      echo "ANSIBLE_CHECK=" >> $GITHUB_ENV
    fi

- name: Display Config
  run: |
    echo "Target: ${{ inputs.hosts }}"
    echo "Limit: ${{ inputs.limit }}"
    # ... more display code ...

- name: Test Connectivity
  run: |
    ansible ${{ inputs.hosts }} -m ping
    # ... more test code ...
```

### After (Using This Action)

```yaml
- name: Configure Ansible and Run Pre-flight Checks
  id: ansible_config
  uses: RabbahSystems/rabbah-actions-library/configure-ansible@main
  with:
    hosts: ${{ inputs.hosts }}
    inventory: inventory/hosts.yml
    limit: ${{ inputs.limit }}
    verbosity: ${{ inputs.verbosity }}
    dry_run: ${{ inputs.dry_run }}

# Use the outputs in your ansible commands
- name: Run Playbook
  run: |
    ansible-playbook playbook.yml \
      -i inventory/hosts.yml \
      ${{ steps.ansible_config.outputs.all_flags }}
```

## 🏗️ Integration with Other Actions

Works seamlessly with:

- [setup-ansible](../setup-ansible) - Install Ansible environment
- [setup-ssh](../setup-ssh) - Configure SSH keys for remote access

**Important:** Always run `setup-ansible` and `setup-ssh` BEFORE `configure-ansible` in your workflow.

## 📊 Performance Impact

- **Configuration:** < 1 second
- **Inventory verification:** 2-5 seconds (depends on inventory size)
- **Connectivity test:** 5-30 seconds (depends on host count and network)
- **Total overhead:** Typically under 30 seconds

## 🤝 Contributing

To enhance this action:

1. Fork the repository
2. Create a feature branch
3. Add your improvements
4. Submit a pull request

## 📜 Changelog

### v1.0.0 (2025-01-08)

- Initial release
- Dynamic parameter configuration
- Pre-flight checks (inventory, connectivity)
- GitHub Actions UI enhancements (annotations, groups, summary)
- Multiple output formats for flexibility

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
