# Install WireGuard GitHub Action

A composite GitHub Action to install and configure WireGuard VPN, enabling access to internal services and private networks from GitHub Actions workflows.

## Description

This action installs WireGuard on Ubuntu runners, configures a VPN connection using a base64-encoded peer configuration file, and establishes a secure tunnel to your private network. Perfect for accessing internal GCP resources, private databases, or any infrastructure behind a VPN during CI/CD workflows.

## Features

- ✅ Installs WireGuard and required tools
- ✅ Configures VPN using base64-encoded configuration
- ✅ Supports custom network interface names
- ✅ Displays connection status and routing information
- ✅ Lightweight and fast setup
- ✅ Works on `ubuntu-latest` runners

## Usage

### Basic Example

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Setup WireGuard VPN
    uses: ./.github/actions/setup-wireguard # Adjust path to your action
    with:
      config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}

  - name: Access internal resources
    run: |
      # Now you can access internal services
      curl http://internal-api.local
      ssh user@internal-server
```

### Custom Interface Name

```yaml
steps:
  - name: Setup WireGuard VPN
    uses: ./.github/actions/setup-wireguard
    with:
      config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}
      interface: wg1
```

### Complete Workflow Example

```yaml
name: Deploy to Internal Infrastructure

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup WireGuard VPN
        uses: ./.github/actions/setup-wireguard
        with:
          config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}

      - name: Wait for connection
        run: sleep 3

      - name: Test connectivity
        run: |
          ping -c 3 10.0.0.1
          echo "VPN connection established!"

      - name: Deploy application
        run: |
          # SSH to internal server
          ssh user@10.0.0.10 "cd /app && ./deploy.sh"

      - name: Cleanup
        if: always()
        run: sudo wg-quick down wg0 || true
```

## Inputs

| Input       | Description                                      | Required | Default |
| ----------- | ------------------------------------------------ | -------- | ------- |
| `config`    | Base64-encoded WireGuard peer configuration file | Yes      | -       |
| `interface` | Network interface name for WireGuard             | No       | `wg0`   |

## Prerequisites

### 1. WireGuard Configuration File

Create a WireGuard client configuration file (`wg0.conf`):

```ini
[Interface]
PrivateKey = YOUR_PRIVATE_KEY_HERE
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = SERVER_PUBLIC_KEY_HERE
Endpoint = vpn.example.com:51820
AllowedIPs = 10.0.0.0/24, 192.168.0.0/16
PersistentKeepalive = 25
```

### 2. Encode Configuration to Base64

```bash
# Linux
cat wg0.conf | base64 -w 0

# macOS
cat wg0.conf | base64
```

### 3. Add to GitHub Secrets

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `WIREGUARD_CONFIG_BASE64`
4. Value: Paste the base64-encoded string
5. Click **Add secret**

## Configuration File Format

Your WireGuard configuration file should follow this structure:

```ini
[Interface]
PrivateKey = <client_private_key>
Address = <client_vpn_ip>/24
DNS = <dns_server> (optional)

[Peer]
PublicKey = <server_public_key>
Endpoint = <server_ip_or_hostname>:<port>
AllowedIPs = <networks_to_route_through_vpn>
PersistentKeepalive = 25
```

### Example for GCP Internal Network Access

```ini
[Interface]
PrivateKey = cGFPdGhpcyBpcyBub3QgYSByZWFsIGtleQ==
Address = 10.0.0.2/24

[Peer]
PublicKey = c2VydmVyIHB1YmxpYyBrZXkgZ29lcyBoZXJl
Endpoint = 34.123.45.67:51820
AllowedIPs = 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
PersistentKeepalive = 25
```

## Troubleshooting

### Connection Issues

If the VPN doesn't connect properly:

```yaml
- name: Debug WireGuard
  run: |
    sudo wg show
    sudo journalctl -u wg-quick@wg0 -n 50
    ip route
```

### DNS Resolution Issues

If you can't resolve hostnames:

```yaml
- name: Check DNS
  run: |
    cat /etc/resolv.conf
    nslookup internal-host.local
```

### Firewall Issues

Ensure your WireGuard server allows connections from GitHub Actions IP ranges:

```bash
# On your WireGuard server
sudo ufw allow 51820/udp
```

### Connection Timeout

Add a wait step after establishing the VPN:

```yaml
- name: Setup WireGuard VPN
  uses: ./.github/actions/setup-wireguard
  with:
    config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}

- name: Wait for VPN handshake
  run: |
    for i in {1..10}; do
      if sudo wg show wg0 | grep -q "latest handshake"; then
        echo "✅ VPN connected"
        break
      fi
      echo "Waiting for handshake... ($i/10)"
      sleep 2
    done
```

## Security Best Practices

1. **Rotate Keys Regularly**: Generate new WireGuard key pairs periodically
2. **Limit Access**: Use `AllowedIPs` to restrict which networks are accessible
3. **Use Environment-Specific Secrets**: Create separate VPN configs for dev/staging/prod
4. **Clean Up**: Always disconnect VPN in the cleanup step:

```yaml
- name: Cleanup
  if: always()
  run: sudo wg-quick down wg0 || true
```

## Advanced Usage

### Multiple VPN Connections

```yaml
- name: Setup Dev VPN
  uses: ./.github/actions/setup-wireguard
  with:
    config: ${{ secrets.DEV_WIREGUARD_CONFIG }}
    interface: wg0

- name: Setup Prod VPN
  uses: ./.github/actions/setup-wireguard
  with:
    config: ${{ secrets.PROD_WIREGUARD_CONFIG }}
    interface: wg1
```

### With SSH Access

```yaml
- name: Setup SSH Key
  run: |
    mkdir -p ~/.ssh
    echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa

- name: Setup WireGuard VPN
  uses: ./.github/actions/setup-wireguard
  with:
    config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}

- name: Deploy via SSH
  run: |
    ssh-keyscan 10.0.0.10 >> ~/.ssh/known_hosts
    ssh user@10.0.0.10 "docker-compose up -d"
```

### With Ansible

```yaml
- name: Setup WireGuard VPN
  uses: ./.github/actions/setup-wireguard
  with:
    config: ${{ secrets.WIREGUARD_CONFIG_BASE64 }}

- name: Run Ansible Playbook
  run: |
    sudo apt-get install -y ansible
    ansible-playbook -i inventory.ini playbook.yml
```

## Limitations

- Only works on Ubuntu runners (`ubuntu-latest`, `ubuntu-22.04`, `ubuntu-20.04`)
- Requires `sudo` access (available by default on GitHub-hosted runners)
- WireGuard server must be accessible from GitHub Actions IP ranges

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This action is available under the MIT License.

## Related Resources

- [WireGuard Official Documentation](https://www.wireguard.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Composite Actions Guide](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

## Support

For issues and questions:

- Open an issue in this repository
- Check existing issues for solutions
- Review the troubleshooting section above

---

**Note**: Always ensure your WireGuard configuration is kept secure and never commit it directly to your repository. Use GitHub Secrets for sensitive data.
