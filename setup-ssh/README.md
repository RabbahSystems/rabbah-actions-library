# Setup SSH Action

> 🔐 **Secure SSH private key configuration for automated deployments and remote operations**

## 📋 Description

This composite action securely configures SSH private keys in GitHub Actions workflows, enabling automated deployments, Git operations over SSH, and remote command execution. It handles key decoding, proper permissions, and SSH client configuration.

## ✨ Features

- 🔑 Base64 SSH key decoding and installation
- 🛡️ Secure file permissions (600) automatically set
- ⚙️ SSH config file generation
- 🎯 Configurable host key checking
- 📝 Custom key filename support
- ✅ Setup verification and logging

## 📥 Inputs

| Input                  | Description                     | Required | Default                   | Type     |
| ---------------------- | ------------------------------- | -------- | ------------------------- | -------- |
| `ssh_private_key`      | Base64 encoded SSH private key  | ✅ Yes   | -                         | `string` |
| `key_filename`         | SSH key filename (without path) | ❌ No    | `id_ed25519_deploy_admin` | `string` |
| `strict_host_checking` | Enable strict host key checking | ❌ No    | `no`                      | `string` |

### Input Details

#### `ssh_private_key`

- Must be base64 encoded
- Store in GitHub Secrets
- Supports RSA, ED25519, ECDSA key types

#### `key_filename`

- Filename only (no path required)
- Will be placed in `~/.ssh/` directory
- Examples: `deploy_key`, `id_rsa`, `github_actions_key`

#### `strict_host_checking`

- Options: `yes`, `no`, `accept-new`
- `no`: Disable checking (CI/CD friendly)
- `yes`: Strict checking (more secure)
- `accept-new`: Accept new hosts automatically

## 📤 Outputs

This action provides no explicit outputs but configures:

- SSH private key at `~/.ssh/{key_filename}`
- SSH config at `~/.ssh/config`
- Proper permissions for SSH files

## 🎯 Usage

### Basic Usage

```yaml
name: Deploy to Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup SSH
        uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
        with:
          ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy to Server
        run: |
          ssh user@example.com "cd /app && git pull && npm install"
```

### With Custom Key Filename

```yaml
- name: Setup SSH with Custom Key
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.DEPLOY_SSH_KEY }}
    key_filename: production_deploy_key

- name: Use SSH
  run: ssh deployuser@prod.example.com "systemctl restart app"
```

### Multiple SSH Keys

```yaml
- name: Setup Deploy Key
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.DEPLOY_KEY }}
    key_filename: deploy_key

- name: Setup Git Key
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.GIT_SSH_KEY }}
    key_filename: git_key
```

### With Ansible

```yaml
- uses: RabbahSystems/rabbah-actions-library/setup-ansible@main

- name: Setup SSH for Ansible
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.ANSIBLE_SSH_KEY }}
    key_filename: ansible_key

- name: Run Ansible Playbook
  run: |
    ansible-playbook -i inventory/production \
      --private-key ~/.ssh/ansible_key \
      playbook.yml
```

### Git Operations Over SSH

```yaml
- name: Setup SSH for Git
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.GIT_SSH_KEY }}

- name: Clone Private Repository
  run: |
    git clone git@github.com:your-org/private-repo.git
    cd private-repo
    git checkout develop
```

### With rsync

```yaml
- name: Setup SSH
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}

- name: Sync Files
  run: |
    rsync -avz --delete \
      ./dist/ \
      user@example.com:/var/www/html/
```

## 🔧 Technical Details

### File Structure Created

```
~/.ssh/
├── {key_filename}     # Private key (600 permissions)
└── config            # SSH client config (600 permissions)
```

### SSH Config Template

```
Host *
  IdentityFile ~/.ssh/{key_filename}
  StrictHostKeyChecking {strict_host_checking}
```

### Security Measures

- Private key permissions: `600` (read/write for owner only)
- SSH config permissions: `600`
- Base64 encoding prevents key exposure in logs
- Keys stored in GitHub Secrets

## 🔒 Security Best Practices

### Generating SSH Keys

```bash
# ED25519 (recommended)
ssh-keygen -t ed25519 -C "github-actions@your-org.com" -f deploy_key

# RSA (fallback for older systems)
ssh-keygen -t rsa -b 4096 -C "github-actions@your-org.com" -f deploy_key
```

### Encoding Keys for GitHub Secrets

```bash
# Encode the private key
base64 -w 0 < deploy_key > deploy_key_encoded.txt

# On macOS
base64 -i deploy_key -o deploy_key_encoded.txt
```

### Adding to GitHub Secrets

1. Go to Settings → Secrets → Actions
2. Click "New repository secret"
3. Name: `SSH_PRIVATE_KEY`
4. Value: Paste the base64 encoded content
5. Click "Add secret"

### Key Rotation

Recommended rotation schedule:

- Production keys: Every 90 days
- Development keys: Every 180 days
- Compromised keys: Immediately

## 🐛 Troubleshooting

### Issue: Permission denied (publickey)

**Causes & Solutions:**

1. Wrong key: Verify the correct key is in secrets
2. Key not added to server: Add public key to `~/.ssh/authorized_keys`
3. Wrong username: Check SSH username is correct

### Issue: Host key verification failed

**Solution:** Set `strict_host_checking: "no"` for CI/CD, or add host to known_hosts:

```yaml
- run: ssh-keyscan -H example.com >> ~/.ssh/known_hosts
```

### Issue: Bad base64 encoding

**Solution:** Re-encode the key properly:

```bash
# Linux
base64 -w 0 < private_key

# macOS
base64 < private_key | tr -d '\n'
```

### Issue: Key format not supported

**Solution:** Convert key to OpenSSH format:

```bash
ssh-keygen -p -m PEM -f private_key
```

## 📊 Performance

- **Setup time:** < 2 seconds
- **Memory usage:** Minimal
- **Disk usage:** < 10KB

## 🧪 Testing

### Test SSH Connection

```yaml
- name: Test SSH Connection
  run: |
    ssh -o ConnectTimeout=5 \
        -o ConnectionAttempts=1 \
        user@example.com "echo 'SSH connection successful'"
```

### Verify Setup

```yaml
- name: Verify SSH Setup
  run: |
    ls -la ~/.ssh/
    cat ~/.ssh/config
    ssh-add -l || true
```

## 🔄 Version Compatibility

| Component      | Version             | Notes                  |
| -------------- | ------------------- | ---------------------- |
| GitHub Runners | All                 | Ubuntu, macOS, Windows |
| SSH Protocol   | 2                   | Modern SSH v2          |
| Key Types      | RSA, ED25519, ECDSA | All common types       |
| Base64         | Standard            | RFC 4648               |

## 📝 Advanced Examples

### Multi-Server Deployment

```yaml
- name: Setup SSH
  uses: RabbahSystems/rabbah-actions-library/setup-ssh@main
  with:
    ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}

- name: Deploy to Multiple Servers
  run: |
    for server in web1.example.com web2.example.com db1.example.com; do
      echo "Deploying to $server"
      ssh user@$server "cd /app && ./deploy.sh"
    done
```

### With Jump Host

```yaml
- name: Configure SSH with Jump Host
  run: |
    cat >> ~/.ssh/config << EOF
    Host jump
      HostName jump.example.com
      User jumpuser
      
    Host internal-*
      ProxyJump jump
      User appuser
    EOF

- name: Deploy through Jump Host
  run: ssh internal-web1 "systemctl restart app"
```

## 🤝 Contributing

To improve this action:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📜 Changelog

### v1.0.0 (2025-01-08)

- Initial release
- Base64 key decoding
- SSH config generation
- Configurable host checking

## 🔗 Related Actions

- [setup-ansible](../setup-ansible) - Ansible environment setup
- [setup-vpn](../setup-vpn) - VPN connection (coming soon)
- [setup-kubectl](../setup-kubectl) - Kubernetes access (coming soon)

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
